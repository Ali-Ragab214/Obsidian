# Tenancy — تصميم الـ API

> **Base Path:** `/api/v1`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md) · [security-strategy](../../architecture/security-strategy.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/organization` | All authenticated | بيانات الـ Org الحالية |
| PATCH | `/organization` | `GYM_OWNER` | تعديل اسم الـ Org |
| GET | `/branches` | All authenticated | قائمة الفروع المسموحة |
| POST | `/branches` | `GYM_OWNER` | إنشاء فرع |
| GET | `/branches/{id}` | Owner / Branch Scope | تفاصيل فرع |
| PATCH | `/branches/{id}` | `GYM_OWNER` | تعديل فرع |
| POST | `/branches/{id}/deactivate` | `GYM_OWNER` | تعطيل فرع |

> **مفيش** `DELETE /branches/{id}` — ممنوع الحذف.

---

## 2. `GET /organization`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "a1b2c3d4-e5f6-4a5b-8c9d-0e1f2a3b4c5d",
    "slug": "gold-gym-maadi",
    "name": "Gold Gym Maadi",
    "status": "ACTIVE",
    "branchCount": 2,
    "createdAt": "2026-01-10T08:00:00Z"
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Errors:** `401 TOKEN_INVALID`, `404 RESOURCE_NOT_FOUND`.

---

## 3. `PATCH /organization`

**Request:**
```json
{
  "name": "Gold Gym Maadi — Branch 1"
}
```

**Validation:**
- `name`: `@NotBlank`, `@Size(min = 2, max = 255)`.

**Response `200 OK`:** نفس شكل `GET`.

**Rules:**
- الـ `slug` مش قابل للتعديل — لو اتبعت → `400 VALIDATION_FAILED`.
- الـ `status` مش قابل للتعديل من الـ API.
- Audit: `ORGANIZATION_UPDATED` مع `beforeState` / `afterState`.

**Errors:** `403 ACCESS_DENIED`, `400 VALIDATION_FAILED`.

---

## 4. `GET /branches`

**Query Params:** `active` (اختياري، default `true`), `page`, `size`, `sort`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "id": "99999999-8888-7777-6666-555555555555",
        "name": "Main Branch",
        "address": "12 Road 9, Maadi",
        "timezone": "Africa/Cairo",
        "active": true,
        "createdAt": "2026-01-10T08:05:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1,
    "first": true,
    "last": true
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Scoping:**
- `GYM_OWNER` → كل فروع الـ Org.
- غيره → الفرع المخصص فقط.

---

## 5. `POST /branches`

**Request:**
```json
{
  "name": "Nasr City Branch",
  "address": "5 Abbas El Akkad, Nasr City",
  "timezone": "Africa/Cairo"
}
```

**Validation:**
- `name`: `@NotBlank`, `@Size(2, 255)`.
- `timezone`: `@NotBlank` + IANA Valid.
- `orgId` **ممنوع** في الـ Body.

**Response `201 Created`:** `BranchResponse`.

**Errors:**
- `409 DUPLICATE_RESOURCE` — اسم فرع مكرر في نفس الـ Org (اختياري).
- `400 VALIDATION_FAILED`.

---

## 6. `PATCH /branches/{id}`

**Request:** نفس حقول الإنشاء، كلها اختيارية.

**Rules:**
- لازم الفرع تابع للـ Org.
- تعديل `active` ممنوع من هنا — استخدم `/deactivate`.
- Audit: `BRANCH_UPDATED`.

**Errors:** `404 RESOURCE_NOT_FOUND`, `403 ACCESS_DENIED`.

---

## 7. `POST /branches/{id}/deactivate`

**Request:** بدون Body.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Branch deactivated",
  "data": { "id": "99999999-...", "active": false },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Rules:**
- ممنوع تعطيل آخر فرع نشط → `422 BUSINESS_RULE_VIOLATION`.
- البيانات التاريخية بتفضل.
- Audit: `BRANCH_DEACTIVATED`.

---

## 8. الـ DTOs

```java
public record OrganizationResponse(
    UUID id,
    String slug,
    String name,
    OrganizationStatus status,
    long branchCount,
    Instant createdAt
) {}

public record UpdateOrganizationRequest(
    @NotBlank @Size(min = 2, max = 255) String name
) {}

public record CreateBranchRequest(
    @NotBlank @Size(min = 2, max = 255) String name,
    @Size(max = 500) String address,
    @NotBlank String timezone
) {}

public record UpdateBranchRequest(
    @Size(min = 2, max = 255) String name,
    @Size(max = 500) String address,
    String timezone
) {}

public record BranchResponse(
    UUID id,
    String name,
    String address,
    String timezone,
    boolean active,
    Instant createdAt
) {}
```

---

## 9. الصلاحيات والـ Scoping

```java
@PreAuthorize("hasRole('GYM_OWNER')")
public OrganizationResponse updateOrganization(UpdateOrganizationRequest request) { ... }

@PreAuthorize("hasRole('GYM_OWNER')")
public BranchResponse createBranch(CreateBranchRequest request) { ... }

@PreAuthorize("hasRole('GYM_OWNER') or @branchSecurity.canAccessBranch(#id)")
public BranchResponse getBranch(UUID id) { ... }
```

- الـ `orgId` من `TenantContext` دايماً.
- أي `branchId` بيتحقق في الـ Service مش الـ Controller بس.
- فرع تابع لـ Org تانية → **404**.

---

## 10. مرتبط بـ

- [`domain-design.md`](./domain-design.md) — القواعد.
- [`testing-plan.md`](./testing-plan.md) — اختبارات الـ API.
- [api-design العام](../../api-design.md) — الـ Envelope والأخطاء.
