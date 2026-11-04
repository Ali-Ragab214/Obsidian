# Plan — خطة التنفيذ

> **الـ Phase:** 2 · **المدة:** 2–3 أيام · **الترتيب:** قبل `subscription`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-PLN-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-001, TSK-AUD-001 |
| **المخرجات** | `V7__create_plans.sql`, `Plan.java`, `PlanType.java` |
| **معيار القبول** | RLS + CHECKs + Unique `(org_id, lower(name))` |

**خطوات:**
1. الأعمدة الأساسية.
2. `CHECK (duration_days > 0)`.
3. `CHECK (price_minor > 0)`.
4. `CHECK (session_count IS NULL OR session_count > 0)`.
5. `CHECK (freeze_max_days >= 0 AND grace_days >= 0)`.
6. `CHECK (type IN (...))`.
7. `CREATE UNIQUE INDEX uk_plans_org_name ON plans(org_id, lower(name));`.
8. `SELECT app.enable_tenant_rls('plans');`.

---

### TSK-PLN-002 — Domain + Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PLN-001 |
| **المخرجات** | `PlanService`, `PlanServiceImpl`, `PlanMapper` |
| **معيار القبول** | Create/Update/Deactivate/Activate + Validation |

**خطوات:**
1. `create(CreatePlanRequest)`.
2. `update(UUID, UpdatePlanRequest)`.
3. `deactivate` / `activate`.
4. `get` / `list`.
5. Audit لكل تغيير.
6. `@PreAuthorize("hasRole('GYM_OWNER')")`.

---

### TSK-PLN-003 — `PlanQueryService`

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PLN-002 |
| **المخرجات** | `PlanQueryService`, `PlanPriceSnapshot` |
| **معيار القبول** | `subscription` تقدر تقرأ Snapshot |

**خطوات:**
1. `getPlan`, `getActivePlan`.
2. `getPriceSnapshot`.
3. `listActivePlans`.
4. ممنوع إرجاع Entity.

---

### TSK-PLN-004 — API

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PLN-002 |
| **المخرجات** | `PlanController` + 6 Endpoints |
| **معيار القبول** | Owner للكتابة + الكل للقراءة |

**خطوات:**
1. `POST /plans`.
2. `GET /plans?active=`.
3. `GET /plans/{id}`.
4. `PATCH /plans/{id}`.
5. `POST /plans/{id}/deactivate`.
6. `POST /plans/{id}/activate`.

---

### TSK-PLN-005 — Snapshot Independence Test

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PLN-003, `subscription` |
| **المخرجات** | Integration Test |
| **معيار القبول** | تغيير سعر الباقة لا يغير `price_minor_snapshot` |

---

### TSK-PLN-006 — Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PLN-005 |
| **المخرجات** | Unit + Repo + API + Isolation |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

---

## 2. الترتيب

```text
TSK-PLN-001 Migration + Entity
      ↓
TSK-PLN-002 Service
      ↓
TSK-PLN-003 Query Service
      ↓
TSK-PLN-004 API
      ↓
TSK-PLN-005 Snapshot Test
      ↓
TSK-PLN-006 Tests
```

---

## 3. معايير القبول النهائية

- [ ] `plans` Migration + RLS + Constraints.
- [ ] `price_minor > 0` مفروض.
- [ ] `session_count` مطلوب للباقات القائمة على الجلسات.
- [ ] `PT_PACKAGE` → `grants_access = false` افتراضياً.
- [ ] اسم الباقة فريد داخل الـ Org.
- [ ] ممنوع حذف باقة.
- [ ] تعطيل لا يؤثر على الاشتراكات القائمة.
- [ ] تغيير السعر لا يؤثر على الـ Snapshots.
- [ ] `PlanQueryService` جاهز للـ `subscription`.
- [ ] Audit + Isolation Tests.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| تعديل السعر يؤثر على العقود | Snapshot + Test |
| باقة معطلة تُستخدم | `getActivePlan` + Test |
| Session Count ناقص | Service Validation + DB CHECK |
| اسم مكرر | Unique Index |
| Time Restriction مفهوم غلط | توثيق Data only |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
