# Reporting — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.reporting`  
> **الطبيعة:** **Read-Only** — مفيش Entities ومفيش Migrations.  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [api-design](../../api-design.md)

---

## 1. المبدأ

`reporting` مش Feature بتملك بيانات. هي **Read Model** فوق البيانات اللي Features تانية بتملكها.

```text
 reporting
    │
    ├── ReportingQueryRepository (JdbcTemplate / Native SQL)
    │       ├── organizations / branches
    │       ├── members
    │       ├── subscriptions / subscription_events
    │       ├── payments
    │       ├── check_ins
    │       ├── risk_scores
    │       └── message_queue
    │
    └── ReportingService (تجميع + حسابات + Branch Scope)
```

### القواعد

1. **ممنوع** `@Entity` جديد.
2. **ممنوع** `INSERT` / `UPDATE` / `DELETE`.
3. **ممنوع** استدعاء `Command Service` لأي Feature.
4. **مسموح** SQL مباشر للقراءة فقط — استثناء موثق.
5. كل Query فيها `org_id = :orgId` صريح + RLS.
6. كل Query فيها `branch_id` لو المستخدم Manager.
7. النتائج دايمًا **Projections** — مش Entities.

---

## 2. الـ Time Boundaries

| الحد | التعريف |
|---|---|
| `today` | `LocalDate.now(Clock.system(CAIRO))` |
| `weekStart` | الاثنين 00:00 Cairo |
| `weekEnd` | الأحد 23:59:59.999 Cairo |
| `monthStart` | أول يوم في الشهر 00:00 Cairo |
| `monthEnd` | آخر يوم في الشهر 23:59:59.999 Cairo |

```java
public final class ReportingPeriods {
    public static LocalDate weekStart(LocalDate today) {
        return today.with(TemporalAdjusters.previousOrSame(DayOfWeek.MONDAY));
    }
    public static LocalDate weekEnd(LocalDate today) {
        return weekStart(today).plusDays(6);
    }
}
```

- كل التخزين UTC، وكل الحدود بتتحول لـ `Instant` قبل الـ Query.
- ممنوع `BETWEEN` على `DATE` مع `TIMESTAMPTZ` بدون تحويل.

---

## 3. `DashboardSummary`

### 3.1 الشكل

```java
public record DashboardSummary(
    RenewalsSummary renewalsThisWeek,
    AtRiskSummary atRiskSummary,
    long revenueMonthToDateMinor,
    String revenueMonthToDateFormatted,
    double renewalRatePercentage,
    double monthlyChurnPercentage,
    Instant generatedAt
) {}

public record RenewalsSummary(
    long expiringCount,
    long renewedCount,
    long pendingCount,
    long totalValueMinor,
    String totalValueFormatted
) {}

public record AtRiskSummary(
    long criticalCount,
    long highCount,
    long totalAtRiskValueMinor,
    String totalAtRiskValueFormatted
) {}
```

### 3.2 `renewalsThisWeek`

```sql
WITH cohort AS (
    SELECT s.id, s.member_id, s.price_minor_snapshot
    FROM subscriptions s
    JOIN members m ON m.id = s.member_id
    WHERE s.org_id = :orgId
      AND (:branchId IS NULL OR m.branch_id = :branchId)
      AND s.status IN ('ACTIVE','GRACE','EXPIRED')
      AND s.end_date BETWEEN :weekStart AND :weekEnd
),
renewed AS (
    SELECT DISTINCT c.id
    FROM cohort c
    JOIN subscriptions s2 ON s2.renewal_of = c.id
    WHERE s2.status IN ('ACTIVE','GRACE','PENDING_PAYMENT')
)
SELECT
    (SELECT COUNT(*) FROM cohort)                          AS expiring_count,
    (SELECT COUNT(*) FROM renewed)                         AS renewed_count,
    (SELECT COALESCE(SUM(price_minor_snapshot), 0)
       FROM cohort)                                        AS total_value_minor;
```

