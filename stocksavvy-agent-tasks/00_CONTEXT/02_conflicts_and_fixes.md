# 02 — Conflicts & refactoring (all 24 items)

Each item is also repeated inside the task file that must fix it. Tick them in `PROGRESS.md`.

## 4. Conflicts & Refactoring Needed

Severity: **P0** = blocks startup/isolation/demo; **P1** = correctness or safety; **P2** = hygiene.

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 1 | P0 | Tenant isolation not enforced | No `CurrentTenantIdentifierResolver`/`MultiTenantConnectionProvider`; `TenantSchemaResolver` fails open; `hibernate.multi_tenancy: SCHEMA` inert | Add Hibernate resolver/provider with `search_path`; fail closed; add isolation tests. Keep schema-per-tenant. | 0 |
| 2 | P0 | Platform tables missing | `db/migration/common/V1__init_Common_tables.sql` is 0 bytes | Add `V2__create_platform_tables.sql` (do not edit an applied V1); create `tenants`, `users` with tenant-scoped uniqueness | 0 |
| 3 | P0 | App cannot start / no login | Both PEMs are 0 bytes; no `AuthController` (not found) | Generate keys outside VCS (or provide a dev keypair); add `POST /api/v1/auth/login` | 0 |
| 4 | P0 | `created_by` is `NOT NULL`, no auditor | `AbstractEntity.@CreatedBy`; no `AuditorAware` (not found) | Add `AuditorAware<String>` (security user, else `system`); set `updated_by` properly | 0 |
| 5 | P0 | Ledger contract allows mutation | `StockMvtService extends BasicService` (exposes update/delete); `StockMvtServiceImpl` empty | Do not extend `BasicService` for stock; add append-only `StockLedgerService`; optional DB trigger to reject UPDATE/DELETE | 1 |
| 6 | P0 | Docker DB lacks pgvector | `docker-compose.yml` uses `postgres:17.5` | Switch to a pgvector-enabled image (e.g., `pgvector/pgvector:pg17`) or install the extension; run `CREATE EXTENSION vector` once | 0/4 |
| 7 | P1 | `StockMvt.dateMvt` is `LocalDate` | Entity + tenant migration | Add `occurred_at timestamptz` (business date stays `date_mvt`); anomaly rules use `occurred_at` | 1 |
| 8 | P1 | Product update is broken | `ProductServiceImpl.update` always calls `checkIfProductAlreadyExistsByReference`; rebuilds a new `Product` and sets only `id` | Load existing entity, exclude self from duplicate check, update fields; guard category null | 1 |
| 9 | P1 | `ProductResponse` incomplete | No `id`/`categoryId`; `category.getName()` NPEs | Add `id`, `categoryId`; null-safe mapper | 1 |
| 10 | P1 | Tenant registration/approval inconsistent | `registerTenant` never saves; `createAdminUserForTenant` never called; `approveTenant` sets ACTIVE before provisioning and swallows errors; `activateTenant` skips provisioning; `UserServiceImpl.createUser` rejects `ROLE_USER` with a misleading “platform admin” message | Save PENDING; provision then ACTIVE; create admin user; add `provisioning_error`; fix role validation; add controllers | 0 |
| 11 | P1 | Global exception mapping incomplete | `BusinessException`, `EntityNotFoundException`, `DuplicateKeyException`, `IllegalStateException`, malformed JSON → HTTP 500 | Add handlers; return 400/401/403/404/409; add JSON security entry point/denied handler | 0 |
| 12 | P1 | Password hashes exposed | `UserMapper.toResponse` copies `password`; `TenantResponse` declares `adminPassword` | Remove both fields from responses | 0 |
| 13 | P1 | No authorization rules | No `@PreAuthorize`/`@Secured` anywhere | Add `@ManagerOnly`, `@StaffOnly`, `@PlatformAdminOnly` meta-annotations and enforce | 0/1 |
| 14 | P1 | Tenant status/user status not checked | Filter only validates signature; `enabled`/`deleted` ignored | Resolver returns descriptor (schema, status); reject non-ACTIVE; check user on login | 0 |
| 15 | P1 | No optimistic locking on stock | No `@Version`/`@Lock`; balances will lose updates | `stock_balances.version` + `SELECT ... FOR UPDATE` (or atomic update) in the ledger transaction | 1 |
| 16 | P1 | No indexes for planned queries | Tenant V1 has only unique constraints | Add composite indexes per table (listed in each spec) | 1–8 |
| 17 | P2 | Soft delete inconsistent | `deleted` columns exist; category/product delete is hard; `findAll` includes deleted | Standardize soft delete for master data; exclude `deleted` in queries | 1 |
| 18 | P2 | Inert/dead tenant code | `TenantFilter`, `TenantHibernateFilter` commented out; `@Cacheable` without `@EnableCaching` | Delete dead filters; enable a simple cache; evict on tenant change | 0 |
| 19 | P2 | RPC-style category API | `/create`, `/update/{id}`, `/delete/{id}` | Keep for compatibility; use REST style for all new endpoints; optionally refactor later | 9 |
| 20 | P2 | `show-sql: true`, CORS `*`, no Actuator/OpenAPI scheme | `application.yml`, `SecurityConfig` | Disable SQL logging outside dev; CORS allowlist; add OpenAPI bearer scheme; optional Actuator | 9 |
| 21 | P2 | Package name typo `saas_multi_tennant_app` | `src/main/java/...` | Do **not** rename (high churn, no functional gain); note for future | — |
| 22 | P2 | No tests | Only `contextLoads` | Add unit/slice/integration/tenant-isolation tests (see each spec) | 0+ |
| 23 | P2 | `UserRepository.findByUsername` is global; `username`/`email` unique globally; `TenantRepository.ExistsByCompanyCode` has non-standard capitalization | Entity + repo | Make unique `(tenant_id, username)` and `(tenant_id, email)`; login uses company code + username; rename to `existsByCompanyCode` | 0 |
| 24 | P2 | Flyway V1 empty may already be applied | `flyway_schema_history` | Never edit applied V1; add V2+ (or clean the dev DB once, documented) | 0 |
