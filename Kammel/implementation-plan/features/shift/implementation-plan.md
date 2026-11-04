# Shift — خطة التنفيذ (P1)

> **الـ Phase:** Post-MVP · **المدة:** 3–4 أيام · **الترتيب:** بعد `payment` وقبل تقارير الورديات  
> **المرتبط:** [`README.md`](./README.md) · [payment/domain-design](../payment/domain-design.md)

---

## 1. الـ Domain

### 1.1 `CashShift`

| الحقل | النوع | القيود |
|---|---|---|
| `id` | `UUID` | PK |
| `orgId` | `UUID` | `NOT NULL` |
| `branchId` | `UUID` | `NOT NULL` |
| `userId` | `UUID` | `NOT NULL` — صاحب الوردية |
| `openedAt` | `Instant` | `NOT NULL` |
| `closedAt` | `Instant` | `NULL` — `NULL` = مفتوحة |
| `closedBy` | `UUID` | `NULL` — مختلف عن `userId` في Force Close |
| `expectedCashMinor` | `Long` | `NULL` قبل الإغلاق |
| `countedCashMinor` | `Long` | `NULL` قبل الإغلاق |
| `varianceMinor` | `Long` | `NULL` قبل الإغلاق |
| `notes` | `String` | `NULL`, ≤ 500 |

```java
@Entity
@Table(name = "cash_shifts")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class CashShift extends BaseEntity implements TenantScoped {

    private UUID orgId;
    private UUID branchId;
    private UUID userId;
    private Instant openedAt;
    private Instant closedAt;
    private UUID closedBy;
    private Long expectedCashMinor;
    private Long countedCashMinor;
    private Long varianceMinor;

    @Column(length = 500)
    private String notes;

    public boolean isOpen() {
        return closedAt == null;
    }

    public void close(long expectedMinor, long countedMinor, UUID actorId, Instant now) {
        if (!isOpen()) throw new ShiftAlreadyClosedException();
        this.expectedCashMinor = expectedMinor;
        this.countedCashMinor = countedMinor;
        this.varianceMinor = countedMinor - expectedMinor;
        this.closedAt = now;
        this.closedBy = actorId;
    }
}
```

> **ممنوع Setters** بعد الإغلاق. الـ DB Trigger بيمنع أي `UPDATE` على وردية مقفولة.

### 1.2 الـ Indexes

```sql
CREATE UNIQUE INDEX uk_cash_shifts_open_per_user
    ON cash_shifts (org_id, user_id)
    WHERE closed_at IS NULL;

CREATE INDEX idx_cash_shifts_org_branch_opened
    ON cash_shifts (org_id, branch_id, opened_at DESC);
```

- `uk_cash_shifts_open_per_user` بيضمن **BR-029** تحت التزامن.
- `WHERE closed_at IS NULL` — Partial Unique.

### 1.3 ربط الدفع

- `payments.shift_id` nullable في P0 (SC-20).
- في P1:
  - `PaymentService.record` لو `method = CASH`:
    - لازم `shiftId` أو وردية مفتوحة للموظف.
    - لو مفيش → `422 NO_OPEN_SHIFT`.
- `V20__add_payments_shift_fk.sql` تضيف الـ FK بعد إنشاء `cash_shifts`.

### 1.4 الحسابات

```text
expectedCashMinor =
    Σ payments.amount_minor
      WHERE method = 'CASH' AND status = 'COMPLETED'
        AND shift_id = :shiftId
  − Σ refunds.amount_minor
      WHERE method = 'CASH' AND status = 'COMPLETED'
        AND shift_id = :shiftId        // P1 Refund

varianceMinor = countedCashMinor − expectedCashMinor
```

- Non-cash (InstaPay / Bank Transfer / Online) **مستبعد**.
- `countedCashMinor` إدخال يدوي.
- ممنوع حساب المتوقع في الـ Frontend.

### 1.5 Blind Count

- `GET /shifts/current` **مش** بيرجع `expectedCashMinor` قبل الإغلاق.
- بيرجع:
  - `openedAt`
  - `cashPaymentsCount` (عدد، مش مبلغ)
  - `status`
- بعد الإغلاق → كل الحقول.

---

## 2. الـ API

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/shifts/open` | Owner, Manager, Receptionist | فتح وردية |
| GET | `/shifts/current` | Owner, Manager, Receptionist | الوردية المفتوحة |
| POST | `/shifts/{id}/close` | Owner, Manager, Receptionist (صاحبها) | إغلاق |
| POST | `/shifts/{id}/force-close` | Owner, Manager | إغلاق إجباري |
| GET | `/shifts` | Owner, Manager | سجل الورديات |
| GET | `/shifts/{id}` | Owner, Manager | تفاصيل |

### 2.1 `POST /shifts/open`

```json
{
  "branchId": "99999999-8888-7777-6666-555555555555",
  "notes": "Evening shift"
}
```

- `branchId` مطلوب للـ Owner، ومشتق من الـ Token لغيره.
- لو فيه وردية مفتوحة → `409 SHIFT_ALREADY_OPEN`.
- Response `201 Created`:
```json
{
  "success": true,
  "message": "Shift opened",
  "data": {
    "id": "12345678-...",
    "branchId": "99999999-...",
    "userId": "b2c3d4e5-...",
    "status": "OPEN",
    "openedAt": "2026-11-03T14:00:00Z"
  }
}
```

### 2.2 `GET /shifts/current`

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "12345678-...",
    "status": "OPEN",
    "openedAt": "2026-11-03T14:00:00Z",
    "cashPaymentsCount": 12
  }
}
```

