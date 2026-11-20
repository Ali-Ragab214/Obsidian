# P3-T1 — Approval queue & agent audit trail

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 3 — Approval queue + agent audit trail |
| Source section | `6.8` of the original plan |
| Depends on (spec) | P0-T2 (§6.2) (roles), P2-T2 (§6.7) (first concrete handler); can be developed in parallel with P2-T1 (§6.6). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2 |
| Effort | **M** — 3–4 days. |
| Next task | P4-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P3-T1 — "Approval queue & agent audit trail" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Prove the shared HITL mechanism with the PO handler. This mechanism is reused by every agent later.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| C1 | `ApprovalRequest` shared queue | ❌ | Not found. | Human-in-the-loop for every agent |
| C2 | `AgentRunLog` audit trail | ❌ | Not found. | Defense artifact: full transcript |
| C3 | Shared “log every step / approve every action” rule | ❌ | No agent infrastructure exists. | Architectural HITL property |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 16 | P1 | No indexes for planned queries | Tenant V1 has only unique constraints | Add composite indexes per table (listed in each spec) | 1–8 |

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

### 6.8 Approval queue & agent audit trail (C1–C3)

**Goal:** implement the single reusable human-in-the-loop mechanism: every agent logs every step to `AgentRunLog`, and every action that needs a human creates an `ApprovalRequest` that a manager can approve, edit, or reject.

**Spec coverage:** C1 (`ApprovalRequest`), C2 (`AgentRunLog`), C3 (shared log/approve rule).

**Depends on:** 6.2 (roles), 6.7 (first concrete handler); can be developed in parallel with 6.6.

**Spring Boot approach**

- **Data model close to the spec** plus a small parent run and processing fields for operability:
  - `AgentRun`: one row per execution (`runId`), status, trigger, timestamps.
  - `AgentRunLog`: append-only steps (`agentType`, `runId`, `stepNumber`, `input`, `output`, `timestamp`, `linkedApprovalId`). The `created_at` column is the timestamp (exposed as `timestamp` in the DTO) so the entity can reuse `AbstractEntity` auditing.
  - `ApprovalRequest`: `agentType`, `runId`, `stepNumber`, `payload` (JSONB), `reasoning`, `status`, `reviewedBy`, `reviewedAt`, plus `reviewedPayload` (edits) and `reviewNote`.
- **JSON columns:** use Hibernate 6 `@JdbcTypeCode(SqlTypes.JSON)` with `JsonNode` for `payload`/`input`/`output`. Trade-off vs `TEXT`: JSONB is queryable and validates; it requires the PostgreSQL JSON support already present in Hibernate 6. If it causes friction, fall back to `TEXT` + `ObjectMapper` (still works, less queryable).
- **Handler registry:** each agent provides an `ApprovalHandler` bean (`supports(AgentType)` / `handle(ApprovalRequest)`). `ApprovalService` injects `List<ApprovalHandler>` and dispatches by `agentType`. This is simpler and more testable than Spring StateMachine for the approval side.
- **After-commit dispatch:** approving commits the decision, then publishes `ApprovalResolvedEvent`; a `@TransactionalEventListener(AFTER_COMMIT)` calls the handler. This prevents an LLM/PO step from running inside the approval transaction and losing the decision on failure.
- **Idempotency:** `ApprovalRequest` has `@Version` and a status guard; a second approve/reject returns 409. A unique partial index on `(run_id, step_number)` where `status='PENDING'` prevents duplicate pending approvals for the same step.
- **Failure visibility:** add `processed boolean`, `processing_error text`; if the handler fails, the approval remains APPROVED but the dashboard shows “approved, processing failed” and the error is in `AgentRunLog`. Do not silently swallow.
- **Run/log API:** `AgentRunService.startRun/logStep/completeRun/failRun`; `logStep` uses `REQUIRES_NEW` so audit survives a failed business transaction. Every agent must call it at every state transition; add tests that assert the expected number of steps.
- **Trade-off:** Spring StateMachine would add a dependency, configuration, and persistence for little gain; a manual enum + guarded transitions is easier to explain to a committee. Use Spring StateMachine only if the negotiation state machine becomes unmanageable (it should not at 8 states).

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `agent_runs` | `id` (runId UUID), `agent_type`, `status` (`RUNNING/COMPLETED/FAILED/WAITING_APPROVAL/ESCALATED`), `trigger`, `started_at`, `finished_at`, `summary`, `error` | PK id; index `(agent_type, started_at desc)`, `(status)` |
| `agent_run_logs` | `id`, `agent_type`, `run_id`, `step_number`, `step_name`, `input jsonb`, `output jsonb`, `status`, `error`, `linked_approval_id`, `created_at`, `created_by` | index `(run_id, step_number)`, `(agent_type, created_at desc)`, `(linked_approval_id)`; append-only |
| `approval_requests` | `id`, `agent_type`, `run_id`, `step_number`, `payload jsonb`, `reasoning text`, `status` (`PENDING/APPROVED/REJECTED/CANCELLED`), `reviewed_by`, `reviewed_at`, `reviewed_payload jsonb`, `review_note`, `target_type`, `target_id`, `processed`, `processing_error`, `version`, audit | index `(status, agent_type, created_at)`, `(run_id)`, `(target_type, target_id)`; partial unique pending per `(run_id, step_number)` |

