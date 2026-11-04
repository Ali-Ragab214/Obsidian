# Plan — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md)

---

## 1. Unit Tests

| # | الاختبار | المتوقع |
|---|---|---|
| UT-PLN-01 | `durationDays = 0` | رفض |
| UT-PLN-02 | `priceMinor = 0` | رفض |
| UT-PLN-03 | `priceMinor < 0` | رفض |
| UT-PLN-04 | `SESSION_PACK` بدون `sessionCount` | `422` |
| UT-PLN-05 | `PT_PACKAGE` بدون `sessionCount` | `422` |
| UT-PLN-06 | `MONTHLY` مع `sessionCount` | `422` |
| UT-PLN-07 | `PT_PACKAGE` بدون `grantsAccess` | `false` |
| UT-PLN-08 | `MONTHLY` بدون `grantsAccess` | `true` |
| UT-PLN-09 | `freezeMaxDays = 0` | مسموح |
| UT-PLN-10 | `graceDays = 0` | مسموح |
| UT-PLN-11 | `deactivate()` | `active = false` |
| UT-PLN-12 | `isSessionBased()` | true/false صح |

---

## 2. Repository Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RT-PLN-01 | `existsByOrgIdAndNameIgnoreCase` | true/false |
| RT-PLN-02 | نفس الاسم في Org تانية | مسموح |
| RT-PLN-03 | `findAllByOrgIdAndActiveTrue` | Active فقط |
| RT-PLN-04 | `findByIdAndOrgId` | Optional |
| RT-PLN-05 | CHECK `price_minor > 0` | Exception |
| RT-PLN-06 | CHECK `duration_days > 0` | Exception |

---

## 3. API Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-PLN-01 | Owner ينشئ باقة | `201` + Audit |
| AT-PLN-02 | Manager ينشئ | `403` |
| AT-PLN-03 | Receptionist يقرأ | `200` |
| AT-PLN-04 | Trainer يقرأ | `403` |
| AT-PLN-05 | اسم مكرر | `409` |
| AT-PLN-06 | سعر 0 | `400` |
| AT-PLN-07 | تعديل السعر | `200` |
| AT-PLN-08 | تعطيل | `200` + `active=false` |
| AT-PLN-09 | إنشاء اشتراك على معطلة | `422` |
| AT-PLN-10 | `DELETE /plans/{id}` | `405` |
| AT-PLN-11 | باقة Org تانية | `404` |

---

## 4. Snapshot Independence Test

```java
@Test
void changingPlanPrice_doesNotAffectExistingSubscription() {
    Plan plan = fixtures.plan(orgId, priceMinor(180000));
    Subscription sub = fixtures.activeSubscription(orgId, memberId, plan);
    assertThat(sub.getPriceMinorSnapshot()).isEqualTo(180000);

    planService.update(plan.getId(), new UpdatePlanRequest(
        null, null, null, null, 250000L, null, null, null, null));

    Subscription reloaded = subscriptionRepository.findById(sub.getId()).orElseThrow();
    assertThat(reloaded.getPriceMinorSnapshot()).isEqualTo(180000);
}
```

| # | الاختبار | المتوقع |
|---|---|---|
| SN-PLN-01 | تغيير السعر | Snapshot ثابت |
| SN-PLN-02 | تعطيل الباقة | الاشتراك شغال |
| SN-PLN-03 | تغيير المدة | الاشتراك القديم ثابت |
| SN-PLN-04 | اشتراك جديد | السعر الجديد |

---

## 5. Isolation Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-PLN-01 | `GET /plans` كـ Org A | باقات A فقط |
| IT-PLN-02 | `GET /plans/{B}` | `404` |
| IT-PLN-03 | `PATCH /plans/{B}` | `404` |
| IT-PLN-04 | Raw JDBC بدون Context | صفر صفوف |
| IT-PLN-05 | نفس الاسم في Org‑ين | مسموح |

---

## 6. Security Tests

| # | الاختبار | المتوقع |
|---|---|---|
| SC-PLN-01 | Body فيه `orgId` | متجاهل |
| SC-PLN-02 | Receptionist `POST /plans` | `403` |
| SC-PLN-03 | Trainer `GET /plans` | `403` |

---

## 7. Performance

| المسار | الهدف |
|---|---|
| `GET /plans` (50 باقة) | < 150ms |
| `POST /plans` | < 200ms |
| `GET /plans/{id}` | < 100ms |

---

## 8. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-002 | UT-PLN-01..03, AT-PLN-06 |
| BR-025 | SN-PLN-01 |

---

## 9. Definition of Done

- [ ] Unit + Repo + API Tests.
- [ ] Snapshot Independence Test.
- [ ] Isolation + Security Tests.
- [ ] كل Constraints متغطية.
