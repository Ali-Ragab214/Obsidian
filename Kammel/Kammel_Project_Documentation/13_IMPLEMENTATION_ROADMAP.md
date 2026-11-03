# Kammel — Implementation Roadmap & Engineering Tasks

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Scope:** 12-Week Production MVP Plan (Single Developer)  
> **Related:** [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md) · [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) · [10_TESTING_STRATEGY.md](./10_TESTING_STRATEGY.md)

---

## 1. Roadmap Architecture & Execution Philosophy

This roadmap is organized strictly by **functional business capabilities and architectural dependency**, rather than arbitrary technical silos. Each phase culminates in an independently **runnable and testable checkpoint**.

```
Phase 0: Customer Validation & Onboarding Prep (Weeks -2 to 0)
Phase 1: Architectural Foundation & Multi-Tenancy (Weeks 1 to 3)  --> [Checkpoint 1: Deployed & Isolated]
Phase 2: Core Membership & Subscription Domain (Weeks 4 to 6)    --> [Checkpoint 2: Contracts & Billing]
Phase 3: Operations, Attendance & Retention (Weeks 7 to 9)       --> [Checkpoint 3: Full Daily Routine]
Phase 4: Hardening, Observability & Pilot Launch (Weeks 10 to 12) --> [Checkpoint 4: Live Gym Pilot]
```

---

## 2. Phase 0: Business Validation & Discovery (Pre-Code)

### Epic: VAL-001 — Market & Pilot Discovery
- **Goal:** Verify that Egyptian gym owners recognize member churn and cash leakage as urgent, payable problems before writing production code.

#### `TSK-VAL-001`: Conduct 10 Semi-Structured Gym Owner Interviews
- **Priority:** P0 (Gatekeeper) | **Complexity:** Medium | **Dependencies:** None
- **Module:** Product Discovery
- **Description:** Interview 10 independent gym owners across Cairo/Giza using the interview protocol from [00_PROJECT_OVERVIEW.md](./00_PROJECT_OVERVIEW.md).
- **Expected Output:** Documented interview notes matrix ranking top operational friction points.
- **Acceptance Criteria:** At least 3 out of 10 owners explicitly confirm willingness to pilot the software.

#### `TSK-VAL-002`: Obtain Anonymized Sample Excel Member Sheets
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-VAL-001`
- **Module:** Product Discovery / Excel Import
- **Description:** Collect real-world member spreadsheets from 2 prospective pilot gyms to analyze column headers, date anomalies, and phone formats.
- **Expected Output:** 2 real-world test fixture spreadsheets stored in test resources.
- **Acceptance Criteria:** Sample files contain at least 150+ member records with historical expiration dates.

---

## 3. Phase 1: Architectural Foundation & Multi-Tenancy (Weeks 1–3)

### Checkpoint 1 Criteria
> A containerized Spring Boot backend running with PostgreSQL 16 on Docker, executing Flyway migrations, issuing JWTs, enforcing Row-Level Security, and passing automated cross-tenant data isolation tests.

### Epic: FND-001 — Project Infrastructure & Scaffolding

#### `TSK-FND-001`: Initialize Spring Boot 3 / Java 21 Project
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** None | **Module:** `common`
- **Output:** Maven/Gradle project with dependencies (Web, Data JPA, Security, Flyway, Validation, Lombok, Actuator, Testcontainers).
- **Acceptance Criteria:** Application boots on port 8080 with Actuator `/actuator/health` reporting `UP`.

#### `TSK-FND-002`: Docker Compose & PostgreSQL 16 Setup
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-FND-001` | **Module:** DevOps
- **Output:** `docker-compose.yml` defining PostgreSQL 16 with healthchecks and persistent volumes.
- **Acceptance Criteria:** `docker compose up -d` boots a clean PostgreSQL instance accessible by Spring Boot.

#### `TSK-FND-003`: GitHub Actions CI Pipeline Setup
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-FND-001` | **Module:** DevOps
- **Output:** `.github/workflows/ci.yml` running `./gradlew check` with containerized Testcontainers tests.
- **Acceptance Criteria:** Automated PR verification blocks merges on test failures or checkstyle warnings.

---

### Epic: SEC-001 — Multi-Tenancy & Identity Engine

#### `TSK-SEC-001`: Database Migrations for Organizations, Branches & Users
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-FND-002` | **Module:** `identity`, `tenancy`
- **Output:** `V1__create_organizations_and_branches.sql` and `V2__create_users.sql`.
- **Acceptance Criteria:** Flyway cleanly applies DDL with foreign keys, checks, and unique indexes on `(org_id, phone)`.

