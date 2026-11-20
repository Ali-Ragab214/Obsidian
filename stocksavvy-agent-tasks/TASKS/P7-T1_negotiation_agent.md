# P7-T1 — Vendor Negotiation Agent (flagship)

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 7 — Negotiation agent |
| Source section | `6.13` of the original plan |
| Depends on (spec) | P0-T2 (§6.2) (roles), P1-T1 (§6.3) (SKU), P2-T1 (§6.6) (vendor profile), P2-T2 (§6.7) (PO), P3-T1 (§6.8) (approvals/audit), P4-T1 (§6.9)/P4-T2 (§6.10) (RAG), P6-T1 (§6.12) (reorder draft; a manual PO also works). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1 |
| Effort | **L** — 8–10 days (the flagship; do not compress this). |
| Next task | P8-T1 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P7-T1 — "Vendor Negotiation Agent (flagship)" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3, P2-T1, P2-T2, P3-T1, P4-T1, P4-T2, P5-T1, P6-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Flagship. Every outbound message goes through an `ApprovalRequest`. The vendor simulator's floor price must never reach the agent's prompt. After this task run the **full 8-step demo** headlessly.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| E3 | Vendor Negotiation Agent | ❌ | No state machine, messages, simulation, approvals, or RAG. | Flagship negotiation flow |
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

### 6.13 Vendor Negotiation Agent (E3 — flagship)

**Goal:** run a multi-step, stateful, human-in-the-loop negotiation with a simulated vendor, grounded in this vendor’s historical prices and transcripts, and finalize the PO with a full auditable transcript.

**Depends on:** 6.2 (roles), 6.3 (SKU), 6.6 (vendor profile), 6.7 (PO), 6.8 (approvals/audit), 6.9/6.10 (RAG), 6.12 (reorder draft; a manual PO also works).

**Spring Boot approach**

- **Manual enum-based state machine** persisted on `NegotiationCase.status`. Each transition is a guarded service method; invalid transitions throw `IllegalStateException` → 409. This is easier to explain and test than Spring StateMachine, which adds a dependency and configuration for no benefit at this size.
- **States (exactly the spec):** `DRAFT_STRATEGY → AWAITING_APPROVAL → SENT → AWAITING_VENDOR_RESPONSE → EVALUATING_COUNTEROFFER → (AWAITING_APPROVAL → FINALIZED) | DRAFT_STRATEGY | ESCALATED_TO_HUMAN`.
- **Transition table:**

| From | Event | Guard | To | Side effect |
|---|---|---|---|---|
| `DRAFT_STRATEGY` | draft offer | round ≤ maxRounds | `AWAITING_APPROVAL` | RAG context + LLM/template message; create approval; log |
| `AWAITING_APPROVAL` | approval APPROVED | edited payload valid; offer ≤ target and ≥ sanity floor | `SENT` | record outbound `NegotiationMessage`; log |
| `AWAITING_APPROVAL` | approval REJECTED | — | `DRAFT_STRATEGY` (or `CANCELLED`) | record note; re-draft on manager request |
| `SENT` | send | — | `AWAITING_VENDOR_RESPONSE` | simulated send; log |
| `AWAITING_VENDOR_RESPONSE` | vendor reply | — | `EVALUATING_COUNTEROFFER` | parse price/terms; store message |
| `EVALUATING_COUNTEROFFER` | counter ≤ target | — | `AWAITING_APPROVAL` (final) | create final-terms approval |
| `EVALUATING_COUNTEROFFER` | target < counter ≤ walkAway & round < max | — | `DRAFT_STRATEGY` | round++; draft next counter; approval |
| `EVALUATING_COUNTEROFFER` | counter > walkAway or rounds exhausted | — | `ESCALATED_TO_HUMAN` | log reason; dashboard escalation |
| `AWAITING_APPROVAL` (final) | approval APPROVED | — | `FINALIZED` | update PO price/total; ingest transcript; log |
| `ESCALATED_TO_HUMAN` | manual finalize | manager | `FINALIZED` | override price/terms; log |

