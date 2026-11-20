# P8-T1 — Anomaly / Fraud Detection Agent

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 8 — Anomaly agent |
| Source section | `6.14` of the original plan |
| Depends on (spec) | P1-T2 (§6.4) (ledger with timestamps/reasons), P3-T1 (§6.8) (audit logging); independent of RAG/negotiation. |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1, P7-T1 |
| Effort | **M** — 3–4 days. |
| Next task | P9-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P8-T1 — "Anomaly / Fraud Detection Agent" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1, P7-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Strictly flag-only: this agent must NEVER write to stock data and NEVER block a user action. Add a test proving it.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| E4 | Anomaly / Fraud Detection Agent | ❌ | No flags, rules, explanation, or review screen. | Suspicious movements, never auto-corrected |
| E5 | Four separate agents | ❌ | N/A until built; plan keeps each agent isolated. | Explainable responsibilities |

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

### 6.14 Anomaly / Fraud Detection Agent (E4)

**Goal:** flag suspicious stock movements with a plain-language explanation on a review screen, and never modify stock or block a user action.

**Depends on:** 6.4 (ledger with timestamps/reasons), 6.8 (audit logging); independent of RAG/negotiation.

**Spring Boot approach**

- **Rule engine, not ML:** `AnomalyRule` interface with small, unit-testable implementations. `AnomalyDetectionService` runs all enabled rules and collects `FlagCandidate`s.
- **Rules (MVP):**
  1. `OUTSIDE_BUSINESS_HOURS`: an OUT movement (or adjustment) whose `occurred_at` local time is outside `[08:00, 18:00]` (configurable, tenant timezone). Severity medium.
  2. `QUANTITY_OUTLIER`: |quantity| > mean + 3×stddev of the same product/type over the last 30 days, with ≥ 5 samples (exclude the current movement). Severity high.
  3. `REPEATED_SMALL_ADJUSTMENTS`: ≥ 3 OUT `COUNT_ADJUSTMENT`/`MANUAL_ADJUSTMENT` movements with quantity ≤ 5 on the same SKU within 7 days. Severity medium.
  4. `SALES_STOCK_MISMATCH`: **conditional**; see open question. The project has no sales/POS module, so this rule cannot be implemented meaningfully without an imported sales source. Options: (a) add a minimal `sales_records` import (CSV) and compare daily sales to ledger OUT `SALE`; (b) define “recorded sales” as OUT movements with `reason=SALE` and compare to stock delta — tautological; (c) defer. Recommended: implement R1–R3 for the thesis, document R4 as future work, and optionally add a CSV sales import if the committee expects it. Do not fake it.
- **Explanation layer:** a deterministic template always explains the numbers and related movement ids (e.g., “3 small OUT adjustments on SKU-118 in the last 7 days (IDs …), total 12 units, no matching SALE movements”). Optional LLM polish; store `explanation_source`. The LLM cannot change severity, rule code, or data.
- **Trigger:**
  - `@Scheduled` hourly/daily per tenant (`TenantJobRunner`).
  - After a manual stock adjustment: publish `StockMovementPostedEvent` and handle it with `@TransactionalEventListener(AFTER_COMMIT)`; scan only that product. This guarantees the anomaly scan never runs inside or rolls back the stock transaction.
- **Deduplication:** `dedup_key` unique per rule + product + bucket; if an OPEN flag already exists, skip/update the explanation rather than creating duplicates. This prevents a noisy 24/7 scanner.
- **Review workflow:** `OPEN → ACKNOWLEDGED → RESOLVED` or `DISMISSED`, manager-only; `resolution_note` required for resolve/dismiss. **No stock mutation endpoint is called by this agent.** Use a read-only service/repository for the scan; the only writes are `anomaly_flags` and `agent_run_logs`.
- **Manual discrepancy (staff persona):** `POST /api/v1/anomalies/manual` creates a `MANUAL_REPORT` flag with the staff note; this covers Sam’s “flag discrepancies at the shelf” without building a separate module.
- **Trade-off:** a statistical/ML anomaly model is out of scope; rules are explainable and defensible, which is exactly what the spec asks for.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `anomaly_flags` | `id`, `product_id`, `rule_code`, `severity`, `status` (`OPEN/ACKNOWLEDGED/RESOLVED/DISMISSED`), `detected_at`, `window_start`, `window_end`, `title`, `explanation text`, `explanation_source`, `related_movement_ids jsonb`, `dedup_key`, `run_id`, `resolved_by`, `resolved_at`, `resolution_note`, audit | unique `dedup_key`; index `(status, detected_at desc)`, `(product_id, detected_at desc)`, `(rule_code)` |

