# Identity — خطة التنفيذ

> **الـ Phase:** 1 · **المدة:** 4–5 أيام · **الترتيب:** بعد `tenancy`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md) · [implementation-phases](../../development-roadmap/implementation-phases.md)

---

## 1. المهام

### TSK-IDN-001 — Migration: `users`

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-001 |
| **المخرجات** | `V4__create_users.sql` |
| **معيار القبول** | RLS + Unique `(org_id, phone)` + CHECK للـ Role/Owner |

**خطوات:**
1. الأعمدة الأساسية + `password_hash`.
2. `must_change_password`, `last_login_at`, `failed_login_attempts`, `locked_until`.
3. `CHECK (role IN (...))`.
4. `CHECK ((role = 'GYM_OWNER' AND branch_id IS NULL) OR (role <> 'GYM_OWNER' AND branch_id IS NOT NULL))`.
5. Unique `(org_id, phone)` + Partial Unique `(org_id, email)`.
6. `SELECT app.enable_tenant_rls('users');`.

---

### TSK-IDN-002 — Migration: `refresh_tokens`

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-001 |
| **المخرجات** | `V5__create_refresh_tokens.sql` |
| **معيار القبول** | `token_hash` Unique + RLS + Index على `(user_id, expires_at)` |

---

### TSK-IDN-003 — `JwtService` + Filter

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-001 |
| **المخرجات** | `JwtService`, `JwtAuthenticationFilter`, `UserPrincipal` |
| **معيار القبول** | Claims كاملة + Expiry + توقيع HS256 |

**خطوات:**
1. `generateAccessToken(User)` → 15 دقيقة.
2. `parse(token)` → Claims.
3. Filter يضبط `SecurityContext` + `TenantContext`.
4. `finally` → `TenantContext.clear()`.
5. `JWT_SECRET` من Environment.

---

### TSK-IDN-004 — Login

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-003, TSK-TEN-003 |
| **المخرجات** | `POST /auth/login` + `AuthService.login(...)` |
| **معيار القبول** | Org Slug + Lockout + Audit + نفس رسالة الخطأ |

**خطوات:**
1. `OrganizationQueryService.findBySlug`.
2. `TenantTransactionRunner.runInTenant`.
3. البحث بـ Phone أو Email.
4. BCrypt + Lockout.
5. إصدار Tokens.
6. Audit `LOGIN_SUCCESS` / `LOGIN_FAILED`.

---

### TSK-IDN-005 — Refresh + Logout

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-004 |
| **المخرجات** | `POST /auth/refresh`, `POST /auth/logout` |
| **معيار القبول** | Rotation + Reuse Detection + Revocation |

**خطوات:**
1. Parse `orgId` من الـ Token.
2. Hash + Lookup.
3. Revoke القديم + إصدار جديد.
4. Reuse → Revoke All + Audit.
5. Logout → Revoke.

---

### TSK-IDN-006 — User Management APIs

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-004 |
| **المخرجات** | `UserController` + 6 Endpoints |
| **معيار القبول** | Owner فقط + Branch Validation + Audit |

**خطوات:**
1. `GET /users` (Owner).
2. `POST /users` — Temp Password.
3. `GET /users/{id}`.
4. `PATCH /users/{id}`.
5. `POST /users/{id}/deactivate`.
6. `POST /users/{id}/reset-password`.
7. منع تعطيل آخر Owner.

---

### TSK-IDN-007 — Password Change + `mustChangePassword`

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-004 |
| **المخرجات** | `PATCH /auth/password` + Filter Logic |
| **معيار القبول** | Password Policy + Revoke Tokens + منع باقي الـ Endpoints |

**خطوات:**
1. تحقق من كلمة المرور القديمة.
2. Policy: 8–72 + Upper + Lower + Digit + Special.
3. BCrypt encode.
4. `mustChangePassword = false`.
5. Revoke كل Refresh Tokens.
6. Audit `PASSWORD_CHANGED`.

---

### TSK-IDN-008 — Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-IDN-007 |
| **المخرجات** | Unit + Integration + Security + Isolation |
| **معيار القبول** | كل بنود [`testing-plan.md`](./testing-plan.md) ناجحة |

---

## 2. الترتيب

```text
TSK-IDN-001 users
      ↓
TSK-IDN-002 refresh_tokens
      ↓
TSK-IDN-003 JwtService + Filter
      ↓
TSK-IDN-004 Login
      ↓
TSK-IDN-005 Refresh + Logout
      ↓
TSK-IDN-006 User Management
      ↓
TSK-IDN-007 Password Change
      ↓
TSK-IDN-008 Tests
```

> **ترتيب الـ Audit:** الـ Auth Core (TSK-IDN-001..005) بيتبني الأول، وبعدين `audit` Core (TSK-AUD-001/002)، وبعدها استدعاءات `AuditService` في `identity` تتوصل (TSK-AUD-003). كده مفيش Circular Dependency أثناء البناء، والـ Endpoint مش بيتسلم من غير Audit.

---

## 3. معايير القبول النهائية

- [ ] `users` + `refresh_tokens` Migrations + RLS.
- [ ] Login بـ `organizationSlug` يشتغل.
- [ ] Access Token 15 دقيقة + Claims صحيحة.
- [ ] Refresh Rotation + Reuse Detection.
- [ ] Logout يلغي الـ Refresh Token.
- [ ] Lockout بعد 5 محاولات.
- [ ] Owner فقط يدير الموظفين.
- [ ] `GYM_OWNER` → `branch_id = NULL`.
- [ ] تعطيل موظف يلغي Tokens.
- [ ] Password Change يلغي Tokens.
- [ ] `mustChangePassword` بيمنع باقي الـ API.
- [ ] Audit لكل العمليات.
- [ ] Security + Isolation Tests.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| تخمين الـ Org Slug | Rate Limit + نفس رسالة الخطأ |
| Refresh Token Leak | Hash + Rotation + Reuse Detection |
| Lockout DoS على حساب | قفل 15 دقيقة + إبلاغ الـ Owner |
| نسيان `TenantContext.clear()` | Filter + Test |
| `JWT_SECRET` ضعيف | 256-bit عشوائي + Secret Scan |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [security-strategy](../../architecture/security-strategy.md)
