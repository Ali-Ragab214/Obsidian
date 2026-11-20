# 01 — Project snapshot (read before EVERY task)

# StockSavvy — Spring Boot Implementation Plan

**Target:** the feature specification in sections A–G (personas, inventory/stock, approval/audit, RAG, four agents, end-to-end demo).
**Baseline scanned:** `main` working tree, 2026-11-19. Read-only scan; no source code was changed.
**Companion file:** the existing `README.md` is a prior, long-form audit. This plan is based on the actual code and focuses on the remaining build.

**Scope assumptions used below (all are open to confirmation in §9):** single warehouse/location; one currency per tenant/PO (no FX); vendor channel simulated (no email/API); no POS/sales integration (demand = OUT movements with `reason=SALE`); backend-only repository (UI is an open question); AI provider/embedding model pinned before Phase 4; Spring Boot 4.1.2 + Java 21 are fixed.

---

## 1. Executive Summary

- The codebase is an early-stage, package-by-layer Spring Boot monolith: Spring Boot 4.1.2, Java 21, JPA/Hibernate, Flyway, Spring Security + RS256 JWT, springdoc, PostgreSQL, Lombok.
- It **compiles** in the current working tree (`./mvnw -q -DskipTests compile` → exit 0), thanks to an uncommitted fix in `JwtAuthenticationFilter` (`TenantContext.clear()` → `clearCurrentTenant()`). It **cannot start**: both JWT PEM files are 0 bytes, and `db/migration/common/V1__init_Common_tables.sql` is 0 bytes, so `public.tenants` and `public.users` do not exist.
- Only `CategoryController` is exposed. `ProductServiceImpl` exists but has no controller; `StockMvtServiceImpl` is an empty class; there is no `AuthController`, `TenantController`, `UserController`, `ProductController`, or `StockMvtController`.
- **No** vendors, purchase orders, approval queue, agent run log, pgvector, RAG, schedulers, LLM dependency, Spring AI, or frontend exists (not found). All AI/agent features are ❌.
- Multi-tenant **scaffolding** is present (`tenant_id` JWT claim, `TenantContext`, `TenantSchemaResolver`, per-tenant Flyway provisioning, schema-per-tenant migration folder), but **isolation is not enforced**: no `CurrentTenantIdentifierResolver`, no `MultiTenantConnectionProvider`, no `search_path` switching; `hibernate.multi_tenancy: SCHEMA` is inert; the resolver fails open to `public`; the two tenant filters are commented out. Completing this wiring is the first blocking task. It is a completion of the existing design, **not** a redesign.
- Overall target-spec completion is **≈7%** (entities/DTOs/JWT/schema provisioning only). **0% of the end-to-end demo is executable today.**
- Critical path: **Phase 0 → 1 → 2 → 3 → 4 → 7** (foundation → stock → vendors/POs → approval/audit → RAG → negotiation). Forecasting and anomaly detection can run in parallel after their dependencies. Estimated **~10–12 weeks solo**, or **~8–9 weeks on the critical path** with forecasting/anomaly work parallelized.

| Area | Spec coverage today | Why |
|---|---|---|
| A. Personas / RBAC | ~10% | Role enum exists; no persona mapping, no enforcement, no vendor identity |
| B. Inventory & stock | ~10% | `Product`/`StockMvt` entities and migrations exist; no working stock service, controller, balance, vendors, or POs |
| C. Approval & audit | 0% | Nothing exists |
| D. RAG | 0% | Nothing exists; no pgvector/LLM dependency |
| E. Agents | 0% | Nothing exists; no scheduler/state machine |
| F/G. Demo & artifacts | 0% | No runnable workflow, no UI, no seed data |

---

## 2. Current Project Snapshot

### 2.1 Stack and build

| Item | Found | Evidence |
|---|---|---|
| Java | 21 | `pom.xml` `<java.version>` |
| Spring Boot | 4.1.2 | `pom.xml` parent |
| Web | Spring MVC | `spring-boot-starter-webmvc` |
| Persistence | Spring Data JPA / Hibernate | `spring-boot-starter-data-jpa`; `ddl-auto: none` |
| Migrations | Flyway | `spring-boot-starter-flyway`, `flyway-database-postgresql`; `db/migration/common`, `db/migration/tenant` |
| Security | Spring Security + JJWT 0.12.6 (RS256) | `SecurityConfig`, `JwtTokenService`, `JwtAuthenticationFilter` |
| Validation | Bean Validation | `spring-boot-starter-validation`; used only on `CategoryRequest`, `RegisterTenantRequest`, `UserRequest` |
| Docs | springdoc 3.2.0 | `pom.xml`; no `OpenAPI` bean |
| DB | PostgreSQL 17.5 | `docker-compose.yml` (no pgvector image) |
| Tests | One context-load test | `src/test/java/.../ApplicationTests.java` |
| AI/vector | **Not found** | No Spring AI, LangChain4j, OpenAI, Ollama, pgvector |
| Scheduling/async | **Not found** | No `@EnableScheduling`, `@Scheduled`, `@Async`, `@EnableAsync` |
| Caching | **Not found** | `@Cacheable` on `TenantSchemaResolver` is inert (no `@EnableCaching`) |
| Frontend | **Not found** | Backend only; Swagger is the only UI |

