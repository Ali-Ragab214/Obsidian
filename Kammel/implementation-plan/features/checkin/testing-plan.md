# Check-in — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Validation Matrix | كل حالة اشتراك |
| Duplicate | Warning + Confirm |
| Sessions | خصم مرة واحدة |
| Override | Manager فقط + Audit |
| Isolation | صفر تسريب |
| Performance | P95 < 500ms |

---

## 2. Validation Matrix

| # | حالة الاشتراك | المتوقع |
|---|---|---|
| VA-CHK-01 | `ACTIVE` + `grants_access = true` | `200 SUCCESS` |
| VA-CHK-02 | `ACTIVE` + `endDate` خلال 7 أيام | `200 EXPIRING_SOON_ALERT` |
| VA-CHK-03 | `GRACE` | `200 GRACE_WARNING` |
| VA-CHK-04 | `FROZEN` | `422 SUBSCRIPTION_FROZEN` |
| VA-CHK-05 | `EXPIRED` | `422 SUBSCRIPTION_EXPIRED` |
| VA-CHK-06 | `PENDING_PAYMENT` | `422 NO_ACTIVE_SUBSCRIPTION` |
| VA-CHK-07 | `CANCELLED` | `422 NO_ACTIVE_SUBSCRIPTION` |
| VA-CHK-08 | مفيش اشتراك | `422 NO_ACTIVE_SUBSCRIPTION` |
| VA-CHK-09 | `ACTIVE` + `grants_access = false` | `422 NO_ACTIVE_SUBSCRIPTION` |
| VA-CHK-10 | اشتراكين `ACTIVE` | الأقرب انتهاءً |
| VA-CHK-11 | `FROZEN` + اشتراك `ACTIVE` تاني | `200 SUCCESS` |
| VA-CHK-12 | `GRACE` + `grants_access = false` | `422` |

```java
@Test
void frozenSubscription_blocksCheckInWithReason() {
    Subscription sub = fixtures.frozenSubscription(
        LocalDate.of(2026, 11, 10), "Traveling for work");

    assertThatThrownBy(() -> checkInService.checkIn(request, actor))
        .isInstanceOf(SubscriptionFrozenException.class)
        .hasMessageContaining("2026-11-10")
        .hasMessageContaining("Traveling for work");
}
```

---

## 3. Duplicate Tests

| # | السيناريو | المتوقع |
|---|---|---|
| DP-CHK-01 | أول Check-in | `SUCCESS` + Row |
| DP-CHK-02 | تاني Check-in بدون Confirm | `DUPLICATE_WARNING` + مفيش Row |
| DP-CHK-03 | تاني Check-in مع Confirm + Reason | `SUCCESS` + Row |
| DP-CHK-04 | Confirm بدون Reason | `400` |
| DP-CHK-05 | نفس اليوم فرع مختلف | Duplicate |
| DP-CHK-06 | اليوم التالي 00:01 Cairo | مش Duplicate |
| DP-CHK-07 | 23:59 Cairo vs 00:01 UTC | الحساب Cairo |
| DP-CHK-08 | 10 Threads لنفس العضو | Row واحد + 9 Warnings |
| DP-CHK-09 | Check-in على عضو محذوف | `404` |

---

## 4. Session Tests

| # | السيناريو | المتوقع |
|---|---|---|
| SS-CHK-01 | `sessionsRemaining = 12` | 11 |
| SS-CHK-02 | آخر جلسة | 0 + `EXPIRED` |
| SS-CHK-03 | `sessionsRemaining = 0` | `422 NO_SESSIONS_REMAINING` |
| SS-CHK-04 | 5 Threads على 3 جلسات | 3 نجاح + 2 رفض |
| SS-CHK-05 | Duplicate Confirm | خصم جلسة |
| SS-CHK-06 | Access Override | مفيش خصم |
| SS-CHK-07 | Rollback بعد الخصم | الجلسة ترجع |

---

## 5. Member Resolution Tests

