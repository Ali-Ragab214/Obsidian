# تتبع المتطلبات (Requirements Traceability)

> **المرجع:** [`../../Kammel_Project_Documentation/01_BUSINESS_REQUIREMENTS.md`](../../Kammel_Project_Documentation/01_BUSINESS_REQUIREMENTS.md)  
> **الهدف:** التأكد إن كل Business Requirement (BR) له Feature مالكة، وTask تنفيذي، وTest يثبته.

---

## 1. إزاي نقرأ الجدول

- **BR** = Business Rule من الوثيقة الأصلية.
- **Owner Feature** = الميزة المسؤولة عن تنفيذ القاعدة.
- **Enforcement** = المكان اللي القاعدة بتتنفذ فيه (Domain / Service / DB / API).
- **Verified By** = نوع الاختبار اللي يثبتها.
- **Status** = `Planned` في الخطة، ويتحول لـ `Done` في [master-checklist](../development-roadmap/master-checklist.md).

> أي BR مش مغطى أو متعارض، متسجل في [decisions-and-assumptions](./decisions-and-assumptions.md).

---

## 2. متطلبات الإعداد والـ Tenancy

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-001 | كل Organization لازم يكون عندها Branch واحد على الأقل | `tenancy` | Service + DB Check | Service Test + Integration |
| BR-002 | Plan لازم يكون له Name وPrice > 0 وDuration | `plan` | DTO Validation + DB CHECK | Unit + API Test |
| BR-003 | Excel Import يكتشف تكرار الموبايل داخل نفس الـ Org | `member` | Import Validator + DB Unique | Import Integration Test |
| BR-004 | Dashboard Org-scoped والـ Owner يشوف كل الفروع | `reporting` | TenantContext + Role | Tenant Isolation + API Test |

---

## 3. الأعضاء (Members)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-011 | الموبايل هو مفتاح البحث الأساسي | `member` | Index `(org_id, phone)` | Repository Test |
| BR-012 | `(org_id, phone)` فريد | `member` | Partial Unique Index | DB + Service Test |
| BR-013 | WhatsApp Opt-in لازم يتسجل صريح وقت التسجيل | `member` | `whatsapp_opt_in` NOT NULL | API Test |

---

## 4. الحضور (Check-in)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-014 | ممنوع إحراج العضو أو رفضه بشكل قاسي | `checkin` | Response Status + UI Contract | API Test |
| BR-015 | كل Check-in يسجل Time + Branch + Method | `checkin` | Entity NOT NULL | Entity + API Test |
| BR-016 | الـ Override يحتاج Reason + Audit | `checkin` + `audit` | Service + Audit Log | Integration Test |
| BR-017 | Check-in هو أساس Risk Score | `risk` | Query Contract | Risk Service Test |
| BR-018 | التكرار في نفس اليوم Warning مش Block | `checkin` | Service Rule | Service Test |

---

## 5. المدفوعات (Payments)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-006 | كل Discount/Freeze/Refund/Payment Mod متسجل في Audit | `audit` | Audit Service | Integration Test |
| BR-007 | Audit Log Immutable | `audit` | DB Trigger + Revoke Privileges | DB Test |
| BR-010 | Refund قيد عكسي — ممنوع حذف الدفع | `payment` (P1) | Entity + Service | Service Test |
| BR-019 | المبالغ `Long` بالقروش — ممنوع Float/Double | `payment` + `shared` | Type System + Code Review | Arch Test + Unit |
| BR-020 | Idempotency Key يمنع تكرار الدفع | `payment` | Unique `(org_id, idempotency_key)` | Integration Test |
| BR-021 | المدفوعات لا تُحذف أبداً | `payment` | Revoke DELETE + No Delete API | DB Test |
| BR-022 | المدفوعات الجزئية مدعومة | `payment` | Remaining Balance Query | Service Test |
| BR-023 | أرقام الإيصالات متسلسلة لكل Org | `payment` | `document_sequences` + Row Lock | Concurrency Test |
| BR-024 | الدفع الكاش مرتبط بوردية مفتوحة | `payment` + `shift` (P1) | Service Validation | P1 Integration |

---

## 6. الاشتراكات (Subscriptions)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-025 | السعر Snapshot وقت الإنشاء | `subscription` | `price_minor_snapshot` Immutable | Service Test |
| BR-026 | التجديد قبل الانتهاء يمتد من `end_date` | `subscription` | Domain Formula | Unit Test |
| BR-027 | التجديد بعد الانتهاء يبدأ من Payment Date | `subscription` | Domain Formula + New Record | Unit + Integration |
| BR-028 | كل تغيير حالة يسجل `SubscriptionEvent` | `subscription` | Service + Event Listener | Service + DB Test |
| BR-005 | Risk List تعرض Score + Reason + Action + Draft | `risk` | API Contract | API Test |

---

## 7. الأدوار والصلاحيات

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-008 | Manager يشتغل على فرعه فقط | `identity` + `tenancy` | `@PreAuthorize` + BranchSecurity | Security Test |
| BR-009 | Manager ممنوع يشوف فرع تاني حتى في نفس الـ Org | `identity` + `tenancy` | BranchSecurity + Query Filter | Isolation Test |
| BR-037 | Trainer يشوف عملاءه فقط | `identity` + `member` | `assigned_trainer_id` + Query | Security Test |
| BR-038 | Trainer ممنوع يشوف بيانات مالية | `identity` | Role Matrix | Security Test |

---

## 8. الورديات (P1)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-029 | وردية مفتوحة واحدة لكل User | `shift` | Partial Unique Index | P1 DB Test |
| BR-030 | لازم يقفل الوردية قبل فتح جديدة | `shift` | Service Rule | P1 Service Test |
| BR-031 | Variance يظهر في Dashboard | `shift` + `reporting` | Query Contract | P1 Integration |
| BR-032 | الوردية Immutable بعد الإغلاق | `shift` | DB Trigger / Revoke Update | P1 DB Test |

