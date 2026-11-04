# Payment — تسجيل الدفع والتحصيل

> **الـ Phase:** 2 · **الأولوية:** P0 · **الترتيب التنفيذي:** 7  
> **الـ Package:** `com.kammel.features.payment`  
> **المرجع:** [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) · [`../../../Kammel_Project_Documentation/07_API_DESIGN.md`](../../../Kammel_Project_Documentation/07_API_DESIGN.md)

---

## 1. الميزة دي بتعمل إيه؟

`payment` هي **الـ Ledger المالي** بتاع الجيم:

- تسجيل دفعة ضد `Subscription` — كاش، InstaPay، تحويل بنكي، أو Online.
- دعم **الدفع الجزئي** وحساب المتبقي.
- منع تكرار الدفع بـ `Idempotency-Key`.
- توليد **رقم إيصال متسلسل لكل Org** (`KM-2026-000001`).
- توليد إيصال PDF.
- طابور التحصيل (Collections) للمدفوعات الناقصة.
- نشر `PaymentRecordedEvent` بشكل **Synchronous** عشان الاشتراك يتفعّل في نفس الـ Transaction.
- تسجيل `PAYMENT_RECORDED` في الـ Audit.

**مش مسؤولة عن:**
- تغيير حالة الاشتراك → `subscription` (بتسمع الـ Event).
- بيانات العضو → `member`.
- الـ Refund والـ Reversal API → **P1** (العمود موجود من الآن).
- مطابقة الكاش والورديات → `shift` (P1).

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `amount_minor` | المبلغ بالقروش (`BIGINT`). 700 جنيه = `70000`. |
| `price_minor_snapshot` | سعر الاشتراك وقت إنشائه — مرجع المتبقي. |
| `paidTotalMinor` | مجموع الدفعات `COMPLETED` للاشتراك. |
| `outstandingMinor` | `price_minor_snapshot - paidTotalMinor`. |
| `idempotency_key` | مفتاح فريد لكل محاولة دفع — بيمنع التكرار. |
| `receipt_number` | رقم إيصال متسلسل لكل Org لكل سنة. |
| `REVERSED` | إلغاء محاسبي لدفعة غلط — بدون حذف. |
| `Collections` | قائمة الاشتراكات اللي لسه عليها متبقي. |

---

## 3. النطاق

### داخل الـ MVP
- [x] `POST /payments` مع `Idempotency-Key` إلزامي.
- [x] دفع كامل أو جزئي.
- [x] حساب `paidTotalMinor` و`outstandingMinor`.
- [x] `PaymentRecordedEvent` Synchronous.
- [x] رقم إيصال متسلسل لكل Org/سنة.
- [x] إيصال PDF.
- [x] قائمة المدفوعات + تفاصيل دفعة.
- [x] طابور التحصيل.
- [x] Audit `PAYMENT_RECORDED`.
- [x] Immutability على الحقول المالية.

### خارج الـ MVP
- [ ] `POST /payments/{id}/refund` — P1.
- [ ] `POST /payments/{id}/reverse` — P1.
- [ ] ربط الدفع بوردة كاش مفتوحة — P1 (`shift`).
- [ ] Online Payment Gateway / Webhooks — P2.
- [ ] Discounts / Coupons / Installments Schedule — P1/P2.
- [ ] Multi-currency — مش مطلوب.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| تسجيل دفع | ✅ | ✅ | ✅ | ❌ |
| عرض قائمة المدفوعات | ✅ | ✅ (فرعه) | ❌ | ❌ |
| عرض تفاصيل دفعة | ✅ | ✅ (فرعه) | ✅ (دفعة سجلها) | ❌ |
| تحميل الإيصال | ✅ | ✅ (فرعه) | ✅ (دفعة سجلها) | ❌ |
| طابور التحصيل | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Refund / Reversal | ✅ | ✅ (P1) | ❌ | ❌ |

