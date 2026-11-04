# Plan — باقات الاشتراك

> **الـ Phase:** 2 · **الأولوية:** P0 · **الترتيب التنفيذي:** 5  
> **الـ Package:** `com.kammel.features.plan`  
> **المرجع:** [`../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md`](../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md)

---

## 1. الميزة دي بتعمل إيه؟

`plan` هي **قالب الاشتراك**:

- تعريف الباقات (شهري / ربع سنوي / سنوي / جلسات / PT / Day Pass).
- السعر والمدة وعدد الجلسات.
- قواعد التجميد (`freeze_max_days`) والـ Grace (`grace_days`).
- هل الباقة تمنح دخول الجيم (`grants_access`).
- تعطيل الباقة بدون التأثير على الاشتراكات القائمة.

**مش مسؤولة عن:**
- إنشاء الاشتراكات → `subscription`.
- السعر الفعلي للعقد → Snapshot في `subscription`.
- العروض والخصومات → خارج النطاق (P1).
- Time-restricted enforcement → Data only ([ASM-14](../../architecture/decisions-and-assumptions.md#asm-14)).

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `Plan` | قالب باقة. |
| `duration_days` | مدة الاشتراك بالأيام. |
| `session_count` | عدد الجلسات (للباقات القائمة على الجلسات). |
| `price_minor` | السعر بالقروش. |
| `freeze_max_days` | أقصى أيام تجميد مسموحة. |
| `grace_days` | أيام السماح بعد الانتهاء. |
| `grants_access` | هل الباقة تسمح بدخول الجيم. |

---

## 3. النطاق

### داخل الـ MVP
- [x] إنشاء / تعديل / عرض الباقات.
- [x] تعطيل / تفعيل.
- [x] `price_minor` بالقروش.
- [x] `duration_days` + `session_count`.
- [x] `freeze_max_days` + `grace_days`.
- [x] `grants_access`.
- [x] `time_restriction` (Data only).
- [x] منع التعديل على الاشتراكات القائمة.
- [x] `PlanQueryService` للـ `subscription`.

### خارج الـ MVP
- [ ] خصومات / كوبونات.
- [ ] عروض موسمية.
- [ ] Time-restricted enforcement.
- [ ] Multi-branch pricing.
- [ ] نسخ باقة (Clone).
- [ ] حذف باقة.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| إنشاء باقة | ✅ | ❌ | ❌ | ❌ |
| تعديل باقة | ✅ | ❌ | ❌ | ❌ |
| تعطيل/تفعيل | ✅ | ❌ | ❌ | ❌ |
| عرض الباقات | ✅ | ✅ | ✅ | ❌ |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-002 | الـ Plan لازم له Name وPrice > 0 وDuration. |
| PLN-01 | `duration_days > 0`. |
| PLN-02 | `price_minor > 0` (بالقروش). |
| PLN-03 | `session_count > 0` للباقات القائمة على الجلسات. |
| PLN-04 | `freeze_max_days >= 0` و`grace_days >= 0`. |
| PLN-05 | تعديل السعر لا يؤثر على الاشتراكات القائمة. |
| PLN-06 | تعطيل الباقة لا يؤثر على الاشتراكات القائمة. |
| PLN-07 | ممنوع حذف باقة — `active = false`. |
| PLN-08 | `PT_PACKAGE` → `grants_access = false` افتراضياً. |
| PLN-09 | `time_restriction` Data only في الـ MVP. |
| PLN-10 | اسم الباقة فريد داخل الـ Org (Case-insensitive). |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `MoneyUtil`, `ApiResponse` |
| `tenancy` | Compile | `TenantContext` / Org Scope |
| `audit` | Compile | `AuditService` |

**مين بيعتمد على `plan`؟**
`subscription`, `reporting`.

> **ممنوع** `plan` تعتمد على `subscription` — الباقة متعرفش مين اشترك.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `Plan`, القواعد, الحسابات. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `plans` + RLS + Constraints.
- [ ] `Plan` Entity + `PlanType`.
- [ ] CRUD + Deactivate/Activate.
- [ ] `PlanQueryService`.
- [ ] Snapshot Independence Test.
- [ ] Audit.
- [ ] Isolation Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل السعر 0 مسموح؟ | لا — BR-002. |
| هل اسم الباقة فريد؟ | نعم داخل الـ Org. |
| هل PT Package بتدخل الجيم؟ | لا — `grants_access = false`. |
| هل Time Restriction بتتنفذ؟ | لا في الـ MVP. |
