---
title: Security Considerations
cssclasses: [rtl]
tags: [security, findings, threat-model, p0, p1, p2]
section: 12
---

# 12. Security Considerations
⬅️ [[11 - AI Capabilities]] | ➡️ [[13 - Performance and Scalability]]

## 12.1 ✅ الضوابط المنفَّذة

| الضابط | التنفيذ | التقييم |
|---|---|---|
| **Password hashing** | BCrypt عبر `BCryptPasswordEncoder` | ✅ Baseline كويس |
| **Token signing** | RS256 بـ private/public key pair | 🟡 اختيار ممتاز؛ **التعامل مع المفاتيح مكسور** |
| **Stateless sessions** | `SessionCreationPolicy.STATELESS` | ✅ مناسب لـ JWT APIs |
| **CSRF** | متقفّل | ✅ مناسب لـ stateless bearer-token API |
| **Authentication requirement** | `anyRequest().authenticated()` | ✅ Default كويس، لكن مفيش endpoints غير categories |
| **Method security** | `@EnableMethodSecurity` | 🟡 مفعّلة **لكن مش مستخدمة** |
| **CORS** | متظبط | 🔴 Wildcard origin؛ **مش production-safe** |
| **Input validation** | Bean Validation على بعض DTOs | 🟡 جزئي وغير متسق |
| **SQL access** | Spring Data derived queries و `JdbcTemplate` parameterised | ✅ مفيش concatenation خام **ما عدا schema DDL** |
| **Error handling** | Generic 500 للأخطاء غير المتوقعة | 🟡 مابيسرّبش stack traces للـ clients، لكنه بيحوّل أخطاء متوقعة كتير لـ 500 |

## 12.2 🔎 الـ Findings

> [!info] الـ Severity بتعكس **خطر الـ production**، مش مجهود الإصلاح.

### 🔴 Critical (P0)

| ID | Finding | الدليل |
|---|---|---|
| **P0-1** | **Build fails** | `ProvisioningServiceImpl.java:47` فيه `... , e);o ;` — و`TenantProvisioningException` ناقصها constructor بـ `(String, Throwable)` |
| **P0-2** | **JWT keys فاضية** | `src/main/resources/certs/private_key.pem` و `public_key.pem` حجمهم **0 bytes** |
| **P0-3** | **Common schema migration فاضية** | `db/migration/common/V1__init_Common_tables.sql` حجمها 0 bytes |
| **P0-4** | **Tenant isolation مش متفرض** | مفيش `CurrentTenantIdentifierResolver` / `MultiTenantConnectionProvider`؛ `TenantContext.getCurrentSchema()` مش مستخدمة؛ الـ repositories مش tenant-scoped؛ الـ resolver **fail-open** لـ `public` |
| **P0-5** | **Stock movement engine غايب** | methods `StockMvtServiceImpl` فاضية؛ مفيش controller؛ مفيش insufficient-stock check |
| **P0-6** | **مفيش login endpoint** | `SecurityConfig` بيسمح بـ `/api/v1/auth/**` لكن مفيش auth controller |
| **P0-7** | **Schema-name injection و rollback هدّام** | `ProvisioningServiceImpl` بيبني SQL بـ `String.format("CREATE SCHEMA ... %s", schemaName)` و `DROP SCHEMA ... CASCADE`؛ `companyCode` عليه `@NotBlank` بس؛ `CREATE SCHEMA IF NOT EXISTS` ممكن تطابق schema حي موجود **ويتمسح عند الفشل** |
| **P0-8** | **تعرّض الـ password hash** | `UserMapper.toResponse` بيحط `password`؛ `UserResponse` بيكشفه؛ `TenantResponse` فيه `adminPassword` |
| **P0-9** | **Privilege escalation في إنشاء المستخدمين** | `UserServiceImpl.createUser` بيرفض `ROLE_USER` لكن **بيسمح بـ `ROLE_PLATFORM_ADMIN`**؛ `updateUser` بيعمل العكس |

### 🟠 High (P1)

