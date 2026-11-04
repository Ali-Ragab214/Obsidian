# Risk — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.risk.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. `RiskScore`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `orgId` | `UUID` | `NOT NULL` | RLS |
| `branchId` | `UUID` | `NOT NULL` | **SC-23** — فلترة المدير |
| `memberId` | `UUID` | PK جزء | — |
| `computedAt` | `Instant` | PK جزء | 03:00 Cairo |
| `score` | `Integer` | `NULL` أو 0..100 | `NULL` لـ `INSUFFICIENT_DATA` |
| `riskLevel` | `RiskLevel` | `NOT NULL` | مخزّن |
| `reasons` | `JsonNode` | `NOT NULL` | JSONB Array |
| `signals` | `JsonNode` | `NULL` | JSONB Object |
| `modelVersion` | `String` | `NOT NULL`, ≤ 30 | `rules-v1` |
| `recommendedAction` | `String` | `NULL`, ≤ 500 | عربي |
| `draftMessage` | `String` | `NULL` | عربي |

```java
@Entity
@Table(name = "risk_scores")
@IdClass(RiskScoreId.class)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class RiskScore implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(name = "branch_id", nullable = false)
    private UUID branchId;

    @Id
    @Column(name = "member_id", nullable = false)
    private UUID memberId;

    @Id
    @Column(name = "computed_at", nullable = false)
    private Instant computedAt;

    @Column
    private Integer score;

    @Enumerated(EnumType.STRING)
    @Column(name = "risk_level", nullable = false, length = 20)
    private RiskLevel riskLevel;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(nullable = false, columnDefinition = "jsonb")
    private JsonNode reasons;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(columnDefinition = "jsonb")
    private JsonNode signals;

    @Column(name = "model_version", nullable = false, length = 30)
    private String modelVersion;

    @Column(name = "recommended_action", length = 500)
    private String recommendedAction;

    @Column(name = "draft_message", columnDefinition = "text")
    private String draftMessage;
}

public record RiskScoreId(UUID memberId, Instant computedAt) implements Serializable {}
```

> **Append-Only** مع استثناء واحد: إعادة تشغيل نفس اليوم بتعمل `UPSERT` على نفس `(member_id, computed_at)` — عشان الـ Job يبقى Idempotent.

---

## 2. الـ Enums

```java
public enum RiskLevel {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL,
    INSUFFICIENT_DATA
}
```

> `INSUFFICIENT_DATA` مش ضمن 0..100 — هو حالة بيانات، مش درجة.

---

## 3. الـ Formula

```text
RiskScore = min(100, Σ points(signal_i))
```

- كل إشارة بتدي نقاط ثابتة (مش نسبة من وزن).
- مجموع الأوزان = 100.
- الـ Score دايمًا `Integer`.

---

## 4. الـ Signals (rules-v1)

| # | Signal | Max Points | الحساب |
|:---:|---|:---:|---|
| 1 | Days Since Last Visit | 30 | `<=3 → 0` · `4–7 → 10` · `8–13 → 20` · `>=14 → 30` |
| 2 | Attendance Frequency Drop | 25 | مقارنة آخر 14 يوم بـ 14 يوم اللي قبلهم |
| 3 | Days Remaining | 20 | `<=3 → 20` · `4–7 → 15` · `8–14 → 5` · `>14 → 0` |
| 4 | Payment History Friction | 10 | `delayedActivations > 0 → 10` |
| 5 | Freeze Frequency | 10 | `freezeCount >= 2 → 10` |
| 6 | Tenure Hazard | 5 | `tenureDays < 60 → 5` |

### 4.1 Attendance Frequency Drop

```text
visitsLast14  = countVisits(memberId, today-13, today+1)
visitsPrior14 = countVisits(memberId, today-27, today-13)

لو visitsPrior14 == 0:
    dropPct = 0          // مفيش Baseline — مش هنعاقب عضو جديد
    points  = 0
غير كده:
    dropPct = round((visitsPrior14 - visitsLast14) / visitsPrior14 * 100)
    dropPct > 50 → 25
    dropPct > 25 → 15
    else → 0
```

- `dropPct` ممكن يكون سالب (تحسن) → 0.
- ممنوع `double` في التخزين — الحساب مؤقت وبعدين `int`.

### 4.2 Days Since Last Visit

```text
lastVisit = getLastVisit(memberId)
لو lastVisit == null:
    daysSince = DAYS.between(member.createdAt(Cairo), today)
غير كده:
    daysSince = DAYS.between(lastVisit(Cairo), today)
```

