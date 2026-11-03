# Kammel — Database Design & Flyway Strategy

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [04_DOMAIN_MODEL.md](./04_DOMAIN_MODEL.md) · [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. Design Principles

| Principle | Implementation |
|-----------|---------------|
| Source of truth | PostgreSQL is the single source of truth for all business data |
| Multi-tenancy | Shared schema — all tenants in same tables, isolated by `org_id` |
| Financial precision | BIGINT for all amounts (minor units / piasters). Never `double` or `float`. |
| Primary keys | UUID (`gen_random_uuid()`) |
| Timestamps | `TIMESTAMPTZ` (UTC), displayed in `Africa/Cairo` by application |
| Soft deletion | `deleted_at TIMESTAMPTZ` on members. No hard delete for financial records. |
| Immutability | `audit_log` and `subscription_events` — no UPDATE or DELETE |
| Consistent columns | `created_at`, `updated_at` on mutable entities |

---

## 2. Table Definitions

### 2.1 organizations

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| name | VARCHAR(255) | NOT NULL |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE', CHECK IN ('ACTIVE','SUSPENDED','CLOSED') |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.2 branches

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| name | VARCHAR(255) | NOT NULL |
| address | VARCHAR(500) | |
| timezone | VARCHAR(50) | NOT NULL, DEFAULT 'Africa/Cairo' |
| active | BOOLEAN | NOT NULL, DEFAULT true |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.3 users

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| branch_id | UUID | FK → branches(id), NULL for GYM_OWNER |
| name | VARCHAR(100) | NOT NULL |
| phone | VARCHAR(20) | NOT NULL |
| email | VARCHAR(255) | |
| password_hash | VARCHAR(255) | NOT NULL |
| role | VARCHAR(30) | NOT NULL |
| active | BOOLEAN | NOT NULL, DEFAULT true |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

**Unique:** `(org_id, phone)`, `(org_id, email) WHERE email IS NOT NULL`

### 2.4 members

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| branch_id | UUID | NOT NULL, FK → branches(id) |
| full_name | VARCHAR(100) | NOT NULL |
| phone | VARCHAR(20) | NOT NULL |
| gender | VARCHAR(10) | |
| birth_date | DATE | |
| photo_url | VARCHAR(500) | |
| source | VARCHAR(20) | |
| notes | VARCHAR(1000) | |
| whatsapp_opt_in | BOOLEAN | NOT NULL, DEFAULT false |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| deleted_at | TIMESTAMPTZ | |

**Unique:** `(org_id, phone) WHERE deleted_at IS NULL`

### 2.5 plans

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| name | VARCHAR(100) | NOT NULL |
| type | VARCHAR(30) | NOT NULL |
| duration_days | INTEGER | NOT NULL, CHECK > 0 |
| session_count | INTEGER | |
| price_minor | BIGINT | NOT NULL, CHECK >= 0 |
| freeze_max_days | INTEGER | NOT NULL, DEFAULT 0 |
| grace_days | INTEGER | NOT NULL, DEFAULT 0 |
| time_restriction | VARCHAR(20) | |
| active | BOOLEAN | NOT NULL, DEFAULT true |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.6 subscriptions

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| member_id | UUID | NOT NULL, FK → members(id) |
| plan_id | UUID | NOT NULL, FK → plans(id) |
| price_minor_snapshot | BIGINT | NOT NULL, CHECK >= 0 |
| start_date | DATE | NOT NULL |
| end_date | DATE | NOT NULL, CHECK (end_date >= start_date) |
| status | VARCHAR(20) | NOT NULL |
| sessions_remaining | INTEGER | |
| frozen_days_used | INTEGER | NOT NULL, DEFAULT 0 |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.7 subscription_events (Immutable)

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| subscription_id | UUID | NOT NULL, FK → subscriptions(id) |
| type | VARCHAR(30) | NOT NULL |
| from_status | VARCHAR(20) | |
| to_status | VARCHAR(20) | NOT NULL |
| actor_id | UUID | FK → users(id) |
| reason | VARCHAR(500) | |
| metadata | JSONB | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.8 payments (Never Deleted)

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| subscription_id | UUID | NOT NULL, FK → subscriptions(id) |
| member_id | UUID | NOT NULL, FK → members(id) |
| amount_minor | BIGINT | NOT NULL, CHECK > 0 |
| method | VARCHAR(20) | NOT NULL |
| reference | VARCHAR(100) | |
| received_by | UUID | NOT NULL, FK → users(id) |
| shift_id | UUID | FK → cash_shifts(id) |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'COMPLETED' |
| idempotency_key | VARCHAR(100) | NOT NULL, UNIQUE |
| notes | VARCHAR(500) | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.9 refunds

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| payment_id | UUID | NOT NULL, FK → payments(id) |
| amount_minor | BIGINT | NOT NULL, CHECK > 0 |
| reason | VARCHAR(500) | NOT NULL |
| approved_by | UUID | NOT NULL, FK → users(id) |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.10 cash_shifts

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| branch_id | UUID | NOT NULL, FK → branches(id) |
| user_id | UUID | NOT NULL, FK → users(id) |
| opened_at | TIMESTAMPTZ | NOT NULL |
| closed_at | TIMESTAMPTZ | |
| expected_cash_minor | BIGINT | |
| counted_cash_minor | BIGINT | |
| notes | VARCHAR(500) | |

### 2.11 check_ins

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| branch_id | UUID | NOT NULL, FK → branches(id) |
| member_id | UUID | NOT NULL, FK → members(id) |
| subscription_id | UUID | FK → subscriptions(id) |
| method | VARCHAR(20) | NOT NULL |
| checked_in_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |
| overridden_by | UUID | FK → users(id) |
| override_reason | VARCHAR(500) | |

### 2.12 message_queue

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| member_id | UUID | NOT NULL, FK → members(id) |
| template | VARCHAR(30) | NOT NULL |
| rendered_text | TEXT | NOT NULL |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'PENDING' |
| due_date | DATE | NOT NULL |
| wa_me_link | VARCHAR(500) | |
| sent_at | TIMESTAMPTZ | |
| sent_by | UUID | FK → users(id) |
| attributed_subscription_id | UUID | FK → subscriptions(id) |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.13 risk_scores

| Column | Type | Constraints |
|--------|------|-------------|
| member_id | UUID | NOT NULL, FK → members(id) |
| computed_at | TIMESTAMPTZ | NOT NULL |
| score | INTEGER | NOT NULL, CHECK BETWEEN 0 AND 100 |
| reasons | JSONB | NOT NULL |
| signals | JSONB | |
| model_version | VARCHAR(30) | NOT NULL |
| recommended_action | VARCHAR(500) | |
| draft_message | TEXT | |

**PK:** `(member_id, computed_at)`

### 2.14 audit_log (Immutable)

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, DEFAULT gen_random_uuid() |
| org_id | UUID | NOT NULL, FK → organizations(id) |
| actor_id | UUID | NOT NULL, FK → users(id) |
| action | VARCHAR(50) | NOT NULL |
| entity_type | VARCHAR(50) | NOT NULL |
| entity_id | UUID | NOT NULL |
| before_state | JSONB | |
| after_state | JSONB | |
| reason | VARCHAR(500) | |
| ip_address | VARCHAR(45) | |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

### 2.15 shedlock

| Column | Type | Constraints |
|--------|------|-------------|
| name | VARCHAR(64) | PK |
| lock_until | TIMESTAMPTZ | NOT NULL |
| locked_at | TIMESTAMPTZ | NOT NULL |
| locked_by | VARCHAR(255) | NOT NULL |

---

## 3. Indexing Strategy

```sql
-- Member phone lookup (check-in, registration)
CREATE UNIQUE INDEX idx_members_org_phone ON members(org_id, phone) WHERE deleted_at IS NULL;

-- Subscription expiry (dashboard, scheduler)
CREATE INDEX idx_subs_org_status_end ON subscriptions(org_id, status, end_date) WHERE status IN ('ACTIVE','GRACE');

-- Check-in history (risk score)
CREATE INDEX idx_checkins_org_member ON check_ins(org_id, member_id, checked_in_at DESC);

-- Payments per shift (reconciliation)
CREATE INDEX idx_payments_shift ON payments(shift_id) WHERE shift_id IS NOT NULL;

-- Payments per subscription (total paid)
CREATE INDEX idx_payments_sub ON payments(subscription_id, status);

-- Message queue (pending today)
CREATE INDEX idx_msgs_pending ON message_queue(org_id, status, due_date) WHERE status = 'PENDING';

-- Audit log (entity history)
CREATE INDEX idx_audit_entity ON audit_log(org_id, entity_type, entity_id, created_at DESC);

-- Subscription events (timeline)
CREATE INDEX idx_sub_events ON subscription_events(subscription_id, created_at);

-- Risk scores (latest)
CREATE INDEX idx_risk_latest ON risk_scores(member_id, computed_at DESC);

-- Users login
CREATE UNIQUE INDEX idx_users_org_phone ON users(org_id, phone);

-- Members by branch
CREATE INDEX idx_members_branch ON members(org_id, branch_id) WHERE deleted_at IS NULL;
```

All composite indexes start with `org_id` to serve tenant-scoped queries efficiently.

---

## 4. Financial Precision

| Display | Database (BIGINT) |
|---------|-------------------|
| 700.00 EGP | 70000 |
| 1,200.50 EGP | 120050 |
| 0.00 EGP | 0 |

Conversion only in the application presentation layer. Aggregations (`SUM`, `AVG`) remain exact with integer arithmetic.

---

## 5. Flyway Migration Plan

### Naming Convention

`V{version}__{description}.sql` (double underscore)

### Ordered Migrations

```
db/migration/
├── V1__create_organizations_and_branches.sql
├── V2__create_users.sql
├── V3__create_members.sql
├── V4__create_plans.sql
├── V5__create_subscriptions_and_events.sql
├── V6__create_payments_and_refunds.sql
├── V7__create_cash_shifts.sql
├── V8__create_check_ins.sql
├── V9__create_message_queue.sql
├── V10__create_risk_scores.sql
├── V11__create_audit_log.sql
├── V12__create_shedlock.sql
├── V13__create_indexes.sql
├── V14__enable_rls_policies.sql
```

### Migration Rules

1. **Immutable once applied** — never edit a deployed migration.
2. **New changes** → new version files.
3. **One concern per file.**
4. **Backward compatible** — add with defaults, don't drop in same release.
5. **Tested in CI** via Testcontainers.

### Safe Schema Evolution

| Change | Approach |
|--------|---------|
| Add column | `ALTER TABLE ADD COLUMN ... DEFAULT ...` |
| Remove column | Stop reading → later migration drops it |
| Rename column | Add new → migrate → drop old (3 steps) |
| Add constraint | Add `NOT VALID` → `VALIDATE CONSTRAINT` separately |

---

## 6. RLS Policies

Every table with `org_id` gets Row-Level Security. See [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md) for full details.

```sql
ALTER TABLE {table} ENABLE ROW LEVEL SECURITY;
ALTER TABLE {table} FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON {table}
    USING (org_id = current_setting('app.current_org')::uuid);
```
