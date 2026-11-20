# Minimum test matrix for the defense

## Appendix C — Test matrix (minimum for defense)

| Area | Test | Type |
|---|---|---|
| Tenancy | Tenant A data invisible to B (list/get/update/delete) | Integration |
| Tenancy | Unknown/suspended tenant fails closed | Integration |
| Auth | Login, wrong password, disabled user, suspended tenant | Integration |
| RBAC | Manager vs staff vs other tenant vs anonymous on every mutating endpoint | Integration |
| Stock | Balance = ledger sum; append-only; insufficient stock; concurrency | Unit + integration |
| Stock | Receipt and count workflows | Integration |
| Vendors/PO | CRUD, unique constraints, lifecycle transitions, totals | Integration |
| Approvals | Approve/reject/edit, double approve, handler failure, tenant isolation | Integration |
| Audit | Required log steps for each agent; transcript ordering | Integration |
| RAG | Chunking, idempotency, metadata filters, no-answer, tenant isolation | Unit + integration |
| Forecast | Moving average/SES math, insufficient history, trend, LLM fallback | Unit |
| Reorder | Quantity formula, vendor selection, no duplicate drafts, no autonomous approval | Unit + integration |
| Negotiation | All state transitions, approval gating, accept/counter/escalate, transcript ingestion | Unit + integration |
| Anomaly | Rule boundaries, dedup, after-commit safety, no stock write | Unit + integration |
| Demo | Seeder idempotency, reset guard, 8-step headless flow | Integration |
