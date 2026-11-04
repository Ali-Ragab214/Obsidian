# Shift — مطابقة ورديات الكاش (P1)

> **الـ Phase:** Post-MVP · **الأولوية:** P1 · **الـ Package:** `com.kammel.features.shift`  
> **المرجع:** [`../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md`](../../../Kammel_Project_Documentation/08_BUSINESS_LOGIC_AND_STATE_MACHINES.md)

---

## 1. الميزة دي بتعمل إيه؟

`shift` بتحمي **فلوس الدرج**:

- الاستقبال بيفتح وردية قبل ما يستلم كاش.
- كل دفعة كاش بتتربط بالوردية المفتوحة.
- عند الإغلاق: الاستقبال بيدخل **العدّ الفعلي** بدون ما يشوف المتوقع (Blind Count).
- النظام بيحسب **الفرق (Variance)**.
- الوردية بتتقفل وتبقى **غير قابلة للتعديل**.
- الـ Owner/Manager بيشوفوا الفروق في الـ Dashboard.

**مش مسؤولة عن:**
- تسجيل الدفع → `payment`.
- الـ Refund → `payment` (P1).
- تقارير الإيراد → `reporting`.

---

## 2. ليه P1؟

- `0-doc` اعتبرها P0، لكن الوثائق التقنية والـ `1-doc2` اعتبروها P1.
- القرار [`C-02`](../../architecture/decisions-and-assumptions.md#2-التعارضات-اللي-تم-حلها-resolved-conflicts): **P1 افتراضياً**.
- تتقدم لـ P0 لو مقابلات Phase 0 أثبتت إن السرقة/الفروق هي الوجع الأول.
- الـ Schema من أول يوم فيها `payments.shift_id` nullable — عشان الربط ميتكسرش.

---

## 3. المصطلحات

| المصطلح | المعنى |
|---|---|
| `CashShift` | وردية كاش لموظف واحد في فرع. |
| `expected_cash_minor` | المتوقع = كاش مدفوع − كاش مرتجع. |
| `counted_cash_minor` | العدّ الفعلي في الدرج. |
| `variance_minor` | `counted - expected`. |
| `Blind Count` | الموظف مش شايف المتوقع قبل ما يدخل العدّ. |
| `Shortage` | Variance سالب — فلوس ناقصة. |
| `Overage` | Variance موجب — فلوس زيادة. |

---

## 4. القواعد

| ID | القاعدة |
|---|---|
| BR-024 | الدفع الكاش لازم يكون مرتبط بوردة مفتوحة. |
| BR-029 | وردية مفتوحة واحدة لكل User. |
| BR-030 | لازم يقفل الوردية قبل فتح جديدة. |
| BR-031 | Variance يظهر في Owner Dashboard. |
| BR-032 | الوردية Immutable بعد الإغلاق. |
| SHF-01 | `expected = Σ cash payments − Σ cash refunds`. |
| SHF-02 | Non-cash payments مستبعدة من الدرج. |
| SHF-03 | العدّ Blind — المتوقع مخفي قبل الإغلاق. |
| SHF-04 | Variance = `counted − expected`. |
| SHF-05 | ممنوع فتح وردية بفرع مختلف عن فرع الموظف. |
| SHF-06 | Manager/Owner يقدر يعمل Force Close. |
| SHF-07 | ممنوع تعديل وردية مقفولة. |
| SHF-08 | كل فتح/إغلاق/Force Close له Audit. |

---

## 5. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| فتح وردية | ✅ | ✅ | ✅ (فرعه) | ❌ |
| إغلاق ورديته | ✅ | ✅ | ✅ | ❌ |
| Force Close | ✅ | ✅ | ❌ | ❌ |
| عرض ورديات الفرع | ✅ | ✅ | ❌ | ❌ |
| Variance Report | ✅ | ✅ (فرعه) | ❌ | ❌ |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `MoneyUtil`, `TenantContext` |
| `tenancy` | Compile | `BranchQueryService` |
| `identity` | Compile | `UserQueryService` |
| `payment` | Read | `PaymentQueryService` (Cash totals) |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `shift`؟**
`payment` (Cash validation)، `reporting` (Variance Report).

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`implementation-plan.md`](./implementation-plan.md) | Domain + API + Testing + Tasks. |

> **ملاحظة:** لأن `shift` P1، الوثائق consolidated في ملفين بدل خمسة. التفاصيل الكاملة (Domain/API/Tests) جوه `implementation-plan.md`.

---

## 8. Definition of Done

- [ ] `V19__create_cash_shifts.sql` + Partial Unique Index.
- [ ] `V20__add_payments_shift_fk.sql`.
- [ ] `CashShift` Entity.
- [ ] فتح/إغلاق/Force Close.
- [ ] Blind Count.
- [ ] Variance.
- [ ] ربط الدفع الكاش.
- [ ] Immutability.
- [ ] Audit.
- [ ] Reporting.
- [ ] Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل Refund جزء من الـ Expected؟ | نعم — بعد ما الـ Refund يتضاف P1. |
| هل الوردية بتقفل تلقائياً آخر اليوم؟ | لا — Force Close يدوي. |
| هل الموظف يقدر يفتح وردية في فرع تاني؟ | لا. |
| هل الفرق بيتم تسويته ماليًا؟ | لا — تسجيل + مراجعة بشرية. |
| هل Cash Drawer Hardware؟ | لا — إدخال يدوي. |
