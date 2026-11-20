# 03 — Feature gap matrix

## 3. Feature Gap Matrix

Legend: ✅ Done · 🟡 Partial · ❌ Missing. “Related user story” refers to the spec section.

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| A1 | Manager persona (Mona) | 🟡 | No `MANAGER`/persona role; `ROLE_COMPANY_ADMIN`/`ROLE_ADMINISTRATOR` exist but no permission checks. No dashboard. | Approvals, negotiation targets, anomaly review |
| A2 | Staff persona (Sam) | 🟡 | `ROLE_USER`/`ROLE_SALES_OPERATOR` exist; no stock-posting permission, no receipt/count endpoint, no discrepancy flag endpoint. | Stock counts, deliveries, shelf discrepancies |
| A3 | Vendor external persona | ❌ | No vendor user/portal; no simulation. Vendors module itself is missing. | Counter-offers (simulated) |
| B1 | SKU management | 🟡 | `Product`/`ProductRepository`/`ProductServiceImpl` exist; `reference` acts as SKU. No controller, no `id` in response, no UoM/category id, update bug (duplicate check always runs), mapper NPE if category is null. | Manager maintains SKUs |
| B2 | Ledger stock movements (append-only) | 🟡 | `StockMvt` entity + `stock_mvts` table exist. `StockMvtServiceImpl` is empty; no controller; no balance projection; `BasicService` still exposes `update`/`delete` (conflicts with append-only); `dateMvt` is `LocalDate`, insufficient for anomaly hours. | Full movement history |
| B3 | Staff stock counts / delivery receipts | ❌ | No receipt/count service, no `stock_counts`, no PO link, no validation of negative stock. | Staff updates real quantities |
| B4 | Low-stock dashboard | ❌ | `Product.alertThreshold` exists but no query/endpoint; no dashboard. | Manager sees what needs attention |
| B5 | Full movement history per SKU | ❌ | `StockMvtRepository` has no query methods; no endpoint; no pagination/filtering. | Investigate discrepancies |
| B6 | Vendors module (catalog, price, lead time) | ❌ | No `Vendor`, `VendorProduct`, `VendorNegotiationProfile`; no tables/endpoints. | Vendor selection, pricing |
| B7 | Purchase orders (draft → approved → finalized) | ❌ | No `PurchaseOrder`/lines; no lifecycle; no numbering; no link to stock receipts. | Draft/approve POs |
| C1 | `ApprovalRequest` shared queue | ❌ | Not found. | Human-in-the-loop for every agent |
| C2 | `AgentRunLog` audit trail | ❌ | Not found. | Defense artifact: full transcript |
| C3 | Shared “log every step / approve every action” rule | ❌ | No agent infrastructure exists. | Architectural HITL property |
| D1 | RAG knowledge sources (contracts, catalogs, transcripts, stock reports) | ❌ | No document upload, no ingestion, no transcript generation. | Grounded answers |
| D2 | Ingestion pipeline (chunk → embed → pgvector + metadata) | ❌ | No pgvector dependency, no vector table, no embedding model, no chunker. | Grounded answers |
| D3 | Retrieval/grounding with metadata filters and source display | ❌ | Nothing exists. | Negotiation opening offer grounded in history |
| D4 | RAG chat assistant | ❌ | Nothing exists. | “What did we last pay vendor X for SKU-118?” |
| E1 | Demand Forecasting Agent | ❌ | No forecast entity/service/scheduler/LLM. | Forecast with plain-language reason |
| E2 | Reorder / Restock Agent | ❌ | No vendor selection, PO draft, or approval. | Draft PO when stock is low |
| E3 | Vendor Negotiation Agent | ❌ | No state machine, messages, simulation, approvals, or RAG. | Flagship negotiation flow |
| E4 | Anomaly / Fraud Detection Agent | ❌ | No flags, rules, explanation, or review screen. | Suspicious movements, never auto-corrected |
| E5 | Four separate agents | ❌ | N/A until built; plan keeps each agent isolated. | Explainable responsibilities |
| F | End-to-end demo workflow (8 steps) | ❌ | None of the 8 steps is executable. | Committee demo |
| G1 | Approval queue artifact (pending negotiation + reasoning) | ❌ | No approval queue. | Defense artifact |
| G2 | Full negotiation transcript artifact | ❌ | No `AgentRunLog`/messages. | Defense artifact |
| G3 | RAG answer with visible sources artifact | ❌ | No RAG. | Defense artifact |
| G4 | Anomaly flag with explanation artifact | ❌ | No anomaly agent. | Defense artifact |
