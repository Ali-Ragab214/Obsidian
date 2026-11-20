# P1-T1 — SKU management

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 1 — SKU + stock ledger + stock operations |
| Source section | `6.3` of the original plan |
| Depends on (spec) | P0-T1 (§6.1), P0-T2 (§6.2). |
| Tasks that must be done before this one | P0-T1, P0-T2 |
| Effort | **M** — 2–3 days. |
| Next task | P1-T2 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P1-T1 — "SKU management" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

The migration file `db/migration/tenant/V2__product_and_stock.sql` is **shared** with P1-T2. Create it here with the product changes and extend it (or add the stock part) in P1-T2 — never edit an already-applied migration.

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| B1 | SKU management | 🟡 | `Product`/`ProductRepository`/`ProductServiceImpl` exist; `reference` acts as SKU. No controller, no `id` in response, no UoM/category id, update bug (duplicate check always runs), mapper NPE if category is null. | Manager maintains SKUs |

## 🐞 Known issues this task must fix (from the conflict log)

| # | Sev | Conflict | Evidence | Action | Phase |
|---|---|---|---|---|---|
| 8 | P1 | Product update is broken | `ProductServiceImpl.update` always calls `checkIfProductAlreadyExistsByReference`; rebuilds a new `Product` and sets only `id` | Load existing entity, exclude self from duplicate check, update fields; guard category null | 1 |
| 9 | P1 | `ProductResponse` incomplete | No `id`/`categoryId`; `category.getName()` NPEs | Add `id`, `categoryId`; null-safe mapper | 1 |
| 13 | P1 | No authorization rules | No `@PreAuthorize`/`@Secured` anywhere | Add `@ManagerOnly`, `@StaffOnly`, `@PlatformAdminOnly` meta-annotations and enforce | 0/1 |
| 16 | P1 | No indexes for planned queries | Tenant V1 has only unique constraints | Add composite indexes per table (listed in each spec) | 1–8 |
| 17 | P2 | Soft delete inconsistent | `deleted` columns exist; category/product delete is hard; `findAll` includes deleted | Standardize soft delete for master data; exclude `deleted` in queries | 1 |

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

### 6.3 SKU management (B1)

**Goal:** expose a complete, validated SKU master so products can be managed through the API (today only the category API is reachable).

**Depends on:** 6.1, 6.2.

**Spring Boot approach**

- Add a `ProductController` following REST style (`/api/v1/products`), reusing `ProductServiceImpl` after fixes.
- Keep `reference` as the SKU and add a `sku` alias only if the UI needs it; do **not** create a second identifier. Add `unitOfMeasure` (e.g., `UNIT`, `KG`, `L`) and `active` for completeness; both are simple columns.
- Use Jakarta Validation on `ProductRequest`: `@NotBlank reference`, `@NotBlank name`, `@Positive price`, `@PositiveOrZero alertThreshold`, `@NotNull categoryId`.
- Soft delete (`deleted = true`) instead of hard delete; all list/get queries filter `deleted = false`.
- Fix `ProductServiceImpl.update` to load the existing entity and exclude itself from the duplicate-reference check. Fix `ProductMapper` to include `id` and `categoryId`, and guard a null category.
- Add `availableQuantity` to `ProductResponse` only after 6.4 exists (a read model query joining `stock_balances`); until then expose `0` or omit. Do not fake it.
- Trade-off: adding a separate `Sku` entity is unnecessary; `Product` is the SKU in this domain.

**Entities / tables**

| Table | Change |
|---|---|
| `products` (tenant) | Add `unit_of_measure varchar(20) default 'UNIT'`, `active boolean default true`; keep `reference` unique, `price`, `alert_threshold`, `category_id`; index `(category_id)`, `(deleted, active)`, `(reference)` |

**Classes**

- `controllers/ProductController.java`
- Modify `services/impl/ProductServiceImpl.java`, `mappers/ProductMapper.java`, `requests/ProductRequest.java`, `responses/ProductResponse.java`
- `ProductRepository`: `findByReferenceIgnoreCaseAndDeletedFalse`, `findAllByDeletedFalse(Pageable)`, `findByIdAndDeletedFalse`
- `db/migration/tenant/V2__product_and_stock.sql`

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `GET` | `/api/v1/products?page=&size=&search=&lowStock=` | — | `PageResponse<ProductResponse>` | Manager, Staff |
| `GET` | `/api/v1/products/{id}` | — | `ProductResponse` | Manager, Staff |
| `POST` | `/api/v1/products` | `ProductRequest` | `201` + `ProductResponse` (with id) | Manager |
| `PUT` | `/api/v1/products/{id}` | `ProductRequest` | `ProductResponse` | Manager |
| `DELETE` | `/api/v1/products/{id}` | — | `204` (soft delete) | Manager |
| `GET` | `/api/v1/products/by-sku/{reference}` | — | `ProductResponse` | Manager, Staff |

**Multi-tenancy considerations**

- `ProductRepository` is tenant-schema scoped by Hibernate routing; no `tenant_id` column is added.
- Reference uniqueness is per tenant schema (correct for schema-per-tenant).
- Never accept a tenant id in the request.

**HITL / safety**

- Not an agent feature. Manager-only writes; staff read-only.
- No stock mutation from product edits.

**Testing**

- CRUD integration test in a provisioned tenant.
- Duplicate reference on create → 409; update with own reference → 200 (regression test for the current bug).
- Soft-deleted product is invisible and cannot be fetched by id.
- Cross-tenant: product A not visible to tenant B.

**DoD**

- [ ] Product CRUD reachable via Swagger and tests.
- [ ] `ProductResponse` has `id`, `categoryId`, no NPE for uncategorized products.
- [ ] Soft delete works and is filtered everywhere.
- [ ] Validation errors return 400 with field messages.
- [ ] Cross-tenant isolation test passes.

**Effort:** **M** — 2–3 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P1-T2
