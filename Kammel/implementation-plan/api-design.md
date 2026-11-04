# تصميم الـ API العام (Global API Design)

> **المرجع:** [`../Kammel_Project_Documentation/07_API_DESIGN.md`](../Kammel_Project_Documentation/07_API_DESIGN.md)  
> **الملفات المرتبطة:** [shared-foundation](./architecture/shared-foundation.md) · [security-strategy](./architecture/security-strategy.md) · [feature-dependencies](./architecture/feature-dependencies.md)

---

## 1. المبادئ

| المبدأ | القاعدة |
|---|---|
| **Base URL** | `/api/v1` — أي Breaking Change → `/api/v2`. |
| **JSON** | `camelCase` للمفاتيح. |
| **Dates** | `YYYY-MM-DD` للتواريخ، ISO-8601 UTC للـ Timestamps. |
| **Money** | `amountMinor` (`Long` قروش) + `amountFormatted` للعرض. |
| **Tenant** | `orgId` **ممنوع** في أي Request Body — من الـ JWT فقط. |
| **DTOs** | ممنوع Entity تخرج للـ API. |
| **Envelope** | `ApiResponse<T>` للنجاح، `ErrorResponse` للأخطاء. |
| **Idempotency** | إلزامي على `POST /payments`. |
| **Auth** | `Authorization: Bearer <accessToken>` لكل Endpoint ما عدا Auth. |

---

## 2. الـ Envelopes

### 2.1 `ApiResponse<T>`

```json
{
  "success": true,
  "message": "Member registered successfully",
  "data": { "id": "a0b1c2d3-..." },
  "timestamp": "2026-11-03T10:15:00Z"
}
```

| الحالة | الشكل |
|---|---|
| `201 Created` | `ApiResponse<T>` مع `data` |
| `200 OK` | `ApiResponse<T>` مع `data` |
| `204 No Content` | **بدون Body** |
| `4xx/5xx` | `ErrorResponse` |

### 2.2 `PageResponse<T>` (جوه `data`)

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [],
    "page": 0,
    "size": 20,
    "totalElements": 245,
    "totalPages": 13,
    "first": true,
    "last": false
  },
  "timestamp": "2026-11-03T10:15:00Z"
}
```

### 2.3 `ErrorResponse`

```json
{
  "timestamp": "2026-11-03T12:00:00Z",
  "status": 422,
  "error": "INVALID_STATE_TRANSITION",
  "message": "Subscription is already frozen",
  "path": "/api/v1/subscriptions/777.../freeze",
  "traceId": "9f8e7d6c5b4a3210",
  "details": []
}
```

`details[]`:
```json
{ "field": "phone", "rejectedValue": "123", "message": "Invalid Egyptian mobile number" }
```

---

## 3. أكواد الأخطاء الموحدة

| Error Key | HTTP | متى |
|---|:---:|---|
| `VALIDATION_FAILED` | 400 | Bean Validation |
| `INVALID_ARGUMENT` | 400 | Input غير منطقي |
| `MALFORMED_JSON` | 400 | JSON غير صالح |
| `AUTHENTICATION_FAILED` | 401 | Login غلط |
| `TOKEN_EXPIRED` | 401 | Access Token منتهي |
| `TOKEN_INVALID` | 401 | توقيع/صيغة غلط |
| `ACCESS_DENIED` | 403 | Role/Branch غير مسموح |
| `RESOURCE_NOT_FOUND` | 404 | مش موجود **أو** تابع لمستأجر تاني |
| `DUPLICATE_RESOURCE` | 409 | Phone/Email مكرر |
| `IDEMPOTENCY_CONFLICT` | 409 | نفس الـ Key ببيانات مختلفة |
| `INVALID_STATE_TRANSITION` | 422 | State Machine |
| `BUSINESS_RULE_VIOLATION` | 422 | قاعدة بيزنس عامة |
| `SUBSCRIPTION_FROZEN` | 422 | Check-in على اشتراك مجمد |
| `NO_ACTIVE_SUBSCRIPTION` | 422 | مفيش اشتراك صالح |
| `NO_SESSIONS_REMAINING` | 422 | خلصت الجلسات |
| `IMPORT_VALIDATION_FAILED` | 422 | Excel فيه أخطاء |
| `PORTAL_LINK_INVALID` | 401 | Magic Link منتهي/مستخدم/ملغي (P1) |
| `PORTAL_SESSION_INVALID` | 401 | Refresh Token غير صالح (P1) |
| `PORTAL_ACCESS_DENIED` | 401 | Member غير نشط أو Org موقوفة (P1) |
| `ORGANIZATION_NOT_FOUND` | 404 | Slug غير موجود (P1) |
| `RENEWAL_REQUEST_INVALID_STATE` | 422 | Transition غير مسموح (P1) |
| `RATE_LIMITED` | 429 | تجاوز الحد |
| `INTERNAL_SERVER_ERROR` | 500 | خطأ غير متوقع |
| `SERVICE_UNAVAILABLE` | 503 | DB/Pilot Maintenance |

> **قاعدة مهمة:** أي مورد تابع لمستأجر تاني يرجع **404** مش 403 — عشان مانكشفش وجوده.

---

## 4. الـ Pagination والفلترة

| Param | Default | القيود |
|---|---|---|
| `page` | `0` | ≥ 0 |
| `size` | `20` | 1–100 |
| `sort` | `createdAt,desc` | Whitelist per Endpoint |
| `search` | — | Trim + Min 2 chars |

- ممنوع `sort` على عمود غير مسموح → 400.
- ممنوع `Page<T>` من Spring مباشرة.
- الـ `search` بيستخدم Parameter Binding — ممنوع String Concatenation.

---

## 5. كتالوج الـ Endpoints (P0)

### 5.1 Authentication — `/api/v1/auth`

| Method | Path | Auth | الوصف |
|---|---|---|---|
| POST | `/login` | Public | `{organizationSlug, username, password}` → Tokens |
| POST | `/refresh` | Public (Refresh Token) | Rotation |
| POST | `/logout` | Bearer | إلغاء Refresh Token |
| GET | `/me` | Bearer | بيانات المستخدم الحالي |
| PATCH | `/password` | Bearer | تغيير كلمة المرور |

```json
// POST /auth/login
{
  "organizationSlug": "gold-gym-maadi",
  "username": "01012345678",
  "password": "Password123!"
}
```

```json
// 200 OK
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "v1.a1b2c3d4-...<random>",
    "expiresInSeconds": 900,
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

