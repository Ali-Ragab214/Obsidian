# P2-T2 — Purchase orders

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 2 — Vendors + purchase orders |
| Source section | `6.7` of the original plan |
| Depends on (spec) | P1-T1 (§6.3), P2-T1 (§6.6); approval infra is wired in P3-T1 (§6.8). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1 |
| Effort | **M** — 3–4 days. |
| Next task | P3-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P2-T2 — "Purchase orders" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Approval of agent-created POs is wired in P3-T1. Until then, only the manual manager path exists. No agent path may reach `APPROVED`/`FINALIZED` without an `ApprovalRequest`.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| B7 | Purchase orders (draft → approved → finalized) | ❌ | No `PurchaseOrder`/lines; no lifecycle; no numbering; no link to stock receipts. | Draft/approve POs |

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

### 6.7 Purchase orders (B7)

**Goal:** implement a purchase order with lines and a guarded lifecycle (`DRAFT → APPROVED → FINALIZED`, plus `NEGOTIATING`/`CANCELLED`), usable by the reorder and negotiation agents.

**Depends on:** 6.3, 6.6; approval infra is wired in 6.8.

**Spring Boot approach**

- `PurchaseOrder` aggregate with `PurchaseOrderLine` children (`@OneToMany(cascade = ALL, orphanRemoval = true)`), a human-readable reference (`PO-2026-000123`) from a per-tenant PostgreSQL sequence, and a `@Version` for optimistic locking.
- **State transitions live in a service**, not in setters: `submitForApproval`, `approve`, `startNegotiation`, `finalize`, `cancel`. Invalid transitions throw `IllegalStateException` → 409.
- **Approval is not a PO endpoint.** Agent-created POs become `APPROVED` only through `ApprovalService` (6.8). Manual manager-created POs may be approved directly by a manager, recorded as a manual audit entry; no agent path can bypass approval. This keeps one decision path for autonomous work while allowing a manager to create a PO by hand.
- **“Created for real” interpretation:** the draft PO row is persisted immediately (so it has lines, a vendor, and a total to review); it is not committed to the vendor until `APPROVED`/`FINALIZED`. This is the simplest model that gives the manager a concrete object to approve. Document it.
- **Receiving:** `PurchaseOrderLine.received_quantity` is incremented by stock receipts (6.4); a line is complete when `received >= ordered`. PO status can expose `PARTIALLY_RECEIVED`/`RECEIVED` later; the MVP can keep `FINALIZED` and show received quantities.
- **Totals:** compute `line_total = quantity * unit_price`; `subtotal`/`total` in the PO currency. Use `BigDecimal`, scale 4 for unit price, 2 for totals. No tax engine in the MVP.
- **Trade-off:** a full procurement module (RFQ, goods receipt notes, invoices) is out of scope. One PO + lines + receipt link is enough for the demo and defensible.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `purchase_orders` | `id`, `reference`, `vendor_id`, `status`, `currency`, `subtotal`, `total`, `expected_delivery_date`, `notes`, `negotiation_id` (nullable, added in 6.13), `created_by`, `approved_by`, `approved_at`, `finalized_at`, `version`, audit | unique `reference`; index `(status, created_at)`, `(vendor_id, status)`; FK `vendor_id` |
| `purchase_order_lines` | `id`, `purchase_order_id`, `product_id`, `quantity`, `unit_price`, `line_total`, `received_quantity` | unique `(purchase_order_id, product_id)`; FKs; check `quantity > 0`, `unit_price >= 0` |
| sequence | `CREATE SEQUENCE purchase_order_number_seq` | per tenant schema |

**Classes**

- `entities/PurchaseOrder.java`, `PurchaseOrderLine.java`, `PurchaseOrderStatus.java`.
- `repositories/PurchaseOrderRepository.java`, `PurchaseOrderLineRepository.java`.
- `services/PurchaseOrderService.java` + impl; `services/PurchaseOrderNumberGenerator.java`.
- `controllers/PurchaseOrderController.java`.
- `mappers/PurchaseOrderMapper.java`.
- `requests/PurchaseOrderRequest.java`, `PurchaseOrderLineRequest.java`; `responses/PurchaseOrderResponse.java`, `PurchaseOrderSummaryResponse.java`.
- `db/migration/tenant/V3__vendors_and_purchase_orders.sql` (combined with 6.6).

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `GET` | `/api/v1/purchase-orders?status=&vendorId=&page=&size=` | — | `PageResponse<PurchaseOrderSummaryResponse>` | Manager |
| `GET` | `/api/v1/purchase-orders/{id}` | — | `PurchaseOrderResponse` (lines, approvals) | Manager |
| `POST` | `/api/v1/purchase-orders` | `PurchaseOrderRequest` (vendor, lines) | `201` DRAFT | Manager |
| `POST` | `/api/v1/purchase-orders/{id}/submit-for-approval` | — | approval id | Manager |
| `POST` | `/api/v1/purchase-orders/{id}/negotiate` | `{targetUnitPrice, walkAwayUnitPrice, maxRounds}` | negotiation id | Manager |
| `POST` | `/api/v1/purchase-orders/{id}/finalize` | — | FINALIZED PO | Manager (after approval/escalation) |
| `POST` | `/api/v1/purchase-orders/{id}/cancel` | `{reason}` | CANCELLED | Manager |
| `GET` | `/api/v1/purchase-orders/{id}/price-history` | — | lines + vendor | Manager |

**Multi-tenancy considerations**

- PO tables live in the tenant schema; the sequence is per schema.
- PO references are unique per tenant, not globally.
- Vendor/product IDs are tenant-local; validate both belong to the current schema (a cross-tenant UUID would simply not resolve).
- Scheduled/agent-created POs run under an explicit tenant context.

**HITL / safety**

- No **agent-created** PO can move to `APPROVED` or `FINALIZED` without an `ApprovalRequest`. The service checks the approval id; manual manager actions are logged in `AgentRunLog`. Enforce in the service, not just the controller.
- Rejecting an approval moves the PO to `CANCELLED` (or leaves it `DRAFT` for editing — choose “DRAFT + rejection note”; document).
- Spend is never committed autonomously.

**Testing**

- Lifecycle: DRAFT → submit → approval → APPROVED → FINALIZED; invalid transitions → 409.
- Total/line math with `BigDecimal`.
- Reference sequence increments per tenant and resets logically per schema.
- Cannot approve a PO from another tenant (404).
- Rejection path leaves the PO auditable.
- Receipt updates `received_quantity`.

**DoD**

- [ ] Manager can see a draft PO with vendor, lines, totals, and reasoning.
- [ ] Approval is the only path to `APPROVED`.
- [ ] `FINALIZED` is reachable only after approval/negotiation.
- [ ] Receipts can reference a PO.
- [ ] Cross-tenant isolation tests.

**Effort:** **M** — 3–4 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P3-T1