> **Receptionist** بيسجل الدفع وبيطبع الإيصال، لكنه مش بيشوف تقارير مالية مجمعة.

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-019 | كل المبالغ `Long` بالقروش — ممنوع `double`/`float`. |
| BR-020 | `Idempotency-Key` يمنع تسجيل نفس الدفع مرتين. |
| BR-021 | الدفعات لا تُحذف أبداً — التصحيح بـ Reversal/Refund. |
| BR-022 | الدفع الجزئي مسموح، والمتبقي ظاهر في التحصيل. |
| BR-023 | أرقام الإيصالات متسلسلة لكل Org. |
| BR-024 | الدفع الكاش يترتبط بوردة مفتوحة — **P1**. |
| PAY-01 | `amount_minor > 0` دائماً. |
| PAY-02 | ممنوع دفع أكتر من المتبقي في الـ MVP — `422 PAYMENT_EXCEEDS_BALANCE`. |
| PAY-03 | `member_id` بيتاخد من الاشتراك — مش من الـ Request. |
| PAY-04 | الدفع لا يغيّر `subscription.status` مباشرة — Event فقط. |
| PAY-05 | `receipt_number` لا يتكرر داخل نفس الـ Org. |
| PAY-06 | كل حقول الدفع المالية Immutable بعد الإنشاء. |
| PAY-07 | `PaymentRecordedEvent` لازم يخلص قبل الـ Commit. |
| PAY-08 | الـ Replay لنفس `Idempotency-Key` يرجع نفس الدفعة — مش ينشئ واحدة جديدة. |
| PAY-09 | ممنوع دفع على اشتراك `CANCELLED` أو `EXPIRED` بدون تجديد. |
| PAY-10 | `paidTotalMinor` بيشمل الدفعات `COMPLETED` فقط. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `MoneyUtil`, `TenantContext`, Events |
| `tenancy` | Compile | `BranchQueryService` (للفلترة) |
| `identity` | Compile | `UserQueryService` |
| `member` | Read | `MemberQueryService` للتحقق من العضو والفرع |
| `subscription` | Compile | `SubscriptionQueryService`, `SubscriptionService.activateIfFullyPaid` |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `payment`؟**
`subscription` (Event)، `reporting` (Read Contract)، `shift` (P1).

> **ممنوع:** `payment` يستدعي `SubscriptionRepository` أو `MemberRepository` مباشرة.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `Payment`, `DocumentSequence`, الـ Idempotency، الإيصال. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام والترتيب. |
| [`api-design.md`](./api-design.md) | الـ Endpoints والـ DTOs. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migrations `payments` + `document_sequences` + RLS + Grants.
- [ ] `Payment` + `DocumentSequence` Entities.
- [ ] `app.next_document_number` مستخدمة فعلياً.
- [ ] `Idempotency-Key` شغال تحت التزامن.
- [ ] `PaymentRecordedEvent` بينشر `totalPaidMinor`.
- [ ] التفعيل بيحصل في نفس الـ Transaction.
- [ ] إيصال PDF بالعربي والإنجليزي.
- [ ] قائمة المدفوعات + التحصيل.
- [ ] Audit + Immutability Tests.
- [ ] كل الاختبارات في [`testing-plan.md`](./testing-plan.md) خضراء.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل نسمح بالدفع الزائد؟ | لا في MVP — `422` (PAY-02). |
| هل الإيصال يتولد PDF ولا HTML؟ | PDF بـ OpenPDF. |
| هل الدفع الكاش يمنع بدون وردية؟ | لا في MVP — `shift_id` nullable. |
| هل Refund ينقص `paidTotal`؟ | P1 — بيتم بـ Reverse Entry. |
| هل الدفع على اشتراك `GRACE` يقبل؟ | نعم — بيموّل تجديد/سداد متبقي. |
| هل `ONLINE` بيستخدم Gateway؟ | لا — تسجيل يدوي في MVP. |
