# Subscription — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| State Machine | كل Transition صالحة/ممنوعة |
| Renewal | قبل/بعد الانتهاء + Grace |
| Freeze | تمديد دقيق + حد أقصى |
| Grace | انتقال صحيح |
| Scheduler | Tenant Scope + ShedLock |
| Isolation | صفر تسريب |

---

## 2. State Transition Matrix

| # | From | Action | To | Expected |
|---|---|---|---|---|
| ST-SUB-01 | — | Create | `PENDING_PAYMENT` | ✅ + `CREATED` |
| ST-SUB-02 | `PENDING_PAYMENT` | Full Payment | `ACTIVE` | ✅ + `ACTIVATED` |
| ST-SUB-03 | `PENDING_PAYMENT` | Partial Payment | `PENDING_PAYMENT` | ✅ مفيش Event حالة |
| ST-SUB-04 | `PENDING_PAYMENT` | Cancel | `CANCELLED` | ✅ + `CANCELLED` |
| ST-SUB-05 | `PENDING_PAYMENT` | Freeze | — | ❌ `422` |
| ST-SUB-06 | `ACTIVE` | Freeze | `FROZEN` | ✅ + `FROZEN` |
| ST-SUB-07 | `FROZEN` | Unfreeze | `ACTIVE` | ✅ + `UNFROZEN` |
| ST-SUB-08 | `FROZEN` | Freeze | — | ❌ `422` |
| ST-SUB-09 | `ACTIVE` | End Date + Grace | `GRACE` | ✅ + `GRACE_ENTERED` |
| ST-SUB-10 | `ACTIVE` | End Date بدون Grace | `EXPIRED` | ✅ + `EXPIRED` |
| ST-SUB-11 | `GRACE` | Grace End | `EXPIRED` | ✅ + `EXPIRED` |
| ST-SUB-12 | `GRACE` | Freeze | — | ❌ `422` |
| ST-SUB-13 | `ACTIVE` | Cancel | `CANCELLED` | ✅ |
| ST-SUB-14 | `FROZEN` | Cancel | `CANCELLED` | ✅ |
| ST-SUB-15 | `EXPIRED` | Activate | — | ❌ `422` |
| ST-SUB-16 | `CANCELLED` | Activate | — | ❌ `422` |
| ST-SUB-17 | `ACTIVE` | Sessions = 0 | `EXPIRED` | ✅ |

---

## 3. Renewal Tests

| # | السيناريو | المتوقع |
|---|---|---|
| RN-SUB-01 | تجديد قبل الانتهاء 5 أيام | New `start = old.endDate`, `end = start + 30` |
| RN-SUB-02 | تجديد يوم الانتهاء | New `start = old.endDate` |
| RN-SUB-03 | تجديد بعد الانتهاء | New `start = today` |
| RN-SUB-04 | تجديد في Grace | Old `EXPIRED` + New `today` |
| RN-SUB-05 | تجديد `EXPIRED` | New `today` |
| RN-SUB-06 | تجديد `FROZEN` | `422` |
| RN-SUB-07 | تجديد `PENDING_PAYMENT` | `422` |
| RN-SUB-08 | تغيير الباقة عند التجديد | Snapshot جديد |
| RN-SUB-09 | Override Start Date | Manager فقط + Reason + Audit |
| RN-SUB-10 | Receptionist Override | `403` |
| RN-SUB-11 | Old Subscription | لسه `ACTIVE` (قبل الانتهاء) |
| RN-SUB-12 | Price Snapshot | ثابت للفترة القديمة |

```java
@Test
void renewBeforeExpiry_startsFromCurrentEndDate() {
    Subscription old = fixtures.activeSubscription(endDate(2026, 12, 10));
    Plan plan = fixtures.plan(durationDays(30));

    SubscriptionResponse renewed = subscriptionService.renew(
        old.getId(), new RenewSubscriptionRequest(plan.getId(), null, null));

    assertThat(renewed.startDate()).isEqualTo(LocalDate.of(2026, 12, 10));
    assertThat(renewed.endDate()).isEqualTo(LocalDate.of(2027, 1, 9));
}
```

---

## 4. Freeze Tests

| # | السيناريو | المتوقع |
|---|---|---|
| FR-SUB-01 | تجميد 7 أيام | `FROZEN` + `scheduled = today+7` |
| FR-SUB-02 | تجاوز `freezeMaxDays` | `422 FREEZE_LIMIT_EXCEEDED` |
| FR-SUB-03 | تجميد بدون Reason | `400` |
| FR-SUB-04 | فك بعد 3 أيام من 7 | `endDate += 3` + `frozenDaysUsed = 3` |
| FR-SUB-05 | فك في نفس اليوم | `endDate += 0` |
| FR-SUB-06 | فك بعد 10 أيام من 7 | `actualDays = 7` (Cap) |
| FR-SUB-07 | Auto-Unfreeze | يشتغل في `scheduled` |
| FR-SUB-08 | تجميد مكرر | `422` |
| FR-SUB-09 | Check-in على `FROZEN` | `422 SUBSCRIPTION_FROZEN` |
| FR-SUB-10 | `frozenDaysUsed` بعد فك | ≤ `freezeMaxDays` |

