# StockSavvy — Agent Task Pack (START HERE)

## بالعربي (للمستخدم)
1. حط الفولدر ده جوه المشروع (مثلاً في `docs/agent-tasks/`) أو جنبه.
2. افتح الـ agent وادّيه **تاسك واحد بس** في المرة: افتح ملف التاسك من فولدر `TASKS/`، انسخ البلوك اللي اسمه **"Prompt for the agent"**، وأرفق معاه الفولدر كله.
3. خلّي الـ agent يخلص ويكتب في `HANDOFF_LOG.md` ويعلّم في `PROGRESS.md`. راجع الشغل وشغّل الـ tests بنفسك.
4. بعد ما تتأكد إن الـ DoD كله اتحقق، روح للتاسك اللي بعده بالترتيب. **متعديش تاسك قبل ما اللي قبله يخلص.**
5. الـ `99_REFERENCE/IMPLEMENTATION_PLAN_ORIGINAL.md` هو الملف الأصلي كامل ومتعدلش فيه حاجة. كل تاسك فيه الـ spec بتاعه منسوخ حرفياً، فمفيش حاجة بتتنسى.

## For the agent
You are working on a Spring Boot 4.1.2 / Java 21 schema-per-tenant project. This pack splits the full implementation plan into ordered, self-contained tasks. Always read `00_CONTEXT/00_agent_rules.md` first.

## Folder map
```
stocksavvy-agent-tasks/
├── README.md                      <- you are here
├── PROGRESS.md                    <- master tracker: every task, every DoD item, every conflict fix
├── HANDOFF_LOG.md                 <- what each finished task changed (read before starting the next)
├── 00_CONTEXT/                    <- read before EVERY task
│   ├── 00_agent_rules.md
│   ├── 01_project_snapshot.md     (scan results + shared conventions)
│   ├── 02_conflicts_and_fixes.md  (all 24 issues)
│   ├── 03_gap_matrix.md
│   ├── 04_roadmap.md
│   └── 05_package_map_and_migrations.md
├── TASKS/                         <- one file per task, in execution order
└── 99_REFERENCE/                  <- dependency graph, demo checklist, risks/open questions, test matrix, original plan
```

## Execution order (do not reorder)
| # | Task | Phase | Effort | File |
|---|---|---|---|---|
| 1 | **P0-T1** Platform bootstrap & tenant isolation completion | 0 | **L** — 7–9 days. | `TASKS/P0-T1_platform_bootstrap_and_tenant_isolation.md` |
| 2 | **P0-T2** Personas, authentication & RBAC | 0 | **M** — 2–3 days (mostly wiring tests). | `TASKS/P0-T2_personas_auth_rbac.md` |
| 3 | **P1-T1** SKU management | 1 | **M** — 2–3 days. | `TASKS/P1-T1_sku_management.md` |
| 4 | **P1-T2** Stock ledger, balances & stock operations | 1 | **L** — 6–8 days. | `TASKS/P1-T2_stock_ledger_balances_operations.md` |
| 5 | **P1-T3** Low-stock dashboard | 1 | **S** — 1–2 days. | `TASKS/P1-T3_low_stock_dashboard.md` |
| 6 | **P2-T1** Vendors & pricing | 2 | **M** — 3–4 days. | `TASKS/P2-T1_vendors_and_pricing.md` |
| 7 | **P2-T2** Purchase orders | 2 | **M** — 3–4 days. | `TASKS/P2-T2_purchase_orders.md` |
| 8 | **P3-T1** Approval queue & agent audit trail | 3 | **M** — 3–4 days. | `TASKS/P3-T1_approval_queue_and_audit_trail.md` |
| 9 | **P4-T1** RAG foundation: ingestion, chunking, embeddings, pgvector | 4 | **L** — 6–8 days. | `TASKS/P4-T1_rag_foundation_ingestion.md` |
| 10 | **P4-T2** RAG retrieval, grounding & chat assistant | 4 | **M** — 4–5 days. | `TASKS/P4-T2_rag_retrieval_and_chat.md` |
| 11 | **P5-T1** Demand Forecasting Agent | 5 | **M** — 3–4 days. | `TASKS/P5-T1_demand_forecasting_agent.md` |
| 12 | **P6-T1** Reorder / Restock Automation Agent | 6 | **M** — 4–5 days. | `TASKS/P6-T1_reorder_agent.md` |
| 13 | **P7-T1** Vendor Negotiation Agent (flagship) | 7 | **L** — 8–10 days (the flagship; do not compress this). | `TASKS/P7-T1_negotiation_agent.md` |
| 14 | **P8-T1** Anomaly / Fraud Detection Agent | 8 | **M** — 3–4 days. | `TASKS/P8-T1_anomaly_detection_agent.md` |
| 15 | **P9-T1** Dashboard, demo data & defense artifacts | 9 | **M** — 4–5 days. | `TASKS/P9-T1_dashboard_demo_data_defense_artifacts.md` |

**Parallelism:** P5-T1 (forecasting) and P8-T1 (anomaly) are independent of the negotiation chain and may run in parallel with P6/P7 if two people work. Everything else is sequential.

**Critical path:** P0 → P1 → P2 → P3 → P4 → P7 → P9.

## Before you start: blocking decisions
Open `99_REFERENCE/risks_open_questions.md` §9.1 and decide the blocking questions (AI provider/embedding model, UI scope, etc.) **before P4-T1**. Tasks use the plan's recommended defaults when you do not decide.

## Demo checkpoints
- After **P6-T1**: run the first half of the 8-step demo with stub LLM/embeddings.
- After **P7-T1**: run the full 8-step demo headlessly.
- **P9-T1**: capture the four defense artifacts.

## Why nothing is lost
Each `TASKS/*.md` contains its section of the plan **verbatim**, plus its slice of the gap matrix and conflict log. `PROGRESS.md` collects every DoD checkbox. The untouched original is in `99_REFERENCE/`.