| ID | Finding | الدليل | الإصلاح المطلوب |
|---|---|---|---|
| **P1-1** | **Tenant onboarding no-op / غير متسق** | `registerTenant` مش بيحفظ؛ `approveTenant` بيحط `ACTIVE` قبل الـ provisioning وبيبلع الفشل؛ `activateTenant` بيتخطى الـ provisioning؛ `createAdminUserForTenant` مش مستخدمة؛ مفيش `@Transactional` | **State machine transactional**، persist للتسجيل، provisioning **idempotent**، إنشاء admin مفعّل، إظهار الأخطاء |
| **P1-2** | **الـ inserts بتفشل بسبب `created_by`** | `AbstractEntity.createdBy` هو `nullable=false`؛ الـ tenant migration عاملها `NOT NULL`؛ مفيش `AuditorAware` | ضيف `AuditorAware` مدعوم بـ security context مع **system fallback** للـ provisioning |
| **P1-3** | **Product update مكسور** | الـ duplicate check بيشتغل قبل تحميل الـ entity فبيلاقي نفسه؛ الـ mapper بيعمل entity جديدة و`setId` بتضيّع audit fields | حدّث الـ managed entity؛ استبعد الـ current id من الـ duplicate check؛ ضيف update test |
| **P1-4** | **Hard delete و soft delete غير مستخدم** | `CategoryServiceImpl.delete` و `ProductServiceImpl.delete` بينادوا `repository.delete`؛ أعمدة `deleted` مش في الـ queries؛ الـ FK مفيهاش cascade | Soft delete + query filters + referential checks؛ خلّي الحركات **append-only** |
| **P1-5** | **فجوات الـ validation** | `ProductRequest` / `StockMvtRequest` مفيهمش constraints؛ `RegisterTenantRequest` ناقصه `@Email` / `@Pattern`؛ الـ pagination بلا حدود | Bean Validation + custom validators + caps لحجم الصفحة + tests |
| **P1-6** | **فجوات الـ exception mapping** | `BusinessException`, `EntityNotFoundException`, `DuplicateKeyException`, `IllegalStateException`, malformed JSON, type mismatches → كلها 500 | Map لـ 400/404/409 بنموذج خطأ مقروء للآلة |
| **P1-7** | **مفيش authorization** | مفيش `@PreAuthorize` / permission checks؛ أي authenticated user يقدر ينادي أي endpoint | Method security + permission matrix لكل role |
| **P1-8** | **Stale/deleted users و suspended tenants** | الـ filter بيثق في JWT claims؛ مفيش lookup لحالة tenant/user؛ `loadUserByUsername` بيشمل المحذوفين | تحقق من حالة tenant/user مع كل request أو short tokens + revocation؛ استبعد المحذوفين |
| **P1-9** | **Token lifecycle** | Access token 24 ساعة؛ مفيش refresh/logout/revocation/key rotation؛ الـ role claim يفضل صالح بعد تغيير الـ role | Access tokens قصيرة، rotating refresh tokens، `jti` / token version، خطة rotation |
| **P1-10** | **Perimeter controls ضعيفة** | CORS `*`؛ مفيش rate limiting؛ مفيش login lockout؛ مفيش security headers صريحة | قيّد الـ origins، أضف rate limiting و headers وحماية brute-force |
| **P1-11** | **مفيش security tests / CI** | context-load test بس؛ مفيش vulnerability أو dependency scanning | ضيف test matrix من [[14 - Testing]] و CI gates |
| **P1-12** | **مفيش observability** | مفيش Actuator ولا metrics backend ولا tracing؛ `show-sql: true` في الـ default profile | Health/metrics، structured logs، correlation IDs؛ عطّل SQL logging بره الـ dev |
| **P1-13** | **Indexes ناقصة** | PK/unique indexes بس؛ PostgreSQL مابيعملش index للـ FKs | ضيف indexes من [[08 - Database Design]] §8.5 مع تحقق من query plans |

### 🟡 Medium (P2)