- لو العضو جديد ومجاش خالص → `daysSince` = عمر العضوية.

### 4.3 Days Remaining

```text
accessSub = findAccessSubscription(memberId)   // ACTIVE أو GRACE
daysRemaining = DAYS.between(today, accessSub.endDate)
```

- في الـ `GRACE` بيكون سالب → يقع في `<=3` → 20 نقطة.
- لو مفيش Access Sub → العضو مش ضمن الـ Job أصلاً.

### 4.4 Payment History Friction — Proxy

الـ `risk` مش بيقرأ `payments` مباشرة (احترام حدود الـ Features). بدل كده:

```java
subscriptionQueryService.countDelayedActivations(memberId, 3)
```

**التعريف:**

```text
delayedActivation = Subscription
    WHERE (status != 'PENDING_PAYMENT'
           AND EXISTS activation_event
           AND activation_event.created_at > subscription.created_at + 3 days)
       OR (status = 'PENDING_PAYMENT'
           AND subscription.created_at < now - 7 days)
```

- `> 0` → 10 نقاط.
- **القيد:** ده Proxy لتأخر التفعيل/السداد، مش تحليل دفع كامل. لو عايزين دقة أعلى → نضيف `risk → payment` Read Contract في P1.
- الـ Query Service بيتوسع بـ `countDelayedActivations(UUID memberId, int graceDays)`.

### 4.5 Freeze Frequency

```java
subscriptionQueryService.countFreezes(memberId)
```

- عدد `SubscriptionEvent` من نوع `FROZEN` لكل اشتراكات العضو.
- `>= 2` → 10 نقاط.

> التجميد المتكرر إشارة لعدم الانتظام — حتى لو العضو رجع.

### 4.6 Tenure Hazard

```text
tenureDays = DAYS.between(member.createdAt(Cairo), today)
tenureDays < 60 → 5
```

- أول 60 يوم هي أعلى فترة تسريب.

---

## 5. التصنيف

| Score | Level | اللون | الإجراء |
|---|:---:|:---:|---|
| 0–30 | `LOW` | أخضر | متابعة عادية — مفيش إجراء. |
| 31–60 | `MEDIUM` | أصفر | مراقبة + رسالة ودية اختيارية. |
| 61–80 | `HIGH` | برتقالي | تواصل خلال الأسبوع برسالة "Miss You". |
| 81–100 | `CRITICAL` | أحمر | تواصل النهارده + عرض تجديد. |
| — | `INSUFFICIENT_DATA` | رمادي | استنى بيانات حضور. |

```java
public static RiskLevel fromScore(int score) {
    if (score <= 30) return LOW;
    if (score <= 60) return MEDIUM;
    if (score <= 80) return HIGH;
    return CRITICAL;
}
```

> الحدود **شاملة** — 30 → `LOW`، 31 → `MEDIUM`، 80 → `HIGH`، 81 → `CRITICAL`.

---

## 6. `INSUFFICIENT_DATA`

### 6.1 الشروط

```text
لو tenureDays < 14:
    score = null
    riskLevel = INSUFFICIENT_DATA
    reasons = ["بيانات غير كافية — العضو جديد (أقل من 14 يوم)."]
    recommendedAction = "انتظر أسبوعين بيانات حضور قبل التقييم."
    draftMessage = null
```

- `ASM-17`: 14 يوم حد أدنى.
- مفيش `score = 0` للعضو الجديد — لأن ده هيديه `LOW` مضلل.
- الـ At-Risk List بتستبعد `INSUFFICIENT_DATA` افتراضياً.

### 6.2 بعد 14 يوم

- حتى لو مفيش حضور خالص → Score محسوب.
- `daysSinceLastVisit` هيكون 14+ → 30 نقطة، وممكن يوصل `MEDIUM`/`HIGH` حسب باقي الإشارات.

---

## 7. الـ Reasons

### 7.1 التوليد

| الشرط | النص |
|---|---|
| `daysSince >= 14` | `"غياب {n} يوم متواصل"` |
| `daysSince 8–13` | `"آخر زيارة من {n} يوم"` |
| `dropPct > 50` | `"انخفاض الحضور بنسبة {pct}% خلال آخر 14 يوم"` |
| `dropPct 26–50` | `"الحضور قل بنسبة {pct}% مقارنة بالأسبوعين اللي فاتوا"` |
| `daysRemaining <= 14` | `"الاشتراك ينتهي خلال {n} يوم ({date})"` |
| `delayedActivations > 0` | `"تأخر في تفعيل/سداد {n} اشتراك"` |
| `freezeCount >= 2` | `"تجميد الاشتراك {n} مرات"` |
| `tenureDays < 60` | `"عضو جديد — أول 60 يوم"` |
| Score = 0 | `"الحضور منتظم ومفيش إشارات خطر"` |

