# Tenancy — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.tenancy.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. الكيانات

### 1.1 `Organization`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | `gen_random_uuid()` |
| `slug` | `String` | `UNIQUE`, `NOT NULL`, 3–50 | `^[a-z0-9-]+$` — غير قابل للتغيير |
| `name` | `String` | `NOT NULL`, 2–255 | اسم الجيم |
| `status` | `OrganizationStatus` | `NOT NULL` | `ACTIVE` افتراضياً |
| `createdAt` | `Instant` | `NOT NULL` | UTC |
| `updatedAt` | `Instant` | `NOT NULL` | UTC |

```java
@Entity
@Table(name = "organizations")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Organization extends BaseEntity {

    @Column(nullable = false, unique = true, length = 50)
    private String slug;

    @Column(nullable = false, length = 255)
    private String name;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private OrganizationStatus status = OrganizationStatus.ACTIVE;

    public void rename(String newName) {
        if (newName == null || newName.isBlank()) {
            throw new BusinessException("Organization name is required");
        }
        this.name = newName.trim();
    }

    public void suspend() { this.status = OrganizationStatus.SUSPENDED; }
    public void activate() { this.status = OrganizationStatus.ACTIVE; }
    public void close()   { this.status = OrganizationStatus.CLOSED; }

    public boolean isActive() { return status == OrganizationStatus.ACTIVE; }
}
```

> **ملاحظة:** `Organization` مش `TenantScoped` لأنها هي الـ Tenant نفسه.

### 1.2 `Branch`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL`, FK | Tenant |
| `name` | `String` | `NOT NULL`, 2–255 | "Main Branch" |
| `address` | `String` | `NULL`, ≤ 500 | اختياري |
| `timezone` | `String` | `NOT NULL` | `Africa/Cairo` |
| `active` | `boolean` | `NOT NULL` | `true` |
| `createdAt` | `Instant` | `NOT NULL` | — |

```java
@Entity
@Table(name = "branches")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Branch extends BaseEntity implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(nullable = false, length = 255)
    private String name;

    @Column(length = 500)
    private String address;

    @Column(nullable = false, length = 50)
    private String timezone = "Africa/Cairo";

    @Column(nullable = false)
    private boolean active = true;

    public void deactivate() { this.active = false; }
    public void activate()   { this.active = true; }
}
```

---

## 2. الـ Enums

```java
public enum OrganizationStatus {
    ACTIVE, SUSPENDED, CLOSED
}
```

