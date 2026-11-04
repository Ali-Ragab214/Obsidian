# Audit — السجل غير القابل للتعديل

> **الـ Phase:** 1 (Core) / 3 (API) · **الأولوية:** P0 · **الترتيب التنفيذي:** 3  
> **الـ Package:** `com.kammel.features.audit`  
> **المرجع:** [`../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md`](../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md) · [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md)

---

## 1. الميزة دي بتعمل إيه؟

`audit` هي **الذاكرة القانونية** للنظام. بتسجل كل عملية حساسة بشكل **Append-Only**:

- مين عمل إيه (`actorType`, `actorId`).
- على أنهي كيان (`entityType`, `entityId`).
- الحالة قبل وبعد (`beforeState`, `afterState`).
- السبب (`reason`) وIP.

**مش مسؤولة عن:**
- Application Logs العادية → `shared` / Logging.
- Metrics / Tracing → Observability.
- SIEM / Log Aggregation → خارج النطاق.
- Audit الـ DB نفسه (`pg_audit`) → خارج النطاق.

---

## 2. قرار التوقيت

| المرحلة | المحتوى |
|---|---|
| **Phase 1** | `AuditLog` Entity + `AuditService` + Migration + RLS + Triggers. |
| **Phase 2** | استخدامه في `member`, `plan`, `subscription`, `payment`. |
| **Phase 3** | `GET /audit-logs` API + فلاتر + شاشة الـ Owner. |

> **ليه بدري؟** لأن أول عملية Freeze/Override/Reversal لازم يكون لها أثر. لو استنينا Phase 3 هيبقى فيه Gap أمني ([C-10](../../architecture/decisions-and-assumptions.md#2-التعارضات-اللي-تم-حلها-resolved-conflicts)).

---

## 3. النطاق

### داخل الـ MVP
- [x] `audit_log` Immutable + RLS.
- [x] `AuditService.record(...)`.
- [x] `AuditContext` (IP / User Agent / Trace ID).
- [x] كتالوج Actions موحد.
- [x] تسجيل كل العمليات الحساسة من الـ Features.
- [x] Query API للـ Owner.
- [x] فلترة بالـ Actor / Action / Entity / Date.
- [x] منع UPDATE/DELETE على مستوى الـ DB والـ Grants.

### خارج الـ MVP
- [ ] تصدير CSV/PDF.
- [ ] Retention Archival تلقائي.
- [ ] Alerting على عمليات مشبوهة.
- [ ] Audit للقراءات (Read Access Logging).
- [ ] SIEM Integration.

---

## 4. المستخدمون

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| تسجيل Audit (تلقائي) | ✅ | ✅ | ✅ | ✅ |
| عرض Audit Log | ✅ | ❌ | ❌ | ❌ |
| تصدير | ❌ (P1) | ❌ | ❌ | ❌ |

> **الـ Manager مش بيشوف Audit Log** في الـ MVP. لو محتاج تاريخ اشتراك، بيشوف `GET /subscriptions/{id}/events` (Subscription Timeline) — مش الـ Audit العام.

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-006 | كل Discount/Freeze/Refund/Payment Mod متسجل في Audit. |
| BR-007 | Audit Log Immutable. |
| AUD-01 | ممنوع UPDATE/DELETE على `audit_log` — DB Trigger + Grants. |
| AUD-02 | كل Operation حساسة تتسجل **في نفس الـ Transaction**. |
| AUD-03 | ممنوع PII في Application Logs — الـ Audit هو السجل الآمن. |
| AUD-04 | `actorType = SYSTEM` → `actorId = NULL`. |
| AUD-05 | `reason` إلزامي للـ Freeze / Cancel / Override / Reversal. |
| AUD-06 | ممنوع تسجيل Password / Token / Full Request Body. |
| AUD-07 | الـ Query API للـ Owner فقط. |
| AUD-08 | الاحتفاظ 5 سنوات (مراجعة قانونية قبل الإطلاق). |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `TenantContext`, `SecurityUtils`, `ApiResponse` |
| `tenancy` | Compile | `OrganizationQueryService` (للتحقق عند الـ Query) |

> `actor_id` في `audit_log` **مفيش عليه FK** لـ `users` — عشان الـ Audit يفضل قادر يسجل حتى لو المستخدم اتحذف أو اتعطل. عشان كده `audit` مش محتاج `identity` في الـ Migration.

**مين بيعتمد على `audit`؟**
كل الـ Features اللي بتعمل عمليات حساسة — عبر `AuditService` Interface فقط.

> **ممنوع** `audit` يعتمد على أي Feature. الـ Dependency اتجاهها واحد.

---

## 7. كتالوج الـ Actions

| المجال | Actions |
|---|---|
| Tenancy | `ORGANIZATION_PROVISIONED`, `ORGANIZATION_UPDATED`, `BRANCH_CREATED`, `BRANCH_UPDATED`, `BRANCH_DEACTIVATED` |
| Identity | `LOGIN_SUCCESS`, `LOGIN_FAILED`, `LOGOUT`, `PASSWORD_CHANGED`, `PASSWORD_RESET`, `REFRESH_TOKEN_REUSE`, `USER_CREATED`, `USER_UPDATED`, `USER_ROLE_CHANGED`, `USER_DEACTIVATED` |
| Member | `MEMBER_CREATED`, `MEMBER_UPDATED`, `MEMBER_DELETED`, `MEMBER_IMPORTED`, `MEMBER_TRAINER_ASSIGNED` |
| Plan | `PLAN_CREATED`, `PLAN_UPDATED`, `PLAN_DEACTIVATED` |
| Subscription | `SUBSCRIPTION_CREATED`, `SUBSCRIPTION_ACTIVATED`, `SUBSCRIPTION_RENEWED`, `SUBSCRIPTION_FROZEN`, `SUBSCRIPTION_UNFROZEN`, `SUBSCRIPTION_CANCELLED`, `SUBSCRIPTION_EXPIRED` |
| Payment | `PAYMENT_RECORDED`, `PAYMENT_REVERSED` |
| Check-in | `CHECKIN_RECORDED`, `CHECKIN_OVERRIDDEN` |
| Notification | `MESSAGE_SENT`, `MESSAGE_CANCELLED` |

---

## 8. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `AuditLog`, `AuditCommand`, القواعد. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Query API. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 9. Definition of Done

- [ ] Migration `audit_log` + RLS + Immutability Trigger.
- [ ] `AuditLog` Entity.
- [ ] `AuditService.record(...)`.
- [ ] `AuditContext` (IP / User Agent).
- [ ] كل الـ Features بتسجل عملياتها.
- [ ] `GET /audit-logs` + فلاتر.
- [ ] ممنوع UPDATE/DELETE على مستوى DB وApp.
- [ ] مفيش PII في Logs.
- [ ] Immutability + Isolation Tests.

---

## 10. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل Audit failure يفشل العملية؟ | نعم للعمليات المالية/الحساسة. |
| هل Login Failed يتسجل؟ | نعم — Best Effort في Transaction منفصلة. |
| مدة الاحتفاظ؟ | 5 سنوات — تتأكد قانونياً. |
| هل Manager يشوف Audit فرعه؟ | لا في الـ MVP. |
