# Notification — خطة التنفيذ

> **الـ Phase:** 3 · **المدة:** 4–5 أيام · **الترتيب:** بعد `risk`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-NOT-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-001, TSK-MEM-001 |
| **المخرجات** | `V13__create_message_queue.sql`, `MessageQueueItem` |
| **معيار القبول** | RLS + Branch + Status Guard + Indexes |

**خطوات:**
1. `message_queue` بكل الأعمدة.
2. `branch_id UUID NOT NULL` — SC-13.
3. `opened_at TIMESTAMPTZ` — SC-24.
4. `wa_me_link TEXT` — SC-27.
5. `is_transactional BOOLEAN NOT NULL DEFAULT false` — SC-29.
6. `CHECK (template IN (...))` — يشمل `GRACE_EXPOSURE` + `PORTAL_MAGIC_LINK`.
7. `CHECK (status IN (...))`.
8. `app.guard_message_update()` Trigger:
   - ممنوع `DELETE`.
   - ممنوع تعديل `rendered_text` / `member_id` / `template` / `due_date` / `is_transactional`.
   - Status Transitions المسموحة فقط.
9. `SELECT app.enable_tenant_rls('message_queue')`.
10. `REVOKE DELETE`.
11. Indexes:
    - `(org_id, status, due_date) WHERE status = 'PENDING'`
    - `(org_id, member_id, template, sent_at DESC)`
    - `(org_id, branch_id, status)`
    - `(org_id, attributed_subscription_id) WHERE attributed_subscription_id IS NOT NULL`

---

### TSK-NOT-002 — Templates + Renderer

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-001 |
| **المخرجات** | `MessageTemplateRenderer`, 3 قوالب شغالة + 3 معرّفين (منهم `PORTAL_MAGIC_LINK` Transactional) |
| **معيار القبول** | كل متغير موجود + Render صحيح |

**خطوات:**
1. `RENEWAL_REMINDER`.
2. `GRACE_EXPOSURE`.
3. `MISS_YOU`.
4. `WELCOME` / `COLLECTION` — معرّفين بدون توليد.
5. `PORTAL_MAGIC_LINK` — P1 Transactional (بدون Cooldown/Window).
6. `Map<String,String>` Variables.
7. رفض المتغير الناقص.
8. `WaMeLinkBuilder` + `PhoneUtil.toE164Egypt`.
9. Unit Tests للنص + الـ Link.

---

### TSK-NOT-003 — Queue Generation Job

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-002, TSK-RSK-005 |
| **المخرجات** | `MessageQueueJob` + ShedLock |
| **معيار القبول** | 6:00 AM + Trigger Priority + Cooldown |

**خطوات:**
1. `@Scheduled(cron = "0 0 6 * * *", zone = "Africa/Cairo")`.
2. `@SchedulerLock(name = "messageQueueJob", lockAtMostFor = "15m")`.
3. `TenantTransactionRunner`.
4. `listMembersWithAccess`.
5. Opt-in + Cooldown.
6. Trigger Priority:
   - `GRACE_EXPOSURE`
   - `RENEWAL_REMINDER` (T-7 / T-3 / T-1)
   - `MISS_YOU`
7. Render + Link + Save `PENDING`.
8. Idempotency Test.

---

### TSK-NOT-004 — Lifecycle Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-003 |
| **المخرجات** | `NotificationService` |
| **معيار القبول** | Open / Sent / Cancel + Time Window + Opt-in |

**خطوات:**
1. `markOpened(id)` — `PENDING → OPENED`.
2. `markSent(id)` — `OPENED → SENT_CONFIRMED`.
3. `cancel(id)` — Manager/Owner.
4. Time Window Check 9–21.
5. Opt-in Check.
6. `sentBy` + `sentAt`.
7. Audit `MESSAGE_SENT` / `MESSAGE_CANCELLED`.
8. `MessageSentEvent` بعد الـ Commit.

---

### TSK-NOT-005 — Attribution

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-004, TSK-SUB-003 |
| **المخرجات** | `MessageAttributionListener` |
| **معيار القبول** | 7 أيام + Latest Sent + Idempotent |

**خطوات:**
1. `@TransactionalEventListener(AFTER_COMMIT)`.
2. `SubscriptionRenewedEvent`.
3. Query `SENT_CONFIRMED` خلال 7 أيام.
4. `attributed_subscription_id` لو `NULL`.
5. Test: رسالة قديمة، رسالة PENDING، رسالة مكررة.

---

### TSK-NOT-006 — Query Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-004 |
| **المخرجات** | `NotificationQueryService` |
| **معيار القبول** | Queue + Details + Attribution Stats |

**خطوات:**
1. `findLastCreatedAt`.
2. `listQueue` — Default اليوم + PENDING.
3. `getMessage`.
4. `getAttributionStats`.
5. Branch Scope.
6. Enrich بـ Member + SentBy.

---

### TSK-NOT-007 — API + Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-NOT-005, TSK-NOT-006 |
| **المخرجات** | `NotificationController` + 6 Endpoints + Tests |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

**خطوات:**
1. `GET /notifications/queue`.
2. `GET /notifications/{id}`.
3. `PATCH /notifications/{id}/opened`.
4. `PATCH /notifications/{id}/sent`.
5. `PATCH /notifications/{id}/cancel`.
6. `GET /notifications/attribution`.
7. Tests.

---

## 2. الترتيب

```text
TSK-NOT-001 Migration + Entity
      ↓
TSK-NOT-002 Templates + Renderer
      ↓
TSK-NOT-003 Queue Generation Job
      ↓
TSK-NOT-004 Lifecycle Service
      ↓
TSK-NOT-005 Attribution
      ↓
TSK-NOT-006 Query Service
      ↓
TSK-NOT-007 API + Tests
```

---

## 3. معايير القبول النهائية

- [ ] `message_queue` + RLS + Branch.
- [ ] `opened_at` + `wa_me_link TEXT`.
- [ ] `is_transactional` + `PORTAL_MAGIC_LINK` (P1).
- [ ] 3 قوالب شغالة + 3 معرّفين (`WELCOME`, `COLLECTION`, `PORTAL_MAGIC_LINK`).
- [ ] Job 6:00 AM + ShedLock.
- [ ] Trigger Priority صحيحة.
- [ ] Cooldown 3 أيام.
- [ ] Time Window 9–21.
- [ ] Opt-in.
- [ ] Lifecycle + Audit.
- [ ] `wa.me` Link صحيح.
- [ ] Attribution 7 أيام.
- [ ] Queue + Attribution API.
- [ ] Tests خضراء.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| Spam للعضو | Cooldown + Opt-in |
| إرسال قبل 9 | Time Window + Test |
| فتح ≠ إرسال | `OPENED` منفصل |
| Link طويل | `TEXT` (SC-27) |
| Job يكرر رسائل | Cooldown + ShedLock |
| Attribution غلط | 7 أيام + Latest + `NULL` Check |
| Rendering ناقص | Exception + Test |
| Manager يلغى رسالة مهمة | Audit + Reason |
| Cloud API لاحقاً | `NotificationChannel` Interface |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
