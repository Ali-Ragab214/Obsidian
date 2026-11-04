# Identity — تسجيل الدخول والموظفين والصلاحيات

> **الـ Phase:** 1 · **الأولوية:** P0 · **الترتيب التنفيذي:** 2  
> **الـ Package:** `com.kammel.features.identity`  
> **المرجع:** [`../../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md`](../../../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. الميزة دي بتعمل إيه؟

`identity` هي المسؤولة عن **دخول الموظفين** (مش الأعضاء):

- `User` — حساب الموظف (Owner / Manager / Receptionist / Trainer).
- `RefreshToken` — Rotation + Revocation.
- `JWT` — Access Token قصير العمر.
- إدارة الموظفين (إنشاء/تعديل/تعطيل/Reset Password).
- الأدوار والصلاحيات الأساسية.
- Account Lockout.

**مش مسؤولة عن:**
- تسجيل دخول الأعضاء (PWA) → `member-portal` (P1).
- إنشاء الجيمات → `tenancy` Provisioning.
- صلاحيات الفروع التفصيلية → `tenancy` + `shared/security`.
- 2FA / SSO / OTP → P1.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `User` | موظف بيشتغل على النظام ويسجل دخول. |
| `Member` | عميل الجيم — **مش** User ([D-12](../../architecture/decisions-and-assumptions.md#d-12-member--user)). |
| `Access Token` | JWT قصير العمر (15 دقيقة). |
| `Refresh Token` | Token طويل العمر (7 أيام) مخزّن كـ Hash. |
| `Rotation` | كل Refresh بيصدر Token جديد ويلغي القديم. |
| `organizationSlug` | معرف الجيم في شاشة الدخول. |
| `mustChangePassword` | إجبار تغيير كلمة المرور المؤقتة. |

---

## 3. النطاق (Scope)

### داخل الـ MVP
- [x] Login بـ `organizationSlug` + `phone/email` + `password`.
- [x] Access Token (JWT HS256, 15m).
- [x] Refresh Token Rotation (7d, Hash in DB).
- [x] Logout + Revocation.
- [x] `GET /auth/me`.
- [x] تغيير كلمة المرور.
- [x] إدارة الموظفين بواسطة الـ Owner.
- [x] تعطيل موظف + إلغاء Tokens.
- [x] Reset Password بواسطة الـ Owner.
- [x] Account Lockout بعد 5 محاولات.
- [x] الأدوار الأربعة.
- [x] `mustChangePassword` flow.

### خارج الـ MVP
- [ ] 2FA / OTP.
- [ ] SSO / Google Login.
- [ ] Member Login.
- [ ] `PLATFORM_ADMIN`.
- [ ] Forgot Password عن طريق Email.
- [ ] Session Management UI.
- [ ] Password History.

---

## 4. الأدوار

| Role | النطاق | `branch_id` |
|---|---|---|
| `GYM_OWNER` | كل الـ Org | `NULL` |
| `BRANCH_MANAGER` | فرع واحد | `NOT NULL` |
| `RECEPTIONIST` | فرع واحد | `NOT NULL` |
| `TRAINER` | فرع واحد + عملاؤه | `NOT NULL` |

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-008 | Manager يشتغل على فرعه فقط. |
| BR-009 | Manager ممنوع يشوف فرع تاني. |
| BR-037 | Trainer يشوف عملاءه فقط. |
| BR-038 | Trainer ممنوع يشوف بيانات مالية. |
| IDN-01 | مفيش Self-Registration — الـ Owner فقط ينشئ موظفين. |
| IDN-02 | `(org_id, phone)` و`(org_id, email)` فريدين. |
| IDN-03 | `GYM_OWNER` → `branch_id = NULL`؛ غيره → `NOT NULL`. |
| IDN-04 | Password Hash بـ BCrypt Strength 12. |
| IDN-05 | 5 محاولات فاشلة → قفل 15 دقيقة. |
| IDN-06 | تعطيل مستخدم → إلغاء كل Refresh Tokens. |
| IDN-07 | تغيير كلمة المرور → إلغاء كل Refresh Tokens. |
| IDN-08 | Refresh Reuse → إلغاء كل العائلة + Audit. |
| IDN-09 | Org `SUSPENDED`/`CLOSED` → Login مرفوض. |
| IDN-10 | Access Token 15 دقيقة — مفيش Denylist في الـ MVP. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `JwtService`, `TenantContext`, `ApiResponse` |
| `tenancy` | Compile | `OrganizationQueryService`, `BranchQueryService` |
| `audit` | Compile | `AuditService.record(...)` |

**مين بيعتمد على `identity`؟**
`audit`, `member`, `subscription`, `payment`, `checkin`, `notification`, `reporting`.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | `User`, `RefreshToken`, القواعد. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] Migrations `users` + `refresh_tokens` + RLS.
- [ ] `User` + `RefreshToken` Entities.
- [ ] `JwtService` + `JwtAuthenticationFilter`.
- [ ] Login / Refresh / Logout.
- [ ] Rotation + Reuse Detection.
- [ ] User Management APIs.
- [ ] Password Change + Reset.
- [ ] Lockout.
- [ ] Security Tests + Isolation Tests.
- [ ] Audit لكل العمليات الحساسة.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| هل الـ Owner يقدر يكون له أكثر من حساب؟ | نعم — لكن كلهم `GYM_OWNER`. |
| هل نسمح بـ Email Login؟ | نعم — `phone` أو `email`. |
| هل كلمة المرور المؤقتة تظهر في الـ API؟ | مرة واحدة فقط عند الإنشاء/Reset. |
| مدة القفل؟ | 15 دقيقة بعد 5 محاولات. |
