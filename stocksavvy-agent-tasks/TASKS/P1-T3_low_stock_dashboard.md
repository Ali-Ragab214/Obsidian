# P1-T3 — Low-stock dashboard

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 1 — SKU + stock ledger + stock operations |
| Source section | `6.5` of the original plan |
| Depends on (spec) | P1-T1 (§6.3), P1-T2 (§6.4); forecast fields enrich it after P5-T1 (§6.11). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2 |
| Effort | **S** — 1–2 days. |
| Next task | P2-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P1-T3 — "Low-stock dashboard" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Reuses the low-stock query from P1-T2. Do not duplicate it.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| B4 | Low-stock dashboard | ❌ | `Product.alertThreshold` exists but no query/endpoint; no dashboard. | Manager sees what needs attention |

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

### 6.5 Low-stock dashboard (B4)

**Goal:** give the manager a single view of SKUs at or below the reorder threshold, with on-hand, threshold, forecast (if available), and suggested action.

**Depends on:** 6.3, 6.4; forecast fields enrich it after 6.11.

**Spring Boot approach**

- Query `products` joined to `stock_balances` where `on_hand <= alert_threshold` and `deleted = false`; sort by severity (`on_hand / threshold`) and reference.
- Add `GET /api/v1/dashboard/low-stock` (or reuse `/stock/low-stock`) and include it in the aggregate dashboard (6.15).
- If a forecast exists (6.11), add `dailyRate`, `daysOfCover = on_hand / dailyRate`, `trendPct`, and a `shortfall` flag; if not, return `forecast: null`. Do not compute a fake forecast.
- Trade-off: a SQL view (`v_low_stock`) makes the dashboard fast but adds migration coupling; a service query is simpler and adequate for thesis data volume.

**Entities / tables**

- No new tables. Uses `products`, `stock_balances`, `demand_forecasts` (nullable join).
- Index `products(deleted, alert_threshold)` and `stock_balances(on_hand)` if the dataset grows.

**Classes**

- `services/DashboardService.java` + impl (or `LowStockService`).
- `controllers/DashboardController.java` (shared with 6.15).
- `responses/LowStockItemResponse.java` (`productId, sku, name, onHand, reorderPoint, daysOfCover, forecastDailyRate, trendPct, suggestedOrderQty`).

**API endpoints**

| Method | Path | Response | Roles |
|---|---|---|---|
| `GET` | `/api/v1/dashboard/low-stock?limit=` | `List<LowStockItemResponse>` | Manager |
| `GET` | `/api/v1/stock/low-stock` | same (alias used by stock screens) | Manager |

**Multi-tenancy considerations**

- The query runs in the tenant schema; no `tenant_id` filter needed. No caching of cross-tenant results.
- If a dashboard cache is added, key it by tenant + date.

**HITL / safety**

- Read-only. The manager decides whether to trigger reorder/negotiation.

**Testing**

- Boundary: `on_hand == threshold` is included; `on_hand == threshold + 1` is not.
- Product with no balance row is treated as 0 (or hidden if not initialized — choose “0 and included”, document).
- Forecast join absent → no NPE.
- Tenant isolation.

**DoD**

- [ ] Manager sees SKU-118 and its threshold/on-hand in one call.
- [ ] Boundary cases tested.
- [ ] Response includes a suggested quantity only when a forecast exists (or a clearly labeled heuristic).

**Effort:** **S** — 1–2 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P2-T1
