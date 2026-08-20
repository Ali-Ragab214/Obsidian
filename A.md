# Contract Lifecycle Management (CLM) Platform

## Complete System Design, Architecture & Implementation Roadmap

---

## 1. The Business Problem

**The problem in plain terms:** every company that signs more than a handful of contracts eventually loses track of them. Contracts live in email attachments, shared drives, someone's desktop, or a filing cabinet. Nobody can answer simple questions quickly: "Which vendor contracts auto-renew next month?" "Do we have any NDA with Company X?" "Which leases have a below-market termination penalty?" The information exists, but it's locked inside unstructured PDF/Word documents that nobody has time to re-read.

**Who has this problem:** legal departments, procurement/vendor management teams, HR (employment contracts), finance (renewal cost forecasting), and operations in mid-size to large companies (roughly 200+ active contracts is where spreadsheets stop working).

**Why traditional contract management (folders + spreadsheets) fails:**

- No structured lifecycle — contracts silently expire because nobody was tracking a date.
- No approval trail — anyone with drive access can "finalize" a contract without the right sign-off.
- No searchability of _content_ — you can find a file by name, not by clause.
- No audit trail for compliance (SOX, ISO, GDPR-adjacent obligations, etc.).
- Knowledge walks out the door when the one person who "remembers what's in the contracts" leaves.

**What happens when contracts are poorly managed:** missed renewal negotiation windows (locked into unfavorable auto-renewals), missed termination windows (stuck paying for unused services), compliance violations discovered too late, disputes where nobody can quickly locate the governing clause, and duplicated/contradictory agreements with the same vendor.

**Value this system provides:** a single source of truth with an enforced approval workflow, proactive deadline management, and the ability to ask natural-language questions across the whole contract portfolio instead of manually reading hundreds of PDFs.

**What makes this different from a simple CRUD app:** the differentiators are (1) a real state machine with authorization rules per transition, (2) an approval workflow engine, (3) versioning/amendment history that must remain legally traceable, and (4) an AI/RAG layer that must be _trustworthy_ — it has to cite evidence and must never be treated as a legal authority. A basic "contracts table with a status column" is not this system; the workflow rules and the RAG evidence chain are the actual engineering problems.

**Where AI provides real value:**

- Answering semantic questions across many documents at once (something no human wants to do manually).
- Surfacing clauses that match a pattern ("show me restrictive liability clauses") across the whole portfolio.
- Summarizing a long contract into digestible obligations.
- Flagging _candidates_ for risk review.

**Where AI must NOT be trusted:**

- Making the actual approval/rejection decision.
- Being the sole source for a legal determination ("this clause is enforceable/unenforceable").
- Silently changing contract data or status.
- Being cited as a legal record without pointing back to the exact source text and page.

**Traditional Backend Features vs AI/RAG Features (explicit split):**

|Traditional Backend|AI/RAG|
|---|---|
|Contract CRUD, lifecycle state machine|Semantic search / natural-language Q&A|
|Approval workflow engine|Clause extraction & classification|
|Document storage & versioning|Risk flagging (advisory only)|
|Notifications/reminders (date-driven)|Contract summarization|
|RBAC & audit logging|Contract comparison|
|Renewal/amendment tracking|Missing-clause detection|

---

## 2. Actors & Role/Permission Model

|Actor|Who they are|Can see|Can create|Can modify|Can approve/reject|Key workflows|
|---|---|---|---|---|---|---|
|**Requester / Contract Owner**|Any employee who initiates a contract (e.g., a department lead needing a vendor)|Their own contracts + contracts they're a stakeholder on|New contract drafts, upload documents|Their own contracts while in Draft|No|Draft → Submit for review|
|**Contract Manager**|Owns the CLM process operationally (often legal ops or procurement ops)|All contracts (subject to confidentiality tier)|Any contract, on behalf of others|Metadata, parties, workflow assignment|Can approve low-risk/standard contracts if delegated|Oversees full lifecycle, assigns reviewers|
|**Legal Reviewer / Legal Officer**|In-house counsel or legal team member|Contracts routed to them, or all if authorized|Comments, redlines/annotations (metadata, not the file itself)|Clause tags, risk annotations|Approve/reject at the legal-review step|Legal Review stage|
|**Department Approver**|Budget/business owner of the contract (e.g., department head)|Contracts belonging to their department|—|—|Approve/reject at business-approval step|Business Approval stage|
|**Finance/Compliance Approver** (optional, for high-value contracts)|Finance controller or compliance officer|Contracts above a value/risk threshold|—|—|Final sign-off for high-value contracts|Final Approval stage|
|**Administrator**|System/IT administrator|Everything (system-level, not necessarily document content)|Users, roles, departments, system config|Permissions, workflow templates|No approval authority by default|User & config management|
|**Auditor**|Internal/external audit function|Read-only access to everything + full audit log|Nothing|Nothing|No|Compliance review, read-only|
|**AI Assistant (system actor)**|Not a human — a system principal with its own scoped permissions|Only documents the requesting user is authorized to see (never bypasses RBAC)|Suggestions/insights (not contract data)|Nothing directly|No|Answers questions, flags risk, always attributed|

**Key permission principles:**

- No one can approve a contract they authored or own (segregation of duties — see Business Rules).
- Confidential contracts (e.g., executive employment, M&A) are visible only to an explicitly granted access list, regardless of role.
- The AI Assistant's retrieval is always scoped by the querying user's actual permissions — it cannot be used as a permission-escalation shortcut.

---

## 3. The Contract Lifecycle

```
Draft → Internal Review → Legal Review → Business Approval → (Final Approval*) → Active
                                                                                    │
                                                        ┌───────────────────────────┼───────────────────────────┐
                                                        ▼                           ▼                           ▼
                                                    Amendment                   Renewal                   Expiration/Termination
                                                        │                           │
                                                        └──────────► back to Active ◄┘
```

