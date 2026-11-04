# Member — إدارة الأعضاء

> **الـ Phase:** 2 · **الأولوية:** P0 · **الترتيب التنفيذي:** 4  
> **الـ Package:** `com.kammel.features.member`  
> **المرجع:** [`../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md`](../../../Kammel_Project_Documentation/04_DOMAIN_MODEL.md)

---

## 1. الميزة دي بتعمل إيه؟

`member` هي سجل **عملاء الجيم**:

- تسجيل عضو جديد بسرعة من الاستقبال.
- البحث بالموبايل / الاسم / الكود.
- تعديل البيانات.
- Soft Delete.
- توليد `member_code` و`qr_token`.
- استيراد Excel للأعضاء الحاليين.
- ربط العضو بكابتن (`assigned_trainer_id`).
- تحديث حالة العضو (`ACTIVE` / `INACTIVE`) بناءً على الاشتراكات.

**مش مسؤولة عن:**
- تسجيل الدخول — الأعضاء مش Users ([D-12](../../architecture/decisions-and-assumptions.md#d-12-member--user)).
- الاشتراكات → `subscription`.
- المدفوعات → `payment`.
- الحضور → `checkin`.
- Timeline الكامل → `reporting`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `Member` | عميل الجيم. |
| `member_code` | كود تسلسلي لكل Org — `M-000123`. |
| `qr_token` | UUID ثابت للـ QR Card. |
| `whatsapp_opt_in` | موافقة صريحة للرسائل. |
| `assigned_trainer_id` | الكابتن المسؤول (اختياري). |
| `deleted_at` | Soft Delete. |
| `status` | `ACTIVE` / `INACTIVE` — Cached Projection. |

---

## 3. النطاق

### داخل الـ MVP
- [x] تسجيل عضو + Validation.
- [x] البحث بالموبايل (أساسي) / الاسم / الكود.
- [x] تعديل البيانات.
- [x] Soft Delete.
- [x] `member_code` تسلسلي لكل Org.
- [x] `qr_token` تلقائي.
- [x] ربط كابتن.
- [x] استيراد Excel على مرحلتين (`dryRun`).
- [x] تحديث الحالة من أحداث الاشتراك.
- [x] فلترة Trainer لعملائه.
- [x] `GET /members/lookup`.
- [x] قالب Excel للتحميل.

### خارج الـ MVP
- [ ] رفع صور (Photo URL فقط).
- [ ] Member Login.
- [ ] Leads / Pipeline.
- [ ] استيراد المدفوعات التاريخية.
- [ ] Merge أعضاء مكررين.
- [ ] Face Recognition / Turnstile.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| تسجيل عضو | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |
| عرض قائمة الأعضاء | ✅ | ✅ (فرعه) | ✅ (فرعه) | ✅ (عملاؤه) |
| عرض عضو | ✅ | ✅ (فرعه) | ✅ (فرعه) | ✅ (عميله) |
| تعديل عضو | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |
| Soft Delete | ✅ | ✅ (فرعه) | ❌ | ❌ |
| استيراد Excel | ✅ | ✅ | ❌ | ❌ |
| ربط كابتن | ✅ | ✅ | ❌ | ❌ |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-003 | Excel Import يكتشف تكرار الموبايل داخل نفس الـ Org. |
| BR-011 | الموبايل هو مفتاح البحث الأساسي. |
| BR-012 | `(org_id, phone)` فريد بين غير المحذوفين. |
| BR-013 | `whatsapp_opt_in` لازم يتسجل صريح. |
| BR-037 | Trainer يشوف عملاءه فقط. |
| MEM-01 | `member_code` فريد لكل Org وغير قابل للتغيير. |
| MEM-02 | `qr_token` UUID فريد عالمياً. |
| MEM-03 | Soft Delete فقط — ممنوع Hard Delete. |
| MEM-04 | الموبايل المصري فقط (`010/011/012/015` + 8 أرقام). |
| MEM-05 | `status` Cached — المصدر الحقيقي هو `subscription`. |
| MEM-06 | Import ≤ 5,000 صف و≤ 10 MB. |
| MEM-07 | `dryRun=true` افتراضياً. |
| MEM-08 | ممنوع استيراد مدفوعات تاريخية في الـ MVP. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `BaseEntity`, `@ValidPhone`, `ApiResponse` |
| `tenancy` | Compile | `BranchQueryService` |
| `identity` | Compile | `UserQueryService` (Trainer) |
| `audit` | Compile | `AuditService` |
| `subscription` | Event | `MemberAccessChangedEvent` |

**مين بيعتمد على `member`؟**
`subscription`, `payment`, `checkin`, `risk`, `notification`, `reporting`.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `Member`, القواعد, Import. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migration `members` + RLS + Partial Unique.
- [ ] `Member` Entity + Enums.
- [ ] CRUD + Search + Lookup.
- [ ] Soft Delete.
- [ ] `member_code` + `qr_token`.
- [ ] Excel Import (Dry Run + Commit).
- [ ] Trainer Scoping.
- [ ] `MemberAccessChangedEvent` Listener.
- [ ] Audit.
- [ ] Isolation + Import Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل العضو ممكن يكون له أكتر من كابتن؟ | لا — `assigned_trainer_id` واحد ([ASM-13](../../architecture/decisions-and-assumptions.md#asm-13)). |
| هل الموبايل المحذوف ممكن يتسجل تاني؟ | نعم — Partial Unique. |
| هل Soft Delete بيلغي الاشتراك؟ | لا — الاشتراك بيفضل، لكن Check-in بيتمنع. |
| استيراد الاشتراكات التاريخية؟ | نعم — بدون Payment Records. |
