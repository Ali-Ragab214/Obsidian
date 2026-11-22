# SECURITY HARDENING PLAN — endpoints, roles, `@PreAuthorize`, `requestMatchers`

> Source: the agent's permission report (roles list, endpoint map, gaps). This file turns it into (1) a findings report, (2) a target permission matrix, (3) an ordered implementation plan the agent can execute. Anything marked **VERIFY** is an inference that must be confirmed in the real code before changing it.

## ملخص بالعربي

- المشروع دلوقتي: الـ tenants والـ users متأمّنين بالأدوار، لكن **Categories / Products / Stock-mvts مفتوحين لأي مستخدم مسجّل** (حتى `ROLE_USER`).
- الخطة: أمان على طبقتين: `requestMatchers` في `SecurityConfig` (حماية خشنة حسب المسار والـ HTTP method) + `@PreAuthorize` على الـ controllers (حماية دقيقة). الاتنين مع بعض، ولو واحد غلط التاني يمسك.
- في **3 قرارات** لازم تاخدها قبل التنفيذ (قسم 2). كاتب لكل قرار القيمة الافتراضية.
- أخطر حاجتين جديدتين: `PUT/DELETE` على `stock-mvts` (بيكسروا الـ ledger)، واحتمال إن `users` (موجودين في `public`) يتقرأوا عبر الـ tenants (**VERIFY**).

---

## 1. Findings report (what needs fixing)

|#|Severity|Finding|Status|
|---|---|---|---|
|F1|**High**|`CategoryController`, `ProductController`, `StockMvtController` have no `@PreAuthorize`: any authenticated user (even `ROLE_USER`) can create/update/delete.|Confirmed (agent report)|
|F2|**High**|`PUT /api/v1/stock-mvts/{id}` and `DELETE /api/v1/stock-mvts/{id}` exist. The stock ledger is append-only: movements must never be edited or deleted. Replace with `POST /{id}/reverse` (see `StockMvt` reversal design).|Confirmed (endpoints listed)|
|F3|**High**|`ROLE_PLATFORM_ADMIN` has no tenant, yet is "any authenticated" on tenant-business endpoints. It would run with an empty `TenantContext`. Must be denied there, and tenant resolution must fail closed.|**VERIFY** behavior with an empty tenant|
|F4|**High**|`users` lives in the shared `public` schema. By-id operations (`GET/PUT/DELETE/enable/disable /users/{id}`) must check that the user belongs to the caller's tenant, otherwise a `COMPANY_ADMIN` of tenant A could touch tenant B's users (IDOR).|**VERIFY** in `UserServiceImpl`|
|F5|Medium|JWT lifetime is 24h (`app.jwt.access-token-expiration = 86400000`). A disabled user or suspended tenant keeps working until the token expires; status is checked only at login.|Confirmed|
|F6|Medium|`PUBLIC_URLS` uses `/api/v1/auth/**`: any future endpoint under `/auth` (e.g. `/auth/me`) becomes public by accident.|Confirmed|
|F7|Medium|Mixed SpEL style (`hasAuthority('ROLE_PLATFORM_ADMIN')` vs `hasRole('COMPANY_ADMIN')`). Works, but fragile.|Confirmed|
|F8|Medium|`ROLE_ADMINISTRATOR` can only read users; `ROLE_USER` and `ROLE_SALES_OPERATOR` are declared but never enforced. Role semantics undefined.|Fixed by section 3 matrix|
|F9|Medium|Fallback is `anyRequest().authenticated()`. A new, forgotten controller is open to every logged-in user. Use `denyAll()` as the fallback.|Confirmed|
|F10|Medium|Swagger / `/v3/api-docs` are public in every profile.|Confirmed (PUBLIC_URLS)|
|F11|Medium|Denials produced at filter level (`requestMatchers`) must return the JSON `ErrorResponse` (401/403), not an HTML/empty body.|**VERIFY** (entry point / access-denied handler)|
|F12|Low|Public `POST /api/v1/auth/register` can be abused (spam tenants). By design, but needs rate limiting or captcha later.|Accepted for now|
|F13|Low|Self-lockout: a `COMPANY_ADMIN` can disable/delete themselves or the last admin of the company.|**VERIFY**|
|F14|Low|No automated test proves who can call what.|Confirmed|

---

## 2. Decisions (defaults are applied unless you change them)

