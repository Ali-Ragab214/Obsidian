# P2-T1 — Vendors & pricing

> **Status:** ⬜ not started  _(change to ✅ when every DoD item is verified)_

| Field | Value |
|---|---|
| Phase | 2 — Vendors + purchase orders |
| Source section | `6.6` of the original plan |
| Depends on (spec) | P1-T1 (§6.3). |
| Tasks that must be done before this one | P0-T1, P0-T2, P1-T1, P1-T2, P1-T3 |
| Effort | **M** — 3–4 days. |
| Next task | P2-T2 |

## 🤖 Prompt for the agent (copy & paste this block)

```text
You are implementing task P2-T1 — "Vendors & pricing" of the StockSavvy Spring Boot project.

1. Read, in order: 00_CONTEXT/00_agent_rules.md, 00_CONTEXT/01_project_snapshot.md, the latest entries in HANDOFF_LOG.md, then the SPEC section of this file.
2. Verify prerequisites in the REAL code (not only in PROGRESS.md): P0-T1, P0-T2, P1-T1, P1-T2, P1-T3. If one is missing, STOP and report it.
3. Re-scan the parts of the repo this task touches; earlier tasks may have changed them. Cite real file/class names.
4. Implement ONLY this task, exactly as specified below. Do not start the next task. Do not rename packages or refactor unrelated code.
5. Follow "Suggested internal order". Write all tests listed in the spec. Run ./mvnw clean verify.
6. Tick the DoD items in PROGRESS.md only if you actually verified them. Fix the "Known issues" listed in this file.
7. Append an entry to HANDOFF_LOG.md (files changed, migrations added, deviations, open questions).
8. Stop and report a short summary. Wait for the next task.
```

## ⚠️ Task notes

`db/migration/tenant/V3__vendors_and_purchase_orders.sql` is **shared** with P2-T2. Write the vendor tables here; P2-T2 adds the PO tables (same file if not yet applied, otherwise a new version).

## 🎯 Spec features covered (from the gap matrix)

| # | Feature (spec) | Status | Notes / evidence | Related user story |
|---|---|---|---|---|
| B6 | Vendors module (catalog, price, lead time) | ❌ | No `Vendor`, `VendorProduct`, `VendorNegotiationProfile`; no tables/endpoints. | Vendor selection, pricing |

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

### 6.6 Vendors & pricing (B6)

**Goal:** maintain a vendor master, a per-vendor product catalog (price, lead time, MOQ), and the configuration needed for simulation and selection.

**Depends on:** 6.3.

**Spring Boot approach**

- Entities + CRUD service + controller, following the existing mapper/DTO conventions.
- Vendor selection uses exact SQL over `vendor_products` (price, lead time) plus last-paid history from PO lines; RAG is used for contract terms/history narrative, not for exact numbers.
- Price history is **derived from finalized PO lines**, not stored twice. A `vendor_product_prices` history table is unnecessary for the MVP; if list prices change over time, keep `valid_from`/`valid_to` and query the active row.
- Vendor negotiation profile is separate from the vendor master so simulation parameters are explicit and testable.
- Trade-off: one catalog row per `(vendor, product)` is simple but does not keep list-price history. The thesis can explain that actual paid prices come from POs; add temporal rows later if needed.

**Entities / tables**

| Table | Key columns | Constraints / indexes |
|---|---|---|
| `vendors` | `id`, `name`, `code`, `email`, `phone`, `default_lead_time_days`, `currency`, `active`, `notes`, audit | unique `code`; index `(active)` |
| `vendor_products` | `id`, `vendor_id`, `product_id`, `unit_price numeric(19,4)`, `currency`, `lead_time_days`, `min_order_qty`, `active`, `valid_from`, `valid_to`, audit | unique `(vendor_id, product_id)`; index `(product_id, active)`; FK to `vendors`, `products` |
| `vendor_negotiation_profiles` | `vendor_id` (PK/FK), `floor_price_ratio numeric(5,4)` (e.g., 0.90), `concession_rate numeric(5,4)`, `max_rounds_default`, `active` | one row per vendor; simulation-only, documented |

