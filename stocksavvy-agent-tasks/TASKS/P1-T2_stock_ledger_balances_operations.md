# P1-T2 — Stock ledger, balances & stock operations

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 1 — SKU + stock ledger + stock operations |
| Source section | `6.4` of the original plan |
| Depends on (spec) | P0-T1 (§6.1), P0-T2 (§6.2), P1-T1 (§6.3). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1 |
| Effort | **L** — 6–8 days. |
| Next task | P1-T3 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P1-T2 — "Stock ledger, balances & stock operations" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

Shares `db/migration/tenant/V2__product_and_stock.sql` with P1-T1. If V2 is already applied locally, add the stock changes as a new migration number and update Appendix B in `00_CONTEXT/05_package_map_and_migrations.md`. `StockMvtService` must NOT extend `BasicService` (append-only ledger).

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| B2 | Ledger stock movements (append-only) | 🟡 | `StockMvt` entity + `stock_mvts` table exist. `StockMvtServiceImpl` is empty; no controller; no balance projection; `BasicService` still exposes `update`/`delete` (conflicts with append-only); `dateMvt` is `LocalDate`, insufficient for anomaly hours. | Full movement history |
| B3 | Staff stock counts / delivery receipts | ❌ | No receipt/count service, no `stock_counts`, no PO link, no validation of negative stock. | Staff updates real quantities |
| B5 | Full movement history per SKU | ❌ | `StockMvtRepository` has no query methods; no endpoint; no pagination/filtering. | Investigate discrepancies |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 5 | P0 | Ledger contract allows mutation | `StockMvtService extends BasicService` (exposes update/delete); `StockMvtServiceImpl` empty | Do not extend `BasicService` for stock; add append-only `StockLedgerService`; optional DB trigger to reject UPDATE/DELETE | 1 |
| 7 | P1 | `StockMvt.dateMvt` is `LocalDate` | Entity + tenant migration | Add `occurred_at timestamptz` (business date stays `date_mvt`); anomaly rules use `occurred_at` | 1 |
| 15 | P1 | No optimistic locking on stock | No `@Version`/`@Lock`; balances will lose updates | `stock_balances.version` + `SELECT ... FOR UPDATE` (or atomic update) in the ledger transaction | 1 |
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

### 6.4 Stock ledger, balances & stock operations (B2, B3, B5)

**Goal:** make stock movements append-only and authoritative, maintain a balance per SKU, let staff receive deliveries and record physical counts, and expose full per-SKU history.

**Spec coverage:** B2 (ledger), B3 (staff receipts/counts), B5 (movement history).

**Depends on:** 6.1, 6.2, 6.3.

**Spring Boot approach**

- **Append-only ledger:** `StockMvt` is immutable. Do not extend `BasicService` for stock. Add `StockLedgerService` with `post(PostMovementCommand)` only; no update/delete. Optionally add a PostgreSQL trigger in the tenant migration that raises on `UPDATE`/`DELETE` of `stock_mvts` — a strong, demonstrable invariant.
- **Balance projection:** `stock_balances` row per product, updated in the same transaction as the movement under a row lock (`SELECT ... FOR UPDATE`). Compute `balance_after` on the movement for audit. Provide `POST /stock/products/{id}/recompute-balance` (manager) to rebuild from the ledger if drift is suspected.
- **Movement types:** keep `TypeMvt.IN/OUT` (avoids changing the DB check constraint). Add `reason` (`PURCHASE_RECEIPT`, `SALE`, `CUSTOMER_RETURN`, `DAMAGE`, `COUNT_ADJUSTMENT`, `MANUAL_ADJUSTMENT`, `OPENING_BALANCE`, `OTHER`) and `source_type`/`source_id` (e.g., `PURCHASE_ORDER`, PO id).
- **Timestamp:** keep `date_mvt` as the business date; add `occurred_at timestamptz` (default now) for anomaly hours and ordering.
- **Receipts:** `POST /stock/receipts` records an IN movement linked to a PO line (optional PO), increments balance, and (optionally) marks the PO line received. This is Sam’s “goods arrived” flow.
- **Stock counts:** `POST /stock/counts` records physical `countedQty`, snapshots `systemQty`, computes `variance`, and posts one `COUNT_ADJUSTMENT` IN or OUT movement for the difference. The count record is kept for audit and anomaly analysis. Never overwrite the balance directly.
- **Insufficient stock:** reject OUT movements that would make `on_hand < 0` with 409, except an explicit manager override for `COUNT_ADJUSTMENT` (physical counts can reveal shrinkage). Document the rule.
- **Concurrency:** `stock_balances.version` (`@Version`) plus `FOR UPDATE`; a retry on optimistic-lock failure is not needed if the row lock is used, but keep `@Version` for safety.
- **History:** `GET /stock/products/{id}/movements?from=&to=&type=&page=` returns the ledger, newest first, with `balanceAfter`, reason, actor, and source.
- **Low-stock query:** `GET /stock/low-stock` (also used by 6.5) returns products where `on_hand <= reorder_point` with a suggested shortfall. `reorder_point` is `alert_threshold` for the MVP; add a separate column later only if needed.
- **Trade-off:** maintaining a balance is denormalized; a derived `SUM` is simpler but slow and hard to lock. Maintained balance + recompute endpoint is the defensible choice.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `stock_mvts` (existing) | add `occurred_at timestamptz not null`, `reason varchar(40) not null`, `source_type varchar(40)`, `source_id varchar(255)`, `balance_after integer`, `unit_cost numeric(19,4)`; make `product_id NOT NULL` | index `(product_id, occurred_at desc)`, `(product_id, date_mvt)`, `(reason, occurred_at)`; optional immutability trigger |
| `stock_balances` (new) | `product_id` (PK/FK), `on_hand integer not null default 0`, `reserved integer not null default 0`, `version bigint`, `updated_at` | PK `product_id`; check `on_hand >= 0` (unless overridden) |
| `stock_counts` (new) | `id`, `product_id`, `counted_qty`, `system_qty`, `variance`, `note`, `counted_by`, `counted_at`, `adjustment_movement_id` | index `(product_id, counted_at desc)`; FK to `stock_mvts` (nullable) |

