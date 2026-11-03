
# Kammel (كمّل) — Project Overview

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Status:** Implementation-Ready Specification

---

## 1. Executive Summary

**Kammel** (كمّل — "Keep Going") is a **Multi-Tenant Gym Retention & Revenue Management SaaS** platform designed for the Egyptian market.

**The one-line pitch:**

> Kammel tells the gym owner who will leave next month, prepares the message to bring them back, and shows where money is going.

Kammel is **not** another Gym Management System. It is a **Retention & Collections Engine** built on top of a lightweight operational core. The core (members, subscriptions, payments, attendance) collects data. The value the owner pays for is: **more renewals, less revenue leakage, and fewer lost members**.

The strategy rests on four pillars:

1. **Start with a narrow wedge:** Renewal and collections management — not every module at once.
2. **Build on channels people already use:** WhatsApp, cash, InstaPay.
3. **Sell a result** ("We brought back X members") — not a system.
4. **AI comes last**, starting with explainable rules before any machine learning, because early-stage data is insufficient for reliable models.

---

## 2. Product Vision

Kammel transforms gym operational data into **actionable business decisions**.

| Traditional Gym System | Kammel |
|----------------------|--------|
| Shows member data: name, subscription end date | Analyzes patterns: "Ahmed absent 12 days, attendance dropped 60%, subscription expires in 5 days" |
| Requires manual follow-up | Prepares a ready-to-send WhatsApp message with personalized content |
| Reports static numbers | Recommends specific actions: "Contact today — high churn risk" |
| Data management | **Decision support and business intelligence** |

The platform is designed as:

- **Arabic-first** with full RTL support
- **WhatsApp-native** for communication (the primary channel Egyptian gym staff and members use)
- **Local-payment-aware** — cash, InstaPay, bank transfers — not built around credit card auto-renewal
- **Multi-tenant** — serving independent gyms and small chains from a single deployment

---

## 3. Problem Statement

### What Happens in a Typical Egyptian Gym

| Problem | Impact |
|---------|--------|
| Members leave silently — attendance drops gradually, then they don't renew | Revenue loss discovered too late to act |
| Renewals and collections handled manually by reception via phone or WhatsApp | Inconsistent follow-up, no accountability |
| Cash passes through staff without daily reconciliation | Undetected cash leakage |
| Leads from Instagram and WhatsApp get lost in chat threads | Wasted marketing spend |
| PT session packages partially consumed without tracking | Revenue not fully realized |
| Data lives in notebooks, Excel, or legacy systems without reporting | No business visibility |

### Why the Gap Exists

- Global gym management tools are not Arabic-first and don't integrate with local payment methods or WhatsApp as a primary channel.
- Local alternatives (if any) lack retention intelligence — they are CRUD systems without actionable insights.
- "AI workout generators" are ChatGPT wrappers with no competitive moat. The real barrier to entry is **operational data** (attendance + payments + renewals) collected from the gym itself.

> [!NOTE]
> All claims about the Egyptian gym market, competitor landscape, and gym owner behavior are **hypotheses** that must be validated through interviews before building. See [01_BUSINESS_REQUIREMENTS.md](./01_BUSINESS_REQUIREMENTS.md) for validation requirements.

---

## 4. Target Users

### Primary Personas

| Persona | Role | Primary Need | Success Metric |
|---------|------|-------------|----------------|
| **Gym Owner** (Buyer) | Decision-maker, revenue oversight | Know who will leave, where money goes, per-branch performance | Higher revenue, lower leakage |
| **Branch Manager / Receptionist** (Daily User) | Frontline operations | Fast, simple screens that don't slow down during rush hours | Operational efficiency |
| **Trainer** (Secondary User) | Client management | See assigned clients, track compliance | Client retention |
| **Member** (End User) | Gym client | Know subscription status, easy entry, feel valued | Convenience |

### Key Insight

The **buyer** (gym owner) and the **daily user** (receptionist) are different people with different needs. The system must sell to the owner (dashboards, revenue insights) but be usable by the receptionist (speed, simplicity). If the receptionist finds it difficult, the system will be abandoned.

---

## 5. Target Market

