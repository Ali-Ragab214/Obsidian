# 05 — Package map & migration order

## Appendix A — Proposed package map (additive, no reorganization)

```
ali.ragab.saas_multi_tennant_app
├── agents/
│   ├── common/        AgentType, ApprovalHandler, ApprovalResolvedEvent,
│   │                  AgentRunContext, TenantJobRunner, LlmGateway
│   ├── forecast/      ForecastingEngine, SES/MovingAverage, DemandForecastAgent
│   ├── reorder/       ReorderAgent, VendorSelector, ReorderQuantityCalculator,
│   │                  ReorderApprovalHandler
│   ├── negotiation/   NegotiationStateMachine, NegotiationStrategyAgent,
│   │                  SimulatedVendorResponder, NegotiationApprovalHandler,
│   │                  NegotiationTranscriptIngestor
│   ├── anomaly/       AnomalyRule implementations, AnomalyDetectionService,
│   │                  AnomalyExplanationService, AnomalyJob, listener
│   └── rag/           KnowledgeIngestionService, SectionChunker, RetrievalService,
│                      RagChatService, VectorStoreRepository, QueryFilterExtractor
├── config/            TenantIdentifierResolver, TenantConnectionProvider,
│                      HibernateTenantConfig, AuditorConfig, AiConfig, OpenApiConfig,
│                      DemoDataSeeder
├── controllers/       Auth, Tenant, User, Product, Stock, Vendor, PurchaseOrder,
│                      Approval, AgentRun, Knowledge, RagChat, Forecast, Reorder,
│                      Negotiation, Anomaly, Dashboard, Demo
├── entities/          existing + Vendor, VendorProduct, PurchaseOrder, ApprovalRequest,
│                      AgentRun, AgentRunLog, DemandForecast, NegotiationCase,
│                      NegotiationMessage, KnowledgeDocument, KnowledgeChunk,
│                      AnomalyFlag, StockBalance, StockCount
├── mappers/           existing + one mapper per new aggregate
├── repositories/      existing + one repository per new entity
├── requests/          one request DTO per command
├── responses/         one response DTO per read model
├── services/          existing + new interfaces
├── services/impl/     existing + new implementations
├── events/            StockMovementPostedEvent, PurchaseOrderFinalizedEvent,
│                      NegotiationFinalizedEvent
└── security/          ManagerOnly, StaffOnly, PlatformAdminOnly, Roles
```

Existing packages and class names are kept. The `agents/` subtree is the only structural addition; it groups each agent’s prompt/state machine so the four responsibilities stay separable (spec E.5).

---

## Appendix B — Migration order

| Schema | Migration | Contents |
|---|---|---|
| `public` | `V2__create_platform_tables.sql` | `tenants` (add timezone/currency/provisioning_error), `users` with tenant-scoped uniqueness |
| `public` | `V3__platform_indexes.sql` | status/tenant indexes |
| `public` | `V4__enable_vector.sql` | `CREATE EXTENSION IF NOT EXISTS vector` (or an init script) |
| `tenant_*` | `V2__product_and_stock.sql` | product fields (UoM/active), `stock_mvts` additions, `stock_balances`, `stock_counts`, indexes, optional immutability trigger |
| `tenant_*` | `V3__vendors_and_purchase_orders.sql` | `vendors`, `vendor_products`, `vendor_negotiation_profiles`, `purchase_orders`, `purchase_order_lines`, sequence, indexes |
| `tenant_*` | `V4__agent_infrastructure.sql` | `agent_runs`, `agent_run_logs`, `approval_requests`, indexes |
| `tenant_*` | `V5__rag_knowledge.sql` | `knowledge_documents`, `knowledge_chunks` (`vector(768)`), `rag_queries`, HNSW index |
| `tenant_*` | `V6__demand_forecasts.sql` | `demand_forecasts` + indexes |
| `tenant_*` | `V7__negotiations.sql` | `negotiation_cases`, `negotiation_messages`, PO link |
| `tenant_*` | `V8__anomaly_flags.sql` | `anomaly_flags` + dedup/indexes |

**Flyway rules:** never edit an applied migration; the empty common `V1` may already be recorded, so add `V2`. Provisioning runs all tenant migrations for new tenants; `TenantMigrationService` applies new ones to existing tenants. Back up before running against a real tenant schema. The `vector` extension is database-wide, not schema-wide.