- **Grounding:** before drafting, retrieve (a) the last 3 PO lines for this vendor + SKU via **exact SQL**, (b) contract/catalog chunks filtered by `vendorId` (and `sku` if known), (c) prior negotiation transcripts for this vendor. Inject numbered context into the prompt. The opening offer is computed as a deterministic baseline (e.g., `min(target, lastPaid * 0.98)` or the target discount) and then optionally adjusted by the LLM within a hard bound. **The LLM cannot change the target or walk-away, and its offer is clamped to a code-defined sanity range (e.g., `target × 0.90` to `target`); the walk-away is a separate final-acceptance bound.** Store the baseline, final offer, and citations.
- **Price semantics (buy negotiation):** lower is better.
  - `listPrice` = current vendor catalog unit price.
  - `targetUnitPrice` = manager’s desired price (demo: 8% below list).
  - `walkAwayUnitPrice` = maximum acceptable; must be ≥ target.
  - Accept if vendor counter ≤ target; counter if ≤ walkAway and rounds remain; escalate if > walkAway or rounds exhausted.
- **Simulated vendor:** `SimulatedVendorResponder` reads `vendor_negotiation_profiles` (hidden floor ratio, concession rate) and responds deterministically:
  - if agent offer ≥ floor → accept at the offer (or floor);
  - else counter at `offer + (floor - offer) * concessionRate`, bounded by list;
  - if rounds exhausted → hold.
  Use a deterministic seed (negotiation id / vendor id) so the demo is repeatable. **Do not put the floor in the agent prompt**; the agent must not know it. Expose a manual `POST /negotiations/{id}/simulate-vendor-response` for a click-driven demo, and an `IMMEDIATE` mode for tests. Default `MANUAL` so the committee can see each state.
- **Approvals:** every outbound message (opening and each counter) creates an `ApprovalRequest`; final terms create a final approval. The payload includes `{round, offerUnitPrice, quantity, draftMessage, target, walkAway, citations}` and reasoning. The manager can edit the message/offer via `reviewedPayload`; the handler validates bounds and uses the edited values. This satisfies “approve/edit/reject before anything goes to the vendor.”
- **Finalization:** on final approval, update the PO lines to the agreed unit price, recompute totals, set PO `APPROVED`/`FINALIZED`, close the negotiation, and ingest a transcript document (`sourceType=NEGOTIATION_TRANSCRIPT`, `sourceRefId=negotiationId`) after commit. The transcript includes every message and the reasoning from `AgentRunLog`, so it becomes future RAG grounding.
- **Escalation:** set `ESCALATED_TO_HUMAN` + `escalation_reason`; show it on the dashboard. No silent stall. The manager can manually finalize (override) or cancel.
- **Transcript artifact:** `GET /negotiations/{id}/transcript` returns an ordered timeline of messages + approvals + run logs, with roles, timestamps, prices, and reasoning. This is defense artifact G2.
- **Trade-off:** real email/API vendor integration is explicitly out of scope per the spec; the simulation boundary is documented. A full negotiation policy (multi-issue: price, lead time, payment terms) is simplified to price + quantity; document this as a thesis limitation.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `negotiation_cases` | `id`, `purchase_order_id`, `vendor_id`, `status`, `round`, `max_rounds`, `target_unit_price`, `walk_away_unit_price`, `list_unit_price`, `current_offer_unit_price`, `agreed_unit_price`, `currency`, `strategy_summary`, `escalation_reason`, `run_id`, `version`, audit | FK PO/vendor; index `(status)`, `(vendor_id, created_at)`, `(purchase_order_id)`; check `walk_away >= target` |
| `negotiation_messages` | `id`, `negotiation_id`, `round`, `direction` (`AGENT/VENDOR/MANAGER/SYSTEM`), `message_type` (`OPENING_OFFER/COUNTER_OFFER/ACCEPT/REJECT/SYSTEM`), `unit_price`, `quantity`, `body text`, `terms jsonb`, `created_at`, `created_by` | index `(negotiation_id, created_at)`; FK |
| `purchase_orders` | add `negotiation_id` (nullable) | FK to `negotiation_cases` (or keep the FK on the case only; avoid a circular FK) |

**Classes**