|#|Question|Default used in this plan|
|---|---|---|
|D1|May `PLATFORM_ADMIN` use tenant business endpoints (categories/products/stock)?|**No.** It has no tenant. It only manages tenants.|
|D2|What is `ROLE_ADMINISTRATOR`?|Operations manager: writes catalog, records/reverses stock, **reads** users. Cannot create/update/delete users (that stays `COMPANY_ADMIN`).|
|D3|What can `ROLE_SALES_OPERATOR` do?|Read catalog and stock; create **OUT** movements only (sales). No catalog writes, no IN movements, no reversals.|
|D4|What can `ROLE_USER` do?|Read-only (catalog and stock).|
|D5|Who can reverse a stock movement?|`COMPANY_ADMIN`, `ADMINISTRATOR`.|
|D6|JWT lifetime now?|Reduce to **30 minutes** (config only). Per-request tenant/user status check comes with the auth task.|
|D7|Swagger in non-dev profiles?|**Disabled.**|

---

## 3. Target permission matrix

Legend: ✅ allowed · ❌ denied (403) · 🌐 public · Roles: PA = PLATFORM_ADMIN, CA = COMPANY_ADMIN, ADM = ADMINISTRATOR, SO = SALES_OPERATOR, U = USER. Unauthenticated callers always get 401 on anything that is not 🌐.

|Endpoint|PA|CA|ADM|SO|U|
|---|---|---|---|---|---|
|`POST /api/v1/auth/login`|🌐|🌐|🌐|🌐|🌐|
|`POST /api/v1/auth/register`|🌐|🌐|🌐|🌐|🌐|
|`GET /api/v1/tenants`|✅|❌|❌|❌|❌|
|`POST /api/v1/tenants/{id}/approve`|✅|❌|❌|❌|❌|
|`POST /api/v1/tenants/{id}/activate`|✅|❌|❌|❌|❌|
|`POST /api/v1/tenants/{id}/deactivate`|✅|❌|❌|❌|❌|
|`POST /api/v1/tenants/{id}/suspend`|✅|❌|❌|❌|❌|
|`POST /api/v1/tenants/migrate` (if it exists)|✅|❌|❌|❌|❌|
|`POST /api/v1/users`|❌|✅|❌|❌|❌|
|`GET /api/v1/users`, `GET /api/v1/users/{id}`|❌|✅|✅|❌|❌|
|`PUT /api/v1/users/{id}`|❌|✅|❌|❌|❌|
|`DELETE /api/v1/users/{id}`|❌|✅|❌|❌|❌|
|`PUT /api/v1/users/{id}/enable` and `/disable`|❌|✅|❌|❌|❌|
|`GET /api/v1/categories`, `GET /api/v1/categories/{id}`|❌|✅|✅|✅|✅|
|`POST /categories/create`, `PUT /categories/update/{id}`, `DELETE /categories/delete/{id}`|❌|✅|✅|❌|❌|
|`GET /api/v1/products`, `GET /api/v1/products/{id}`|❌|✅|✅|✅|✅|
|`POST/PUT/DELETE /api/v1/products...`|❌|✅|✅|❌|❌|
|`GET /api/v1/stock-mvts`, `/{id}`, `/product/{product-id}`|❌|✅|✅|✅|✅|
|`POST /api/v1/stock-mvts` (create movement)|❌|✅|✅|✅ (OUT only)|❌|
|`POST /api/v1/stock-mvts/{id}/reverse`|❌|✅|✅|❌|❌|
|`PUT /api/v1/stock-mvts/{id}`|**removed**|||||
|`DELETE /api/v1/stock-mvts/{id}`|**removed**|||||

---

## 4. Implementation plan (do in this order)

### S1. Role constants and meta-annotations (fixes F7, F8)

Create `security/Roles.java` and small annotations so every controller says _what_ it needs, not SpEL strings.

```java
public final class Roles {
    public static final String PLATFORM_ADMIN = "PLATFORM_ADMIN";
    public static final String COMPANY_ADMIN  = "COMPANY_ADMIN";
    public static final String ADMINISTRATOR  = "ADMINISTRATOR";
    public static final String SALES_OPERATOR = "SALES_OPERATOR";
    public static final String USER           = "USER";
    private Roles() {}
}
```

Annotations (each in `security/annotations/`, `@Retention(RUNTIME)`, `@Target({METHOD, TYPE})`):

|Annotation|`@PreAuthorize` value|
|---|---|
|`@PlatformAdminOnly`|`hasRole('PLATFORM_ADMIN')`|
|`@CompanyAdminOnly`|`hasRole('COMPANY_ADMIN')`|
|`@ManagerOnly`|`hasAnyRole('COMPANY_ADMIN','ADMINISTRATOR')`|
|`@StockOperator`|`hasAnyRole('COMPANY_ADMIN','ADMINISTRATOR','SALES_OPERATOR')`|
|`@TenantMember`|`hasAnyRole('COMPANY_ADMIN','ADMINISTRATOR','SALES_OPERATOR','USER')`|

