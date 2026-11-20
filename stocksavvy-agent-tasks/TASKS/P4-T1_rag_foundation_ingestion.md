# P4-T1 — RAG foundation: ingestion, chunking, embeddings, pgvector

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 4 — RAG foundation (ingest + retrieve) |
| Source section | `6.9` of the original plan |
| Depends on (spec) | P0-T1 (§6.1) (tenant schema), P2-T1 (§6.6) (vendors/catalog), P2-T2 (§6.7) (PO lines); transcripts arrive in P7-T1 (§6.13). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1 |
| Effort | **L** — 6–8 days. |
| Next task | P4-T2 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P4-T1 — "RAG foundation: ingestion, chunking, embeddings, pgvector" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Requires the pgvector-enabled Postgres image from P0-T1. No test may call a real LLM/embedding provider: use the deterministic stub.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| D1 | RAG knowledge sources (contracts, catalogs, transcripts, stock reports) | ❌ | No document upload, no ingestion, no transcript generation. | Grounded answers |
| D2 | Ingestion pipeline (chunk → embed → pgvector + metadata) | ❌ | No pgvector dependency, no vector table, no embedding model, no chunker. | Grounded answers |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 6 | P0 | Docker DB lacks pgvector | `docker-compose.yml` uses `postgres:17.5` | Switch to a pgvector-enabled image (e.g., `pgvector/pgvector:pg17`) or install the extension; run `CREATE EXTENSION vector` once | 0/4 |
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

### 6.9 RAG foundation: ingestion, chunking, embeddings, pgvector (D1, D2)

**Goal:** ingest contracts, vendor catalog records, finalized PO price lines, negotiation transcripts, and stock reports into tenant-scoped pgvector chunks with metadata, so retrieval can be filtered by vendor/SKU/source.

**Spec coverage:** D1 (knowledge sources), D2 (chunk/embed/store with metadata).

**Depends on:** 6.1 (tenant schema), 6.6 (vendors/catalog), 6.7 (PO lines); transcripts arrive in 6.13.

**Spring Boot approach**

- **Spring AI** for `ChatClient`/`EmbeddingModel` (official Spring integration, one BOM). Verify the Spring AI BOM supports Spring Boot 4.1.x at implementation time; if not, use a thin `LlmGateway` HTTP adapter to an OpenAI-compatible endpoint. Do **not** adopt LangChain4j: two AI ecosystems add no value here, and the agent logic is deterministic state machines, not a LangChain chain.
- **Do not use `PgVectorStore` autoconfiguration.** It binds one table/schema and cannot follow schema-per-tenant requests. Instead:
  - Use Spring AI only for `EmbeddingModel` (and later `ChatClient`).
  - Store chunks in each tenant’s `knowledge_chunks` table.
  - Query with `JdbcTemplate`, schema-qualified (`"tenant_x".knowledge_chunks`) with a validated schema name. This is the key isolation decision and should be explained in the thesis.
  - Insert embeddings as `CAST(? AS vector)` from a `float[]` string; query with `embedding <=> CAST(? AS vector)`. Optionally add `com.pgvector:pgvector` for a typed `PGvector`; the string cast keeps the code dependency-free and explicit.
- **Dimension:** choose one and lock it in the migration. Recommended: **768** with Ollama `nomic-embed-text` for an offline, no-cost demo; alternative: 1536 with OpenAI `text-embedding-3-small`. Make it `app.rag.embedding-dimension` and document that changing it requires a migration + re-embedding. Use `vector(768)` and an HNSW index `vector_cosine_ops` (optional at demo scale).
- **Chunking (custom, explainable):**
  - Contracts: `SectionChunker` splits on headings/numbered clauses, target ~600–800 tokens, ~80-token overlap. Do not use a blind token splitter for contracts.
  - Vendor catalog: one chunk per `VendorProduct` row, templated text.
  - PO price history: one chunk per PO line, templated (`PO reference, date, vendor, SKU, qty, unit price, currency`).
  - Negotiation transcript: one chunk per round/message, including reasoning and terms.
  - Stock report: one chunk per product/week summary.
