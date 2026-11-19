---
title: Core Features — Implemented vs Not Implemented
cssclasses: [rtl]
tags: [features, status, audit]
section: 3
---

# 3. Core Features: Implemented vs. Not Implemented
⬅️ [[02 - Business Problem]] | ➡️ [[04 - Technology Stack]]

## 🏷️ Status Legend

| Label | المعنى |
|---|---|
| ✅ **Implemented** | الكود موجود ومربوط بـ entry point وـ مفروض يشتغل بعد إصلاح الـ build/runtime blockers |
| 🟡 **Partially implemented** | الكود موجود بس ناقص أو unreachable أو فيه correctness gap معروف |
| 🧱 **Scaffolded** | الـ types أو الـ config أو الـ infrastructure موجودة لكن الـ feature مش قابل للاستخدام |
| ❌ **Not implemented** | مفيش implementation حقيقية |

> [!warning] مهم
> علشان المشروع **مش بيعمل compile**، مفيش business feature شغّال end-to-end. الجدول تحت بيوصف **اكتمال الكود** (*code completeness*) مش **جاهزية التشغيل** (*runtime readiness*).

---

## 📊 جدول الـ Capabilities

### 🗂️ Catalog

| Capability | Status | الدليل والملاحظات |
|---|---|---|
| **Category CRUD** | 🟡 | `CategoryController`, `CategoryServiceImpl`, `CategoryRepository`, `CategoryMapper`, DTOs. الـ API الوحيد المكشوف. **Hard delete**، مفيش tenant predicate، وجدول `categories` مش موجود في الـ `public` schema |
| **Category pagination** | ✅ (in code) | `GET /api/v1/categories?page=&size=` بيرجّع `PageResponse<CategoryResponse>`. مفيش size cap ولا sorting |
| **Product catalog** | 🟡 | `ProductServiceImpl` فيه create/update/find/list/delete، بس مفيش controller. الـ update فيه duplicate-check bug؛ الـ delete هو hard delete |
| **Product–category association** | 🟡 | `Product.category` هي `@ManyToOne`. الـ `ProductMapper.toResponse` بيعمل `category.getName()` من غير null guard، و `ProductResponse.categoryId` عمره ما بيتملى |
| **Product stock visibility** | ❌ | `ProductResponse` مفيهوش available quantity؛ الـ mapper فيه comment: `.availableQuantity() to be later implemented` |

### 🏢 Tenancy

| Capability | Status | الدليل والملاحظات |
|---|---|---|
| **Tenant registration** | ❌ | `TenantServiceImpl.registerTenant(...)` بيعمل validate للـ duplicates وبيبني `Tenant` لكن **مش بيحفظه أبدًا**، وكمان مش بيعمل provision للـ schema ولا بيعمل admin user |
| **Tenant approval / provisioning** | 🟡 | `approveTenant` بيحدد الـ status وبينادي `ProvisioningServiceImpl` اللي بيعمل schema وبيشغّل Flyway. الأخطاء **بتتبلع** والـ tenant بيرجع `PENDING` من غير ما الـ caller يعرف. تهيئة الـ default data عبارة عن method فاضية |
| **Tenant activation / suspension** | 🧱 | الـ methods موجودة، لكن `activateTenant` مش بيعمل provisioning وتحولات الـ state مش متسقة مع `approveTenant`. مفيش controller |
| **Tenant listing** | 🧱 | `PageResponse<TenantResponse> findAll(...)` موجودة، مفيش controller، ومش بتفلتر الـ soft-deleted tenants |

### 👤 Identity & Security

| Capability | Status | الدليل والملاحظات |
|---|---|---|
| **User management** | 🟡 | `UserServiceImpl` فيه create/update/delete/get/list/enable/disable لكن مفيش controller. الـ role validation غلط (شوف [[12 - Security Considerations]]). `UserResponse` بيكشف الـ **password hash** |
| **Admin user creation on tenant approval** | ❌ | `createAdminUserForTenant` موجودة لكنها `private` ومش بتتنادى أبدًا، وكمان مش بتعمل `enabled = true` |
| **JWT token generation / validation** | 🧱 | `JwtTokenService` بيولّد ويتحقق من RS256 tokens. الـ PEM files المتعملها commit حجمها **0 bytes** فالـ startup هيفشل. مفيش login endpoint |
| **Stateless security configuration** | ✅ (in code) | `SecurityConfig`: CSRF متقفّل، CORS مفعّل، Swagger و `/api/v1/auth/**` public، وأي حاجة تانية محتاجة authentication. مفيش authentication entry point ولا authorization rules |
| **JWT request authentication** | 🟡 | `JwtAuthenticationFilter` بيقرا الـ Bearer token وبيحدد `TenantContext` وبيحل الـ schema وبيملّى `SecurityContext`. **مش بيتحقق** من حالة الـ tenant ولا الـ user |

### 📦 Stock

| Capability | Status | الدليل والملاحظات |
|---|---|---|
| **Stock IN / Stock OUT** | ❌ | `StockMvtServiceImpl.create(...)` فاضية. `StockMvtController` مش موجود |
| **Stock transfers** | ❌ | مفيش `Warehouse` entity، ولا transfer model، ولا endpoint. `TypeMvt` فيه `IN` و `OUT` بس |
| **Stock adjustments** | ❌ | مفيش adjustment reason ولا approval ولا endpoint |
| **Inventory balances** | ❌ | مفيش balance table ولا balance query ولا projection |
| **Insufficient-stock prevention** | ❌ | مفيش check في أي مكان |
| **Concurrent stock safety** | ❌ | مفيش `@Version` ولا pessimistic lock ولا atomic conditional update |
| **Movement history / audit trail** | 🧱 | جدول `stock_mvts` و entity `StockMvt` موجودين لكن مفيش service logic ولا immutability، وعقد الـ service لسه بيكشف update/delete. مفيش secondary indexes ولا warehouse columns |