| ID | Finding | الدليل | الإصلاح المطلوب |
|---|---|---|---|
| **P2-1** | **Non-RESTful API paths** | `/create`, `/update/{id}`, `/delete/{id}` | Resource-oriented paths و status codes صحيحة |
| **P2-2** | **Pagination/filtering gaps** | مفيش max size/sort/filters؛ page سالب → 500 | حدّ للـ size، allowlist للـ sort، filters وvalidation |
| **P2-3** | **OpenAPI gaps** | مفيش security scheme ولا examples ولا error schema | اظبط springdoc بـ bearer auth و `ProblemDetail` |
| **P2-4** | **Dead/duplicated code** | `TenantFilter` و `TenantHibernateFilter` متعمل لهم comment؛ `TenantHibernateFilter` بيشاور على filter غير معرّف؛ unused imports؛ `updatedCategory` مش مستخدم؛ typo في اسم الـ package | احذف أو كمّل بعد قرار الـ tenancy |
| **P2-5** | **Duplicate identity data** | اسم/email/username/password الـ admin متخزّنين في `tenants` و `users` | خلّي الـ credentials في `users` بس واشتق حقول العرض |
| **P2-6** | **Soft-delete inconsistency** | `TenantRepository.findAll` بيشمل المحذوفين؛ user queries بتستبعدهم | طبّق سياسة متسقة |
| **P2-7** | **Case-sensitive uniqueness** | اسم الـ category ومرجع الـ product بيتفحصوا case-insensitive لكن constraint الـ DB case-sensitive | استخدم functional unique indexes وتعامل مع `DataIntegrityViolation` |
| **P2-8** | **Eager association / null risk** | `Product.category` هي `@ManyToOne` (EAGER by default)؛ الـ mapper بيعمل dereference من غير null check | استخدم projection/join fetch؛ تعامل مع category الـ null |

## 12.3 🎯 Threat Model Highlights

| التهديد | التعرّض الحالي | الضابط المستهدف |
|---|---|---|
| **Cross-tenant data access** | 🔴 عالي — العزل مش متفرض | Schema routing/RLS + tenant predicates إجبارية + tests |
| **Tenant impersonation via header** | 🟢 منخفض حاليًا (الـ filter متعطّل) | **ماتثقش أبدًا في `X-TenantID`**؛ استخدم الـ verified JWT |
| **SQL injection via schema name** | 🔴 عالي | Identifier validation صارم و DDL آمن |
| **Credential leakage** | 🔴 عالي — password hashes في DTOs | شيل الحقول السرية؛ response tests |
| **Privilege escalation** | 🔴 عالي — bug في فحص الـ role | Server-side role allowlist و permission tests |
| **Brute-force login** | ⚪ مش منطبق بعد (مفيش login) لكنه قريب | Rate limiting و lockout و MFA |
| **Token theft/replay** | 🔴 تأثير عالي مع tokens 24 ساعة | Short TTL و refresh rotation و revocation |
| **Prompt injection via inventory data** | 🔵 مستقبلي | Tool allowlist وفصل data/instructions و validation |
| **Unsafe AI SQL** | 🔵 مستقبلي | Semantic layer و read-only role و حقن tenant predicate |
| **Supply-chain vulnerability** | ❔ مجهول | Dependency scanning و **SBOM** |

## 12.4 🔑 الأسرار والـ Configuration

- `.env` موجود في `.gitignore` (كويس)، لكن الـ repo بيتتبّع `src/main/resources/certs/*.pem` (فاضية حاليًا). **شيلها من version control** وضيف **خطوة توليد موثّقة**.
- **ماتخزّنش مفاتيح production** في الـ repo أو الـ Docker images. استخدم **secret manager** أو mounted secrets.
- وفّر `.env.example` بـ placeholders من غير credentials حقيقية.
- **اعمل rotate** لزوج مفاتيح JWT قبل الـ production ووثّق إجراء overlap/rotation.
- اتأكد إن الـ logs **ماتطبعش tokens أو passwords أو بيانات شخصية كاملة**.
- اظبط **secrets مختلفة لكل بيئة**.

## 12.5 الخصوصية والـ Compliance

بيانات المخزون ممكن تحتوي أسماء موردين/عملاء ونشاط موظفين. **قبل الإطلاق التجاري:**

- حدّد **data retention و deletion policies** لكل tenant.
- وفّر **tenant data export و deletion** (طلبات على غرار GDPR، حسب الاختصاص القضائي).
- قلّل البيانات الشخصية في **AI prompts و embeddings و logs**.
- وقّع **data-processing agreements** مع مزوّدي الموديلات ووثّق أماكن معالجة البيانات.
- قيّد وصول الـ platform-admin لبيانات الـ tenants **وسجّله في audit**.
- شفّر البيانات **أثناء النقل (TLS)** و**عند التخزين** (database/disk encryption).
- وثّق الـ **sub-processors**.

> [!note] ده **مش استشارة قانونية**؛ الالتزامات الفعلية بتعتمد على السوق المستهدف.

---
⬅️ [[11 - AI Capabilities]] | ➡️ [[13 - Performance and Scalability]]
