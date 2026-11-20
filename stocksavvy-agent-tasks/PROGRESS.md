# PROGRESS — master tracker

Tick an item only when it is **verified in code and tests**. Order matters: top to bottom (phases 5 and 8 may run in parallel with 6/7).

## Task status

| Done | Task | Phase | File |
|---|---|---|---|
| ⬜ | **P0-T1** Platform bootstrap & tenant isolation completion | 0 | `TASKS/P0-T1_platform_bootstrap_and_tenant_isolation.md` |
| ⬜ | **P0-T2** Personas, authentication & RBAC | 0 | `TASKS/P0-T2_personas_auth_rbac.md` |
| ⬜ | **P1-T1** SKU management | 1 | `TASKS/P1-T1_sku_management.md` |
| ⬜ | **P1-T2** Stock ledger, balances & stock operations | 1 | `TASKS/P1-T2_stock_ledger_balances_operations.md` |
| ⬜ | **P1-T3** Low-stock dashboard | 1 | `TASKS/P1-T3_low_stock_dashboard.md` |
| ⬜ | **P2-T1** Vendors & pricing | 2 | `TASKS/P2-T1_vendors_and_pricing.md` |
| ⬜ | **P2-T2** Purchase orders | 2 | `TASKS/P2-T2_purchase_orders.md` |
| ⬜ | **P3-T1** Approval queue & agent audit trail | 3 | `TASKS/P3-T1_approval_queue_and_audit_trail.md` |
| ⬜ | **P4-T1** RAG foundation: ingestion, chunking, embeddings, pgvector | 4 | `TASKS/P4-T1_rag_foundation_ingestion.md` |
| ⬜ | **P4-T2** RAG retrieval, grounding & chat assistant | 4 | `TASKS/P4-T2_rag_retrieval_and_chat.md` |
| ⬜ | **P5-T1** Demand Forecasting Agent | 5 | `TASKS/P5-T1_demand_forecasting_agent.md` |
| ⬜ | **P6-T1** Reorder / Restock Automation Agent | 6 | `TASKS/P6-T1_reorder_agent.md` |
| ⬜ | **P7-T1** Vendor Negotiation Agent (flagship) | 7 | `TASKS/P7-T1_negotiation_agent.md` |
| ⬜ | **P8-T1** Anomaly / Fraud Detection Agent | 8 | `TASKS/P8-T1_anomaly_detection_agent.md` |
| ⬜ | **P9-T1** Dashboard, demo data & defense artifacts | 9 | `TASKS/P9-T1_dashboard_demo_data_defense_artifacts.md` |

---

## P0-T1 — Platform bootstrap & tenant isolation completion

- [ ] App starts with a generated dev keypair and a clean database.
- [ ] `POST /auth/login` returns a JWT; `/auth/me` works.
- [ ] `Tenant`/`User` are in `public`; business queries route to `tenant_<code>` (verified by SQL logging/test).
- [ ] Unknown tenant / non-ACTIVE tenant fails closed.
- [ ] `created_by` is populated for HTTP and `system` for background writes.
- [ ] Two-tenant isolation test passes for list, get-by-id, update, delete.
- [ ] `docker-compose` PostgreSQL has pgvector available.
- [ ] No dead tenant filters remain.

**Conflict-log items to fix here:** #1, #2, #3, #4, #6, #10, #11, #12, #14, #18, #22, #23, #24
- [ ] Conflict #1 fixed
- [ ] Conflict #2 fixed
- [ ] Conflict #3 fixed
- [ ] Conflict #4 fixed
- [ ] Conflict #6 fixed
- [ ] Conflict #10 fixed
- [ ] Conflict #11 fixed
- [ ] Conflict #12 fixed
- [ ] Conflict #14 fixed
- [ ] Conflict #18 fixed
- [ ] Conflict #22 fixed
- [ ] Conflict #23 fixed
- [ ] Conflict #24 fixed

**Spec features covered:** (foundation: unblocks everything)

## P0-T2 — Personas, authentication & RBAC

- [ ] Manager and staff demo users exist and have different permissions.
- [ ] Every mutating endpoint has an explicit role rule.
- [ ] Vendor persona is documented as simulated (no login).
- [ ] 401/403 responses use the standard error shape.

**Conflict-log items to fix here:** #13, #14
- [ ] Conflict #13 fixed
- [ ] Conflict #14 fixed

**Spec features covered:** A1, A2, A3

## P1-T1 — SKU management

- [ ] Product CRUD reachable via Swagger and tests.
- [ ] `ProductResponse` has `id`, `categoryId`, no NPE for uncategorized products.
- [ ] Soft delete works and is filtered everywhere.
- [ ] Validation errors return 400 with field messages.
- [ ] Cross-tenant isolation test passes.

**Conflict-log items to fix here:** #8, #9, #13, #16, #17
- [ ] Conflict #8 fixed
- [ ] Conflict #9 fixed
- [ ] Conflict #13 fixed
- [ ] Conflict #16 fixed
- [ ] Conflict #17 fixed

**Spec features covered:** B1

## P1-T2 — Stock ledger, balances & stock operations

- [ ] Balance always equals the ledger sum after a test sequence.
- [ ] `stock_mvts` has no update/delete path.
- [ ] Staff can receive goods and record a count end-to-end.
- [ ] History and low-stock endpoints work.
- [ ] Concurrency test passes without negative on-hand.
- [ ] `StockMovementPostedEvent` is published after commit.

