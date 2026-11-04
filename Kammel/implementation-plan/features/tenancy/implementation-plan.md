# Tenancy — خطة التنفيذ

> **الـ Phase:** 1 · **المدة التقديرية:** 3–4 أيام · **الترتيب:** أول Feature بعد `shared`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md) · [implementation-phases](../../development-roadmap/implementation-phases.md)

---

## 1. الترتيب والمهام

### TSK-TEN-001 — Migrations: `organizations` + `branches`

| البند | التفصيل |
|---|---|
| **الأولوية** | P0 |
| **الاعتماديات** | `V1__create_extensions_and_helpers.sql` |
| **المخرجات** | `V2__create_organizations.sql`, `V3__create_branches.sql` |
| **معيار القبول** | Flyway Clean Migrate ناجح + RLS مفعّلة على `branches` + `slug` Unique |

**خطوات:**
1. `organizations` بدون RLS + `slug` Unique + `status` CHECK.
2. `branches` مع `org_id` FK + `timezone` + `active`.
3. `SELECT app.enable_tenant_rls('branches');`.
4. Index `(org_id, active)`.
5. Trigger `updated_at` على `organizations`.

**اختبارات:** Migration Test + RLS Coverage.

---

### TSK-TEN-002 — Domain Entities

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-001 |
| **المخرجات** | `Organization.java`, `Branch.java`, `OrganizationStatus.java` |
| **معيار القبول** | مفيش Setter للـ `slug` + `equals/hashCode` على `id` |

**خطوات:**
1. `Organization extends BaseEntity`.
2. `Branch extends BaseEntity implements TenantScoped`.
3. `rename()` / `deactivate()` / `activate()`.
4. `@NoArgsConstructor(access = PROTECTED)`.
5. ممنوع `@Data`.

---

### TSK-TEN-003 — Repositories + Query Services

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-002 |
| **المخرجات** | `OrganizationRepository`, `BranchRepository`, `OrganizationQueryService`, `BranchQueryService` |
| **معيار القبول** | كل Query بترجع DTO + كلها Tenant-scoped |

**خطوات:**
1. `findBySlug`, `findAllActiveIds`.
2. `findAllByOrgIdOrderByNameAsc`, `countByOrgIdAndActiveTrue`.
3. `OrganizationSummary` / `BranchSummary` Records.
4. `validateBranchInOrg` بترمي `ResourceNotFoundException`.

---

### TSK-TEN-004 — Organization API

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-003 |
| **المخرجات** | `OrganizationController` + `GET/PATCH /organization` |
| **معيار القبول** | Owner فقط يعدل + الاسم Validation + Audit |

**خطوات:**
1. `GET /organization` → `OrganizationResponse`.
2. `PATCH /organization` → `UpdateOrganizationRequest(name)`.
3. `@PreAuthorize("hasRole('GYM_OWNER')")` على التعديل.
4. `AuditService.record(ORGANIZATION_UPDATED, ...)`.

---

### TSK-TEN-005 — Branch API

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-003, TSK-TEN-004 |
| **المخرجات** | `BranchController` + CRUD (بدون Delete) |
| **معيار القبول** | Owner فقط ينشئ/يعدل/يعطّل + Branch Scoping |

**خطوات:**
1. `GET /branches` — Owner كل الفروع، غيره فرعه.
2. `POST /branches` — `orgId` من Context.
3. `PATCH /branches/{id}`.
4. `POST /branches/{id}/deactivate` — يمنع آخر فرع.
5. Audit لكل عملية.

---

### TSK-TEN-006 — Provisioning CLI

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-001, TSK-TEN-002 |
| **المخرجات** | `ProvisioningCommand` (Spring Shell أو `CommandLineRunner`) |
| **معيار القبول** | أمر واحد ينشئ Org + Branch + Owner + Audit |

```bash
./mvnw spring-boot:run -Dspring-boot.run.arguments="--provision.slug=gold-gym --provision.name=Gold Gym --provision.owner-phone=01000000001"
```

**خطوات:**
1. يشتغل بـ `kammel_owner` أو Profile `provisioning`.
2. يتحقق من الـ slug.
3. ينشئ Org + Branch + Owner.
4. كلمة مرور مؤقتة + `mustChangePassword = true`.
5. Audit `ORGANIZATION_PROVISIONED`.
6. ممنوع يشتغل في `prod` بدون Confirmation Flag.

---

### TSK-TEN-007 — Tenant Isolation & Branch Scoping Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-005 |
| **المخرجات** | `TenancyIsolationTest`, `BranchScopingTest` |
| **معيار القبول** | Org A مش شايف فروع Org B + Manager مش شايف فرع تاني |

**خطوات:**
1. Org‑ين + 2 Branches لكل واحدة.
2. `GET /branches` كـ Owner A → 2 فقط.
3. `GET /branches/{B}` → 404.
4. Manager A1 يفتح Branch A2 → 403.
5. Raw JDBC → صفر صفوف بدون Context.

---

### TSK-TEN-008 — Checkpoint 1 Integration

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-TEN-006, TSK-TEN-007 |
| **المخرجات** | Provisioning → Login → `GET /organization` E2E |
| **معيار القبول** | السيناريو كامل ينجح على Testcontainers |

---

## 2. ترتيب التنفيذ

```text
TSK-TEN-001  Migrations
      ↓
TSK-TEN-002  Entities
      ↓
TSK-TEN-003  Repositories + Query Services
      ↓
TSK-TEN-004  Organization API
      ↓
TSK-TEN-005  Branch API
      ↓
TSK-TEN-006  Provisioning CLI
      ↓
TSK-TEN-007  Isolation Tests
      ↓
TSK-TEN-008  Checkpoint
```

---

## 3. معايير القبول النهائية

- [ ] `organizations` + `branches` Migrations تعمل من DB فاضية.
- [ ] RLS على `branches` و`SELECT app.enable_tenant_rls` موجود.
- [ ] `slug` Unique وغير قابل للتغيير.
- [ ] كل Org عندها Branch واحد على الأقل.
- [ ] ممنوع تعطيل آخر فرع نشط.
- [ ] ممنوع `DELETE` على `branches`.
- [ ] `GET /branches` بترجع الفروع الصح حسب الـ Role.
- [ ] كل العمليات الحساسة متسجلة في Audit.
- [ ] Tenant Isolation Test + Branch Scoping Test ناجحين.
- [ ] Provisioning CLI موثق في `README` التشغيل.

---

## 4. المخاطر

| المخاطرة | الاحتمال | التخفيف |
|---|---|---|
| نسيان RLS على `branches` | متوسط | RLS Coverage Test في CI |
| Provisioning يشتغل بالـ Role الغلط | متوسط | Profile منفصل + تحقق من `current_user` |
| تعطيل آخر فرع | منخفض | Check في الـ Service + Test |
| تسريب `slug` بين جيمات | منخفض | Unique Index |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [implementation-phases](../../development-roadmap/implementation-phases.md)
