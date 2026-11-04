# Subscription — خطة التنفيذ

> **الـ Phase:** 2 · **المدة:** 6–7 أيام · **الترتيب:** قبل `payment`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-SUB-001 — Migrations + Entities

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-001, TSK-PLN-001 |
| **المخرجات** | `V8__create_subscriptions.sql`, `V9__create_subscription_events.sql` |
| **معيار القبول** | RLS + CHECKs + Immutability Trigger على Events |

**خطوات:**
1. `subscriptions` بكل الأعمدة + `org_id` + `renewal_of` (SC-25).
2. `CHECK (end_date >= start_date)`.
3. `CHECK (status IN (...))`.
4. `CHECK (frozen_days_used >= 0)`.
5. `CHECK (renewal_of IS NULL OR renewal_of <> id)`.
6. `subscription_events` + `org_id` + Trigger.
7. `SELECT app.enable_tenant_rls(...)` للاتنين.
8. Indexes `(org_id, status, end_date)` + `(org_id, member_id)` + Partial على `renewal_of`.

---

### TSK-SUB-002 — State Machine

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-001 |
| **المخرجات** | `SubscriptionStateMachine` + `InvalidStateTransitionException` |
| **معيار القبول** | كل Transition صالحة/ممنوعة متغطية Unit Test |

**خطوات:**
1. `create(member, plan, startDate)`.
2. `activate(sub, actor)`.
3. `freeze(sub, days, reason, actor)`.
4. `unfreeze(sub, actor, now)`.
5. `enterGrace(sub)`.
6. `expire(sub, reason)`.
7. `cancel(sub, reason, actor)`.
8. `consumeSession(sub)`.
9. كل Transition بتسجل Event.

---

### TSK-SUB-003 — Service + Query Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-002 |
| **المخرجات** | `SubscriptionService`, `SubscriptionQueryService` |
| **معيار القبول** | CRUD + Renew + Freeze + Cancel + Query |

**خطوات:**
1. `create` — Member + Plan Validation + Snapshot.
2. `renew` — قبل/بعد الانتهاء (C-20).
3. `freeze` / `unfreeze` / `cancel`.
4. `findAccessSubscription`.
5. `hasActiveMembership`.
6. `@PreAuthorize`.

---

### TSK-SUB-004 — Payment Listener

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-003 |
| **المخرجات** | `SubscriptionEventListener` |
| **معيار القبول** | `PaymentRecordedEvent` → تفعيل عند اكتمال المبلغ |

**خطوات:**
1. `@TransactionalEventListener` أو `@EventListener` (Synchronous).
2. `totalPaidMinor >= priceMinorSnapshot` → `activate`.
3. Idempotent — لو `ACTIVE` بالفعل → تجاهل.
4. Event `ACTIVATED`.

---

### TSK-SUB-005 — Expiration / Grace Job

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-003 |
| **المخرجات** | `SubscriptionExpirationJob` + ShedLock |
| **معيار القبول** | `ACTIVE` منتهي → `GRACE`/`EXPIRED` + `GRACE` منتهي → `EXPIRED` |

**خطوات:**
1. `@Scheduled(cron = "0 0 2 * * *", zone = "Africa/Cairo")`.
2. `@SchedulerLock(name = "subscriptionExpiration")`.
3. لكل Org → `TenantTransactionRunner`.
4. `processExpirationsForTenant()`.
5. `SubscriptionExpiredEvent` + `MemberAccessChangedEvent`.

---

### TSK-SUB-006 — Auto-Unfreeze Job

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-005 |
| **المخرجات** | `AutoUnfreezeJob` |
| **معيار القبول** | `scheduled_unfreeze_date <= today` → Unfreeze + Extend |

**خطوات:**
1. جوه نفس الـ Job أو Job منفصل.
2. `findByStatusAndScheduledUnfreezeDateLessThanEqual(FROZEN, today)`.
3. `unfreeze` مع `actualDays`.
4. Event `UNFROZEN`.

---

### TSK-SUB-007 — API

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-003 |
| **المخرجات** | `SubscriptionController` + 8 Endpoints |
| **معيار القبول** | CRUD + Renew/Freeze/Unfreeze/Cancel/Events |

**خطوات:**
1. `POST /subscriptions`.
2. `GET /subscriptions`.
3. `GET /subscriptions/{id}`.
4. `POST /{id}/renew`.
5. `POST /{id}/freeze`.
6. `POST /{id}/unfreeze`.
7. `POST /{id}/cancel`.
8. `GET /{id}/events`.

---

### TSK-SUB-008 — Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-007 |
| **المخرجات** | State Matrix + Renewal + Freeze + Jobs + Concurrency + Isolation |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

---

## 2. الترتيب

```text
TSK-SUB-001 Migrations + Entities
      ↓
TSK-SUB-002 State Machine
      ↓
TSK-SUB-003 Service + Query
      ↓
TSK-SUB-004 Payment Listener
      ↓
TSK-SUB-005 Expiration Job
      ↓
TSK-SUB-006 Auto-Unfreeze Job
      ↓
TSK-SUB-007 API
      ↓
TSK-SUB-008 Tests
```

---

## 3. معايير القبول النهائية

- [ ] Migrations + RLS + Immutability.
- [ ] State Machine كامل.
- [ ] `EXPIRED → ACTIVE` ممنوع.
- [ ] `CANCELLED → ACTIVE` ممنوع.
- [ ] Renewal بينشئ سجل جديد (C-20).
- [ ] Freeze يمدد عند Unfreeze.
- [ ] Grace بيشتغل.
- [ ] Scheduler بيشتغل في Tenant Scope.
- [ ] `PaymentRecordedEvent` بيفعّل.
- [ ] Session Deduction.
- [ ] `MemberAccessChangedEvent`.
- [ ] كل تغيير حالة له Event.
- [ ] Tests.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| تعديل `status` يدوي | No Setter + Arch Test |
| Renewal يكسر السعر | سجل جديد + Snapshot |
| تجميد مكرر | State Machine |
| Job يشتغل على Tenant غلط | `TenantTransactionRunner` |
| ShedLock مش مفعّل | Test على Instance‑ين |
| Grace يحسب غلط | Clock + Test |
| حذف اشتراك | Grants + No API |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
