# Kammel — Business Requirements

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [00_PROJECT_OVERVIEW.md](./00_PROJECT_OVERVIEW.md) · [02_MVP_AND_PRODUCT_ROADMAP.md](./02_MVP_AND_PRODUCT_ROADMAP.md)

---

## 1. Gym Owner Workflows

### 1.1 Onboarding & Initial Setup

| Field | Detail |
|-------|--------|
| **Actor** | Gym Owner (assisted by Platform Admin for organization creation) |
| **Preconditions** | Owner has agreed to use Kammel (pilot or paid subscription) |
| **Main Flow** | 1. Platform Admin creates Organization record → provisions first Owner account. 2. Owner logs in, creates Branch(es). 3. Owner creates membership Plans (monthly, quarterly, etc.) with pricing, duration, freeze rules, grace period. 4. Owner creates staff accounts (Manager, Receptionist, Trainer) and assigns them to branches. 5. Owner uploads existing member data from Excel (column mapping → validation → preview → import). 6. Owner reviews imported subscriptions and active members. 7. System is ready for daily operations. |
| **Alternative Flows** | A1: Gym has no existing data → skip import, start fresh. A2: Import has errors → system shows error report, owner corrects and re-imports. |
| **Business Rules** | BR-001: Each organization must have at least one branch. BR-002: Plans must have a name, price (> 0), and duration. BR-003: Excel import must detect duplicate phone numbers within the same organization. |
| **Expected Outcome** | Organization fully configured, staff accounts active, member data loaded. |
| **Failure Scenarios** | F1: Excel file format unsupported → clear error message. F2: Duplicate phone numbers in import → highlighted for resolution. |

### 1.2 Daily Dashboard Review

| Field | Detail |
|-------|--------|
| **Actor** | Gym Owner |
| **Preconditions** | Organization has active members and operational data |
| **Main Flow** | 1. Owner opens dashboard. 2. Reviews renewal summary: subscriptions expiring this week, renewed count, pending count, total value in EGP. 3. Reviews at-risk list: members with high risk scores, sorted by risk level and subscription value. 4. Reviews collections: outstanding partial payments and overdue amounts. 5. Reviews cash variance: expected vs. actual per shift and employee. 6. Takes action: clicks "Send Reminder" or drills into member details. |
| **Business Rules** | BR-004: Dashboard data is organization-scoped (owner sees all branches). BR-005: At-risk list shows risk score, reason, suggested action, and draft message. |
| **Expected Outcome** | Owner has a clear picture of business health and actionable next steps. |

### 1.3 Revenue Leakage Detection

| Field | Detail |
|-------|--------|
| **Actor** | Gym Owner |
| **Preconditions** | System has payment and subscription history |
| **Main Flow** | 1. Owner reviews "Revenue Leakage" dashboard card. 2. System flags: frequent/suspicious freezes, large unauthorized discounts, modified or reversed payments, PT sessions paid but unused, expired members still checking in. 3. Owner investigates flagged items via audit log (who, when, before/after, reason). 4. Owner takes corrective action (policy change, staff discussion). |
| **Business Rules** | BR-006: Every discount, freeze, refund, or payment modification is audited. BR-007: Audit log is immutable — no edits or deletions. |

---

## 2. Branch Manager Workflows

### 2.1 Daily Branch Operations

| Field | Detail |
|-------|--------|
| **Actor** | Branch Manager |
| **Preconditions** | Manager is assigned to a specific branch |
| **Main Flow** | 1. Reviews branch-level dashboard (renewals, check-ins, collections for their branch only). 2. Approves or processes discount requests from reception. 3. Approves freeze requests (validates reason and remaining freeze days). 4. Processes refunds (validates reason, creates reverse entry). 5. Monitors shift closings and cash variances for branch staff. |
| **Business Rules** | BR-008: Manager can only view and operate on their assigned branch. BR-009: Manager cannot access other branches' data even within the same organization. BR-010: Refunds require mandatory reason and create reverse entries (no payment deletion). |
| **Failure Scenarios** | F1: Manager attempts to access another branch → denied with appropriate error. |

---

## 3. Receptionist Workflows

