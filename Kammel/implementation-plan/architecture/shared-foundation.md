# الـ Shared Foundation — المكونات المشتركة

> **المرجع:** [`../../Kammel_Project_Documentation/03_SYSTEM_ARCHITECTURE.md`](../../Kammel_Project_Documentation/03_SYSTEM_ARCHITECTURE.md) · [`../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md`](../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md)  
> **الملفات المرتبطة:** [project-structure](./project-structure.md) · [security-strategy](./security-strategy.md) · [database-strategy](./database-strategy.md)

---

## 1. الفلسفة

`shared/` مش "كل حاجة مش عارفة تحطها فين". هو **بنية تحتية عابرة للـ Features** فقط.

### ✅ يدخل في `shared/`
- Base Entities.
- API Envelopes.
- Global Exception Handling.
- Security & Tenant Context.
- Integrations Events Contracts.
- Pure Utilities.
- Generic Validators.
- Framework Config.

### ❌ ممنوع يدخل في `shared/`
- أي Business Entity (`Member`, `Subscription`, `Payment`).
- أي Business Rule (`Freeze`, `Renewal`, `Risk`).
- أي Feature-specific DTO.
- أي Feature-specific Exception.
- أي Query خاص بـ Feature.

### اختبار سريع
> لو المكوّن ده بيتغير بسبب **قاعدة بيزنس** → مكانه Feature.  
> لو بيتغير بسبب **تقنية أو Framework** → مكانه `shared`.

---

## 2. `shared/domain`

### 2.1 `BaseEntity`
| السؤال | الإجابة |
|---|---|
| **إيه هو؟** | `@MappedSuperclass` فيه `UUID id`, `createdAt`, `updatedAt`. |
| **ليه؟** | توحيد الـ PK والتواريخ وتقليل التكرار. |
| **مسؤوليته؟** | Identity + Timestamps فقط. |
| **مين بيعتمد عليه؟** | كل الـ Entities ما عدا الـ Composite Key Entities. |
| **إمتى يتعمل؟** | Phase 1 — قبل أول Entity. |
| **مين مايملكوش؟** | أي Feature — هو Infrastructure. |

**ملاحظات:**
- `equals/hashCode` على `id` فقط، وبطريقة تتحمل `id = null`.
- ممنوع `@Data` على Entity.
- `createdAt` / `updatedAt` بـ `@CreationTimestamp` / `@UpdateTimestamp`.

### 2.2 `BaseAuditableEntity`
- بيضيف `createdBy` و`updatedBy` (UUID) للكيانات اللي محتاجة تتبع.
- **مش كل Entity محتاجة** — Calendar/Log Tables مش لازم.
- الـ Audit الحقيقي في Feature `audit`، مش هنا.

### 2.3 `TenantScoped`
```java
public interface TenantScoped {
    UUID getOrgId();
    void setOrgId(UUID orgId);
}
```
- **ليه؟** يخلي الـ Tenant Assignment واضح ويسهل الـ Arch Tests.
- **مين بيعتمد عليه؟** كل Entity فيها `org_id`.
- **مش بديل عن RLS** — هو Marker + Contract.

---

## 3. `shared/api`

### 3.1 `ApiResponse<T>`
| السؤال | الإجابة |
|---|---|
| **إيه هو؟** | Envelope موحد: `success`, `message`, `data`, `timestamp`. |
| **ليه؟** | Frontend يتعامل مع شكل واحد. |
| **مسؤوليته؟** | Presentation فقط. |
| **مين بيعتمد عليه؟** | كل الـ Controllers. |
| **إمتى؟** | Phase 1. |
| **مين مايملكوش؟** | أي Feature — هو Contract عام. |

```json
{
  "success": true,
  "message": "Member registered successfully",
  "data": { "id": "..." },
  "timestamp": "2026-11-03T10:15:00Z"
}
```

> **قاعدة:** `204 No Content` مش هتستخدم `ApiResponse`. باقي الردود كلها مغلفة.