Example:

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('COMPANY_ADMIN') or hasRole('ADMINISTRATOR')")
public @interface ManagerOnly {}
```

Use **`hasRole` everywhere**, including platform admin (replace `hasAuthority('ROLE_PLATFORM_ADMIN')`). The authority strings stay `ROLE_*` (from `User.getAuthorities()` and the JWT claim), so `hasRole('X')` matches `ROLE_X`.

### S2. `SecurityConfig`: coarse rules with `requestMatchers` (fixes F6, F9, F10)

Make sure `@EnableMethodSecurity` is present, the session policy is `STATELESS`, and CSRF is disabled only because the API is stateless JWT. Order matters: the **first** matching rule wins.

```java
http.authorizeHttpRequests(auth -> auth
    // --- public ---
    .requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()
    .requestMatchers(HttpMethod.POST, "/api/v1/auth/login", "/api/v1/auth/register").permitAll()
    // swagger: only when the dev profile enables it (see S6)

    // --- platform ---
    .requestMatchers("/api/v1/tenants/**").hasRole("PLATFORM_ADMIN")

    // --- users (shared public schema) ---
    .requestMatchers(HttpMethod.GET, "/api/v1/users/**").hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR")
    .requestMatchers("/api/v1/users/**").hasRole("COMPANY_ADMIN")

    // --- stock ledger: append-only, never PUT/DELETE ---
    .requestMatchers(HttpMethod.PUT,    "/api/v1/stock-mvts/**").denyAll()
    .requestMatchers(HttpMethod.DELETE, "/api/v1/stock-mvts/**").denyAll()
    .requestMatchers(HttpMethod.POST,   "/api/v1/stock-mvts/*/reverse").hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR")
    .requestMatchers(HttpMethod.POST,   "/api/v1/stock-mvts").hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR", "SALES_OPERATOR")

    // --- tenant business data: reads for all four tenant roles ---
    .requestMatchers(HttpMethod.GET, "/api/v1/categories/**", "/api/v1/products/**", "/api/v1/stock-mvts/**")
        .hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR", "SALES_OPERATOR", "USER")
    // --- catalog writes: managers only ---
    .requestMatchers("/api/v1/categories/**", "/api/v1/products/**")
        .hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR")

    // --- fail closed ---
    .anyRequest().denyAll()
);
```

Notes:

- Replace `/api/v1/auth/**` with the **explicit** public list above (F6). Future `/auth/me`, `/auth/refresh` etc. are decided one by one (`/auth/me` needs authentication).
- `anyRequest().denyAll()` means a forgotten controller is closed, not open (F9). If actuator/other paths are needed, add them explicitly.
- Adjust the path patterns if the real controllers differ (the category verbs `/create`, `/update/{id}`, `/delete/{id}` are covered by `/api/v1/categories/**`).

### S3. `@PreAuthorize` on every controller (fine-grained)

`requestMatchers` and `@PreAuthorize` must say the **same thing** (defense in depth; a test in S7 enforces it).

- `TenantController`: class-level `@PlatformAdminOnly`.
- `UserController`: `POST`, `PUT`, `DELETE`, `enable`, `disable` → `@CompanyAdminOnly`; both `GET` → `@ManagerOnly`.
- `CategoryController`, `ProductController`: all `GET` → `@TenantMember`; `POST/PUT/DELETE` → `@ManagerOnly`.
- `StockMvtController`: all `GET` → `@TenantMember`; `POST` create → `@StockOperator`; `POST /{id}/reverse` → `@ManagerOnly`; **delete the `PUT` and `DELETE` methods** (F2).

`SALES_OPERATOR` can only create OUT movements. Enforce it in the service (not only the controller):

```java
@Transactional
public StockMvtResponse create(StockMvtRequest request) {
    final boolean isManager = hasAnyRole("COMPANY_ADMIN", "ADMINISTRATOR");
    if (!isManager && request.getTypeMvt() != TypeMvt.OUT) {
        throw new AccessDeniedException("Sales operators may only record OUT movements");
    }
    // ... existing logic
}
```

(Adjust the accessor name to the real `StockMvtRequest`; `hasAnyRole` is a small helper over `SecurityContextHolder`.)

### S4. Stock ledger endpoints (fixes F2)

- Remove `PUT` and `DELETE` from `StockMvtController` and the matching service methods.
- Add `POST /api/v1/stock-mvts/{id}/reverse` with body `{ "reason": "..." }` (`@NotBlank`, max 500). It creates a new opposite movement linked by `reversal_of_id` (unique, so a movement can be reversed only once). Details: see the `StockMvt` reversal design already discussed (migration adds `reversal_of_id`).
- Remove `@SQLDelete`/`@SQLRestriction` from `StockMvt` if still present.

### S5. Tenant isolation inside the shared `users` table (fixes F4) — VERIFY FIRST

Because `users` is in `public`, every by-id operation must be scoped by the caller's tenant (taken from the JWT/`TenantContext`, never from the request).

```java
private User loadOwned(String userId) {
    final String tenantId = TenantContext.requireTenantId();   // fail closed if missing
    return userRepository.findByIdAndTenantId(userId, tenantId)
            .orElseThrow(() -> new EntityNotFoundException("User not found")); // 404, not 403: do not leak existence
}
```

Use it in get/update/delete/enable/disable. `findAll` must already be scoped (`findAllByTenantId`). Also:

- A `COMPANY_ADMIN` cannot assign `PLATFORM_ADMIN` (already enforced) and cannot create users in another tenant.
- Self-protection (F13): block disabling/deleting yourself and the last active `COMPANY_ADMIN` of the tenant (409).
- `PLATFORM_ADMIN` (no tenant) must be rejected on tenant-business endpoints (D1) and `TenantContext.requireTenantId()` must throw when there is no tenant (F3).

### S6. Configuration hygiene (fixes F5, F10, F11)

- `app.jwt.access-token-expiration`: set to `1800000` (30 min) (D6).
- Swagger/OpenAPI: enable only in the `dev` profile (`springdoc.api-docs.enabled=false` and `springdoc.swagger-ui.enabled=false` elsewhere), and permit those paths only in a dev-only `SecurityFilterChain`/matcher.
- Provide JSON `AuthenticationEntryPoint` (401) and `AccessDeniedHandler` (403) using the project's `ErrorResponse`, registered in `exceptionHandling(...)`, so denials from `requestMatchers` have the same body as denials from `@PreAuthorize`.

### S7. Tests (fixes F14)

Add a parameterized authorization test, one row per (endpoint, role) from the section 3 matrix.

- Use `@WebMvcTest` (or `@SpringBootTest` + `@AutoConfigureMockMvc`), `@Import(SecurityConfig.class)`, services as `@MockitoBean`, and `@WithMockUser(roles = ...)`.
- Expected results: allowed → anything except 401/403; denied → 403; no authentication → 401.
- Extra tests:
    - `PUT` and `DELETE` on `/api/v1/stock-mvts/**` → 403 for every role (and the methods no longer exist).
    - `SALES_OPERATOR` creating an IN movement → 403; creating OUT → success.
    - `PLATFORM_ADMIN` on `/api/v1/products` → 403.
    - Tenant isolation: `COMPANY_ADMIN` of tenant A requesting a user of tenant B by id → 404; update/delete/enable/disable → 404.
    - A request whose path matches no rule → 403 (`denyAll`).
    - A 401/403 produced at filter level returns the JSON `ErrorResponse`.
    - A consistency test: for every controller method, the `requestMatchers` rule and the `@PreAuthorize` annotation allow the same roles (or at least: no method without an authorization annotation).

---

## 5. What needs fixing before this works (checklist for the agent)

- [ ] Confirm `@EnableMethodSecurity` is on and session policy is `STATELESS`.
- [ ] Confirm what `TenantContext` returns for a `PLATFORM_ADMIN` token (F3) and make business endpoints fail closed.
- [ ] Read `UserServiceImpl` and confirm/repair tenant scoping on every by-id operation (F4).
- [ ] Confirm `StockMvtRequest` field names and the reversal migration (`reversal_of_id`) are in place before adding `reverse`.
- [ ] Confirm whether `POST /api/v1/tenants/migrate` exists and secure it as platform-admin only.
- [ ] Check there are no other controllers (health, actuator, test endpoints) that `denyAll` would break.

## 6. Definition of Done

- [ ] Every endpoint matches the section 3 matrix (verified by the parameterized test).
- [ ] No controller method without an authorization annotation; `anyRequest().denyAll()` is the fallback.
- [ ] `PUT`/`DELETE` on stock movements are gone; `reverse` works and a movement can be reversed once.
- [ ] `SALES_OPERATOR` cannot create IN movements; `ROLE_USER` is read-only; `PLATFORM_ADMIN` cannot reach tenant business data.
- [ ] Cross-tenant user access returns 404 (tested).
- [ ] 401/403 bodies are JSON `ErrorResponse` from both `requestMatchers` and `@PreAuthorize`.
- [ ] JWT lifetime is 30 min; Swagger is disabled outside `dev`.
- [ ] `./mvnw clean verify` is green, plus a manual curl run for one allowed and one denied call per role.

## 7. Report format (what the agent must hand back)

Write `SECURITY_HARDENING_REPORT.md` with: summary; table of F1–F14 with status (Fixed / Partially / Not done / Not applicable) and how it was verified; decisions applied (D1–D7) and any deviation; files changed; the commands and real outputs of the curl checks; what was **not** verified. Never claim a check passed unless it was run.

## 8. Out of scope (later tasks)

- Refresh tokens, token revocation list, per-request user/tenant status check (auth task).
- Rate limiting / captcha on `POST /auth/register` (F12).
- Renaming category endpoints to plain REST (`/categories`, `/categories/{id}`).
- Platform-admin seeding (first admin creation).