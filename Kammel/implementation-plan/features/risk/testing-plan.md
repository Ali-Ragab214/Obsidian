# Risk — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Signals | كل Boundary |
| Score | 0..100 + Level صحيح |
| Reasons | مفهومة + مش فاضية |
| Draft | HIGH/CRITICAL فقط |
| Job | Idempotent + Tenant Scope |
| Isolation | صفر تسريب |

---

## 2. Signal Tests

### 2.1 Days Since Last Visit

| # | أيام | Points |
|---|---|---|
| SG-RSK-01 | 0 | 0 |
| SG-RSK-02 | 3 | 0 |
| SG-RSK-03 | 4 | 10 |
| SG-RSK-04 | 7 | 10 |
| SG-RSK-05 | 8 | 20 |
| SG-RSK-06 | 13 | 20 |
| SG-RSK-07 | 14 | 30 |
| SG-RSK-08 | 60 | 30 |

### 2.2 Attendance Drop

| # | Last 14 | Prior 14 | Drop | Points |
|---|---|---|---|---|
| SG-RSK-09 | 4 | 4 | 0% | 0 |
| SG-RSK-10 | 3 | 4 | 25% | 0 |
| SG-RSK-11 | 2 | 4 | 50% | 15 |
| SG-RSK-12 | 1 | 4 | 75% | 25 |
| SG-RSK-13 | 0 | 0 | — | 0 |
| SG-RSK-14 | 4 | 0 | — | 0 |
| SG-RSK-15 | 0 | 4 | 100% | 25 |

### 2.3 Days Remaining

| # | أيام | Points |
|---|---|---|
| SG-RSK-16 | -3 (Grace) | 20 |
| SG-RSK-17 | 0 | 20 |
| SG-RSK-18 | 3 | 20 |
| SG-RSK-19 | 4 | 15 |
| SG-RSK-20 | 7 | 15 |
| SG-RSK-21 | 8 | 5 |
| SG-RSK-22 | 14 | 5 |
| SG-RSK-23 | 15 | 0 |

### 2.4 Payment Friction / Freeze / Tenure

| # | السيناريو | Points |
|---|---|---|
| SG-RSK-24 | `delayedActivations = 0` | 0 |
| SG-RSK-25 | `delayedActivations = 1` | 10 |
| SG-RSK-26 | `freezeCount = 1` | 0 |
| SG-RSK-27 | `freezeCount = 2` | 10 |
| SG-RSK-28 | `tenureDays = 59` | 5 |
| SG-RSK-29 | `tenureDays = 60` | 0 |

---

## 3. Score + Level Tests

| # | السيناريو | Score | Level |
|---|---|---|---|
| SC-RSK-01 | كل الإشارات 0 | 0 | `LOW` |
| SC-RSK-02 | 30 | 30 | `LOW` |
| SC-RSK-03 | 31 | 31 | `MEDIUM` |
| SC-RSK-04 | 60 | 60 | `MEDIUM` |
| SC-RSK-05 | 61 | 61 | `HIGH` |
| SC-RSK-06 | 80 | 80 | `HIGH` |
| SC-RSK-07 | 81 | 81 | `CRITICAL` |
| SC-RSK-08 | 100 | 100 | `CRITICAL` |
| SC-RSK-09 | مجموع > 100 | 100 | `CRITICAL` |
| SC-RSK-10 | 6 إشارات كاملة | 100 | `CRITICAL` |

```java
@ParameterizedTest
@CsvSource({
    "0,LOW", "30,LOW", "31,MEDIUM", "60,MEDIUM",
    "61,HIGH", "80,HIGH", "81,CRITICAL", "100,CRITICAL"
})
void levelBoundaries(int score, RiskLevel expected) {
    assertThat(RiskLevel.fromScore(score)).isEqualTo(expected);
}
```

---

## 4. Insufficient Data Tests

| # | السيناريو | المتوقع |
|---|---|---|
| ID-RSK-01 | Tenure 13 يوم | `INSUFFICIENT_DATA` + `score = null` |
| ID-RSK-02 | Tenure 14 يوم | Score محسوب |
| ID-RSK-03 | عضو جديد بدون حضور | `INSUFFICIENT_DATA` |
| ID-RSK-04 | Reasons | فيها "بيانات غير كافية" |
| ID-RSK-05 | Draft | `null` |
| ID-RSK-06 | At-Risk List | مستبعد افتراضياً |
| ID-RSK-07 | `level=INSUFFICIENT_DATA` | يظهر |

