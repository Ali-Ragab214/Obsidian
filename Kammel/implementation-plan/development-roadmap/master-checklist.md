# الـ Master Checklist — متابعة التنفيذ

> **الاستخدام:** علّم على المهمة بعد ما تتأكد إنها **خلصت ومختبرة** — مش بعد ما الكود اتكتب.  
> **المرتبط:** [implementation-phases](./implementation-phases.md) · [integration-roadmap](./integration-roadmap.md) · [README](../README.md)

---

## 0. قبل ما تبدأ

- [ ] Java 21 + Maven + Docker مثبتين.
- [ ] PostgreSQL 16 شغال محلياً أو عبر Docker.
- [ ] `JWT_SECRET` متولد (256-bit) ومحفوظ في `.env` — **مش** في Git.
- [ ] Repo فيه `.gitignore` + `.editorconfig` + `README`.
- [ ] Profiles: `dev`, `test`, `staging`, `prod`.
- [ ] OpenAPI + Postman Collection.
- [ ] CI Pipeline يفشل لو Test أو Architecture فشلت.

---

## 1. Phase 0 — التحقق

- [ ] 5–7 مقابلات أصحاب جيمات.
- [ ] Baseline Metrics (نسبة التجديد، طرق التحصيل).
- [ ] Prototype اتعرض.
- [ ] 2 LOI موقعة.
- [ ] Assumptions ASM-01..ASM-27 اتراجعت.
- [ ] نموذج التسعير اتحدد.
- [ ] Pilot Data جاهزة.
- [ ] قرار Go / No-Go موثق.
- [ ] **CP0 معتمد.**

---

## 2. Phase 1 — الأساس

### 2.1 Shared Foundation

- [ ] `V1__create_extensions_and_helpers.sql` (pgcrypto + RLS Helpers + `touch_updated_at` + `forbid_mutation`).
- [ ] `pom.xml` + Spring Boot 3 + Java 21.
- [ ] `BaseEntity` + `BaseAuditableEntity`.
- [ ] `TenantScoped` Interface.
- [ ] `ApiResponse<T>` + `PageResponse<T>` + `ErrorResponse`.
- [ ] `BusinessException` + `GlobalExceptionHandler`.
- [ ] `MoneyUtil` (Long Minor) + `PhoneUtil` + `DateUtil` (Cairo).
- [ ] `@ValidPhone`.
- [ ] `TenantContext` + `TenantContextCleanupFilter`.
- [ ] `JwtService` + `UserPrincipal` + `JwtAuthenticationFilter`.
- [ ] `TenantAwareJpaTransactionManager` + `RlsSessionInitializer`.
- [ ] `TenantTransactionRunner`.
- [ ] `SecurityConfig` + `OpenApiConfig` + `JacksonConfig`.
- [ ] `shared/contracts/events` Package.
- [ ] `V16__create_shedlock.sql` + `V17__create_indexes.sql` + `V18__create_sequences_and_constraints.sql`.
- [ ] Shared Foundation Tests.
- [ ] **Checkpoint: مفيش Feature تبدأ قبل ده.**

### 2.2 Tenancy

- [ ] `V2__create_organizations.sql` + `V3__create_branches.sql`.
- [ ] RLS Helper Functions.
- [ ] `Organization` + `Branch` Entities.
- [ ] Organization CRUD (Owner).
- [ ] Branch CRUD (Owner).
- [ ] Slug Validation + Uniqueness.
- [ ] Branch Scope.
- [ ] Provisioning CLI.
- [ ] Seed Data.
- [ ] Isolation Tests.
- [ ] **Feature DoD مكتمل** — [tenancy](../features/tenancy/README.md).

### 2.3 Identity

- [ ] `V4__create_users.sql` + `V5__create_refresh_tokens.sql`.
- [ ] `User` + `RefreshToken`.
- [ ] Login بـ `organizationSlug` + `username`.
- [ ] Access Token (15m) + Refresh Token (7d).
- [ ] Refresh Rotation + Reuse Detection.
- [ ] Logout + Revocation.
- [ ] `GET /auth/me`.
- [ ] Change Password.
- [ ] Owner User Management.
- [ ] Disable User + Revoke Tokens.
- [ ] Reset Password.
- [ ] Account Lockout (5/15m).
- [ ] `mustChangePassword`.
- [ ] Rate Limit.
- [ ] Identity Tests + Isolation.
- [ ] **Feature DoD مكتمل** — [identity](../features/identity/README.md).