- `agents/negotiation/NegotiationState.java`, `NegotiationCase.java` (entity), `NegotiationMessage.java`, `NegotiationService.java` + impl, `NegotiationStateMachine.java` (guarded transitions).
- `agents/negotiation/NegotiationContextBuilder.java` (SQL history + RAG).
- `agents/negotiation/NegotiationStrategyAgent.java` (opening/counter drafting; `LlmGateway` + template fallback).
- `agents/negotiation/SimulatedVendorResponder.java`, `VendorSimulationPolicy.java`.
- `agents/negotiation/NegotiationApprovalHandler.java` (implements `ApprovalHandler`).
- `agents/negotiation/NegotiationTranscriptIngestor.java` (after-commit RAG ingestion).
- `repositories/NegotiationCaseRepository.java`, `NegotiationMessageRepository.java`.
- `controllers/NegotiationController.java`.
- `requests/StartNegotiationRequest.java` (`{purchaseOrderId, targetUnitPrice, walkAwayUnitPrice, maxRounds}`), `SimulateVendorResponseRequest.java`, `ManualFinalizeRequest.java`.
- `responses/NegotiationResponse.java`, `NegotiationMessageResponse.java`, `NegotiationTranscriptResponse.java`.
- `resources/prompts/negotiation-opening-v1.st`, `negotiation-counter-v1.st`, `negotiation-evaluate-v1.st`.
- `db/migration/tenant/V7__negotiations.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/negotiations` | `StartNegotiationRequest` | negotiation (DRAFT_STRATEGY) | Manager |
| `GET` | `/api/v1/negotiations?status=&vendorId=&page=&size=` | — | `PageResponse<NegotiationResponse>` | Manager |
| `GET` | `/api/v1/negotiations/{id}` | — | negotiation + current approval | Manager |
| `GET` | `/api/v1/negotiations/{id}/messages` | — | ordered messages | Manager |
| `GET` | `/api/v1/negotiations/{id}/transcript` | — | messages + approvals + run logs (artifact) | Manager |
| `POST` | `/api/v1/negotiations/{id}/draft` | — | next approval created | Manager |
| `POST` | `/api/v1/negotiations/{id}/simulate-vendor-response` | `{counterUnitPrice?, body?}` (optional override for demo) | vendor message + state | Manager |
| `POST` | `/api/v1/negotiations/{id}/manual-finalize` | `{agreedUnitPrice, note}` | FINALIZED | Manager |
| `POST` | `/api/v1/negotiations/{id}/cancel` | `{reason}` | CANCELLED | Manager |

Approval decisions use `/api/v1/approvals/{id}/approve|reject`.

**Multi-tenancy considerations**

- Negotiation cases/messages live in the tenant schema; RAG context is retrieved only from this tenant and this vendor.
- The simulated vendor is internal; no external identity or tenant.
- Scheduled/LLM work runs under the tenant context captured at the start; `@Async` would need explicit propagation (prefer synchronous for the demo).
- Transcript ingestion is tenant-scoped and idempotent by `(sourceType, negotiationId)`.

**HITL / safety**

- **Nothing goes to the vendor without a manager approval.** Every outbound message has its own approval.
- Final terms require a second, explicit approval.
- LLM output is clamped/validated; it cannot change the walk-away, quantity, or vendor.
- Max rounds → escalation, never an infinite loop.
- The full transcript is immutable/auditable; edits are recorded in `reviewedPayload` and messages.
- The simulated vendor’s floor is not in the prompt, preventing the agent from “knowing” the answer.

**Testing**

- State machine: every valid/invalid transition (table-driven, no DB).
- Approval gating: no message is `SENT` without approval; edit path uses the edited payload; bounds validation.
- Simulated vendor: deterministic responses for offer below/at/above floor; max-round escalation.
- Accept path → final approval → PO updated → transcript ingested once.
- Escalation path → dashboard query shows it; no silent stall.
- RAG context: only this vendor/SKU; citations present in reasoning.
- LLM failure → deterministic template message still requires approval (demo remains runnable).
- Cross-tenant negotiation isolation.

**DoD**

- [ ] Demo: opening offer grounded in last 3 PO prices + contract/transcript citations.
- [ ] Manager approves/edits every outbound message.
- [ ] Simulated counter-offer is evaluated against target/walk-away/max rounds.
- [ ] Agreement → final approval → PO finalized → transcript searchable.
- [ ] Max rounds → escalation visible.
- [ ] Transcript endpoint returns the full artifact.

**Effort:** **L** — 8–10 days (the flagship; do not compress this).

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P8-T1
