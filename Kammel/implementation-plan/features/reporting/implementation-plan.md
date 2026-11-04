# Reporting — خطة التنفيذ

> **الـ Phase:** 3 · **المدة:** 4–5 أيام · **الترتيب:** آخر Feature  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-REP-001 — Query Repository Foundation

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-008, TSK-CHK-007, TSK-RSK-007, TSK-NOT-007 |
| **المخرجات** | `ReportingQueryRepository` + Projections |
| **معيار القبول** | Read-Only + Org/Branch Scoped + مفيش N+1 |

**خطوات:**
1. `JdbcTemplate` أو `EntityManager` Native.
2. Named Parameters — ممنوع String Concatenation.
3. كل Query فيها `org_id`.
4. Branch Filter موحد.
5. Projections Records.
6. `@Transactional(readOnly = true)`.
7. Query Count Test.

---

### TSK-REP-002 — Dashboard Summary

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-REP-001 |
| **المخرجات** | `getSummary(...)` |
| **معيار القبول** | Renewals + At-Risk + Revenue + Churn |

**خطوات:**
1. `weekStart` / `weekEnd` (الاثنين–الأحد Cairo).
2. Renewals Cohort + `renewal_of`.
3. At-Risk Latest + Counts.
4. Revenue MTD (`COMPLETED` فقط).
5. Churn Cohort.
6. `renewalRate` / `churnRate` بحذر من القسمة على صفر.
7. `MoneyUtil` للتنسيق.

---

### TSK-REP-003 — Renewals + Collections

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-REP-001 |
| **المخرجات** | `listRenewals(...)`, `getCollections(...)` |
| **معيار القابل** | Dates + Renewed Flag + Aging Buckets |

**خطوات:**
1. Renewals Query + `renewal_of`.
2. `daysRemaining`.
3. Collections Query + `paid` CTE.
4. `outstanding`.
5. Aging Buckets.
6. Totals.
7. Pagination.

---

### TSK-REP-004 — Attendance + Risk

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-REP-001 |
| **المخرجات** | `getAttendance(...)`, `getRiskDistribution(...)` |
| **معيار القبول** | Daily Series + Peak Hour + Risk Counts |

**خطوات:**
1. Attendance Daily Series.
2. `uniqueMembers`.
3. `peakHour` (Cairo).
4. `overrideCount`.
5. Branch Breakdown.
6. Risk Latest Distribution.
7. `averageScore`.
8. Top 5 Risks.
9. Value للـ HIGH/CRITICAL.

---

### TSK-REP-005 — Member Timeline

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-REP-001 |
| **المخرجات** | `getMemberTimeline(...)` |
| **معيار القبول** | 5 مصادر + ترتيب + Pagination |

**خطوات:**
1. `UNION ALL` على 5 جداول.
2. `ORDER BY occurred_at DESC`.
3. Mapping لـ `TimelineItem`.
4. Enrichment بسيط.
5. Branch Check.
6. Pagination 20/100.
7. Test على عضو فيه كل الأنواع.

---

### TSK-REP-006 — API + Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-REP-002..005 |
| **المخرجات** | `ReportingController` + 6 Endpoints + Tests |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

**خطوات:**
1. `GET /dashboard/summary`.
2. `GET /dashboard/renewals`.
3. `GET /dashboard/collections`.
4. `GET /dashboard/attendance`.
5. `GET /dashboard/risk`.
6. `GET /dashboard/members/{id}/timeline`.
7. Branch + Role Tests.
8. Performance Tests.

---

## 2. الترتيب

```text
TSK-REP-001 Query Repository Foundation
      ↓
TSK-REP-002 Summary
      ↓
TSK-REP-003 Renewals + Collections
      ↓
TSK-REP-004 Attendance + Risk
      ↓
TSK-REP-005 Member Timeline
      ↓
TSK-REP-006 API + Tests
```

---

## 3. معايير القبول النهائية

- [ ] `ReportingQueryRepository` Read-Only.
- [ ] كل Query Org-scoped.
- [ ] Branch Scope.
- [ ] Summary KPIs صحيحة.
- [ ] Renewals + `renewal_of`.
- [ ] Collections + Aging.
- [ ] Attendance + Peak Hour.
- [ ] Risk Distribution.
- [ ] Timeline 5 مصادر.
- [ ] Empty States.
- [ ] مفيش N+1.
- [ ] P95 < 500ms.
- [ ] Tests خضراء.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| Query بطيء | Indexes + Date Range + Projections |
| Manager يشوف Org Revenue | C-14 + Role Test |
| N+1 | Join / Batch + Query Count |
| Churn غلط | Definition موثق + Tests |
| Renewal Rate > 100% | `renewal_of` Unique + Test |
| Timeline بطيء | Union + Limit + Index |
| Timezone غلط | Cairo Boundaries + Test |
| Cache Stale | مفيش Cache في MVP |
| SQL Injection | Named Params |
| Read Model يتعارض مع Writes | Read-Only TX + Arch Test |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
