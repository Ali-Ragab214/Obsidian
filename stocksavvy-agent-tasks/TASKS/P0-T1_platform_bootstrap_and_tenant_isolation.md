# P0-T1 — Platform bootstrap & tenant isolation completion

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 0 — Runnable foundation + tenant isolation |
| Source section | `6.1` of the original plan |
| Depends on (spec) | nothing. |
| Tasks that must be done before this one | none |
| Effort | **L** — 7–9 days. |
| Next task | P0-T2 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P0-T1 — "Platform bootstrap & tenant isolation completion" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): none. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

**Important:** the original request said multi-tenancy was already done in the Spring project. The scan found the *scaffolding* (JWT claim, `TenantContext`, provisioning) but **no enforced schema routing** (no `CurrentTenantIdentifierResolver` / `MultiTenantConnectionProvider`). First verify this in the code. If routing really exists (e.g. on another branch), skip the matching items and write the reason in `HANDOFF_LOG.md`. Do NOT redesign the schema-per-tenant model.

## 🎯 Spec features covered

(foundation: unblocks everything)

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 1 | P0 | Tenant isolation not enforced | No `CurrentTenantIdentifierResolver`/`MultiTenantConnectionProvider`; `TenantSchemaResolver` fails open; `hibernate.multi_tenancy: SCHEMA` inert | Add Hibernate resolver/provider with `search_path`; fail closed; add isolation tests. Keep schema-per-tenant. | 0 |
| 2 | P0 | Platform tables missing | `db/migration/common/V1__init_Common_tables.sql` is 0 bytes | Add `V2__create_platform_tables.sql` (do not edit an applied V1); create `tenants`, `users` with tenant-scoped uniqueness | 0 |
| 3 | P0 | App cannot start / no login | Both PEMs are 0 bytes; no `AuthController` (not found) | Generate keys outside VCS (or provide a dev keypair); add `POST /api/v1/auth/login` | 0 |
| 4 | P0 | `created_by` is `NOT NULL`, no auditor | `AbstractEntity.@CreatedBy`; no `AuditorAware` (not found) | Add `AuditorAware<String>` (security user, else `system`); set `updated_by` properly | 0 |
| 6 | P0 | Docker DB lacks pgvector | `docker-compose.yml` uses `postgres:17.5` | Switch to a pgvector-enabled image (e.g., `pgvector/pgvector:pg17`) or install the extension; run `CREATE EXTENSION vector` once | 0/4 |
| 10 | P1 | Tenant registration/approval inconsistent | `registerTenant` never saves; `createAdminUserForTenant` never called; `approveTenant` sets ACTIVE before provisioning and swallows errors; `activateTenant` skips provisioning; `UserServiceImpl.createUser` rejects `ROLE_USER` with a misleading “platform admin” message | Save PENDING; provision then ACTIVE; create admin user; add `provisioning_error`; fix role validation; add controllers | 0 |
| 11 | P1 | Global exception mapping incomplete | `BusinessException`, `EntityNotFoundException`, `DuplicateKeyException`, `IllegalStateException`, malformed JSON → HTTP 500 | Add handlers; return 400/401/403/404/409; add JSON security entry point/denied handler | 0 |
| 12 | P1 | Password hashes exposed | `UserMapper.toResponse` copies `password`; `TenantResponse` declares `adminPassword` | Remove both fields from responses | 0 |
| 14 | P1 | Tenant status/user status not checked | Filter only validates signature; `enabled`/`deleted` ignored | Resolver returns descriptor (schema, status); reject non-ACTIVE; check user on login | 0 |
| 18 | P2 | Inert/dead tenant code | `TenantFilter`, `TenantHibernateFilter` commented out; `@Cacheable` without `@EnableCaching` | Delete dead filters; enable a simple cache; evict on tenant change | 0 |
| 22 | P2 | No tests | Only `contextLoads` | Add unit/slice/integration/tenant-isolation tests (see each spec) | 0+ |
| 23 | P2 | `UserRepository.findByUsername` is global; `username`/`email` unique globally; `TenantRepository.ExistsByCompanyCode` has non-standard capitalization | Entity + repo | Make unique `(tenant_id, username)` and `(tenant_id, email)`; login uses company code + username; rename to `existsByCompanyCode` | 0 |
| 24 | P2 | Flyway V1 empty may already be applied | `flyway_schema_history` | Never edit applied V1; add V2+ (or clean the dev DB once, documented) | 0 |

## 🧭 Suggested internal order