---

## 5. Grace / Expiration Tests

| # | السيناريو | المتوقع |
|---|---|---|
| GR-SUB-01 | `graceDays = 5` + `endDate` فات | `GRACE` + `graceEndsAt = end + 5` |
| GR-SUB-02 | `graceDays = 0` + `endDate` فات | `EXPIRED` |
| GR-SUB-03 | `today > graceEndsAt` | `EXPIRED` |
| GR-SUB-04 | Check-in في `GRACE` | مسموح + Warning |
| GR-SUB-05 | `EXPIRING_SOON` قبل 7 أيام | `true` |
| GR-SUB-06 | `EXPIRING_SOON` قبل 8 أيام | `false` |
| GR-SUB-07 | Scheduler على Org‑ين | كل واحدة بياناتها |

---

## 6. Session Tests

| # | السيناريو | المتوقع |
|---|---|---|
| SS-SUB-01 | `sessionsRemaining = 12` → consume | 11 |
| SS-SUB-02 | آخر جلسة | 0 + `EXPIRED` |
| SS-SUB-03 | `sessionsRemaining = 0` → consume | `422 NO_SESSIONS_REMAINING` |
| SS-SUB-04 | 5 Threads على 3 جلسات | 3 نجاح + 2 رفض |
| SS-SUB-05 | Check-in على `EXPIRED` | `422` |

---

## 7. Concurrency Tests

| # | الاختبار | المتوقع |
|---|---|---|
| CT-SUB-01 | تجميد متزامن | واحد ينجح |
| CT-SUB-02 | تجديد متزامن | سجلين مختلفين أو واحد حسب الـ Lock |
| CT-SUB-03 | تفعيل متزامن من Payment | تفعيل واحد |
| CT-SUB-04 | Session Deduction | لا Negative |

---

## 8. Scheduler Tests

| # | الاختبار | المتوقع |
|---|---|---|
| SC-SUB-01 | ShedLock Instance‑ين | Job يشتغل مرة |
| SC-SUB-02 | Tenant Scope | كل Org في Transaction |
| SC-SUB-03 | فشل Org واحدة | الباقي يكمل |
| SC-SUB-04 | `TenantContext.clear` | مفيش Leakage |
| SC-SUB-05 | Auto-Unfreeze | يشتغل قبل Expiration |

---

## 9. Isolation Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-SUB-01 | `GET /subscriptions` كـ Org A | اشتراكات A فقط |
| IT-SUB-02 | `GET /subscriptions/{B}` | `404` |
| IT-SUB-03 | Freeze اشتراك B | `404` |
| IT-SUB-04 | `memberId` من Org B | `404` |
| IT-SUB-05 | Raw JDBC بدون Context | صفر صفوف |

---

## 10. Audit / Events Tests

| # | العملية | Event | Audit |
|---|---|---|---|
| AE-SUB-01 | Create | `CREATED` | `SUBSCRIPTION_CREATED` |
| AE-SUB-02 | Activate | `ACTIVATED` | `SUBSCRIPTION_ACTIVATED` |
| AE-SUB-03 | Renew | `RENEWED` | `SUBSCRIPTION_RENEWED` |
| AE-SUB-04 | Freeze | `FROZEN` | `SUBSCRIPTION_FROZEN` |
| AE-SUB-05 | Unfreeze | `UNFROZEN` | `SUBSCRIPTION_UNFROZEN` |
| AE-SUB-06 | Cancel | `CANCELLED` | `SUBSCRIPTION_CANCELLED` |
| AE-SUB-07 | Expire | `EXPIRED` | `SUBSCRIPTION_EXPIRED` |
| AE-SUB-08 | Immutability | — | `UPDATE`/`DELETE` مرفوض |

---

## 11. Performance

| المسار | الهدف |
|---|---|
| `GET /subscriptions/{id}` | P95 < 150ms |
| `POST /subscriptions` | P95 < 300ms |
| `POST /freeze` | P95 < 300ms |
| Expiration Job (5000 اشتراك) | < 2 دقيقة |

---

## 12. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-025 | RN-SUB-12, SN-PLN-01 |
| BR-026 | RN-SUB-01 |
| BR-027 | RN-SUB-03 |
| BR-028 | AE-SUB-01..08 |
| C-04 | GR-SUB-05/06 |
| C-05 | FR-SUB-04/05 |
| C-11/C-20 | RN-SUB-04/05 |

---

## 13. Definition of Done

- [ ] State Matrix كاملة.
- [ ] Renewal + Freeze + Grace + Session Tests.
- [ ] Concurrency + Scheduler Tests.
- [ ] Isolation + Audit Tests.
- [ ] مفيش `status` بيتعدل خارج State Machine.
- [ ] كل Transition له Event.