- `pendingCount = expiringCount - renewedCount` — مش استعلام منفصل.
- `renewalRate = renewedCount / expiringCount * 100` — `0` لو المقام صفر.
- `renewal_of` هو المصدر — SC-25.

**مثال:**

| البند | القيمة |
|---|---|
| `expiringCount` | 18 |
| `renewedCount` | 7 |
| `pendingCount` | 11 |
| `totalValueMinor` | 1,260,000 |
| `renewalRate` | 38.9% |

### 3.3 `atRiskSummary`

```sql
WITH latest AS (
    SELECT DISTINCT ON (member_id)
           member_id, branch_id, score, risk_level
    FROM risk_scores
    WHERE org_id = :orgId
      AND (:branchId IS NULL OR branch_id = :branchId)
    ORDER BY member_id, computed_at DESC
)
SELECT
    COUNT(*) FILTER (WHERE risk_level = 'CRITICAL') AS critical_count,
    COUNT(*) FILTER (WHERE risk_level = 'HIGH')     AS high_count
FROM latest;
```

- `totalAtRiskValueMinor` = مجموع `price_minor_snapshot` لأحدث اشتراك `ACTIVE`/`GRACE` لكل عضو `HIGH`/`CRITICAL`.
- لو مفيش Scores → 0 + `computedAt = null`.

### 3.4 `revenueMonthToDate`

```sql
SELECT COALESCE(SUM(p.amount_minor), 0)
FROM payments p
JOIN members m ON m.id = p.member_id
WHERE p.org_id = :orgId
  AND (:branchId IS NULL OR m.branch_id = :branchId)
  AND p.status = 'COMPLETED'
  AND p.created_at >= :monthStartInstant
  AND p.created_at <  :nowInstant;
```

- `REVERSED` مستبعد تلقائياً.
- مفيش Refund في MVP — لو اتضاف P1، بيتخصم هنا.

### 3.5 `monthlyChurnPercentage`

```text
cohort = أعضاء عندهم اشتراك ACTIVE/GRACE/EXPIRED
         بيتقاطع مع بداية الشهر
         (start_date <= monthStart AND end_date >= monthStart)

churned = من الـ cohort، اللي معندهمش أي اشتراك
          ACTIVE/GRACE و end_date >= today

churnRate = churned / cohort * 100
```

```sql
WITH cohort AS (
    SELECT DISTINCT s.member_id
    FROM subscriptions s
    JOIN members m ON m.id = s.member_id
    WHERE s.org_id = :orgId
      AND (:branchId IS NULL OR m.branch_id = :branchId)
      AND s.start_date <= :monthStart
      AND s.end_date   >= :monthStart
),
active_now AS (
    SELECT DISTINCT s.member_id
    FROM subscriptions s
    WHERE s.org_id = :orgId
      AND s.status IN ('ACTIVE','GRACE')
      AND s.end_date >= :today
)
SELECT
    (SELECT COUNT(*) FROM cohort) AS cohort_count,
    (SELECT COUNT(*) FROM cohort c
      WHERE NOT EXISTS (SELECT 1 FROM active_now a WHERE a.member_id = c.member_id)
    ) AS churned_count;
```

- `churnRate = churnedCount / cohortCount * 100`.
- العضو الجديد بعد بداية الشهر مش داخل الـ Cohort.
- العضو اللي جدّد قبل الانتهاء لسه `active_now` → مش Churned.

> **ملاحظة:** ده Churn تقريبي للـ MVP — مش Cohort Analysis كامل. الـ P1 ممكن يضيف شهر‑بشهر.

---

## 4. `Renewals`

### 4.1 الشكل

```java
public record RenewalItem(
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberPhone,
    String planName,
    LocalDate startDate,
    LocalDate endDate,
    long priceMinorSnapshot,
    String priceFormatted,
    String status,
    Integer daysRemaining,
    boolean renewed,
    UUID renewalSubscriptionId
) {}
```