* Final Approval only applies to contracts above a configurable value/risk threshold.

|State|Meaning|Who moves it here|Conditions|Actions allowed|On enter|On exit|Approval needed?|
|---|---|---|---|---|---|---|---|
|**Draft**|Being authored, not yet real|Requester creates it|None|Edit freely, attach documents, delete|Assign owner|Validate required fields present|No|
|**Internal Review**|Owner's team sanity-checks it before legal|Requester submits|Required metadata + at least one document attached|Comment, request changes, send back to Draft|Notify assigned reviewer|—|No (informal)|
|**Legal Review**|Legal checks clauses/risk|Contract Manager routes it|Passed internal review|Annotate, tag clauses, request changes, reject to Draft|Notify legal reviewer|Legal sign-off recorded|Yes (legal)|
|**Business Approval**|Budget owner approves|Legal approves|Legal sign-off exists|Approve/reject|Notify approver|Decision recorded, immutable|Yes (business)|
|**Final Approval** (conditional)|High-value sign-off|Business approver approves, threshold met|Contract value/risk ≥ threshold|Approve/reject|Notify finance/compliance|Decision recorded|Yes (conditional)|
|**Active**|Legally in force|System, on final approval|All approvals complete|View, initiate amendment/renewal, terminate|Lock core terms, start reminder schedule|—|No|
|**Amendment (in progress)**|A change to an active contract is being drafted|Contract Manager / Owner|Contract is Active|Same review/approval sub-flow as new contract, scoped to the delta|Create new version, keep prior version immutable|Merge into contract, version incremented|Yes (same chain, can be lightweight for minor changes)|
|**Renewal (in progress)**|Extending term / renegotiating|Contract Manager, triggered by reminder or manually|Contract nearing expiration|Negotiate new terms, may re-enter review chain|Link to original contract|New version becomes Active, or contract moves to Expired if not renewed|Yes, if terms change materially|
|**Expired**|Term ended without renewal|System (date-driven)|End date reached, no active renewal|View only, initiate late renewal (exception path)|Notify stakeholders|—|No|
|**Terminated**|Ended early/voluntarily|Authorized approver|Termination reason required|View only|Record termination reason & date|—|Yes (termination is itself an approval-gated action)|
|**Archived**|Retained for records, out of active management|System, after retention period|Configurable retention period post-expiration/termination|Read-only, export|—|—|No|

**Edge cases to design for:**

- Contract rejected at Legal Review → must return to Draft with reviewer comments attached, not silently deleted.
- Amendment started but abandoned → original contract must remain valid and unaffected.
- Contract expires while an amendment is mid-review → business rule needed (auto-extend review deadline vs. force expiration).
- Two renewals attempted simultaneously (race condition) → must be prevented with an optimistic lock / single "renewal in progress" flag.
- Confidential contract visibility during workflow — approvers must be granted temporary access without permanently widening the ACL.
- Termination of a contract that has active amendments in flight.

---

## 4. Core Domain Model

- **Contract** — the aggregate root. Represents the legal agreement conceptually (not a specific file). Owns lifecycle state, parties, key dates, and links to versions/documents.
- **ContractVersion** — an immutable snapshot representing the terms at a point in time (initial signing, or after an amendment). Contracts always have ≥1 version; amendments create new versions rather than mutating history.
- **ContractDocument** — the actual file(s) attached to a version (signed PDF, scanned copy, source docx). Multiple documents can attach to one version (main doc + exhibits).
- **Party** — an organization or individual that is a signatory or counterparty. Reusable across contracts (a vendor appears in many contracts).
- **Clause** (metadata, not raw text ownership) — a tagged, categorized reference to a specific provision inside a document (e.g., "Termination", "Liability Cap", "Non-Compete") produced either by manual tagging or AI extraction, always pointing back to a location in the source document.
- **ApprovalWorkflow** — a definition of what approval steps a given contract type/value requires.
- **ApprovalStep** — an instance of one step within a workflow run for a specific contract (who, when, decision, comments).
- **Renewal** — a record linking an expiring contract to its renewal outcome (renewed with new version, or lapsed).
- **Amendment** — a record of a change made to an Active contract, linked to the version it produced.
- **Reminder** — a scheduled, date-driven trigger (e.g., "60 days before expiration") tied to a contract.
- **Notification** — an actual message delivered to a user, generated from a Reminder or a workflow event.
- **User** — system principal, linked to roles, department, and permission scope.
- **Department** — organizational grouping used for ownership and approval routing.
- **AuditLog** — an immutable, append-only record of every state-changing action (who, what, when, before/after).
- **AIInsight** — a distinct entity for AI-generated output (answer, risk flag, summary) that stores the source citations, confidence, and a link to human review status. Kept structurally separate from actual contract data so it's clear an AI insight is _not_ contract truth.

**Important relationships/rules:**

- A `ContractVersion` is immutable once created; an amendment produces a _new_ version, never edits an old one.
- `Clause` records always carry a pointer (document ID + page/offset) back to source text — never store extracted clause text as free-standing "truth" disconnected from its source.
- `AIInsight` is never directly linked as "part of" the contract's legal terms — it's an annotation layer.

---

## 5. Functional Features (with priority)

