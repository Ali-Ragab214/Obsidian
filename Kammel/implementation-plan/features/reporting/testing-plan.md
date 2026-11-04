# Reporting — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| KPIs | أرقام صحيحة |
| Renewals | `renewal_of` + Cohort |
| Collections | Aging + Outstanding |
| Attendance | Cairo + Peak |
| Timeline | 5 مصادر + ترتيب |
| Scope | Org + Branch |
| Read-Only | صفر Writes |

---

## 2. Summary Tests

| # | السيناريو | المتوقع |
|---|---|---|
| SM-REP-01 | 18 منتهي + 7 متجدد | `expiring=18`, `renewed=7`, `pending=11` |
| SM-REP-02 | مفيش منتهي | `renewalRate = 0` |
| SM-REP-03 | كلهم متجددين | `renewalRate = 100` |
| SM-REP-04 | Revenue MTD | مجموع `COMPLETED` فقط |
| SM-REP-05 | دفعة `REVERSED` | مستبعدة |
| SM-REP-06 | دفعة قبل الشهر | مستبعدة |
| SM-REP-07 | Churn Cohort | عضو من بداية الشهر |
| SM-REP-08 | عضو جديد | مش في الـ Cohort |
| SM-REP-09 | عضو جدّد | مش Churned |
| SM-REP-10 | At-Risk Counts | من Latest Scores |
| SM-REP-11 | مفيش Risk | أصفار + `computedAt=null` |
| SM-REP-12 | Manager | `403` |

```java
@Test
void summary_renewalRate_handlesZeroCohort() {
    DashboardSummary summary = reportingService.summary(orgId, null);

    assertThat(summary.renewalsThisWeek().expiringCount()).isZero();
    assertThat(summary.renewalRatePercentage()).isZero();
    assertThat(summary.renewalRatePercentage()).isNotNaN();
}
```

---

## 3. Renewals Tests

| # | السيناريو | المتوقع |
|---|---|---|
| RN-REP-01 | اشتراك ينتهي هذا الأسبوع | ظاهر |
| RN-REP-02 | `renewal_of` موجود | `renewed=true` |
| RN-REP-03 | تجديد بسعر جديد | السعر القديم في الصف |
| RN-REP-04 | `daysRemaining` سالب (Expired) | سالب |
| RN-REP-05 | Manager فرع A | A فقط |
| RN-REP-06 | Date Range > 366 | `400` |
| RN-REP-07 | `from > to` | `400` |
| RN-REP-08 | Pagination | مستقر |

---

## 4. Collections Tests

| # | السيناريو | المتوقع |
|---|---|---|
| CL-REP-01 | اشتراك بمتبقي | ظاهر |
| CL-REP-02 | اشتراك مدفوع | مش ظاهر |
| CL-REP-03 | Aging 0-7 | Bucket صح |
| CL-REP-04 | Aging 31+ | Bucket صح |
| CL-REP-05 | Total | مجموع كل الصفوف |
| CL-REP-06 | Manager فرع | فرعه فقط |
| CL-REP-07 | مفيش متأخرات | أصفار + `items=[]` |
| CL-REP-08 | Buckets على كل النتائج | مش الصفحة |

---

## 5. Attendance Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-REP-01 | 7 أيام | 7 صفوف Daily |
| AT-REP-02 | يوم بدون حضور | صفر — مش محذوف |
| AT-REP-03 | `uniqueMembers` | Distinct |
| AT-REP-04 | `peakHour` | 19 لو أعلى |
| AT-REP-05 | Cairo 23:30 | اليوم الصح |
| AT-REP-06 | Override Count | صحيح |
| AT-REP-07 | Manager فرع | فرعه |
| AT-REP-08 | Range > 90 | `400` |
| AT-REP-09 | مفيش حضور | `total=0`, `average=0` |
| AT-REP-10 | Branch Breakdown | مجموع = Total |

---

## 6. Risk Report Tests

| # | السيناريو | المتوقع |
|---|---|---|
| RK-REP-01 | Latest Scores | Counts صح |
| RK-REP-02 | مفيش Scores | أصفار |
| RK-REP-03 | `averageScore` | يستبعد `INSUFFICIENT_DATA` |
| RK-REP-04 | Top Risks | أعلى 5 |
| RK-REP-05 | Value | اشتراكات HIGH/CRITICAL |
| RK-REP-06 | Manager فرع | فرعه |
| RK-REP-07 | Receptionist | `403` |

---

## 7. Timeline Tests

| # | السيناريو | المتوقع |
|---|---|---|
| TL-REP-01 | عضو فيه 5 أنواع | 5 Types |
| TL-REP-02 | الترتيب | `occurredAt DESC` |
| TL-REP-03 | Pagination | 20/صفحة |
| TL-REP-04 | Filter `type=PAYMENT` | Payments فقط |
| TL-REP-05 | عضو Org B | `404` |
| TL-REP-06 | Manager فرع B | `404` |
| TL-REP-07 | Receptionist فرعه | `200` |
| TL-REP-08 | مفيش أحداث | `content=[]` |
| TL-REP-09 | Date Range | يفلتر |
| TL-REP-10 | Amount | `Long` + Formatted |

---

## 8. Scope + Security Tests

| # | الاختبار | المتوقع |
|---|---|---|
| SC-REP-01 | Owner كل الفروع | `200` |
| SC-REP-02 | Owner `branchId` | Filter |
| SC-REP-03 | Manager فرع تاني | `403` / Filter |
| SC-REP-04 | Receptionist Summary | `403` |
| SC-REP-05 | Trainer أي Endpoint | `403` |
| SC-REP-06 | Raw JDBC بدون Context | صفر صفوف |
| SC-REP-07 | Org A → Org B | صفر |
| SC-REP-08 | SQL Injection في `search` | Named Params |

---

## 9. Read-Only / Architecture Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RO-REP-01 | مفيش `@Entity` في `reporting` | Arch Test |
| RO-REP-02 | مفيش `Repository.save` | Arch Test |
| RO-REP-03 | `@Transactional(readOnly = true)` | Reflection |
| RO-REP-04 | مفيش Command Service | Arch Test |
| RO-REP-05 | كل Query فيها `org_id` | Code Review + Test |
| RO-REP-06 | مفيش N+1 | Query Count ≤ 10 |

---

## 10. Performance

| المسار | الهدف |
|---|---|
| `GET /summary` | P95 < 500ms |
| `GET /renewals` | P95 < 300ms |
| `GET /collections` | P95 < 400ms |
| `GET /attendance` | P95 < 400ms |
| `GET /risk` | P95 < 400ms |
| `GET /timeline` | P95 < 300ms |
| 5000 عضو / 100k Check-in | مفيش Full Scan |

---

## 11. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-004 | SC-REP-01/02 |
| BR-005 | RK-REP-01..04 |
| BR-031 | P1 — Shift Variance |
| C-14 | SM-REP-12 + SC-REP-04 |
| REP-03 | RO-REP-01..04 |
| REP-04 | RO-REP-06 |
| REP-05 | RN-REP-06 + AT-REP-08 |
| REP-07 | AT-REP-05 |

---

## 12. Definition of Done

- [ ] Summary KPIs صحيحة.
- [ ] Renewals + `renewal_of`.
- [ ] Collections + Aging.
- [ ] Attendance + Peak.
- [ ] Risk Distribution.
- [ ] Timeline 5 مصادر.
- [ ] Scope + Security.
- [ ] Read-Only Architecture Tests.
- [ ] Empty States.
- [ ] Performance.
