# Kammel — MVP Definition & Product Roadmap

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [00_PROJECT_OVERVIEW.md](./00_PROJECT_OVERVIEW.md) · [13_IMPLEMENTATION_ROADMAP.md](./13_IMPLEMENTATION_ROADMAP.md)

---

## 1. MVP Philosophy

The MVP must prove one thing: **Kammel can reliably manage a gym's most important daily operations and demonstrate measurable value.**

### Guiding Principles

1. **Small enough to build** — a single developer in 10–12 weeks.
2. **Complete enough to validate** — a real gym can use it daily for 14 days without reverting to paper.
3. **Valuable enough to justify** — the owner can see which members are at risk and how much revenue is at stake.
4. **No premature optimization** — no Redis, no AI/ML, no complex integrations until the core works.

### Success Definition

> A real gym uses Kammel daily for 14 consecutive days without reverting to their notebook or Excel sheet, and the owner can point to at least one renewal that was influenced by a system-generated reminder.

### Source Document Reconciliation

| Topic | Doc0 (Proposal) | Doc1 (Evaluation) | This Document's Resolution |
|-------|-----------------|--------------------|-----------------------------|
| MVP timeline | 8 weeks | "Too ambitious for one developer" | **12 weeks** (more realistic, includes testing & pilot prep) |
| Cash reconciliation | P0 | "Depends on interviews — may be #1 pain point" | **Default P1**, promote to P0 if interviews confirm it as primary pain |
| Risk Score | P0 (rules only) | P0 ("product identity") | **P0** — essential to Kammel's differentiation |
| Redis | "Later if needed" | "Without it initially" | **Not in MVP** — add when performance data justifies |
| Dynamic QR | P0 for static, P1 for dynamic | P1 for dynamic | **P0 for static QR only**, dynamic QR in P1 |

---

## 2. P0 — Essential MVP

### 2.1 Authentication & User Management

| Field | Detail |
|-------|--------|
| **Description** | JWT-based login, password hashing, user provisioning by owner |
| **Business Value** | Security foundation; every operation requires authenticated identity |
| **Why P0** | Nothing works without auth |
| **Dependencies** | None (first thing to build) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: User can log in with phone/email + password. AC2: JWT access token (short-lived) + refresh token issued. AC3: Passwords hashed with bcrypt. AC4: Invalid credentials return 401. AC5: Rate limiting on login (5 per minute per IP). |

### 2.2 Multi-Tenancy Foundation

| Field | Detail |
|-------|--------|
| **Description** | Organization/Branch entities, org_id on all tables, PostgreSQL RLS, tenant context from JWT |
| **Business Value** | Enables serving multiple gyms; data isolation prevents leakage |
| **Why P0** | Architectural foundation — retrofitting multi-tenancy is extremely costly |
| **Dependencies** | Authentication (2.1) |
| **Complexity** | High |
| **Acceptance Criteria** | AC1: Every table has org_id column. AC2: RLS policies active on all tables. AC3: Automated integration test: Gym A cannot access Gym B data via ANY endpoint. AC4: Tenant context set per request from JWT, cleared after request. |

### 2.3 Member Management

| Field | Detail |
|-------|--------|
| **Description** | Register members, search by name/phone, view member profile, soft delete |
| **Business Value** | Core data — all other features depend on member records |
| **Why P0** | Foundation for subscriptions, payments, attendance |
| **Dependencies** | Multi-tenancy (2.2) |
| **Complexity** | Low–Medium |
| **Acceptance Criteria** | AC1: Register member with name, phone, gender, source, opt-in. AC2: Search by phone (partial match). AC3: Unique (org_id, phone) enforced. AC4: Soft delete (deleted_at, not hard delete). |

### 2.4 Excel Import