### 4.2 الاستعلام

```sql
SELECT s.id, s.member_id, m.full_name, m.phone,
       p.name AS plan_name, s.start_date, s.end_date,
       s.price_minor_snapshot, s.status,
       s2.id AS renewal_subscription_id
FROM subscriptions s
JOIN members m ON m.id = s.member_id
JOIN plans p   ON p.id = s.plan_id
LEFT JOIN subscriptions s2 ON s2.renewal_of = s.id
WHERE s.org_id = :orgId
  AND (:branchId IS NULL OR m.branch_id = :branchId)
  AND s.end_date BETWEEN :from AND :to
  AND s.status IN ('ACTIVE','GRACE','EXPIRED')
ORDER BY s.end_date ASC, m.full_name ASC
LIMIT :limit OFFSET :offset;
```

- `renewed = renewalSubscriptionId != null`.
- `daysRemaining = DAYS.between(today, endDate)` — ممكن يكون سالب.
- Default range = الأسبوع الحالي.

---

## 5. `Collections`

### 5.1 الشكل

```java
public record CollectionsReport(
    long totalOutstandingMinor,
    String totalOutstandingFormatted,
    long count,
    List<AgingBucket> buckets,
    List<CollectionRow> items,
    Instant generatedAt
) {}

public record AgingBucket(
    String label,
    long count,
    long outstandingMinor,
    String outstandingFormatted
) {}

public record CollectionRow(
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberPhone,
    String planName,
    long priceMinorSnapshot,
    long paidTotalMinor,
    long outstandingMinor,
    String outstandingFormatted,
    int daysOutstanding,
    String agingBucket
) {}
```

### 5.2 الحساب

```sql
WITH paid AS (
    SELECT subscription_id, SUM(amount_minor) AS paid_minor
    FROM payments
    WHERE org_id = :orgId AND status = 'COMPLETED'
    GROUP BY subscription_id
)
SELECT s.id, s.member_id, m.full_name, m.phone, p.name,
       s.price_minor_snapshot,
       COALESCE(paid.paid_minor, 0) AS paid_minor,
       s.price_minor_snapshot - COALESCE(paid.paid_minor, 0) AS outstanding_minor,
       (CURRENT_DATE - s.created_at::date) AS days_outstanding
FROM subscriptions s
JOIN members m ON m.id = s.member_id
JOIN plans p   ON p.id = s.plan_id
LEFT JOIN paid ON paid.subscription_id = s.id
WHERE s.org_id = :orgId
  AND (:branchId IS NULL OR m.branch_id = :branchId)
  AND s.status = 'PENDING_PAYMENT'
  AND s.price_minor_snapshot - COALESCE(paid.paid_minor, 0) > 0
ORDER BY days_outstanding DESC;
```

### 5.3 الـ Buckets

| Label | الشرط |
|---|---|
| `0-7` | `daysOutstanding <= 7` |
| `8-14` | `8..14` |
| `15-30` | `15..30` |
| `31+` | `> 30` |

- الحساب في الـ Service بعد جلب الصفوف (أو `CASE` في SQL).
- `totalOutstandingMinor` = مجموع كل الصفوف.
- Manager → فرعه فقط.
- Pagination على الصفوف، لكن الـ Buckets محسوبة على كل النتائج — استعلام تجميعي منفصل لو الأداء مطلوب.

---

## 6. `Attendance`

### 6.1 الشكل

```java
public record AttendanceReport(
    LocalDate from,
    LocalDate to,
    long totalCheckIns,
    long uniqueMembers,
    double averagePerDay,
    int peakHour,
    long overrideCount,
    List<DailyAttendance> daily,
    List<BranchAttendance> branches,
    Instant generatedAt
) {}

public record DailyAttendance(LocalDate date, long count, long uniqueMembers) {}
public record BranchAttendance(UUID branchId, String branchName, long count) {}
```

### 6.2 الاستعلام