- **Independent gyms** and **small gym chains** (2-5 branches) in Egypt
- Gyms with **100–500+ active members**
- Currently using paper notebooks, Excel spreadsheets, or outdated management software
- Located in urban and semi-urban areas with WhatsApp penetration

> [!IMPORTANT]
> Market size, competitor pricing, and gym owner willingness to pay are **assumptions requiring validation** through the 10-interview process described in [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md).

---

## 6. Core Value Proposition

### The Break-Even Argument

A gym with 300 members, average subscription 700 EGP, monthly churn 8%:

- **24 members** leave every month → **16,800 EGP** in lost monthly revenue
- If Kammel reduces churn from 8% to 6% → **6 additional members** stay → **4,200 EGP** retained
- Kammel subscription ~1,000 EGP/month → saving **just 2 members** (1,400 EGP) covers the cost

This is the core sales argument. It must be validated with real gym data during the pilot.

### Three Pillars of Value

1. **Retention Intelligence** — Identify at-risk members before they leave, with explainable reasons and ready-to-send messages
2. **Revenue Leakage Detection** — Track suspicious freezes, unauthorized discounts, modified payments, cash variances
3. **Operational Efficiency** — Automate renewal reminders, streamline check-in, generate receipts, reconcile cash shifts

---

## 7. Main Business Objectives

1. **Reduce member churn** through proactive risk detection and targeted outreach
2. **Eliminate revenue leakage** through payment tracking, audit logging, and cash reconciliation
3. **Automate renewal follow-ups** through WhatsApp message queue workflows
4. **Provide real-time business visibility** through the owner dashboard with actionable KPIs
5. **Enable data-driven decisions** by replacing guesswork with measurable metrics
6. **Support multi-branch operations** with branch-level access control and consolidated reporting
7. **Prove measurable ROI** — the system must demonstrate its value in EGP, not just features

---

## 8. Functional Scope

| Module | P0 (MVP) | P1 (Post-MVP) | P2 (Advanced) |
|--------|----------|---------------|---------------|
| Members & Plans | CRUD + Excel Import + basic plans | Coupons, discounts | Family/corporate memberships |
| Subscriptions | Full lifecycle + renewal + freeze | Smart expiration alerts | Auto-renewal |
| Payments | Cash/transfer recording + receipts + shift close | Paymob payment links | Installments |
| Attendance | Name/phone search + static QR | Dynamic QR (server-verified) | Hardware integration |
| Roles & Permissions | 4 roles + audit log | Custom permissions | SSO |
| Owner Dashboard | Renewals + collections + risk list | Trainer performance, peak hours | Branch comparison |
| WhatsApp | Message queue with wa.me links | Official Cloud API | Scheduled campaigns |
| Leads CRM | — | Simple funnel | Instagram integration |
| Member App | — | PWA (subscription, QR, streak) | Challenges, badges |
| AI | Rule-based Risk Score | Business assistant (Spring AI) | Trainer Copilot, Nutrition |

For detailed feature definitions, see [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md).

---

## 9. Non-Functional Requirements

| Requirement | Target | Notes |
|------------|--------|-------|
| **Multi-Tenancy** | Complete data isolation between gyms | org_id + PostgreSQL RLS + application filtering |
| **Financial Integrity** | Zero tolerance for calculation errors | Amounts in minor units (piasters), never float/double |
| **Auditability** | Every sensitive operation logged immutably | Who, when, before/after, reason |
| **Check-in Response** | < 1 second | Critical for reception during rush hours |
| **Availability** | Standard SaaS uptime (99.5%+) | Single VPS initially, monitored |
| **Arabic RTL** | Full right-to-left UI support | Arabic-first design |
| **Mobile Responsive** | Usable on tablets and phones | Reception often uses tablet |
| **Security** | HTTPS, encrypted backups, hashed passwords | bcrypt/Argon2, JWT, rate limiting |
| **Data Privacy** | Egyptian Data Protection Law #151/2020 compliance | Legal review required before launch |
| **Backup** | Daily encrypted backup to external storage | With tested recovery procedure |
| **Concurrent Users** | Support peak-hour usage per gym (5-15 simultaneous) | Not a high-scale challenge initially |

---

