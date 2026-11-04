# استراتيجية قاعدة البيانات (Database Strategy)

> **المرجع:** [`../../Kammel_Project_Documentation/05_DATABASE_DESIGN.md`](../../Kammel_Project_Documentation/05_DATABASE_DESIGN.md) · [`../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md`](../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md)  
> **الملفات المرتبطة:** [entity-relationships](./entity-relationships.md) · [security-strategy](./security-strategy.md) · [decisions-and-assumptions](./decisions-and-assumptions.md)

---

## 1. المبادئ

| المبدأ | التنفيذ |
|---|---|
| **Single Source of Truth** | PostgreSQL 16 هو المصدر الوحيد للحقيقة. |
| **Shared Schema + RLS** | جدول واحد لكل الكيانات، وعزل بـ `org_id` + Row-Level Security. |
| **Financial Precision** | كل مبلغ `BIGINT` بالقروش. ممنوع `NUMERIC` بكسور أو `DOUBLE PRECISION`. |
| **UUID Keys** | `UUID DEFAULT gen_random_uuid()` — ممنوع Sequential IDs مكشوفة. |
| **UTC Always** | `TIMESTAMPTZ`، والعرض بـ `Africa/Cairo`. |
| **Immutability** | `audit_log`, `subscription_events`, `check_ins` — ممنوع `UPDATE`/`DELETE`. |
| **Fail Closed** | لو الـ Tenant Context مش متضبط → RLS ترجع صفر صفوف، مش كل الصفوف. |
| **Migration per Feature** | كل Migration تخص Feature واحدة وبتفعّل RLS لنفسها. |

---

## 2. أدوار قاعدة البيانات (Database Roles)

| Role | الاستخدام | الصلاحيات | RLS |
|---|---|---|---|
| `kammel_owner` | Migrations + Provisioning CLI | Owner على الـ Schema | `BYPASSRLS` implicit كـ Owner |
| `kammel_app` | التطبيق وقت التشغيل | `SELECT/INSERT/UPDATE/DELETE` حسب الجدول | **خاضع للـ RLS** |
| `kammel_readonly` | تحليلات/دعم (اختياري) | `SELECT` فقط | خاضع للـ RLS |

```sql
-- Provisioning (مرة واحدة، خارج Flyway)
CREATE ROLE kammel_owner LOGIN PASSWORD :'owner_pwd';
CREATE ROLE kammel_app   LOGIN PASSWORD :'app_pwd' NOSUPERUSER NOBYPASSRLS NOCREATEDB NOCREATEROLE;

CREATE DATABASE kammel OWNER kammel_owner;
\c kammel
GRANT USAGE ON SCHEMA public, app TO kammel_app;
```

> **ممنوع تماماً** إن التطبيق يتصل بـ `postgres` أو أي `SUPERUSER`. `SUPERUSER` بيتخطى الـ RLS حتى مع `FORCE ROW LEVEL SECURITY`.

### 2.1 Grants الحساسة

```sql
-- Append-Only
REVOKE UPDATE, DELETE ON audit_log, subscription_events, check_ins FROM kammel_app;
GRANT SELECT, INSERT ON audit_log, subscription_events, check_ins TO kammel_app;

-- Payments: التعديل مسموح على أعمدة الحالة فقط
REVOKE UPDATE, DELETE ON payments FROM kammel_app;
GRANT SELECT, INSERT ON payments TO kammel_app;
GRANT UPDATE (status, reversed_at, reversed_by) ON payments TO kammel_app;

-- Organizations: الـ App يقرأ، ويعدّل الاسم فقط. الحالة والإضافة للـ Owner CLI.
GRANT SELECT ON organizations TO kammel_app;
GRANT UPDATE (name) ON organizations TO kammel_app;
```

---

## 3. كتالوج الـ Migrations

### 3.1 ترتيب الـ MVP