---

## 5. Reasons + Draft Tests

| # | السيناريو | المتوقع |
|---|---|---|
| RD-RSK-01 | غياب 14 يوم | Reason فيه "14" |
| RD-RSK-02 | Drop 65% | Reason فيه "65%" |
| RD-RSK-03 | ينتهي خلال 4 أيام | Reason فيه التاريخ |
| RD-RSK-04 | Score 0 | "الحضور منتظم" |
| RD-RSK-05 | HIGH | Draft موجود |
| RD-RSK-06 | CRITICAL | Draft موجود |
| RD-RSK-07 | MEDIUM | Draft `null` |
| RD-RSK-08 | LOW | Draft `null` |
| RD-RSK-09 | Draft | فيه اسم العضو + الجيم |
| RD-RSK-10 | Reasons Count | ≤ 6 |
| RD-RSK-11 | ممنوع PII | مفيش Phone في Reasons |

---

## 6. Job Tests

| # | الاختبار | المتوقع |
|---|---|---|
| JB-RSK-01 | 3:00 AM Cairo | يشتغل |
| JB-RSK-02 | ShedLock Instance‑ين | مرة واحدة |
| JB-RSK-03 | إعادة التشغيل نفس اليوم | نفس الصف — Upsert |
| JB-RSK-04 | Org‑ين | كل واحدة في Transaction |
| JB-RSK-05 | فشل عضو | الباقي يكمل |
| JB-RSK-06 | `TenantContext.clear` | مفيش Leakage |
| JB-RSK-07 | عضو محذوف | مش بيتقيّم |
| JB-RSK-08 | عضو بدون اشتراك | مش بيتقيّم |
| JB-RSK-09 | `computed_at` | 03:00 Cairo |
| JB-RSK-10 | 500 عضو | < 5 دقائق |

---

## 7. Query / API Tests

| # | الاختبار | المتوقع |
|---|---|---|
| API-RSK-01 | At-Risk Default | HIGH + CRITICAL |
| API-RSK-02 | At-Risk `level=HIGH` | HIGH فقط |
| API-RSK-03 | At-Risk Manager | فرعه فقط |
| API-RSK-04 | Member Score | آخر Score |
| API-RSK-05 | مفيش Score | `404` |
| API-RSK-06 | Trainer عميله | `200` |
| API-RSK-07 | Trainer عضو تاني | `403` |
| API-RSK-08 | Summary | Counts صحيحة |
| API-RSK-09 | Recompute Owner | `200` |
| API-RSK-10 | Recompute Manager | `403` |
| API-RSK-11 | Recompute بدون اشتراك | `422` |
| API-RSK-12 | Receptionist | `403` |

---

## 8. Isolation Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-RSK-01 | At-Risk كـ Org A | A فقط |
| IT-RSK-02 | Member من Org B | `404` |
| IT-RSK-03 | Summary كـ Org A | A فقط |
| IT-RSK-04 | Raw JDBC بدون Context | صفر صفوف |
| IT-RSK-05 | نفس Member ID في Org‑ين | مستحيل — UUID |

---

## 9. Performance

| المسار | الهدف |
|---|---|
| Scoring لعضو واحد | < 100ms |
| Job 500 عضو | < 5 دقائق |
| `GET /risk/at-risk` | P95 < 300ms |
| `GET /risk/summary` | P95 < 500ms |
| `POST /risk/recompute` | P95 < 800ms |

---

## 10. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-005 | API-RSK-01 + RD-RSK-05/06 |
| BR-017 | SG-RSK-01..15 |
| ASM-17 | ID-RSK-01..07 |
| RSK-01 | SC-RSK-09 |
| RSK-05 | JB-RSK-03 |
| RSK-08 | API-RSK-01 |
| RSK-11 | RD-RSK-05/06 |

---

## 11. Definition of Done

- [ ] كل Signal Boundaries مغطاة.
- [ ] Level Boundaries مغطاة.
- [ ] `INSUFFICIENT_DATA` شغال.
- [ ] Reasons + Draft.
- [ ] Job Idempotent + Tenant Scope.
- [ ] At-Risk + Summary + Recompute.
- [ ] Branch + Trainer Scope.
- [ ] Isolation + Performance.
- [ ] مفيش Score يدوي.
