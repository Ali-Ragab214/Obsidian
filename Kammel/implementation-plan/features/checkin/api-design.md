# Check-in — تصميم الـ API

> **Base Path:** `/api/v1/checkins`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/checkins` | Owner, Manager, Receptionist | تسجيل حضور |
| GET | `/checkins` | Owner, Manager, Receptionist | قائمة الحضور |
| GET | `/checkins/today` | Owner, Manager, Receptionist | حضور اليوم |
| GET | `/checkins/lookup` | Owner, Manager, Receptionist | بحث بدون تسجيل |

---

## 2. `POST /checkins`

**Request:**
```json
{
  "identifier": "01012345678",
  "method": "PHONE_SEARCH",
  "branchId": null,
  "confirmDuplicate": false,
  "override": false,
  "overrideReason": null
}
```

**Validation:**

| الحقل | القاعدة |
|---|---|
| `identifier` | `@NotBlank`, 3–100 |
| `method` | `@NotNull` + من الـ Enum |
| `branchId` | مطلوب للـ Owner فقط |
| `confirmDuplicate` | Default `false` |
| `override` | Default `false` |
| `overrideReason` | إلزامي مع `confirmDuplicate=true` أو `override=true` |

**Response `200 OK` — Success:**
```json
{
  "success": true,
  "message": "Welcome Ahmed!",
  "data": {
    "checkInId": "44444444-5555-6666-7777-888888888888",
    "status": "SUCCESS",
    "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "memberName": "Ahmed Tarek",
    "memberCode": "M-000123",
    "planName": "Gold 3-Month Membership",
    "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "daysRemaining": 24,
    "sessionsRemaining": null,
    "warning": null,
    "requiresConfirmation": false,
    "firstCheckInAt": null,
    "duplicateConfirmed": false,
    "overridden": false,
    "checkedInAt": "2026-11-03T18:15:00Z"
  },
  "timestamp": "2026-11-03T18:15:00Z"
}
```

**Response `200 OK` — Grace Warning:**
```json
{
  "success": true,
  "message": "Welcome Ahmed! Your subscription is in grace period until 2026-11-08.",
  "data": {
    "status": "GRACE_WARNING",
    "daysRemaining": -2,
    "warning": "فترة السماح تنتهي يوم 2026-11-08. جدّد عشان ماتخسرش أيام.",
    "...": "..."
  }
}
```

**Response `200 OK` — Duplicate Warning (بدون تسجيل):**
```json
{
  "success": true,
  "message": "Ahmed already checked in today at 10:15 AM.",
  "data": {
    "checkInId": null,
    "status": "DUPLICATE_WARNING",
    "memberId": "a0b1c2d3-...",
    "memberName": "Ahmed Tarek",
    "firstCheckInAt": "2026-11-03T08:15:00Z",
    "requiresConfirmation": true,
    "warning": "العضو مسجل حضور النهارده الساعة 10:15 صباحاً. تأكد قبل التسجيل تاني.",
    "duplicateConfirmed": false,
    "overridden": false
  }
}
```

**Response `200 OK` — Duplicate Confirmed:**
```json
{
  "success": true,
  "message": "Duplicate check-in confirmed",
  "data": {
    "checkInId": "44444444-...",
    "status": "SUCCESS",
    "duplicateConfirmed": true,
    "overridden": true,
    "warning": "تم تأكيد الحضور المكرر بواسطة Manager.",
    "...": "..."
  }
}
```

**Response `200 OK` — Access Override:**
```json
{
  "success": true,
  "message": "Override check-in recorded",
  "data": {
    "checkInId": "44444444-...",
    "status": "SUCCESS",
    "overridden": true,
    "warning": "تم تسجيل دخول استثنائي. الاشتراك منتهي.",
    "...": "..."
  }
}
```

**Errors:**

| الحالة | Error | Details |
|---|---|---|
| `404` | `MEMBER_NOT_FOUND` | — |
| `409` | `AMBIGUOUS_MEMBER` | `candidates[]` |
| `400` | `INVALID_IDENTIFIER_METHOD` | — |
| `403` | `BRANCH_ACCESS_DENIED` | — |
| `422` | `BRANCH_INACTIVE` | — |
| `422` | `NO_ACTIVE_SUBSCRIPTION` | `outstandingMinor?` |
| `422` | `SUBSCRIPTION_FROZEN` | `frozenUntil`, `reason` |
| `422` | `SUBSCRIPTION_EXPIRED` | `expiredOn` |
| `422` | `NO_SESSIONS_REMAINING` | — |
| `403` | `OVERRIDE_NOT_ALLOWED` | — |

**مثال `422 SUBSCRIPTION_FROZEN`:**
```json
{
  "timestamp": "2026-11-03T18:15:00Z",
  "status": 422,
  "error": "SUBSCRIPTION_FROZEN",
  "message": "Subscription is frozen until 2026-11-10. Reason: Traveling for work.",
  "path": "/api/v1/checkins",
  "traceId": "...",
  "details": [
    { "field": "frozenUntil", "rejectedValue": "2026-11-10", "message": "Freeze end date" },
    { "field": "reason", "rejectedValue": "Traveling for work", "message": "Freeze reason" }
  ]
}
```

---

## 3. `GET /checkins`

**Query:** `date`, `from`, `to`, `branchId`, `memberId`, `method`, `overridden`, `page`, `size`, `sort`.

**Response `200 OK`:** `PageResponse<CheckInSummaryResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "id": "44444444-...",
        "memberId": "a0b1c2d3-...",
        "memberName": "Ahmed Tarek",
        "memberCode": "M-000123",
        "branchId": "99999999-...",
        "branchName": "Maadi",
        "subscriptionId": "77777777-...",
        "planName": "Gold 3-Month",
        "method": "PHONE_SEARCH",
        "checkedInAt": "2026-11-03T18:15:00Z",
        "overridden": false
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1,
    "first": true,
    "last": true
  }
}
```

**Rules:**
- `from` / `to` اختياريين — Default اليوم.
- المدى الأقصى 90 يوم.
- Manager/Receptionist → فرعهم فقط.

---

## 4. `GET /checkins/today`

**Query:** `branchId` (Owner فقط).

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "date": "2026-11-03",
    "totalCheckIns": 47,
    "uniqueMembers": 42,
    "overriddenCount": 2,
    "duplicateCount": 3,
    "branchBreakdown": [
      { "branchId": "99999999-...", "branchName": "Maadi", "count": 30 }
    ]
  }
}
```