|Feature|Purpose|Main user|Priority|
|---|---|---|---|
|Contract Management (CRUD + lifecycle)|Core entity and state machine|All|**MVP**|
|Party Management|Reusable counterparties|Contract Manager|**MVP**|
|Document Management (upload/versioning)|Store and version source files|All|**MVP**|
|Approval Workflow Engine|Enforce sign-off chain|Legal, Approvers|**MVP**|
|Notifications (expiration/renewal reminders)|Prevent missed deadlines|All|**MVP**|
|RBAC & Confidentiality|Access control|All|**MVP**|
|Audit Logging|Compliance trail|Auditor|**MVP**|
|RAG Q&A over contracts|Semantic search & question answering|Legal, Contract Manager|**MVP** (this is the differentiator — see rationale below)|
|Clause tagging/extraction (AI-assisted)|Structure unstructured text|Legal|**Important**|
|Risk flagging|Surface candidates for review|Legal|**Important**|
|Contract comparison|Diff two contracts/versions|Legal|**Advanced**|
|Obligation extraction & tracking|Track ongoing duties (e.g., quarterly reports owed)|Contract Manager|**Advanced**|
|Renewal/termination clause analytics|Portfolio-level insight|Legal/Finance|**Advanced**|
|Full-text keyword search (non-AI)|Fallback simple search|All|**Important**|
|E-signature integration|Replace manual signing|Requester|**Optional/Future**|
|Multi-tenant SaaS support|Serve multiple companies|Admin|**Optional/Future**|

**Why RAG Q&A is MVP and not "advanced":** the entire premise of this project (stated in your brief) is that this system is differentiated by semantic understanding, not just CRUD. A CLM without any AI feature is indistinguishable from a portfolio project you could build in a weekend. However, the RAG _pipeline_ can be scoped small for MVP (a handful of documents, basic retrieval, no reranking) and made more sophisticated later — that's the right way to include it without overbuilding.

---

## 6. AI & RAG Architecture

### Pipeline

```
Contract Upload
   → Document Extraction (text + layout, e.g., PDF → text with page anchors)
   → Cleaning/Normalization
   → Chunking (semantic, ~500–1000 tokens, with overlap, preserving page/section anchors)
   → Embedding generation (async job)
   → Vector storage (with metadata: contract_id, version_id, document_id, page, clause_type if tagged)
   → [User asks a question]
   → Query embedding
   → Vector similarity retrieval, filtered by the user's permitted contract set (RBAC filter BEFORE similarity, not after)
   → (Optional) Reranking of top-K candidates for precision
   → LLM synthesis with retrieved chunks as context, instructed to answer only from provided context
   → Answer + citations (contract, document, page/section) returned together, never separated
```

**What gets indexed:** the extracted text of active, non-superseded contract versions. Superseded versions are _not_ included in default retrieval (but remain queryable in an explicit "historical" search mode) so users don't get answers from outdated terms by accident.

**Metadata stored alongside each vector:** `contract_id`, `version_id`, `document_id`, `department_id`/`tenant_id`, `confidentiality_tier`, `page_number`, `chunk_index`, `clause_type` (if tagged). This metadata is what makes access control and evidence citation possible — it is not optional.

**Isolation between contracts / companies:** every retrieval query applies a mandatory metadata filter derived from the _querying user's_ actual RBAC-permitted contract set, computed server-side, before the similarity search runs. The vector database is never queried "openly" and then filtered afterward — that pattern leaks embeddings-level information and is a real security bug pattern to avoid.

**Versions:** only the current (or specifically Active) version of a contract is embedded into the default index; when an amendment produces a new version, the old version's vectors are excluded from default retrieval but not deleted (needed for historical/audit queries).

**Updates:** re-embedding is triggered by the async pipeline whenever a new version's document is uploaded, not by an in-place edit (because ContractVersion is immutable).

**Deleted/expired contracts:** expired/terminated contracts remain indexed but are tagged with status metadata; whether they appear in results depends on the query context (a legal researcher asking "show me all NDAs, including expired" should get them; a default "what's currently active" query should not).

**Citations/evidence:** every AI answer must return the literal chunk(s) it used, with a pointer back to contract + document + page, rendered next to the answer. If the system cannot produce a citation for a claim, it should say it doesn't have enough information rather than answering unsupported.

**Reducing hallucination:**

- Constrain the LLM prompt to answer _only_ from retrieved context, explicitly instructed to say "not found in the provided contracts" when the context doesn't support an answer.
- Always show the citations so a human can verify.
- Keep answers evidence-scoped rather than encouraging free-form legal conclusions.

**Evaluating RAG quality:** maintain a small, hand-labeled test set of realistic questions with known correct answers/citations; measure retrieval precision/recall (did the right chunk get retrieved) separately from answer quality (did the LLM use it correctly); spot-check citation accuracy manually, especially early on.

### AI Use Cases Ranked (Business Value × Complexity)

|Use case|Business value|Technical complexity|Priority|
|---|---|---|---|
|Contract Q&A (RAG)|High|Medium|**MVP**|
|Semantic contract search|High|Medium (shares pipeline with Q&A)|**MVP**|
|Summarization|Medium-High|Low-Medium|**Important**|
|Clause discovery/tagging|High|Medium-High|**Important**|
|Missing clause detection|Medium|Medium|**Advanced**|
|Risk detection (flagging)|High (but sensitive)|High|**Advanced**|
|Contract comparison|Medium|High|**Advanced**|
|Obligation extraction|Medium|High|**Advanced/Future**|

---

## 7. AI Safety & Human-in-the-Loop

- **AI-generated insight**: any output (answer, summary, risk flag, clause tag) is stored as an `AIInsight`, clearly separated from contract data, always carrying: the model output, the source citations, and a confidence indicator.
- **Human review**: risk flags and clause tags generated by AI start in a "suggested" state and require a human (Legal Reviewer) to accept/reject before they're treated as authoritative metadata.
- **Approval**: AI can never execute an approval/rejection transition. It can only annotate a contract in review with observations.
- **Auditability**: every AI insight is logged the same way human actions are — who asked, what was retrieved, what was answered, and whether a human later acted on it.
- **AI limitations to state explicitly in the product**: AI does not provide legal advice, does not guarantee completeness (it might miss a clause phrased unusually), and its confidence is not a legal certainty.