1. **Runnable base:** generate the dev RSA keypair outside VCS (`app.jwt.*` paths), test keypair under `src/test/resources/certs`; switch `docker-compose.yml` to a pgvector-enabled image; clean `application.yml` (`show-sql` only in dev).
2. **Platform migrations:** `common/V2__create_platform_tables.sql` and `V3__platform_indexes.sql` (never edit empty V1). Fix `Tenant`/`User` entities (`@Table(schema = "public")`, composite uniques, no password fields in responses), repositories (`findByCompanyCodeIgnoreCase`, `existsByCompanyCode`, `findByTenantIdAndUsernameAndDeletedFalse`).
3. **Fail-closed tenant resolution:** `TenantContext` (`requireTenant`, `runAsPlatform`, `TenantDescriptor`), `TenantSchemaResolver` (cached descriptor, throws on unknown/non-ACTIVE), `@EnableCaching`, new exceptions.
4. **Hibernate routing:** `TenantIdentifierResolver`, `TenantConnectionProvider` (`SET search_path`, reset on release), `HibernateTenantConfig`.
5. **Audit:** `AuditorConfig` (`AuditorAware<String>`; `system` for background writes).
6. **Errors & security shape:** extend `GlobalExceptionHandler` (400/401/403/404/409), JSON entry point + denied handler, OpenAPI bearer scheme, `finally` clear in `JwtAuthenticationFilter`, remove dead tenant filters.
7. **Auth:** `AuthService` + `AuthController` (`/auth/login`, `/auth/me`, `/auth/register-tenant`).
8. **Tenant & user lifecycle:** fix `TenantServiceImpl` (save PENDING → provision → ACTIVE, store `provisioning_error`, create admin user), `UserServiceImpl` role validation, `TenantController`, `UserController`, `TenantMigrationService` (`/platform/tenants/migrate`).
9. **Tests:** startup, login, schema routing, two-tenant isolation (list/get/update/delete), fail-closed, migrations idempotent, 401/403.
10. DoD, `PROGRESS.md`, `HANDOFF_LOG.md`.

---

## 📘 SPEC (verbatim from the plan — do not skip any bullet)

### 6.1 Phase 0 — Platform bootstrap & tenant isolation completion

**Goal:** make the application start, authenticate, route every business query to the correct tenant schema, and fail closed when the tenant is unknown/suspended.

**Depends on:** nothing.

**Spring Boot approach**

- **Hibernate schema multi-tenancy** via `CurrentTenantIdentifierResolver<String>` + `MultiTenantConnectionProvider<String>`, registered in a `HibernatePropertiesCustomizer`.
  - Resolver reads `TenantContext.getCurrentSchema()`; throws `TenantResolutionException` if missing for a business operation.
  - Provider extends `AbstractMultiTenantConnectionProvider`; on `getConnection(tenantId)`, leases a pooled connection and runs `SET search_path TO "<schema>", public`; on `releaseConnection`, resets to `public` and returns it.
  - Why this over `AbstractRoutingDataSource`: one Hikari pool, no per-tenant pool explosion, and the schema boundary stays physical. Trade-off: connection state must be reset; test this explicitly.
- **Fail-closed resolver:** `TenantSchemaResolver` returns a `TenantDescriptor` (`schema`, `status`, `timezone`, `currency`) and throws on unknown/errored tenants. Never fall back to `public` for business data.
- **Platform context:** `TenantContext.runAsPlatform(...)` sets schema `public`; `Tenant`/`User` entities are explicitly mapped with `@Table(schema = "public")`. Business services call `TenantContext.requireTenant()` (or rely on the resolver throwing). `AuthService` wraps login/tenant lookup in a platform context so `public.users`/`public.tenants` can be read before a tenant is known; business requests always have a tenant context from the JWT filter.
- **Caching:** `@EnableCaching` + `ConcurrentMapCacheManager` for tenant descriptors; evict on tenant status/name change. No Redis for the thesis.
- **Provisioning:** save tenant as `PENDING`; on approval, create schema + run Flyway + create admin user; only then set `ACTIVE`. Store `provisioning_error` on failure; do not swallow it.
- **Existing-tenant migrations:** `TenantMigrationService.migrateAll()` iterates ACTIVE tenants and runs Flyway per schema; expose `POST /api/v1/platform/tenants/migrate` (platform admin) and an optional startup flag (`app.tenancy.migrate-on-startup=false` by default).
- **Migrations:** keep the empty common `V1` untouched; add `V2__create_platform_tables.sql` (because V1 may already be recorded). Tenant schemas: add `V2+`; never edit applied migrations.
- **JWT keys:** generate an RSA keypair outside VCS for dev (`openssl genpkey...`), load paths from `app.jwt.*`; document that committed empty files must be removed or regenerated. For tests, use a test-only keypair under `src/test/resources/certs`.
- **pgvector:** change `docker-compose.yml` to a pgvector-enabled PostgreSQL image; run `CREATE EXTENSION IF NOT EXISTS vector;` once per database (public migration or init script). This is needed before Phase 4 but is cheap to fix now.

**Entities / tables (public schema)**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `tenants` | `id`, `company_name`, `company_code`, `email`, `status`, `admin_full_name`, `admin_email`, `admin_user_name`, `admin_password` (hash), `timezone`, `currency`, `provisioning_error`, audit | `company_code` unique (lowercase, `^[a-z][a-z0-9_]{2,40}$`), `email` unique; index `status` |
| `users` | `id`, `tenant_id`, `username`, `email`, `password`, `first_name`, `last_name`, `role`, `enabled`, `deleted`, audit | unique `(tenant_id, username)`, unique `(tenant_id, email)`; FK `tenant_id → public.tenants(id)`; index `(tenant_id, deleted)` |