---

## 9. الإشعارات (Notifications)

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-033 | فتح `wa.me` ≠ إرسال — لازم تأكيد | `notification` | Status `OPENED` → `SENT_CONFIRMED` | Service Test |
| BR-034 | رسالة واحدة لكل Template لكل عضو كل 3 أيام | `notification` | Cooldown Query | Service Test |
| BR-035 | الإرسال 9ص–9م بتوقيت القاهرة | `notification` | Service Time Window | Unit Test |
| BR-036 | الرسائل للـ Opt-in فقط | `notification` | `whatsapp_opt_in = true` | Service Test |

---

## 10. متطلبات غير وظيفية (NFR)

| NFR | الهدف | Feature/Owner | Verified By |
|---|---|---|---|
| Tenant Isolation | صفر تسريب | `tenancy` + `shared/security` | Tenant Isolation Suite |
| Check-in Latency | < 1s (هدف < 500ms) | `checkin` | Performance Test |
| Financial Precision | `Long` Piasters | `payment` + `shared` | Arch Test |
| Audit Immutability | ممنوع UPDATE/DELETE | `audit` | DB Trigger Test |
| Arabic RTL | دعم كامل | Frontend | Manual QA |
| Backup | يومي مشفر + Restore | DevOps | Restore Runbook |
| Privacy | Law #151/2020 | `security-strategy` | Legal Review + Checklist |

---

## 11. الـ BRs اللي محتاجة تأكيد (Assumed)

| BR | السبب | القرار المؤقت |
|---|---|---|
| BR-029 / BR-030 / BR-032 | مش موجودة صريحة في المصدر | نعتمدها كقواعد ورديات طبيعية — P1 |
| BR-034 | الرقم 3 أيام افتراض | يتظبط بعد الـ Pilot |
| BR-035 | وقت الإرسال افتراض | 9ص–9م بتوقيت القاهرة |

---

## 12. قواعد إضافية ناتجة عن تحليل الوثائق

القواعد دي **مش موجودة كـ BR صريح**، لكنها لازمة لتنفيذ النظام بشكل صحيح. متسجلة هنا ومربوطة بـ Assumption:

| ID | القاعدة | السبب | المرجع |
|---|---|---|---|
| TR-01 | `risk_scores` و`subscription_events` لازم يكون فيهم `org_id` | RLS policy بتستخدم `org_id` والجدولين في الوثيقة من غير العمود | [ASM-08](./decisions-and-assumptions.md#asm-08) |
| TR-02 | `payments.idempotency_key` فريد على `(org_id, key)` مش Globally | منع تعارض بين جيمات | [ASM-06](./decisions-and-assumptions.md#asm-06) |
| TR-03 | `member_code` و`qr_token` لازم يتضافوا للـ Member | البحث بالكود وQR الثابت | [ASM-05](./decisions-and-assumptions.md#asm-05) |
| TR-04 | `assigned_trainer_id` لازم يتضاف للـ Member | Trainer يشوف عملاءه فقط | [ASM-13](./decisions-and-assumptions.md#asm-13) |
| TR-05 | `frozen_at` و`scheduled_unfreeze_date` لازم يتضافوا | حساب أيام التجميد الفعلية | [ASM-03](./decisions-and-assumptions.md#asm-03) |
| TR-06 | `refresh_tokens` table مطلوبة | Refresh Token Rotation | [ASM-15](./decisions-and-assumptions.md#asm-15) |
| TR-07 | `document_sequences` table مطلوبة | Receipt Numbering المتسلسل | [ASM-06](./decisions-and-assumptions.md#asm-06) |
| TR-08 | `plan.grants_access` مطلوب | التفرقة بين Membership وPT Package | [ASM-16](./decisions-and-assumptions.md#asm-16) |
| TR-09 | `app.current_member` + Restrictive RLS مطلوبين للبوابة | عزل الأعضاء داخل نفس الـ Org | MP-10 |
| TR-10 | `message_queue.is_transactional` مطلوب | Magic Link بيتجاوز Cooldown/Window | MP-05 |

---

## 12.1 متطلبات P1 — Member Portal

| BR | القاعدة | Owner Feature | Enforcement | Verified By |
|---|---|---|---|---|
| BR-039 | العضو يشوف بياناته هو فقط | `member-portal` | JWT + `app.current_member` + RLS | Isolation Test |
| BR-040 | مفيش كلمة مرور للعضو — Magic Link فقط | `member-portal` | Auth Flow | Integration Test |
| BR-041 | البوابة Read-only ما عدا طلب التجديد | `member-portal` | API Scope + No Write Contracts | Arch + API Test |

---

## 13. مراجعة التغطية

| المجال | عدد الـ BRs | مغطى | ملاحظات |
|---|---|---|---|
| Setup & Tenancy | 4 | 4 | — |
| Members | 3 | 3 | — |
| Check-in | 5 | 5 | — |
| Payments | 6 | 6 | Refund/Shift P1 |
| Subscriptions | 5 | 5 | — |
| Roles | 4 | 4 | — |
| Shifts | 4 | 4 | P1 |
| Notifications | 4 | 4 | — |
| Member Portal | 3 | 3 | P1 |
| **الإجمالي** | **41** | **41** | 3 Assumed + 7 P1 |

> **نتيجة المراجعة:** كل BR في الوثيقة الأصلية له Owner وEnforcement وTest. الـ BRs الافتراضية متسجلة بوضوح، والـ P1 متأخرة بشكل مقصود.