| # | identifier | method | المتوقع |
|---|---|---|---|
| MR-CHK-01 | `01012345678` | `PHONE_SEARCH` | Success |
| MR-CHK-02 | `M-000123` | `MEMBER_CODE` | Success |
| MR-CHK-03 | UUID | `STATIC_QR` | Success |
| MR-CHK-04 | `Ahmed Tarek` | `NAME_SEARCH` | Success لو فريد |
| MR-CHK-05 | `Ahmed` | `NAME_SEARCH` | `409 AMBIGUOUS_MEMBER` |
| MR-CHK-06 | Phone + `STATIC_QR` | — | `400 INVALID_IDENTIFIER_METHOD` |
| MR-CHK-07 | Member من Org تانية | — | `404` |
| MR-CHK-08 | Member محذوف | — | `404` |
| MR-CHK-09 | Member غير نشط | — | `404` |

---

## 6. Override Tests

| # | السيناريو | المتوقع |
|---|---|---|
| OV-CHK-01 | Manager + `EXPIRED` + Reason | Row + Audit |
| OV-CHK-02 | Owner + `FROZEN` + Reason | Row + Audit |
| OV-CHK-03 | Manager + `NO_ACTIVE` + Reason | Row + `subscription_id = null` |
| OV-CHK-04 | Receptionist Override | `403` |
| OV-CHK-05 | Override بدون Reason | `400` |
| OV-CHK-06 | Override على `ACTIVE` | تجاهل / Success عادي |
| OV-CHK-07 | Audit Action | `CHECKIN_OVERRIDDEN` |
| OV-CHK-08 | Audit Metadata | النوع + Subscription ID |

---

## 7. Branch Tests

| # | السيناريو | المتوقع |
|---|---|---|
| BR-CHK-01 | Owner يحدد فرع | Success |
| BR-CHK-02 | Owner بدون فرع | `400` |
| BR-CHK-03 | Manager فرع تاني | `403` |
| BR-CHK-04 | Receptionist فرع تاني | `403` |
| BR-CHK-05 | فرع غير نشط | `422` |
| BR-CHK-06 | Manager `GET /today` | فرعه فقط |
| BR-CHK-07 | Owner `GET /today` | كل الفروع |

---

## 8. Immutability Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IM-CHK-01 | `UPDATE check_ins` | DB Exception |
| IM-CHK-02 | `DELETE check_ins` | DB Exception |
| IM-CHK-03 | Service يحاول التعديل | مفيش Setter |
| IM-CHK-04 | Override Reason > 500 | DB/DTO يرفض |

---

## 9. Isolation Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-CHK-01 | `POST /checkins` لعضو Org B | `404` |
| IT-CHK-02 | `GET /checkins` كـ Org A | A فقط |
| IT-CHK-03 | `GET /checkins/{B}` | `404` |
| IT-CHK-04 | Raw JDBC بدون Context | صفر صفوف |
| IT-CHK-05 | نفس Phone في Org‑ين | كل واحدة تشوف عضوها |

---

## 10. Audit Tests

| # | العملية | Audit |
|---|---|---|
| AU-CHK-01 | Check-in عادي | `CHECKIN_RECORDED` |
| AU-CHK-02 | Duplicate Confirm | `CHECKIN_OVERRIDDEN` |
| AU-CHK-03 | Access Override | `CHECKIN_OVERRIDDEN` |
| AU-CHK-04 | Blocked Check-in | مفيش Audit (مفيش تغيير) |
| AU-CHK-05 | Actor | المستخدم الحالي |

---

## 11. Performance

| المسار | الهدف |
|---|---|
| `POST /checkins` (Phone) | P95 < 500ms |
| `POST /checkins` (QR) | P95 < 300ms |
| `POST /checkins` (Name) | P95 < 800ms |
| `GET /checkins/today` | P95 < 300ms |
| 50 Check-in متزامن | < 5s |
| 5000 عضو | Index Scan — مفيش Full Scan |

---

## 12. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-014 | VA-CHK-04/05 + الرسائل |
| BR-015 | DP-CHK-01 + الحقول |
| BR-016 | OV-CHK-01..08 |
| BR-017 | SS-CHK-01 + Risk Integration |
| BR-018 | DP-CHK-02/03 |
| CHK-05 | SS-CHK-04 |
| CHK-07 | DP-CHK-05/06/07 |
| CHK-08 | OV-CHK-04 |
| C-17 | Trainer `403` |

---

## 13. Definition of Done

- [ ] Validation Matrix كاملة.
- [ ] Duplicate + Confirm.
- [ ] Session Deduction تحت التزامن.
- [ ] Override + Audit.
- [ ] Branch + Isolation.
- [ ] Immutability.
- [ ] P95 < 500ms.
- [ ] مفيش Check-in بدون اشتراك إلا Override موثق.