`@Table(schema = "public")` on `Tenant` and `User`. Remove `unique = true` from `User.username`/`email`; use composite table constraints. Add `TenantRepository.findByCompanyCodeIgnoreCase`, `UserRepository.findByTenantIdAndUsernameAndDeletedFalse`.

**Classes to create/modify**

- `config/TenantIdentifierResolver.java`, `config/TenantConnectionProvider.java`, `config/HibernateTenantConfig.java` (customizer).
- `config/TenantContext.java` (add `requireTenant()`, `runAsPlatform()`, `TenantDescriptor`), `config/TenantSchemaResolver.java` (fail-closed, cached descriptor).
- `config/AuditorConfig.java` (`AuditorAware<String>`).
- `security/AuthController.java`, `services/AuthService.java` + impl, `requests/LoginRequest.java`, `responses/LoginResponse.java`.
- `controllers/TenantController.java`, `controllers/UserController.java`; `services/impl/TenantServiceImpl` fixes; `services/impl/UserServiceImpl` fixes.
- `services/TenantMigrationService.java` + impl.
- `config/OpenApiConfig.java` (bearer scheme), `config/SecurityConfig` (entry point/denied handler, public URLs).
- `exceptions/TenantResolutionException.java`, `exceptions/TenantNotActiveException.java`; extend `GlobalExceptionHandler`.
- `db/migration/common/V2__create_platform_tables.sql`, `db/migration/common/V3__platform_indexes.sql`.
- Modify `entities/Tenant.java`, `entities/User.java`, `repositories/TenantRepository.java`, `repositories/UserRepository.java`, `application.yml`, `docker-compose.yml`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/auth/login` | `{companyCode, username, password}` | `{accessToken, tokenType, expiresIn, user:{id,username,role,tenantId}}` | Public |
| `GET` | `/api/v1/auth/me` | — | current user + tenant | Authenticated |
| `POST` | `/api/v1/auth/register-tenant` | `RegisterTenantRequest` | `201` + tenant id (status `PENDING`) | Public (rate-limit later) |
| `GET` | `/api/v1/platform/tenants?status=` | — | `PageResponse<TenantResponse>` | Platform admin |
| `POST` | `/api/v1/platform/tenants/{id}/approve` | — | `200` (provisioned + admin created) | Platform admin |
| `POST` | `/api/v1/platform/tenants/{id}/suspend` | — | `200` | Platform admin |
| `POST` | `/api/v1/platform/tenants/{id}/activate` | — | `200` (provision if needed) | Platform admin |
| `POST` | `/api/v1/platform/tenants/migrate` | — | migration summary | Platform admin |
| `GET/POST/PUT/DELETE` | `/api/v1/users...` | existing `UserRequest` | existing `UserResponse` | Manager / platform admin |

**Multi-tenancy considerations**

- This phase *is* the tenant isolation mechanism. Every later feature inherits it.
- `JdbcTemplate`/raw SQL does **not** go through Hibernate; schema-qualify with a validated identifier (`"tenant_x"`), e.g. `public.tenants` in the resolver and `tenant_x.knowledge_chunks` in Phase 4. Never concatenate an unvalidated `companyCode`.
- `TenantContext.clear()` must run in a `finally` block. Scheduled jobs set/clear context explicitly. If `@Async` is introduced, add a `TaskDecorator` that copies `TenantContext` into the worker thread; never rely on the ThreadLocal propagating by itself.
- Login resolves the tenant by company code and rejects `PENDING`/`INACTIVE`/`SUSPENDED`.
- Add an isolation test: create tenants A and B, write a product as A, authenticate as B, and assert 404 for A's id, empty list, and no cross-tenant update/delete.

**Human-in-the-loop / safety**

- No agent actions yet. The safety property is fail-closed isolation: unknown/missing tenant → error, not `public`.
- Tenant approval is a platform-admin action; no business data is exposed cross-tenant.

**Testing**

- Unit: resolver fail-closed, schema-name validation, status rejection.
- Integration (Testcontainers): startup with keys, login, schema routing, tenant A/B isolation for `Category` and `Product`.
- Migration: clean DB → `V2` platform tables; provision tenant → tenant schema + Flyway history; run `migrateAll()` twice (idempotent).
- Security: invalid/expired token → 401; suspended tenant → 403; staff calling manager endpoint → 403.

**Definition of Done**

- [ ] App starts with a generated dev keypair and a clean database.
- [ ] `POST /auth/login` returns a JWT; `/auth/me` works.
- [ ] `Tenant`/`User` are in `public`; business queries route to `tenant_<code>` (verified by SQL logging/test).
- [ ] Unknown tenant / non-ACTIVE tenant fails closed.
- [ ] `created_by` is populated for HTTP and `system` for background writes.
- [ ] Two-tenant isolation test passes for list, get-by-id, update, delete.
- [ ] `docker-compose` PostgreSQL has pgvector available.
- [ ] No dead tenant filters remain.

**Effort:** **L** — 7–9 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P0-T2
