# P5-T1 — Demand Forecasting Agent

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 5 — Demand forecasting agent |
| Source section | `6.11` of the original plan |
| Depends on (spec) | P1-T2 (§6.4) (movement history), P0-T1 (§6.1) (scheduler/tenant context), P3-T1 (§6.8) (audit logging). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2 |
| Effort | **M** — 3–4 days. |
| Next task | P6-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P5-T1 — "Demand Forecasting Agent" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Can run in parallel with P8-T1 if two people work. Numbers are computed in Java; the LLM only writes the explanation (with a deterministic fallback).

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| E1 | Demand Forecasting Agent | ❌ | No forecast entity/service/scheduler/LLM. | Forecast with plain-language reason |
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

### 6.11 Demand Forecasting Agent (E1)

**Goal:** produce a per-SKU demand forecast with a confidence level and a plain-language explanation, run nightly and on demand, without requiring approval.

**Depends on:** 6.4 (movement history), 6.1 (scheduler/tenant context), 6.8 (audit logging).

**Spring Boot approach**

- **Deterministic statistics first:** a `ForecastingEngine` interface with two implementations:
  - `MovingAverageForecaster`: average daily demand over the last N days (default 14/28).
  - `ExponentialSmoothingForecaster`: SES with alpha 0.3 (default), simple and explainable.
  - Select via `app.forecast.method` (default `SES`). No Python, no ML model. This is the defensible choice for a thesis.
- **Demand series:** OUT movements with `reason = SALE` (legacy/null reason treated as SALE for backfill), bucketed by `date_mvt`, zero-filled for missing days. Exclude `COUNT_ADJUSTMENT`/`DAMAGE`/`MANUAL_ADJUSTMENT`. If there are fewer than `minHistoryDays` (default 14) of data, return `LOW` confidence and state “insufficient history”.
- **Horizon:** default `leadTimeDays + reviewPeriodDays` (vendor lead time from the selected/default vendor, review period configurable). Forecast daily rate and total demand over the horizon.
- **Confidence:** simple, explainable heuristic:
  - `HIGH`: ≥ 21 days of history and coefficient of variation ≤ 0.5;
  - `MEDIUM`: ≥ 14 days and CV ≤ 0.8;
  - `LOW`: otherwise.
  Document the thresholds; they are not magic.
- **Trend:** `(avg(last 7) - avg(previous 7)) / avg(previous 7) * 100`, guarded against zero. Used in the explanation and reorder reasoning.
- **Explanation:** deterministic template (always produced) + optional LLM polish. The numbers are computed by Java and passed into the prompt; the LLM is instructed not to change them. Store both `explanation` and `explanationSource` (`TEMPLATE`/`LLM`). If the LLM fails, keep the template. This is a key defensibility point.
- **Trigger:** `@Scheduled` nightly coordinator (`0 0 2 * * *`, configurable) + `POST /api/v1/forecasts/run` (manager, optional productId). The scheduler uses `TenantJobRunner` to iterate ACTIVE tenants, set/clear `TenantContext`, and run each tenant in its own transaction.
- **Audit:** one `AgentRun` per tenant per batch (or per product for a manual run); one `AgentRunLog` step per SKU with input (window/no. of observations) and output (rate, confidence, trend). No `ApprovalRequest`.
- **Trade-off:** `@Scheduled` + a per-tenant loop is enough for a single-instance thesis deployment. Quartz/Spring Batch add persistence, job tables, and operational surface; if multi-instance deployment is required, add ShedLock rather than Quartz.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `demand_forecasts` | `id`, `product_id`, `generated_at`, `method` (`MOVING_AVERAGE`/`SES`), `window_days`, `horizon_days`, `daily_rate numeric(12,4)`, `predicted_demand numeric(12,2)`, `confidence` (`LOW/MEDIUM/HIGH`), `trend_pct numeric(8,2)`, `explanation text`, `explanation_source`, `model_version`, `run_id`, audit | index `(product_id, generated_at desc)`, `(confidence)`; FK product |

**Classes**

- `agents/forecast/ForecastingEngine.java`, `MovingAverageForecaster.java`, `ExponentialSmoothingForecaster.java`, `ForecastResult.java`, `ConfidenceCalculator.java`, `DemandSeriesBuilder.java`.
- `agents/forecast/DemandForecastAgent.java` (orchestrates run + logs + explanation).
- `entities/DemandForecast.java`; `repositories/DemandForecastRepository.java`.
- `services/DemandForecastService.java` + impl; `controllers/ForecastController.java`.
- `agents/common/TenantJobRunner.java` (shared by forecast/reorder/anomaly schedulers).
- `responses/ForecastResponse.java`, `ForecastRunSummaryResponse.java`.
- `db/migration/tenant/V6__demand_forecasts.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/forecasts/run` | `{productId?, windowDays?, horizonDays?}` | run summary | Manager |
| `GET` | `/api/v1/forecasts?productId=&latest=true&page=&size=` | — | `PageResponse<ForecastResponse>` | Manager, Staff (read) |
| `GET` | `/api/v1/forecasts/products/{id}/latest` | — | latest forecast with explanation | Manager, Staff |
| `GET` | `/api/v1/forecasts/products/{id}/history?limit=` | — | forecast history | Manager |

**Multi-tenancy considerations**

- Forecasts are tenant-local; the nightly coordinator must never rely on an ambient ThreadLocal. `TenantJobRunner` queries `public.tenants` for ACTIVE tenants, sets the tenant/schema, runs, clears in `finally`.
- No cross-tenant model training or pooled data. If a global model is ever added, it must be explicitly consented to; not in this thesis.
- The scheduler is disabled during tests or replaced by a manual trigger.

**HITL / safety**

- Fully autonomous and informational; no approval.
- Never writes stock or POs.
- The explanation may not alter the numbers; store the computed values separately from the narrative.
- Low-confidence forecasts are labeled as such; the reorder agent must not treat LOW confidence as certainty.

**Testing**

- Known series → expected moving average/SES (tolerance `1e-6`).
- Zero/insufficient history → LOW + clear explanation.
- Trend calculation with zero denominator.
- Scheduler iterates two tenants and isolates results.
- LLM failure → template explanation still stored.
- Run log contains one step per SKU.

**DoD**

- [ ] Nightly and on-demand forecasts work for SKU-118.
- [ ] Each forecast has confidence + plain-language explanation.
- [ ] Numbers are reproducible from the stored window/method.
- [ ] No approval is created.
- [ ] Tenant job runner test passes.

**Effort:** **M** — 3–4 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P6-T1
