# How to use the plan (original guidance)

## Appendix D — How to use this plan

- **Start at Phase 0.** Do not create vendor/PO/agent tables until schema routing and login are proven with the two-tenant isolation test. Every later table inherits the isolation mechanism.
- **Build vertical slices.** Each phase should end with a working endpoint + test, not a pile of entities. The DoD checklists are the gate.
- **Keep the four agents separate.** A shared `ApprovalRequest`/`AgentRunLog`/`LlmGateway` is the only cross-cutting code. Do not merge prompts or state machines.
- **Prefer deterministic code for numbers.** LLMs write explanations and messages; Java computes quantities, prices, balances, and thresholds. This is the core defensibility argument.
- **Demo early.** After Phase 6, run the first half of the 8-step workflow with stub LLM/embeddings. After Phase 7, run the full workflow. Do not wait for Phase 9 to discover integration gaps.

---

*Scan method: read-only review of `pom.xml`, all `src/main/java` packages, both Flyway migration folders, `application.yml`, `docker-compose.yml`, tests, git history/branches, and the existing README audit; `./mvnw -q -DskipTests compile` was run to confirm the current build state. No source file was modified. The only file written is this plan.*