**Conflict-log items to fix here:** #5, #7, #15, #16
- [ ] Conflict #5 fixed
- [ ] Conflict #7 fixed
- [ ] Conflict #15 fixed
- [ ] Conflict #16 fixed

**Spec features covered:** B2, B3, B5

## P1-T3 — Low-stock dashboard

- [ ] Manager sees SKU-118 and its threshold/on-hand in one call.
- [ ] Boundary cases tested.
- [ ] Response includes a suggested quantity only when a forecast exists (or a clearly labeled heuristic).

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** B4

## P2-T1 — Vendors & pricing

- [ ] Vendor catalog supports price, lead time, MOQ, currency.
- [ ] Last-paid price is derivable from PO lines (after 6.7).
- [ ] Simulation profile is stored but not leaked to the agent prompt.
- [ ] Manager can manage vendors via Swagger.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** B6

## P2-T2 — Purchase orders

- [ ] Manager can see a draft PO with vendor, lines, totals, and reasoning.
- [ ] Approval is the only path to `APPROVED`.
- [ ] `FINALIZED` is reachable only after approval/negotiation.
- [ ] Receipts can reference a PO.
- [ ] Cross-tenant isolation tests.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** B7

## P3-T1 — Approval queue & agent audit trail

- [ ] A pending PO approval is visible with payload and reasoning.
- [ ] Approve/reject/edit works and is audit-logged.
- [ ] Every run can be replayed from its logs.
- [ ] No agent can bypass the approval service to commit spend.
- [ ] Cross-tenant isolation test.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** C1, C2, C3

## P4-T1 — RAG foundation: ingestion, chunking, embeddings, pgvector

- [ ] Upload a contract and a catalog row; both are searchable with metadata.
- [ ] Ingestion is idempotent and tenant-scoped.
- [ ] A failed ingestion does not break the triggering business transaction.
- [ ] `rag_queries` records retrieval for the artifact.
- [ ] pgvector extension and image are documented/reproducible.

**Conflict-log items to fix here:** #6, #16
- [ ] Conflict #6 fixed
- [ ] Conflict #16 fixed

**Spec features covered:** D1, D2

## P4-T2 — RAG retrieval, grounding & chat assistant

- [ ] “What did we last pay vendor X for SKU-118?” returns an answer with the PO/transcript source and exact price.
- [ ] Sources are visible in the response and stored.
- [ ] Unanswerable questions are refused, not hallucinated.
- [ ] Cross-tenant retrieval test passes.
- [ ] Demo works without an external API key (stub/template mode).

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** D3, D4

## P5-T1 — Demand Forecasting Agent

- [ ] Nightly and on-demand forecasts work for SKU-118.
- [ ] Each forecast has confidence + plain-language explanation.
- [ ] Numbers are reproducible from the stored window/method.
- [ ] No approval is created.
- [ ] Tenant job runner test passes.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** E1, E5

## P6-T1 — Reorder / Restock Automation Agent

- [ ] SKU-118 low stock → a draft PO with vendor, quantity, price, and reasoning.
- [ ] Manager sees it in the approval queue and can approve/reject/edit.
- [ ] No PO leaves DRAFT without an approval.
- [ ] Approval can route to negotiation instead of finalizing.
- [ ] Idempotency and tenant isolation tests pass.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** E2, E5

## P7-T1 — Vendor Negotiation Agent (flagship)

- [ ] Demo: opening offer grounded in last 3 PO prices + contract/transcript citations.
- [ ] Manager approves/edits every outbound message.
- [ ] Simulated counter-offer is evaluated against target/walk-away/max rounds.
- [ ] Agreement → final approval → PO finalized → transcript searchable.
- [ ] Max rounds → escalation visible.
- [ ] Transcript endpoint returns the full artifact.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** E3, E5

## P8-T1 — Anomaly / Fraud Detection Agent

- [ ] A seeded out-of-hours adjustment produces a flag with a readable explanation.
- [ ] Repeated small adjustments produce one deduplicated flag.
- [ ] The agent never writes stock (code/test proof).
- [ ] Manager can acknowledge/resolve; staff can file a manual report.
- [ ] Scheduler and after-commit trigger both work.
- [ ] Demo artifact G4 is available.

**Conflict-log items to fix here:** #16
- [ ] Conflict #16 fixed

**Spec features covered:** E4, E5

## P9-T1 — Dashboard, demo data & defense artifacts

- [ ] One call returns the morning dashboard with SKU-118 low + forecast.
- [ ] Seeder creates a complete, repeatable demo in < 60 seconds.
- [ ] All four artifacts are reachable and readable.
- [ ] Swagger has a bearer “Authorize” button and organized tags.
- [ ] The 8-step demo runs end-to-end without manual DB edits.

**Conflict-log items to fix here:** #19, #20
- [ ] Conflict #19 fixed
- [ ] Conflict #20 fixed

**Spec features covered:** F, G1, G2, G3, G4

## Phase gates (from the plan)

- [ ] After Phase 6: run the **first half** of the 8-step demo with stub LLM/embeddings.
- [ ] After Phase 7: run the **full** 8-step demo headlessly.
- [ ] Phase 9: four defense artifacts captured (approval queue, negotiation transcript, RAG answer with sources, anomaly flag).
- [ ] Conflict #21 (package typo): intentionally NOT renamed.
- [ ] Conflict #22 (tests): minimum test matrix in `99_REFERENCE/test_matrix.md` satisfied.