#### `TSK-SEC-002`: JWT Authentication & Refresh Token Rotation
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-SEC-001` | **Module:** `identity`
- **Output:** `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, and `POST /api/v1/auth/logout`.
- **Acceptance Criteria:** Issues 15-minute access token containing `org_id`, `branch_id`, and `role`. Rotates refresh tokens securely.

#### `TSK-SEC-003`: PostgreSQL Row-Level Security (RLS) Implementation
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-SEC-001` | **Module:** `common/security`
- **Output:** `V14__enable_rls_policies.sql` and Spring transaction interceptor setting `SET LOCAL app.current_org`.
- **Acceptance Criteria:** Non-superuser database queries strictly reject rows from differing organizations.

#### `TSK-SEC-004`: Automated Cross-Tenant Isolation Integration Test
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-SEC-002`, `TSK-SEC-003` | **Module:** `testing`
- **Output:** Testcontainers integration test creating two organizations and asserting zero cross-tenant query leaks.
- **Acceptance Criteria:** 100% pass rate in CI; build fails immediately if leakage occurs.

---

## 4. Phase 2: Core Membership & Subscription Domain (Weeks 4–6)

### Checkpoint 2 Criteria
> Receptionists can register members, batch-import existing members from Excel spreadsheets, create membership plans, issue event-driven subscriptions, record cash/InstaPay payments with idempotency, and print sequential PDF receipts.

### Epic: MEM-001 — Member Management & Excel Ingestion

#### `TSK-MEM-001`: Member Persistence & CRUD APIs
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-SEC-003` | **Module:** `members`
- **Output:** Member entity, repository, service, and `POST /api/v1/members`, `GET /api/v1/members`.
- **Acceptance Criteria:** Soft deletion implemented; duplicate phone numbers within the same gym rejected with HTTP 409.

#### `TSK-MEM-002`: Excel Sheet Ingestion & Reconciliation Service
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-MEM-001` | **Module:** `members`
- **Output:** `POST /api/v1/members/import` parsing Apache POI Excel uploads with error preview.
- **Acceptance Criteria:** Parses 500 rows in < 3 seconds; returns row-by-row error report for invalid mobile numbers.

---

### Epic: SUB-001 — Subscription Lifecycle & State Machine

#### `TSK-SUB-001`: Plan Templates & Price Snapshotting
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-SEC-003` | **Module:** `plans`
- **Output:** `POST /api/v1/plans` supporting duration, sessions, grace period, and freeze rules.
- **Acceptance Criteria:** Plan price changes do not retroactively alter active subscriptions.

#### `TSK-SUB-002`: Subscription Event-Driven State Machine Engine
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-SUB-001`, `TSK-MEM-001` | **Module:** `subscriptions`
- **Output:** State transitions (`PENDING_PAYMENT`, `ACTIVE`, `FROZEN`, `GRACE`, `EXPIRED`) producing `SubscriptionEvent` entities.
- **Acceptance Criteria:** Renewal prior to expiration extends from `end_date` without losing days; freezing extends `end_date`.

---

### Epic: PAY-001 — Payment Recording & Financial Ledger

#### `TSK-PAY-001`: Idempotent Payment Recording & Sequential Receipts
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-SUB-002` | **Module:** `payments`
- **Output:** `POST /api/v1/payments` with `Idempotency-Key` header, minor currency calculation, and PDF receipt rendering.
- **Acceptance Criteria:** Duplicate requests return identical responses without duplicate debiting; full payment triggers subscription activation.

#### `TSK-PAY-002`: Partial Payments & Outstanding Collection Tracking
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-PAY-001` | **Module:** `payments`
- **Output:** `GET /api/v1/payments/collections` listing members with outstanding subscription balances.
- **Acceptance Criteria:** Subscription stays `PENDING_PAYMENT` until cumulative payments match or exceed the price snapshot.

---

## 5. Phase 3: Operations, Attendance & Retention (Weeks 7–9)

### Checkpoint 3 Criteria
> Sub-second member check-in operational via name search and static QR, nightly scheduled jobs calculating explainable churn risk scores, owner dashboard populated with renewal KPIs, and WhatsApp reminder queue operating via `wa.me` links.

### Epic: ATT-001 — Attendance & Reception Check-In

#### `TSK-ATT-001`: Multi-Mode Attendance Check-In Service
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-SUB-002` | **Module:** `checkin`
- **Output:** `POST /api/v1/checkins` supporting phone search and static QR card lookup.
- **Acceptance Criteria:** Response time < 500ms; friendly alert for expired members; frozen members blocked with reason.

#### `TSK-ATT-002`: Check-In Session Deduction & Duplicate Alerts
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-ATT-001` | **Module:** `checkin`
- **Output:** Session decrementing for session-based plans; warning alert for duplicate same-day visits.
- **Acceptance Criteria:** Decrements `sessions_remaining`; allows staff override with mandatory reason logged in audit.