| Field | Detail |
|-------|--------|
| **Description** | Upload Excel, map columns, validate, preview errors, import members + subscriptions |
| **Business Value** | Critical for onboarding — a gym with 400 members won't enter them one by one |
| **Why P0** | Without import, adoption is blocked. This is the "10-minute wow demo" feature. |
| **Dependencies** | Member Management (2.3), Plans (2.5) |
| **Complexity** | Medium–High |
| **Acceptance Criteria** | AC1: Upload .xlsx file. AC2: Map columns to fields. AC3: Validation: duplicate phones, invalid dates, missing required fields. AC4: Preview with error highlighting. AC5: Import with transactional safety (rollback on failure). AC6: Import existing subscriptions with correct end dates. |

### 2.5 Membership Plans

| Field | Detail |
|-------|--------|
| **Description** | Create/edit plans with price, duration, freeze rules, grace period, plan types |
| **Business Value** | Template for all subscriptions |
| **Why P0** | Subscriptions can't exist without plans |
| **Dependencies** | Multi-tenancy (2.2) |
| **Complexity** | Low |
| **Acceptance Criteria** | AC1: Create plan with type, duration, price, freeze max days, grace days. AC2: Price stored in minor units. AC3: Deactivate plan (existing subscriptions unaffected). AC4: Plan types: MONTHLY, QUARTERLY, SEMI_ANNUAL, ANNUAL, SESSION_PACK, DAY_PASS. |

### 2.6 Subscription Management

| Field | Detail |
|-------|--------|
| **Description** | Full lifecycle: create → activate → freeze → grace → expire → renew, with event tracking |
| **Business Value** | The core business object — everything revolves around subscriptions |
| **Why P0** | Subscriptions are the revenue unit |
| **Dependencies** | Members (2.3), Plans (2.5), Payments (2.7) |
| **Complexity** | **High** — state machine, events, renewal logic, freeze rules |
| **Acceptance Criteria** | AC1: Create subscription → PENDING_PAYMENT. AC2: Payment activates subscription. AC3: Renewal before expiry extends from end_date. AC4: Renewal after expiry starts from payment date. AC5: Freeze with reason, extends end_date, respects max days. AC6: Grace period transitions. AC7: Every state change creates SubscriptionEvent. AC8: Price snapshot at creation. AC9: Invalid transitions rejected with clear error. |

### 2.7 Payment Recording

| Field | Detail |
|-------|--------|
| **Description** | Record cash/InstaPay/transfer payments, partial payments, PDF receipt, idempotency |
| **Business Value** | Revenue recording and tracking |
| **Why P0** | No revenue visibility without payment records |
| **Dependencies** | Subscriptions (2.6) |
| **Complexity** | Medium–High |
| **Acceptance Criteria** | AC1: Record payment with method, amount, reference. AC2: Idempotency key prevents duplicates. AC3: Partial payments tracked (remaining balance visible). AC4: Full payment activates subscription. AC5: Amounts in minor units (piasters). AC6: Receipt with sequential number generated. |

### 2.8 Basic Attendance

| Field | Detail |
|-------|--------|
| **Description** | Check-in via name/phone search + static QR. Subscription status validation. |
| **Business Value** | Attendance data feeds Risk Score; daily operational necessity |
| **Why P0** | Foundation for retention analytics |
| **Dependencies** | Members (2.3), Subscriptions (2.6) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Search member by name/phone, record check-in. AC2: Static QR scan triggers check-in. AC3: Expired subscription shows alert + "Renew Now" (not rejection). AC4: Frozen subscription blocks with reason. AC5: Duplicate same-day: warning (not hard block). AC6: Override with reason logged in audit. AC7: Response time < 1 second. |

### 2.9 Role-Based Access Control

| Field | Detail |
|-------|--------|
| **Description** | Four roles (Owner, Manager, Receptionist, Trainer) with @PreAuthorize method-level security |
| **Business Value** | Prevents unauthorized access to financial data and cross-branch data |
| **Why P0** | Security and compliance requirement |
| **Dependencies** | Authentication (2.1) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Owner sees all branches. AC2: Manager restricted to assigned branch. AC3: Receptionist can register, check-in, collect payments — cannot access reports or modify payments. AC4: Trainer sees only assigned clients, no financial data. |

### 2.10 Audit Logging