**Classes**

- `services/StockLedgerService.java` + `services/impl/StockLedgerServiceImpl.java`; `services/StockCountService.java` + impl.
- `entities/StockBalance.java`, `entities/StockCount.java`, `entities/MovementReason.java`; modify `entities/StockMvt.java`.
- `repositories/StockBalanceRepository.java` (with `@Lock(PESSIMISTIC_WRITE)` finder), `StockCountRepository.java`; extend `StockMvtRepository` with history/sum queries.
- `controllers/StockController.java` (movements, receipts, counts, history, low-stock, recompute).
- `requests/StockReceiptRequest.java`, `StockCountRequest.java`, `StockMovementRequest.java`; `responses/StockMovementResponse.java`, `StockBalanceResponse.java`, `StockCountResponse.java`, `LowStockItemResponse.java`.
- `mappers/StockMvtMapper.java`, `StockBalanceMapper.java`.
- Events: `events/StockMovementPostedEvent.java` (used by anomaly in 6.14).
- `db/migration/tenant/V2__product_and_stock.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `POST` | `/api/v1/stock/movements` | `{productId, type, quantity, reason, occurredAt?, comment?}` | `201` + movement + balance | Staff, Manager |
| `POST` | `/api/v1/stock/receipts` | `{productId, quantity, purchaseOrderId?, unitCost?, occurredAt?}` | `201` + movement + balance | Staff, Manager |
| `POST` | `/api/v1/stock/counts` | `{productId, countedQty, note?}` | `201` + count + adjustment movement | Staff, Manager |
| `GET` | `/api/v1/stock/products/{id}/movements?from=&to=&type=&page=&size=` | — | `PageResponse<StockMovementResponse>` | Manager, Staff |
| `GET` | `/api/v1/stock/products/{id}/balance` | — | `StockBalanceResponse` | Manager, Staff |
| `GET` | `/api/v1/stock/low-stock` | — | `List<LowStockItemResponse>` | Manager |
| `POST` | `/api/v1/stock/products/{id}/recompute-balance` | — | recomputed balance | Manager |

**Multi-tenancy considerations**

- Ledger, balances, and counts live in the tenant schema; no cross-tenant queries.
- `stock_balances` is keyed by product id (UUID), which is already tenant-local.
- The anomaly listener must not run inside the posting transaction (see 6.14); it reads after commit with the same tenant context.
- Background recompute jobs iterate tenants explicitly.

**HITL / safety**

- Stock is mutated only by explicit staff/manager commands; agents never write stock.
- Append-only ledger; corrections are new movements, not edits.
- Insufficient-stock rule prevents silent negative balances; manager override is explicit and audited.
- Every movement records the actor (`created_by`) and reason.

**Testing**

- Pure unit tests for balance math and variance.
- Integration: receipt increments, OUT decrements, insufficient OUT → 409, count variance posts the right adjustment.
- Concurrency: two parallel OUTs cannot both succeed against a balance of 1.
- Append-only: attempt to update/delete a movement → rejected (service has no method; DB trigger test if enabled).
- History pagination/filtering; balance = sum of ledger after a random sequence.
- Cross-tenant ledger isolation.

**DoD**

- [ ] Balance always equals the ledger sum after a test sequence.
- [ ] `stock_mvts` has no update/delete path.
- [ ] Staff can receive goods and record a count end-to-end.
- [ ] History and low-stock endpoints work.
- [ ] Concurrency test passes without negative on-hand.
- [ ] `StockMovementPostedEvent` is published after commit.

**Effort:** **L** — 6–8 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P1-T3