### 5.2 Tenancy — `/api/v1`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/organization` | All | بيانات الـ Org |
| PATCH | `/organization` | Owner | تعديل الاسم |
| GET | `/branches` | All | فروع الـ Org (Owner) / فرعه (غيره) |
| POST | `/branches` | Owner | إنشاء فرع |
| GET | `/branches/{id}` | Owner / Branch | تفاصيل فرع |
| PATCH | `/branches/{id}` | Owner | تعديل فرع |
| POST | `/branches/{id}/deactivate` | Owner | تعطيل فرع |

### 5.3 Users — `/api/v1/users`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/users` | Owner | قائمة الموظفين |
| POST | `/users` | Owner | إنشاء موظف |
| GET | `/users/{id}` | Owner | تفاصيل موظف |
| PATCH | `/users/{id}` | Owner | تعديل |
| POST | `/users/{id}/deactivate` | Owner | تعطيل |
| POST | `/users/{id}/reset-password` | Owner | Reset |

### 5.4 Members — `/api/v1/members`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/members` | Owner, Manager, Receptionist | تسجيل عضو |
| GET | `/members` | Owner, Manager, Receptionist, Trainer | بحث/قائمة (Trainer مقيّد) |
| GET | `/members/{id}` | Owner, Manager, Receptionist, Trainer | تفاصيل |
| PATCH | `/members/{id}` | Owner, Manager, Receptionist | تعديل |
| DELETE | `/members/{id}` | Owner, Manager | Soft Delete |
| GET | `/members/lookup?phone=` | Owner, Manager, Receptionist | بحث سريع |
| POST | `/members/import?dryRun=true` | Owner, Manager | Excel Preview/Commit |
| GET | `/members/import/template` | Owner, Manager | تحميل قالب Excel |

```json
// POST /members
{
  "fullName": "Ahmed Tarek",
  "phone": "01012345678",
  "gender": "MALE",
  "birthDate": "1994-05-12",
  "branchId": "99999999-8888-7777-6666-555555555555",
  "source": "INSTAGRAM",
  "notes": "Prefers morning workouts",
  "whatsappOptIn": true,
  "assignedTrainerId": null
}
```