### 2.4 Audit

- [ ] `V15__create_audit_log.sql`.
- [ ] `AuditLog` Entity + Immutability Trigger.
- [ ] `AuditService.record(...)`.
- [ ] ربطه بكل العمليات الحساسة في Phase 1.
- [ ] Audit Query API (Owner فقط).
- [ ] Pagination + Filters.
- [ ] Audit Tests.
- [ ] **Feature DoD مكتمل** — [audit](../features/audit/README.md).

### 2.5 Phase 1 Hardening

- [ ] Security Headers + CORS.
- [ ] Tenant Isolation Suite كامل.
- [ ] Concurrency Tests (Login / Refresh).
- [ ] OpenAPI محدث.
- [ ] Error Catalog.
- [ ] Staging Deploy.
- [ ] Backup/Restore Drill.
- [ ] **CP1 معتمد.**

---

## 3. Phase 2 — النواة التشغيلية

### 3.1 Member

- [ ] `V6__create_members.sql`.
- [ ] `Member` Entity + `member_code` + `qr_token`.
- [ ] Member Code Generation (`pg_advisory_xact_lock`).
- [ ] Create/Update/Search/Lookup.
- [ ] Soft Delete.
- [ ] Excel Import (Dry Run + Commit).
- [ ] Import Template.
- [ ] Import Error Report.
- [ ] Member Query Service.
- [ ] Member Tests + Isolation.
- [ ] **Feature DoD مكتمل** — [member](../features/member/README.md).

### 3.2 Plan

- [ ] `V7__create_plans.sql`.
- [ ] `Plan` Entity.
- [ ] CRUD + Deactivate.
- [ ] Price > 0 (BR-002 / SC-22).
- [ ] Plan Query Service.
- [ ] Plan Tests.
- [ ] **Feature DoD مكتمل** — [plan](../features/plan/README.md).

### 3.3 Subscription

- [ ] `V8__create_subscriptions.sql` + `V9__create_subscription_events.sql`.
- [ ] `Subscription` + `SubscriptionEvent`.
- [ ] State Machine.
- [ ] Create `PENDING_PAYMENT`.
- [ ] Price Snapshot.
- [ ] Renewal + `renewal_of` (SC-25).
- [ ] Freeze / Unfreeze.
- [ ] Cancel.
- [ ] Expiry Job.
- [ ] `EXPIRING_SOON` Computed.
- [ ] `MemberAccessChangedEvent`.
- [ ] `SubscriptionRenewedEvent` بعد التفعيل.
- [ ] `SubscriptionQueryService`.
- [ ] Subscription Tests + Concurrency.
- [ ] **Feature DoD مكتمل** — [subscription](../features/subscription/README.md).

### 3.4 Payment

- [ ] `V10__create_payments.sql`.
- [ ] `V11__create_document_sequences.sql` + `app.next_document_number()`.
- [ ] `Payment` Entity.
- [ ] Record Payment + Idempotency.
- [ ] Partial Payments.
- [ ] Overpayment Reject.
- [ ] Receipt PDF.
- [ ] Reversal.
- [ ] Collections Query.
- [ ] `PaymentRecordedEvent` + `totalPaidMinor`.
- [ ] Payment Tests + Concurrency.
- [ ] **Feature DoD مكتمل** — [payment](../features/payment/README.md).

### 3.5 Phase 2 Integration

- [ ] Payment → Subscription Activation.
- [ ] Renewal → New Record.
- [ ] Freeze → End Date Extension.
- [ ] Collections Queue.
- [ ] Member Timeline.
- [ ] E2E: عضو → اشتراك → دفع → تفعيل.
- [ ] Performance Baseline.
- [ ] **CP2 معتمد.**

---

## 4. Phase 3 — الاحتفاظ

### 4.1 Check-in

- [ ] `V12__create_check_ins.sql`.
- [ ] `CheckIn` Entity + Immutability.
- [ ] Phone/Name/Code Search.
- [ ] Static QR.
- [ ] Subscription Validation.
- [ ] Grace Handling.
- [ ] Frozen Block.
- [ ] Duplicate Warning.
- [ ] Override (Manager/Owner) + Reason.
- [ ] Session Consumption.
- [ ] Audit.
- [ ] Search Performance < 500ms.
- [ ] Check-in Tests + Isolation.
- [ ] **Feature DoD مكتمل** — [checkin](../features/checkin/README.md).

### 4.2 Risk

