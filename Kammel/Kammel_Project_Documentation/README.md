# Kammel (كمّل) — Complete Technical Documentation & Blueprint

> **Product:** Kammel (كمّل) — Multi-Tenant Gym Retention & Revenue Management SaaS  
> **Target Market:** Egyptian Gyms & Fitness Centers  
> **Technology Stack:** Java 21+, Spring Boot 3, PostgreSQL 16 (RLS), Flyway, Redis (Post-MVP), Docker Compose  
> **Architecture:** Feature-Based Modular Monolith with Multi-Tenancy  
> **Documentation Version:** 1.0 (November 2026)

---

## 1. Documentation Purpose & Navigation

This documentation folder serves as the **single source of truth** for building, testing, deploying, and maintaining the **Kammel** SaaS platform. It reconciles the original product proposal (`0-doc.md`) and the comprehensive engineering evaluation (`1-doc2.md`) into an implementation-ready blueprint.

Every document in this suite is designed to be directly actionable for a senior software engineer without guessing domain rules or architecture decisions.

### Complete Documentation Index

| File | Title | Primary Responsibility |
|---|---|---|
| [**00_PROJECT_OVERVIEW.md**](./00_PROJECT_OVERVIEW.md) | **Project Overview** | Vision, problem statement, personas, value proposition, and KPIs. |
| [**01_BUSINESS_REQUIREMENTS.md**](./01_BUSINESS_REQUIREMENTS.md) | **Business Requirements** | Detailed actor workflows, lifecycle rules, preconditions, and failure paths. |
| [**02_MVP_AND_PRODUCT_ROADMAP.md**](./02_MVP_AND_PRODUCT_ROADMAP.md) | **MVP Scope & Roadmap** | Scoping, P0 vs P1 vs P2 prioritization, milestone checkpoints, and trade-offs. |
| [**03_SYSTEM_ARCHITECTURE.md**](./03_SYSTEM_ARCHITECTURE.md) | **System Architecture** | Modular Monolith design, feature packaging, request lifecycle, and Mermaid diagrams. |
| [**04_DOMAIN_MODEL.md**](./04_DOMAIN_MODEL.md) | **Domain Model & Entities** | Complete domain entities, attributes, relationships, invariants, and Mermaid ERD. |
| [**05_DATABASE_DESIGN.md**](./05_DATABASE_DESIGN.md) | **Database & Flyway Strategy** | PostgreSQL table definitions, indexing strategy, constraints, and Flyway migration plan. |
| [**06_SECURITY_AND_MULTI_TENANCY.md**](./06_SECURITY_AND_MULTI_TENANCY.md) | **Security & Multi-Tenancy** | Authentication, JWT lifecycle, RBAC, branch isolation, and PostgreSQL Row-Level Security. |
| [**07_API_DESIGN.md**](./07_API_DESIGN.md) | **REST API & DTO Contracts** | REST endpoint catalog, request/response DTO schemas, validation rules, and error envelopes. |
| [**08_BUSINESS_LOGIC_AND_STATE_MACHINES.md**](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) | **State Machines & Domain Logic** | Subscription state transitions, payment idempotency, cash reconciliation, and risk engine. |
| [**09_REDIS_CACHING.md**](./09_REDIS_CACHING.md) | **Redis Caching Strategy** | Caching boundaries, tenant-aware cache keys, TTL policy, and resilience. |
| [**10_TESTING_STRATEGY.md**](./10_TESTING_STRATEGY.md) | **Testing & Quality Assurance** | Testing pyramid, Testcontainers, automated multi-tenant isolation tests, and CI gates. |
| [**11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md**](./11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md) | **Jobs & WhatsApp Workflows** | ShedLock distributed jobs, expiration cron, WhatsApp templates, and attribution. |
| [**12_AI_INTEGRATION_ROADMAP.md**](./12_AI_INTEGRATION_ROADMAP.md) | **AI & Machine Learning Roadmap** | Post-MVP prerequisites, supervised churn models, and Spring AI assistant boundaries. |
| [**13_IMPLEMENTATION_ROADMAP.md**](./13_IMPLEMENTATION_ROADMAP.md) | **Implementation Tasks** | 12-week task breakdown, dependencies, acceptance criteria, and checkpoints. |
| [**14_ENGINEERING_STANDARDS.md**](./14_ENGINEERING_STANDARDS.md) | **Engineering Standards** | Java conventions, DI rules, Lombok policies, logging standards, and PR checklist. |