**Build/start status**

- `./mvnw -q -DskipTests compile` succeeds in the current tree (verified).
- `JwtTokenService.init()` loads `certs/private_key.pem` and `certs/public_key.pem`; both are **0 bytes** → startup fails.
- The common migration is **0 bytes** → `tenants`/`users` missing.
- `AuditorAware` is **not found**, while `created_by` is `NOT NULL` in the tenant migration → inserts fail.
- There is **no login endpoint**, so even after keys/migrations are fixed, no JWT can be obtained through the API.

> **Note on the existing README:** its claim that the project does not compile is stale for this working tree — `mvn compile` succeeds. Its other major findings (empty PEMs, empty common migration, no tenant routing, no login, no tests) are confirmed by this scan.

### 2.2 Architecture style and conventions

The project is **package-by-layer**:

```
controllers/   CategoryController (only REST controller)
services/      interfaces + services/impl/*Impl
repositories/  Spring Data JPA interfaces
entities/      JPA entities + enums
mappers/       hand-written @Component mappers
requests/      request DTOs
responses/     response DTOs
exceptions/    domain exceptions + GlobalExceptionHandler
config/        TenantContext, TenantSchemaResolver, BeansConfig, (disabled) tenant filters
security/      JWT filter/service, SecurityConfig
common/        AbstractEntity, PageResponse
properties/    @ConfigurationProperties
```

Conventions to reuse:

- Request/response DTOs are separate from entities; no entity is returned from a controller.
- Manual mappers (`CategoryMapper`, `ProductMapper`, `UserMapper`, `TenantMapper`) annotated `@Component`.
- `BasicService<I,O>` defines `create`, `update`, `findById`, `delete`, `findAll(page,size)`.
- `*ServiceImpl` uses `@Service`, `@RequiredArgsConstructor`, `@Slf4j`, and `jakarta.transaction.Transactional` (new code should prefer `org.springframework.transaction.annotation.Transactional` for `readOnly`/rollback control).
- Pagination uses `PageResponse<T>.of(Page<T>)`.
- Errors use `ErrorResponse` + `GlobalExceptionHandler`.
- IDs are UUID strings (`@GeneratedValue(strategy = GenerationType.UUID)`), not numeric.
- Tenant business tables live in `tenant_<company_code>` schemas; identity lives in `public`.

Gaps in conventions: no MapStruct, no RFC 9457 `ProblemDetail`, no OpenAPI security scheme, no page-size cap, RPC-style category paths (`/create`, `/update/{id}`, `/delete/{id}`), and soft-delete columns that are not consistently respected.

### 2.3 Current inventory (what actually exists)

| Layer | Classes | State |
|---|---|---|
| Entities | `Tenant`, `User`, `Category`, `Product`, `StockMvt`; enums `TenantStatus`, `UserRole`, `TypeMvt` | Present; `StockMvt` has `LocalDate dateMvt`, no timestamp/reason/source |
| Repositories | `TenantRepository`, `UserRepository`, `CategoryRepository`, `ProductRepository`, `StockMvtRepository` | Present; mostly `JpaRepository`; `StockMvtRepository` has no methods |
| Services | `TenantServiceImpl`, `UserServiceImpl`, `ProvisioningServiceImpl`, `CategoryServiceImpl`, `ProductServiceImpl`, `StockMvtServiceImpl` | `StockMvtServiceImpl` is empty; `registerTenant` never saves; `createAdminUserForTenant` is private and never called |
| Controllers | `CategoryController` | Only CRUD surface in the app |
| Security | `SecurityConfig`, `JwtTokenService`, `JwtAuthenticationFilter`, `UserRole` | No login endpoint; no `@PreAuthorize` anywhere; role claim is trusted |
| Migrations | common `V1` (empty), tenant `V1` (`categories`, `products`, `stock_mvts`) | No platform tables; no indexes beyond unique constraints |
| Tests | `ApplicationTests.contextLoads()` | No unit/integration/security/tenant tests |

`StockMvtServiceImpl` (all methods empty) is the clearest evidence that stock is not implemented. `ProductServiceImpl` works at the service level but is unreachable (no controller).

### 2.4 Multi-tenancy: intended design vs. enforced reality

**Intended model (keep it):**

1. `public` schema: `tenants`, `users`, platform identity.
2. `tenant_<company_code>` schema: all business data (`categories`, `products`, `stock_mvts`, and every future table).
3. JWT carries `tenant_id`; `JwtAuthenticationFilter` sets `TenantContext` (ThreadLocal) and resolves the schema via `TenantSchemaResolver`.
4. `ProvisioningServiceImpl` creates the schema and runs `db/migration/tenant` with programmatic Flyway on tenant approval.
5. Tenant lifecycle: `PENDING → ACTIVE → INACTIVE/SUSPENDED` (`TenantStatus`).