---

### Epic: RET-001 — Retention Risk Engine & WhatsApp Queue

#### `TSK-RET-001`: Rule-Based Nightly Risk Scoring Engine
- **Priority:** P0 | **Complexity:** **HIGH** | **Dependencies:** `TSK-ATT-001`, `TSK-SUB-002` | **Module:** `risk`
- **Output:** Nightly scheduled job computing 0–100 risk score based on absence, attendance drop, and days remaining.
- **Acceptance Criteria:** Stored in `risk_scores` with human-readable Arabic explanation bullets; handles new members gracefully.

#### `TSK-RET-002`: WhatsApp Message Queue & `wa.me` Deep Link Dispatch
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-RET-001` | **Module:** `notifications`
- **Output:** `GET /api/v1/notifications/queue` and `PATCH /api/v1/notifications/{id}/sent`.
- **Acceptance Criteria:** Pre-renders Arabic message templates; enforces 3-day cooldown per member; opening `wa.me` does not mark as sent until confirmed.

#### `TSK-RET-003`: Renewal Attribution Tracker
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-RET-002`, `TSK-PAY-001` | **Module:** `notifications`
- **Output:** Event listener binding renewal payments made within 7 days of a confirmed WhatsApp outreach.
- **Acceptance Criteria:** Links `attributed_subscription_id` on message; populates owner ROI dashboard.

---

### Epic: REP-001 — Executive Dashboard & Audit Trail

#### `TSK-REP-001`: Owner Dashboard Aggregation Service
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-PAY-001`, `TSK-RET-001` | **Module:** `reporting`
- **Output:** `GET /api/v1/dashboard/summary` returning renewals due, at-risk list, monthly revenue, and renewal rate %.
- **Acceptance Criteria:** Executes in < 300ms using composite tenant indexes; owner sees consolidated numbers across all branches.

#### `TSK-REP-002`: Immutable Audit Log Interceptor
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-SEC-003` | **Module:** `audit`
- **Output:** Aspect-oriented or entity-listener interceptor recording discounts, freezes, refunds, and overrides.
- **Acceptance Criteria:** Captures actor, action, timestamp, and before/after JSONB state; records cannot be updated or deleted.

---

## 6. Phase 4: Hardening, Observability & Pilot Launch (Weeks 10–12)

### Checkpoint 4 Criteria
> Deployed on production VPS, automated daily encrypted database backups tested, receptionist trained, Excel data imported from pilot gym, and 14-day continuous daily operational pilot initiated.

### Epic: OPS-001 — Deployment, Backup & Production Hardening

#### `TSK-OPS-001`: Production VPS Setup & Nginx SSL Reverse Proxy
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-FND-002` | **Module:** DevOps
- **Output:** Ubuntu VPS running Docker, Nginx with Let's Encrypt SSL, UFW firewall.
- **Acceptance Criteria:** Enforces HTTPS; qualifies for A rating on SSL Labs; exposes only ports 80 and 443.

#### `TSK-OPS-002`: Automated Daily Encrypted PostgreSQL Backups
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-OPS-001` | **Module:** DevOps
- **Output:** Daily cron script executing `pg_dump`, encrypting with GPG, and streaming to external cloud storage (e.g., S3/Wasabi).
- **Acceptance Criteria:** Documented restore runbook tested successfully on a secondary staging environment.

#### `TSK-OPS-003`: Application Observability (Actuator + Sentry)
- **Priority:** P0 | **Complexity:** Low | **Dependencies:** `TSK-OPS-001` | **Module:** `common`
- **Output:** Real-time error alerting via Sentry and JVM performance metrics via Spring Boot Actuator.
- **Acceptance Criteria:** Unhandled 500 exceptions trigger real-time alerts without leaking customer PII in traces.

---

### Epic: PLT-001 — Pilot Deployment & Onboarding

#### `TSK-PLT-001`: Pilot Gym Data Migration & Onboarding Run
- **Priority:** P0 | **Complexity:** Medium | **Dependencies:** `TSK-MEM-002`, `TSK-OPS-001` | **Module:** Operations
- **Output:** Pilot gym member data imported from their real-world spreadsheet; active plans configured.
- **Acceptance Criteria:** 100% of current active members loaded with correct subscription expiration dates.

#### `TSK-PLT-002`: Reception Staff Training & 14-Day Pilot Execution
- **Priority:** P0 | **Complexity:** High | **Dependencies:** `TSK-PLT-001` | **Module:** Operations
- **Output:** 1-hour receptionist training; physical cheat sheet placed at desk; 14-day daily monitoring.
- **Acceptance Criteria:** Gym runs 14 consecutive days without reverting to paper; at least 70% of attendance captured via Kammel.
