# 04 — Roadmap (sequencing rationale + phase summary)

## 5. Implementation Roadmap

### 5.1 Sequencing rationale

The suggested starting point (foundations → vendors/POs → approval/audit → RAG → forecasting → reorder → negotiation → anomaly → dashboard) is correct in spirit, but the real code adds one non-negotiable phase **before** foundations: the app does not start and tenant routing does not exist. The order below is therefore:

1. **Phase 0 — Runnable foundation & tenant isolation completion.** Nothing else can be safely built until a request can be authenticated, routed to a tenant schema, and audited.
2. **Phase 1 — Core inventory & stock ledger.** Reorder forecasting/agents depend on real movements and balances.
3. **Phase 2 — Vendors & purchase orders.** Approval handlers need a concrete business object to act on.
4. **Phase 3 — Approval queue & audit trail.** The shared HITL mechanism, proven with the PO handler.
5. **Phase 4 — RAG foundation.** Ingestion + retrieval; negotiation is grounded in vendor history.
6. **Phase 5 — Demand forecasting.** Simple, deterministic, independent of approvals.
7. **Phase 6 — Reorder agent.** Combines stock, forecast, vendors, PO draft, approvals.
8. **Phase 7 — Negotiation agent.** Flagship; depends on 2, 3, 4, 6.
9. **Phase 8 — Anomaly agent.** Depends on the ledger and audit log; technically parallelizable with 5–7, placed here for demo sequencing.
10. **Phase 9 — Dashboard, demo data, defense artifacts.** Integrates everything and makes it presentable.

Anomaly detection could be built earlier (it only needs Phase 1 + Phase 3 for logging), and forecasting is independent of vendors. If two developers are available, run **5 and 8 in parallel** with **6/7**. Everything else is on the critical path.

### 5.2 Phase summary

| Phase | Name | Why now | Depends on | Main outputs | Effort | Critical path |
|---|---|---|---|---|---|---|
| 0 | Runnable foundation + tenant isolation | App cannot start; isolation is unenforced | — | Hibernate routing, fail-closed resolver, platform migration, login, RBAC, AuditorAware, error handling, pgvector image | **L** (7–9 d) | ✅ |
| 1 | SKU + stock ledger + stock operations | Every agent/report needs real quantities | 0 | Product API, append-only ledger, balances, receipts, counts, history, low-stock query | **L** (7–9 d) | ✅ |
| 2 | Vendors + purchase orders | Reorder/negotiation need vendors and a PO object | 1 | Vendor catalog, PO lifecycle, price history, numbering | **M** (3–4 d) | ✅ |
| 3 | Approval queue + agent audit trail | Shared HITL mechanism; prove with PO approval | 2 | `ApprovalRequest`, `AgentRunLog`, `AgentRun`, handlers, endpoints | **M** (3–4 d) | ✅ |
| 4 | RAG foundation (ingest + retrieve) | Grounds negotiation and chat | 0–2 | pgvector, documents/chunks, embeddings, ingestion triggers, retrieval service | **L** (6–8 d) | ✅ |
| 5 | Demand forecasting agent | Needed by reorder; low risk, high demo value | 1 | Forecast engine, table, scheduled/on-demand runs, explanation | **M** (3–4 d) | ⬜ |
| 6 | Reorder agent | Demo step 2; exercises approvals | 1, 2, 3, 5 | Vendor selection, PO draft, reasoning, approval flow | **M** (4–5 d) | ✅ |
| 7 | Negotiation agent | Flagship demo steps 3–7 | 2, 3, 4, 6 | State machine, messages, simulated vendor, RAG grounding, transcript ingestion | **L** (8–10 d) | ✅ |
| 8 | Anomaly agent | Demo step 8; independent | 1, 3 | Rules, flags, explanations, review API | **M** (3–4 d) | ⬜ |
| 9 | Dashboard, demo data, artifacts | Committee presentation | all | Aggregate dashboard, seeder/reset, OpenAPI, artifact endpoints, polish | **M** (4–5 d) | ✅ |

**Total:** ~10–12 weeks for one developer. Parallelizing 5/8 with 6/7 reduces calendar time to ~8–9 weeks; the critical path (0→1→2→3→4→7→9) is ~43 developer-days.