This is a legitimate schema-per-tenant (“silo”) design for inventory data. Keep it.

**What is actually enforced today:**

| Mechanism | Status | Evidence |
|---|---|---|
| `tenant_id` in JWT | ✅ present | `JwtTokenService.generateAccessToken` |
| ThreadLocal tenant/schema | 🟡 present | `TenantContext` |
| Schema resolution | 🟡 present, **fail-open** | `TenantSchemaResolver` returns `public` on any exception |
| Hibernate schema routing | ❌ **not found** | No `CurrentTenantIdentifierResolver`, no `MultiTenantConnectionProvider`, no `search_path` switching |
| `hibernate.multi_tenancy: SCHEMA` | ❌ inert | `application.yml`; Hibernate 6 needs resolver/provider beans, a property alone does nothing |
| Hibernate filter | ❌ disabled | `TenantHibernateFilter` is commented out and references an undefined `tenantFilter` |
| Header tenant filter | ❌ disabled | `TenantFilter` is commented out (good: tenant must come from JWT, not `X-TenantID`) |
| `tenant_id` column on business tables | ❌ absent | Tenant migration has no `tenant_id` (by design for schema-per-tenant) |
| Tenant status check | ❌ absent | Suspended tenant tokens still work |
| User `enabled`/`deleted` check on requests | ❌ absent | Filter trusts the token |
| Isolation tests | ❌ absent | No test asserts tenant A cannot read tenant B |
| Caching of schema resolution | ❌ inert | `@Cacheable` without `@EnableCaching` |

**Consequence:** if business tables are created in `public` (the default connection schema), every tenant query is unscoped. Adding vendors, POs, approvals, forecasts, or vectors before fixing routing would bake a cross-tenant data leak into the AI features. **Phase 0 is therefore non-negotiable.**

**Important scope note:** this is not a request to redesign multi-tenancy. The existing schema-per-tenant model, provisioning, JWT claim, and ThreadLocal are kept. Phase 0 adds the missing enforcement layer (resolver/provider, fail-closed resolution, status checks, migrations, login) and removes dead/inert code. A `git grep` across `main`, `dev`, and `feat/tenant-multi-tenancy` found no `CurrentTenantIdentifierResolver` or `MultiTenantConnectionProvider`; if you have a local branch/environment where these exist, reuse it and skip the corresponding Phase 0 items.

### 2.5 Request path and security as found

1. `JwtAuthenticationFilter` validates the Bearer token, reads `sub`, `tenant_id`, `role`, sets `TenantContext`, resolves the schema, and populates `SecurityContext`.
2. `SecurityConfig` is stateless, permits `/api/v1/auth/**` and Swagger, and requires authentication for everything else.
3. There is **no** `AuthController`, so no token can be issued. `JwtAuthenticationFilter` skips `/api/v1/auth/login`, but the endpoint does not exist.
4. `@EnableMethodSecurity` is present, but **no** `@PreAuthorize`/`@Secured` exists (not found). Any authenticated user can call any endpoint.
5. `JwtAuthenticationFilter` clears `TenantContext` after the chain but **not in a `finally` block** → ThreadLocal leak if a downstream filter throws.
6. The current working tree contains an uncommitted change to `JwtAuthenticationFilter` (kept; not modified by this plan).

---

## Shared conventions for every task (from §6)

**Shared conventions for every spec below**

- New entities extend `AbstractEntity` (UUID id, audit columns, soft-delete flag) unless the table is append-only (then still extend it for `created_at`/`created_by`, but never update).
- New repositories are Spring Data JPA interfaces in `repositories/`.
- New DTOs go in `requests/`/`responses/`; manual mappers in `mappers/`; controllers in `controllers/`; services in `services/` + `services/impl/`.
- Agent-specific code lives under a new `agents/` subtree (`agents/common`, `agents/forecast`, `agents/reorder`, `agents/negotiation`, `agents/anomaly`, `agents/rag`) because each agent owns a prompt + state machine. This extends the existing layer packages rather than reorganizing them.
- New API paths are REST-style under `/api/v1`; category endpoints remain as-is.
- All business tables go into the tenant schema, created by a new `db/migration/tenant/V*.sql` migration. `public` holds only identity/platform data.
- Every new tenant migration must also be applied to existing tenants via `TenantMigrationService` (see 6.1).
- Tests use JUnit 5 + Spring Boot Test + Testcontainers (PostgreSQL with pgvector). No test calls a real LLM; `LlmGateway` has a deterministic stub.

**Why four separate agents (spec E.5):** each agent owns one responsibility, one prompt set, one state/failure model, and one review surface: forecast (numeric, no approval), reorder (draft + spend approval), negotiation (multi-step + per-message approval), anomaly (rules + flag-only). A single “inventory agent” would mix a statistical job, a spend workflow, an external conversation, and a fraud review into one prompt/state machine, making failures hard to explain and the demo hard to bound. The shared `ApprovalRequest`/`AgentRunLog`/`LlmGateway` is the only cross-cutting code; agent logic stays in `agents/<name>`.

---