### 3.1 Member Registration

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | Receptionist logged in, assigned to a branch |
| **Main Flow** | 1. New person arrives at gym. 2. Receptionist opens "Register Member" screen. 3. Enters: full name, phone number (required), gender, birth date, source (Instagram, friend, walk-in, ad, referral), notes, WhatsApp opt-in consent. 4. System validates: phone format, duplicate phone within organization. 5. Member record created with unique ID and member code. 6. Optionally: create subscription immediately. |
| **Alternative Flows** | A1: Phone already exists → show existing member, suggest subscription renewal. A2: Bulk import → see Excel Import workflow. |
| **Business Rules** | BR-011: Phone number is the primary search key. BR-012: (org_id, phone) must be unique — no duplicate phones per gym. BR-013: WhatsApp opt-in must be explicitly captured at registration. |
| **Validation** | Phone: required, valid Egyptian mobile format. Name: required, 2–100 characters. Gender: enum (MALE, FEMALE). Source: enum. |

### 3.2 Member Check-In

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | Member exists in system |
| **Main Flow** | 1. Member arrives at reception. 2. Receptionist searches by name, phone, or member code (P0), OR scans member's static QR card/phone (P0). 3. System finds member and checks subscription status. 4. **ACTIVE**: Record check-in (timestamp, branch, method). Display "Welcome" confirmation. 5. **EXPIRING_SOON** (≤7 days): Record check-in + display alert "Subscription expires in X days" with "Renew Now" button. 6. If session-based plan: auto-deduct one session, display remaining. |
| **Alternative Flows** | A1: **EXPIRED** → Alert receptionist with "Subscription expired" + "Renew Now" button. NOT a hard block or embarrassing rejection — handle gracefully. A2: **FROZEN** → Block entry with clear message "Subscription frozen until {date}. Reason: {reason}." A3: **GRACE** → Allow entry + warning "Grace period, expires in X days." A4: **No subscription** → Alert receptionist, suggest registration. A5: **Duplicate check-in same day** → Warning "Already checked in at {time} today." Receptionist can override with reason. A6: **Session-based, 0 remaining** → Alert "All sessions used. Renew to continue." |
| **Business Rules** | BR-014: Check-in must never hard-block a member in a way that causes public embarrassment. BR-015: Every check-in records: timestamp, branch, method, subscription reference. BR-016: Overrides require a reason and are logged in audit. BR-017: Check-in data is the foundation for Risk Score calculation. BR-018: Duplicate same-day check-in is warned, not blocked (reception can override). |

### 3.3 Payment Collection

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | Member has an existing subscription (or creating one simultaneously) |
| **Main Flow** | 1. Receptionist selects member and subscription. 2. System shows: plan price, amount paid so far, remaining balance. 3. Receptionist enters: amount received, payment method (Cash / InstaPay / Bank Transfer), reference number (for non-cash). 4. Client provides Idempotency Key (or system generates one). 5. System records payment, links to current open cash shift (if cash). 6. If total payments ≥ plan price → subscription activated/renewed. 7. If partial → subscription remains pending, remaining balance tracked. 8. Receipt generated (PDF with sequential number). 9. Receipt can be sent via WhatsApp (wa.me link). |
| **Business Rules** | BR-019: Amounts stored as integers in minor units (piasters). Never float/double. BR-020: Idempotency key prevents duplicate payment recording. BR-021: Payments are NEVER deleted. Corrections via refund/reversal only. BR-022: Partial payments are supported. System tracks remaining balance. BR-023: Receipt numbers are sequential per organization. BR-024: Cash payments must be linked to an open shift. |

