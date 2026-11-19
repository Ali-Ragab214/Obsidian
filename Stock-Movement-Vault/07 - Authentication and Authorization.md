---
title: Authentication and Authorization
cssclasses: [rtl]
tags: [auth, jwt, rbac, security]
section: 7
---

# 7. Authentication and Authorization
⬅️ [[06 - Multi-Tenancy Strategy]] | ➡️ [[08 - Database Design]]

## 7.1 الموجود حاليًا

| Component | المسؤولية | Status |
|---|---|---|
| `JwtTokenService` | بيولّد **RS256 access tokens** فيها `sub`, `tenant_id`, `role`, `iat`, `exp`, `iss`، وبيتحقق من الـ signature والـ expiry | 🧱 Scaffolded؛ المفاتيح **فاضية** |
| `JwtAuthenticationFilter` | بيستخرج الـ Bearer token، يتحقق منه، يضبط `TenantContext`، يحل الـ schema، ويملّى `SecurityContext` | 🟡 Partial |
| `SecurityConfig` | Stateless chain، CSRF متقفّل، CORS مفعّل، Swagger و `/api/v1/auth/**` public، وأي request تاني authenticated | ✅ Implemented (in code) |
| `User` | بينفّذ `UserDetails`؛ الـ authorities مشتقة من **role واحد** | ✅ Implemented |
| `UserServiceImpl` | بينفّذ `UserDetailsService`؛ بيدير المستخدمين | 🟡 Partial |
| `BeansConfig` | **BCrypt** password encoder | ✅ Implemented |

> [!success] استخدام **RS256** اختيار كويس
> المفتاح الخاص للتوقيع بيفضل سري، والمفتاح العام للتحقق بيتوزّع. **لكن** ملفات المفاتيح الحالية متعملها commit وهي **فاضية**، ومفيش **key management strategy**.

## 7.2 ❌ الناقص

> [!failure] الفجوات
> - **مفيش login endpoint.** `SecurityConfig` بيسمح بـ `/api/v1/auth/**` لكن مفيش `AuthController` في الـ repo. مستحيل تاخد token من الـ API.
> - **مفيش login flow أو authentication wiring صريح.** مفيش login controller ولا `AuthenticationManager` / `DaoAuthenticationProvider` configuration، ولا flow بيفحص الـ password وحالة الـ tenant.
> - **مفيش refresh tokens ولا logout ولا revocation.** الـ access token بيعيش **24 ساعة** (`86400000 ms`) ومينفعش يتلغى.
> - **مفيش authorization rules.** `@EnableMethodSecurity` موجودة، لكن مفيش ولا `@PreAuthorize` ولا `@Secured` ولا permission check في الـ codebase. أي authenticated user هيقدر يناديه أي endpoint هيتضاف.
> - **مفيش tenant-status check.** token لـ tenant suspended أو inactive بيفضل صالح لحد الـ expiry.
> - **مفيش user-status check على الـ requests.** `enabled` و `deleted` مش بيتفحصوا في الـ filter. و`loadUserByUsername` **مش بيستبعد** الـ soft-deleted users.
> - **مفيش role hierarchy أو permission model.** الـ `role` claim متوثّق فيه طول عمر الـ token. لو الـ admin غيّر role مستخدم، الـ token القديم يفضل بصلاحيته القديمة.
> - **مفيش rate limiting ولا lockout ولا MFA ولا invitation flow.**
> - **مفيش authentication entry point / access-denied handler.** الاستجابات 401 / 403 مش متشكّلة صراحةً، فالـ API clients ممكن ماتلاقيش payloads متسقة.
> - **CORS بيسمح بـ `*`.** مقبول للتجارب المحلية، **مش للـ production**.
> - **الـ Password hashes بترجع في الـ responses.** `UserMapper.toResponse` بينسخ `user.getPassword()` في `UserResponse`، و`TenantResponse` فيه حقل `adminPassword` (مش متملّي حاليًا). الاتنين عقود API **خطيرة**.

## 7.3 الـ Role Model

الـ enum الحالي:

- `ROLE_PLATFORM_ADMIN`
- `ROLE_COMPANY_ADMIN`
- `ROLE_ADMINISTRATOR`
- `ROLE_USER`
- `ROLE_SALES_OPERATOR`

### 💡 Permission matrix مقترحة (تتفرض **server-side** مش في الـ UI بس)

| Capability | Platform admin | Company admin | Administrator | Sales operator | User |
|---|---|---|---|---|---|
| Create/suspend tenants | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage tenant users and roles | ❌ (support فقط، مع audit) | ✅ | ✅ | ❌ | ❌ |
| Manage warehouses | ❌ | ✅ | ✅ | ❌ | ❌ |
| Manage products/categories | ❌ | ✅ | ✅ | ❌ | Read |
| Post stock IN/OUT | ❌ | ✅ | ✅ | ✅ (OUT بس، لو اتمنحت) | ❌ |
| Post transfers/adjustments | ❌ | ✅ | ✅ | ❌ | ❌ |
| View reports | Platform aggregate بس | ✅ | ✅ | محدود | محدود |
| Use AI assistant | ❌ (مفيش tenant data) | ✅ | ✅ | ✅ (read-only) | ✅ (read-only) |

> [!warning] المصفوفة دي **مقترحة**. الكود الحالي **مفيهوش أي enforcement**.

## 7.4 🔐 الـ Authentication Flow الموصى بيه

1. `POST /api/v1/auth/login` بـ **tenant code** (أو tenant-agnostic lookup) و username و password.
2. حل الـ tenant بالـ code؛ ارفض **المجهول** و `PENDING` و `INACTIVE` و `SUSPENDED`.
3. حمّل المستخدم بـ `(tenant_id, username)`؛ ارفض المحذوف أو المعطّل.
4. تحقق من **BCrypt hash** بـ constant-time check.
5. أصدر **short-lived access token** (مثلًا **15 دقيقة**) و **rotating refresh token**.
6. حط `sub` و `tenant_id` و `role` و `token_version` و `jti` في الـ access token.
7. مع كل request: تحقق من الـ signature والـ expiry وحالة الـ tenant والـ token version؛ واختياريًا افحص revocation list.
8. وفّر `POST /api/v1/auth/refresh` و `POST /api/v1/auth/logout`.
9. فرض الـ permissions بـ **method security** وـ **tenant-scoped service checks**.

> [!note] مدد الـ tokens واستراتيجية الـ refresh **TBD**؛ تتحدد حسب الـ **threat model** وتجربة المستخدم.

## 7.5 🛡️ أمان الـ Password والـ Account

- **BCrypt** مستخدم، وده مناسب. فكّر في **strength قابلة للضبط** و **rehash-on-login**.
- فرض **password policy** وفحص **breached passwords**.
- أضف **login rate limiting** و **temporary lockout** بعد محاولات فاشلة متكررة.
- **ماتسجّلش أبدًا** passwords أو hashes في الـ logs.
- **ماترجّعش** password fields في أي response DTO.
- فكّر في **MFA** لـ company administrators و platform administrators.
- استخدم **email verification** و **expiring invitation tokens** لأعضاء الفريق.

---
⬅️ [[06 - Multi-Tenancy Strategy]] | ➡️ [[08 - Database Design]]