**Classes**

- `agents/common/AgentType.java`, `AgentRunStatus.java`, `ApprovalStatus.java`.
- `entities/AgentRun.java`, `AgentRunLog.java`, `ApprovalRequest.java`.
- `repositories/AgentRunRepository.java`, `AgentRunLogRepository.java`, `ApprovalRequestRepository.java`.
- `services/AgentRunService.java` + impl; `services/ApprovalService.java` + impl; `services/ApprovalDispatcher.java`.
- `agents/common/ApprovalHandler.java`, `ApprovalResolvedEvent.java`, `AgentRunContext.java` (step counter helper).
- `controllers/ApprovalController.java`, `controllers/AgentRunController.java`.
- `requests/ApprovalDecisionRequest.java` (`{decision, editedPayload?, note?}`); `responses/ApprovalResponse.java`, `AgentRunResponse.java`, `AgentRunLogResponse.java`.
- `db/migration/tenant/V4__agent_infrastructure.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `GET` | `/api/v1/approvals?status=PENDING&agentType=&page=&size=` | — | `PageResponse<ApprovalResponse>` (payload + reasoning + run) | Manager |
| `GET` | `/api/v1/approvals/{id}` | — | approval + linked run/logs + target summary | Manager |
| `POST` | `/api/v1/approvals/{id}/approve` | `{editedPayload?, note?}` | updated approval; handler result | Manager |
| `POST` | `/api/v1/approvals/{id}/reject` | `{note}` | updated approval | Manager |
| `GET` | `/api/v1/agent-runs?agentType=&status=&page=&size=` | — | `PageResponse<AgentRunResponse>` | Manager |
| `GET` | `/api/v1/agent-runs/{runId}` | — | run + ordered logs + approvals | Manager |
| `GET` | `/api/v1/agent-runs/{runId}/logs` | — | ordered log entries | Manager |

**Multi-tenancy considerations**

- All three tables live in the tenant schema. A manager only sees their tenant’s queue.
- Background jobs create approvals under the tenant context; `runId` is globally unique but resolved within the schema.
- `reviewed_by` stores the user id; display name is resolved from `public.users` (no cross-schema FK, to keep tenant schemas self-contained). Document this trade-off.
- Never accept a tenant id in the payload; the tenant is the schema of the authenticated request.

**HITL / safety**

- This is the shared HITL mechanism. Rule: **every agent step writes an `AgentRunLog`; every outbound/committing action creates an `ApprovalRequest` and waits for a manager decision.**
- Forecast and anomaly agents do not create approvals (informational/flag-only); they still log runs/steps.
- Only a manager can approve/reject. Staff can view their own actions but not the queue (or read-only if desired).
- Approval edits are stored in `reviewedPayload`; handlers must use it, not the original payload.

**Testing**

- Approve → handler called exactly once after commit; double approve → 409.
- Reject → handler called with rejection; target moves to the defined state.
- Handler failure → approval remains APPROVED with `processing_error`; run log contains the error.
- Run/log ordering and step numbering; every agent integration test asserts the expected log steps.
- Cross-tenant approval isolation.
- Staff calling approve → 403.

**DoD**

- [ ] A pending PO approval is visible with payload and reasoning.
- [ ] Approve/reject/edit works and is audit-logged.
- [ ] Every run can be replayed from its logs.
- [ ] No agent can bypass the approval service to commit spend.
- [ ] Cross-tenant isolation test.

**Effort:** **M** — 3–4 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P4-T1