### 3.2 `PageResponse<T>`
- `content`, `page`, `size`, `totalElements`, `totalPages`, `first`, `last`.
- **ليه؟** توحيد الـ Pagination.
- **مين بيعتمد عليه؟** كل List Endpoints.
- **ملاحظة:** ممنوع ترجيع `Page<T>` من Spring مباشرة.

### 3.3 `ErrorResponse`
- `timestamp`, `status`, `error`, `message`, `path`, `traceId`, `details[]`.
- `details` فيها `field`, `rejectedValue`, `message`.
- **ليه؟** Frontend يعرف يعرض الأخطاء بشكل منظم.

---

## 4. `shared/exception`

| المكوّن | الاستخدام | HTTP |
|---|---|---|
| `BusinessException` | Base لكل أخطاء البيزنس | حسب النوع |
| `ResourceNotFoundException` | مورد مش موجود | 404 |
| `DuplicateResourceException` | تكرار (Phone/Email) | 409 |
| `InvalidStateTransitionException` | State Machine | 422 |
| `TenantMismatchException` | مورد تابع لمستأجر تاني | **404** (مش 403) |
| `IdempotencyConflictException` | نفس الـ Key ببيانات مختلفة | 409 |
| `GlobalExceptionHandler` | `@RestControllerAdvice` | كل الأنواع |

### قواعد مهمة
- `TenantMismatchException` ترجع **404** مش 403 — عشان ما نكشفش وجود الـ Entity.
- ممنوع Stack Trace في Production Response.
- كل خطأ 500 بيتسجل بـ `traceId` + `orgId` + `userId` (بدون PII).
- `MethodArgumentNotValidException` بترجع كل الـ Field Errors مرة واحدة.

### إمتى يتعمل؟
Phase 1 — قبل أول Controller.

### مين مايملكوش؟
- `SubscriptionFrozenException` → Feature `subscription`.
- `PaymentAlreadyProcessedException` → Feature `payment`.
- `ImportValidationException` → Feature `member`.

---

## 5. `shared/security`

### 5.1 `TenantContext`
| السؤال | الإجابة |
|---|---|
| **إيه هو؟** | `ThreadLocal` فيه `orgId`, `branchId`, `userId`, `role`. |
| **ليه؟** | كل Query محتاجة الـ Tenant بدون تمريره في كل Method. |
| **مسؤوليته؟** | Request-scoped Context فقط. |
| **مين بيعتمد عليه؟** | كل الـ Features + Repositories + RLS. |
| **إمتى؟** | Phase 1. |
| **مين مايملكوش؟** | أي Feature. |

**قواعد:**
- `clear()` في `finally` دايماً.
- ممنوع الاعتماد عليه في `@Async` بدون نقل صريح للـ Context.
- الـ Background Jobs بتضبطه صريح قبل كل Tenant.

### 5.2 `JwtService`
- توليد/فك Access Token (`HS256` في MVP).
- Claims: `sub`, `org_id`, `branch_id`, `role`, `name`, `jti`.
- Access TTL: 15 دقيقة.
- Refresh TTL: 7 أيام، ويتخزن **Hash** في DB.
- الـ Secret من `JWT_SECRET` — ممنوع في الـ Git.

### 5.3 `JwtAuthenticationFilter`
- يقرأ `Authorization: Bearer`.
- يتحقق من التوقيع والـ Expiry.
- يضبط `SecurityContext` + `TenantContext`.
- يمسح الـ Context في `finally`.

### 5.4 `UserPrincipal`
- `userId`, `orgId`, `branchId`, `role`, `name`.
- implements `UserDetails`.

### 5.5 `BranchSecurity`
```java
@PreAuthorize("hasRole('GYM_OWNER') or @branchSecurity.canAccessBranch(#branchId)")
```
- `GYM_OWNER` → كل الفروع.
- غيره → الفرع المخصص فقط.
- **مهم:** التحقق لازم يحصل في الـ Service Layer كمان، مش Controller بس.