## 10. Business Assumptions Requiring Validation

| # | Assumption | Validation Method |
|---|-----------|-------------------|
| 1 | Gym owners in Egypt consider member churn a top problem | Gym owner interviews |
| 2 | Gym owners are willing to pay 600–2,500 EGP/month for a solution | Pricing interviews + pilot |
| 3 | Receptionists will adopt a digital system if it's simple enough | Pilot observation |
| 4 | WhatsApp renewal reminders measurably increase renewals | Holdout group comparison |
| 5 | Cash reconciliation is a strong enough pain point to justify the feature | Gym owner interviews |
| 6 | Most Egyptian gyms use paper/Excel, not existing software | Market research |
| 7 | The first 60 days of membership are the highest churn risk period | Analysis of pilot gym data |
| 8 | Gym owners trust a SaaS with their financial and member data | Interview + security demo |
| 9 | A single developer can build and support the MVP in 10-12 weeks | Realistic effort tracking |
| 10 | Arabic-first UX is a meaningful differentiator vs. global tools | Competitive analysis |

---

## 11. Product Success Metrics

### North Star Metric

> **Value of renewals attributed to system interventions (in EGP)**
>
> A message was sent → member renewed within 7 days → renewal value counted.

This directly ties Kammel's output to the owner's revenue.

### Product Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| Check-in registration rate | % of daily check-ins recorded through the system | ≥ 70% during pilot |
| Time to first value | From Excel import to first renewal reminder sent | < 1 day |
| Message-to-renewal rate | % of sent renewal reminders that result in renewal within 7 days | Track, optimize |
| Daily active usage | Gym staff using the system daily without reverting to paper | 14 consecutive days |

### Business Metrics

| Metric | Definition | Target (6 months) |
|--------|-----------|-------------------|
| Paying gyms | Number of gyms on paid subscriptions | 5–10 |
| MRR | Monthly recurring revenue | Track |
| Gym churn rate | % of gyms that stop paying per month | < 10% |

### KPI Definitions (Fixed from Day 1)

| KPI | Formula |
|-----|---------|
| Renewal Rate | Members who renewed ÷ members whose subscription ended in the period |
| Monthly Churn | Members who didn't renew after grace period ÷ active members at start of month |
| ARPM (Avg Revenue Per Member) | Subscription revenue ÷ active members |
| Weekly visits per member | Total check-ins ÷ active members ÷ weeks |
| Average member lifetime | Average months from first subscription to final churn |

### Pilot Success Criteria

Written and agreed with the pilot gym **before** the pilot starts:

1. Daily system usage for **14 consecutive days** without reverting to paper/Excel
2. **≥ 70%** of check-ins recorded through the system
3. Measurable renewal attribution: a clear number of renewals linked to system-generated reminders

---

## 12. Out-of-Scope Features

The following are explicitly **not** part of Kammel's scope in the foreseeable future:

| Feature | Reason |
|---------|--------|
| Full ERP / accounting / tax management | Kammel is not an accounting system |
| Hardware (turnstiles, access gates) | Adds hardware dependency and support complexity |
| Payment aggregation (holding customer funds) | Regulatory complexity; Kammel records payments, doesn't process them |
| Gym marketplace / member discovery | Different business model entirely |
| Native mobile app (iOS/Android stores) | PWA is sufficient; avoids store review delays |
| AI in the initial MVP | Insufficient data; rule-based scoring first |
| Trainer Copilot / Nutrition AI | Deferred until core system validated and generating revenue |
| Workout programming / exercise database | Not core to retention & revenue management |
| Medical / health data management | Legal liability and complexity |

---

## 13. Document Navigation

This document is part of the **Kammel Project Documentation** set. See the [README.md](./README.md) for the complete index.

### Key Related Documents

| Document | Purpose |
|----------|---------|
| [01_BUSINESS_REQUIREMENTS.md](./01_BUSINESS_REQUIREMENTS.md) | Detailed business workflows and rules |
| [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md) | MVP scope, phases, and timeline |
| [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) | Technical architecture and design |
| [13_IMPLEMENTATION_ROADMAP.md](./13_IMPLEMENTATION_ROADMAP.md) | Task-by-task implementation plan |