> **مفيش `PLATFORM_ADMIN`** — Provisioning بـ CLI ([ASM-07](../../architecture/decisions-and-assumptions.md#asm-07)).

---

## 3. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-TEN-01 | `slug` يطابق `^[a-z0-9-]{3,50}$` | DTO + DB CHECK |
| INV-TEN-02 | `slug` فريد عالمياً | DB Unique |
| INV-TEN-03 | `name` مش فاضي ومش مسافات | Domain + DTO |
| INV-TEN-04 | كل Org عندها Branch نشط واحد على الأقل | Service (قبل تعطيل فرع) |
| INV-TEN-05 | `branch.orgId` لازم يطابق `TenantContext.orgId` | Service + RLS |
| INV-TEN-06 | ممنوع `DELETE` على `branches` | DB Grants + No API |
| INV-TEN-07 | ممنوع تعديل `slug` بعد الإنشاء | No Setter + DTO |
| INV-TEN-08 | Org `SUSPENDED`/`CLOSED` → Login مرفوض | `AuthService` |
| INV-TEN-09 | `timezone` لازم يكون IANA صالح | `ZoneId.of()` |
| INV-TEN-10 | ممنوع تعطيل آخر Branch نشط | `BranchService.deactivate` |

---

## 4. قواعد الـ Domain

### 4.1 إنشاء Organization (Provisioning)

```text
1. تحقق من صحة الـ slug وفرده.
2. أنشئ Organization(status = ACTIVE).
3. أنشئ Branch افتراضي (Main Branch).
4. أنشئ User(role = GYM_OWNER, branchId = null) بكلمة مرور مؤقتة.
5. سجّل Audit: ORGANIZATION_PROVISIONED.
6. كل ده في Transaction واحدة كـ kammel_owner.
```

> **ممنوع** إنشاء Org من الـ API في الـ MVP.

### 4.2 تعديل Organization

- الـ Owner يقدر يعدل `name` فقط.
- `status` بيتغير من CLI/Admin فقط.
- `slug` ثابت.

### 4.3 إنشاء Branch

- `orgId` من `TenantContext` — مش من الـ Body.
- `name` مطلوب.
- `timezone` افتراضي `Africa/Cairo`.
- بعد الإنشاء → Audit `BRANCH_CREATED`.

### 4.4 تعطيل Branch

```text
1. تحقق إن الفرع تابع للـ Org.
2. عدّ الفروع النشطة.
3. لو العدد = 1 → throw LastActiveBranchException (422).
4. active = false.
5. Audit BRANCH_DEACTIVATED.
```

- البيانات القديمة (Members, Payments, Check-ins) بتفضل مربوطة بالفرع.
- ممنوع عمليات جديدة على فرع معطّل.

### 4.5 حماية الـ Branch Scope

```java
public void assertBranchAccess(UUID branchId) {
    Branch branch = branchRepository.findById(branchId)
        .orElseThrow(ResourceNotFoundException::new);

    if (!branch.getOrgId().equals(TenantContext.getOrgId())) {
        throw new ResourceNotFoundException(); // 404 مش 403
    }

    if (!branch.isActive()) {
        throw new BusinessException("Branch is inactive");
    }

    UserPrincipal current = SecurityUtils.getCurrentUser();
    if (current.getRole() != Role.GYM_OWNER
        && !branchId.equals(current.getBranchId())) {
        throw new AccessDeniedException("Branch access denied");
    }
}
```

---

## 5. الـ Repositories

```java
public interface OrganizationRepository extends JpaRepository<Organization, UUID> {
    Optional<Organization> findBySlug(String slug);
    boolean existsBySlug(String slug);

    @Query("SELECT o.id FROM Organization o WHERE o.status = 'ACTIVE'")
    List<UUID> findAllActiveIds();
}

public interface BranchRepository extends JpaRepository<Branch, UUID> {
    List<Branch> findAllByOrgIdOrderByNameAsc(UUID orgId);
    Optional<Branch> findByIdAndOrgId(UUID id, UUID orgId);
    long countByOrgIdAndActiveTrue(UUID orgId);
    boolean existsByOrgIdAndNameIgnoreCase(UUID orgId, String name);
}
```

> **ملاحظة:** `findAllActiveIds` بتتستخدم من الـ Jobs — بتشتغل خارج RLS لأن `organizations` بدون RLS.

---

## 6. الـ Query Services

```java
public interface OrganizationQueryService {
    OrganizationSummary getOrganization(UUID orgId);
    Optional<OrganizationAuthView> findBySlug(String slug);
    List<UUID> listActiveOrgIds();
}

public interface BranchQueryService {
    BranchSummary getBranch(UUID branchId);
    List<BranchSummary> listBranches(UUID orgId);
    void validateBranchInOrg(UUID branchId, UUID orgId);
    boolean isActive(UUID branchId);
}
```

- بترجع **DTOs** — ممنوع Entity.
- `OrganizationAuthView` فيه `id`, `slug`, `status` فقط — يستخدمه `identity`.

---

## 7. الأخطاء الدومانية

| Exception | HTTP | متى |
|---|:---:|---|
| `DuplicateOrganizationSlugException` | 409 | Slug مستخدم |
| `LastActiveBranchException` | 422 | تعطيل آخر فرع |
| `InactiveBranchException` | 422 | عملية على فرع معطّل |
| `BranchNotFoundInOrgException` | 404 | فرع تابع لـ Org تانية |
| `OrganizationSuspendedException` | 401 | Login على Org موقوفة |

---

## 8. أحداث الـ Domain

| Event | متى | المستمع |
|---|---|---|
| `OrganizationProvisionedEvent` | بعد Provisioning | `audit` |
| `BranchCreatedEvent` | بعد إنشاء فرع | `audit` |
| `BranchDeactivatedEvent` | بعد تعطيل فرع | `audit` |

> مش كلها Integration Events — الأغلب Domain Events داخلية.

---

## 9. علاقات الـ JPA

- `Organization` و`Branch` في نفس الـ Feature → مسموح `@OneToMany` نظرياً.
- **لكن** هنستخدم `orgId` كـ `UUID` في `Branch` بدل `@ManyToOne` عشان:
  - أداء أفضل (مفيش Join غير لازم).
  - تجنب Lazy Loading.
  - توحيد النمط مع باقي الـ Features.
- ممنوع أي علاقة JPA مع `User` أو `Member`.

---

## 10. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md) — المهام.
- [`api-design.md`](./api-design.md) — الـ Endpoints.
- [database-strategy](../../architecture/database-strategy.md) — الـ Migration وRLS.