### 5.6 `TenantAwareJpaTransactionManager`
- بيمدد `JpaTransactionManager`.
- عند بداية كل Transaction بينفذ:
```sql
SET LOCAL app.current_org = '<orgId>';
```
- **ليه؟** RLS محتاجة القيمة على نفس الـ Connection.
- **ليه `SET LOCAL`؟** بتنتهي مع الـ Transaction، فمفيش Leakage في HikariCP.
- **بديل مرفوض:** `SET` عادي (بيفضل على الـ Connection).
- **بديل مرفوض:** `Hibernate @TenantId` لوحده (مش Defense in Depth).

### 5.7 `TenantContextCleanupFilter`
- `finally { TenantContext.clear(); }`
- بيمنع تسريب الـ Context بين الـ Requests.

### 5.8 `SecurityUtils`
- `getCurrentUserId()`, `getCurrentOrgId()`, `getCurrentRole()`.
- Throw `IllegalStateException` لو مفيش Context — مش ترجع null.

---

## 6. `shared/persistence`

### 6.1 `TenantTransactionRunner`
```java
public <T> T runInTenant(UUID orgId, Supplier<T> work) {
    TenantContext.setOrgId(orgId);
    try {
        return transactionTemplate.execute(status -> work.get());
    } finally {
        TenantContext.clear();
    }
}
```
- **ليه؟** كل Background Job لازم يشتغل في Tenant Scope.
- **مين بيعتمد عليه؟** Risk Job, Notification Job, Expiration Job, Reporting Jobs.
- **قاعدة:** أي Job بيلف على Organizations لازم يستخدمه.

### 6.2 `RlsSessionInitializer`
- يتأكد إن `app.current_org` متضبط قبل أول Query.
- بيتكامل مع `TenantAwareJpaTransactionManager`.

---

## 7. `shared/config`

| Config | المسؤولية | ملاحظات |
|---|---|---|
| `SecurityConfig` | Filter Chain + CORS + Method Security | Stateless، ممنوع CSRF للـ API |
| `JpaConfig` | `TenantAwareJpaTransactionManager` + Auditing | — |
| `SchedulingConfig` | `ThreadPoolTaskScheduler` + ShedLock | Timezone `Africa/Cairo` |
| `OpenApiConfig` | Swagger + JWT Scheme | `/swagger-ui` في Dev/Staging فقط |
| `JacksonConfig` | ISO-8601 UTC + `Long` as Number | ممنوع تحويل المبالغ لـ Double |
| `AsyncConfig` | Executor للـ Side Effects | مش للـ Financial Transactions |

---

## 8. `shared/validation`

### `@ValidPhone`
- Regex: `^(010|011|012|015)[0-9]{8}$`.
- بيطبع الرقم قبل التحقق (Trim + إزالة `+20`).
- **ليه؟** أرقام مصرية فقط + منع أخطاء الإدخال.
- **مين بيعتمد عليه؟** Member, User, Import.
- **مين مايملكوش؟** أي Feature — Validator عام.

> **ملاحظة:** الـ Validation دي Format فقط. التحقق من التكرار مسؤولية الـ Feature.

---

## 9. `shared/util`

| Utility | المسؤولية | قواعد |
|---|---|---|
| `MoneyUtil` | تحويل/تنسيق القروش | ممنوع `double` في الحساب |
| `PhoneUtil` | تطبيع `01xxxxxxxxx` و`201xxxxxxxxx` | يستخدم في `wa.me` |
| `DateUtil` | UTC ↔ `Africa/Cairo` | التخزين UTC دايماً |

### `MoneyUtil` — تحذير
الـ API بتاع `toMajor` ممكن يرجع `BigDecimal` مش `double`:
```java
public static BigDecimal toMajor(long minor) {
    return BigDecimal.valueOf(minor, 2);
}
```
> ممنوع `double` حتى في الـ Display Logic.

---

## 10. `shared/contracts/events`

