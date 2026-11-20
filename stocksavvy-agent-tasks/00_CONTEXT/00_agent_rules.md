# 00 — Agent rules of engagement (read before EVERY task)

## Golden rules
1. **One task at a time.** Implement only the task you were given. Never start the next one.
2. **Verify, don't assume.** Before coding, re-scan the repo parts the task touches. Earlier tasks change the code. Cite real file/class names. If something is missing, write "not found".
3. **Do not redesign what works.** Keep the schema-per-tenant model, package-by-layer layout, manual mappers, `BasicService`, `PageResponse`, `ErrorResponse`. Do NOT rename the `saas_multi_tennant_app` package.
4. **Additive changes.** Never edit an already-applied Flyway migration. Add a new version. Every new tenant migration must also reach existing tenants via `TenantMigrationService`.
5. **Tenant safety.** Never accept a tenant id from a request. Business data lives only in `tenant_<code>` schemas. Raw SQL/`JdbcTemplate` must schema-qualify with a validated identifier. Background jobs set and clear `TenantContext` explicitly (in `finally`).
6. **Human in the loop is architectural.** Every agent writes `AgentRunLog` at every step and uses `ApprovalRequest` before any spend or external message. Spend is never committed autonomously. The anomaly agent never modifies stock.
7. **Deterministic numbers.** Java computes quantities, prices, balances, thresholds. The LLM writes explanations/messages only, with a deterministic fallback. No test calls a real LLM (`LlmGateway` has a stub).
8. **Simple and defensible beats clever.** This is a graduation project that will be examined.
9. **Ambiguity:** do not guess silently. Put it in `HANDOFF_LOG.md` under "Open questions" and pick the plan's recommended default.

## Workflow for every task
1. Read `01_project_snapshot.md`, the task file, and the latest entries of `HANDOFF_LOG.md`.
2. Check prerequisites in the real code.
3. Post a short plan: files to create/modify, migration number, endpoints.
4. Implement following the task's "Suggested internal order".
5. Write the tests listed in the spec. Run `./mvnw clean verify`.
6. Verify every DoD checkbox for real, then tick it in `PROGRESS.md` (only what is truly verified).
7. Append an entry to `HANDOFF_LOG.md` (template inside).
8. Stop and report. Wait for the next task.

## Commit convention
`feat(P1-T2): append-only stock ledger + balances` — one task, one or a few commits; never mix tasks.

## Stop conditions (report instead of continuing)
- A prerequisite task is not actually done.
- A required decision from `99_REFERENCE/risks_open_questions.md` §9.1 (blocking decisions) is unanswered and the plan gives no default.
- Tests for tenant isolation fail.
