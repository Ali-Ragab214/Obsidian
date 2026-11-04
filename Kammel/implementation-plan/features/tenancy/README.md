# Tenancy — المنظمات والفروع وعزل البيانات

> **الـ Phase:** 1 · **الأولوية:** P0 · **الترتيب التنفيذي:** 1  
> **الـ Package:** `com.kammel.features.tenancy`  
> **المرجع:** [`../../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md`](../../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. الميزة دي بتعمل إيه؟

`tenancy` هي **حدود العزل** في كمّل. هي المسؤولة عن:

- `Organization` — الجيم نفسه (الـ Tenant).
- `Branch` — الفرع الفيزيائي داخل الجيم.
- الـ `org_id` اللي كل الميزات التانية بتعتمد عليه.
- التحقق إن أي `branchId` تابع للـ Org الصح.
- Provisioning الجيم الجديد (CLI).

**مش مسؤولة عن:**
- تسجيل دخول الموظفين → `identity`.
- إدارة اشتراكات العملاء → `subscription`.
- فواتير الـ SaaS نفسها → خارج النطاق.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `Organization` | الجيم ككيان تجاري — الـ Tenant. |
| `Branch` | فرع فيزيائي داخل الجيم. |
| `slug` | معرف فريد للجيم في الـ URL/Login — مثال `gold-gym-maadi`. |
| `active branch` | فرع شغال وممكن يتم عليه عمليات. |
| `organization status` | `ACTIVE` / `SUSPENDED` / `CLOSED`. |

---

## 3. النطاق (Scope)

### داخل الـ MVP
- [x] إنشاء Org + أول Branch + أول `GYM_OWNER` (CLI).
- [x] عرض/تعديل بيانات الـ Org (الاسم فقط).
- [x] إدارة الفروع (إضافة/تعديل/تعطيل).
- [x] `OrganizationQueryService` + `BranchQueryService`.
- [x] التحقق من ملكية الفرع للـ Org.
- [x] تعطيل فرع بدون حذف بياناته.
- [x] RLS على `branches`.

### خارج الـ MVP
- [ ] `PLATFORM_ADMIN` UI.
- [ ] Self-service Signup.
- [ ] فواتير الـ SaaS.
- [ ] نقل فرع بين Org‑ين.
- [ ] حذف Org نهائياً.
- [ ] Multi-timezone per branch (كل الفروع `Africa/Cairo` افتراضياً).

---

## 4. المستخدمون والصلاحيات

| العملية | Owner | Branch Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| عرض بيانات الـ Org | ✅ | ✅ | ✅ | ✅ |
| تعديل اسم الـ Org | ✅ | ❌ | ❌ | ❌ |
| عرض كل الفروع | ✅ | ❌ (فرعه فقط) | ❌ (فرعه فقط) | ❌ (فرعه فقط) |
| إنشاء فرع | ✅ | ❌ | ❌ | ❌ |
| تعديل فرع | ✅ | ❌ | ❌ | ❌ |
| تعطيل فرع | ✅ | ❌ | ❌ | ❌ |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-001 | كل Org لازم يكون عندها **Branch واحد على الأقل** نشط. |
| BR-004 | الـ Owner يشوف كل الفروع؛ غيره يشوف فرعه فقط. |
| BR-008 | Manager يشتغل على فرعه فقط. |
| BR-009 | Manager ممنوع يشوف فرع تاني حتى في نفس الـ Org. |
| TEN-01 | `slug` فريد عالمياً وغير قابل للتغيير بعد الإنشاء. |
| TEN-02 | ممنوع تعطيل آخر فرع نشط في الـ Org. |
| TEN-03 | ممنوع حذف فرع — `active = false` فقط. |
| TEN-04 | أي `branchId` في Request لازم يتحقق إنه تابع للـ Org الحالي. |
| TEN-05 | Org بحالة `SUSPENDED` → كل المستخدمين يرفضوا Login. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `TenantScoped`, `ApiResponse`, RLS |
| — | — | **مفيش اعتمادية على أي Feature تانية** |

**مين بيعتمد على `tenancy`؟**
`identity`, `audit`, `member`, `plan`, `subscription`, `payment`, `checkin`, `notification`, `reporting`.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا — النطاق والقواعد. |
| [`domain-design.md`](./domain-design.md) | الكيانات والـ Invariants والـ Enums. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام بالترتيب ومعايير القبول. |
| [`api-design.md`](./api-design.md) | الـ Endpoints والـ DTOs. |
| [`testing-plan.md`](./testing-plan.md) | خطة الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `organizations` + `branches` + RLS.
- [ ] `Organization` و`Branch` Entities.
- [ ] `OrganizationQueryService` + `BranchQueryService`.
- [ ] APIs مع `@PreAuthorize`.
- [ ] Provisioning CLI.
- [ ] Tenant Isolation Test.
- [ ] Branch Scoping Test.
- [ ] mفيش `DELETE` على `branches`.
- [ ] التوثيق محدّث.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل الـ Owner يقدر يغيّر الـ slug؟ | لا — ثابت بعد الإنشاء. |
| هل ممكن فرع بدون عنوان؟ | نعم — `address` اختياري. |
| هل ممكن Org بدون Branch؟ | لا — Provisioning بينشئ الاتنين. |
| Timezone لكل فرع؟ | `Africa/Cairo` افتراضياً؛ الحقل موجود للمستقبل. |