- [ ] `V14__create_risk_scores.sql` + `branch_id` (SC-23).
- [ ] `RiskScore` Entity.
- [ ] 6 Signals + Weights.
- [ ] `rules-v1`.
- [ ] Scoring Job 03:00 Cairo.
- [ ] ShedLock + Idempotent Upsert.
- [ ] `INSUFFICIENT_DATA` < 14 يوم.
- [ ] Reasons JSONB.
- [ ] Draft Message.
- [ ] Risk API.
- [ ] Risk Tests + Job Tests.
- [ ] **Feature DoD مكتمل** — [risk](../features/risk/README.md).

### 4.3 Notification

- [ ] `V13__create_message_queue.sql`.
- [ ] `MessageQueueItem` Entity.
- [ ] 3 قوالب شغالة (`RENEWAL_REMINDER`, `GRACE_EXPOSURE`, `MISS_YOU`) + `WELCOME`/`COLLECTION` معرّفين.
- [ ] `wa.me` Link + `opened_at` (SC-24/SC-27).
- [ ] Generation Job 06:00 Cairo.
- [ ] 3-Day Cooldown.
- [ ] Send Window 09:00–21:00.
- [ ] Opt-in Only.
- [ ] Status Lifecycle.
- [ ] Attribution 7 Days.
- [ ] Notification Tests.
- [ ] **Feature DoD مكتمل** — [notification](../features/notification/README.md).

### 4.4 Reporting

- [ ] `ReportingQueryRepository`.
- [ ] Owner Summary.
- [ ] Renewal Rate.
- [ ] Attributed Revenue.
- [ ] Collections.
- [ ] Attendance.
- [ ] Branch Comparison.
- [ ] Member Timeline.
- [ ] Branch Scope.
- [ ] Dashboard UI.
- [ ] Export.
- [ ] Reporting Tests + Isolation.
- [ ] **Feature DoD مكتمل** — [reporting](../features/reporting/README.md).

### 4.5 Phase 3 Integration

- [ ] Check-in → Risk Signals.
- [ ] Risk → Notification.
- [ ] Renewal → Attribution.
- [ ] Reporting → كل الـ KPIs.
- [ ] E2E: غياب → Risk → رسالة → تجديد → Attribution.
- [ ] **CP3 معتمد.**

---

## 5. Phase 4 — Pilot-Ready

### 5.1 Security

- [ ] Security Review (OWASP Top 10).
- [ ] Dependency Scan (CVEs).
- [ ] Secrets Audit — مفيش Secret في Git.
- [ ] Rate Limiting على Login + Portal.
- [ ] File Upload Limits.
- [ ] مفيش PII في Logs.
- [ ] مفيش `permitAll` غير مبرر.
- [ ] RLS على كل الجداول.
- [ ] مفيش Superuser للـ App.

### 5.2 Performance

- [ ] Load Test: 350 عضو.
- [ ] Check-in p95 < 500ms.
- [ ] Dashboard p95 < 1s.
- [ ] Index Review.
- [ ] Connection Pool.
- [ ] Query Plan Review للـ Reporting.

### 5.3 Operations

- [ ] Docker Image + Compose.
- [ ] Staging + Production.
- [ ] HTTPS + Domain.
- [ ] Backup يومي + Retention.
- [ ] Restore Drill.
- [ ] Monitoring + Alerts.
- [ ] Log Aggregation.
- [ ] Runbook (DB Down / Disk Full / Job Failed).
- [ ] Rollback Plan.

### 5.4 Pilot

- [ ] بيانات الجيم مستوردة ومطابقة.
- [ ] تدريب الاستقبال.
- [ ] دليل استخدام عربي.
- [ ] خطة دعم أول أسبوع.
- [ ] **CP4 معتمد — Go/No-Go.**

---

## 6. Cross-Cutting — طول المشروع

### 6.1 Database

- [ ] كل Migration بتشتغل من DB نضيفة.
- [ ] كل Migration ليها Rollback Plan.
- [ ] RLS Helper Functions ثابتة.
- [ ] Immutability Triggers.
- [ ] Indexes على كل Foreign Keys.
- [ ] Partial Unique Indexes.
- [ ] `document_sequences` Atomic.
- [ ] Backup/Restore.

### 6.2 Security

- [ ] `orgId` ممنوع من الـ Request.
- [ ] Branch Scope.
- [ ] `@PreAuthorize`.
- [ ] 404 بدل 403 للـ Cross-Tenant.
- [ ] Refresh Rotation.
- [ ] Account Lockout.
- [ ] Audit للعمليات الحساسة.
- [ ] Secrets من Environment.

