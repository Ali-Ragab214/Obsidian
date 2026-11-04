# Audit — خطة التنفيذ

> **الـ Phase:** 1 (Core) / 3 (API) · **المدة:** 2–3 أيام Core + 2 أيام API  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md) · [implementation-phases](../../development-roadmap/implementation-phases.md)

---

## 1. المهام

### TSK-AUD-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الـ Phase** | 1 |
| **الاعتماديات** | TSK-TEN-001 (V1–V3) — `actor_id` **مفيش عليه FK** لـ `users`، فالـ Migration مش محتاجة `identity` |
| **المخرجات** | `V15__create_audit_log.sql`, `AuditLog.java`, `ActorType.java` |
| **معيار القبول** | RLS + Immutability Trigger + Grants + CHECK `actor_type` |

**خطوات:**
1. الجدول بكل الأعمدة + `org_id`.
2. `CHECK ((actor_type = 'SYSTEM' AND actor_id IS NULL) OR (actor_type = 'USER' AND actor_id IS NOT NULL))`.
3. `SELECT app.enable_tenant_rls('audit_log');`.
4. `CREATE TRIGGER trg_audit_log_immutable BEFORE UPDATE OR DELETE ...`.
5. `REVOKE UPDATE, DELETE ... FROM kammel_app;`.
6. Indexes: `(org_id, created_at DESC)`, `(org_id, entity_type, entity_id, created_at DESC)`, `(org_id, actor_id, created_at DESC)`.

---

### TSK-AUD-002 — `AuditService` + Context

| البند | التفصيل |
|---|---|
| **الـ Phase** | 1 |
| **الاعتماديات** | TSK-AUD-001 |
| **المخرجات** | `AuditService`, `AuditServiceImpl`, `AuditCommand`, `AuditContext` |
| **معيار القبول** | `record` / `recordSystem` / `recordBestEffort` + JSON Snapshots |

**خطوات:**
1. `AuditCommand` Record.
2. `AuditServiceImpl.record` — `orgId` من `TenantContext`.
3. `recordSystem` — `actorType = SYSTEM`.
4. `recordBestEffort` — `REQUIRES_NEW` + Catch.
5. `AuditContext` — IP / User Agent / Trace ID.
6. Jackson لتحويل الـ Snapshots.

---

### TSK-AUD-003 — دمج الـ Audit في الـ Features

| البند | التفصيل |
|---|---|
| **الـ Phase** | 2 |
| **الاعتماديات** | TSK-AUD-002 + كل Feature |
| **المخرجات** | استدعاءات `AuditService` في Services |
| **معيار القبول** | كل Action في الكتالوج له Call Site |

**قائمة الدمج:**

| Feature | Actions |
|---|---|
| `tenancy` | `ORGANIZATION_*`, `BRANCH_*` |
| `identity` | `LOGIN_*`, `USER_*`, `PASSWORD_*`, `REFRESH_TOKEN_REUSE` |
| `member` | `MEMBER_*` |
| `plan` | `PLAN_*` |
| `subscription` | `SUBSCRIPTION_*` |
| `payment` | `PAYMENT_*` |
| `checkin` | `CHECKIN_*` |
| `notification` | `MESSAGE_*` |

---

### TSK-AUD-004 — Query API

| البند | التفصيل |
|---|---|
| **الـ Phase** | 3 |
| **الاعتماديات** | TSK-AUD-002 |
| **المخرجات** | `AuditLogController` + `GET /audit-logs` |
| **معيار القبول** | Owner فقط + فلاتر + Pagination |

**خطوات:**
1. `GET /audit-logs` — فلاتر `actorId`, `action`, `entityType`, `entityId`, `from`, `to`.
2. `GET /audit-logs/{id}`.
3. `GET /audit-logs/entities/{entityType}/{entityId}`.
4. `AuditLogSpecifications`.
5. `@PreAuthorize("hasRole('GYM_OWNER')")`.

---

### TSK-AUD-005 — Immutability Verification

| البند | التفصيل |
|---|---|
| **الـ Phase** | 1 |
| **الاعتماديات** | TSK-AUD-001 |
| **المخرجات** | DB Tests + Grants Verification |
| **معيار القبول** | `UPDATE`/`DELETE` → Exception أو Permission Denied |

---

### TSK-AUD-006 — Tests

| البند | التفصيل |
|---|---|
| **الـ Phase** | 1/3 |
| **الاعتماديات** | TSK-AUD-004, TSK-AUD-005 |
| **المخرجات** | Unit + Integration + Isolation + Immutability |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) بالكامل |

---

## 2. الترتيب

```text
Phase 1:
TSK-AUD-001  Migration + Entity
      ↓
TSK-AUD-002  AuditService + Context
      ↓
TSK-AUD-005  Immutability Verification
      ↓
(الـ Features التانية بتستخدمه)

Phase 3:
TSK-AUD-004  Query API
      ↓
TSK-AUD-006  Tests
```

---

## 3. معايير القبول النهائية

- [ ] `audit_log` موجود + RLS + Trigger.
- [ ] `kammel_app` مش قادر يعمل `UPDATE`/`DELETE`.
- [ ] `AuditService.record` شغال في نفس الـ Transaction.
- [ ] كل عملية حساسة متسجلة.
- [ ] `actorType = SYSTEM` للـ Jobs.
- [ ] مفيش PII في `action` أو Application Logs.
- [ ] `GET /audit-logs` للـ Owner فقط.
- [ ] الفلاتر شغالة.
- [ ] Isolation Test ناجح.
- [ ] Immutability Test ناجح.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| نسيان Audit في Feature جديدة | Checklist + Arch Test + Code Review |
| Audit بيكسر Transaction | `record` مصمم عشان يفشل معاها (مقصود) |
| PII في `beforeState` | Snapshots مختصرة + Code Review |
| بطء الكتابة | Indexes + Append-Only + Partitioning في P1 لو لازم |
| تسريب Audit بين المستأجرين | RLS + Isolation Test |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [requirements-traceability](../../architecture/requirements-traceability.md)
