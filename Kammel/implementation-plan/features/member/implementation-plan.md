# Member — خطة التنفيذ

> **الـ Phase:** 2 · **المدة:** 5–6 أيام · **الترتيب:** بعد `audit`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-MEM-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-AUD-001 |
| **المخرجات** | `V6__create_members.sql`, `Member.java`, Enums |
| **معيار القبول** | RLS + Partial Unique + `member_code` + `qr_token` |

**خطوات:**
1. الأعمدة الأساسية + `deleted_at`.
2. `UNIQUE (org_id, phone) WHERE deleted_at IS NULL`.
3. `UNIQUE (org_id, member_code)`.
4. `UNIQUE (qr_token)`.
5. `SELECT app.enable_tenant_rls('members');`.
6. Indexes `(org_id, branch_id, status)`, `(org_id, assigned_trainer_id)`.

---

### TSK-MEM-002 — Member Code + QR

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-001 |
| **المخرجات** | `MemberCodeGenerator` + `QrTokenGenerator` |
| **معيار القبول** | 20 Threads → أكواد فريدة بدون Gaps |

**خطوات:**
1. `pg_advisory_xact_lock(hashtext(orgId))`.
2. `MAX(SUBSTRING(member_code FROM 3)) + 1`.
3. تنسيق `M-%06d`.
4. `qr_token = UUID.randomUUID()`.
5. Concurrency Test.

---

### TSK-MEM-003 — CRUD + Search

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-002, TSK-TEN-003, TSK-IDN-003 |
| **المخرجات** | `MemberService`, `MemberController` |
| **معيار القبول** | CRUD + Search + Lookup + Branch/Trainer Validation |

**خطوات:**
1. `create`, `update`, `get`, `search`, `lookup`.
2. `@PreAuthorize` + Branch Scope.
3. `MemberSpecifications` للبحث.
4. Trainer Filter تلقائي.
5. Audit.

---

### TSK-MEM-004 — Soft Delete

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-003 |
| **المخرجات** | `DELETE /members/{id}` + `Member.softDelete` |
| **معيار القبول** | `deleted_at` + `INACTIVE` + Audit + مفيش Hard Delete |

**خطوات:**
1. Owner/Manager فقط.
2. `deletedAt = now()`.
3. `status = INACTIVE`.
4. Audit `MEMBER_DELETED`.
5. استبعاد المحذوف من كل Queries.

---

### TSK-MEM-005 — Excel Import (Dry Run)

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-003 |
| **المخرجات** | `MemberImportService.parseAndValidate(...)` |
| **معيار القبول** | 500 صف < 3s + تقرير Row-by-Row |

**خطوات:**
1. Apache POI `XSSFWorkbook`.
2. تحقق من MIME/Extension/Size.
3. قراءة الأعمدة بالـ Header.
4. Validation لكل صف.
5. `ImportResultResponse`.
6. ممنوع Formula Evaluation.

---

### TSK-MEM-006 — Excel Import (Commit)

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-005 |
| **المخرجات** | `POST /members/import?dryRun=false` |
| **معيار القبول** | إدخال الصفوف السليمة + اشتراك اختياري + Rollback عند خطأ DB |

**خطوات:**
1. إعادة الـ Validation.
2. إنشاء Members + Codes + QR.
3. إنشاء Subscription (لو `plan_name`).
4. `SubscriptionEvent(CREATED, imported=true)`.
5. Audit `MEMBER_IMPORTED`.
6. Transaction واحدة.

---

### TSK-MEM-007 — Member Status Sync

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-003, `subscription` |
| **المخرجات** | `MemberEventListener` |
| **معيار القبول** | `MemberAccessChangedEvent` بيحدث `status` |

**خطوات:**
1. `@TransactionalEventListener(AFTER_COMMIT)`.
2. `hasActiveMembership=true` → `ACTIVE`.
3. `false` → `INACTIVE`.
4. Idempotent.
5. Audit اختياري.

---

### TSK-MEM-008 — Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-MEM-007 |
| **المخرجات** | Unit + Repo + API + Import + Isolation + Trainer |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

---

## 2. الترتيب

```text
TSK-MEM-001 Migration + Entity
      ↓
TSK-MEM-002 Code + QR
      ↓
TSK-MEM-003 CRUD + Search
      ↓
TSK-MEM-004 Soft Delete
      ↓
TSK-MEM-005 Import Dry Run
      ↓
TSK-MEM-006 Import Commit
      ↓
TSK-MEM-007 Status Sync
      ↓
TSK-MEM-008 Tests
```

---

## 3. معايير القبول النهائية

- [ ] `members` Migration + RLS.
- [ ] `member_code` تسلسلي وفريد.
- [ ] `qr_token` ثابت وفريد.
- [ ] Phone فريد بين غير المحذوفين.
- [ ] Soft Delete شغال ومفيش Hard Delete.
- [ ] البحث بالموبايل/الاسم/الكود.
- [ ] Trainer يشوف عملاءه فقط.
- [ ] Import Dry Run + Commit.
- [ ] 500 صف < 3s.
- [ ] `MemberAccessChangedEvent` بيحدث الحالة.
- [ ] Audit لكل عملية.
- [ ] Isolation Tests.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| تكرار `member_code` تحت التزامن | Advisory Lock + Concurrency Test |
| ملف Excel ضخم | Size/Row Limit + Streaming |
| بيانات مكررة في الملف | Validation داخل الملف |
| استيراد اشتراك بدون Payment | توثيق واضح — مش Revenue |
| Trainer يشوف عضو غلط | Query Filter + Test |
| Soft Delete يكسر التقارير | استبعاد `deleted_at` في Queries |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
