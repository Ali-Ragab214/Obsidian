# Identity — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.identity.domain`  
> **المرتبط:** [`README.md`](./README.md) · [security-strategy](../../architecture/security-strategy.md) · [entity-relationships](../../architecture/entity-relationships.md)

---

## 1. الكيانات

### 1.1 `User`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL`, FK | Tenant |
| `branchId` | `UUID` | `NULL` للـ Owner | FK → branches |
| `name` | `String` | `NOT NULL`, 2–100 | — |
| `phone` | `String` | `NOT NULL` | Login Identifier |
| `email` | `String` | `NULL` | Login ثانوي |
| `passwordHash` | `String` | `NOT NULL` | BCrypt |
| `role` | `Role` | `NOT NULL` | 4 أدوار |
| `active` | `boolean` | `NOT NULL` | `true` |
| `mustChangePassword` | `boolean` | `NOT NULL` | `false` |
| `lastLoginAt` | `Instant` | `NULL` | — |
| `failedLoginAttempts` | `int` | `NOT NULL` | `0` |
| `lockedUntil` | `Instant` | `NULL` | — |
| `createdAt` / `updatedAt` | `Instant` | `NOT NULL` | — |

```java
@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User extends BaseEntity implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(name = "branch_id")
    private UUID branchId;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, length = 20)
    private String phone;

    @Column(length = 255)
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 30)
    private Role role;

    @Column(nullable = false)
    private boolean active = true;

    @Column(name = "must_change_password", nullable = false)
    private boolean mustChangePassword = false;

    @Column(name = "last_login_at")
    private Instant lastLoginAt;

    @Column(name = "failed_login_attempts", nullable = false)
    private int failedLoginAttempts = 0;

    @Column(name = "locked_until")
    private Instant lockedUntil;

    public boolean isLocked(Instant now) {
        return lockedUntil != null && lockedUntil.isAfter(now);
    }

    public void recordFailedLogin(Instant now, int threshold, Duration lockDuration) {
        this.failedLoginAttempts++;
        if (this.failedLoginAttempts >= threshold) {
            this.lockedUntil = now.plus(lockDuration);
        }
    }

    public void recordSuccessfulLogin(Instant now) {
        this.failedLoginAttempts = 0;
        this.lockedUntil = null;
        this.lastLoginAt = now;
    }

    public void deactivate() { this.active = false; }
    public void activate()   { this.active = true; }
}
```

