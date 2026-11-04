# Plan — تصميم الـ API

> **Base Path:** `/api/v1/plans`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/plans` | `GYM_OWNER` | إنشاء باقة |
| GET | `/plans` | Owner, Manager, Receptionist | قائمة |
| GET | `/plans/{id}` | Owner, Manager, Receptionist | تفاصيل |
| PATCH | `/plans/{id}` | `GYM_OWNER` | تعديل |
| POST | `/plans/{id}/deactivate` | `GYM_OWNER` | تعطيل |
| POST | `/plans/{id}/activate` | `GYM_OWNER` | تفعيل |

---

## 2. `POST /plans`

**Request:**
```json
{
  "name": "Gold 3-Month Membership",
  "type": "QUARTERLY",
  "durationDays": 90,
  "sessionCount": null,
  "priceMinor": 180000,
  "freezeMaxDays": 14,
  "graceDays": 5,
  "timeRestriction": null,
  "grantsAccess": true
}
```

**Validation:**
- `name`: `@NotBlank`, `@Size(2,100)`.
- `type`: `@NotNull`.
- `durationDays`: `@Min(1)`.
- `priceMinor`: `@Min(1)`.
- `sessionCount`: `@Min(1)` لو موجود.
- `freezeMaxDays` / `graceDays`: `@Min(0)`.

**Response `201 Created`:**
```json
{
  "success": true,
  "message": "Plan created",
  "data": {
    "id": "33333333-4444-5555-6666-777777777777",
    "name": "Gold 3-Month Membership",
    "type": "QUARTERLY",
    "durationDays": 90,
    "sessionCount": null,
    "priceMinor": 180000,
    "priceFormatted": "1,800.00 EGP",
    "freezeMaxDays": 14,
    "graceDays": 5,
    "timeRestriction": null,
    "grantsAccess": true,
    "active": true,
    "createdAt": "2026-11-03T10:00:00Z"
  }
}
```

**Errors:** `409 DUPLICATE_RESOURCE`, `422 BUSINESS_RULE_VIOLATION`.

---

## 3. `GET /plans`

**Query:** `active` (default `true`), `type`, `search`, `page`, `size`, `sort`.

**Response:** `PageResponse<PlanResponse>`.

**Rules:**
- `GYM_OWNER` / Manager / Receptionist → كل باقات الـ Org.
- Trainer → `403`.

---

## 4. `GET /plans/{id}`

**Response `200 OK`:** `PlanResponse`.

**Errors:** `404 RESOURCE_NOT_FOUND` — لو Org تانية.

---

## 5. `PATCH /plans/{id}`

**Request (اختياري):**
```json
{
  "name": "Gold 3-Month Membership (2027)",
  "durationDays": 90,
  "priceMinor": 190000,
  "freezeMaxDays": 14,
  "graceDays": 5,
  "grantsAccess": true
}
```

**Rules:**
- `type` / `sessionCount` بيتحققوا مع بعض.
- تعديل السعر لا يغير الـ Snapshots.
- Audit `PLAN_UPDATED` مع قبل/بعد.

**Errors:** `409`, `404`, `422`.

---

## 6. `POST /plans/{id}/deactivate`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Plan deactivated",
  "data": { "id": "33333333-...", "active": false }
}
```

**Rules:**
- الاشتراكات القائمة تكمل.
- ممنوع إنشاء اشتراك جديد على الباقة.
- Audit `PLAN_DEACTIVATED`.

---

## 7. `POST /plans/{id}/activate`

**Response `200 OK`:** `active = true`.

- Audit `PLAN_UPDATED` أو `PLAN_ACTIVATED`.

---

## 8. الـ DTOs

```java
public record CreatePlanRequest(
    @NotBlank @Size(min = 2, max = 100) String name,
    @NotNull PlanType type,
    @Min(1) int durationDays,
    @Min(1) Integer sessionCount,
    @Min(1) long priceMinor,
    @Min(0) int freezeMaxDays,
    @Min(0) int graceDays,
    @Size(max = 20) String timeRestriction,
    Boolean grantsAccess
) {}

public record UpdatePlanRequest(
    @Size(min = 2, max = 100) String name,
    PlanType type,
    @Min(1) Integer durationDays,
    @Min(1) Integer sessionCount,
    @Min(1) Long priceMinor,
    @Min(0) Integer freezeMaxDays,
    @Min(0) Integer graceDays,
    @Size(max = 20) String timeRestriction,
    Boolean grantsAccess
) {}

public record PlanResponse(
    UUID id,
    String name,
    PlanType type,
    int durationDays,
    Integer sessionCount,
    long priceMinor,
    String priceFormatted,
    int freezeMaxDays,
    int graceDays,
    String timeRestriction,
    boolean grantsAccess,
    boolean active,
    Instant createdAt
) {}
```

---

## 9. قواعد الحقول

### `sessionCount`
- مطلوب لـ `SESSION_PACK` و`PT_PACKAGE`.
- `null` للأنواع التانية.
- لو اتبعت لنوع غير مناسب → `422`.

### `grantsAccess`
- لو `null` في الإنشاء:
  - `PT_PACKAGE` → `false`.
  - غير كده → `true`.
- في التعديل: `null` = سيبها زي ما هي.

### `timeRestriction`
- `MORNING` / `EVENING` / `LADIES` أو `null`.
- Data only — مفيش Enforcement في الـ MVP.

---

## 10. الصلاحيات

```java
@PreAuthorize("hasRole('GYM_OWNER')")
PlanResponse create(CreatePlanRequest request);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PageResponse<PlanResponse> list(PlanFilter filter, Pageable pageable);
```

---

## 11. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
