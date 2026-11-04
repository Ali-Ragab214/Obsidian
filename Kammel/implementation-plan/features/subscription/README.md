# Subscription — دورة حياة الاشتراك

> **الـ Phase:** 2 · **الأولوية:** P0 · **الترتيب التنفيذي:** 6  
> **الـ Package:** `com.kammel.features.subscription`  
> **المرجع:** [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md)

---

## 1. الميزة دي بتعمل إيه؟

`subscription` هي **قلب النظام**:

- إنشاء اشتراك من باقة (`PENDING_PAYMENT`).
- تفعيل تلقائي عند اكتمال الدفع.
- تجديد (Renewal) قبل/بعد الانتهاء.
- تجميد (Freeze) وفك تجميد (Unfreeze) مع تمديد المدة.
- Grace Period.
- انتهاء تلقائي (Scheduler).
- إلغاء (Cancel).
- خصم الجلسات (Session-based).
- تاريخ كامل عبر `SubscriptionEvent` غير القابل للتعديل.

**مش مسؤولة عن:**
- تسجيل الدفع نفسه → `payment`.
- التحقق من هوية العضو → `member`.
- Check-in → `checkin`.
- حساب الـ Risk → `risk`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `PENDING_PAYMENT` | اشتراك متسجل، مستني دفع. |
| `ACTIVE` | نشط وصالح. |
| `FROZEN` | مجمد — ممنوع دخول. |
| `GRACE` | فترة سماح بعد الانتهاء. |
| `EXPIRED` | انتهى. |
| `CANCELLED` | ملغي بقرار إداري. |
| `price_minor_snapshot` | سعر العقد وقت الإنشاء — ثابت. |
| `frozen_days_used` | مجموع أيام التجميد الفعلية. |
| `grace_ends_at` | نهاية الـ Grace. |

---

## 3. النطاق

### داخل الـ MVP
- [x] إنشاء اشتراك `PENDING_PAYMENT`.
- [x] تفعيل عند اكتمال الدفع.
- [x] تجديد قبل/بعد الانتهاء.
- [x] تجميد/فك تجميد + تمديد.
- [x] Grace Period.
- [x] انتهاء تلقائي.
- [x] إلغاء.
- [x] خصم جلسات.
- [x] `SubscriptionEvent` Immutable.
- [x] `EXPIRING_SOON` محسوب.
- [x] `SubscriptionQueryService`.
- [x] `MemberAccessChangedEvent`.

### خارج الـ MVP
- [ ] Refund (P1).
- [ ] Pause بدل Freeze.
- [ ] Upgrade/Downgrade mid-term.
- [ ] Retroactive Renewal (Manager override فقط).
- [ ] Auto-renewal بالبطاقة.
- [ ] Family/Corporate plans.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| إنشاء اشتراك | ✅ | ✅ | ✅ | ❌ |
| تجديد | ✅ | ✅ | ✅ | ❌ |
| تجميد / فك تجميد | ✅ | ✅ | ❌ | ❌ |
| إلغاء | ✅ | ✅ | ❌ | ❌ |
| Override Start Date | ✅ | ✅ | ❌ | ❌ |
| عرض الاشتراك | ✅ | ✅ | ✅ | ❌ |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-025 | السعر Snapshot وقت الإنشاء. |
| BR-026 | التجديد قبل الانتهاء يمتد من `end_date`. |
| BR-027 | التجديد بعد الانتهاء يبدأ من تاريخ الدفع. |
| BR-028 | كل تغيير حالة يسجل `SubscriptionEvent`. |
| SUB-01 | ممنوع تعديل `status` يدوي — State Machine فقط. |
| SUB-02 | `EXPIRING_SOON` محسوب — مش متخزن. |
| SUB-03 | `EXPIRED → ACTIVE` ممنوع. |
| SUB-04 | `CANCELLED → ACTIVE` ممنوع. |
| SUB-05 | التجميد من `ACTIVE` فقط. |
| SUB-06 | `frozen_days_used` لا يتجاوز `plan.freeze_max_days`. |
| SUB-07 | فك التجميد يمدد `end_date` بعدد الأيام الفعلية. |
| SUB-08 | كل تجديد ينشئ `Subscription` جديد (C-20). |
| SUB-09 | ممنوع حذف اشتراك. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `DateUtil`, Events |
| `tenancy` | Compile | `TenantContext` |
| `member` | Compile | `MemberQueryService` |
| `plan` | Compile | `PlanQueryService` |
| `audit` | Compile | `AuditService` |
| `payment` | Event | `PaymentRecordedEvent` |

**مين بيعتمد على `subscription`؟**
`payment`, `checkin`, `risk`, `notification`, `reporting`, `member` (Event).

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | State Machine + Formulas. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migrations `subscriptions` + `subscription_events` + RLS.
- [ ] `Subscription` + `SubscriptionEvent` Entities.
- [ ] State Machine كامل.
- [ ] إنشاء/تفعيل/تجديد/تجميد/فك/إلغاء.
- [ ] Scheduler للانتهاء والـ Grace والـ Auto-Unfreeze.
- [ ] Session Deduction.
- [ ] `SubscriptionQueryService`.
- [ ] `MemberAccessChangedEvent`.
- [ ] Audit + Events.
- [ ] State Transition + Concurrency Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| تجديد قبل الانتهاء = نفس السجل؟ | لا — سجل جديد (C-20). |
| Grace Renewal؟ | سجل جديد + القديم `EXPIRED`. |
| Refund عند الإلغاء؟ | لا في MVP. |
| تمديد التجميد تلقائي؟ | نعم عند `scheduled_unfreeze_date`. |
| اشتراكين نشطين؟ | مسموح — `grants_access` بيحدد الدخول. |
