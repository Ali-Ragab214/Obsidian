# Plan — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.plan.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md)

---

## 1. `Plan`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL` | Tenant |
| `name` | `String` | `NOT NULL`, 2–100 | فريد داخل الـ Org |
| `type` | `PlanType` | `NOT NULL` | 8 أنواع |
| `durationDays` | `int` | `> 0` | — |
| `sessionCount` | `Integer` | `NULL` أو `> 0` | للباقات القائمة على الجلسات |
| `priceMinor` | `long` | `> 0` | قروش |
| `freezeMaxDays` | `int` | `>= 0` | `0` = ممنوع التجميد |
| `graceDays` | `int` | `>= 0` | `0` = مفيش Grace |
| `timeRestriction` | `String` | `NULL`, ≤ 20 | `MORNING` / `EVENING` |
| `grantsAccess` | `boolean` | `NOT NULL` | `true` |
| `active` | `boolean` | `NOT NULL` | `true` |
| `createdAt` / `updatedAt` | `Instant` | `NOT NULL` | — |

```java
@Entity
@Table(name = "plans")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Plan extends BaseEntity implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(nullable = false, length = 100)
    private String name;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 30)
    private PlanType type;

    @Column(name = "duration_days", nullable = false)
    private int durationDays;

    @Column(name = "session_count")
    private Integer sessionCount;

    @Column(name = "price_minor", nullable = false)
    private long priceMinor;

    @Column(name = "freeze_max_days", nullable = false)
    private int freezeMaxDays;

    @Column(name = "grace_days", nullable = false)
    private int graceDays;

    @Column(name = "time_restriction", length = 20)
    private String timeRestriction;

    @Column(name = "grants_access", nullable = false)
    private boolean grantsAccess = true;

    @Column(nullable = false)
    private boolean active = true;

    public boolean isSessionBased() {
        return sessionCount != null && sessionCount > 0;
    }

    public void deactivate() { this.active = false; }
    public void activate()   { this.active = true; }
}
```

---

## 2. `PlanType`

```java
public enum PlanType {
    MONTHLY,
    QUARTERLY,
    SEMI_ANNUAL,
    ANNUAL,
    SESSION_PACK,
    DAY_PASS,
    PT_PACKAGE,
    TIME_RESTRICTED
}
```

| النوع | `session_count` | `grants_access` الافتراضي |
|---|:---:|:---:|
| `MONTHLY` / `QUARTERLY` / `SEMI_ANNUAL` / `ANNUAL` | `null` | `true` |
| `SESSION_PACK` | مطلوب | `true` (قابل للتغيير) |
| `DAY_PASS` | `null` | `true` |
| `PT_PACKAGE` | مطلوب | **`false`** |
| `TIME_RESTRICTED` | `null` | `true` |

> **مفيش ربط صارم** بين النوع و`duration_days` (30/90/180/365). الجيم ممكن يعرّف 28 أو 45 يوم. الـ `duration_days` هو المصدر.

---

## 3. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-PLN-01 | `name` 2–100 وغير فاضي | DTO + DB |
| INV-PLN-02 | `name` فريد داخل الـ Org | Unique Index `(org_id, lower(name))` |
| INV-PLN-03 | `duration_days > 0` | DB CHECK + DTO |
| INV-PLN-04 | `price_minor > 0` | DB CHECK + DTO |
| INV-PLN-05 | `session_count > 0` لو موجود | DB CHECK + DTO |
| INV-PLN-06 | `freeze_max_days >= 0` | DB CHECK |
| INV-PLN-07 | `grace_days >= 0` | DB CHECK |
| INV-PLN-08 | `session_count` مطلوب لـ `SESSION_PACK`/`PT_PACKAGE` | Service |
| INV-PLN-09 | `grants_access = false` افتراضياً لـ `PT_PACKAGE` | Service |
| INV-PLN-10 | ممنوع حذف | No API + Grants |
| INV-PLN-11 | تعديل السعر لا يغير الـ Snapshots | Service + Test |
| INV-PLN-12 | تعطيل لا يؤثر على القائم | Service + Test |

---

## 4. قواعد الإنشاء

```text
1. orgId من TenantContext.
2. تحقق من الاسم الفريد (Case-insensitive).
3. تحقق من النوع والحقول:
   - SESSION_PACK / PT_PACKAGE → session_count إلزامي.
   - غيرهم → session_count = null (أو يترفض).
4. grantsAccess:
   - لو مبعوت → استخدمه.
   - لو مش مبعوت → PT_PACKAGE = false، غير كده = true.
5. priceMinor > 0.
6. durationDays > 0.
7. active = true.
8. Audit PLAN_CREATED.
```

