# Identity — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [security-strategy](../../architecture/security-strategy.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Login آمن | نفس رسالة الخطأ لكل فشل |
| Token Lifecycle | Rotation + Revocation + Reuse Detection |
| Lockout | 5 محاولات → قفل |
| Tenant Isolation | Org A مش بيدخل Org B |
| RBAC | كل Role بصلاحياته فقط |
| Branch Scope | Manager/Receptionist/Trainer مقيّدين |

---

## 2. Unit Tests

| # | الاختبار | المتوقع |
|---|---|---|
| UT-IDN-01 | `User.isLocked(now)` قبل/بعد `lockedUntil` | true/false |
| UT-IDN-02 | `recordFailedLogin` 5 مرات | `lockedUntil` متظبط |
| UT-IDN-03 | `recordSuccessfulLogin` | Reset العداد |
| UT-IDN-04 | `RefreshToken.isActive` منتهي/ملغي | false |
| UT-IDN-05 | `revoke(now, replacement)` | الحقول صح |
| UT-IDN-06 | Password Policy — ضعيف | `WeakPasswordException` |
| UT-IDN-07 | Password Policy — قوي | قبول |
| UT-IDN-08 | JWT Claims | كل الحقول صح |
| UT-IDN-09 | JWT منتهي | `TokenExpiredException` |
| UT-IDN-10 | JWT توقيع غلط | `TokenInvalidException` |

---

## 3. Repository Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RT-IDN-01 | `findByOrgIdAndPhone` | Optional صحيح |
| RT-IDN-02 | `findByOrgIdAndEmail` | Optional صحيح |
| RT-IDN-03 | Duplicate Phone في نفس Org | Exception |
| RT-IDN-04 | نفس Phone في Org تانية | مسموح |
| RT-IDN-05 | `findByTokenHash` | Optional صحيح |
| RT-IDN-06 | `revokeAllByUserId` | كلهم ملغيين |
| RT-IDN-07 | Owner بـ `branch_id` | CHECK Exception |

---

## 4. Auth Flow Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-IDN-01 | Login صحيح | `200` + Tokens |
| AT-IDN-02 | Password غلط | `401` + عداد يزيد |
| AT-IDN-03 | User مش موجود | `401` نفس الرسالة |
| AT-IDN-04 | Org Slug غلط | `401` |
| AT-IDN-05 | Org `SUSPENDED` | `401` |
| AT-IDN-06 | User `active=false` | `401` |
| AT-IDN-07 | 5 محاولات فاشلة | `423` |
| AT-IDN-08 | Lockout بينتهي | Login يشتغل |
| AT-IDN-09 | Login بـ Email | `200` |
| AT-IDN-10 | `mustChangePassword=true` | Flag في الـ Response |
| AT-IDN-11 | Endpoint تاني مع `mustChangePassword` | `403` |
| AT-IDN-12 | `/auth/password` مع `mustChangePassword` | `204` |

---

## 5. Token Tests

| # | السيناريو | المتوقع |
|---|---|---|
| TT-IDN-01 | Refresh صحيح | Tokens جديدة + القديم ملغي |
| TT-IDN-02 | Refresh Token ملغي | `401` |
| TT-IDN-03 | Refresh Token منتهي | `401` |
| TT-IDN-04 | Reuse | `401` + كل العائلة ملغية |
| TT-IDN-05 | Refresh Token من Org تانية | `401` |
| TT-IDN-06 | Logout | Refresh ملغي |
| TT-IDN-07 | Logout مرتين | `204` |
| TT-IDN-08 | Access Token منتهي | `401 TOKEN_EXPIRED` |
| TT-IDN-09 | Access Token توقيع غلط | `401 TOKEN_INVALID` |
| TT-IDN-10 | Password Change | كل Refresh ملغية |

---

## 6. RBAC / Security Tests

| # | السيناريو | المتوقع |
|---|---|---|
| ST-IDN-01 | Receptionist `POST /users` | `403` |
| ST-IDN-02 | Manager `POST /users` | `403` |
| ST-IDN-03 | Owner `POST /users` | `201` |
| ST-IDN-04 | Manager ينشئ موظف في فرع تاني | `404`/`403` |
| ST-IDN-05 | Body فيه `orgId` | متجاهل |
| ST-IDN-06 | تعطيل آخر Owner | `422` |
| ST-IDN-07 | Reset Password لموظف Org تانية | `404` |
| ST-IDN-08 | JWT فيه `org_id` تانية | مفيش وصول |

---

## 7. Tenant Isolation Tests

```java
@Test
void userFromOrgACannotAuthenticateAgainstOrgB() {
    fixtures.createUser(orgB, "01011112222", "Password123!");

    login("org-a", "01011112222", "Password123!")
        .expectStatus().isUnauthorized();
}

@Test
void rawJdbcWithoutContextReturnsNoUsers() {
    try (Connection c = dataSource.getConnection()) {
        ResultSet rs = c.createStatement().executeQuery("SELECT count(*) FROM users");
        rs.next();
        assertThat(rs.getInt(1)).isZero();
    }
}
```

| # | الاختبار | المتوقع |
|---|---|---|
| IT-IDN-01 | نفس Phone في Org‑ين | كل واحد بيدخل Org بتاعته |
| IT-IDN-02 | `GET /users` كـ Owner A | مستخدمو A فقط |
| IT-IDN-03 | `GET /users/{B}` | `404` |
| IT-IDN-04 | Raw JDBC بدون Context | صفر صفوف |
| IT-IDN-05 | `UPDATE users` لـ Org B | 0 Rows |

---

## 8. Password / Reset Tests

| # | السيناريو | المتوقع |
|---|---|---|
| PT-IDN-01 | إنشاء موظف | Temp Password + `mustChangePassword` |
| PT-IDN-02 | Reset Password | Temp Password جديدة + إلغاء Tokens |
| PT-IDN-03 | تغيير Password بكلمة قديمة غلط | `401` |
| PT-IDN-04 | Password قصيرة | `400` |
| PT-IDN-05 | Password بدون Special | `400` |
| PT-IDN-06 | Password نفس القديمة | `400` |

---

## 9. Audit Tests

| # | العملية | Action المتوقع |
|---|---|---|
| AU-IDN-01 | Login ناجح | `LOGIN_SUCCESS` |
| AU-IDN-02 | Login فاشل | `LOGIN_FAILED` |
| AU-IDN-03 | إنشاء موظف | `USER_CREATED` |
| AU-IDN-04 | تغيير Role | `USER_ROLE_CHANGED` |
| AU-IDN-05 | تعطيل | `USER_DEACTIVATED` |
| AU-IDN-06 | Reset Password | `PASSWORD_RESET` |
| AU-IDN-07 | Reuse | `REFRESH_TOKEN_REUSE` |

---

## 10. Performance

| المسار | الهدف |
|---|---|
| Login | P95 < 300ms |
| Refresh | P95 < 150ms |
| `GET /users` | P95 < 200ms |

---

## 11. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-008 | ST-IDN-04 |
| BR-009 | IT-IDN-03 |
| BR-037 | (مع `member`) |
| BR-038 | (مع `member`) |

---

## 12. Definition of Done

- [ ] Unit + Repository + Auth Flow + Token Tests.
- [ ] RBAC + Isolation Tests.
- [ ] Audit Tests.
- [ ] مفيش Tokens في الـ Logs.
- [ ] كل الاختبارات ناجحة في CI.
