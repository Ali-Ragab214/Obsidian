# P6-T1 — Reorder / Restock Automation Agent

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 6 — Reorder agent |
| Source section | `6.12` of the original plan |
| Depends on (spec) | P1-T2 (§6.4) (stock), P1-T3 (§6.5) (low-stock query), P2-T1 (§6.6) (vendors), P2-T2 (§6.7) (POs), P3-T1 (§6.8) (approvals/audit), P5-T1 (§6.11) (forecast). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1 |
| Effort | **M** — 4–5 days. |
| Next task | P7-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P6-T1 — "Reorder / Restock Automation Agent" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Drafting is autonomous; committing spend is never autonomous. No duplicate drafts for the same SKU.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| E2 | Reorder / Restock Agent | ❌ | No vendor selection, PO draft, or approval. | Draft PO when stock is low |
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

### 6.12 Reorder / Restock Automation Agent (E2)

**Goal:** when a SKU is low or forecast to fall short, draft a PO from the best vendor with quantities and reasoning, then require manager approval before it becomes real.

**Depends on:** 6.4 (stock), 6.5 (low-stock query), 6.6 (vendors), 6.7 (POs), 6.8 (approvals/audit), 6.11 (forecast).

**Spring Boot approach**

- **Trigger:** scheduled scan (e.g., daily 06:00) + manual `POST /api/v1/agents/reorder/run` + optional after-movement event (debounced; avoid drafting on every sale). For the thesis, scheduled + manual is enough and deterministic for the demo.
- **Pipeline (each step logged):**
  1. Find low-stock products (`on_hand <= alert_threshold`) and/or forecasts where `daysOfCover < leadTime + safetyDays`.
  2. Compute order quantity deterministically:
     `target = dailyRate * (leadTimeDays + reviewPeriodDays) + safetyStock - onHand - onOrder`;
     `qty = max(0, ceil(target))`, rounded up to `min_order_qty`.
     `onOrder` = sum of open PO lines (APPROVED/FINALIZED, `ordered - received`). Document every input.
  3. Select candidate vendors from `vendor_products` (active, for the product):
     - filter by `lead_time_days <= maxLeadTime` (configurable);
     - rank by effective unit price, then lead time, then last-paid history (a simple weighted score or lexicographic order; document the exact formula).
     Provide the top 1–3 candidates in the reasoning.
  4. Draft a `DRAFT` PO with the selected vendor and lines + a reasoning string (template + optional LLM polish). The LLM may explain but **must not change** quantity/vendor/price.
  5. Create an `ApprovalRequest` (`agentType=REORDER`) with payload `{purchaseOrderId, vendorId, lines, total, forecast, alternatives}` and reasoning. Set `AgentRun` to `WAITING_APPROVAL`.
  6. On approval: if `negotiate=false`, mark the PO `APPROVED` and then `FINALIZED` (or leave FINALIZED to a manager/receipt step; choose APPROVED and expose finalize); if `negotiate=true`, create a negotiation case (6.13) with the manager’s target/walk-away.
  7. On rejection: keep the PO `DRAFT` with the rejection note (editable) or cancel it; never auto-reorder.
- **Idempotency / no spam:** before drafting, check for an existing pending approval or open draft for the same `(productId, vendorId)`; skip and log. Add a `dedup_key = productId + ":" + vendorId + ":" + runDate` or a service check.
- **No autonomous spend:** the reorder agent can create only `DRAFT` POs. The transition to `APPROVED`/`FINALIZED` is exclusively in the approval handler. Enforce with a service method that requires an approval id.
- **Scheduling:** `@Scheduled` coordinator + `TenantJobRunner`; one `AgentRun` per tenant, steps per product. Use a per-tenant lock (simple DB advisory lock or a `job_locks` table) to avoid overlapping runs; for a single instance, an `AtomicBoolean` is enough, but a DB lock is more defensible.
- **Trade-off:** a full optimization model (EOQ, multi-vendor split) is unnecessary. A deterministic target-level formula + best-price vendor is explainable and sufficient for the demo.

**Entities / tables**

- Uses `purchase_orders`, `purchase_order_lines`, `vendors`, `vendor_products`, `demand_forecasts`, `approval_requests`, `agent_runs`, `agent_run_logs`.
- Optional `reorder_recommendations` table if you want to show skipped/considered candidates without creating a PO. For the MVP, the `ApprovalRequest.payload` and `AgentRunLog` already contain this. No new table required.

**Classes**

- `agents/reorder/ReorderAgent.java` (orchestrator).
- `agents/reorder/ReorderCandidate.java`, `VendorSelector.java`, `ReorderQuantityCalculator.java`, `ReorderReasoningBuilder.java`.
- `agents/reorder/ReorderApprovalHandler.java` (implements `ApprovalHandler`).
- `services/ReorderService.java` + impl; `controllers/ReorderAgentController.java`.
- `requests/ReorderRunRequest.java` (`{productId?, dryRun?}`); `responses/ReorderRunSummaryResponse.java`, `ReorderDraftResponse.java`.
- `agents/common/TenantJobRunner.java` (from 6.11).
- No new migration beyond 6.7/6.8 (optional indexes).

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/agents/reorder/run` | `{productId?, dryRun?}` | run summary + drafted approvals | Manager |
| `GET` | `/api/v1/agents/reorder/runs?page=&size=` | — | run history | Manager |
| `GET` | `/api/v1/agents/reorder/runs/{runId}` | — | steps + drafts | Manager |
| `GET` | `/api/v1/purchase-orders?status=DRAFT` | — | drafted POs | Manager |

Approval endpoints are the shared `/api/v1/approvals/{id}/approve|reject`.

**Multi-tenancy considerations**

- All queries are tenant-schema scoped; the scheduled job sets context per tenant and clears it.
- Vendor/product/PO IDs are tenant-local; never accept them from another tenant.
- `onOrder` must only count this tenant’s POs.
- Overlapping scheduler runs are prevented per tenant.

**HITL / safety**

- **Drafting is autonomous; committing spend is never autonomous.**
- The agent can only create `DRAFT` POs; approval is mandatory.
- The LLM cannot change quantities, vendor choice, or prices; it only writes reasoning.
- Rejection is recorded; no retry loop without a new run.
- The manager can edit the drafted quantity/vendor via `reviewedPayload`; the approval handler applies the edited payload and records it.

**Testing**

- Quantity formula: known on-hand/forecast/lead time/MOQ → expected quantity.
- Vendor selection: lowest price wins; lead-time filter; tie-break deterministic.
- No duplicate drafts for the same product/vendor/run.
- Approval → PO APPROVED/FINALIZED; rejection → no PO approval; no autonomous transition.
- LLM failure → template reasoning still produced.
- Two-tenant run isolation; scheduler context cleared.
- Run logs contain one step per candidate and the approval link.

**DoD**

- [ ] SKU-118 low stock → a draft PO with vendor, quantity, price, and reasoning.
- [ ] Manager sees it in the approval queue and can approve/reject/edit.
- [ ] No PO leaves DRAFT without an approval.
- [ ] Approval can route to negotiation instead of finalizing.
- [ ] Idempotency and tenant isolation tests pass.

**Effort:** **M** — 4–5 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P7-T1