| # | الملف | الـ Feature | المحتوى |
|:---:|---|---|---|
| V1 | `V1__create_extensions_and_helpers.sql` | `shared` | `pgcrypto` + `app.current_org_id()` + `app.enable_tenant_rls()` + `app.touch_updated_at()` + `app.forbid_mutation()` |
| V2 | `V2__create_organizations.sql` | `tenancy` | `organizations` + `slug` Unique. **بدون RLS** (Tenant Registry). |
| V3 | `V3__create_branches.sql` | `tenancy` | `branches` + RLS + Index `(org_id, active)` |
| V4 | `V4__create_users.sql` | `identity` | `users` + Role Check + RLS + Unique `(org_id, phone)` |
| V5 | `V5__create_refresh_tokens.sql` | `identity` | `refresh_tokens` + Hash Unique + RLS |
| V6 | `V6__create_members.sql` | `member` | `members` + `member_code` + `qr_token` + Soft Delete + RLS |
| V7 | `V7__create_plans.sql` | `plan` | `plans` + `grants_access` + RLS |
| V8 | `V8__create_subscriptions.sql` | `subscription` | `subscriptions` + Freeze/Cancel Columns + RLS |
| V9 | `V9__create_subscription_events.sql` | `subscription` | `subscription_events` + `org_id` + Immutability Trigger + RLS |
| V10 | `V10__create_payments.sql` | `payment` | `payments` + `receipt_number` + `(org_id, idempotency_key)` Unique + RLS |
| V11 | `V11__create_document_sequences.sql` | `payment` | `document_sequences` + `app.next_document_number()` + RLS |
| V12 | `V12__create_check_ins.sql` | `checkin` | `check_ins` + Override Check + Immutability Trigger + RLS |
| V13 | `V13__create_message_queue.sql` | `notification` | `message_queue` + `branch_id` + `opened_at` + `is_transactional` + Status Guard + RLS |
| V14 | `V14__create_risk_scores.sql` | `risk` | `risk_scores` + `org_id` + `branch_id` + `risk_level` + RLS |
| V15 | `V15__create_audit_log.sql` | `audit` | `audit_log` + `actor_type` + Immutability Trigger + RLS |
| V16 | `V16__create_shedlock.sql` | `shared` | `shedlock` (بدون `org_id` — مش بيانات مستأجر) |
| V17 | `V17__create_indexes.sql` | `shared` | كل الـ Indexes المركبة + Partial Indexes |
| V18 | `V18__create_sequences_and_constraints.sql` | `shared` | CHECK Constraints المؤجلة + Defaults + Comments |

### 3.2 ترتيب كيانات P1 (بعد الـ MVP)

| # | الملف | الـ Feature | المحتوى |
|:---:|---|---|---|
| V19 | `V19__create_cash_shifts.sql` | `shift` | `cash_shifts` + Partial Unique (Open Shift per User) + RLS |
| V20 | `V20__add_payments_shift_fk.sql` | `payment` | إضافة FK `payments.shift_id → cash_shifts.id` |
| V21 | `V21__create_refunds.sql` | `payment` | `refunds` + RLS + منع التعديل |
| V22 | `V22__create_leads.sql` | `lead` | `leads` + `lead_followups` (Immutability Trigger) + RLS |
| V23 | `V23__create_member_portal_tokens.sql` | `member-portal` | `member_portal_tokens` + `app.current_member_id()` + Restrictive RLS على `members`/`subscriptions`/`payments`/`check_ins` |
| V24 | `V24__create_portal_renewal_requests.sql` | `member-portal` | `portal_renewal_requests` + Partial Unique + RLS |
| V25 | `V25__add_organization_branding.sql` | `tenancy` | `organizations.logo_url` + `primary_color` (SC-28) |
| V26 | `V26__create_ai_model_registry.sql` | `ai` (P2) | `ai_model_registry` — Global |
| V27 | `V27__add_risk_prediction_columns.sql` | `ai` (P2) | `risk_scores.churn_probability` (SC-30) |

> **قاعدة:** ممنوع تعديل Migration اتطبقت. أي تغيير = ملف جديد.

---

## 4. الـ RLS — التنفيذ الكامل

### 4.1 Helper Functions (V1)

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE SCHEMA IF NOT EXISTS app;

-- بيرجع NULL لو الـ Context مش متضبط → RLS تفشل مغلقة
CREATE OR REPLACE FUNCTION app.current_org_id()
RETURNS uuid
LANGUAGE sql
STABLE
AS $$
    SELECT NULLIF(current_setting('app.current_org', true), '')::uuid;
$$;