| Field | Detail |
|-------|--------|
| **Description** | Immutable log of sensitive operations: who, when, action, entity, before/after, reason |
| **Business Value** | Accountability, fraud detection, regulatory compliance |
| **Why P0** | Doc0 explicitly requires audit for all sensitive operations |
| **Dependencies** | Authentication (2.1), Multi-tenancy (2.2) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Discounts, freezes, refunds, payment mods, overrides, role changes logged. AC2: Log includes actor, timestamp, before/after JSONB, reason. AC3: Log is append-only (no update/delete). AC4: Owner can view audit trail per entity. |

### 2.11 Renewal Dashboard

| Field | Detail |
|-------|--------|
| **Description** | Owner dashboard showing: renewals this week, at-risk members, collection list, revenue summary |
| **Business Value** | The "selling" feature — this is what makes the owner see value |
| **Why P0** | Without dashboard, owner has no reason to use Kammel |
| **Dependencies** | Subscriptions (2.6), Payments (2.7), Risk Score (2.13) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Renewals this week: count, renewed, pending, EGP value. AC2: At-risk list: members sorted by risk and value. AC3: Collections: outstanding partial payments. AC4: KPIs: renewal rate, churn rate, ARPM. |

### 2.12 WhatsApp Message Queue

| Field | Detail |
|-------|--------|
| **Description** | System generates pre-filled renewal/miss-you messages; receptionist sends via wa.me links |
| **Business Value** | Automates the most impactful intervention — renewal reminders |
| **Why P0** | Core to Kammel's value proposition (retention through outreach) |
| **Dependencies** | Members (2.3), Subscriptions (2.6), Risk Score (2.13) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Daily job generates pending messages for expiring subscriptions. AC2: Templates personalized with member name, gym name, dates. AC3: Receptionist sees queue, clicks "Open in WhatsApp". AC4: Receptionist confirms sending → status SENT_CONFIRMED. AC5: Duplicate prevention (one per template per member per 3 days). AC6: Attribution: if member renews within 7 days, linked to message. |

### 2.13 Basic Risk Score

| Field | Detail |
|-------|--------|
| **Description** | Rule-based risk scoring: weighted signals → score 0–100 + explanation + recommended action |
| **Business Value** | This IS Kammel's identity — transforming data into actionable insights |
| **Why P0** | Differentiates Kammel from every gym CRUD system |
| **Dependencies** | Attendance (2.8), Subscriptions (2.6) |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: Nightly batch computation. AC2: Score 0–100 with level (LOW/MEDIUM/HIGH/CRITICAL). AC3: Reasons array (explainable). AC4: Recommended action text. AC5: Draft message in Arabic. AC6: New members with < 14 days data flagged as "insufficient data". |

### 2.14 CI/CD Pipeline & Deployment

| Field | Detail |
|-------|--------|
| **Description** | Docker Compose, GitHub Actions, automated testing, deployment to VPS |
| **Business Value** | Reliable deployment, prevents regression |
| **Why P0** | Must deploy to validate with real users |
| **Dependencies** | All features |
| **Complexity** | Medium |
| **Acceptance Criteria** | AC1: docker-compose up starts app + PostgreSQL. AC2: GitHub Actions runs tests on every push. AC3: Tenant isolation tests must pass in CI. AC4: Single-command deployment to VPS. |

---

## 3. P1 — Post-MVP Features

| # | Feature | Business Value | Dependencies | Complexity |
|---|---------|---------------|-------------|------------|
| 1 | Cash Shift Management & Reconciliation | Detect cash leakage per shift and employee | Payments | Medium |
| 2 | Refund Processing | Handle corrections properly with reverse entries | Payments, Audit | Medium |
| 3 | Dynamic QR Check-in | Prevent card sharing (server-verified, 30-sec rotation) | Attendance | Medium |
| 4 | WhatsApp Cloud API | Automated sending via official BSP (no manual step) | Message Queue | High |
| 5 | Leads CRM | Track prospects: new → contacted → trial → subscribed/lost | Members | Medium |
| 6 | Member PWA | White-label app: subscription status, QR, attendance streak | Members, Auth | High |
| 7 | Paymob Payment Links | Online payment option via payment links | Payments | Medium |
| 8 | Revenue Reports | Trainer performance, peak hours, branch comparison | Dashboard | Medium |
| 9 | Coupons & Discounts | Structured discount management (vs. ad-hoc) | Plans, Subscriptions | Low |
| 10 | Smart Expiration Alerts | Multi-channel, multi-step reminder sequences | Notifications | Medium |