---

## 8. Background Jobs & Async Processing

|Job|Why async|Trigger|On failure|Retry?|Idempotent?|
|---|---|---|---|---|---|
|Document text extraction|Can be slow (large PDFs, OCR)|Document upload|Mark document `processing_failed`, notify uploader|Yes, limited retries with backoff|Yes — re-running produces the same extracted text|
|Embedding generation|LLM/embedding API latency, batching efficiency|After successful extraction|Mark chunk batch failed, retry|Yes|Yes — re-embedding the same chunk overwrites, doesn't duplicate|
|Contract indexing (vector upsert)|Same as above|After embeddings generated|Retry, alert if repeated failure|Yes|Yes (upsert by chunk ID)|
|Expiration/renewal reminders|Date-driven, scheduled, not user-triggered|Daily scheduled scan|Log and alert on missed run|N/A (next run catches up)|Yes — recomputing "contracts expiring in 60 days" is naturally idempotent|
|Notification delivery (email etc.)|Don't block the triggering action on an external mail provider|Any workflow event or reminder|Dead-letter queue, retry|Yes|Should be — use a delivery-attempt ID to avoid duplicate sends|
|AI analysis (summarization, risk flagging)|LLM latency, cost control (batch when possible)|Manual trigger or on version activation|Mark insight generation failed, allow manual retry|Yes|Yes — regenerating overwrites the prior suggestion|

General principle: anything that calls an external/slow service (LLM, embeddings, email) or that isn't needed synchronously for the user to get a response should be async. All jobs should be designed idempotent because retries are guaranteed to happen eventually in any real system.

---

## 9. Redis Usage