```sql
SELECT (c.checked_in_at AT TIME ZONE 'Africa/Cairo')::date AS day,
       COUNT(*) AS check_ins,
       COUNT(DISTINCT c.member_id) AS unique_members
FROM check_ins c
WHERE c.org_id = :orgId
  AND (:branchId IS NULL OR c.branch_id = :branchId)
  AND c.checked_in_at >= :fromInstant
  AND c.checked_in_at <  :toInstant
GROUP BY day
ORDER BY day;
```

- `peakHour`:

```sql
SELECT EXTRACT(HOUR FROM (checked_in_at AT TIME ZONE 'Africa/Cairo')) AS hour,
       COUNT(*) AS cnt
FROM check_ins
WHERE ...
GROUP BY hour
ORDER BY cnt DESC
LIMIT 1;
```

- `averagePerDay = total / number_of_days` — `0` لو مفيش أيام.
- المدى الأقصى **90 يوم**.
- Default آخر 7 أيام.

---

## 7. `Risk Distribution`

### 7.1 الشكل

```java
public record RiskReport(
    long lowCount,
    long mediumCount,
    long highCount,
    long criticalCount,
    long insufficientDataCount,
    Double averageScore,
    long highCriticalValueMinor,
    String highCriticalValueFormatted,
    Instant computedAt,
    List<TopRiskMember> topRisks
) {}

public record TopRiskMember(
    UUID memberId,
    String memberName,
    int score,
    String riskLevel,
    String recommendedAction
) {}
```

### 7.2 الحساب

- Latest Score لكل عضو (`DISTINCT ON`).
- `averageScore` = `AVG(score)` للـ Scores غير `NULL` — `null` لو مفيش.
- `topRisks` = أعلى 5 `CRITICAL`/`HIGH` بالـ Score.
- `highCriticalValueMinor` من اشتراكاتهم الحالية.
- Manager → فرعه.

---

## 8. Member Timeline

### 8.1 الشكل

```java
public record TimelineItem(
    String type,          // SUBSCRIPTION, PAYMENT, CHECK_IN, MESSAGE, RISK
    UUID entityId,
    Instant occurredAt,
    String title,
    String description,
    Long amountMinor,
    String amountFormatted,
    JsonNode metadata
) {}
```

### 8.2 الـ Union

```sql
SELECT 'SUBSCRIPTION' AS type, se.id AS entity_id, se.created_at AS occurred_at,
       se.type AS title, se.reason AS description,
       NULL::bigint AS amount_minor, se.metadata
FROM subscription_events se
WHERE se.org_id = :orgId AND se.subscription_id IN (
    SELECT id FROM subscriptions WHERE org_id = :orgId AND member_id = :memberId
)

UNION ALL

SELECT 'PAYMENT', p.id, p.created_at, p.method::text,
       p.receipt_number, p.amount_minor, NULL::jsonb
FROM payments p
WHERE p.org_id = :orgId AND p.member_id = :memberId

UNION ALL

SELECT 'CHECK_IN', c.id, c.checked_in_at, c.method::text,
       NULL, NULL::bigint, NULL::jsonb
FROM check_ins c
WHERE c.org_id = :orgId AND c.member_id = :memberId

UNION ALL

SELECT 'MESSAGE', mq.id, mq.created_at, mq.template::text,
       mq.status::text, NULL::bigint, NULL::jsonb
FROM message_queue mq
WHERE mq.org_id = :orgId AND mq.member_id = :memberId

UNION ALL

SELECT 'RISK', NULL, rs.computed_at, rs.risk_level::text,
       rs.recommended_action, rs.score::bigint, rs.reasons
FROM risk_scores rs
WHERE rs.org_id = :orgId AND rs.member_id = :memberId

ORDER BY occurred_at DESC
LIMIT :limit OFFSET :offset;
```