### 5.5 Plans — `/api/v1/plans`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/plans` | Owner | إنشاء باقة |
| GET | `/plans` | Owner, Manager, Receptionist | قائمة (Active/All) |
| GET | `/plans/{id}` | Owner, Manager, Receptionist | تفاصيل |
| PATCH | `/plans/{id}` | Owner | تعديل |
| POST | `/plans/{id}/deactivate` | Owner | تعطيل |

### 5.6 Subscriptions — `/api/v1/subscriptions`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/subscriptions` | Owner, Manager, Receptionist | إنشاء `PENDING_PAYMENT` |
| GET | `/subscriptions` | Owner, Manager, Receptionist | فلترة `memberId`, `status` |
| GET | `/subscriptions/{id}` | Owner, Manager, Receptionist | تفاصيل |
| POST | `/subscriptions/{id}/renew` | Owner, Manager, Receptionist | تجديد |
| POST | `/subscriptions/{id}/freeze` | Owner, Manager | تجميد |
| POST | `/subscriptions/{id}/unfreeze` | Owner, Manager | فك تجميد |
| POST | `/subscriptions/{id}/cancel` | Owner, Manager | إلغاء |
| GET | `/subscriptions/{id}/events` | Owner, Manager | Timeline |

```json
// POST /subscriptions
{
  "memberId": "a0b1c2d3-...",
  "planId": "33333333-...",
  "startDate": "2026-11-05"
}
```

```json
// POST /subscriptions/{id}/renew
{
  "planId": "33333333-...",
  "overrideStartDate": null,
  "reason": null
}
```

