# Lead — قمع المبيعات (P1)

> **الـ Phase:** Post-MVP · **الأولوية:** P1 · **الـ Package:** `com.kammel.features.lead`  
> **المرجع:** [`../../../Kammel_Project_Documentation/02_MVP_AND_PRODUCT_ROADMAP.md`](../../../Kammel_Project_Documentation/02_MVP_AND_PRODUCT_ROADMAP.md)

---

## 1. الميزة دي بتعمل إيه؟

`lead` بتحوّل **اهتمام** لدخول الجيم إلى **عضو مدفوع**:

- تسجيل Leads من Instagram / Facebook / Walk-in / Referral / WhatsApp.
- متابعة الحالة: `NEW → CONTACTED → TRIAL → NEGOTIATING → WON / LOST`.
- تحديد موظف مسؤول (`assigned_to`).
- تسجيل متابعات (Follow-ups) ومواعيد تالية.
- تحويل الـ Lead لعضو (`Member`) واشتراك اختياري.
- قمع مبيعات (Funnel) للـ Owner/Manager.

**مش مسؤولة عن:**
- إنشاء العضو نفسه → `member`.
- إنشاء الاشتراك → `subscription`.
- تسجيل الدفع → `payment`.
- رسائل الـ WhatsApp → `notification`.

---

## 2. ليه P1؟

- الـ MVP بيركّز على **الاحتفاظ** بعضو موجود، مش **استقطاب** عضو جديد.
- أول 14 يوم Pilot محتاجينهم يثبتوا إن النظام بيشتغل على العضو الحالي.
- الـ Lead Funnel بيدي قيمة واضحة لصاحب الجيم، لكنه **مش شرط** للـ Retention Loop.
- لو الـ Pilot طلب Leads → يتقدم P1 مبكر.

---

## 3. المصطلحات

| المصطلح | المعنى |
|---|---|
| `Lead` | شخص مهتم لسه مش عضو. |
| `source` | مصدر الـ Lead. |
| `assigned_to` | الموظف المسؤول عن المتابعة. |
| `follow-up` | متابعة مسجلة. |
| `converted` | اتحول لعضو. |
| `lost_reason` | سبب الفقد. |
| `funnel` | توزيع الـ Leads على المراحل. |

---

## 4. القواعد

| ID | القاعدة |
|---|---|
| LED-01 | `(org_id, phone)` فريد للـ Leads النشطة — ممنوع تكرار. |
| LED-02 | `phone` بصيغة مصرية صحيحة. |
| LED-03 | `WON`/`LOST` حالات نهائية. |
| LED-04 | التحويل بينشئ `Member` — مش نسخة. |
| LED-05 | ممنوع تحويل Lead مرتين. |
| LED-06 | كل تغيير حالة له Audit. |
| LED-07 | Manager/Receptionist → فرعهم فقط. |
| LED-08 | `lost_reason` إلزامي عند `LOST`. |
| LED-09 | `next_follow_up_at` لا يكون في الماضي عند الحفظ. |
| LED-10 | ممنوع حذف Lead — `LOST` بدل الحذف. |

---

## 5. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| إنشاء Lead | ✅ | ✅ | ✅ | ❌ |
| تعديل Lead | ✅ | ✅ | ✅ (فرعه) | ❌ |
| تغيير الحالة | ✅ | ✅ | ✅ (فرعه) | ❌ |
| تحويل لعضو | ✅ | ✅ | ✅ (فرعه) | ❌ |
| Funnel | ✅ | ✅ (فرعه) | ❌ | ❌ |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `PhoneUtil`, `TenantContext` |
| `tenancy` | Compile | `BranchQueryService` |
| `identity` | Compile | `UserQueryService` |
| `member` | Compile | `MemberService.create` |
| `subscription` | Compile | `SubscriptionService.create` (اختياري) |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `lead`؟**
`reporting` (Funnel Report) — P1.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`implementation-plan.md`](./implementation-plan.md) | Domain + API + Testing + Tasks. |

> **ملاحظة:** لأن `lead` P1، الوثائق consolidated في ملفين بدل خمسة.

---

## 8. Definition of Done

- [ ] `V22__create_leads.sql` (يضم `leads` + `lead_followups`) + RLS.
- [ ] `Lead` + `LeadFollowUp` Entities.
- [ ] State Machine.
- [ ] Follow-ups.
- [ ] Conversion لـ Member + Subscription.
- [ ] Funnel.
- [ ] Audit.
- [ ] Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل الـ Lead بيتمسح؟ | لا — `LOST`. |
| هل التحويل بيعمل اشتراك تلقائي؟ | اختياري — `planId` في الطلب. |
| هل فيه WhatsApp Automation للـ Leads؟ | لا — P2. |
| هل فيه Trial Pass؟ | نعم — `TRIAL` Status + `DAY_PASS` Plan اختياري. |
| هل الـ Lead بيظهر في Member Search؟ | لا — جدول منفصل. |