**Classes**

- `agents/anomaly/AnomalyRule.java`, `OutsideBusinessHoursRule.java`, `QuantityOutlierRule.java`, `RepeatedSmallAdjustmentsRule.java` (and optional `SalesMismatchRule`).
- `agents/anomaly/AnomalyDetectionService.java` + impl, `AnomalyExplanationService.java`, `AnomalyFlag.java` entity.
- `repositories/AnomalyFlagRepository.java`, read-only movement query methods.
- `controllers/AnomalyController.java`; `requests/ResolveAnomalyRequest.java`, `ManualDiscrepancyRequest.java`; `responses/AnomalyFlagResponse.java`.
- `agents/anomaly/AnomalyJob.java` (`@Scheduled` + `TenantJobRunner`); `agents/anomaly/StockMovementAnomalyListener.java` (`@TransactionalEventListener`).
- `db/migration/tenant/V8__anomaly_flags.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `GET` | `/api/v1/anomalies?status=OPEN&severity=&productId=&page=&size=` | — | `PageResponse<AnomalyFlagResponse>` | Manager |
| `GET` | `/api/v1/anomalies/{id}` | — | flag + explanation + related movements | Manager |
| `POST` | `/api/v1/anomalies/{id}/acknowledge` | `{note?}` | updated flag | Manager |
| `POST` | `/api/v1/anomalies/{id}/resolve` | `{note}` | updated flag | Manager |
| `POST` | `/api/v1/anomalies/{id}/dismiss` | `{note}` | updated flag | Manager |
| `POST` | `/api/v1/anomalies/manual` | `{productId, observedQty?, note}` | MANUAL_REPORT flag | Staff, Manager |
| `POST` | `/api/v1/agents/anomaly/run` | `{productId?}` | run summary | Manager |

**Multi-tenancy considerations**

- Flags are tenant-local; the scheduled scan iterates tenants explicitly.
- Business hours/timezone come from the tenant descriptor; do not use the server timezone.
- The after-commit listener captures the tenant id before the async boundary (if async is used); prefer synchronous after-commit for the MVP.
- No cross-tenant baseline: statistical thresholds are computed per tenant/product.

**HITL / safety**

- **Strictly flag-only.** The agent has no dependency on stock write methods; the scan is `@Transactional(readOnly = true)` except for flag/log writes.
- Never blocks a stock posting: listener exceptions are caught/logged; the business transaction is already committed.
- The manager resolves manually; staff can file a manual report.
- Explanations cite movement ids and numbers; no speculative accusations.
- No auto-correction, no auto-deletion, no auto-blocking.

**Testing**

- Table-driven rule tests: boundary times, exactly 3 adjustments, outlier with < 5 samples (no flag), zero variance.
- Dedup: two scans in the same bucket → one flag.
- After-commit listener never rolls back a movement; a failing rule does not fail the request.
- Manual report → flag; staff cannot resolve.
- Resolve/dismiss state transitions and audit fields.
- Tenant isolation and tenant timezone.
- Run log steps for the scan.

**DoD**

- [ ] A seeded out-of-hours adjustment produces a flag with a readable explanation.
- [ ] Repeated small adjustments produce one deduplicated flag.
- [ ] The agent never writes stock (code/test proof).
- [ ] Manager can acknowledge/resolve; staff can file a manual report.
- [ ] Scheduler and after-commit trigger both work.
- [ ] Demo artifact G4 is available.

**Effort:** **M** — 3–4 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P9-T1
