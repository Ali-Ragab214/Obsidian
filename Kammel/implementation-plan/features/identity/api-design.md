# Identity — تصميم الـ API

> **Base Path:** `/api/v1/auth` + `/api/v1/users`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md) · [security-strategy](../../architecture/security-strategy.md)

---

## 1. الـ Endpoints

| Method | Path | Auth | الوصف |
|---|---|---|---|
| POST | `/auth/login` | Public | تسجيل الدخول |
| POST | `/auth/refresh` | Public (Refresh) | Rotation |
| POST | `/auth/logout` | Bearer | إلغاء Refresh Token |
| GET | `/auth/me` | Bearer | المستخدم الحالي |
| PATCH | `/auth/password` | Bearer | تغيير كلمة المرور |
| GET | `/users` | Owner | قائمة الموظفين |
| POST | `/users` | Owner | إنشاء موظف |
| GET | `/users/{id}` | Owner | تفاصيل موظف |
| PATCH | `/users/{id}` | Owner | تعديل موظف |
| POST | `/users/{id}/deactivate` | Owner | تعطيل |
| POST | `/users/{id}/reset-password` | Owner | Reset |

---

## 2. `POST /auth/login`

**Request:**
```json
{
  "organizationSlug": "gold-gym-maadi",
  "username": "01012345678",
  "password": "Password123!"
}
```

**Validation:**
- `organizationSlug`: `@NotBlank`, lowercase.
- `username`: `@NotBlank` — Phone أو Email.
- `password`: `@NotBlank`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "v1.a1b2c3d4-...<opaque>",
    "expiresInSeconds": 900,
    "mustChangePassword": false,
    "user": {
      "id": "e2f3a4b5-...",
      "name": "Ali Hassan",
      "role": "RECEPTIONIST",
      "organizationId": "a1b2c3d4-...",
      "branchId": "99999999-..."
    }
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Errors:**
- `401 AUTHENTICATION_FAILED` — أي فشل (بدون تفصيل).
- `423 ACCOUNT_LOCKED` — Lockout.
- `401 ORGANIZATION_SUSPENDED`.

---

## 3. `POST /auth/refresh`

**Request:**
```json
{ "refreshToken": "v1.a1b2c3d4-...<opaque>" }
```

**Response `200 OK`:** نفس شكل Login (Tokens جديدة + `mustChangePassword`).

**Rules:**
- القديم يتم إلغاؤه.
- Reuse → إلغاء كل العائلة + `401`.

**Errors:** `401 INVALID_REFRESH_TOKEN`.

---

## 4. `POST /auth/logout`

**Headers:** `Authorization: Bearer`.

**Request (اختياري):**
```json
{ "refreshToken": "v1...." }
```

**Response `204 No Content`.**

- لو الـ Body فاضي → إلغاء Refresh Token المرتبط بالـ Access Token إن أمكن.
- دايماً `204` حتى لو الـ Token ملغي.

---

## 5. `GET /auth/me`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "e2f3a4b5-...",
    "name": "Ali Hassan",
    "phone": "01012345678",
    "email": "ali@goldgym.com",
    "role": "RECEPTIONIST",
    "organizationId": "a1b2c3d4-...",
    "branchId": "99999999-...",
    "mustChangePassword": false,
    "lastLoginAt": "2026-11-03T09:55:00Z"
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

---

## 6. `PATCH /auth/password`

**Request:**
```json
{
  "currentPassword": "Temp123!",
  "newPassword": "NewPassword456!"
}
```

**Validation:**
- 8–72 حرف.
- Upper + Lower + Digit + Special.
- `newPassword != currentPassword`.

**Response `204 No Content`.**

**Rules:**
- إلغاء كل Refresh Tokens.
- `mustChangePassword = false`.
- Audit `PASSWORD_CHANGED`.

**Errors:** `400 WEAK_PASSWORD`, `401 AUTHENTICATION_FAILED`.

> **`mustChangePassword = true`** → كل الـ Endpoints بترجع `403 PASSWORD_CHANGE_REQUIRED` ما عدا `/auth/password` و`/auth/logout` و`/auth/me`.

---

## 7. `GET /users`