---

## 4. P2 — Advanced Features

| # | Feature | Notes |
|---|---------|-------|
| 1 | Auto-renewal | Requires reliable payment provider integration |
| 2 | Installment payments | Split subscription cost across multiple scheduled payments |
| 3 | Custom permissions | Beyond 4 fixed roles — configurable per organization |
| 4 | Branch comparison analytics | Side-by-side performance metrics |
| 5 | Hardware integration (turnstiles) | Requires hardware partnerships |
| 6 | Referral program | Member-refer-member with tracking |
| 7 | Scheduled WhatsApp campaigns | Bulk messaging on schedule |
| 8 | Instagram lead integration | Auto-import leads from Instagram DMs |
| 9 | SSO | Single sign-on for enterprise clients |
| 10 | Family/corporate memberships | Complex subscription structures |

---

## 5. P3 — Future AI-Powered Capabilities

| # | Feature | Prerequisites | Approach |
|---|---------|---------------|----------|
| 1 | ML-based Churn Prediction | 1000+ members, multiple renewal cycles, labeled outcomes | Logistic Regression → ONNX in Java |
| 2 | Arabic Business Assistant | Working dashboard APIs, tenant-scoped tools | Spring AI + Function Calling |
| 3 | Trainer Copilot | Training log data, exercise database | Spring AI |
| 4 | Egyptian Nutrition AI | Curated food database reviewed by nutritionist | Spring AI + RAG |

> [!IMPORTANT]
> AI features must NOT be implemented until: (1) core system is production-stable, (2) at least 3–6 months of operational data exists, (3) rule-based Risk Score has been validated against actual churn outcomes. See [12_AI_INTEGRATION_ROADMAP.md](./12_AI_INTEGRATION_ROADMAP.md).

---

## 6. Explicit MVP Exclusions

The following are **NOT** in the MVP:

| Excluded Feature | Reason |
|-----------------|--------|
| Cash shift reconciliation | Moved to P1 unless interviews confirm it as #1 pain point |
| Refunds | Rarely needed in first 14 days of pilot |
| Paymob integration | Adds third-party complexity |
| WhatsApp Cloud API | Cost, BSP setup, template approval delays |
| Leads CRM | Not core to retention wedge |
| Member PWA | Members are passive in MVP |
| Any AI/ML | Insufficient data |
| Redis caching | Premature optimization |
| Dynamic QR | Static QR sufficient for MVP |
| Auto-renewal | Requires payment provider integration |
| Nutrition / Workout AI | Not core, high liability |

---

## 7. Implementation Timeline

### Phase 0: Validation (Weeks −2 to 0) — Before Code

| Task | Output |
|------|--------|
| Conduct 10 gym owner interviews | Interview notes, pain point ranking |
| Identify top 3 problems (churn? cash? renewals?) | Validated or adjusted wedge |
| Select 2–3 pilot gyms | Signed pilot agreement |
| Get sample Excel member data (with consent) | Test import data |
| Agree on written pilot success metrics | Measurable criteria |

**Gate:** If < 3 of 10 gym owners show real interest → reconsider the wedge before writing code.

### Phase 1: Foundation (Weeks 1–3)

| Week | Deliverables |
|------|-------------|
| 1 | Project setup, Docker Compose, CI pipeline, PostgreSQL + Flyway, project structure |
| 2 | Identity module (users, auth, JWT), Organization/Branch entities |
| 3 | Multi-tenancy (org_id, RLS policies, tenant context), tenant isolation test |

**Checkpoint 1:** Auth works, tenant isolation verified, app deploys to VPS.

### Phase 2: Core Domain (Weeks 4–6)

