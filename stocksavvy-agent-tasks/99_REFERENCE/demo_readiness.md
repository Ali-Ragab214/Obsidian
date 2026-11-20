# Demo readiness (8-step story + defense artifacts)

## 8. Demo Readiness Checklist (section F)

| # | Demo step | Required features | Primary endpoints / action | Artifact | Phase | Status |
|---|---|---|---|---|---|---|
| 1 | Morning dashboard: SKU-118 below threshold; forecast trending up | SKU, ledger, low-stock, forecast, dashboard | `GET /dashboard` (or `/dashboard/low-stock` + `/forecasts/products/{id}/latest`) | Dashboard response | 1, 5, 9 | ❌ |
| 2 | Reorder agent drafts a PO with best vendor + reasoning | Vendors, POs, reorder, approvals, run log | `POST /agents/reorder/run` → `GET /approvals?status=PENDING` | Approval payload + reasoning | 2, 3, 6 | ❌ |
| 3 | Manager routes PO to negotiation with target 8% below list | POs, negotiation start, roles | `POST /purchase-orders/{id}/negotiate` | Negotiation case | 7 | ❌ |
| 4 | Opening offer grounded in last 3 orders; manager approves | RAG, negotiation strategy, approvals | `POST /negotiations/{id}/draft` → approve | Approval with citations | 3, 4, 7 | ❌ |
| 5 | Simulated counter-offer evaluated; loop once | Negotiation state machine, simulator | `POST /negotiations/{id}/simulate-vendor-response` | Messages + run logs | 7 | ❌ |
| 6 | Agreement → final approval → PO finalized; transcript stored/searchable | Negotiation finalize, PO, RAG ingestion | approve final → `GET /negotiations/{id}/transcript` | Transcript artifact (G2) | 7 | ❌ |
| 7 | RAG assistant answers “what price did we just agree on?” with citation | RAG chat, transcript source | `POST /ai/chat` | Chat sources (G3) | 4, 9 | ❌ |
| 8 | Anomaly flag on another SKU; manager reviews/resolves; no data change | Anomaly rules, flags, dashboard | `GET /anomalies?status=OPEN` → resolve | Flag + explanation (G4) | 8, 9 | ❌ |

### 8.1 Defense artifacts (section G)

| Artifact | Where to show it | Must contain |
|---|---|---|
| G1 Approval queue with pending negotiation step | `GET /api/v1/approvals?status=PENDING` or dashboard | Agent type, run id, step, draft message/PO, plain-language reasoning, target/walk-away, citations |
| G2 Full negotiation transcript | `GET /api/v1/negotiations/{id}/transcript` | Every agent/vendor message, round, price, approval, reasoning, timestamps |
| G3 RAG answer with visible sources | `POST /api/v1/ai/chat` + `GET /api/v1/ai/chat/{queryId}` | Answer, numbered sources with document/vendor/SKU/date/score, “not enough information” path |
| G4 Anomaly flag with explanation | `GET /api/v1/anomalies/{id}` | Rule, severity, related movement ids, plain-language explanation, resolution state |

### 8.2 Suggested demo data (seeded)

- Tenant `demo`, manager `mona`, staff `sam`.
- `SKU-118`: on hand 4, reorder point 10, 60–90 days of SALE history trending up; vendor Acme list 13.50, last paid 12.40, lead time 5 days.
- `SKU-204`: out-of-hours manual adjustment for the anomaly step.
- Vendor Acme negotiation profile: floor ratio 0.90, concession 0.5, max rounds 3 (hidden from prompts).
- One pending reorder approval left by the seeder (optional) so the demo can start at step 2 or run step 2 live.