### 🧱 Cross-cutting

| Capability | Status | الدليل والملاحظات |
|---|---|---|
| **Multi-tenant isolation** | ❌ | الـ tenant context بيتحدد لكن عمره ما بيتستخدم في Hibernate routing أو filtering أو RLS. شوف [[06 - Multi-Tenancy Strategy]] |
| **Auditing (`created_by`, `updated_by`)** | 🟡 | `AbstractEntity` بيستخدم Spring Data auditing و `@EnableJpaAuditing`، لكن **مفيش `AuditorAware` bean**، فـ `created_by` بيفضل null بينما الـ tenant migration معرّفه `NOT NULL` |
| **Global exception handling** | 🟡 | بيتعامل مع `ResourceNotFoundException` و `DuplicateResourceException` وأخطاء bean-validation. الباقي (domain exceptions, `EntityNotFoundException`, `DuplicateKeyException`, malformed JSON, type mismatches) بيقع في **HTTP 500** |
| **Bean validation** | 🟡 | `CategoryRequest` بس فيه constraints حقيقية. `ProductRequest` و `StockMvtRequest` مفيهمش أي حاجة. `RegisterTenantRequest` ناقصه `@Email` وطول وpattern |
| **Common schema migration** | ❌ | `db/migration/common/V1__init_Common_tables.sql` **فاضي (0 bytes)**. جداول `tenants` و `users` مش هتتعمل في DB جديدة |
| **Tenant schema migration** | ✅ (in code) | `db/migration/tenant/V1__init_DB_For_Tenant.sql` بيعمل `categories` و `products` و `stock_mvts` |
| **API documentation** | 🧱 | springdoc موجود و paths الـ Swagger public. مفيش OpenAPI config ولا security scheme، والـ category endpoints بس هي اللي بتظهر |
| **Automated tests** | 🧱 | test واحد context-load. `./mvnw test` مش هيشتغل لأن الـ compile فاشل. مفيش unit / slice / integration / security / concurrency tests |
| **Observability** | ❌ | مفيش Actuator ولا metrics backend ولا tracing ولا structured logging ولا correlation IDs. `show-sql` مفعّل في الـ default profile |
| **Caching** | ❌ | مفيش cache provider ولا `@EnableCaching`. الـ `@Cacheable` على `TenantSchemaResolver` **ملهاش أي تأثير** حاليًا |
| **Background jobs / events** | ❌ | مفيش scheduler ولا queue ولا outbox ولا async executor |
| **AI capabilities** | ❌ | مفيش Spring AI dependency ولا model config ولا vector store ولا tools ولا prompts ولا AI endpoints |
| **Docker / deployment** | 🟡 | `docker-compose.yml` بيشغّل PostgreSQL بس. مفيش `Dockerfile` ولا production profile ولا CI pipeline |

---

## 3.1 إيه اللي بيشتغل end-to-end النهاردة؟

> [!failure] ولا حاجة.
> الـ build بيفشل قبل ما التطبيق يشتغل. وبعد إصلاح الـ build، لسه فيه blockers بتمنع أي request شغال:
> 1. `JwtTokenService` مش قادر يحمّل الـ PEM files الفاضية.
> 2. الـ common migration مبتعملش جداول `tenants` أو `users`.
> 3. الـ JPA بيكتب الـ business entities في الـ default `public` schema، وجداول `categories` و `products` و `stock_mvts` **مش موجودة** فيها.
> 4. حتى لو الجداول اتعملت، `created_by` هو `NOT NULL` ومفيش `AuditorAware` يديله قيمة.
> 5. مفيش **login endpoint**، فمستحيل تاخد JWT صالح من المنتج نفسه.

ده طبيعي في مشروع لسه ما خلصش، لكن لازم يتقال بوضوح عشان الـ README ما يبالغش في النضج.

## 3.2 أصول الكود اللي تستاهل الإبقاء عليها

> [!success] المشروع مش scaffold فاضي. فيه أساسات مفيدة:
> - **Package structure** منظّم وبطبقات متسقة.
> - **Request/Response DTOs** منفصلة عن الـ entities.
> - **Manual mappers** بتمنع كشف الـ entities مباشرة.
> - **Spring Data repository interfaces**.
> - **Paginated response wrapper**.
> - **Flyway-based schema versioning**.
> - فكرة **tenant-aware provisioning** بـ programmatic Flyway.
> - **Auditable base entity**.
> - **RS256 JWT infrastructure** (أقوى كـ default من shared HMAC secret).
> - هيكل **global exception handler**.
> - `open-in-view: false` و **Hikari pool** متظبط.
> - **Docker Compose** للـ PostgreSQL المحلي.
> - **Role enum** وتنفيذ `UserDetails`.

التوصيات اللي جاية بتخص **إكمال وتقوية** الأساس ده، مش استبداله.

---
⬅️ [[02 - Business Problem]] | ➡️ [[04 - Technology Stack]]