-- تفعيل RLS + Policy موحدة لأي جدول فيه org_id
CREATE OR REPLACE FUNCTION app.enable_tenant_rls(p_table text)
RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
    EXECUTE format('ALTER TABLE %I ENABLE ROW LEVEL SECURITY', p_table);
    EXECUTE format('ALTER TABLE %I FORCE ROW LEVEL SECURITY', p_table);
    EXECUTE format('DROP POLICY IF EXISTS tenant_isolation ON %I', p_table);
    EXECUTE format(
        'CREATE POLICY tenant_isolation ON %I
             FOR ALL
             USING (org_id = app.current_org_id())
             WITH CHECK (org_id = app.current_org_id())',
        p_table
    );
END;
$$;

-- updated_at تلقائي
CREATE OR REPLACE FUNCTION app.touch_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$;

-- منع أي UPDATE/DELETE على الجداول Append-Only
CREATE OR REPLACE FUNCTION app.forbid_mutation()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE EXCEPTION 'Table % is append-only: % is forbidden', TG_TABLE_NAME, TG_OP
        USING ERRCODE = '55000';
END;
$$;
```

### 4.2 استخدام الـ Helper في كل Migration

```sql
-- مثال: V3__create_branches.sql
CREATE TABLE branches (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    org_id      UUID NOT NULL REFERENCES organizations(id),
    name        VARCHAR(255) NOT NULL,
    address     VARCHAR(500),
    timezone    VARCHAR(50) NOT NULL DEFAULT 'Africa/Cairo',
    active      BOOLEAN NOT NULL DEFAULT true,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_branches_org_active ON branches(org_id, active);
SELECT app.enable_tenant_rls('branches');
```

> **Checkpoint:** أي جدول فيه `org_id` لازم يكون فيه `SELECT app.enable_tenant_rls('...')` في نفس الملف. مراجعة الـ PR ترفض غير كده.

### 4.3 ضبط الـ Context

```sql
SET LOCAL app.current_org = 'a1b2c3d4-e5f6-4a5b-8c9d-0e1f2a3b4c5d';
```

- `SET LOCAL` جوه Transaction فقط.
- بينتهي تلقائياً عند `COMMIT`/`ROLLBACK` — مفيش Leakage في HikariCP.
- التطبيق بينفذه عبر `TenantAwareJpaTransactionManager` — مش يدوي في الـ Repositories.
- لو الـ Context مش متضبط: `app.current_org_id()` ترجع `NULL` → `org_id = NULL` → صفر صفوف.

### 4.4 استثناء `organizations`

`organizations` هي الـ Tenant Registry ومفيهاش بيانات بيزنس حساسة، وRLS عليها بتكسر Login (مش عارفين الـ Org قبل الـ JWT).

**الحل:**
- RLS **متقفلة** على `organizations`.
- `kammel_app` عنده `SELECT` كامل و`UPDATE(name)` فقط.
- `INSERT` / `UPDATE(status)` / `DELETE` ممنوعين على `kammel_app` — بيتموا عبر Provisioning CLI بـ `kammel_owner`.
- كل الجداول التانية **إجبارية** RLS.

### 4.5 عزل العضو داخل نفس الـ Org (P1 — Member Portal)

الـ RLS الأساسية بتحمي **بين الجيمات**. البوابة محتاجة حماية **بين أعضاء نفس الجيم**:

```sql
-- V23
CREATE OR REPLACE FUNCTION app.current_member_id()
RETURNS uuid
LANGUAGE sql STABLE AS $$
    SELECT NULLIF(current_setting('app.current_member', true), '')::uuid;
$$;

-- Restrictive = AND مع الـ Policy الأساسية
CREATE POLICY portal_member_scope ON members AS RESTRICTIVE FOR SELECT
    USING (app.current_member_id() IS NULL OR id = app.current_member_id());

CREATE POLICY portal_member_scope ON subscriptions AS RESTRICTIVE FOR SELECT
    USING (app.current_member_id() IS NULL OR member_id = app.current_member_id());

CREATE POLICY portal_member_scope ON payments AS RESTRICTIVE FOR SELECT
    USING (app.current_member_id() IS NULL OR member_id = app.current_member_id());

CREATE POLICY portal_member_scope ON check_ins AS RESTRICTIVE FOR SELECT
    USING (app.current_member_id() IS NULL OR member_id = app.current_member_id());
```

- لما `app.current_member` مش متضبط (Staff) → `NULL` → الـ Policy بتسمح، والـ Org Policy هي الحاكمة.
- لما Portal Context شغال → كل استعلام مقيد بالعضو.
- **مش بديل** عن `member_id = :memberId` في الـ Repository — ده دفاع تاني.

---

## 5. الـ Immutability Triggers

### 5.1 Audit Log / Subscription Events / Check-ins

```sql
CREATE TRIGGER trg_audit_log_immutable
    BEFORE UPDATE OR DELETE ON audit_log
    FOR EACH ROW EXECUTE FUNCTION app.forbid_mutation();

CREATE TRIGGER trg_subscription_events_immutable
    BEFORE UPDATE OR DELETE ON subscription_events
    FOR EACH ROW EXECUTE FUNCTION app.forbid_mutation();

CREATE TRIGGER trg_check_ins_immutable
    BEFORE UPDATE OR DELETE ON check_ins
    FOR EACH ROW EXECUTE FUNCTION app.forbid_mutation();
```

### 5.2 Payments — Status فقط

```sql
CREATE OR REPLACE FUNCTION app.guard_payment_update()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        RAISE EXCEPTION 'payments cannot be deleted' USING ERRCODE = '55000';
    END IF;

    IF NEW.amount_minor      <> OLD.amount_minor
       OR NEW.subscription_id <> OLD.subscription_id
       OR NEW.member_id       <> OLD.member_id
       OR NEW.method          <> OLD.method
       OR NEW.receipt_number  <> OLD.receipt_number
       OR NEW.idempotency_key <> OLD.idempotency_key
       OR NEW.created_at      <> OLD.created_at THEN
        RAISE EXCEPTION 'financial fields of payments are immutable' USING ERRCODE = '55000';
    END IF;

    IF NOT (OLD.status = 'COMPLETED' AND NEW.status = 'REVERSED') THEN
        RAISE EXCEPTION 'invalid payment status transition: % -> %', OLD.status, NEW.status
            USING ERRCODE = '55000';
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_payments_guard
    BEFORE UPDATE OR DELETE ON payments
    FOR EACH ROW EXECUTE FUNCTION app.guard_payment_update();
```

### 5.3 Message Queue — Status فقط

```sql
CREATE OR REPLACE FUNCTION app.guard_message_update()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        RAISE EXCEPTION 'message_queue rows cannot be deleted' USING ERRCODE = '55000';
    END IF;

    IF NEW.rendered_text <> OLD.rendered_text
       OR NEW.member_id  <> OLD.member_id
       OR NEW.template   <> OLD.template
       OR NEW.due_date   <> OLD.due_date THEN
        RAISE EXCEPTION 'message content is immutable' USING ERRCODE = '55000';
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_message_queue_guard
    BEFORE UPDATE OR DELETE ON message_queue
    FOR EACH ROW EXECUTE FUNCTION app.guard_message_update();
```

> **ملاحظة:** Triggers دي Defense-in-Depth. الـ Service Layer لازم يمنع العمليات دي قبل ما توصل للـ DB، والـ Tests تثبت الاتنين.

### 5.4 Member Portal — Token + Renewal Request (P1)

```sql
CREATE OR REPLACE FUNCTION app.guard_portal_token_update()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        RAISE EXCEPTION 'portal tokens cannot be deleted' USING ERRCODE = '55000';
    END IF;

    IF NEW.token_hash <> OLD.token_hash
       OR NEW.member_id <> OLD.member_id
       OR NEW.token_type <> OLD.token_type
       OR NEW.org_id <> OLD.org_id THEN
        RAISE EXCEPTION 'portal token identity is immutable' USING ERRCODE = '55000';
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_portal_token_guard
    BEFORE UPDATE OR DELETE ON member_portal_tokens
    FOR EACH ROW EXECUTE FUNCTION app.guard_portal_token_update();

CREATE OR REPLACE FUNCTION app.guard_renewal_request_update()
RETURNS trigger
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        RAISE EXCEPTION 'renewal requests cannot be deleted' USING ERRCODE = '55000';
    END IF;

    IF NEW.member_id <> OLD.member_id
       OR NEW.org_id <> OLD.org_id
       OR NEW.branch_id <> OLD.branch_id
       OR NEW.created_at <> OLD.created_at THEN
        RAISE EXCEPTION 'renewal request identity is immutable' USING ERRCODE = '55000';
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_renewal_request_guard
    BEFORE UPDATE OR DELETE ON portal_renewal_requests
    FOR EACH ROW EXECUTE FUNCTION app.guard_renewal_request_update();
```

---

## 6. ترقيم المستندات (Document Sequences)

### 6.1 الجدول

```sql
CREATE TABLE document_sequences (
    org_id        UUID NOT NULL REFERENCES organizations(id),
    document_type VARCHAR(30) NOT NULL,
    year          INTEGER NOT NULL,
    next_value    BIGINT NOT NULL DEFAULT 1 CHECK (next_value > 0),
    PRIMARY KEY (org_id, document_type, year)
);

SELECT app.enable_tenant_rls('document_sequences');
```

### 6.2 الدالة الذرية

```sql
CREATE OR REPLACE FUNCTION app.next_document_number(
    p_org_id UUID,
    p_type   VARCHAR,
    p_year   INTEGER
)
RETURNS BIGINT
LANGUAGE plpgsql
AS $$
DECLARE
    v_value BIGINT;
BEGIN
    INSERT INTO document_sequences (org_id, document_type, year, next_value)
    VALUES (p_org_id, p_type, p_year, 1)
    ON CONFLICT (org_id, document_type, year)
    DO UPDATE SET next_value = document_sequences.next_value + 1
    RETURNING next_value INTO v_value;

    RETURN v_value;
END;
$$;
```

- `ON CONFLICT DO UPDATE` بياخد Row Lock تلقائي — آمن تحت التزامن.
- التنسيق في الـ Application: `KM-%d-%06d` → `KM-2026-000001`.
- لو محتاجين Reset سنوي → الـ `year` جزء من الـ PK.
- **ممنوع** استخدام PostgreSQL `SEQUENCE` عادية لأنها Global ومش Per-Org.

---

## 7. القيود (Constraints Catalog)

| الجدول | Constraint | التعريف |
|---|---|---|
| `organizations` | `chk_org_status` | `status IN ('ACTIVE','SUSPENDED','CLOSED')` |
| `organizations` | `uk_org_slug` | `UNIQUE (slug)` — `slug` lowercase |
| `organizations` | `chk_org_primary_color` | `primary_color ~ '^#[0-9A-Fa-f]{6}$'` (SC-28) |
| `users` | `chk_user_role` | `role IN ('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST','TRAINER')` |
| `users` | `chk_owner_branch` | `(role = 'GYM_OWNER' AND branch_id IS NULL) OR (role <> 'GYM_OWNER' AND branch_id IS NOT NULL)` |
| `members` | `chk_member_status` | `status IN ('ACTIVE','INACTIVE')` |
| `members` | `chk_member_gender` | `gender IS NULL OR gender IN ('MALE','FEMALE')` |
| `plans` | `chk_plan_type` | `type IN ('MONTHLY','QUARTERLY','SEMI_ANNUAL','ANNUAL','SESSION_PACK','DAY_PASS','PT_PACKAGE','TIME_RESTRICTED')` |
| `plans` | `chk_plan_duration` | `duration_days > 0` |
| `plans` | `chk_plan_price` | `price_minor > 0` (SC-22) |
| `plans` | `chk_plan_sessions` | `session_count IS NULL OR session_count > 0` |
| `subscriptions` | `chk_sub_dates` | `end_date >= start_date` |
| `subscriptions` | `chk_sub_status` | `status IN ('PENDING_PAYMENT','ACTIVE','FROZEN','GRACE','EXPIRED','CANCELLED')` |
| `subscriptions` | `chk_sub_frozen_days` | `frozen_days_used >= 0` |
| `subscriptions` | `chk_sub_sessions` | `sessions_remaining IS NULL OR sessions_remaining >= 0` |
| `subscriptions` | `chk_sub_renewal_self` | `renewal_of IS NULL OR renewal_of <> id` (SC-25) |
| `payments` | `chk_payment_amount` | `amount_minor > 0` |
| `payments` | `chk_payment_method` | `method IN ('CASH','INSTAPAY','BANK_TRANSFER','ONLINE')` |
| `payments` | `chk_payment_status` | `status IN ('COMPLETED','REVERSED')` |
| `check_ins` | `chk_checkin_method` | `method IN ('NAME_SEARCH','PHONE_SEARCH','MEMBER_CODE','STATIC_QR','DYNAMIC_QR')` (SC-26) |
| `check_ins` | `chk_override_reason` | `overridden_by IS NULL OR override_reason IS NOT NULL` |
| `message_queue` | `chk_message_template` | `template IN ('RENEWAL_REMINDER','GRACE_EXPOSURE','MISS_YOU','WELCOME','COLLECTION','PORTAL_MAGIC_LINK')` (SC-24/SC-29) |
| `message_queue` | `chk_message_status` | `status IN ('PENDING','OPENED','SENT_CONFIRMED','CANCELLED','FAILED')` |
| `risk_scores` | `chk_risk_score` | `score IS NULL OR score BETWEEN 0 AND 100` |
| `risk_scores` | `chk_risk_level` | `risk_level IS NULL OR risk_level IN ('LOW','MEDIUM','HIGH','CRITICAL','INSUFFICIENT_DATA')` |
| `risk_scores` | `chk_churn_probability` | `churn_probability IS NULL OR churn_probability BETWEEN 0 AND 1` (SC-30) |
| `portal_renewal_requests` | `chk_renewal_status` | `status IN ('NEW','CONTACTED','CONVERTED','DISMISSED')` |
| `portal_renewal_requests` | `chk_renewal_dismiss_reason` | `status <> 'DISMISSED' OR outcome_note IS NOT NULL` |
| `audit_log` | `chk_actor_type` | `actor_type IN ('USER','SYSTEM')` |
| `audit_log` | `chk_actor_required` | `(actor_type = 'SYSTEM' AND actor_id IS NULL) OR (actor_type = 'USER' AND actor_id IS NOT NULL)` |

---

## 8. الفهارس (Index Catalog)

```sql
-- Tenancy
CREATE UNIQUE INDEX uk_organizations_slug ON organizations(slug);
CREATE INDEX idx_branches_org_active ON branches(org_id, active);

-- Identity
CREATE UNIQUE INDEX uk_users_org_phone ON users(org_id, phone);
CREATE UNIQUE INDEX uk_users_org_email ON users(org_id, email) WHERE email IS NOT NULL;
CREATE INDEX idx_users_org_branch ON users(org_id, branch_id);
CREATE UNIQUE INDEX uk_refresh_token_hash ON refresh_tokens(token_hash);
CREATE INDEX idx_refresh_user_active ON refresh_tokens(user_id, expires_at) WHERE revoked_at IS NULL;

-- Members
CREATE UNIQUE INDEX uk_members_org_phone ON members(org_id, phone) WHERE deleted_at IS NULL;
CREATE UNIQUE INDEX uk_members_org_code ON members(org_id, member_code);
CREATE UNIQUE INDEX uk_members_qr ON members(qr_token);
CREATE INDEX idx_members_branch_status ON members(org_id, branch_id, status) WHERE deleted_at IS NULL;
CREATE INDEX idx_members_trainer ON members(org_id, assigned_trainer_id) WHERE deleted_at IS NULL;

-- Plans
CREATE INDEX idx_plans_org_active ON plans(org_id, active);

-- Subscriptions
CREATE INDEX idx_subs_org_status_end ON subscriptions(org_id, status, end_date)
    WHERE status IN ('ACTIVE','GRACE');
CREATE INDEX idx_subs_org_member ON subscriptions(org_id, member_id, created_at DESC);
CREATE INDEX idx_subs_org_plan ON subscriptions(org_id, plan_id);
CREATE INDEX idx_subs_renewal_of ON subscriptions(renewal_of) WHERE renewal_of IS NOT NULL;

-- Subscription Events
CREATE INDEX idx_sub_events_org_sub ON subscription_events(org_id, subscription_id, created_at);
CREATE INDEX idx_sub_events_org_type ON subscription_events(org_id, type, created_at DESC);

-- Payments
CREATE UNIQUE INDEX uk_payments_idempotency ON payments(org_id, idempotency_key);
CREATE UNIQUE INDEX uk_payments_receipt ON payments(org_id, receipt_number);
CREATE INDEX idx_payments_sub_status ON payments(subscription_id, status);
CREATE INDEX idx_payments_org_member ON payments(org_id, member_id, created_at DESC);
CREATE INDEX idx_payments_org_created ON payments(org_id, created_at DESC);
CREATE INDEX idx_payments_shift ON payments(shift_id) WHERE shift_id IS NOT NULL;

-- Check-ins
CREATE INDEX idx_checkins_org_member_time ON check_ins(org_id, member_id, checked_in_at DESC);
CREATE INDEX idx_checkins_org_branch_time ON check_ins(org_id, branch_id, checked_in_at DESC);
CREATE INDEX idx_checkins_org_time ON check_ins(org_id, checked_in_at DESC);

-- Notifications
CREATE INDEX idx_msgs_pending ON message_queue(org_id, status, due_date) WHERE status = 'PENDING';
CREATE INDEX idx_msgs_member_template ON message_queue(org_id, member_id, template, sent_at DESC);
CREATE INDEX idx_msgs_branch_status ON message_queue(org_id, branch_id, status);
CREATE INDEX idx_msgs_attribution ON message_queue(org_id, attributed_subscription_id)
    WHERE attributed_subscription_id IS NOT NULL;

-- Risk
CREATE INDEX idx_risk_latest ON risk_scores(org_id, member_id, computed_at DESC);
CREATE INDEX idx_risk_level ON risk_scores(org_id, risk_level, score DESC) WHERE risk_level IS NOT NULL;
CREATE INDEX idx_risk_branch ON risk_scores(org_id, branch_id, computed_at DESC);

-- Audit
CREATE INDEX idx_audit_org_time ON audit_log(org_id, created_at DESC);
CREATE INDEX idx_audit_entity ON audit_log(org_id, entity_type, entity_id, created_at DESC);
CREATE INDEX idx_audit_actor ON audit_log(org_id, actor_id, created_at DESC);

-- Member Portal (P1)
CREATE UNIQUE INDEX uk_mpt_hash ON member_portal_tokens(org_id, token_hash);
CREATE INDEX idx_mpt_member_active ON member_portal_tokens(org_id, member_id, token_type)
    WHERE revoked_at IS NULL;
CREATE INDEX idx_mpt_expiry ON member_portal_tokens(expires_at)
    WHERE revoked_at IS NULL;
CREATE UNIQUE INDEX uk_portal_renewal_open ON portal_renewal_requests(org_id, member_id)
    WHERE status = 'NEW';
CREATE INDEX idx_portal_renewal_queue ON portal_renewal_requests(org_id, branch_id, status, created_at DESC);
```

### قواعد الفهارس
1. كل Index مركّب يبدأ بـ `org_id` — عشان RLS + Query Pattern.
2. Partial Indexes للـ Soft Delete والحالات النشطة.
3. ممنوع Index على عمود Low-Cardinality لوحده (`status` فقط).
4. أي Query جديدة في التقارير → `EXPLAIN ANALYZE` قبل الـ Merge.
5. `pg_stat_statements` مفعّل في الـ Staging لرصد الـ Slow Queries.

---

## 9. استخدام JSONB

| الجدول | العمود | الشكل | ملاحظات |
|---|---|---|---|
| `subscription_events` | `metadata` | Object | `{ "frozenDays": 7, "oldEndDate": "..." }` |
| `risk_scores` | `reasons` | Array of Strings | أسباب عربية/إنجليزية مفهومة |
| `risk_scores` | `signals` | Object | القيم الخام للحساب |
| `audit_log` | `before_state` / `after_state` | Object | Snapshot مختصر للكيانات |

**قواعد:**
- ممنوع JSONB كبديل عن أعمدة أساسية.
- ممنوع Query على JSONB في المسارات الحرجة بدون GIN Index.
- `reasons` لازم يكون Array — الـ API بترجعه كما هو.

---

## 10. Provisioning وSeed

```text
scripts/
├── provision-org.sh        # إنشاء Org + Branch + GYM_OWNER (كـ kammel_owner)
├── seed-dev-data.sql       # بيانات تجريبية للـ dev فقط
└── restore-drill.sh        # اختبار استرجاع Backup
```

**قواعد:**
- Provisioning بـ CLI/SQL كـ `kammel_owner` — مفيش `PLATFORM_ADMIN` في الـ MVP ([ASM-07](./decisions-and-assumptions.md#asm-07)).
- بيانات الـ Seed ممنوعة في `prod`.
- كل Org جديدة لازم يبقى عندها Branch واحد على الأقل (BR-001).
- كلمة مرور الـ Owner الأولية تتغير إجبارياً عند أول Login.

---

## 11. الـ Backup والاسترجاع

| العنصر | السياسة |
|---|---|
| **Daily Backup** | `pg_dump -Fc` يومياً 2:00 AM بتوقيت القاهرة. |
| **التشفير** | GPG بـ Key خارج السيرفر قبل الرفع. |
| **التخزين** | S3/Wasabi — Retention 30 يوم. |
| **WAL Archiving** | PITR اختياري في أول شهر Pilot (RPO 15 دقيقة). |
| **Restore Drill** | مرة كل شهر على Staging + تسجيل الوقت. |
| **RTO / RPO** | RTO ≤ 4 ساعات / RPO ≤ 24 ساعة (MVP). |

### Restore Runbook
```bash
# 1. إنشاء DB جديدة
createdb -O kammel_owner kammel_restore

# 2. فك التشفير والاسترجاع
gpg --decrypt backup.dump.gpg | pg_restore -d kammel_restore --no-owner

# 3. التأكد من عدد الصفوف
psql -d kammel_restore -c "SELECT count(*) FROM members;"

# 4. تشغيل Flyway validate
./mvnw flyway:validate -Dflyway.url=jdbc:postgresql://localhost:5432/kammel_restore
```

> **Checkpoint 4:** لازم يكون فيه Restore Drill ناجح موثق قبل الـ Pilot.

---

## 12. اختبار الـ Migrations

| الاختبار | الأداة | معيار النجاح |
|---|---|---|
| Clean Migrate | Testcontainers PostgreSQL 16 | `flyway migrate` من DB فاضية بدون أخطاء |
| Validate | `flyway validate` | Checksums متطابقة |
| RLS Coverage | SQL Query على `pg_class`/`pg_policy` | كل جدول فيه `org_id` عنده Policy |
| RLS Behavior | JDBC كـ `kammel_app` | Org A مش شايف Org B |
| Immutability | SQL مباشر | `UPDATE audit_log` → Exception |
| Sequence Concurrency | 20 Threads | أرقام إيصالات فريدة ومتسلسلة |
| Rollback Safety | Manual | Migration فاشلة مابتسيبش Schema نص-نص |

```sql
-- RLS Coverage Check
SELECT c.relname
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname = 'public'
  AND c.relkind = 'r'
  AND EXISTS (
      SELECT 1 FROM information_schema.columns col
      WHERE col.table_name = c.relname AND col.column_name = 'org_id'
  )
  AND NOT EXISTS (SELECT 1 FROM pg_policy p WHERE p.polrelid = c.oid);
-- المتوقع: صفر صفوف
```

---

## 13. أخطاء شائعة

| الخطأ | النتيجة | الصح |
|---|---|---|
| `SET` بدل `SET LOCAL` | Tenant Leakage بين الـ Requests | `SET LOCAL` |
| `SUPERUSER` للتطبيق | RLS بتتخطى بالكامل | `kammel_app` Non-Superuser |
| RLS في Migration واحدة في الآخر | فترة بدون حماية | مع كل جدول |
| `org_id` بدون Index | Full Scan + بطء | Index يبدأ بـ `org_id` |
| `DELETE` على Payments | فقدان Ledger | Reversal |
| `NUMERIC(10,2)` للمبالغ | تعارض مع `Long` | `BIGINT` قروش |
| Sequence عادية للإيصالات | أرقام مشتركة بين الجيمات | `document_sequences` |
| Trigger بس بدون Service Validation | أخطاء 500 بدل 422 | Validation في الاتنين |
| Migration معدّلة بعد التطبيق | Checksum Mismatch | Migration جديدة |

---

## 14. مرتبط بـ

- [entity-relationships](./entity-relationships.md) — الكيانات والعلاقات.
- [security-strategy](./security-strategy.md) — RLS وAuth وThreat Model.
- [testing-strategy](../testing-strategy.md) — اختبارات الـ DB.
- [implementation-phases](../development-roadmap/implementation-phases.md) — توقيت كل Migration.