### 6.3 Testing

- [ ] Unit Tests لكل Domain Rule.
- [ ] Integration Tests لكل Endpoint.
- [ ] Isolation Suite 100%.
- [ ] Concurrency Tests للمسارات الحرجة.
- [ ] Testcontainers PostgreSQL 16 — ممنوع H2.
- [ ] Architecture Boundary Tests.
- [ ] E2E للـ Critical Flows.
- [ ] Load Test قبل الـ Pilot.

### 6.4 API

- [ ] `ApiResponse` Envelope موحد.
- [ ] Error Codes ثابتة.
- [ ] Pagination موحدة.
- [ ] Validation Messages.
- [ ] OpenAPI محدث.
- [ ] DTOs — مفيش Entity.
- [ ] Idempotency للـ Payments.

### 6.5 Documentation

- [ ] كل Feature `README` محدث.
- [ ] Master Checklist محدثة.
- [ ] Decisions & Assumptions محدثة.
- [ ] Runbook.
- [ ] دليل المستخدم.

---

## 7. P1 — بعد نجاح الـ Pilot

### 7.1 Shift

- [ ] `V19__create_cash_shifts.sql` + Partial Unique.
- [ ] `V20__add_payments_shift_fk.sql`.
- [ ] `V21__create_refunds.sql` + Reversal Flow.
- [ ] Open / Close / Force Close.
- [ ] Blind Count.
- [ ] Variance.
- [ ] Cash Payment Link.
- [ ] Immutability.
- [ ] Audit.
- [ ] Tests.
- [ ] **DoD** — [shift](../features/shift/README.md).

### 7.2 Lead

- [ ] `V22__create_leads.sql` (يضم `leads` + `lead_followups`).
- [ ] Public Form + QR.
- [ ] Funnel States.
- [ ] Conversion to Member.
- [ ] Source Analytics.
- [ ] Tests.
- [ ] **DoD** — [lead](../features/lead/README.md).

### 7.3 Member Portal

- [ ] `V23__create_member_portal_tokens.sql`.
- [ ] `V24__create_portal_renewal_requests.sql`.
- [ ] `V25__add_organization_branding.sql`.
- [ ] Magic Link Flow.
- [ ] Portal JWT Scope.
- [ ] Member Isolation (GUC + RLS).
- [ ] `/portal/me`, `/portal/subscription`, `/portal/qr`.
- [ ] `/portal/checkins` + Streak.
- [ ] `/portal/payments` + Receipt.
- [ ] Renewal Requests + Staff Queue.
- [ ] Branding + PWA.
- [ ] Tests + Isolation.
- [ ] **DoD** — [member-portal](../features/member-portal/README.md).

### 7.4 WhatsApp Cloud API

- [ ] `NotificationChannel` Adapter.
- [ ] Template Approval.
- [ ] Opt-in Management.
- [ ] Delivery Webhooks.
- [ ] Fallback `wa.me`.
- [ ] Cost Monitoring.

### 7.5 AI

- [ ] 3 شهور بيانات.
- [ ] Holdout Experiment.
- [ ] ML Model (ONNX / FastAPI).
- [ ] مقارنة بالقواعد.
- [ ] Owner Assistant.
- [ ] **DoD** — [ai-overview](../ai/ai-overview.md).

---

## 8. Pilot — 14 يوم

- [ ] يوم 1–3: مرافقة يومية.
- [ ] يوم 4–7: قياس المؤشرات.
- [ ] يوم 8–10: مقابلات.
- [ ] يوم 11–14: تثبيت.
- [ ] ≥ 70% Check-in Coverage.
- [ ] تجديد واحد على الأقل مربوط برسالة.
- [ ] Owner بيستخدم Dashboard أسبوعياً.
- [ ] NPS ≥ 8.
- [ ] تقرير نتائج مكتوب.
- [ ] قرار الاستمرار / التعديل.

---

## 9. Change Log

| التاريخ | التغيير | السبب | بواسطة |
|---|---|---|---|
| — | — | — | — |

---

## 10. مرتبط بـ

- [implementation-phases](./implementation-phases.md)
- [integration-roadmap](./integration-roadmap.md)
- [README](../README.md)
- [testing-strategy](../testing-strategy.md)
- [database-strategy](../architecture/database-strategy.md)
- [security-strategy](../architecture/security-strategy.md)