> **سلوك `renew`:**
> - قبل الانتهاء → تمديد نفس الـ Subscription من `end_date`.
> - بعد الانتهاء/في الـ Grace → **Subscription جديد** يبدأ من تاريخ الدفع ([C-11](./architecture/decisions-and-assumptions.md#2-التعارضات-اللي-تم-حلها-resolved-conflicts)).

### 5.7 Payments — `/api/v1/payments`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/payments` | Owner, Manager, Receptionist | تسجيل دفع (Idempotency-Key) |
| GET | `/payments` | Owner, Manager | قائمة |
| GET | `/payments/{id}` | Owner, Manager | تفاصيل |
| GET | `/payments/{id}/receipt` | Owner, Manager, Receptionist | PDF |
| GET | `/payments/collections` | Owner, Manager | متأخرات |

```http
POST /api/v1/payments
Authorization: Bearer ...
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7
Content-Type: application/json
```

```json
{
  "subscriptionId": "77777777-...",
  "amountMinor": 180000,
  "method": "INSTAPAY",
  "reference": "TXN-987654321",
  "notes": "Full payment"
}
```

```json
// 201 Created
{
  "success": true,
  "message": "Payment recorded",
  "data": {
    "id": "55555555-...",
    "subscriptionId": "77777777-...",
    "memberId": "a0b1c2d3-...",
    "amountMinor": 180000,
    "amountFormatted": "1,800.00 EGP",
    "method": "INSTAPAY",
    "reference": "TXN-987654321",
    "receiptNumber": "KM-2026-000421",
    "status": "COMPLETED",
    "subscriptionStatus": "ACTIVE",
    "createdAt": "2026-11-03T11:00:00Z"
  },
  "timestamp": "2026-11-03T11:00:00Z"
}
```

### 5.8 Check-ins — `/api/v1/checkins`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/checkins` | Owner, Manager, Receptionist | تسجيل حضور |
| GET | `/checkins` | Owner, Manager, Receptionist | فلترة `date`, `branchId` |
| GET | `/checkins/today` | Owner, Manager, Receptionist | حضور اليوم |
| GET | `/checkins/lookup?identifier=` | Owner, Manager, Receptionist | بحث بدون تسجيل |

```json
// POST /checkins
{
  "identifier": "01012345678",
  "method": "PHONE_SEARCH",
  "overrideReason": null
}
```

```json
// 200 OK
{
  "success": true,
  "message": "Welcome Ahmed!",
  "data": {
    "checkInId": "44444444-...",
    "status": "SUCCESS",
    "memberName": "Ahmed Tarek",
    "planName": "Gold 3-Month",
    "daysRemaining": 24,
    "sessionsRemaining": null,
    "warning": null
  },
  "timestamp": "2026-11-03T18:15:00Z"
}
```

```json
// 422 SUBSCRIPTION_FROZEN
{
  "timestamp": "2026-11-03T18:15:00Z",
  "status": 422,
  "error": "SUBSCRIPTION_FROZEN",
  "message": "Subscription is frozen until 2026-11-10. Reason: Traveling for work.",
  "path": "/api/v1/checkins",
  "traceId": "...",
  "details": []
}
```

### 5.9 Risk — `/api/v1/risk`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/risk/at-risk` | Owner, Manager | قائمة At-Risk (فلترة level/branch) |
| GET | `/risk/members/{memberId}` | Owner, Manager, Trainer (عميله) | آخر Score + Reasons |
| GET | `/risk/summary` | Owner, Manager | توزيع المستويات |
| POST | `/risk/recompute` | Owner | إعادة حساب يدوية (اختياري) |

### 5.10 Notifications — `/api/v1/notifications`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/notifications/queue` | Owner, Manager, Receptionist | طابور اليوم |
| GET | `/notifications/{id}` | Owner, Manager, Receptionist | تفاصيل |
| PATCH | `/notifications/{id}/opened` | Owner, Manager, Receptionist | فتح `wa.me` (مش إرسال) |
| PATCH | `/notifications/{id}/sent` | Owner, Manager, Receptionist | تأكيد الإرسال |
| PATCH | `/notifications/{id}/cancel` | Owner, Manager | إلغاء |
| GET | `/notifications/attribution` | Owner | ROI Attribution |

### 5.11 Reporting — `/api/v1/dashboard`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/dashboard/summary` | Owner | KPIs رئيسية |
| GET | `/dashboard/renewals` | Owner, Manager | تجديدات الأسبوع |
| GET | `/dashboard/collections` | Owner, Manager | متأخرات |
| GET | `/dashboard/attendance` | Owner, Manager | حضور حسب الفترة |
| GET | `/dashboard/risk` | Owner, Manager | توزيع المخاطر |
| GET | `/dashboard/members/{id}/timeline` | Owner, Manager, Receptionist | اشتراكات + مدفوعات + حضور |

> **Manager:** يشوف بيانات فرعه فقط (`branchId` إلزامي أو مشتق من الـ Token).

### 5.12 Audit — `/api/v1/audit-logs`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/audit-logs` | Owner | فلترة `actorId`, `action`, `entityType`, `from`, `to` |
| GET | `/audit-logs/{id}` | Owner | تفاصيل |
| GET | `/audit-logs/entities/{entityType}/{entityId}` | Owner | تاريخ كيان |

### 5.13 Health & Ops

| Method | Path | Auth | الوصف |
|---|---|---|---|
| GET | `/actuator/health` | Public (Internal) | Liveness/Readiness |
| GET | `/actuator/info` | Public (Internal) | Build Info |
| GET | `/actuator/metrics` | Internal فقط | Prometheus/Micrometer |

> `/actuator/*` بيتقفل من بره عبر Nginx ما عدا `/health` للـ Load Balancer.

### 5.14 P1 Endpoints (Post-MVP)

#### Shift — `/api/v1/shifts`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/shifts/open` | Owner, Manager, Receptionist | فتح وردية |
| GET | `/shifts/current` | Owner, Manager, Receptionist | الوردية المفتوحة (Blind Count) |
| POST | `/shifts/{id}/close` | Owner, Manager, Receptionist (صاحبها) | إغلاق |
| POST | `/shifts/{id}/force-close` | Owner, Manager | إغلاق إجباري |
| GET | `/shifts` | Owner, Manager | سجل الورديات |
| GET | `/shifts/{id}` | Owner, Manager | تفاصيل |

#### Lead — `/api/v1/leads`

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/leads` | Owner, Manager, Receptionist | إنشاء |
| GET | `/leads` | Owner, Manager, Receptionist | قائمة |
| GET | `/leads/{id}` | Owner, Manager, Receptionist | تفاصيل |
| PATCH | `/leads/{id}/status` | Owner, Manager, Receptionist | تغيير الحالة |
| POST | `/leads/{id}/follow-ups` | Owner, Manager, Receptionist | إضافة متابعة |
| GET | `/leads/{id}/follow-ups` | Owner, Manager, Receptionist | متابعات |
| POST | `/leads/{id}/convert` | Owner, Manager, Receptionist | تحويل لعضو |
| GET | `/leads/funnel` | Owner, Manager | قمع المبيعات |

#### Member Portal — `/api/v1/portal`

| Method | Path | Auth | الوصف |
|---|---|---|---|
| GET | `/portal/branding?slug=` | Public | هوية الجيم |
| POST | `/portal/auth/request-link` | Public + Rate Limit | طلب Magic Link |
| POST | `/portal/auth/verify` | Public + Rate Limit | Verify → Session |
| POST | `/portal/auth/refresh` | Public | Rotation |
| POST | `/portal/auth/logout` | Member | Logout |
| GET | `/portal/me` | Member | بيانات العضو |
| GET | `/portal/subscription` | Member | الاشتراك الحالي + التاريخ |
| GET | `/portal/qr` | Member | QR Payload |
| GET | `/portal/qr/image` | Member | QR PNG |
| GET | `/portal/checkins` | Member | الحضور + Streak |
| GET | `/portal/payments` | Member | المدفوعات |
| GET | `/portal/payments/{id}/receipt` | Member | إيصال PDF |
| POST | `/portal/renewal-requests` | Member | طلب تجديد |
| GET | `/portal/renewal-requests` | Member | طلباته |
| GET | `/portal/admin/renewal-requests` | Owner, Manager, Receptionist | Queue الاستقبال |
| PATCH | `/portal/admin/renewal-requests/{id}` | Owner, Manager, Receptionist | تحديث الحالة |

> **Member Portal:** كل الـ Endpoints بتستخدم `scope = MEMBER` — ممنوع Staff Token، وممنوع Portal Token على `/api/v1/**` العادية.

---

## 6. قواعد الـ Request/Response

### 6.1 المبالغ

| الحقل | النوع | مثال |
|---|---|---|
| `amountMinor` | `Long` | `180000` |
| `amountFormatted` | `String` (Presentation فقط) | `"1,800.00 EGP"` |

- الـ Client يعتمد على `*Minor` في أي حساب.
- ممنوع `double` في الـ JSON Deserialization — `Long` فقط.

### 6.2 الـ Enums

- بتتبعت كـ `UPPER_SNAKE_CASE`.
- أي قيمة غير معروفة → `400 VALIDATION_FAILED`.
- ممنوع ترجمة قيم الـ Enum في الـ API.

### 6.3 الـ Nullability

- الحقول الاختيارية بتتحط `null` صريح، مش بتتشال.
- `PATCH` بيستخدم `JsonNullable` أو `Optional` — عشان نفرق بين "مش مبعوت" و"اتبعت null".

### 6.4 الـ Concurrency

| السيناريو | الحل |
|---|---|
| تعديل Subscription في نفس الوقت | `SELECT ... FOR UPDATE` على Subscription |
| تسجيل Payment متزامن | Idempotency + Row Lock |
| رقم إيصال | `document_sequences` Atomic |
| خصم جلسة | Row Lock + `sessions_remaining > 0` |

---

## 7. الـ OpenAPI

- SpringDoc OpenAPI 3.
- Swagger UI متاح في `dev`/`staging` فقط.
- كل Endpoint عليه:
  - `@Operation(summary, description)`.
  - `@ApiResponses` لكل الحالات.
  - `@SecurityRequirement(name = "bearerAuth")`.
- الـ Schemas بتتولد من الـ DTOs — ممنوع كتابة YAML يدوي.
- الـ Contract Snapshot بيتحفظ في `src/test/resources/openapi/` ويتقارن في CI.

---

## 8. الـ Versioning والـ Deprecation

| التغيير | الإجراء |
|---|---|
| إضافة حقل اختياري | نفس الإصدار |
| إضافة Endpoint | نفس الإصدار |
| تغيير نوع حقل | `/v2` |
| حذف حقل | `/v2` |
| تغيير معنى Enum | `/v2` |

- أي Endpoint deprecated → Header `Deprecation: true` + `Sunset` + توثيق.
- مدة دعم الإصدار القديم: 90 يوم على الأقل.

---

## 9. مرتبط بـ

- [api-design لكل Feature](./features/) — التفاصيل والـ Payloads.
- [security-strategy](./architecture/security-strategy.md) — الصلاحيات والـ Rate Limiting.
- [testing-strategy](./testing-strategy.md) — API Tests.
- [decisions-and-assumptions](./architecture/decisions-and-assumptions.md) — C-15 (Envelope).