---

## 2. Recommended Reading Order

Depending on your role and focus, consult the documents in the following order:

### For Business Stakeholders & Product Managers:
1. [00_PROJECT_OVERVIEW.md](./00_PROJECT_OVERVIEW.md) $\to$ Executive vision & economic break-even formula.
2. [01_BUSINESS_REQUIREMENTS.md](./01_BUSINESS_REQUIREMENTS.md) $\to$ User workflows & business rules.
3. [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md) $\to$ What is built in MVP vs post-MVP.

### For Software Architects & Lead Engineers:
1. [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) $\to$ Modular monolith architecture & package boundaries.
2. [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md) $\to$ Multi-tenant isolation & PostgreSQL RLS defense layers.
3. [04_DOMAIN_MODEL.md](./04_DOMAIN_MODEL.md) & [05_DATABASE_DESIGN.md](./05_DATABASE_DESIGN.md) $\to$ Schema, relationships, and Flyway strategy.
4. [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) $\to$ Subscription state machine & financial ledger rules.

### For Daily Hands-On Developers:
1. [14_ENGINEERING_STANDARDS.md](./14_ENGINEERING_STANDARDS.md) $\to$ Coding rules, Lombok guidelines, and PR checklist.
2. [13_IMPLEMENTATION_ROADMAP.md](./13_IMPLEMENTATION_ROADMAP.md) $\to$ Pick up tasks in dependency order.
3. [07_API_DESIGN.md](./07_API_DESIGN.md) $\to$ DTO contracts and endpoint specifications.
4. [10_TESTING_STRATEGY.md](./10_TESTING_STRATEGY.md) $\to$ Unit and integration test expectations.

---

## 3. High-Level Architectural Summary

```
                      ┌──────────────────────────────────────────────┐
                      │             React Web Dashboard              │
                      │       (Admin UI / RTL Arabic Support)        │
                      └──────────────────────┬───────────────────────┘
                                             │ HTTPS / JWT
                                             ▼
                      ┌──────────────────────────────────────────────┐
                      │              Spring Boot 3 (API)             │
                      │               Java 21 Virtual Threads         │
                      ├──────────────────────────────────────────────┤
                      │  Security: JWT + RBAC + Tenant Context       │
                      │  Modularity: Feature-Based Package Design    │
                      ├──────────────────────────────────────────────┤
                      │  identity │ tenancy │ members │ plans        │
                      │  subscriptions │ payments │ checkin          │
                      │  notifications │ risk │ reporting │ audit    │
                      └──────────────┬───────────────────────────────┘
                                     │
                 ┌───────────────────┴───────────────────┐
                 ▼                                       ▼
    ┌─────────────────────────┐             ┌─────────────────────────┐
    │      PostgreSQL 16      │             │     Redis (Post-MVP)    │
    │  - Shared Schema        │             │  - Dashboard Aggregates │
    │  - Row-Level Security   │             │  - Rate Limiting        │
    │  - Flyway Versioning    │             │  - Token Blacklist      │
    └─────────────────────────┘             └─────────────────────────┘
```

---

## 4. MVP Implementation Phases Matrix

| Phase | Duration | Core Business Capability | Checkpoint Goal |
|---|:---:|---|---|
| **Phase 0** | Weeks −2 to 0 | Validation interviews & sample Excel data | 3 confirmed gym pilots |
| **Phase 1** | Weeks 1 to 3 | Multi-tenancy, JWT Auth, PostgreSQL RLS, Docker CI | **CP1:** Containerized app passing tenant isolation tests |
| **Phase 2** | Weeks 4 to 6 | Members CRUD, Excel import, Plans, Subscriptions, Payments | **CP2:** End-to-end member contract & payment recording |
| **Phase 3** | Weeks 7 to 9 | Attendance check-in, Risk Score engine, WhatsApp queue, Dashboard | **CP3:** Full daily operational flow operable by reception |
| **Phase 4** | Weeks 10 to 12 | VPS deployment, GPG backup automation, Staff training | **CP4:** Live 14-day continuous pilot in production gym |