|Use case|Recommendation|Why|
|---|---|---|
|Caching (contract detail reads, permission lookups)|**Recommended**|Contract detail views and permission checks are read-heavy; caching reduces DB load meaningfully.|
|Distributed locks (e.g., "only one renewal process at a time per contract")|**Recommended**|Prevents the race-condition edge case identified in the lifecycle section.|
|Background job support (queue backing, e.g., with a job library)|**Recommended**|Needed regardless for the async pipeline above; Redis is a reasonable lightweight backing store before reaching for Kafka.|
|Rate limiting (API abuse protection, AI query throttling)|**Recommended**|AI calls cost money; rate limiting per user protects both cost and abuse.|
|Session-related use cases|**Optional**|Only relevant if you're not using pure stateless JWT auth; with JWT, largely unnecessary.|
|Temporary data (e.g., "in-progress renewal draft" scratch state)|**Optional**|Nice convenience, not essential — could live in Postgres as a draft row instead.|
|RAG-related caching (cache repeated identical queries' answers)|**Optional**, useful later|Helps cost/latency for common questions, but adds staleness-management complexity; not needed for MVP.|

**Do not** use Redis just as a second database for primary contract data — Postgres is the source of truth throughout.

---

## 10. PostgreSQL, Object Storage & Vector Storage

- **PostgreSQL**: all structured, relational, transactional data — Contract, ContractVersion (metadata, not files), Party, User, ApprovalWorkflow/Step, Reminder, Notification, AuditLog, and clause _metadata_ (tags, categories, pointers). This is the system of record.
- **Object/File Storage** (S3-compatible): the actual binary documents (PDF/DOCX files). Never store binary blobs in Postgres — it bloats the database, hurts backup/restore times, and Postgres isn't optimized for large object serving. Store only a reference (URI/key) in Postgres.
- **Vector Storage**: chunk embeddings + retrieval metadata for RAG.

**pgvector vs. dedicated vector DB:** for this project's realistic scale (a portfolio/enterprise-mid-size system, not billions of vectors), **pgvector is the right choice**. It keeps the architecture simpler (one database technology to operate, transactional consistency between contract metadata and vectors, no extra service to run), and its performance is entirely sufficient below roughly millions of vectors. Reach for a dedicated vector database (e.g., Qdrant/Pinecone/Weaviate) only if you outgrow pgvector's performance at real scale — don't add that complexity upfront. This directly answers your instruction to avoid unnecessary technology: a **separate vector database is not justified for this project's MVP or realistic V2 scale.**

---

## 11. Security

- **Authentication**: standard credential-based login issuing JWTs (or session tokens), with refresh-token rotation.
- **Authorization**: RBAC at the role level (what actions a role can generally take) _plus_ resource-level checks (can this specific user act on this specific contract) — role alone is not sufficient because department scoping and confidentiality tiers both narrow access further.
- **Resource-level access**: every contract read/write endpoint checks not just "does this role allow X" but "is this user in this contract's authorized set" (owner, department member, explicitly granted, or admin/auditor).
- **Contract confidentiality**: a `confidentiality_tier` field with an explicit access list for the highest tier, checked on every read — including AI retrieval (see below).
- **Tenant isolation** (if multi-tenant, see Section 12): every query scoped by tenant ID at the data-access layer, not just the application layer, to make cross-tenant leakage structurally hard.
- **Document access**: file downloads go through a signed, time-limited URL generated only after the same authorization check as viewing the contract — never expose raw, permanent object storage URLs.
- **API security**: standard practices — input validation, rate limiting, HTTPS only, least-privilege service accounts for background jobs.
- **Audit logging**: every state-changing action and every document access is logged immutably (who, when, what, from where).
- **Sensitive operations** (termination, confidentiality tier change, permission grants) require additional confirmation and are always logged with elevated visibility to auditors.
- **AI access control**: this is the one people forget — **the AI must enforce the exact same permission boundary as the regular API.** A user cannot ask the AI a question and get an answer sourced from a contract they aren't authorized to view. This is enforced by filtering the vector retrieval by the user's permitted contract IDs _before_ similarity search, as described in Section 6.

**Preventing User A from accessing User B's confidential contracts:** the combination of (1) resource-level authorization checks on every read path — API and AI alike, (2) confidentiality-tier ACLs checked server-side, and (3) never trusting a client-supplied "which contracts am I allowed to see" list — the permitted set is always computed server-side from the user's actual role/department/grants at query time.

---

## 12. Multi-Tenancy

|Approach|Trade-off|
|---|---|
|Single-tenant|Simplest; fine if this serves one company only|
|Shared DB, shared schema (tenant_id column)|Efficient, scalable, but requires strict discipline to never miss a tenant_id filter|
|Shared DB, separate schema per tenant|Stronger isolation, harder migrations, more operational overhead|
|Separate database per tenant|Strongest isolation, expensive operationally, only justified at real SaaS scale|

**Recommendation: build the MVP as single-tenant, but design the schema with a `tenant_id`/`organization_id` column from day one** (shared schema pattern) even if you don't build tenant-switching UI yet. This is the pragmatic middle ground: it costs almost nothing to add the column now, and it means if you ever want to demonstrate "this could be SaaS," you're not doing a disruptive migration later. Actually implementing full multi-tenant onboarding, billing, and cross-tenant admin tooling is **explicitly out of MVP scope** — that's real product surface area, not a backend afterthought, and would meaningfully delay the parts of this project that are actually differentiating (workflow + RAG).

---

## 13. Architecture Recommendation

**Modular Monolith, not Microservices.** Microservices solve organizational scaling problems (many independent teams) and independent-deployment needs. A solo/portfolio project has neither — microservices here would only add network complexity, distributed transaction headaches, and deployment overhead without a corresponding benefit. A modular monolith gives you clean module boundaries (which is the actual valuable skill to demonstrate) without the operational tax.

**Feature-Based Clean Architecture + Vertical Slices — confirmed as the right choice**, with a caveat on CQRS. Your instinct here is good and I won't talk you out of it:

- **Feature-based/vertical-slice** organization (folder-per-feature: `contracts/`, `workflow/`, `rag/`, etc., each containing its own use cases) keeps the codebase navigable as it grows and maps cleanly to the feature list in Section 5.
- **Clean Architecture principles** (domain logic independent of framework/infrastructure) are worth applying at the level of "keep business rules out of controllers and out of JPA entities," without going full ports-and-adapters ceremony everywhere — that level of purity has diminishing returns for a project of this size and can slow you down while learning.
- **CQRS**: apply it _pragmatically_, not dogmatically. Full CQRS with separate write/read models and eventual consistency is overkill for this system's actual query patterns. What's genuinely useful is the **lighter form of CQRS** — separate "command" classes (that mutate state, go through validation/business rules) from "query" classes (that just read and shape data for the UI, can bypass some domain ceremony for performance). This gives you the main benefit (simpler, more honest read paths) without building an event-sourced/eventually-consistent system you don't need.
- **Event-driven processing**: use it selectively, specifically for the async jobs identified in Section 8 (a lightweight in-process or Redis-backed event/queue for "document uploaded → trigger extraction," etc.), not as an architecture-wide messaging backbone between "microservices" that don't exist.

**Bottom line recommendation:** Modular Monolith · Feature-based Vertical Slices · Pragmatic Clean Architecture · Light CQRS (command/query separation without full event sourcing) · Targeted async events for background jobs.

---

## 14. System Components

|Component|Responsibility|
|---|---|
|Identity & Access|Authentication, RBAC, resource-level authorization|
|Contract Management|Contract aggregate, lifecycle state machine, versions|
|Party Management|Counterparty records|
|Document Management|Upload, storage reference, retrieval, signed URLs|
|Workflow|Approval workflow definitions and step execution|
|Notification|Reminder scheduling, message delivery|
|Search|Keyword search (traditional)|
|AI/RAG|Extraction, chunking, embedding, retrieval, Q&A, insights|
|Audit|Immutable event logging, audit queries|
|Scheduling|Cron-like jobs (expiration scans, reminder triggers)|

**Communication:** within the monolith, modules communicate through well-defined internal interfaces (application services), not by reaching into each other's data models directly. State-changing events (e.g., "ContractActivated") are published internally so Notification/AI modules can react without Contract Management needing to know about them — this is where the "light event-driven" piece from Section 13 lives.

---

## 15. Important Business Rules (prioritized)

1. **Segregation of duties**: a user cannot approve a contract they authored or are the listed owner of.
2. A contract cannot move to **Active** until all required approval steps for its workflow (determined by contract type/value) are complete.
3. An **Active** contract's core terms cannot be edited directly — only through a versioned **Amendment**, which goes through review.
4. **Rejection at any review/approval step** returns the contract to Draft (or the prior amendable state) with the rejection reason attached — it is never silently deleted.
5. An approval, once recorded, **cannot be edited**, only superseded by a new decision if the workflow explicitly allows re-review.
6. **Termination** is itself an approval-gated action, not a simple status flip — someone with authority must record why.
7. Confidential-tier contracts require **explicit access grants**; department membership alone is not sufficient.
8. Only one **renewal** or **amendment** process may be in progress per contract at a time (prevents race conditions).
9. **Expiration is date-driven and automatic** — the system must not rely on a human remembering to close out an expired contract.
10. Every state transition and every document access must produce an **audit log entry** — this is not optional/toggleable.
11. AI-generated insights are **advisory** and require human acceptance before being treated as authoritative contract metadata.

---

## 16. Main User Journeys / Use Cases

1. **Create Contract** — Requester fills metadata (type, parties, department, value) → saved as Draft.
2. **Upload Contract Document** — Requester attaches file(s) to the current version → triggers async extraction/indexing.
3. **Submit Contract for Review** — Owner submits Draft → system validates required fields → moves to Internal Review, notifies reviewer.
4. **Legal Review & Annotate** — Legal Officer opens contract, reviews AI-suggested clause tags, adds comments, approves or rejects.
5. **Business Approval** — Department Approver reviews legal sign-off + summary, approves or rejects.
6. **Contract Goes Active** — On final required approval, system locks terms, schedules reminders, activates.
7. **Renew Contract** — Reminder fires N days before expiration → Contract Manager initiates renewal → may re-enter review if terms change → produces new version or lets it lapse to Expired.
8. **Amend Active Contract** — Owner/Manager proposes a change → new version drafted → goes through (possibly lightweight) review → merges into contract history on approval.
9. **Search Contracts Semantically (RAG)** — User asks a natural-language question → system retrieves permitted, relevant chunks → LLM answers with citations.
10. **Ask AI About a Specific Contract** — User opens one contract, asks a scoped question (e.g., "summarize the termination clause") → answer restricted to that contract's chunks.
11. **Review AI Risk Flags** — Legal Officer sees AI-suggested risk flags queued for review → accepts or dismisses each.
12. **Receive Expiration Reminder** — System notifies relevant stakeholders at configured intervals before expiration.
13. **Terminate Contract Early** — Authorized user initiates termination with a reason → goes through an approval gate → contract moves to Terminated.
14. **Audit Review** — Auditor queries full history of a contract or a user's actions, read-only.

---

## 17. MVP Definition

**MVP must include:** contract CRUD + full lifecycle state machine, document upload/versioning, one realistic approval workflow (2–3 steps), RBAC with confidentiality tiers, expiration reminders (scheduled job), audit logging, and a working RAG pipeline (upload → extract → chunk → embed → retrieve → answer with citations) over a modest document set. This satisfies your explicit requirement that the MVP demonstrate **Contract Lifecycle + Workflow + Document Management + RAG**, not just CRUD.

**V2:** clause tagging/extraction (AI-assisted), risk flagging with human review queue, renewal/amendment flows fully built out, keyword search fallback, richer notification channels.

**Advanced:** contract comparison, obligation extraction/tracking, portfolio-level analytics dashboards, reranking in the RAG pipeline, RAG evaluation harness.

**Future:** e-signature integration, true multi-tenant SaaS onboarding, mobile app, external API for integrations (e.g., procurement systems).

---

## 18. Development Phases & Roadmap

Each phase lists Goal · Key Deliverables · Prerequisite Understanding · Dependencies · Definition of Done · Estimated time (at 2–3 hrs/day).

**Phase 0 — Domain Design & Requirements** Goal: nail down the domain model, lifecycle, and workflow rules on paper before touching code. Deliverables: this document refined into your own diagrams (state machine diagram, ER-level domain diagram, actor/permission matrix). Prerequisite understanding: none beyond what's in this document. DoD: you can explain the lifecycle and role model to someone else without notes. Estimate: **6–8 hrs (~3 days)**

**Phase 1 — Project Foundation** Goal: repo structure, module boundaries, base Spring Boot setup, Docker Compose for local Postgres/Redis. Deliverables: runnable skeleton app, module folder structure reflecting vertical slices, Flyway baseline. Prerequisite: Spring Boot basics, Docker basics. Dependencies: Phase 0. DoD: app boots, connects to Dockerized Postgres, health check works. Estimate: **10–14 hrs (~5 days)**

**Phase 2 — Identity & Security** Goal: authentication, RBAC, resource-level authorization foundation. Deliverables: User/Role/Department models, login/JWT, permission-check mechanism reusable across features. Prerequisite: Spring Security fundamentals, JWT concepts. Dependencies: Phase 1. DoD: can log in, get a token, and a protected endpoint correctly rejects unauthorized roles. Estimate: **14–18 hrs (~7 days)**

**Phase 3 — Contract Management Core** Goal: Contract, ContractVersion, Party entities + lifecycle state machine (without workflow approvals yet — just valid state transitions). Deliverables: CRUD, transition validation, resource-level access checks integrated. Prerequisite: state machine design patterns, transactional boundaries. Dependencies: Phase 2. DoD: can create a contract, move it through states via API with rules enforced, unauthorized users blocked. Estimate: **18–24 hrs (~9 days)**

**Phase 4 — Document Management** Goal: file upload, object storage integration, versioned document linkage, signed-URL download. Deliverables: upload endpoint, storage abstraction, download with authorization check. Prerequisite: S3-compatible storage basics (e.g., MinIO locally). Dependencies: Phase 3. DoD: can upload a document to a contract, download it only if authorized, file never publicly reachable. Estimate: **12–16 hrs (~6 days)**

**Phase 5 — Approval Workflow Engine** Goal: configurable multi-step approval chain, segregation-of-duties enforcement. Deliverables: ApprovalWorkflow/Step models, approve/reject endpoints, notifications on step change. Prerequisite: workflow/state-machine patterns, understanding of your own business rules list. Dependencies: Phase 3. DoD: a contract can be routed through a real 2–3 step approval chain end to end, with rejection returning it correctly. Estimate: **16–20 hrs (~8 days)**

**Phase 6 — Background Jobs & Notifications** Goal: async job infrastructure, expiration/renewal reminder scheduling, notification delivery. Deliverables: scheduled job for expiration scanning, async job runner, email/notification delivery with retry. Prerequisite: async processing patterns, idempotency concepts, Spring's `@Scheduled` or a job library. Dependencies: Phase 3, Phase 2 (for who to notify). DoD: a contract nearing its configured reminder window reliably produces a notification, verified via test data with manipulated dates. Estimate: **12–16 hrs (~6 days)**

**Phase 7 — RAG Pipeline (Ingestion)** Goal: document extraction → chunking → embedding → pgvector storage, fully async and triggered by document upload. Deliverables: extraction pipeline, chunking strategy, embedding job, vector storage with metadata. Prerequisite: Spring AI basics, embedding models, pgvector extension, chunking strategies. Dependencies: Phase 4, Phase 6 (async infra). DoD: uploading a contract document results in retrievable, correctly-scoped vectors in Postgres/pgvector. Estimate: **18–24 hrs (~9 days)**

**Phase 8 — RAG Pipeline (Retrieval & Q&A)** Goal: permission-filtered retrieval, LLM synthesis with citations, contract Q&A endpoint. Deliverables: retrieval query with RBAC filter applied server-side, prompt design constraining answers to context, citation formatting. Prerequisite: prompt engineering basics, RAG evaluation concepts. Dependencies: Phase 7, Phase 2 (permission scoping). DoD: asking a real question returns an answer with accurate citations, and a user cannot get an answer sourced from a contract they can't view (test this explicitly). Estimate: **16–20 hrs (~8 days)**

**Phase 9 — AI Features (Insights Layer)** Goal: summarization, clause tagging suggestions, risk-flag suggestions, all stored as reviewable `AIInsight` records. Deliverables: insight generation jobs, human review/accept-reject UI-facing endpoints. Prerequisite: structured LLM output patterns (JSON mode / structured outputs). Dependencies: Phase 8. DoD: AI-suggested clause tags/risk flags appear in a review queue and require explicit human acceptance before becoming "official" metadata. Estimate: **14–18 hrs (~7 days)**

**Phase 10 — Audit & Compliance** Goal: comprehensive, immutable audit logging across all state-changing actions and document access; audit query endpoints. Deliverables: audit log model, interceptor/aspect for automatic logging, auditor-facing read endpoints. Prerequisite: AOP/interceptor patterns in Spring. Dependencies: all prior phases (logs everything they produce) — can be introduced incrementally starting from Phase 2, but a dedicated pass ensures completeness. DoD: every state transition and document download appears correctly in the audit log with before/after data. Estimate: **10–14 hrs (~5 days)**

**Phase 11 — Testing Pass** Goal: fill in unit/integration/security/workflow test coverage across the system (see Section 20 for what to test where — don't treat this purely as a final phase; write tests alongside each phase too). Deliverables: test suite covering lifecycle rules, permission boundaries, RAG citation accuracy on a fixed test set. Dependencies: all functional phases. DoD: critical business rules (Section 15) each have at least one automated test proving they're enforced. Estimate: **14–18 hrs (~7 days, spread across the project in practice)**

**Phase 12 — Docker & Deployment Packaging** Goal: full Docker Compose for the whole stack (app, Postgres, Redis, object storage), production-style config separation. Deliverables: docker-compose.yml, environment-based configuration, basic deployment documentation. Prerequisite: Docker Compose fundamentals. Dependencies: all functional phases complete. DoD: `docker compose up` brings up a fully working system from a clean machine. Estimate: **8–12 hrs (~5 days)**

**Total estimated hours: ~168–222 hours**

---

## 19. Time Estimation Tracks

|Track|Pace|Approx. total duration|
|---|---|---|
|**Fast Track** (cut scope: skip Phase 9 insights layer and comparison-adjacent features, minimal testing)|3 hrs/day|~9–10 weeks|
|**Realistic Track** (full plan as scoped above, 2–3 hrs/day)|2.5 hrs/day avg|~13–16 weeks (~3–4 months)|
|**Deep Learning Track** (same scope, but you pause each phase to properly study unfamiliar concepts before building)|2 hrs/day|~18–22 weeks (~4.5–5.5 months)|

These are deliberately conservative because RAG-related phases (7–9) and the workflow engine (Phase 5) are where underestimation typically happens — both involve real design decisions, not just typing out known patterns.

---

## 20. Learning Roadmap (per phase)

|Phase|Must Know Before Starting|Nice to Know|Can Learn While Building|
|---|---|---|---|
|1 – Foundation|Spring Boot basics, Docker basics|Gradle/Maven multi-module setup|Flyway specifics|
|2 – Identity/Security|Spring Security filter chain, JWT|OAuth2 concepts|Refresh token rotation details|
|3 – Contract Core|State machine design|DDD aggregate concepts|JPA/Hibernate nuances|
|4 – Documents|Basic object storage concepts (S3 API)|Signed URL generation|MinIO local setup specifics|
|5 – Workflow|Workflow/approval-chain modeling|Saga pattern (conceptually)|Notification triggering wiring|
|6 – Jobs/Notifications|Async processing, idempotency|Message queue concepts|Spring `@Async`/`@Scheduled` specifics|
|7 – RAG Ingestion|Embeddings conceptually, chunking strategies|pgvector specifics|Spring AI API details|
|8 – RAG Retrieval|Prompt engineering basics, RBAC-filtered retrieval concept|Reranking concepts|LLM API specifics, citation formatting|
|9 – AI Insights|Structured LLM output patterns|Confidence scoring approaches|Review-queue UI wiring|
|10 – Audit|AOP/interceptor patterns|Compliance frameworks (context only)|Query optimization for audit logs|
|11 – Testing|Unit vs integration testing basics|Testcontainers|RAG evaluation harness design|
|12 – Deployment|Docker Compose basics|Production config management (secrets)|Multi-service networking details|

---

## 21. Testing Strategy

- **Unit Tests**: domain logic — lifecycle transition rules, business rule validation (e.g., "cannot self-approve"), chunking logic. Write these alongside every phase, not just at the end.
- **Integration Tests**: repository layer against a real (Testcontainers) Postgres, workflow engine end-to-end within the app, storage integration against MinIO.
- **API Tests**: endpoint-level tests for each major use case in Section 16, including negative cases (unauthorized access attempts).
- **Security Tests**: explicit tests proving User A cannot read User B's confidential contract, cannot approve their own contract, cannot bypass RBAC via the AI Q&A endpoint.
- **Workflow Tests**: full lifecycle path tests (Draft → Active), rejection-and-return paths, amendment/renewal paths, race-condition prevention (concurrent renewal attempts).
- **RAG Evaluation**: a fixed set of ~20–30 realistic Q&A pairs with known correct citations; measure retrieval accuracy and citation correctness after any pipeline change.
- **AI Response Evaluation**: spot-check for hallucination (does the answer ever state something not present in cited chunks); confirm the model correctly says "not found" when appropriate.
- **End-to-End Tests**: a small number of full user-journey tests (e.g., create → review → approve → active → renew) exercised through the API.

Testing should be introduced starting Phase 2 (security tests are cheap and critical early), not deferred entirely to Phase 11 — Phase 11 is for filling gaps and adding the RAG evaluation harness, not writing your first test.

---

## 22. DevOps & Deployment

**Local development architecture:** Docker Compose running the Spring Boot app, PostgreSQL (with pgvector extension), Redis, and a local S3-compatible store (MinIO). LLM/embedding calls go to a real external provider (or a mocked one during early development to control cost).

**Production-style architecture (conceptual, not built in MVP):** containerized app behind a load balancer, managed Postgres (with pgvector) and managed Redis, managed object storage (real S3 or equivalent), external LLM provider, plus monitoring/logging aggregation. The point of building it this way locally is that the same container images and config pattern would carry forward to a real deployment with minimal change — you're not building a toy that needs to be rearchitected to go to production.

**Infrastructure components to plan for:** Application (Spring Boot), PostgreSQL, Redis, Object Storage, Vector capability (pgvector, part of Postgres — not separate infra), LLM/embedding provider, Monitoring/Logging.

---

## 23. Observability

- **Application logs**: structured logging (JSON) for all state transitions and errors.
- **Errors**: centralized error tracking, especially around the AI pipeline (extraction failures, embedding failures, LLM timeouts).
- **Performance**: response time tracking on contract/workflow endpoints and on the RAG query path specifically (retrieval time vs. LLM synthesis time, tracked separately).
- **Background jobs**: success/failure counts, queue depth, retry counts for the async pipeline.
- **AI latency & token usage**: track per-query token consumption and cost — this matters both for engineering discipline and because it's a real production concern you can speak to in an interview.
- **RAG retrieval quality**: ongoing sampling against your evaluation set (Section 21), not just a one-time check.
- **Failed document processing**: dashboard/alert on documents stuck in a failed extraction/embedding state.
- **Security events**: failed auth attempts, authorization denials, especially repeated denials from one user (possible probing).

---

## 24. Why This Project Is CV-Worthy

**Backend Engineering:** a real state machine with enforced business rules, transactional integrity across multi-step workflows, clean modular structure.

**System Design:** deliberate architecture choices (monolith vs. microservices, CQRS-lite, storage strategy) made with explicit trade-off reasoning — not default choices.

**Database:** relational modeling for a genuinely non-trivial domain (versioning, immutability of history, confidentiality tiers), plus pgvector integration.

**Security:** RBAC + resource-level + confidentiality-tier authorization enforced consistently across both traditional API and AI retrieval paths — this dual enforcement is a detail many candidates get wrong and is worth highlighting explicitly.

**Distributed/Async Processing:** a real background job pipeline with idempotency and retry design, not just "@Async slapped on a method."

**AI/RAG:** a properly scoped, permission-aware, citation-grounded RAG system — the access-control-aware retrieval design in particular is the kind of detail that separates a toy RAG demo from something that reflects real engineering judgment.

**DevOps:** containerized, reproducible local environment mirroring a production topology.

**What's genuinely impressive vs. plain CRUD:** the lifecycle/workflow engine, the security model (especially AI-aware RBAC), and the RAG evidence/citation design are the standout pieces. Plain contract CRUD, party management, and basic notification delivery are necessary scaffolding but not what will make this stand out — don't over-invest polish time there at the expense of the differentiating pieces.

---

## Final Deliverables Summary

1. **One-page overview**: Section 1.
2. **Actors & responsibilities**: Section 2.
3. **Core domain concepts**: Section 4.
4. **Contract lifecycle**: Section 3.
5. **Feature map**: Section 5.
6. **Architecture recommendation**: Section 13.
7. **Technology stack with justification**: Sections 11–13 (Postgres/Storage/Vector), Section 10 (Redis).
8. **RAG architecture**: Section 6.
9. **Security model**: Section 11.
10. **Async/background processing model**: Section 8.
11. **MVP scope**: Section 17.
12. **Development phases**: Section 18.
13. **Timeline**: Section 19.
14. **Learning roadmap**: Section 20.
15. **Main use cases**: Section 16.
16. **Important business rules**: Section 15.
17. **Testing strategy**: Section 21.
18. **Deployment architecture**: Section 22.
19. **CV value**: Section 24.
20. **Future enhancements**: Section 17 ("Future" row) — e-signature integration, full multi-tenant SaaS, mobile app, external integrations, reranking and RAG evaluation automation, obligation-tracking dashboards, contract comparison tooling.

---

### A note on your stated preferences

You asked me to challenge your assumptions where warranted. I largely agreed with your architectural instincts (Feature-Based Clean Architecture + light CQRS, modular monolith), but pushed back on two things worth flagging explicitly:

- **Dedicated vector database**: not justified — pgvector is the right call for this project's real scale.
- **Full CQRS/event sourcing**: not justified — a pragmatic command/query split without eventual-consistency machinery gets you the real benefit without the operational cost.

Everything else in your technology list (Java, Spring Boot, Spring Security, Spring AI, PostgreSQL, Redis, Docker, Flyway) is justified and used as you intended.