- **Metadata columns (explicit, indexed):** `source_type`, `vendor_id`, `sku`, `doc_date`, `document_id`. Do not hide these in a JSON blob; filters must be indexed.
- **Idempotency:** `knowledge_documents` unique `(source_type, source_ref_id)`; `content_hash` skips unchanged content; re-ingestion deletes old chunks and inserts new ones in one transaction.
- **Text extraction:** Apache Tika (or Spring AI’s Tika reader) for PDF/DOCX/TXT. Trade-off: Tika is a large dependency; PDFBox alone is lighter but only handles PDF. Use Tika for the thesis because vendor contracts arrive in mixed formats.
- **Async vs sync:** ingest uploads synchronously with status `PROCESSING → READY/FAILED` for a simple demo; background re-ingestion (catalog/PO/transcripts) runs after commit. If a document is large, use `@Async` with the tenant id captured explicitly (never rely on ThreadLocal in `@Async`).
- **Failure policy:** an ingestion failure must never fail the business transaction that triggered it (e.g., PO finalization). Use `@TransactionalEventListener(AFTER_COMMIT)` + try/catch + `status=FAILED` + log.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `knowledge_documents` | `id`, `source_type`, `source_ref_id`, `vendor_id`, `sku`, `title`, `content text`, `content_hash`, `doc_date`, `status`, `error`, audit | unique `(source_type, source_ref_id)`; index `(vendor_id, sku)`, `(source_type, doc_date)` |
| `knowledge_chunks` | `id`, `document_id`, `chunk_index`, `content text`, `token_count`, `source_type`, `vendor_id`, `sku`, `doc_date`, `embedding vector(768)`, `created_at` | FK `document_id`; index `(document_id)`, `(vendor_id, sku)`, `(source_type)`; HNSW on `embedding` (optional) |
| `rag_queries` | `id`, `question text`, `answer text`, `retrieved_chunk_ids jsonb`, `filters jsonb`, `latency_ms`, `model`, `created_by`, `created_at` | index `(created_at desc)`; used for the “sources” artifact and debugging |

Also: `CREATE EXTENSION IF NOT EXISTS vector;` once per database (public migration or init script).

**Classes**

- `agents/rag/KnowledgeSourceType.java`, `KnowledgeDocumentStatus.java`.
- `entities/KnowledgeDocument.java`, `KnowledgeChunk.java`, `RagQuery.java`.
- `repositories/KnowledgeDocumentRepository.java`, `KnowledgeChunkRepository.java`, `RagQueryRepository.java`.
- `agents/rag/ingest/KnowledgeIngestionService.java` + impl; `SectionChunker.java`, `RecordChunker.java`; `TextExtractor.java` (Tika).
- `agents/rag/ingest/IngestionEventListeners.java` (`@TransactionalEventListener` for vendor product/PO/negotiation events).
- `services/KnowledgeDocumentService.java` (upload/list/delete); `controllers/KnowledgeController.java`.
- `agents/rag/VectorStoreRepository.java` (JdbcTemplate insert/search; schema-qualified).
- `requests/KnowledgeUploadRequest.java` (multipart + sourceType/vendorId/sku/date); `responses/KnowledgeDocumentResponse.java`.
- `db/migration/tenant/V5__rag_knowledge.sql`; `db/migration/common/V4__enable_vector.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/knowledge/documents` | multipart file + `{sourceType, vendorId?, sku?, docDate?}` | `201` + document (status) | Manager |
| `GET` | `/api/v1/knowledge/documents?sourceType=&vendorId=&sku=&page=&size=` | — | `PageResponse<KnowledgeDocumentResponse>` | Manager |
| `GET` | `/api/v1/knowledge/documents/{id}` | — | document + chunk count/status | Manager |
| `DELETE` | `/api/v1/knowledge/documents/{id}` | — | `204` (soft delete + chunks) | Manager |
| `POST` | `/api/v1/knowledge/documents/{id}/reingest` | — | status | Manager |
| `POST` | `/api/v1/knowledge/reindex` | `{sourceType?}` | job summary | Manager |

**Multi-tenancy considerations**

- Documents/chunks are in the tenant schema. Vector search **must** be schema-qualified for `JdbcTemplate`; Hibernate routing does not apply to raw SQL.
- Never retrieve chunks across schemas. The tenant schema is the hard boundary; metadata filters are an additional narrowing, not the isolation mechanism.
- Embedding calls and chunk storage run under the tenant context; background re-indexing iterates tenants.
- Chat/retrieval logging goes to the tenant’s `rag_queries`.

**HITL / safety**

- Ingestion is read-only with respect to business data. It may store document text, which managers upload; no agent can upload on its own in the MVP.
- Do not ingest secrets or full personal data; document the retention policy (open question).
- A failed re-ingestion must not delete the previous good chunks until the new ones are ready (use a transaction or a version/status switch).

**Testing**

- Chunker: contract sections produce expected boundaries; catalog/PO rows produce one chunk per record.
- Ingestion idempotency: re-run does not duplicate; changed content replaces chunks.
- Tenant isolation: tenant A cannot retrieve tenant B chunks even with a forged `vendorId`.
- Embedding dimension mismatch → clear startup/test failure, not silent corruption.
- Metadata filters: vendor + SKU returns only matching chunks.
- Tika extraction for PDF/TXT (small fixtures).

**DoD**

- [ ] Upload a contract and a catalog row; both are searchable with metadata.
- [ ] Ingestion is idempotent and tenant-scoped.
- [ ] A failed ingestion does not break the triggering business transaction.
- [ ] `rag_queries` records retrieval for the artifact.
- [ ] pgvector extension and image are documented/reproducible.

**Effort:** **L** — 6–8 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P4-T2