### 4.1 أمثلة

**شهري:**
```json
{
  "name": "Monthly Gold",
  "type": "MONTHLY",
  "durationDays": 30,
  "priceMinor": 70000,
  "freezeMaxDays": 7,
  "graceDays": 3,
  "grantsAccess": true
}
```

**PT Package:**
```json
{
  "name": "PT 12 Sessions",
  "type": "PT_PACKAGE",
  "durationDays": 60,
  "sessionCount": 12,
  "priceMinor": 360000,
  "freezeMaxDays": 0,
  "graceDays": 0,
  "grantsAccess": false
}
```

---

## 5. قواعد التعديل

| الحقل | قابل للتعديل؟ | التأثير |
|---|:---:|---|
| `name` | ✅ | فحص التكرار |
| `type` | ✅ | تحقق من `session_count` |
| `durationDays` | ✅ | الاشتراكات القائمة لا تتأثر |
| `sessionCount` | ✅ | الاشتراكات القائمة لا تتأثر |
| `priceMinor` | ✅ | **الـ Snapshot لا يتأثر** |
| `freezeMaxDays` | ✅ | الاشتراكات القائمة تستخدم القيمة وقت التجميد؟ |
| `graceDays` | ✅ | الاشتراكات القائمة — القيمة وقت الانتهاء |
| `timeRestriction` | ✅ | Data only |
| `grantsAccess` | ✅ | الاشتراكات القائمة لا تتأثر |
| `active` | ✅ | من `/deactivate` / `/activate` |

> **قرار مهم:** `freezeMaxDays` و`graceDays` بيتقروا من الـ Plan **وقت العملية**. لو اتغيروا، الاشتراكات القائمة بتتأثر. هل ده مقصود؟
>
> **القرار المؤقت:** نعم — لأنها قواعد تشغيلية للباقة، مش سعر. لو الجيم قلّل الحد، الاشتراكات الجديدة بس تتأثر؟ الأفضل Snapshot لهم كمان.
>
> **للتبسيط في الـ MVP:** `price_minor` فقط هو الـ Snapshot الملزم. `freeze_max_days` و`grace_days` بيتقروا من الـ Plan الحالي. متسجل كـ Assumption في [decisions-and-assumptions](../../architecture/decisions-and-assumptions.md).

---

## 6. تعطيل / تفعيل

```text
POST /plans/{id}/deactivate
1. تحقق إن الباقة تبع الـ Org.
2. active = false.
3. Audit PLAN_DEACTIVATED.
4. الاشتراكات القائمة تكمل عادي.
5. ممنوع إنشاء اشتراك جديد على باقة معطلة.
```

- إعادة التفعيل مسموحة.
- ممنوع حذف الباقة نهائياً.

---

## 7. الـ Query Service

```java
public interface PlanQueryService {
    PlanSummary getPlan(UUID planId);
    PlanSummary getActivePlan(UUID planId);
    List<PlanSummary> listActivePlans(UUID orgId);
    PlanPriceSnapshot getPriceSnapshot(UUID planId);
}

public record PlanPriceSnapshot(
    UUID planId,
    String name,
    long priceMinor,
    int durationDays,
    Integer sessionCount,
    int freezeMaxDays,
    int graceDays,
    boolean grantsAccess
) {}
```

- `getActivePlan` بترمي `PlanInactiveException` لو `active = false`.
- `PlanPriceSnapshot` هو اللي `subscription` بتستخدمه وقت الإنشاء.

---

## 8. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `DuplicatePlanNameException` | 409 | اسم مكرر |
| `PlanNotFoundException` | 404 | مش موجود / Org تانية |
| `PlanInactiveException` | 422 | باقة معطلة |
| `InvalidPlanConfigurationException` | 422 | Session Count ناقص |
| `InvalidPriceException` | 400 | `price <= 0` |

---

## 9. الـ Repository

```java
public interface PlanRepository extends JpaRepository<Plan, UUID> {
    List<Plan> findAllByOrgIdAndActiveTrueOrderByNameAsc(UUID orgId);
    List<Plan> findAllByOrgIdOrderByNameAsc(UUID orgId);
    Optional<Plan> findByIdAndOrgId(UUID id, UUID orgId);
    boolean existsByOrgIdAndNameIgnoreCase(UUID orgId, String name);
    boolean existsByOrgIdAndNameIgnoreCaseAndIdNot(UUID orgId, String name, UUID id);
}
```

---

## 10. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [database-strategy](../../architecture/database-strategy.md)
