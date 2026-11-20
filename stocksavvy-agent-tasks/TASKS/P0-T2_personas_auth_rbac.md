# P0-T2 — Personas, authentication & RBAC

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 0 — Runnable foundation + tenant isolation |
| Source section | `6.2` of the original plan |
| Depends on (spec) | P0-T1 (§6.1) (login, roles, tenant status). |
| Tasks that must be done before this one | P0-T1 |
| Effort | **M** — 2–3 days (mostly wiring tests). |
| Next task | P1-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P0-T2 — "Personas, authentication & RBAC" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Do not start before P0-T1 passes its two-tenant isolation test.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| A1 | Manager persona (Mona) | 🟡 | No `MANAGER`/persona role; `ROLE_COMPANY_ADMIN`/`ROLE_ADMINISTRATOR` exist but no permission checks. No dashboard. | Approvals, negotiation targets, anomaly review |
| A2 | Staff persona (Sam) | 🟡 | `ROLE_USER`/`ROLE_SALES_OPERATOR` exist; no stock-posting permission, no receipt/count endpoint, no discrepancy flag endpoint. | Stock counts, deliveries, shelf discrepancies |
| A3 | Vendor external persona | ❌ | No vendor user/portal; no simulation. Vendors module itself is missing. | Counter-offers (simulated) |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 13 | P1 | No authorization rules | No `@PreAuthorize`/`@Secured` anywhere | Add `@ManagerOnly`, `@StaffOnly`, `@PlatformAdminOnly` meta-annotations and enforce | 0/1 |
| 14 | P1 | Tenant status/user status not checked | Filter only validates signature; `enabled`/`deleted` ignored | Resolver returns descriptor (schema, status); reject non-ACTIVE; check user on login | 0 |

## 🧭 Suggested internal order

1. Migration: add/extend `db/migration/tenant/V*.sql` exactly as in "Entities / tables" (indexes and constraints included).
2. Entities / enums + repositories (tenant-schema scoped, soft-delete aware where relevant).
3. Service layer with the business rules and the **HITL / safety** rules from the spec (enforce in the service, not only the controller).
4. Request/response DTOs, mapper, controller, role annotations (`@ManagerOnly` / `@StaffOnly`) per the endpoint table.
5. Multi-tenancy checks from the spec (explicit tenant context for jobs/async, validated schema identifiers).
6. Tests from the "Testing" list (unit + integration, Testcontainers, stub LLM).
7. Walk the DoD checklist, update `PROGRESS.md` and `HANDOFF_LOG.md`.

---

## 📘 SPEC (verbatim from the plan — do not skip any bullet)

### 6.2 Personas, authentication, RBAC (A1–A3)

**Goal:** make “Manager Mona”, “Staff Sam”, and the simulated vendor persona explicit in authorization, without building a vendor portal.

**Spec coverage:** A1 (Manager), A2 (Staff), A3 (Vendor, simulated).

**Depends on:** 6.1 (login, roles, tenant status).

**Spring Boot approach**

- Extend `UserRole` with `ROLE_MANAGER` and `ROLE_STAFF`; keep existing values for platform/legacy.
- Map privileges:
  - **Manager:** `ROLE_COMPANY_ADMIN`, `ROLE_ADMINISTRATOR`, `ROLE_MANAGER` → approvals, negotiation targets, anomaly review, dashboard, vendor/PO management, forecasts.
  - **Staff:** `ROLE_STAFF`, `ROLE_SALES_OPERATOR`, `ROLE_USER` → post receipts/adjustments, view product/movement history, create a manual discrepancy flag.
  - **Platform admin:** `ROLE_PLATFORM_ADMIN` → tenant lifecycle only.
- Use meta-annotations `@ManagerOnly`, `@StaffOnly`, `@PlatformAdminOnly` (each `@PreAuthorize`) instead of repeating SpEL. Trade-off vs a full permission model: simple, readable, enough for two personas; a permission table is overkill for the thesis.
- **Vendor persona:** no login, no `User` row. Counter-offers come from `SimulatedVendorResponder` (6.13). Document this explicitly in the thesis as a simulation boundary.
- Login flow: find tenant by `companyCode` → check ACTIVE → find user by `(tenantId, username)` → check `enabled`/`deleted` → BCrypt matches → issue JWT. Use a direct `AuthService` rather than a global `AuthenticationManager` because usernames are tenant-scoped. Trade-off: bypasses the provider chain but is explicit and auditable.

**Entities / tables**

- No new tables. `users.role` already exists; add enum values (no migration needed for a varchar column, but the migration check constraint, if any, must allow them).
- Add `users.last_login_at` (optional, P2).

**Classes**

- Modify `entities/UserRole.java`; add `security/ManagerOnly.java`, `StaffOnly.java`, `PlatformAdminOnly.java`, `security/Roles.java`.
- Add `@PreAuthorize` to new controllers; retrofit `CategoryController`/future `ProductController` with manager-only writes and authenticated reads.
- `JwtAuthenticationFilter`: check tenant status via the cached descriptor; clear context in `finally`; reject disabled/deleted users (lookup by id or include a token version — a DB lookup per request is acceptable for the thesis).
- `SecurityConfig`: JSON `AuthenticationEntryPoint`/`AccessDeniedHandler`; keep Swagger/login public.

**API endpoints**

No new persona-specific endpoints beyond 6.1; the persona is enforced on every endpoint below via roles.

**Multi-tenancy considerations**

- Role is a property of a tenant user, not a platform-wide claim. A token is only valid for its `tenant_id`; the filter never trusts a role string without a valid tenant descriptor.
- A vendor has no tenant identity and cannot call the API.

**HITL / safety**

- Manager-only approval/review endpoints; staff cannot approve spend or resolve anomalies.
- No self-escalation: a staff user cannot change their own role (manager-only user management).

**Testing**

- Parameterized tests: each endpoint × {manager, staff, other tenant, anonymous} → expected 200/403/401.
- Suspended tenant token → 403; disabled user → 401/403.

**DoD**

- [ ] Manager and staff demo users exist and have different permissions.
- [ ] Every mutating endpoint has an explicit role rule.
- [ ] Vendor persona is documented as simulated (no login).
- [ ] 401/403 responses use the standard error shape.

**Effort:** **M** — 2–3 days (mostly wiring tests).

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P1-T1