### 7.2 القواعد

- الأسباب **عربية فصحى مبسطة** — مش عامية زيادة عن اللزوم.
- ممنوع أرقام مالية أو Phone في الـ reasons.
- الترتيب: الأهم أولاً (غياب → حضور → اشتراك → دفع → تجميد → tenure).
- الحد الأقصى 6 أسباب.

---

## 8. الإجراء المقترح

| Level | `recommendedAction` |
|---|---|
| `LOW` | `"متابعة عادية — مفيش إجراء مطلوب."` |
| `MEDIUM` | `"راقب العضو، وممكن رسالة ودية لو غاب أسبوع كمان."` |
| `HIGH` | `"تواصل خلال الأسبوع برسالة Miss You وعرض تجديد."` |
| `CRITICAL` | `"تواصل النهارده — عرض تجديد أو جلسة مع الكابتن."` |
| `INSUFFICIENT_DATA` | `"انتظر بيانات حضور أكتر قبل التقييم."` |

---

## 9. الـ Draft Message

### 9.1 الشروط

- يتولد لـ `HIGH` و`CRITICAL` فقط.
- `MEDIUM` → `null` (مش عايزين Spam).
- `LOW` / `INSUFFICIENT_DATA` → `null`.

### 9.2 القالب

```text
يا {memberName}، وحشتنا في {gymName}! 🏋️‍♂️
لاحظنا إنك بقالك {daysAbsent} يوم مجتش.
{endDateClause}
تحب نجهّزلك ميعاد تمرين الأسبوع ده؟ 💪
```

- `endDateClause`:
  - لو `daysRemaining <= 14`: `"واشتراكك بينتهي يوم {endDate}."`
  - غير كده: `"كابتن الجيم مستنيك."`
- ممنوع أي خصم أو وعد مالي في الـ Draft — الـ Manager يعدله.
- الـ Draft بيتخزن في `risk_scores.draft_message` و`notification` بيستخدمه.

### 9.3 مثال

```json
{
  "memberName": "Karim Adel",
  "score": 85,
  "riskLevel": "CRITICAL",
  "reasons": [
    "غياب 14 يوم متواصل",
    "انخفاض الحضور بنسبة 65% خلال آخر 14 يوم",
    "الاشتراك ينتهي خلال 4 يوم (2026-11-07)"
  ],
  "recommendedAction": "تواصل النهارده — عرض تجديد أو جلسة مع الكابتن.",
  "draftMessage": "يا كريم، وحشتنا في جولدز جيم! 🏋️‍♂️\nلاحظنا إنك بقالك 14 يوم مجتش.\nواشتراكك بينتهي يوم 2026-11-07.\nتحب نجهّزلك ميعاد تمرين الأسبوع ده؟ 💪"
}
```

---

## 10. الـ Job

### 10.1 الإعداد

```java
@Scheduled(cron = "0 0 3 * * *", zone = "Africa/Cairo")
@SchedulerLock(name = "riskScoringJob", lockAtMostFor = "45m")
public void run() {
    List<UUID> orgIds = organizationQueryService.listActiveOrgIds();
    for (UUID orgId : orgIds) {
        tenantTransactionRunner.runInTenant(orgId, this::scoreTenant);
    }
}
```

### 10.2 خطوات الـ Tenant

1. `List<UUID> memberIds = subscriptionQueryService.listMembersWithAccess()`.
2. لكل عضو:
   - `MemberSummary member = memberQueryService.getMember(memberId)`.
   - `SubscriptionSummary sub = subscriptionQueryService.findAccessSubscription(memberId)`.
   - احسب الإشارات.
   - `upsert`.
   - لو عضو فشل → `log.error` + كمّل.
3. Commit.

### 10.3 `computed_at`

```text
runDate = LocalDate.now(clock.withZone(CAIRO))
computedAt = runDate.atTime(3, 0).atZone(CAIRO).toInstant()
```

- نفس اليوم → نفس `computed_at` → `UPSERT`.
- إعادة التشغيل مش بتنشئ صفوف جديدة.

### 10.4 الـ Upsert

