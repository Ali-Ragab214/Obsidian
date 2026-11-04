# Member — تصميم الـ API

> **Base Path:** `/api/v1/members`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/members` | Owner, Manager, Receptionist | تسجيل |
| GET | `/members` | Owner, Manager, Receptionist, Trainer | بحث/قائمة |
| GET | `/members/{id}` | Owner, Manager, Receptionist, Trainer | تفاصيل |
| PATCH | `/members/{id}` | Owner, Manager, Receptionist | تعديل |
| DELETE | `/members/{id}` | Owner, Manager | Soft Delete |
| GET | `/members/lookup` | Owner, Manager, Receptionist | بحث سريع |
| POST | `/members/import` | Owner, Manager | Excel |
| GET | `/members/import/template` | Owner, Manager | قالب Excel |

> **Timeline:** موجود في `reporting` → `GET /dashboard/members/{id}/timeline`.

---

## 2. `POST /members`

**Request:**
```json
{
  "fullName": "Ahmed Tarek",
  "phone": "01012345678",
  "gender": "MALE",
  "birthDate": "1994-05-12",
  "branchId": "99999999-8888-7777-6666-555555555555",
  "source": "INSTAGRAM",
  "notes": "Prefers morning workouts",
  "whatsappOptIn": true,
  "assignedTrainerId": "b2c3d4e5-...",
  "photoUrl": "https://cdn.example.com/photo.jpg"
}
```

**Validation:**
- `fullName`: `@NotBlank`, `@Size(2,100)`.
- `phone`: `@ValidPhone`.
- `branchId`: `@NotNull`.
- `whatsappOptIn`: `@NotNull`.
- `notes`: `@Size(max=1000)`.

**Response `201 Created`:**
```json
{
  "success": true,
  "message": "Member registered",
  "data": {
    "id": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "memberCode": "M-000123",
    "qrToken": "7f3c2a1b-...",
    "fullName": "Ahmed Tarek",
    "phone": "01012345678",
    "branchId": "99999999-...",
    "status": "ACTIVE",
    "whatsappOptIn": true,
    "assignedTrainerId": "b2c3d4e5-...",
    "createdAt": "2026-11-03T10:00:00Z"
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Errors:** `409 DUPLICATE_RESOURCE` + `memberId` الموجود.

```json
{
  "timestamp": "2026-11-03T10:00:00Z",
  "status": 409,
  "error": "DUPLICATE_RESOURCE",
  "message": "Member with this phone already exists",
  "path": "/api/v1/members",
  "details": [
    { "field": "phone", "rejectedValue": "01012345678", "message": "Existing memberId: a0b1..." }
  ]
}
```

---

## 3. `GET /members`

**Query Params:** `search`, `status`, `branchId`, `trainerId`, `page`, `size`, `sort`.

- `search` → Phone / Name / Member Code.
- `Trainer` → فلتر تلقائي `assignedTrainerId = currentUserId`.
- `Branch Manager / Receptionist` → فلتر تلقائي `branchId`.
- `deletedAt != null` مستبعد دايماً.

**Response:** `PageResponse<MemberSummaryResponse>`.

```json
{
  "id": "a0b1c2d3-...",
  "memberCode": "M-000123",
  "fullName": "Ahmed Tarek",
  "phone": "01012345678",
  "branchId": "99999999-...",
  "status": "ACTIVE",
  "whatsappOptIn": true,
  "hasActiveSubscription": true
}
```

> `hasActiveSubscription` = `status == ACTIVE` (Cached). التفاصيل من `subscription`.

---

## 4. `GET /members/{id}`

**Response `200 OK`:** `MemberResponse` شامل كل الحقول ما عدا `orgId`.

**Errors:** `404 RESOURCE_NOT_FOUND` — مش موجود أو Org تانية أو Trainer مش عميله.

---

## 5. `PATCH /members/{id}`

**Request (اختياري):**
```json
{
  "fullName": "Ahmed T. Hassan",
  "phone": "01098765432",
  "branchId": "99999999-...",
  "source": "FRIEND",
  "notes": "Updated note",
  "whatsappOptIn": false,
  "assignedTrainerId": null
}
```

**Rules:**
- `memberCode` / `qrToken` / `orgId` ممنوع تعديلهم.
- تغيير Phone → فحص التكرار.
- `whatsappOptIn false → true` → `whatsappOptInAt = now()`.
- Audit `MEMBER_UPDATED`.

**Errors:** `409 DUPLICATE_RESOURCE`, `404`, `422`.

---

## 6. `DELETE /members/{id}`

**Response `204 No Content`.**

**Rules:**
- Soft Delete فقط.
- `status = INACTIVE`.
- ممنوع Hard Delete.
- Audit `MEMBER_DELETED`.

---

## 7. `GET /members/lookup`

**Query:** `phone` (إلزامي) أو `code` أو `qr`.

**Response `200 OK`:** `MemberLookupResponse` مختصر (بدون Notes).

**الاستخدام:** شاشة الاستقبال قبل الدفع أو الـ Check-in.

**Errors:** `404 RESOURCE_NOT_FOUND`.

---

## 8. `POST /members/import`

**Query:** `dryRun` (default `true`).

**Request:** `multipart/form-data` — `file: members.xlsx`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Import validated",
  "data": {
    "dryRun": true,
    "totalRows": 350,
    "successCount": 348,
    "failureCount": 2,
    "createdMembers": 0,
    "createdSubscriptions": 0,
    "errors": [
      { "rowNumber": 45, "phone": "01000", "error": "INVALID_PHONE" },
      { "rowNumber": 112, "phone": "01122334455", "error": "DUPLICATE_IN_DB" }
    ]
  }
}
```

**Commit (`dryRun=false`):**
```json
{
  "dryRun": false,
  "totalRows": 350,
  "successCount": 348,
  "failureCount": 2,
  "createdMembers": 348,
  "createdSubscriptions": 120,
  "errors": [ ... ]
}
```

**Rules:**
- ≤ 10 MB، ≤ 5,000 صف.
- `.xlsx` فقط.
- الصفوف الفاشلة لا تُدخل.
- Commit في Transaction واحدة — أي خطأ DB → Rollback كامل.
- Audit `MEMBER_IMPORTED`.

**Errors:** `413 IMPORT_TOO_LARGE`, `422 IMPORT_VALIDATION_FAILED`, `400 INVALID_FILE_TYPE`.

---

## 9. `GET /members/import/template`

**Response:** `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` + Header `Content-Disposition`.

- القالب فيه Header Row + صف مثال + Data Validation للـ Enums.

---

## 10. الـ DTOs

```java
public record CreateMemberRequest(
    @NotBlank @Size(min = 2, max = 100) String fullName,
    @NotBlank @ValidPhone String phone,
    Gender gender,
    @Past LocalDate birthDate,
    @NotNull UUID branchId,
    MemberSource source,
    @Size(max = 1000) String notes,
    @NotNull Boolean whatsappOptIn,
    UUID assignedTrainerId,
    @Size(max = 500) String photoUrl
) {}

public record UpdateMemberRequest(
    @Size(min = 2, max = 100) String fullName,
    @ValidPhone String phone,
    Gender gender,
    @Past LocalDate birthDate,
    UUID branchId,
    MemberSource source,
    @Size(max = 1000) String notes,
    Boolean whatsappOptIn,
    UUID assignedTrainerId,
    @Size(max = 500) String photoUrl
) {}

public record MemberResponse(
    UUID id,
    String memberCode,
    UUID qrToken,
    String fullName,
    String phone,
    Gender gender,
    LocalDate birthDate,
    UUID branchId,
    MemberSource source,
    String notes,
    boolean whatsappOptIn,
    UUID assignedTrainerId,
    MemberStatus status,
    boolean hasActiveSubscription,
    Instant createdAt
) {}

public record MemberSummaryResponse(
    UUID id,
    String memberCode,
    String fullName,
    String phone,
    UUID branchId,
    MemberStatus status,
    boolean whatsappOptIn,
    boolean hasActiveSubscription
) {}

public record ImportResultResponse(
    boolean dryRun,
    int totalRows,
    int successCount,
    int failureCount,
    int createdMembers,
    int createdSubscriptions,
    List<ImportRowError> errors
) {}

public record ImportRowError(int rowNumber, String phone, String error) {}
```

---

## 11. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
MemberResponse create(CreateMemberRequest request);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST','TRAINER')")
PageResponse<MemberSummaryResponse> search(MemberFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
void delete(UUID id);
```

- Trainer → Query Filter `assignedTrainerId`.
- Branch Scope → Service Layer.
- ممنوع `orgId` في أي Body.

---

## 12. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
