# Risk — تقييم خطر التسريب

> **الـ Phase:** 3 · **الأولوية:** P0 · **الترتيب التنفيذي:** 9  
> **الـ Package:** `com.kammel.features.risk`  
> **المرجع:** [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) · [`../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md`](../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md)

---

## 1. الميزة دي بتعمل إيه؟

`risk` هي **محرك الإنذار المبكر**:

- حساب **Risk Score من 0 لـ 100** لكل عضو نشط كل ليلة.
- Score **Rule-Based ومفسّر** — مش ML.
- تخزين **الأسباب** (`reasons`) والقيم الخام (`signals`).
- تصنيف: `LOW` / `MEDIUM` / `HIGH` / `CRITICAL` / `INSUFFICIENT_DATA`.
- تحديد **الإجراء المقترح** لكل مستوى.
- توليد **Draft Message** عربي جاهز للإرسال.
- قائمة At-Risk للـ Owner/Manager.
- ملخص توزيع المخاطر.

**مش مسؤولة عن:**
- إرسال الرسائل → `notification`.
- حساب الحضور → `checkin`.
- الاشتراكات → `subscription`.
- الـ Dashboard → `reporting`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `RiskScore` | مجموع أوزان الإشارات — 0..100. |
| `Signal` | إشارة سلوكية (غياب، انخفاض حضور، قرب انتهاء...). |
| `Weight` | وزن الإشارة في الـ Score. |
| `reasons` | أسباب عربية مفهومة — JSONB Array. |
| `signals` | القيم الخام + النقاط — JSONB Object. |
| `risk_level` | التصنيف المخزّن للفلترة. |
| `INSUFFICIENT_DATA` | أقل من 14 يوم بيانات — مفيش Score. |
| `draft_message` | رسالة عربية مقترحة للـ WhatsApp. |

---

## 3. النطاق

### داخل الـ MVP
- [x] 6 إشارات بأوزان ثابتة.
- [x] Score + Level.
- [x] `reasons` + `signals` + `model_version`.
- [x] `recommended_action`.
- [x] `draft_message`.
- [x] Nightly Job 3:00 AM Cairo.
- [x] `INSUFFICIENT_DATA`.
- [x] At-Risk List + Filters.
- [x] Member Latest Score.
- [x] Summary.
- [x] Manual Recompute (Owner).

### خارج الـ MVP
- [ ] ML Model.
- [ ] Holdout A/B (P1).
- [ ] Per-gym weight tuning.
- [ ] Gym closure awareness (ASM-18).
- [ ] Real-time scoring.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| At-Risk List | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Member Score | ✅ | ✅ (فرعه) | ❌ | ✅ (عميله فقط) |
| Summary | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Manual Recompute | ✅ | ❌ | ❌ | ❌ |
| Draft Message | ✅ | ✅ | ❌ | ❌ |

> **Receptionist** بيشوف الرسائل في `notification`، مش الـ Risk Score الخام.

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-005 | قائمة At-Risk تعرض Score + Reason + Action + Draft. |
| BR-017 | Check-in هو أساس Risk Score. |
| ASM-17 | أقل من 14 يوم بيانات → `INSUFFICIENT_DATA`. |
| RSK-01 | Score دايمًا 0..100 — `min(100, Σ points)`. |
| RSK-02 | `risk_level` مشتق من الـ Score ومخزّن. |
| RSK-03 | كل Score له `reasons` عربية. |
| RSK-04 | كل Score له `model_version`. |
| RSK-05 | Job Idempotent — نفس اليوم = نفس النتيجة. |
| RSK-06 | Score مش بيتعدل يدوي. |
| RSK-07 | `INSUFFICIENT_DATA` → `score = NULL`. |
| RSK-08 | At-Risk = `HIGH` + `CRITICAL` فقط. |
| RSK-09 | Branch من العضو وقت الحساب. |
| RSK-10 | ممنوع Score لعضو محذوف/غير نشط. |
| RSK-11 | Draft Message مش بتتبعت تلقائياً. |
| RSK-12 | Weights ثابتة في `rules-v1`. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `Clock`, `TenantTransactionRunner` |
| `tenancy` | Compile | `OrganizationQueryService` (Job) |
| `member` | Read | `MemberQueryService` |
| `subscription` | Read | `SubscriptionQueryService` |
| `checkin` | Read | `CheckInQueryService` |
| `audit` | Compile | `AuditService.recordSystem` (اختياري) |

**مين بيعتمد على `risk`؟**
`notification` (Read Contract)، `reporting` (Read Contract).

> **ملاحظة:** حساب **Payment Friction** بيستخدم `SubscriptionQueryService.countDelayedActivations(...)` كـ Proxy — مش `payment` مباشرة. التفاصيل في [`domain-design.md`](./domain-design.md#7-الإشارات-والتقييم).

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | Signals + Formula + Levels + Draft. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `risk_scores` + RLS + `branch_id`.
- [ ] `RiskScore` Entity + Composite Key.
- [ ] `RiskScoringService` + `rules-v1`.
- [ ] 6 إشارات + أوزان.
- [ ] `INSUFFICIENT_DATA`.
- [ ] Nightly Job + ShedLock + Tenant Scope.
- [ ] `RiskQueryService`.
- [ ] At-Risk API + Summary + Recompute.
- [ ] Draft Message.
- [ ] Tests + Performance.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل نستخدم ML؟ | لا — `rules-v1` في MVP. |
| هل نحسب Payment Friction من `payment`؟ | لا — Proxy من `subscription_events` (P1 يتحسن). |
| هل Holdout A/B في MVP؟ | لا — P1. |
| هل الـ Score يتحدث Real-time؟ | لا — Nightly فقط + Recompute يدوي. |
| هل Trainer يشوف كل الأعضاء؟ | لا — عميله فقط (`assigned_trainer_id`). |
| هل نحتفظ بكل Scores؟ | نعم — 24 شهر ثم Purge اختياري. |