**Query:** `role`, `branchId`, `active`, `search`, `page`, `size`, `sort`.

**Response:** `PageResponse<UserSummaryResponse>`.

```json
{
  "id": "e2f3a4b5-...",
  "name": "Ali Hassan",
  "phone": "01012345678",
  "email": "ali@goldgym.com",
  "role": "RECEPTIONIST",
  "branchId": "99999999-...",
  "active": true,
  "lastLoginAt": "2026-11-03T09:55:00Z"
}
```

---

## 8. `POST /users`

**Request:**
```json
{
  "name": "Mostafa Ibrahim",
  "phone": "01123456789",
  "email": "mostafa@goldgym.com",
  "role": "BRANCH_MANAGER",
  "branchId": "99999999-8888-7777-6666-555555555555"
}
```

**Response `201 Created`:**
```json
{
  "success": true,
  "message": "User created",
  "data": {
    "id": "b2c3d4e5-...",
    "name": "Mostafa Ibrahim",
    "role": "BRANCH_MANAGER",
    "branchId": "99999999-...",
    "temporaryPassword": "Xy7#kLm2",
    "mustChangePassword": true
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Rules:**
- `temporaryPassword` تظهر **مرة واحدة فقط**.
- `GYM_OWNER` → `branchId` لازم يكون `null`.
- غير الـ Owner → `branchId` إلزامي وتابع للـ Org.
- Audit `USER_CREATED`.

**Errors:** `409 DUPLICATE_RESOURCE`, `422 BUSINESS_RULE_VIOLATION`.

---

## 9. `PATCH /users/{id}`

**Request (كل الحقول اختيارية):**
```json
{
  "name": "Mostafa I. Ibrahim",
  "phone": "01123456789",
  "email": "mostafa.new@goldgym.com",
  "role": "RECEPTIONIST",
  "branchId": "99999999-..."
}
```

**Rules:**
- ممنوع تعديل `orgId`.
- تغيير Role → Audit `USER_ROLE_CHANGED`.
- ممنوع تغيير Role آخر Owner.
- تغيير Branch → التحقق من الـ Org.

---

## 10. `POST /users/{id}/deactivate`

**Response `200 OK`:**
```json
{ "success": true, "message": "User deactivated", "data": { "id": "...", "active": false } }
```

**Rules:**
- إلغاء كل Refresh Tokens.
- ممنوع تعطيل آخر Owner.
- Audit `USER_DEACTIVATED`.

---

## 11. `POST /users/{id}/reset-password`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Password reset",
  "data": {
    "userId": "b2c3d4e5-...",
    "temporaryPassword": "Qw9$rTy3",
    "mustChangePassword": true
  }
}
```

**Rules:**
- إلغاء كل Refresh Tokens.
- Audit `PASSWORD_RESET`.
- ممنوع Reset لآخر Owner بواسطة غير الـ Owner (كلهم Owner أصلاً).

---

## 12. الـ DTOs

```java
public record LoginRequest(
    @NotBlank String organizationSlug,
    @NotBlank String username,
    @NotBlank String password
) {}

public record RefreshRequest(@NotBlank String refreshToken) {}

public record AuthResponse(
    String accessToken,
    String refreshToken,
    long expiresInSeconds,
    boolean mustChangePassword,
    UserSummary user
) {}

public record CreateUserRequest(
    @NotBlank @Size(min = 2, max = 100) String name,
    @ValidPhone String phone,
    @Email String email,
    @NotNull Role role,
    UUID branchId
) {}

public record UpdateUserRequest(
    @Size(min = 2, max = 100) String name,
    @ValidPhone String phone,
    @Email String email,
    Role role,
    UUID branchId
) {}

public record ChangePasswordRequest(
    @NotBlank String currentPassword,
    @NotBlank @Size(min = 8, max = 72) String newPassword
) {}
```

---

## 13. الصلاحيات

```java
@PreAuthorize("hasRole('GYM_OWNER')")
UserResponse createUser(CreateUserRequest request);

@PreAuthorize("hasRole('GYM_OWNER')")
void deactivateUser(UUID userId);

@PreAuthorize("isAuthenticated()")
UserResponse me();
```

---

## 14. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [security-strategy](../../architecture/security-strategy.md)
