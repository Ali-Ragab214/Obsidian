# Dependency graph

## 7. Dependency Graph

### 7.1 Phase-level

```mermaid
flowchart TD
    P0["Phase 0<br/>Runnable foundation + tenant isolation"]
    P1["Phase 1<br/>SKU + stock ledger + balances"]
    P2["Phase 2<br/>Vendors + purchase orders"]
    P3["Phase 3<br/>Approval queue + agent audit trail"]
    P4["Phase 4<br/>RAG foundation"]
    P5["Phase 5<br/>Demand forecasting"]
    P6["Phase 6<br/>Reorder agent"]
    P7["Phase 7<br/>Negotiation agent"]
    P8["Phase 8<br/>Anomaly agent"]
    P9["Phase 9<br/>Dashboard + demo + artifacts"]

    P0 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> P4
    P1 --> P5
    P2 --> P6
    P3 --> P6
    P5 --> P6
    P4 --> P7
    P6 --> P7
    P1 --> P8
    P3 --> P8
    P5 --> P9
    P6 --> P9
    P7 --> P9
    P8 --> P9
    P4 --> P9
```

Critical path: `P0 → P1 → P2 → P3 → P4 → P7 → P9`. `P5` and `P8` are parallelizable with `P6/P7` after their prerequisites.

### 7.2 Feature-level

```mermaid
flowchart LR
    TENANT["Tenant routing + login"]
    SKU["SKU master"]
    LEDGER["Append-only ledger + balances"]
    LOW["Low-stock query"]
    VENDOR["Vendors + catalog"]
    PO["Purchase orders"]
    APPROVAL["Approvals + AgentRunLog"]
    RAG["RAG ingest + retrieve"]
    FC["Forecast"]
    REORDER["Reorder agent"]
    NEGO["Negotiation agent"]
    ANOM["Anomaly agent"]
    DASH["Dashboard + artifacts"]

    TENANT --> SKU --> LEDGER --> LOW
    SKU --> VENDOR --> PO
    PO --> APPROVAL
    LEDGER --> FC
    LEDGER --> ANOM
    APPROVAL --> ANOM
    LOW --> REORDER
    FC --> REORDER
    VENDOR --> REORDER
    PO --> REORDER
    APPROVAL --> REORDER
    RAG --> NEGO
    REORDER --> NEGO
    APPROVAL --> NEGO
    NEGO --> RAG
    LOW --> DASH
    FC --> DASH
    REORDER --> DASH
    NEGO --> DASH
    ANOM --> DASH
```

The `NEGO → RAG` edge is the transcript feedback loop required by demo step 6/7: finalizing a negotiation creates a new searchable source.