```sql
INSERT INTO risk_scores (
    org_id, branch_id, member_id, computed_at, score, risk_level,
    reasons, signals, model_version, recommended_action, draft_message
) VALUES (...)
ON CONFLICT (member_id, computed_at) DO UPDATE SET
    score = EXCLUDED.score,
    risk_level = EXCLUDED.risk_level,
    reasons = EXCLUDED.reasons,
    signals = EXCLUDED.signals,
    recommended_action = EXCLUDED.recommended_action,
    draft_message = EXCLUDED.draft_message;
```

> `org_id` / `branch_id` مش بيتغيروا في الـ Update — العضو مش بيتنقل فرع في نفس اللحظة.

---

## 11. الـ Query Service

```java
public interface RiskQueryService {

    Optional<RiskScoreSummary> getLatestScore(UUID memberId);

    PageResponse<AtRiskMemberSummary> listAtRisk(AtRiskFilter filter, Pageable pageable);

    RiskSummary getSummary(AtRiskFilter filter);

    void recompute(UUID memberId);
}

public record RiskScoreSummary(
    UUID memberId,
    Integer score,
    RiskLevel riskLevel,
    List<String> reasons,
    JsonNode signals,
    String recommendedAction,
    String draftMessage,
    String modelVersion,
    Instant computedAt
) {}

public record AtRiskMemberSummary(
    UUID memberId,
    String fullName,
    String phone,
    Integer score,
    RiskLevel riskLevel,
    List<String> reasons,
    String recommendedAction,
    String draftMessage,
    long subscriptionValueMinor,
    String subscriptionValueFormatted,
    Instant computedAt
) {}

public record RiskSummary(
    long lowCount,
    long mediumCount,
    long highCount,
    long criticalCount,
    long insufficientDataCount,
    long totalAtRiskValueMinor,
    String totalAtRiskValueFormatted
) {}
```

### 11.1 `listAtRisk`

- بيجيب **آخر Score** لكل عضو:

```sql
SELECT DISTINCT ON (member_id) *
FROM risk_scores
WHERE org_id = :orgId
  AND (:branchId IS NULL OR branch_id = :branchId)
ORDER BY member_id, computed_at DESC
```

- بعدين Filter على `risk_level IN ('HIGH','CRITICAL')`.
- Pagination على الـ Outer Query.
- Enrich بـ `memberName` / `phone` عبر `MemberQueryService` (Batch).
- `subscriptionValueMinor` من `SubscriptionQueryService.findAccessSubscription` — لو مفيش، `0`.

### 11.2 `getSummary`

- نفس الـ Latest Query + `GROUP BY risk_level`.
- `totalAtRiskValueMinor` = مجموع `priceMinorSnapshot` للـ `HIGH` + `CRITICAL`.

### 11.3 `recompute`

- Owner فقط.
- بتشتغل لعضو واحد بشكل Synchronous.
- لو العضو مش عنده اشتراك `ACTIVE`/`GRACE` → `422 NO_ACTIVE_SUBSCRIPTION`.
- بتستخدم نفس `RiskScoringService` — مفيش Logic مكرر.

---

## 12. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-RSK-01 | `score` بين 0 و100 أو `NULL` | DB CHECK |
| INV-RSK-02 | `risk_level` من الـ Enum | DB CHECK |
| INV-RSK-03 | `INSUFFICIENT_DATA` → `score = NULL` | Service |
| INV-RSK-04 | `reasons` Array غير فاضي | Service |
| INV-RSK-05 | `model_version` ثابت `rules-v1` | Service |
| INV-RSK-06 | مفيش Score لعضو محذوف | Job Query |
| INV-RSK-07 | `branch_id` = فرع العضو وقت الحساب | Service |
| INV-RSK-08 | Job Idempotent | Upsert + Test |
| INV-RSK-09 | `draft_message` لـ HIGH/CRITICAL فقط | Service |
| INV-RSK-10 | ممنوع تعديل Score يدوي | No Setter + API |
| INV-RSK-11 | `org_id` من الـ Context | Service |
| INV-RSK-12 | كل Org في Transaction منفصلة | `TenantTransactionRunner` |

---

## 13. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `RiskScoreNotFoundException` | 404 | مفيش Score لسه |
| `NoActiveSubscriptionException` | 422 | Recompute لعضو غير نشط |
| `RecomputeNotAllowedException` | 403 | غير Owner |
| `ScoringException` | 500 | فشل حساب عضو — Job بيكمل |

---

## 14. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [checkin/domain-design](../checkin/domain-design.md)