**Classes**

- `entities/Vendor.java`, `VendorProduct.java`, `VendorNegotiationProfile.java`.
- `repositories/VendorRepository.java`, `VendorProductRepository.java`, `VendorNegotiationProfileRepository.java`.
- `services/VendorService.java` + impl (CRUD, add/update catalog row, list vendors for a product).
- `controllers/VendorController.java`.
- `mappers/VendorMapper.java`, `VendorProductMapper.java`.
- `requests/VendorRequest.java`, `VendorProductRequest.java`; `responses/VendorResponse.java`, `VendorProductResponse.java`, `VendorPriceHistoryResponse.java`.
- `db/migration/tenant/V3__vendors_and_purchase_orders.sql`.

**API endpoints**

| Method | Path | Request | Response | Roles |
|---|---|---|---|---|
| `GET` | `/api/v1/vendors?active=&page=&size=` | — | `PageResponse<VendorResponse>` | Manager, Staff (read) |
| `GET` | `/api/v1/vendors/{id}` | — | `VendorResponse` | Manager, Staff |
| `POST` | `/api/v1/vendors` | `VendorRequest` | `201` + vendor | Manager |
| `PUT` | `/api/v1/vendors/{id}` | `VendorRequest` | vendor | Manager |
| `DELETE` | `/api/v1/vendors/{id}` | — | `204` (deactivate) | Manager |
| `GET` | `/api/v1/vendors/{id}/products` | — | catalog rows | Manager, Staff |
| `POST` | `/api/v1/vendors/{id}/products` | `VendorProductRequest` | catalog row | Manager |
| `PUT` | `/api/v1/vendors/{id}/products/{productId}` | `VendorProductRequest` | catalog row | Manager |
| `GET` | `/api/v1/vendors/{id}/products/{productId}/price-history` | — | last PO prices + list price | Manager |
| `PUT` | `/api/v1/vendors/{id}/negotiation-profile` | `{floorPriceRatio, concessionRate, maxRoundsDefault}` | profile | Manager |

**Multi-tenancy considerations**

- Vendors are tenant-local (a tenant’s supplier list). Schema isolation applies.
- No global vendor directory in the MVP; a shared directory would be a platform feature and a cross-tenant data concern.
- Vendor catalog changes trigger RAG re-ingestion (6.9) asynchronously or after commit.

**HITL / safety**

- Not an agent feature. Manager-only writes.
- Simulation parameters (`floor_price_ratio`) are visible to managers in the DB/API? **Do not expose the floor ratio in the manager UI**; expose only `maxRounds` and display terms. Keep the floor in the negotiation profile accessible to the simulator, not the negotiation agent (so the agent cannot “know” the vendor’s floor). This is important for a credible simulation.

**Testing**

- CRUD + unique `(vendor, product)` conflict → 409.
- Price history returns latest PO line, not catalog list price.
- Vendor selection query returns only active rows for the tenant.
- Cross-tenant vendor isolation.

**DoD**

- [ ] Vendor catalog supports price, lead time, MOQ, currency.
- [ ] Last-paid price is derivable from PO lines (after 6.7).
- [ ] Simulation profile is stored but not leaked to the agent prompt.
- [ ] Manager can manage vendors via Swagger.

**Effort:** **M** — 3–4 days.

---

## ✅ Completion protocol

- [ ] `./mvnw clean verify` is green (all new tests included)
- [ ] Every DoD item above is verified and ticked in `PROGRESS.md`
- [ ] Known issues listed in this file are fixed (ticked in `PROGRESS.md`)
- [ ] Cross-tenant isolation test exists for every new table/endpoint
- [ ] `HANDOFF_LOG.md` entry written
- [ ] Next task: P2-T2