| Event | Publisher | Listener | الاستخدام |
|---|---|---|---|
| `PaymentRecordedEvent` | `payment` | `subscription` | تفعيل الاشتراك عند اكتمال المبلغ |
| `SubscriptionActivatedEvent` | `subscription` | `risk`, `reporting` (اختياري) | تحديث الحالة |
| `SubscriptionRenewedEvent` | `subscription` | `notification` | Renewal Attribution |
| `SubscriptionExpiredEvent` | `subscription` | `risk` | تحديث Member Status |

### قواعد
- Events عبارة عن **Records Immutable**.
- ممنوع Entity تتحط جوه Event.
- كل Event فيه `orgId` + `occurredAt`.
- الأحداث الحرجة (تفعيل الاشتراك) تتناول **Synchronously** داخل نفس الـ Transaction.
- Side Effects (Attribution) تتناول بعد الـ Commit.

---

## 11. جدول الاعتماديات

| Shared Component | Features اللي بتعتمد عليه | إمتى يتعمل |
|---|---|---|
| `BaseEntity` | كل الـ Features | Phase 1 / Day 1 |
| `TenantContext` | كل الـ Features | Phase 1 / Day 1 |
| `ApiResponse` / `PageResponse` | كل الـ Controllers | Phase 1 |
| `GlobalExceptionHandler` | كل الـ Controllers | Phase 1 |
| `JwtService` / `JwtAuthenticationFilter` | `identity` + كل الـ Secured Endpoints | Phase 1 |
| `TenantAwareJpaTransactionManager` | كل الـ Tenant Queries | Phase 1 |
| `TenantTransactionRunner` | `risk`, `notification`, `subscription` (Jobs) | Phase 3 |
| `@ValidPhone` | `member`, `identity` | Phase 1 |
| `MoneyUtil` | `payment`, `plan`, `reporting` | Phase 2 |
| `PhoneUtil` | `member`, `notification` | Phase 2 |
| `DateUtil` | `subscription`, `checkin`, `reporting` | Phase 2 |
| `Integration Events` | `payment`, `subscription`, `notification` | Phase 2 |

---

## 12. أخطاء شائعة لازم نتجنبها

| الخطأ | النتيجة | الصح |
|---|---|---|
| `@Autowired` على Field | صعوبة Testing + Cycles | Constructor Injection |
| `@Data` على Entity | Lazy Loading Bugs | `@Getter/@Setter` فقط |
| `FetchType.EAGER` | N+1 / Memory | `LAZY` دايماً |
| `@Transactional` على Controller | Transaction أوسع من اللازم | على Service فقط |
| Log رقم موبايل | PII Leak | Log `memberId` فقط |
| `double` للمبلغ | أخطاء تقريب | `Long` قروش |
| `orgId` من Request Body | Tenant Injection | من `TenantContext` |
| `SET` بدل `SET LOCAL` | Connection Leakage | `SET LOCAL` |
| Entity في Response | تسريب بيانات | DTO |
| `shared` فيه Business Rule | كسر الحدود | Feature |

---

## 13. ترتيب تنفيذ الـ Shared Foundation

1. `pom.xml` + Spring Boot + Profiles.
2. `BaseEntity` + `TenantScoped`.
3. `ApiResponse` + `PageResponse` + `ErrorResponse`.
4. `BusinessException` + `GlobalExceptionHandler`.
5. `MoneyUtil` + `PhoneUtil` + `DateUtil`.
6. `@ValidPhone`.
7. `TenantContext` + `TenantContextCleanupFilter`.
8. `JwtService` + `UserPrincipal` + `JwtAuthenticationFilter`.
9. `TenantAwareJpaTransactionManager` + `RlsSessionInitializer`.
10. `SecurityConfig` + `OpenApiConfig` + `JacksonConfig`.
11. `TenantTransactionRunner`.
12. Integration Events Contracts (لما نحتاجها في Phase 2).

> **Checkpoint:** قبل ما تبدأ `identity`، لازم الـ Shared Foundation تكون مبنية ومختبرة.
