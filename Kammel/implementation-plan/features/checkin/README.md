# Check-in — تسجيل الحضور

> **الـ Phase:** 3 · **الأولوية:** P0 · **الترتيب التنفيذي:** 8  
> **الـ Package:** `com.kammel.features.checkin`  
> **المرجع:** [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md) · [`../../../Kammel_Project_Documentation/07_API_DESIGN.md`](../../../Kammel_Project_Documentation/07_API_DESIGN.md)

---

## 1. الميزة دي بتعمل إيه؟

`checkin` هي **بوابة الدخول اليومية** للجيم:

- بحث عن العضو بـ **الهاتف** أو **كود العضو** أو **QR**.
- التحقق من وجود اشتراك يمنح دخول (`ACTIVE` / `GRACE`).
- منع الدخول لو `FROZEN` مع عرض سبب التجميد وتاريخ فك التجميد.
- تنبيه لو `EXPIRED` مع زر تجديد.
- تحذير لو العضو دخل بالفعل النهارده — **مش منع**.
- خصم جلسة لو الباقة Session-based.
- تسجيل `CheckIn` غير قابل للتعديل.
- Override بسبب + Audit للحالات الاستثنائية.

**مش مسؤولة عن:**
- حساب الاشتراك أو تجديده → `subscription`.
- تسجيل الدفع → `payment`.
- حساب الـ Risk → `risk` (بيقرأ الحضور).
- تقارير الحضور → `reporting`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `identifier` | مدخل البحث: هاتف / `M-000123` / QR Token. |
| `method` | طريقة التسجيل: `PHONE_SEARCH`, `MEMBER_CODE`, `NAME_SEARCH`, `STATIC_QR`. |
| `subscription_id` | الاشتراك اللي منح الدخول — `NULL` في Override بدون اشتراك. |
| `FROZEN` | مجمد — الدخول ممنوع. |
| `GRACE` | فترة سماح — الدخول مسموح مع تحذير. |
| `DUPLICATE_WARNING` | العضو مسجل حضور بالفعل النهارده. |
| `override` | تسجيل حضور استثنائي بقرار Manager/Owner + سبب. |

---

## 3. النطاق

### داخل الـ MVP
- [x] بحث بالهاتف / كود العضو / QR.
- [x] تحقق اشتراك `ACTIVE` / `GRACE`.
- [x] منع `FROZEN` مع رسالة واضحة.
- [x] تنبيه `EXPIRED`.
- [x] Warning للـ `EXPIRING_SOON`.
- [x] Warning للتكرار في نفس اليوم + تأكيد.
- [x] خصم جلسة.
- [x] Override بسبب + Audit.
- [x] `CheckIn` Immutable.
- [x] قائمة حضور اليوم + فلترة.
- [x] `CheckInQueryService` لـ `risk` و`reporting`.

### خارج الـ MVP
- [ ] Dynamic QR (P1).
- [ ] Time-restricted plans enforcement (P1).
- [ ] Trainer check-in / Class booking.
- [ ] Turnstile / Hardware integration.
- [ ] Offline mode.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| تسجيل حضور | ✅ | ✅ | ✅ | ❌ |
| بحث بدون تسجيل | ✅ | ✅ | ✅ | ❌ |
| حضور اليوم | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |
| Override | ✅ | ✅ | ❌ | ❌ |
| تأكيد تكرار | ✅ | ✅ | ✅ | ❌ |

> **Receptionist** بيسجل الحضور اليومي. الـ Override الاستثنائي محتاج Manager/Owner.

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-014 | ممنوع إحراج العضو — الرسائل ودّية وواضحة. |
| BR-015 | كل Check-in يسجل Time + Branch + Method + Subscription. |
| BR-016 | Override يحتاج Reason + Audit. |
| BR-017 | Check-in هو أساس Risk Score. |
| BR-018 | التكرار في نفس اليوم Warning مش Block. |
| CHK-01 | ممنوع Check-in بدون `branch_id`. |
| CHK-02 | `FROZEN` ممنوع — حتى مع Override (الا لو Manager قرر). |
| CHK-03 | `GRACE` مسموح + Warning. |
| CHK-04 | `EXPIRED` ممنوع — يعرض تاريخ الانتهاء. |
| CHK-05 | الجلسة بتتخصم مرة واحدة لكل Check-in. |
| CHK-06 | `CheckIn` ممنوع `UPDATE` / `DELETE`. |
| CHK-07 | التكرار = نفس العضو + نفس اليوم بتوقيت القاهرة. |
| CHK-08 | Override لـ Manager/Owner فقط + Reason + Audit. |
| CHK-09 | ممنوع Check-in لـ `CANCELLED` / `PENDING_PAYMENT`. |
| CHK-10 | الـ Response لازم يوضح سبب المنع بدون أرقام داخلية. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `DateUtil`, `TenantContext` |
| `tenancy` | Compile | `BranchQueryService` |
| `identity` | Compile | `UserQueryService` |
| `member` | Compile | `MemberQueryService` |
| `subscription` | Compile | `SubscriptionQueryService`, `SubscriptionService.consumeSession` |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `checkin`؟**
`risk` (Read Contract)، `reporting` (Read Contract).

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `CheckIn`, Validation Flow, Override. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `check_ins` + RLS + Immutability Trigger.
- [ ] `CheckIn` Entity + Enums.
- [ ] بحث بالهاتف / الكود / QR.
- [ ] Validation Matrix كاملة.
- [ ] Duplicate Warning + Confirm.
- [ ] Session Deduction.
- [ ] Override + Audit.
- [ ] `CheckInQueryService`.
- [ ] API + Tests.
- [ ] P95 < 500ms على 5000 عضو.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل التكرار في فرع تاني يعتبر Duplicate؟ | نعم — على مستوى الـ Org. |
| هل Receptionist يقدر يعمل Override؟ | لا — Manager/Owner فقط. |
| هل `FROZEN` ممكن Override؟ | لا في MVP — الاستثناء يحتاج Manager + سبب. |
| هل الباقة الوقتية بتتنفذ؟ | لا — Data Only (ASM-14). |
| هل Check-in بيعمل Event؟ | لا — `risk`/`reporting` بيقروا بالـ Query. |
| هل QR ثابت ولا متغير؟ | ثابت (`member.qr_token`) في MVP. |
