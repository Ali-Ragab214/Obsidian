---
title: Multi-Tenancy Interview Guide — Stock SaaS
tags: [java, spring-boot, multi-tenancy, interview]
date: 2026-11-25
---

# Multi-Tenancy — Interview Prep

بص يا علي، ده شرح للـ **multi-tenancy** اللي موجود فعلًا في المشروع دلوقتي تحت package `ali.ragab.saas_multi_tennant_app`. أي حاجة مش متحققة في الكود مكتوبة بصراحة **⚠️ جزئي** أو **❌ لسه**؛ ما تدّعيش في الإنترفيو إنها شغالة لو مش موجودة.

## Table of contents

- [[#الصورة الكبيرة]]
- [[#1. الأول يعني إيه Multi-Tenancy؟]]
- [[#2. الـ public schema فيها إيه والـ tenant schema فيها إيه؟]]
- [[#3. الـ flow كامل من register لحد JWT عامل إزاي؟]]
- [[#4. الـ JWT بيشيل الـ tenant إزاي وليه بنعمل clear؟]]
- [[#5. Hibernate بيروح للـ schema الصح إزاي؟]]
- [[#6. ليه One connection pool مش pool لكل tenant؟]]
- [[#7. الـ Security مانع cross-tenant access إزاي؟]]
- [[#8. الـ auditing والـ soft delete ليهم علاقة إيه بالـ multi-tenancy؟]]
- [[#9. أختبر إن A مش شايف B إزاي؟]]
- [[#10. قابلتك مشكلة إيه في الـ multi-tenancy؟]]
- [[#11. أسئلة متوقعة من الـ interviewer]]
- [[#12. الصورة بصراحة]]
- [[#ملخص في 60 ثانية]]

---

## الصورة الكبيرة

الـ design هنا **Schema-per-tenant**: الـ platform metadata والـ users في `public`، وكل شركة active ليها schema باسم `tenant_<company_code>` فيه business tables. الـ login يطلع JWT فيه `tenant_id`، والـ `JwtAuthenticationFilter` يحل منه schema، وHibernate يضبط PostgreSQL **`search_path`** على نفس الـ schema قبل query.

```mermaid
flowchart LR
    A[POST /auth/register] --> B[public.tenants: PENDING]
    B --> C[Platform admin: approveTenant]
    C --> D[CREATE SCHEMA tenant_code]
    D --> E[Flyway tenant V1]
    E --> F[Enable admin + ACTIVE]
    F --> G[POST /auth/login]
    G --> H[JWT: sub, tenant_id, role]
    H --> I[JwtAuthenticationFilter]
    I --> J[TenantSchemaResolver: ACTIVE?]
    J --> K[TenantContext ThreadLocal]
    K --> L[Hibernate provider: SET search_path]
    L --> M[Query hits tenant_code.products/categories/stock_mvts]
```

> [!warning]
> الـ flow ده ماينفعش يتقال إنه end-to-end tested بين tenant A وB؛ الموجود tests للوحدات والـ connection provider، مش integration test كامل للـ API/business data.

---

## 1. الأول: يعني إيه Multi-Tenancy؟

ببساطة، تطبيق واحد بيخدم شركات كتير، بس لازم داتا كل شركة تبقى معزولة عن التانية. الـ **tenant** هنا هي الشركة المسجلة في `public.tenants`، واسم schema بيتولد من `company_code` في `TenantSchemaName.fromCompanyCode()`.

| Approach | الفكرة | العزل | التشغيل | مناسب هنا؟ |
|---|---|---|---|---|
| **database-per-tenant** | DB مستقل لكل شركة | عالي جدًا | backups/migrations/connections أكتر | مش متنفذ |
| **schema-per-tenant** | DB واحدة، schema لكل شركة | قوي على مستوى PostgreSQL namespace | migrations لكل schema | **ده المتنفذ** |
| shared schema + `tenant_id` | نفس الجداول وصفوف عليها tenant_id | بيعتمد جدًا على كل query/filter | أسهل أولًا، ومخاطر forget filter | مش مستخدم للـ business data |

ليه الـ **Schema-per-tenant** مناسب للـ implementation ده؟ `Product` و`Category` و`StockMvt` مش عليهم `tenant_id` في entities أو tenant migration؛ isolation جاي من schema routing، مش من `WHERE tenant_id = ...`. وفي نفس الوقت users/platform metadata متشاركة في `public` و`User.tenant` مربوط بـ `tenant_id`.

```java
// Product ملوش schema ثابت؛ Hibernate يختاره من الـ TenantContext.
@Entity
@Table(name = "products")
public class Product extends AbstractEntity { }
```

> [!tip]
> متقولش “اخترنا” كقرار موثق في README/spec؛ الأدق: “الكود الحالي مطبق Schema-per-tenant، وده باين من `TenantSchemaName` و`MultiTenantConnectionProviderImpl` ومفيش `tenant_id` في business tables.”

### جاوب في الإنترفيو كده

“الـ multi-tenancy عندي معناها شركة ما تشوفش داتا شركة تانية من نفس التطبيق. طبقت **Schema-per-tenant**: الـ platform data في `public` وكل tenant له schema مستقل. ده بيديني isolation أقوى من shared tables من غير تكلفة database مستقل لكل tenant. Hibernate بيحدد الـ schema من الـ JWT ويضبط `search_path` قبل الـ query.”

---

## 2. الـ public schema فيها إيه والـ tenant schema فيها إيه؟

الـ common migration `src/main/resources/db/migration/common/V1__init_Common_tables.sql` بيعمل `public.tenants` و`public.users`. الـ tenant migration `.../tenant/V1__init_DB_For_Tenant.sql` بيعمل `categories` و`products` و`stock_mvts` داخل كل tenant schema.

| مكان الداتا | Tables | Evidence |
|---|---|---|
| `public` | `tenants`, `users` | common `V1__init_Common_tables.sql`; `@Table(..., schema = "public")` في `Tenant` و`User` |
| `tenant_<code>` | `categories`, `products`, `stock_mvts` | tenant `V1__init_DB_For_Tenant.sql`; entities من غير `schema` |
| Flyway history | واحدة في `public` للـ common، وواحدة في كل tenant schema للـ tenant migrations | `application.yml` و`TenantMigrationServiceImpl.migrateSchema()` |

الـ Spring Flyway الأساسي شغال startup على `classpath:db/migration/common` وschema `public` (`spring.flyway.schemas: public`). أما `TenantMigrationServiceImpl.migrateSchema(schemaName)` فبيبني Flyway مستقل بـ `locations("classpath:db/migration/tenant")` و`schemas/defaultSchema(schemaName)`.

```yaml
# application.yml: الـ common migration فقط عند startup.
spring:
  flyway:
    locations:
      - classpath:db/migration/common
    schemas: public
```

```java
// كل tenant schema ليه Flyway execution منفصل.
Flyway.configure()
    .dataSource(this.dataSource)
    .schemas(schemaName)
    .defaultSchema(schemaName)
    .locations("classpath:db/migration/tenant")
    .load().migrate();
```

### جاوب في الإنترفيو كده

“فصلت الـ platform tables عن business tables. `public` فيه tenants وusers، أما كل شركة فلها categories/products/stock movements في schema مستقل. الـ common Flyway migration بتشتغل مرة على `public`، والـ tenant migration بتشتغل عند provisioning وكل startup للـ tenants الـ ACTIVE. كده أقدر أطور structure الـ business data بشكل موحد من غير خلط rows بين الشركات.”

---

## 3. الـ flow كامل من register لحد JWT عامل إزاي؟

1. `AuthenticationController.register()` ينادي `TenantServiceImpl.registerTenant()`.
2. الـ service يتحقق من uniqueness، يعمل `Tenant` بـ **`PENDING`**، ويعمل admin user disabled في `public.users` (`createAdminUserForTenant`).
3. Platform admin ينادي `TenantController.approve()` → `TenantServiceImpl.approveTenant()`.
4. `ProvisioningServiceImpl.provisionTenant()` يعمل schema آمن، يشغل tenant Flyway، ولو فشل schema جديد يتعمله `DROP ... CASCADE`.
5. بعد provisioning الناجح، admin يتعمله `enabled=true` والـ tenant يبقى **`ACTIVE`**.
6. `AuthenticationServiceImpl.login()` يستخدم `AuthenticationManager`، ثم `JwtTokenService.generateAccessToken()` يوقع JWT بـ RSA.

```java
// approveTenant: provisioning قبل enable/admin ACTIVE.
this.provisioningService.provisionTenant(tenant);
adminUser.setEnabled(true);
this.userRepository.save(adminUser);
tenant.setStatus(TenantStatus.ACTIVE);
this.repository.save(tenant);
```

> [!success]
> مشكلة “tenant schema لسه مش موجود” متعالجة للـ onboarding: إنشاء schema + tenant Flyway بقوا في `ProvisioningServiceImpl` وقت approve، مش انتظار startup فقط.

> [!warning]
> `activateTenant()` في نفس service يغيّر `PENDING → ACTIVE` مباشرة **من غير provisioning**. الـ endpoint موجود في `TenantController.activate()`؛ لو اتستخدم بدل `/approve` ممكن tenant يبقى ACTIVE من غير business schema، وساعتها queries ممكن تفشل بـ relation does not exist. دي ثغرة OPEN.

### جاوب في الإنترفيو كده

“الـ register مش بيفتح tenant فورًا؛ بيخزنه PENDING ويعمل admin disabled. الـ platform admin لما يعمل approve بننشئ schema باسم مشتق من company code، نشغل Flyway عليها، وبعد النجاح بس نفعّل admin ونحوّل tenant لـ ACTIVE. وقت login بنطلع RSA-signed JWT فيه user وtenant وrole، وبعده كل request يتوجه للـ schema الصح.”

---

## 4. الـ JWT بيشيل الـ tenant إزاي وليه بنعمل clear؟

`JwtTokenService.generateAccessToken()` يحط `tenant_id` و`role` و`subject=userId`، والـ expiration الافتراضي في `application.yml` هو `86400000` ms (24 ساعة). `JwtAuthenticationFilter.doFilterInternal()` يـ validate token، يقرا claims، يعمل `TenantSchemaResolver.resolveTenantSchema(tenantId)`، ويحط ID/schema في `TenantContext`.

```java
// الـ claims اللي بتربط request بالـ tenant.
.subject(userId)
.claim("tenant_id", tenantId)
.claim("role", role)
.expiration(expiration)
```

```java
// الـ filter يحط context ثم يشيله مهما حصل exception downstream.
TenantContext.setCurrentTenant(tenantId);
final String schemaName = this.tenantSchemaResolver.resolveTenantSchema(tenantId);
TenantContext.setCurrentSchema(schemaName);
// ... filterChain.doFilter(...)
finally {
    TenantContext.clearCurrentTenant();
}
```

`TenantContext` مخزن بـ **ThreadLocal**. Servlet thread ممكن يتعاد استخدامه لrequest تاني؛ لو ماعملناش `remove()` في `finally`، request شركة B ممكن يرث tenant/schema بتوع A. `clearCurrentTenant()` فعليًا يمسح الاتنين `CURRENT_TENANT` و`CURRENT_SCHEMA`.

> [!warning]
> الـ filter بيلف أي exception في authentication ويكمل chain بعد log (`catch (Exception e)`). فحالة tenant غير active/unknown غالبًا تتحول لrequest من غير Authentication ثم Spring Security يرفضها؛ مش response error واضح مخصوص للـ tenant.

### جاوب في الإنترفيو كده

“الـ JWT عندي شايل `tenant_id` كclaim موقّع مع userId والـ role. في `JwtAuthenticationFilter` بحل الـ tenant للـ schema وأخزنهم في ThreadLocal عشان Hibernate يقرأهم في نفس request. بعمل clear في `finally` لأن servlet thread بيتعاد استخدامه؛ من غيره ممكن يحصل tenant context leak. الـ token signed بـ RSA ومدته الافتراضية 24 ساعة.”

---

## 5. Hibernate بيروح للـ schema الصح إزاي؟

في 3 أجزاء متصلة:

| الجزء | مسؤوليته في المشروع |
|---|---|
| `TenantContext` | يحمل tenant ID وresolved schema للـ current thread |
| `CurrentTenantIdentifierResolverImpl` | Hibernate يسأله “أنا على tenant إيه؟”؛ يرجع schema أو يرمي `TenantResolutionException` للـ authenticated tenant user لو context ناقص |
| `MultiTenantConnectionProviderImpl` | ياخد connection من نفس `DataSource`، يعمل `SET search_path TO "tenant_code", public`، ويرجعه بعد reset |

```java
// Hibernate يرفض business request لو tenant ID موجود بس schema مش resolved.
if (tenantId != null && !tenantId.isBlank()) {
    throw new TenantResolutionException(
        "Tenant context has no resolved schema for tenant: " + tenantId);
}
```

```java
// PostgreSQL يقرأ unqualified tables من tenant schema الأول.
statement.execute("SET search_path TO "
    + TenantSchemaName.quote(tenantIdentifier) + ", public");
```

الـ resolver يسمح بـ `public` فقط لـ anonymous/no auth أو authority `ROLE_PLATFORM_ADMIN`; والـ platform entities أصلًا عليها schema صريح (`Tenant` و`User`). الكود الحالي ما فيهوش method اسمها **`runAsPlatform`**؛ متقولهاش كأنها implementation.

> [!tip]
> الـ `TenantSchemaName.fromCompanyCode()` يقبل فقط `[a-z0-9_]{1,50}` ويضيف `tenant_`. والـ `quote()` بيعمل escape للـ identifier. ده مهم لأن `SET search_path` و`CREATE SCHEMA` identifier مش parameter عادي في JDBC.

### جاوب في الإنترفيو كده

“Hibernate مش شايف `tenant_id` في business entities؛ هو بياخد tenant identifier من `CurrentTenantIdentifierResolverImpl`. بعد كده الـ connection provider يضبط PostgreSQL `search_path` للـ schema بتاع الشركة، فـ `select from products` يروح لproducts بتوعها. لو user tenant-authenticated ومفيش schema resolved، resolver بيرمي exception بدل default صامت لـ public.”

---

## 6. ليه One connection pool مش pool لكل tenant؟

المشروع معرف **Hikari** DataSource واحد في `application.yml` (`maximum-pool-size: 10`) و`MultiTenantConnectionProviderImpl` بياخد منه connection لكل tenant. مفيش DataSource/Hikari pool لكل tenant.

ده عملي لما tenants يزيدوا: pool لكل tenant هيستهلك connections وmemory وconfiguration بشكل كبير حتى لو tenant idle. المقابل الخطير: الـ physical connection ممكن يفضل جواه **`search_path`** من tenant سابق.

```java
// قبل رجوع connection للـ pool لازم نرجعه public.
statement.execute("SET search_path TO public");
if (reset) {
    connection.close();
    return;
}
abortQuietly(connection);
```

لو set للـ search_path فشل عند acquire، `getConnection()` يقفل leased connection. ولو reset فشل عند release، بيعمل `connection.abort(...)` بدل ما يرجع connection dirty للـ pool. `MultiTenantConnectionProviderImplTest` يغطي الاتنين، وكمان يثبت إن نفس physical connection رجع `public` قبل ما يستخدمه tenant تاني.

### جاوب في الإنترفيو كده

“استخدمت shared Hikari pool مش pool لكل tenant لتفادي connection explosion مع عدد شركات كبير. العزل بيتعمل per borrowed connection عن طريق `SET search_path`. أهم trade-off إن لازم reset حتمي قبل الرجوع للـ pool؛ عندي reset لـ public، ولو فشل بعمل abort للـ connection. ده عليه test بيقيس `current_schema()` وإعادة استخدام نفس backend PID.”

---

## 7. الـ Security مانع cross-tenant access إزاي؟

الحماية طبقات، مش طبقة واحدة:

1. **JWT signature**: `JwtTokenService.validateToken()` يتحقق بالـ RSA public key.
2. **RBAC**: `SecurityConfig.securityFilterChain()` يقيد endpoints، و`TenantController` محتاج `ROLE_PLATFORM_ADMIN`.
3. **Tenant status**: `TenantSchemaResolver.resolveTenantSchema()` يقرأ `public.tenants` ويرفض non-`ACTIVE` (`TenantNotActiveException`).
4. **Schema isolation**: business ORM queries تتوجه للـ tenant schema بـ `search_path`.
5. **User ownership في public**: `UserServiceImpl.loadOwned()` يستخدم `findByIdAndTenantIdAndDeletedFalse()`.

```java
// extra guard للـ users، غير schema isolation بتاعة business tables.
return this.repository.findByIdAndTenantIdAndDeletedFalse(userId, tenantId)
    .orElseThrow(() -> new EntityNotFoundException("User not found"));
```

### إيه اللي يحصل لو tenant اتعمله suspend؟

للـ **login جديد**: `UserServiceImpl.loadUserByUsername()` يرفض user المرتبط بـ tenant مش `ACTIVE`.

للـ **token قائم**: القصد في `TenantSchemaResolver` إنه يرفض status مش ACTIVE في أول resolve. لكن فيه `@Cacheable("tenantSchemas")` بـ `ConcurrentMapCacheManager` من غير TTL، ومفيش `@CacheEvict` في `suspendTenant()` أو lifecycle methods. لذلك لو schema اتcache وهو ACTIVE، JWT قائم ممكن يفضل يعدي حتى بعد suspend، مش بس لحد expiration (24 ساعة افتراضيًا) بل لحد cache eviction أو app restart. دي **OPEN security gap**، والـ `TenantSchemaResolverCacheTest` نفسه يثبت إن status change مايتشافش قبل eviction.

> [!warning]
> ما تقولش “suspend يبطل كل الـ tokens فورًا”. ده غير صحيح في الكود الحالي. JWT كمان stateless ومفيش denylist/revocation mechanism.

### جاوب في الإنترفيو كده

“العزل مبني على JWT signed + RBAC + schema routing، ومع users في public بعمل ownership query على tenant_id. الـ resolver بيعمل fail-closed لو tenant unknown أو non-active. لكن عندي limitation صريحة: schema resolution cache ملوش eviction عند suspend، فـ token قائم ممكن يفضل يسمح بالـ requests لحد cache eviction/restart؛ الحل إني أعمل evict عند lifecycle change أو أعمل status check/short TTL وtoken revocation.”

---

## 8. الـ auditing والـ soft delete ليهم علاقة إيه بالـ multi-tenancy؟

`AbstractEntity` (للـ business entities و`User`) فيه `created_at`, `updated_at`, **`created_by`**, `updated_by`, `deleted`. `Application` مفعل `@EnableJpaAuditing`، و`JpaAuditingConfig.AuditorAwareImpl` يجيب principal من `SecurityContextHolder` أو `SYSTEM` لو مفيش Authentication.

```java
// الـ auditor يكتب user ID الموجود في JWT filter كـ principal.
return Optional.of(principal.toString());
```

في `Product`/`Category`/`StockMvt`، الـ audit row موجود جوه schema tenant الصح لأن Hibernate connection اتوجهت له. ده بيدي audit per tenant طبيعيًا. `Tenant` له auditing fields مشابهة لكنه في `public`.

الـ soft delete متنفذ بـ `@SQLDelete` و`@SQLRestriction("deleted = false")` في entities زي `Product` و`User` و`Tenant`. يعني delete يعلّم row بدل physical delete، وORM queries العادية تستبعده.

> [!warning]
> الـ soft delete مش permission boundary ولا بديل عن schema isolation. وكمان raw/native SQL لازم يضيف `deleted = false` بنفسه لو محتاج نفس semantics؛ مثال `ProductRepository.detachFromCategory()` native update لا يعتمد على `@SQLRestriction`.

### جاوب في الإنترفيو كده

“الـ auditing مربوط بالـ authenticated principal، فـ `created_by` و`updated_by` بيتسجلوا مع كل business row داخل schema الشركة نفسها. فعلت JPA auditing في `Application` واستخدمت `AuditorAware`. الـ soft delete بـ `@SQLDelete` و`@SQLRestriction` بيحافظ على history ويخفي deleted rows من ORM queries، لكنه مش آلية عزل tenants.”

---

## 9. أختبر إن A مش شايف B إزاي؟

### الموجود فعلًا في tests

| Test | اللي بيثبته |
|---|---|
| `MultiTenantConnectionProviderImplTest.resetsSearchPathAndReusesTheSamePhysicalConnection` | schema A ثم release → `public` ثم schema B على نفس backend connection |
| `MultiTenantConnectionProviderImplTest.abortsConnectionWhenSearchPathResetFails` | dirty connection مايرجعش للـ pool |
| `TenantSchemaResolverTest` | unknown/deleted/non-ACTIVE tenant يرمي exception؛ active يتحول لـ `tenant_acme` |
| `CurrentTenantIdentifierResolverImplTest` | authenticated tenant user بدون context يترفض؛ platform context يرجع `public` |
| `UserServiceImplTest.*ForUserFromAnotherTenant` | get/update/delete/enable/disable user من tenant تاني يرجع not found عبر scoped repository query |
| `TenantSchemaResolverCacheTest` | يثبت cache behavior، ومنها مشكلة status stale قبل eviction |

### اللي مش متغطي

- مفيش integration test بيعمل tenant A وB حقيقيين، يكتب `Product` في A، ثم يطلب list/get/update/delete من JWT بتاع B ويتأكد إنه مش شايفه.
- مفيش MockMvc/API test للـ `JwtAuthenticationFilter` ولا اختبار full login → request → Hibernate query.
- مفيش test لـ suspend مع cached schema يثبت الحظر أو يثبت الحل، ومفيش test provisioning/Flyway end-to-end.
- مفيش test للـ raw/native SQL عبر tenant boundary.

> [!question]
> أفضل test ناقص: اعمل A وB، provision الاتنين، اكتب record بنفس ID/reference في كل schema، ثم JWT-A/JWT-B. كل list/get/update/delete لازم يرى schema بتاعه فقط، وبعد كل request افحص `current_schema()` أو SQL logs.

### جاوب في الإنترفيو كده

“الموجود حاليًا unit tests قوية للـ resolver والـ connection reset، وكمان user ownership tests. لكن ما ادعيش إن عندي full cross-tenant integration suite للـ products؛ دي فجوة واضحة. الاختبار اللي هزوده يعمل tenant A وB فعليًا، ويثبت إن JWT-B ما يقدرش يقرأ أو يعدل record A من خلال API.”

---

## 10. قابلتك مشكلة إيه في الـ multi-tenancy؟

### 10.1 المشكلة: الـ schema كانت مش موجودة لحد startup migration

- **المشكلة:** tenant جديد لو بقى usable قبل إنشاء schema/tables، queries على business tables هتطلع relation does not exist.
- **السبب (root cause):** startup migration لوحده مش provisioning للـ tenant الجديد.
- **الحل:** **FIXED جزئيًا.** `ProvisioningServiceImpl.provisionTenant()` يعمل `createSchema()` ثم `TenantMigrationService.migrateSchema()`، و`approveTenant()` يستدعيه قبل `ACTIVE`.
- **اللي اتعلمته:** onboarding لازم يبقى lifecycle atomic قدر الإمكان: schema + migrations قبل expose tenant. **OPEN:** `TenantServiceImpl.activateTenant()` ما زال ممكن ينقل PENDING لـ ACTIVE بلا provisioning.

### 10.2 المشكلة: الـ resolver كان fail-open ويرجع `public`

- **المشكلة:** النسخة السابقة من `CurrentTenantIdentifierResolverImpl` كانت ترجع `public` لما schema null، و`TenantSchemaResolver` كان يرجع `public` للـ unknown/error؛ ده ممكن يوجّه business request لـ public بدل ما يفشل.
- **السبب (root cause):** fallback عام لـ public بدل distinction بين platform operation وtenant business operation.
- **الحل:** **FIXED في الـ workspace الحالي، غير committed.** resolver يرمي `TenantResolutionException` لو authenticated tenant user من غير schema، و`TenantSchemaResolver` يرمي للـ unknown/deleted/non-ACTIVE. الـ platform admin/anonymous فقط مسموح لهم بـ public.
- **اللي اتعلمته:** **fail-closed** مع tenant business data أهم من convenience. مفيش `runAsPlatform` method في المشروع؛ الفصل الحالي معتمد على security context و`ROLE_PLATFORM_ADMIN`.

### 10.3 المشكلة: تسريب `search_path` من الـ connection pool

- **المشكلة:** connection اتأجر لـ tenant A لو رجع pool وهو لسه `search_path=tenant_a`، tenant B ممكن يقرأ غلط.
- **السبب (root cause):** `search_path` session state في PostgreSQL، والـ pool بيعيد استخدام physical connection.
- **الحل:** **FIXED في الـ workspace الحالي، غير committed.** `releaseConnection()` يعمل `SET search_path TO public` قبل `close()`؛ لو reset فشل يعمل `abort`. لو set عند acquire فشل يقفل connection.
- **اللي اتعلمته:** pooled connection state لازم يتنضف أو يتعدم، ومش كفاية “هنعمل reset غالبًا”. الـ tests متخصصة في السيناريو ده.

### 10.4 المشكلة: `JdbcTemplate` / raw SQL مش بيمر تلقائيًا على Hibernate tenancy

- **المشكلة:** `JdbcTemplate` مش هياخد `CurrentTenantIdentifierResolverImpl` أو `MultiTenantConnectionProviderImpl` كـ Hibernate routing.
- **السبب (root cause):** Hibernate multi-tenancy mechanism خاص بـ ORM connections؛ raw SQL عنده connection عادي.
- **الحل:** **FIXED للـ uses الموجودة، لكن قاعدة مستقبلية.** `TenantSchemaResolver` يقرأ `public.tenants` صراحة، و`ProvisioningServiceImpl` يبني schema name فقط عبر `TenantSchemaName.fromCompanyCode()` ثم `quote()` قبل `CREATE/DROP SCHEMA`. مفيش raw SQL business query بـ `JdbcTemplate` حاليًا.
- **اللي اتعلمته:** schema identifier مش يتبني من request string مباشرة؛ لازم validation/quoting. وأي raw business SQL جديد لازم يثبت schema بشكل آمن أو يمر عبر tenant-aware connection.

### 10.5 المشكلة: token بعد suspend

- **المشكلة:** المفروض tenant suspended ما يعملش access.
- **السبب (root cause):** `TenantSchemaResolver` cached بـ `@Cacheable` في in-memory `ConcurrentMapCacheManager` من غير TTL، و`TenantServiceImpl.suspendTenant()` ما بيعملش eviction. JWT كمان stateless ولا فيه revocation.
- **الحل:** **OPEN.** الكود يرفض login جديد، والـ resolver يرفض suspended tenant فقط عند cache miss. لكن cached active schema يخلي token قائم يعدي لحد cache eviction/app restart، لا مجرد 24 ساعة token expiration.
- **اللي اتعلمته:** caching authorization-adjacent state محتاج invalidation صريح عند lifecycle mutation، وsession/token revocation policy.

### 10.6 المشكلة: username/email globally unique

- **المشكلة:** user في tenant A وuser مختلف في B ما يقدروش يشاركوا نفس username/email.
- **السبب (root cause):** `common/V1__init_Common_tables.sql` عامل `ux_users_username_active` و`ux_users_email_active` من غير `tenant_id`، و`UserServiceImpl.existsByUsername/existsByEmail` global.
- **الحل:** **OPEN/قرار design حالي.** ده قد يكون مقصود عشان login request فيه username/password فقط ومفيش tenant identifier، لكن مش tenant-scoped identity. عكس `(tenant_id, username)` هيتطلب login يعرف tenant code/domain أو tenant chooser.
- **اللي اتعلمته:** identity model لازم يتفق مع login UX؛ schema isolation لا يحل global users table constraints وحده.

### 10.7 المشكلة: Swagger ماكانش عنده Authorize

- **المشكلة:** Swagger UI محتاج يعرف bearer auth scheme عشان يبعث JWT بسهولة.
- **السبب (root cause):** من غير OpenAPI security scheme، UI مش بيظهر Authorize بالصيغة الصح.
- **الحل:** **FIXED.** `OpenApiConfig` فيه `@SecurityScheme(name = "bearerAuth", type = HTTP, scheme = "bearer")` و`@OpenAPIDefinition(security = @SecurityRequirement(name = "bearerAuth"))`.
- **اللي اتعلمته:** documentation/config جزء من قابلية اختبار security، مش business feature وحده.

### 10.8 المشكلة: تعديل Flyway V1 in place

- **المشكلة:** git history يبيّن إن `dfe86cf` عدل `common/V1__init_Common_tables.sql` و`tenant/V1__init_DB_For_Tenant.sql` بعد ما V1 اتضاف في `7b68d5a` / `9c6fd04`.
- **السبب (root cause):** تعديل migration applied يغيّر checksum؛ مع `validate-on-migrate: true` ممكن existing databases تفشل validation.
- **الحل:** **OPEN/غير متأكد منه في DB حالي** — ماقدرناش نتحقق هل في بيئات اتطبق عليها الـ V1 القديم. الصح بعد كده يكون migration جديدة V2 بدل تعديل V1.
- **اللي اتعلمته:** Flyway migrations immutable بعد deployment؛ git history مهم في السؤال ده.

---

## 11. أسئلة متوقعة من الـ interviewer

| سؤال | Model answer قصير | في مشروعنا؟ |
|---|---|---|
| ليه مش database-per-tenant؟ | بيعزل أكتر لكن connection/migration/ops لكل DB أعلى. هنا shared DB + schema يوازن العزل والتشغيل. | **جزئي:** rationale inference، implementation schema-per-tenant فقط |
| ليه مش shared schema + `tenant_id`؟ | business entities هنا مفيهاش tenant_id؛ الـ schema نفسه boundary. shared tables محتاجة discipline/filter في كل query. | **✅ متنفذ كاختيار كود** |
| Hibernate يعرف tenant منين؟ | JWT filter يحط ID+schema في `TenantContext`، والـ resolver يرجع schema. | **✅** |
| إزاي تمنع SQL injection في schema name؟ | validate company code في `TenantSchemaName.fromCompanyCode` ثم quote identifier؛ ما تعملش concatenation من user input. | **✅ للـ provisioning/search_path** |
| إزاي تعمل migration لـ 1000 tenant؟ | batch/queue، observability، retries، version dashboard، وعزل failure per tenant بدل blocking startup. | **⚠️ جزئي:** `migrateAllTenants()` loop sequential ويlog failure؛ مفيش queue/retry/dashboard |
| لو Flyway فشل لtenant واحد؟ | provisioning schema الجديد بيتعمله drop ثم exception؛ startup loop يكمل tenants التانيين بعد log. لازم alert/retry policy production. | **✅ جزئي** |
| إزاي suspend يبطل sessions فورًا؟ | evict schema cache + status check لكل request أو short TTL، JWT denylist/token version، وربما user/session disable. | **❌ لسه:** الكود الحالي عنده stale cache gap |
| إزاي تتعامل مع `@Async` وThreadLocal؟ | ThreadLocal لا ينتقل تلقائيًا؛ مرر tenant explicitly أو TaskDecorator ينسخ/يمسح context، والأفضل explicit context. | **❌ نظري:** مفيش `@Async` في المشروع |
| tenant onboarding من غير downtime؟ | provision async/queue، migrations idempotent، tenant يفضل PENDING لحد health check ثم ACTIVE. | **⚠️ جزئي:** PENDING→approve موجود، لكنه synchronous ومفيش health check |
| limits بتاعة Schema-per-tenant؟ | schemas/migrations/catalog management يزيدوا؛ cross-tenant reporting أصعب، schema count كبير يضغط operations. | **❌ نظري** |
| تعمل cross-tenant reporting إزاي؟ | platform-only service يعمل controlled qualified queries/warehouse/read model، مش tenant request عادي. | **❌ نظري** |
| إزاي تثبت isolation؟ | integration test بtenant A/B + JWTs والـ CRUD، وفحص search_path/reset تحت pool reuse. | **⚠️ جزئي:** provider/unit ownership tests فقط |

---

## 12. الصورة بصراحة

| Feature | الحالة | Evidence |
|---|---|---|
| Schema-per-tenant routing للـ business ORM | ✅ متنفذ | `CurrentTenantIdentifierResolverImpl`, `MultiTenantConnectionProviderImpl`, entities `Product/Category/StockMvt` |
| Platform data في `public` | ✅ متنفذ | `Tenant`, `User`, common `V1__init_Common_tables.sql` |
| Tenant provisioning عند approve | ✅ متنفذ | `TenantServiceImpl.approveTenant()`, `ProvisioningServiceImpl.provisionTenant()` |
| Common وtenant Flyway separation | ✅ متنفذ | `application.yml`, `TenantMigrationServiceImpl` |
| Startup migration للـ ACTIVE tenants | ✅ متنفذ | `TenantSchemaMigrationRunner`, `migrateAllTenants()` |
| Fail-closed tenant resolution | ✅ متنفذ في الـ workspace الحالي | `TenantSchemaResolver`, `CurrentTenantIdentifierResolverImpl`؛ تغييرات local غير committed |
| Safe reset/abort للـ pooled `search_path` | ✅ متنفذ في الـ workspace الحالي | `MultiTenantConnectionProviderImpl`, corresponding tests؛ تغييرات local غير committed |
| Role authorization | ✅ متنفذ | `SecurityConfig.securityFilterChain()` |
| User cross-tenant ownership guard | ✅ متنفذ | `UserServiceImpl.loadOwned()`, `UserRepository.findByIdAndTenantIdAndDeletedFalse()` |
| Business A/B integration isolation test | ❌ لسه | لا يوجد test يكتب/يقرأ Product عبر tenantين حقيقيين |
| Suspend يبطل access فورًا | ❌ لسه | `@Cacheable` بلا eviction في `TenantSchemaResolver`; `suspendTenant()` بلا eviction |
| Token revocation/denylist | ❌ لسه | `JwtTokenService` JWT stateless فقط |
| Safe PENDING→ACTIVE في كل endpoint | ⚠️ جزئي | `/approve` provision صح؛ `activateTenant()` لا provision |
| Global vs per-tenant usernames | ⚠️ قرار design حالي | global unique indexes في common migration؛ login بلا tenant selector |
| Async tenant propagation | ❌ لسه | لا يوجد `@Async`/TaskDecorator في source |
| Flyway V1 immutability | ⚠️ مخاطرة history | `dfe86cf` عدل ملفات V1؛ حالة DB deployed **غير متأكد منه** |

---

## ملخص في 60 ثانية

“أنا عامل الـ **multi-tenancy** بـ **Schema-per-tenant** على PostgreSQL. الـ `public` schema فيه tenants وusers، وكل شركة active ليها schema `tenant_<company_code>` فيه الـ business tables. عند الـ login، الـ JWT بيشيل `tenant_id` وrole، والـ `JwtAuthenticationFilter` يحل schema ويحطه في `ThreadLocal`. Hibernate يقرأه من `CurrentTenantIdentifierResolverImpl`، والـ connection provider يعمل `SET search_path` قبل الـ query ويرجعه `public` قبل pool reuse. الـ provisioning بيعمل schema وFlyway قبل ما tenant يبقى ACTIVE. عندي tests للـ resolver والـ connection reset وuser ownership، لكن لسه ناقص full A/B integration test. وأهم limitation صريحة: suspend مع cached schema مش بيبطل tokens القائمة فورًا، وده محتاج eviction/revocation.”