- مفيش وردية → `data: null`.

### 2.3 `POST /shifts/{id}/close`

```json
{
  "countedCashMinor": 450000,
  "notes": "Counted with manager"
}
```

- `countedCashMinor >= 0`.
- Response:
```json
{
  "success": true,
  "message": "Shift closed",
  "data": {
    "id": "12345678-...",
    "status": "CLOSED",
    "expectedCashMinor": 450000,
    "countedCashMinor": 448000,
    "varianceMinor": -2000,
    "varianceFormatted": "-20.00 EGP",
    "closedAt": "2026-11-03T22:00:00Z"
  }
}
```

- `varianceMinor` سالب = Shortage، موجب = Overage.
- Audit `SHIFT_CLOSED`.

### 2.4 `POST /shifts/{id}/force-close`

- Manager/Owner فقط.
- Body: `countedCashMinor` اختياري + `reason` إلزامي.
- لو مفيش عدّ → `countedCashMinor = null` و`varianceMinor = null`.
- Audit `SHIFT_FORCE_CLOSED` + Reason.

### 2.5 `GET /shifts`

**Query:** `branchId`, `userId`, `from`, `to`, `status`, `hasVariance`, `page`, `size`.

- Manager → فرعه.
- Response يشمل `varianceMinor` + `varianceFormatted`.

---

## 3. الـ Tasks

### TSK-SHF-001 — Migrations + Entity

- `V19__create_cash_shifts.sql` + Partial Unique + RLS.
- `V20__add_payments_shift_fk.sql`.
- Trigger `app.guard_shift_update()`:
  - ممنوع `DELETE`.
  - ممنوع تعديل وردية مقفولة.

### TSK-SHF-002 — Service

- `openShift` — Branch + One-Open Check.
- `closeShift` — Expected + Variance.
- `forceClose` — Manager/Owner.
- `getCurrent` — Blind Count.
- `list` — Branch Scope.
- Audit.

### TSK-SHF-003 — Payment Integration

- `PaymentService` Cash Validation.
- `ShiftQueryService.getOpenShift(userId)`.
- `payment.shift_id` بيتسجل.

### TSK-SHF-004 — API + Reporting

- 6 Endpoints.
- `GET /dashboard/shifts` (P1) — Variance Summary.
- Branch Scope.

### TSK-SHF-005 — Tests

| # | الاختبار | المتوقع |
|---|---|---|
| TS-SHF-01 | فتح وردية | `OPEN` |
| TS-SHF-02 | فتح تانية لنفس User | `409` |
| TS-SHF-03 | فتح في فرع تاني | `403` |
| TS-SHF-04 | إغلاق بعد 12 دفعة كاش | `expected` صحيح |
| TS-SHF-05 | InstaPay | مستبعد |
| TS-SHF-06 | Variance سالب | Shortage |
| TS-SHF-07 | Variance موجب | Overage |
| TS-SHF-08 | Blind Count | المتوقع مخفي |
| TS-SHF-09 | تعديل وردية مقفولة | DB Exception |
| TS-SHF-10 | Force Close بدون عدّ | Variance `null` |
| TS-SHF-11 | Cash Payment بدون وردية | `422` |
| TS-SHF-12 | Manager فرع B | `404` |
| TS-SHF-13 | 10 فتح متزامن | واحد ينجح |
| TS-SHF-14 | Audit | `SHIFT_OPENED/CLOSED/FORCE_CLOSED` |

---

## 4. معايير القبول

- [ ] `cash_shifts` + Partial Unique.
- [ ] `payments.shift_id` FK.
- [ ] فتح/إغلاق/Force Close.
- [ ] Blind Count.
- [ ] Variance صحيح.
- [ ] Cash Payment مربوط.
- [ ] Immutability.
- [ ] Audit.
- [ ] Branch Scope.
- [ ] Tests.

---

## 5. المخاطر

| المخاطرة | التخفيف |
|---|---|
| فتح ورديتين | Partial Unique + Test |
| تعديل بعد الإغلاق | Trigger |
| Variance غلط | Query + Test |
| ربط دفعة بوردية غلط | `shift_id` من الـ Context |
| Force Close يخفي عجز | Reason + Audit |
| Cash Refund مش موجود | P1 — يضاف مع Refund |

---

## 6. مرتبط بـ

- [`README.md`](./README.md)
- [payment/domain-design](../payment/domain-design.md)
- [database-strategy](../../architecture/database-strategy.md)