| Week | Deliverables |
|------|-------------|
| 4 | Member CRUD, phone search, Excel import (upload, mapping, validation) |
| 5 | Plans CRUD, Subscription lifecycle (state machine, events, price snapshot) |
| 6 | Payment recording (cash/transfer, idempotency, partial payments, receipt) |

**Checkpoint 2:** Can register members, create subscriptions, record payments end-to-end.

### Phase 3: Operations & Intelligence (Weeks 7–9)

| Week | Deliverables |
|------|-------------|
| 7 | Attendance (search + static QR), roles & @PreAuthorize, audit logging |
| 8 | Rule-based Risk Score (batch computation, explanations), at-risk member list |
| 9 | Renewal dashboard, WhatsApp message queue (templates, wa.me links, attribution) |

**Checkpoint 3:** Full daily workflow — check-in, view renewals, see risk list, send reminders.

### Phase 4: Polish & Pilot (Weeks 10–12)

| Week | Deliverables |
|------|-------------|
| 10 | Integration tests (especially tenant isolation, payment idempotency, state transitions) |
| 11 | Bug fixes, edge cases, backup setup, monitoring (Actuator + Sentry) |
| 12 | Reception training, real gym data import, pilot launch |

**Checkpoint 4:** Pilot gym operating on Kammel daily. 14-day criterion begins.

### Post-MVP Phases

| Phase | Timeline | Focus |
|-------|----------|-------|
| Phase 5 | Month 4 | Cash reconciliation, refunds, advanced audit |
| Phase 6 | Month 5–6 | WhatsApp Cloud API, Leads CRM, Member PWA, Paymob |
| Phase 7 | Month 7–9 | ML risk model evaluation, Spring AI business assistant |
| Phase 8 | Month 10+ | Advanced analytics, Trainer Copilot (if justified) |

---

## 8. Milestone Checkpoints

| Checkpoint | Week | Criteria | Status |
|-----------|------|----------|--------|
| CP1: Foundation | 3 | Auth + tenancy + RLS + deploy | Runnable |
| CP2: Core Domain | 6 | Members + subscriptions + payments | Testable |
| CP3: Full Workflow | 9 | Check-in + dashboard + risk + WhatsApp | Demo-ready |
| CP4: Pilot-Ready | 12 | Real data, trained staff, monitoring | Pilot launch |
| CP5: Pilot Complete | 14 | 14 days daily use, 70% check-ins, attributed renewals | Validated |

---

## 9. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|-----------|
| MVP takes longer than 12 weeks | High | Medium | Strict scope control, no feature creep |
| Frontend (React) slows down backend dev | Medium | Medium | Use admin template; consider Thymeleaf for MVP if needed |
| Pilot gym doesn't adopt system | High | Medium | Champion on reception staff, simple UX, daily support |
| Cash reconciliation needed in MVP after all | Low | Medium | Architecture supports easy promotion from P1 |
| Single developer burnout | High | Medium | Realistic timeline, clear milestones, weekly progress review |
| Technical debt from speed | Medium | High | Plan refactoring time in post-MVP phases |

---

## 10. Decision Log

| # | Decision | Rationale | Alternatives Considered |
|---|----------|-----------|------------------------|
| D1 | 12-week MVP instead of 8 | Doc1 correctly identifies 8 weeks as too optimistic for one developer | 8 weeks (too tight), 16 weeks (too slow for validation) |
| D2 | Cash reconciliation default P1 | Reduces MVP scope; can be promoted to P0 based on interviews | Always P0 (too much), always P2 (too late if it's the main pain) |
| D3 | Risk Score in P0 | This is Kammel's identity — without it, Kammel is just another gym CRUD | P1 (loses differentiation) |
| D4 | No Redis in MVP | Premature optimization for hundreds of members | Include from start (unnecessary complexity) |
| D5 | Static QR only in MVP | Dynamic QR adds complexity; static QR sufficient to prove value | Both in MVP (scope creep) |
| D6 | wa.me links, not WhatsApp API | Avoids BSP cost, template approvals, API complexity in MVP | Official API from start (too expensive/complex) |