- `type` بيتحول لـ Enum في الـ Service.
- `description` بيتعملها Enrichment بسيط (مثلاً `PAYMENT` → "دفعة 900.00 EGP").
- ممنوع `audit_log` في الـ MVP — بيضيف ضوضاء.
- Manager/Receptionist → فرعهم فقط (العضو لازم يكون في فرعهم).
- Pagination 20/صفحة، أقصى 100.

### 8.3 الترتيب

- `occurredAt DESC`.
- لو نفس اللحظة → `type` كـ Tiebreaker.

---

## 9. Branch Scope

| المستخدم | `branchId` | التطبيق |
|---|---|---|
| Owner | اختياري | لو موجود → Filter، غير كده كل الفروع. |
| Manager | إلزامي من الـ Token | ممنوع يبعت Branch تاني. |
| Receptionist | إلزامي من الـ Token | Timeline بس. |
| Trainer | — | `403`. |

```java
UUID scopeBranchId() {
    UserContext user = SecurityUtils.currentUser();
    return switch (user.role()) {
        case GYM_OWNER -> requestBranchId(); // nullable
        case BRANCH_MANAGER, RECEPTIONIST -> user.branchId();
        default -> throw new AccessDeniedException();
    };
}
```

- كل الـ SQL بيستخدم `(:branchId IS NULL OR m.branch_id = :branchId)`.
- `risk_scores` / `check_ins` / `message_queue` فيهم `branch_id` مباشرة.
- `payments` / `subscriptions` بيحتاجوا Join مع `members`.

---

## 10. Empty States

| الحالة | الاستجابة |
|---|---|
| مفيش تجديدات | `expiringCount = 0` + `renewalRate = 0` |
| مفيش مدفوعات | `revenue = 0` |
| مفيش مخاطر | كل Counts = 0 + `computedAt = null` |
| مفيش حضور | `totalCheckIns = 0` + `daily = []` |
| مفيش Timeline | `content = []` |
| مفيش Collections | `count = 0` + `buckets` كلها صفر |

- ممنوع `null` بدل Array.
- ممنوع `NaN` أو `Infinity` — اقسم بحذر.

---

## 11. الأداء

| القاعدة | التفصيل |
|---|---|
| Indexes | `(org_id, created_at)` على `payments`، `(org_id, member_id, checked_in_at)` على `check_ins`، `(org_id, risk_level)` على `risk_scores`. |
| Projections | ممنوع `SELECT *` — الأعمدة المطلوبة فقط. |
| Pagination | كل القوائم Paginated. |
| N+1 | ممنوع — جلب الأسماء بـ Join أو Batch. |
| Date Range | أقصى 366 يوم (Attendance 90). |
| Counts | استعلامات تجميعية منفصلة — مش في الـ Java. |
| Cache | مفيش في MVP. |
| Read-Only TX | `@Transactional(readOnly = true)`. |

---

## 12. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-REP-01 | مفيش Writes | Arch Test + ReadOnly TX |
| INV-REP-02 | كل Query فيها `org_id` | Code Review + Test |
| INV-REP-03 | Manager Branch Scope | Service + Test |
| INV-REP-04 | المبالغ `Long` | Projection Types |
| INV-REP-05 | مفيش N+1 | Query Count Test |
| INV-REP-06 | Date Range صالح | Validation |
| INV-REP-07 | Empty State | API Test |
| INV-REP-08 | مفيش Entity جديد | Arch Test |
| INV-REP-09 | `renewalRate` بين 0 و100 | Service |
| INV-REP-10 | `churnRate` بين 0 و100 | Service |

---

## 13. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `InvalidDateRangeException` | 400 | `from > to` أو مدى كبير |
| `MemberNotFoundException` | 404 | Timeline لعضو مش موجود / فرع تاني |
| `ReportAccessDeniedException` | 403 | Manager يحاول Summary / Trainer |
| `ReportQueryException` | 500 | فشل استعلام |

---

## 14. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [database-strategy](../../architecture/database-strategy.md#8-الفهارس-index-catalog)