### 3.4 Subscription Creation & Renewal

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | Member exists, plans are configured |
| **Main Flow** | 1. Receptionist selects member. 2. Selects plan from active plans list. 3. System snapshots the current plan price (price changes won't affect this subscription). 4. Creates subscription with status PENDING_PAYMENT, start_date (today or specified). 5. Calculates end_date = start_date + plan.duration_days. 6. Receptionist collects payment (see Payment Collection workflow). 7. On sufficient payment → subscription transitions to ACTIVE. |
| **Renewal Flow** | R1: **Before expiry**: new end_date = current end_date + plan duration. Member doesn't lose remaining days. R2: **After expiry**: new start_date = payment date (or manager-specified date). end_date = start_date + duration. R3: **During grace**: same as after expiry. |
| **Business Rules** | BR-025: Price is snapshotted at subscription creation — plan price changes don't affect existing subscriptions. BR-026: Renewal before expiry extends from end_date, NOT from payment date. BR-027: Renewal after expiry starts from payment date (manager can override with reason). BR-028: Every state change creates a SubscriptionEvent record. |

### 3.5 Cash Shift Management

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | User is assigned to a branch |
| **Main Flow** | 1. Start of work: Receptionist opens a new shift. 2. During shift: all cash payments linked to this shift. 3. End of work: Receptionist closes shift. 4. System calculates expected cash = sum of cash payments − sum of cash refunds in this shift. 5. Receptionist counts actual cash in drawer, enters amount. 6. System calculates variance = counted − expected. 7. If variance ≠ 0: flagged for owner/manager review. 8. Shift record sealed. |
| **Business Rules** | BR-029: Only one open shift per user at a time. BR-030: Previous shift must be closed before opening a new one. BR-031: Variance reported to owner dashboard. BR-032: Shift records are immutable after closing. |

### 3.6 WhatsApp Message Sending

| Field | Detail |
|-------|--------|
| **Actor** | Receptionist |
| **Preconditions** | Message queue has pending messages |
| **Main Flow** | 1. Receptionist opens notification queue. 2. Sees list of pending messages (renewal reminders, miss-you, collection reminders). 3. For each message: reviews content, clicks "Open in WhatsApp". 4. System opens wa.me link with pre-filled message. 5. Receptionist sends message in WhatsApp. 6. Returns to system and confirms "Sent". 7. Message status updates to SENT_CONFIRMED. |
| **Business Rules** | BR-033: Opening wa.me link does NOT equal message sent. Receptionist must confirm. BR-034: One message per template per member per 3 days (duplicate prevention). BR-035: Messages only sent during reasonable hours (9 AM – 9 PM). BR-036: Only to members with whatsapp_opt_in = true. |

---

## 4. Trainer Workflows

### 4.1 Client Management

| Field | Detail |
|-------|--------|
| **Actor** | Trainer |
| **Preconditions** | Trainer assigned to branch, has assigned clients |
| **Main Flow** | 1. Trainer views list of assigned clients. 2. For each client: sees subscription status, last check-in date, attendance pattern. 3. Identifies non-compliant clients (not attending regularly). 4. Can view client's attendance history but NOT financial data. |
| **Business Rules** | BR-037: Trainer sees only their assigned clients. BR-038: Trainer cannot access payment, financial, or other trainers' data. |

---

## 5. Member Workflows

### 5.1 Passive Experience (MVP)

In the MVP, members are mostly passive — they receive WhatsApp messages and check in via reception. Member self-service comes in P1 (PWA).

### 5.2 Member Self-Service (P1 — PWA)

| Feature | Description |
|---------|-------------|
| Login | Phone number + WhatsApp Magic Link (cheaper than SMS OTP) |
| Subscription Status | View current subscription, end date, remaining sessions |
| QR Code | Display personal QR for check-in |
| Attendance History | View check-in streak and calendar |
| Renewal | Request renewal (payment still at reception or via Paymob link) |

---

## 6. Subscription Lifecycle

See [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) for the complete state machine. Summary:

```
PENDING_PAYMENT → ACTIVE → EXPIRING_SOON (computed) → GRACE → EXPIRED
                    ↕
                  FROZEN
                    ↓
                 CANCELLED
```

### Key Rules

1. **State is derived from events**, not from manual status field updates.
2. **Price snapshot** preserved at subscription creation.
3. **Freeze** has a maximum days limit per plan, mandatory reason, and auto-extends end_date.
4. **Grace period** defined per plan (e.g., 3–7 days after expiry).
5. **EXPIRING_SOON** is computed (≤ 7 days remaining), not stored.
6. **Session-based plans** track sessions_remaining, decremented on check-in.

---

## 7. Payment Lifecycle

See [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) for full detail.

### Payment Methods (MVP)

| Method | Details |
|--------|---------|
| Cash | Linked to active cash shift, reconciled at shift close |
| InstaPay / Bank Transfer | Reference number recorded, not in cash count |
| Paymob Link (P1) | Online payment via payment link, webhook for confirmation |

### Key Rules

- **Partial payments** supported — system tracks remaining balance
- **No payment deletion** — corrections via refund (reverse entry)
- **Idempotency** — duplicate prevention via unique key
- **Amounts in piasters** — stored as BIGINT, never float/double
- **Receipt** — PDF with sequential number, sendable via WhatsApp

---

## 8. Attendance Workflows

Three levels of check-in, each works independently:

| Level | Method | Priority | Phase |
|-------|--------|----------|-------|
| 1 | Name/phone/code search from reception screen | Primary — never fails | P0 |
| 2 | Static QR on card or member's phone | Fast, camera-based | P0 |
| 3 | Dynamic QR (server-verified, 30-second rotation) | Prevents card sharing | P1 |

### Guiding Principle

> **The receptionist must never be blocked.** If QR doesn't work, search works. If search is slow, basic lookup works. Check-in must be under 1 second.

---

## 9. Renewal Workflows

1. **Daily scheduled job** identifies subscriptions expiring within 7 days and grace period subscriptions.
2. **Message generation**: pre-filled WhatsApp messages using templates (personalized with name, gym, date).
3. **Queue display**: Receptionist sees pending messages, opens wa.me link, sends, confirms.
4. **Attribution**: If member renews within 7 days of a sent message, system links the renewal to the message.
5. **Dashboard**: Owner sees renewal count, renewal rate, pending renewals, total value in EGP.

---

## 10. Cash Management Workflows

See section 3.5 for shift management details and [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) for reconciliation logic.

### Owner's Cash Visibility

| Metric | Source |
|--------|--------|
| Expected cash per shift | Sum of cash payments in shift |
| Actual cash per shift | Receptionist's count at shift close |
| Variance | Actual − Expected |
| Daily totals per employee | Aggregated across shifts |
| Trends | Variance patterns over time |

---

## 11. Retention Workflows

### Risk Score Computation

Computed nightly as a batch job. Signals:

| Signal | Weight | Description |
|--------|--------|-------------|
| Days since last attendance | ~30% | 0 days = no risk, 14+ days = high risk |
| Attendance frequency change | ~25% | Decrease in visits over last 14 days vs. prior 14 days |
| Remaining subscription days | ~20% | Combined with low attendance = critical risk |
| Payment delay history | ~10% | Count of past late payments |
| Freeze history | ~10% | Number of freezes (behavioral indicator) |
| Membership age | ~5% | First 60 days = highest risk period (assumption) |

### Output per Member

- Score: 0–100
- Level: LOW / MEDIUM / HIGH / CRITICAL
- Reasons: human-readable explanation (e.g., "Absent 12 days, attendance dropped 60%")
- Recommended action (e.g., "Contact today")
- Draft WhatsApp message

### Measurement

- **Holdout testing**: Group A receives reminders, Group B doesn't (with owner consent)
- Compare renewal rates between groups
- This proves (or disproves) Kammel's impact with real data

---

## 12. Business Rules Summary

| Rule ID | Category | Description | Source | Status |
|---------|----------|-------------|--------|--------|
| BR-001 | Setup | Each organization must have at least one branch | Doc0 §12.2 | Confirmed |
| BR-002 | Plans | Plans must have name, price > 0, and duration | Doc0 §6.1 | Confirmed |
| BR-003 | Import | Excel import must detect duplicate phones per org | Doc0 §6.1 | Confirmed |
| BR-004 | Dashboard | Dashboard is org-scoped; owner sees all branches | Doc0 §7 | Confirmed |
| BR-005 | Risk | Risk list shows score, reason, action, draft message | Doc0 §11.1 | Confirmed |
| BR-006 | Audit | Every sensitive operation (discount, freeze, refund, payment mod) is audited | Doc0 §6.5 | Confirmed |
| BR-007 | Audit | Audit log is immutable — no edits or deletions | Doc0 §6.5 | Confirmed |
| BR-008 | Access | Manager operates on their assigned branch only | Doc0 §6.5 | Confirmed |
| BR-009 | Access | Manager cannot access other branches within same org | Doc1 §6 | Confirmed |
| BR-010 | Payments | Refunds create reverse entries; no payment deletion | Doc0 §6.3 | Confirmed |
| BR-011 | Members | Phone number is primary search key | Doc0 §6.1 | Confirmed |
| BR-012 | Members | (org_id, phone) must be unique | Doc0 §13 | Confirmed |
| BR-013 | Privacy | WhatsApp opt-in must be explicit at registration | Doc0 §8 | Confirmed |
| BR-014 | Check-in | Never embarrass a member at check-in | Doc0 §6.4 | Confirmed |
| BR-015 | Check-in | Every check-in records timestamp, branch, method | Doc0 §6.4 | Confirmed |
| BR-016 | Check-in | Overrides require reason + audit | Doc0 §6.4 | Confirmed |
| BR-017 | Risk | Check-in data is foundation for Risk Score | Doc0 §6.4 | Confirmed |
| BR-018 | Check-in | Duplicate same-day check-in warned, not blocked | Doc0 §6.4 | Confirmed |
| BR-019 | Financial | Amounts in minor units (piasters), never float/double | Doc0 §6.3 | Confirmed |
| BR-020 | Financial | Idempotency key prevents duplicate payments | Doc0 §6.3 | Confirmed |
| BR-021 | Financial | Payments never deleted | Doc0 §6.3 | Confirmed |
| BR-022 | Financial | Partial payments supported | Doc0 §6.3 | Confirmed |
| BR-023 | Financial | Receipt numbers sequential per organization | Doc0 §6.3 | Confirmed |
| BR-024 | Financial | Cash payments linked to open shift | Doc0 §6.3 | Confirmed |
| BR-025 | Subscription | Price snapshotted at creation | Doc0 §6.1 | Confirmed |
| BR-026 | Subscription | Renewal before expiry extends from end_date | Doc0 §6.2 | Confirmed |
| BR-027 | Subscription | Renewal after expiry starts from payment date | Doc0 §6.2 | Confirmed |
| BR-028 | Subscription | Every state change creates SubscriptionEvent | Doc0 §6.2 | Confirmed |
| BR-029 | Shift | One open shift per user at a time | Inferred | Assumed |
| BR-030 | Shift | Must close previous shift before opening new one | Inferred | Assumed |
| BR-031 | Shift | Variance reported to owner dashboard | Doc0 §7 | Confirmed |
| BR-032 | Shift | Shift records immutable after closing | Inferred | Assumed |
| BR-033 | Notifications | Opening wa.me ≠ message sent; manual confirmation required | Doc1 §5.4 | Confirmed |
| BR-034 | Notifications | Max one message per template per member per 3 days | Doc0 §8 | Assumed |
| BR-035 | Notifications | Messages during reasonable hours only (9 AM – 9 PM) | Doc0 §8 | Assumed |
| BR-036 | Privacy | WhatsApp messages only to opted-in members | Doc0 §8 | Confirmed |
| BR-037 | Access | Trainer sees only assigned clients | Doc0 §6.5 | Confirmed |
| BR-038 | Access | Trainer has no access to financial data | Doc0 §6.5 | Confirmed |

> [!IMPORTANT]
> Rules marked **Assumed** are reasonable inferences but require validation with gym owners before implementation. Rules marked **Confirmed** are directly stated in the source documents.

---

## 13. Open Questions for Product Owner

1. **Cash reconciliation priority**: Is cash leakage or member churn the bigger pain point? This determines if cash shifts belong in MVP or P1.
2. **Time-restricted plans**: Do Egyptian gyms commonly offer morning/evening plans, or separate gender sessions? Should the MVP support time restrictions?
3. **Discount limits**: What discount percentage should a receptionist be allowed without manager approval?
4. **Freeze frequency**: Can a member freeze multiple times per subscription, or once only?
5. **Multi-subscription**: Can a member have multiple active subscriptions simultaneously (e.g., gym + PT)?
6. **WhatsApp message frequency**: Is 3-day minimum between messages the right interval?
7. **Hosting location**: Inside or outside Egypt? Affects data privacy law compliance and latency.
8. **WhatsApp cost model**: Pass-through to gym owner or included in Kammel subscription?
