# Risks & open questions

## 9. Risks & Open Questions

### 9.1 Blocking decisions

| # | Question / risk | Why it matters | Recommended default | Owner |
|---|---|---|---|---|
| 1 | **Is multi-tenancy actually considered “done”?** The current `main` tree has scaffolding but no Hibernate resolver/provider; `TenantSchemaResolver` fails open; no branch contains a provider. | If you believe it is done on another branch/environment, share it; otherwise every feature built first will leak across tenants. | **Complete the existing schema-per-tenant design** (Phase 0). Do not pivot to shared-schema + `tenant_id` unless you explicitly choose it; pivoting now means changing the JWT/filter/provisioning assumptions. | You |
| 2 | **UI scope.** No frontend exists (not found). | “Dashboard”, “review screen”, and “visible sources” imply screens; APIs alone may be enough for a backend thesis but weaker in a demo. | Swagger + a thin Thymeleaf demo console (4 pages) for the defense; a React SPA is a separate deliverable. Decide before Phase 9. | You / supervisor |
| 3 | **LLM provider and offline demo.** No AI dependency exists. | A demo that depends on a paid API/key/network can fail in front of the committee. | Ollama locally (`llama3.1` + `nomic-embed-text`) with a `DeterministicLlmGateway` fallback; optional OpenAI profile. | You |
| 4 | **Embedding dimension.** pgvector columns need a fixed dimension. | Changing 768 ↔ 1536 later requires a migration + re-embedding. | 768 with `nomic-embed-text`; lock it before Phase 4. | You |
| 5 | **pgvector hosting.** `docker-compose` uses stock PostgreSQL 17.5. | Phase 4 cannot run without the extension. | Switch to `pgvector/pgvector:pg17` (or install the extension on the target DB) and run `CREATE EXTENSION vector` once. | You |
| 6 | **Sales vs stock mismatch rule (E4.4).** No sales/POS module exists (not found). | The spec lists it, but it cannot be computed meaningfully from the ledger alone. | Implement R1–R3 now; treat R4 as future work or add a minimal CSV sales import. State the limitation in the thesis. | You / supervisor |
| 7 | **Negotiation granularity.** Per PO or per line/SKU? | Demo is about SKU-118, but a PO can have multiple lines. | MVP: negotiate a single-product PO (or the PO total with one target); document multi-line as future work. | You |
| 8 | **Target/walk-away semantics.** | A wrong comparison direction would make the agent accept bad prices. | Define clearly: lower is better; `target ≤ walkAway`; accept if counter ≤ target; counter if ≤ walkAway; escalate otherwise. | You |
| 9 | **Warehouse model.** No `Warehouse` entity exists. | Balances/ledger/anomaly all assume a location; multi-warehouse changes keys and transfers. | Single-location MVP; add `warehouse_id` later. State it as a scope decision. | You |
| 10 | **Role mapping.** Existing roles do not include Manager/Staff. | Personas are central to the approval story. | Add `ROLE_MANAGER`/`ROLE_STAFF`; map legacy roles; enforce with meta-annotations. | You |

### 9.2 Technical risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Spring AI version incompatible with Spring Boot 4.1.2 | Medium | Blocks Phase 4 | Pin a compatible Spring AI BOM; fall back to a thin OpenAI-compatible HTTP client behind `LlmGateway`. Verify in a spike before committing. |
| Hibernate `search_path` leaks between pooled connections | Medium | Cross-tenant data leak | Set on lease, reset on release, test with a pool of 1 and >1; log/assert schema per request. |
| Schema-per-tenant migrations drift | Medium | Missing tables in existing tenants | `TenantMigrationService.migrateAll()` + Flyway history per schema + admin endpoint; test on two tenants. |
| LLM latency/timeouts inside HTTP requests | High | Slow approvals/demo | Keep LLM calls outside DB transactions; timeout + template fallback; do not call the LLM for numbers. |
| Vector search returns irrelevant chunks | Medium | Hallucinated/incorrect answers | Metadata filters (vendor/SKU), score threshold, no-answer policy, exact SQL for prices, citations in prompt. |
| LLM changes numbers or invents terms | Medium | Wrong PO/price | Compute numbers in Java; pass them as fixed context; validate/clamp output; never let the model set quantity/vendor/walk-away. |
| Duplicate scheduled drafts/approvals | Medium | Noisy queue | Idempotency keys, per-tenant job lock, partial unique index on pending approvals. |
| Approval handler runs inside the approval transaction | Medium | Approval lost if LLM/PO step fails | `@TransactionalEventListener(AFTER_COMMIT)` + `REQUIRES_NEW` audit logs + `processing_error`. |
| No frontend/only Swagger | High | Weak defense presentation | Thin Thymeleaf console or prepared Postman collection; decide early. |
| Demo data not reproducible | Medium | Demo fails on stage | Idempotent seeder + reset endpoint + fixed IDs/prices/random seed. |
| Testcontainers/pgvector unavailable in CI | Low/Medium | Tests not runnable | Use the pgvector image; document Docker requirement; unit-test math/rules without DB. |
| `.gitignore` ignores `*.md` | Low | Plan not committed | `git add -f IMPLEMENTATION_PLAN.md` if you want it versioned. |
| Package typo `saas_multi_tennant_app` | Low | Cosmetic | Do not rename now; high churn, no functional value. |

### 9.3 Scope questions to confirm

1. Is the committee evaluating a **backend + Swagger** demo, or do they expect a **UI**? This changes Phase 9 effort significantly.
2. Should negotiations be **per PO** or **per SKU/line**? The demo is SKU-centric; the data model can support both, but the MVP should pick one.
3. Is **multi-warehouse** in scope? If yes, add `warehouse_id` to ledger/balances/PO lines before Phase 1; retrofitting is expensive.
4. Is a **real vendor channel** (email/API) expected? The spec says simulated; confirm that a simulator is acceptable for the thesis.
5. Should the **RAG assistant** answer only from documents, or may it call deterministic business queries (e.g., exact last price)? Recommended: hybrid, with exact figures from SQL and narrative from RAG.
6. What is the **business-hours timezone** for tenants? Add `tenants.timezone`; default `UTC` or the demo’s local zone.
7. What is the **forecast window/horizon** for the demo? Recommended: 28-day window, lead time + 7-day review horizon.
8. What **currency** does the demo use? Store a tenant currency and vendor/PO currency; no FX conversion in the MVP.
9. Should **staff** see the approval queue read-only, or not at all? Recommended: no access; staff sees their own stock actions.
10. What **document formats** must be ingested? PDF/DOCX/TXT is the safe Tika scope; scanned images would need OCR (out of scope).
11. Is **audit immutability** required at the DB level (triggers/permissions) or is application-level append-only enough? Recommended: app-level + optional trigger for `stock_mvts` and `agent_run_logs`.
12. What **retention/privacy** rules apply to transcripts and uploaded contracts? Define before production; for the thesis, document retention as a limitation.
