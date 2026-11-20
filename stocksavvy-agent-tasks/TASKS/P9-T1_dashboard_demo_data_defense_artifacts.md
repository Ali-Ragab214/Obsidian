# P9-T1 — Dashboard, demo data & defense artifacts

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 9 — Dashboard, demo data, defense artifacts |
| Source section | `6.15` of the original plan |
| Depends on (spec) | all previous phases (can start the dashboard earlier with placeholders, but finalize last). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1, P7-T1, P8-T1 |
| Effort | **M** — 4–5 days. |
| Next task | — (last task: run the full demo) |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P9-T1 — "Dashboard, demo data & defense artifacts" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1, P7-T1, P8-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Integration phase. Includes the seeder/reset guard, aggregate dashboard, OpenAPI polish and the four defense artifacts.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| F | End-to-end demo workflow (8 steps) | ❌ | None of the 8 steps is executable. | Committee demo |
| G1 | Approval queue artifact (pending negotiation + reasoning) | ❌ | No approval queue. | Defense artifact |
| G2 | Full negotiation transcript artifact | ❌ | No `AgentRunLog`/messages. | Defense artifact |
| G3 | RAG answer with visible sources artifact | ❌ | No RAG. | Defense artifact |
| G4 | Anomaly flag with explanation artifact | ❌ | No anomaly agent. | Defense artifact |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 19 | P2 | RPC-style category API | `/create`, `/update/{id}`, `/delete/{id}` | Keep for compatibility; use REST style for all new endpoints; optionally refactor later | 9 |
| 20 | P2 | `show-sql: true`, CORS `*`, no Actuator/OpenAPI scheme | `application.yml`, `SecurityConfig` | Disable SQL logging outside dev; CORS allowlist; add OpenAPI bearer scheme; optional Actuator | 9 |

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

### 6.15 Dashboard, demo data & defense artifacts (F, G)

**Goal:** expose the morning dashboard, seed a repeatable demo tenant, and make the four defense artifacts easy to show.

**Spec coverage:** F (end-to-end demo), G (defense artifacts), plus E5 (agent separation rationale, see §6 intro).

**Depends on:** all previous phases (can start the dashboard earlier with placeholders, but finalize last).

**Spring Boot approach**

- **Dashboard service:** aggregate read-only queries in the tenant schema:
  - low-stock count + top N items (6.5);
  - latest forecasts for those items (6.11);
  - pending approvals count + latest (6.8);
  - open anomaly count + latest (6.14);
  - active/escalated negotiations (6.13);
  - recent agent runs (6.8).
  Return one `GET /api/v1/dashboard` response for the “morning” screen, plus the existing list endpoints for drill-down.
- **Demo seeder:** `@Profile("demo") ApplicationRunner` (idempotent):
  - creates/uses a tenant `demo` (`companyCode=demo`, timezone, currency), provisions the schema;
  - creates manager `mona` and staff `sam` (BCrypt), plus a platform admin if needed;
  - creates categories, products including `SKU-118` (on hand 4, reorder point 10) and `SKU-204` (anomaly candidate);
  - creates vendors (e.g., Acme, Beta) with catalog prices/lead times and negotiation profiles;
  - seeds 60–90 days of OUT `SALE` movements for SKU-118 and a few low-stock balances;
  - seeds an out-of-hours adjustment on SKU-204 for the anomaly demo;
  - optionally runs the forecast/reorder agents once to leave a pending approval.
  - Use fixed IDs/company code; do not duplicate on restart.
- **Demo reset:** `POST /api/v1/demo/reset` (demo profile + platform admin only) truncates the demo tenant’s business tables and re-seeds. Guard it strictly; never enable in production.
- **Artifact endpoints:** they already exist; add convenience DTOs:
  - G1: `GET /approvals?status=PENDING` (payload + reasoning + target).
  - G2: `GET /negotiations/{id}/transcript` (messages + approvals + run logs).
  - G3: `POST /ai/chat` (answer + sources) + `GET /ai/chat/{queryId}`.
  - G4: `GET /anomalies?status=OPEN` + `GET /anomalies/{id}` (explanation + movement ids).
- **UI decision:** no frontend exists. For the defense, either (a) use Swagger UI + a prepared Postman/curl script, or (b) add a thin Thymeleaf “demo console” with four pages (dashboard, approvals, negotiation transcript, anomalies). A React app is a separate deliverable. See open questions.
- **OpenAPI:** add `@OpenAPIDefinition` (title, version, description), bearer JWT security scheme, tags per module, and example requests. This makes the API presentable without a UI.
- **Observability/polish:** add Actuator health (optional), disable `show-sql` outside dev, add a correlation id to logs/errors, cap page size, and add a `demo` profile to `application.yml`.
- **Trade-off:** a full SPA is out of scope for a backend implementation plan; a thin server-rendered console is the fastest path to a convincing defense if the committee expects screens.

**Entities / tables**

- No new business tables. `demo` profile + seeder; optionally a `demo_seed_state` table to track seeding (fixed IDs are enough).

**Classes**

- `services/DashboardService.java` + impl; `controllers/DashboardController.java`; `responses/DashboardResponse.java`.
- `config/DemoDataSeeder.java` (`@Profile("demo")`), `controllers/DemoController.java` (reset, demo profile only).
- `config/OpenApiConfig.java`; `config/WebConfig.java` (correlation id filter/page cap).
- `resources/application-demo.yml`; `resources/prompts/*` (already added).
- Optional `src/main/resources/templates/demo/*.html` + `spring-boot-starter-thymeleaf` if the thin console is chosen.
- Test fixtures / `demo` SQL script.

**API endpoints**

| Method | Path | Response | Roles |
|---|---|---|---|
| `GET` | `/api/v1/dashboard` | low stock + forecasts + pending approvals + anomalies + negotiations | Manager |
| `GET` | `/api/v1/dashboard/activity` | recent runs/approvals/flags | Manager |
| `POST` | `/api/v1/demo/reset` | reset summary | Platform admin (demo profile only) |

**Multi-tenancy considerations**

- The dashboard is tenant-scoped; one request = one tenant.
- The seeder runs under the demo tenant context; never seeds across tenants.
- The reset endpoint must be impossible to call in non-demo profiles (profile + role + explicit property).

**HITL / safety**

- Dashboard is read-only.
- Demo reset is destructive and guarded; document it as non-production.
- Artifacts expose only the current tenant’s data.

**Testing**

- Dashboard aggregate with seeded data returns the expected counts.
- Seeder is idempotent (run twice → same data, no duplicates).
- Demo reset only works in the demo profile.
- OpenAPI includes the bearer scheme and all tags.
- End-to-end demo test (see §8) runs headlessly against the seeded tenant.

**DoD**

- [ ] One call returns the morning dashboard with SKU-118 low + forecast.
- [ ] Seeder creates a complete, repeatable demo in < 60 seconds.
- [ ] All four artifacts are reachable and readable.
- [ ] Swagger has a bearer “Authorize” button and organized tags.
- [ ] The 8-step demo runs end-to-end without manual DB edits.

**Effort:** **M** — 4–5 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: — (last task: run the full demo)