**Rules:**
- اليوم بتوقيت `Africa/Cairo`.
- Manager → فرعه فقط.
- `duplicateCount` = Check-ins اللي حصلت بعد أول Check-in لنفس العضو في نفس اليوم.

---

## 5. `GET /checkins/lookup`

**Query:** `identifier`, `method`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "memberId": "a0b1c2d3-...",
    "memberName": "Ahmed Tarek",
    "memberCode": "M-000123",
    "phone": "01012345678",
    "status": "ACTIVE",
    "hasActiveSubscription": true,
    "subscription": {
      "id": "77777777-...",
      "planName": "Gold 3-Month",
      "status": "ACTIVE",
      "startDate": "2026-09-05",
      "endDate": "2026-12-03",
      "daysRemaining": 30,
      "sessionsRemaining": null,
      "expiringSoon": false,
      "frozenUntil": null,
      "freezeReason": null
    },
    "checkedInToday": false,
    "firstCheckInAt": null,
    "warnings": []
  }
}
```

**Rules:**
- **مش بيسجل حضور** — للعرض بس.
- لو العضو مالوش اشتراك → `subscription: null` + `warnings: ["NO_ACTIVE_SUBSCRIPTION"]`.
- لو مجمد → `warnings: ["FROZEN"]` + البيانات.
- Receptionist يقدر يستخدمه.

---

## 6. الـ DTOs

```java
public record CheckInRequest(
    @NotBlank @Size(min = 3, max = 100) String identifier,
    @NotNull CheckInMethod method,
    UUID branchId,
    boolean confirmDuplicate,
    boolean override,
    @Size(max = 500) String overrideReason
) {}

public record CheckInResponse(
    UUID checkInId,
    CheckInResultStatus status,
    UUID memberId,
    String memberName,
    String memberCode,
    String planName,
    UUID subscriptionId,
    Integer daysRemaining,
    Integer sessionsRemaining,
    String warning,
    boolean requiresConfirmation,
    Instant firstCheckInAt,
    boolean duplicateConfirmed,
    boolean overridden,
    Instant checkedInAt
) {}

public record CheckInSummaryResponse(
    UUID id,
    UUID memberId,
    String memberName,
    String memberCode,
    UUID branchId,
    String branchName,
    UUID subscriptionId,
    String planName,
    CheckInMethod method,
    Instant checkedInAt,
    boolean overridden
) {}

public record CheckInLookupResponse(
    UUID memberId,
    String memberName,
    String memberCode,
    String phone,
    String status,
    boolean hasActiveSubscription,
    LookupSubscription subscription,
    boolean checkedInToday,
    Instant firstCheckInAt,
    List<String> warnings
) {}

public record LookupSubscription(
    UUID id,
    String planName,
    String status,
    LocalDate startDate,
    LocalDate endDate,
    Integer daysRemaining,
    Integer sessionsRemaining,
    boolean expiringSoon,
    LocalDate frozenUntil,
    String freezeReason
) {}
```

---

## 7. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
CheckInResponse checkIn(CheckInRequest request);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PageResponse<CheckInSummaryResponse> list(CheckInFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
TodayStatsResponse today(UUID branchId);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
CheckInLookupResponse lookup(String identifier, CheckInMethod method);
```

- Trainer → `403`.
- Override → فحص إضافي جوه الـ Service (`OWNER`/`MANAGER` فقط).

---

## 8. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