---

## 5. Documents by Development Phase

| During This Development Phase... | Consult These Specific Documents |
|---|---|
| **Sprint 1–3 (Foundation & Security)** | [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md), [05_DATABASE_DESIGN.md](./05_DATABASE_DESIGN.md), [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md), [14_ENGINEERING_STANDARDS.md](./14_ENGINEERING_STANDARDS.md) |
| **Sprint 4–6 (Core Domain & Billing)** | [04_DOMAIN_MODEL.md](./04_DOMAIN_MODEL.md), [07_API_DESIGN.md](./07_API_DESIGN.md), [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md), [10_TESTING_STRATEGY.md](./10_TESTING_STRATEGY.md) |
| **Sprint 7–9 (Operations & Retention)** | [01_BUSINESS_REQUIREMENTS.md](./01_BUSINESS_REQUIREMENTS.md), [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md), [11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md](./11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md) |
| **Sprint 10–12 (Deployment & Pilot)** | [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md), [10_TESTING_STRATEGY.md](./10_TESTING_STRATEGY.md), [13_IMPLEMENTATION_ROADMAP.md](./13_IMPLEMENTATION_ROADMAP.md) |
| **Post-MVP & Scaling (Phases 5–7)** | [09_REDIS_CACHING.md](./09_REDIS_CACHING.md), [12_AI_INTEGRATION_ROADMAP.md](./12_AI_INTEGRATION_ROADMAP.md) |

---

## 6. Project Readiness Assessment

### 6.1 Documentation Sufficiency Verdict
**Status: READY FOR IMPLEMENTATION**

The technical blueprint provided across these 15 documents provides comprehensive, concrete, and unambiguous technical specifications. A software engineer can clone the repository, initialize the Spring Boot 3 structure, create Flyway migration `V1`, and systematically implement every feature without architectural ambiguity.

### 6.2 Key Identified Source Document Conflicts & Resolutions

| Conflict Area | Original Proposal (`0-doc.md`) | Evaluation Report (`1-doc2.md`) | Blueprint Resolution |
|---|---|---|---|
| **MVP Timeframe** | 8 Weeks | Unrealistic for single developer | **Extended to 12 Weeks** (includes hardening and real pilot). |
| **Cash Shift Reconciliation** | Mandatory in P0 | Questioned whether Churn or Cash is #1 | **Scoped as P1 by default**; promote to P0 if confirmed in Phase 0 interviews. |
| **Redis Caching** | Ambiguous | "Omit in MVP; introduce when needed" | **Excluded from P0**; fully specified in [09_REDIS_CACHING.md](./09_REDIS_CACHING.md) for P1. |
| **WhatsApp Integration** | Mentioned automation | Cautioned on Meta BSP cost & bans | **P0 uses manual `wa.me` links**; Cloud API deferred to P1. |
| **AI Role** | Ambiguous ML timing | Must establish deterministic rules first | **P0 is rule-based only**; AI strictly deferred to Phase 3. |

### 6.3 Critical Open Questions (To Validate Before Sprint 1 Coding)
1. **Pilot Gym Commitment:** Has at least one gym owner signed a pilot agreement permitting data import and staff usage?
2. **Reception Desk Hardware:** Does the pilot gym reception desk use a desktop PC, a laptop, or an Android/iOS tablet? (Dictates responsive UI priority).
3. **Gender Session Separation:** Does the pilot gym operate mixed hours or separate male/female operational windows? If separate, time-restricted plan validation must be prioritized.
4. **Hosting Jurisdiction:** Will the production VPS reside within an Egyptian local data center or European hosting (e.g., Hetzner/AWS)? Must be reviewed against Egyptian Personal Data Protection Law (Law No. 151/2020).