> **`mustChangePassword`** مضاف للـ Provisioning/Reset — [SC-21](../../architecture/entity-relationships.md#6-تصحيحات-وتوسعات-الـ-schema-schema-corrections).

### 1.2 `RefreshToken`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL` | Tenant |
| `userId` | `UUID` | `NOT NULL`, FK | — |
| `tokenHash` | `String` | `UNIQUE`, `NOT NULL` | SHA-256 |
| `expiresAt` | `Instant` | `NOT NULL` | +7 أيام |
| `revokedAt` | `Instant` | `NULL` | — |
| `replacedBy` | `UUID` | `NULL` | Token الجديد |
| `userAgent` | `String` | `NULL` | ≤ 255 |
| `ipAddress` | `String` | `NULL` | ≤ 45 |
| `createdAt` | `Instant` | `NOT NULL` | — |

```java
@Entity
@Table(name = "refresh_tokens")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class RefreshToken {

    @Id
    private UUID id;

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(name = "user_id", nullable = false)
    private UUID userId;

    @Column(name = "token_hash", nullable = false, unique = true, length = 64)
    private String tokenHash;

    @Column(name = "expires_at", nullable = false)
    private Instant expiresAt;

    @Column(name = "revoked_at")
    private Instant revokedAt;

    @Column(name = "replaced_by")
    private UUID replacedBy;

    private String userAgent;
    private String ipAddress;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt;

    public boolean isActive(Instant now) {
        return revokedAt == null && expiresAt.isAfter(now);
    }

    public void revoke(Instant now, UUID replacement) {
        this.revokedAt = now;
        this.replacedBy = replacement;
    }
}
```

---

## 2. الـ Enums

```java
public enum Role {
    GYM_OWNER,
    BRANCH_MANAGER,
    RECEPTIONIST,
    TRAINER
}
```

---

## 3. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-IDN-01 | `phone` فريد داخل الـ Org | DB Unique |
| INV-IDN-02 | `email` فريد داخل الـ Org (لو موجود) | Partial Unique |
| INV-IDN-03 | `GYM_OWNER` → `branchId = NULL` | DB CHECK + Service |
| INV-IDN-04 | الأدوار التانية → `branchId != NULL` | DB CHECK + Service |
| INV-IDN-05 | `branchId` تابع لنفس الـ Org | `BranchQueryService` |
| INV-IDN-06 | ممنوع تعطيل آخر `GYM_OWNER` نشط | Service |
| INV-IDN-07 | `passwordHash` BCrypt 12 | Service |
| INV-IDN-08 | المستخدم المعطّل ممنوع يدخل | AuthService |
| INV-IDN-09 | `lockedUntil > now` → رفض | AuthService |
| INV-IDN-10 | Refresh Token مخزّن Hash فقط | Service |
| INV-IDN-11 | Reuse → إلغاء كل العائلة | RefreshTokenService |
| INV-IDN-12 | `mustChangePassword` → بس `/auth/password` | Filter |

---

## 4. تدفق الـ Login

```text
1. استقبل {organizationSlug, username, password}.
2. OrganizationQueryService.findBySlug(slug)
   → مش موجود / SUSPENDED / CLOSED → 401.
3. TenantTransactionRunner.runInTenant(orgId):
   a. ابحث بـ phone أو email.
   b. مش موجود → 401 (نفس الرسالة).
   c. lockedUntil > now → 423/401.
   d. active = false → 401.
   e. BCrypt.matches → لو فشل: recordFailedLogin + 401.
   f. نجاح: recordSuccessfulLogin.
   g. أصدر Access Token.
   h. أصدر Refresh Token (raw + hash).
   i. خزّن Hash.
   j. Audit LOGIN_SUCCESS.
4. رجّع {accessToken, refreshToken, expiresIn, user, mustChangePassword}.
```

> **ممنوع** كشف الفرق بين "المستخدم مش موجود" و"كلمة المرور غلط" — نفس الـ `401 AUTHENTICATION_FAILED`.

---

## 5. دورة حياة الـ Refresh Token

```text
Login:
  raw = "v1." + orgId + "." + secureRandom(32 bytes)
  hash = SHA-256(raw)
  INSERT refresh_tokens(hash, expires_at = now + 7d)

Refresh:
  1. Parse orgId من الـ raw.
  2. runInTenant(orgId).
  3. hash = SHA-256(random).
  4. ابحث بالـ hash.
  5. مش موجود → 401.
  6. revoked != null → Reuse! ألغي كل Tokens المستخدم + Audit.
  7. expired → 401.
  8. أنشئ Token جديد + ألغي القديم (replaced_by).
  9. أصدر Access Token جديد.

Logout:
  revoked_at = now()

Password Change:
  revoked_at = now() لكل Tokens المستخدم.
```

---

## 6. الـ JWT

```java
public record JwtClaims(
    UUID userId,
    UUID orgId,
    UUID branchId,
    Role role,
    String name,
    String jti
) {}
```

| Claim | المصدر |
|---|---|
| `sub` | `userId` |
| `org_id` | `user.orgId` |
| `branch_id` | `user.branchId` (ممكن null) |
| `role` | `user.role.name()` |
| `name` | `user.name` |
| `jti` | `UUID.randomUUID()` |
| `iat` / `exp` | الآن / +15 دقيقة |

---

## 7. قواعد إدارة الموظفين

### إنشاء موظف
- الـ Owner فقط.
- `orgId` من Context.
- `branchId` إلزامي لغير الـ Owner.
- كلمة مرور مؤقتة → `mustChangePassword = true`.
- Audit `USER_CREATED`.

### تعديل موظف
- الاسم / Phone / Email / Branch / Role.
- تغيير Role بيترتب عليه Audit `USER_ROLE_CHANGED`.
- ممنوع تغيير Role الـ Owner الأخير.

### تعطيل موظف
- `active = false`.
- إلغاء كل Refresh Tokens.
- Audit `USER_DEACTIVATED`.
- ممنوع تعطيل آخر Owner.

### Reset Password
- الـ Owner فقط.
- كلمة مرور مؤقتة + `mustChangePassword = true`.
- إلغاء كل Refresh Tokens.
- Audit `PASSWORD_RESET`.

---

## 8. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `InvalidCredentialsException` | 401 | Login فاشل |
| `AccountLockedException` | 423 | Lockout |
| `UserInactiveException` | 401 | `active = false` |
| `OrganizationSuspendedException` | 401 | Org موقوفة |
| `InvalidRefreshTokenException` | 401 | Refresh غلط/منتهي |
| `RefreshTokenReuseException` | 401 | Reuse |
| `WeakPasswordException` | 400 | Password Policy |
| `LastOwnerException` | 422 | تعطيل آخر Owner |

---

## 9. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [security-strategy](../../architecture/security-strategy.md)
