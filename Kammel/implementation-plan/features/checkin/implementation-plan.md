# Check-in — خطة التنفيذ

> **الـ Phase:** 3 · **المدة:** 4–5 أيام · **الترتيب:** بعد `payment`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-CHK-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-001, TSK-MEM-001 |
| **المخرجات** | `V12__create_check_ins.sql`, `CheckIn` |
| **معيار القبول** | RLS + Immutability Trigger + CHECKs + Indexes |

**خطوات:**
1. `check_ins` بكل الأعمدة + `org_id`.
2. `CHECK (method IN ('NAME_SEARCH','PHONE_SEARCH','MEMBER_CODE','STATIC_QR','DYNAMIC_QR'))` — SC-26.
3. `CHECK (overridden_by IS NULL OR override_reason IS NOT NULL)`.
4. `app.forbid_mutation()` Trigger.
5. `SELECT app.enable_tenant_rls('check_ins')`.
6. `REVOKE UPDATE, DELETE` + `GRANT SELECT, INSERT`.
7. Indexes:
   - `(org_id, member_id, checked_in_at DESC)`
   - `(org_id, branch_id, checked_in_at DESC)`
   - `(org_id, checked_in_at DESC)`

---

### TSK-CHK-002 — Member Resolution

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-001, TSK-MEM-003 |
| **المخرجات** | `MemberResolver` |
| **معيار القبول** | Phone / Code / QR / Name + Ambiguity |

**خطوات:**
1. Trim + Normalize الـ identifier.
2. Detect الشكل (UUID / `M-\d{6}` / Phone / Name).
3. التحقق من تطابق الـ `method`.
4. `MemberQueryService` lookup.
5. رفض المحذوف/غير النشط.
6. `409` لو الاسم مكرر.

---

### TSK-CHK-003 — Subscription Validation

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-002, TSK-SUB-003 |
| **المخرجات** | `CheckInValidator` |
| **معيار القبول** | ACTIVE / GRACE / FROZEN / EXPIRED / PENDING / CANCELLED |

**خطوات:**
1. `findAccessSubscription(memberId)`.
2. `findBlockingSubscription(memberId)`.
3. بناء `CheckInDecision`:
   - `ALLOW` + warnings.
   - `WARN_DUPLICATE`.
   - `BLOCK` + reason + metadata.
4. `grants_access` check.
5. Grace + Expiring Soon warnings.
6. Branch validation.

---

### TSK-CHK-004 — Duplicate Guard + Override

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-003 |
| **المخرجات** | `DuplicateCheckService`, `OverridePolicy` |
| **معيار القبول** | Warning مش Block + Confirm + Audit |

**خطوات:**
1. `hasCheckedInToday(memberId)` على مستوى الـ Org.
2. Advisory Lock `checkin:{org}:{member}:{date}`.
3. لو Duplicate و`!confirmDuplicate` → Response Warning بدون Insert.
4. لو Confirm → تحقق من `overrideReason` + الصلاحية.
5. Access Override:
   - Manager/Owner فقط.
   - `overrideReason` إلزامي.
   - `subscriptionId` = الاشتراك المانع أو `NULL`.
6. Audit `CHECKIN_OVERRIDDEN`.

---

### TSK-CHK-005 — Check-in Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-004, TSK-SUB-003 |
| **المخرجات** | `CheckInService`, `CheckInServiceImpl` |
| **معيار القبول** | كل الـ Matrix + Session Deduction |

**خطوات:**
1. Resolve branch.
2. Resolve member.
3. Validate subscription.
4. Duplicate guard.
5. حفظ `CheckIn`.
6. `consumeSession` لو Session-based.
7. Audit `CHECKIN_RECORDED` / `CHECKIN_OVERRIDDEN`.
8. بناء الـ Response.
9. `@PreAuthorize`.

---

### TSK-CHK-006 — Query Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-005 |
| **المخرجات** | `CheckInQueryService` |
| **معيار القبول** | `getLastVisit`, `countVisits`, `hasCheckedInToday`, `list` |

**خطوات:**
1. `getLastVisit` — `MAX(checked_in_at)`.
2. `getFirstVisit` — `MIN(checked_in_at)`.
3. `countVisits(memberId, from, to)`.
4. `hasCheckedInToday` — Cairo range.
5. `list(filter, pageable)` — Branch Scope.
6. كل الـ Returns DTO.

---

### TSK-CHK-007 — API + Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-005, TSK-CHK-006 |
| **المخرجات** | `CheckInController` + 4 Endpoints + Tests |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

**خطوات:**
1. `POST /checkins`.
2. `GET /checkins`.
3. `GET /checkins/today`.
4. `GET /checkins/lookup`.
5. Performance Test < 500ms.

---

## 2. الترتيب

```text
TSK-CHK-001 Migration + Entity
      ↓
TSK-CHK-002 Member Resolution
      ↓
TSK-CHK-003 Subscription Validation
      ↓
TSK-CHK-004 Duplicate Guard + Override
      ↓
TSK-CHK-005 Check-in Service
      ↓
TSK-CHK-006 Query Service
      ↓
TSK-CHK-007 API + Tests
```

---

## 3. معايير القبول النهائية

- [ ] `check_ins` + RLS + Immutability.
- [ ] بحث بالهاتف / الكود / QR / الاسم.
- [ ] `ACTIVE` → Success.
- [ ] `GRACE` → Success + Warning.
- [ ] `FROZEN` → Block + رسالة واضحة.
- [ ] `EXPIRED` → Block + رسالة تجديد.
- [ ] Duplicate → Warning مش Block.
- [ ] Confirm → Override + Audit.
- [ ] Session Deduction مرة واحدة.
- [ ] `CheckInQueryService` جاهز لـ `risk`.
- [ ] P95 < 500ms.
- [ ] Tests خضراء.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| ازدحام 6–9 مساءً | Index + Query بسيطة + P95 Test |
| Double Click | Duplicate Warning + Advisory Lock |
| خصم جلسة مرتين | `SELECT FOR UPDATE` + Lock |
| عضو غلط بنفس الاسم | `409 AMBIGUOUS_MEMBER` |
| Override بدون رقابة | Manager/Owner + Reason + Audit |
| Check-in بعد منتصف الليل | `DateUtil` Cairo + Clock |
| تسريب بيانات عضو | RLS + Branch Scope |
| بطء البحث بالاسم | Trigram Index أو `ILIKE` مع حد أدنى 3 حروف |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
