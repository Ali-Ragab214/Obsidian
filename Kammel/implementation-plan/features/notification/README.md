# Notification — طابور رسائل WhatsApp

> **الـ Phase:** 3 · **الأولوية:** P0 · **الترتيب التنفيذي:** 10  
> **الـ Package:** `com.kammel.features.notification`  
> **المرجع:** [`../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md`](../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md) · [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md)

---

## 1. الميزة دي بتعمل إيه؟

`notification` هي **قناة التواصل اليدوية** مع العضو:

- Job كل يوم 6:00 صباحاً يبني **طابور رسائل** للاستقبال.
- 3 قوالب أساسية: `RENEWAL_REMINDER` / `MISS_YOU` / `GRACE_EXPOSURE`.
- توليد **`wa.me` Deep Link** — مفيش إرسال تلقائي في الـ MVP.
- الاستقبال بيفتح واتساب، يبعت، وبعدين **يأكد الإرسال** في كمّل.
- منع الـ Spam: رسالة واحدة لكل عضو كل 3 أيام.
- احترام **وقت الإرسال** 9 صباحاً – 9 مساءً بتوقيت القاهرة.
- **Opt-in فقط** — مفيش رسالة لعضو مش موافق.
- **Renewal Attribution** — ربط التجديد بالرسالة خلال 7 أيام.

**مش مسؤولة عن:**
- حساب الـ Risk → `risk` (بتقرأ الـ Draft).
- إرسال WhatsApp API الفعلي → **P1** (`CloudApiChannel`).
- الاشتراكات → `subscription`.
- التقارير → `reporting`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `MessageQueueItem` | رسالة في الطابور. |
| `template` | نوع الرسالة. |
| `rendered_text` | النص النهائي بعد استبدال المتغيرات. |
| `wa_me_link` | `https://wa.me/...?text=...` |
| `OPENED` | الاستقبال فتح الرابط — **مش إرسال**. |
| `SENT_CONFIRMED` | الاستقبال أكد إنه بعت. |
| `attributed_subscription_id` | التجديد اللي اتربط بالرسالة. |
| `cooldown` | 3 أيام بين الرسائل. |
| `opt-in` | موافقة العضو على WhatsApp. |
| `is_transactional` | رسالة بتتجاوز Cooldown/Window (Magic Link). |

---

## 3. النطاق

### داخل الـ MVP
- [x] `message_queue` + RLS.
- [x] Job 6:00 AM Cairo.
- [x] `RENEWAL_REMINDER` (T-7 / T-3 / T-1).
- [x] `GRACE_EXPOSURE`.
- [x] `MISS_YOU` من Risk.
- [x] `wa.me` Link.
- [x] Status Lifecycle: `PENDING → OPENED → SENT_CONFIRMED`.
- [x] Cooldown 3 أيام.
- [x] وقت الإرسال 9–21.
- [x] Opt-in.
- [x] Attribution 7 أيام.
- [x] Queue API + Confirm + Cancel.
- [x] Audit `MESSAGE_SENT` / `MESSAGE_CANCELLED`.

### خارج الـ MVP
- [ ] WhatsApp Cloud API (P1).
- [ ] Delivery/Read Webhooks (P1).
- [ ] `WELCOME` Template (P1).
- [ ] `COLLECTION` Template (P1 — محتاج Payment Read Contract).
- [ ] `PORTAL_MAGIC_LINK` Transactional (P1 — Member Portal).
- [ ] `is_transactional` Bypass (P1).
- [ ] SMS / Email.
- [ ] Message Scheduling / A/B Testing.
- [ ] Holdout Group (P1).

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| طابور اليوم | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |
| تفاصيل رسالة | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |
| فتح `wa.me` | ✅ | ✅ | ✅ | ❌ |
| تأكيد الإرسال | ✅ | ✅ | ✅ | ❌ |
| إلغاء رسالة | ✅ | ✅ | ❌ | ❌ |
| Attribution Report | ✅ | ❌ | ❌ | ❌ |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-033 | فتح `wa.me` ≠ إرسال — لازم تأكيد. |
| BR-034 | رسالة واحدة لكل Template لكل عضو كل 3 أيام. |
| BR-035 | الإرسال 9:00–21:00 بتوقيت القاهرة. |
| BR-036 | Opt-in فقط. |
| NOT-01 | ممنوع إرسال بدون `SENT_CONFIRMED` صريح. |
| NOT-02 | `rendered_text` غير قابل للتعديل بعد الإنشاء. |
| NOT-03 | ممنوع `DELETE` — الإلغاء Status. |
| NOT-04 | حد أقصى رسالة واحدة لكل عضو كل 3 أيام (كل القوالب). |
| NOT-05 | ممنوع إرسال لعضو محذوف/غير نشط. |
| NOT-06 | `branch_id` من العضو وقت الإنشاء. |
| NOT-07 | `wa.me` بيستخدم رقم E.164 مصري. |
| NOT-08 | Attribution خلال 7 أيام من `sent_at`. |
| NOT-09 | `is_transactional = true` بيتجاوز Cooldown + نافذة 09:00–21:00 (P1). |
| NOT-10 | `PORTAL_MAGIC_LINK` ممنوع يتأجل أو يتعمله Batch (P1). |
| NOT-09 | Job Idempotent — نفس اليوم = نفس الطابور. |
| NOT-10 | `SENT_CONFIRMED` حالة نهائية. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `TenantContext`, `PhoneUtil`, Events |
| `tenancy` | Compile | `OrganizationQueryService`, `BranchQueryService` |
| `identity` | Compile | `UserQueryService` |
| `member` | Read | `MemberQueryService` |
| `subscription` | Read + Event | `SubscriptionQueryService`, `SubscriptionRenewedEvent` |
| `risk` | Read | `RiskQueryService` (Draft + Level) |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `notification`؟**
`reporting` (Read Contract).

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `MessageQueueItem`, Lifecycle, Templates, Cooldown. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `message_queue` + RLS + `branch_id` + `opened_at`.
- [ ] `MessageQueueItem` + Enums.
- [ ] Job 6:00 AM + ShedLock + Tenant Scope.
- [ ] 3 قوالب.
- [ ] Cooldown 3 أيام.
- [ ] Time Window 9–21.
- [ ] Opt-in.
- [ ] Status Lifecycle + Audit.
- [ ] `wa.me` Link.
- [ ] Attribution.
- [ ] API + Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل نستخدم Cloud API؟ | لا — `wa.me` في MVP. |
| هل رسالة لكل Template ولا رسالة واحدة كل 3 أيام؟ | واحدة لكل عضو (NOT-04) — الأشد. |
| هل نولّد رسائل قبل 9 صباحاً؟ | نولّد 6 صباحاً، ونعرض 9 صباحاً. |
| هل نرسل لعضو Opt-out؟ | لا — Block + `422`. |
| هل الـ `WELCOME` شغال؟ | لا — P1. |
| هل `COLLECTION` شغال؟ | لا — محتاج Payment Read Contract. |
| هل نعيد المحاولة لو فشل الإرسال؟ | مفيش إرسال آلي — الاستقبال يعيد. |
