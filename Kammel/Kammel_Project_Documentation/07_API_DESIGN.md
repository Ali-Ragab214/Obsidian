# Kammel — REST API Design & DTO Contracts

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) · [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. REST API Standards & Conventions

All endpoints adhere to RESTful architectural principles, providing predictable URLs, standard HTTP status codes, and consistent payload structures.

### 1.1 Base URL & Versioning
- **Base URI:** `/api/v1`
- **URI Versioning:** Breaking API changes will increment the URI version (e.g., `/api/v2`). Backward-compatible additions (such as adding optional response fields) do not warrant a new URI version.

### 1.2 HTTP Verbs & Standard Status Codes

| HTTP Method | Purpose | Typical Success Codes |
|---|---|---|
| `GET` | Retrieve a resource or filtered collection | `200 OK` |
| `POST` | Create a new resource or trigger a non-idempotent business action | `201 Created`, `200 OK` |
| `PATCH` | Partially update an existing resource | `200 OK` |
| `DELETE` | Soft-delete or deactivate a resource | `204 No Content` |

### 1.3 Request & Response Conventions
1. **Case Format:** `camelCase` for JSON payload keys.
2. **Dates & Timestamps:** ISO-8601 UTC string format (e.g., `2026-11-03T10:15:30Z`).
3. **Monetary Amounts:** Represented in minor units (`Long` piasters) in calculation fields and accompanied by formatted display strings in responses (e.g., `"amountMinor": 70000`, `"amountFormatted": "700.00 EGP"`).
4. **DTO Separation:** JPA entities are strictly encapsulated inside the service/repository layers. Controllers consume `RequestDTO` and return `ResponseDTO`.

---

## 2. Pagination, Sorting & Filtering

### 2.1 Query Parameters
Collection endpoints accept standard Spring Data query parameters:
- `page`: 0-indexed page number (default `0`).
- `size`: Items per page (default `20`, max `100`).
- `sort`: Field and direction (e.g., `sort=createdAt,desc`).
- `search`: Case-insensitive partial string matching (e.g., member name or phone).

### 2.2 Standard Paginated Envelope (`PageResponse<T>`)
```json
{
  "content": [ ... ],
  "page": 0,
  "size": 20,
  "totalElements": 245,
  "totalPages": 13,
  "first": true,
  "last": false
}
```

---

## 3. Global Error Handling & Error Response Schema

All unhandled and business exceptions are intercepted by `GlobalExceptionHandler` (`@RestControllerAdvice`).

### 3.1 Standard Error Payload (`ErrorResponse`)
```json
{
  "timestamp": "2026-11-03T12:00:00Z",
  "status": 400,
  "error": "BAD_REQUEST",
  "message": "Validation failed on input request",
  "path": "/api/v1/members",
  "details": [
    {
      "field": "phone",
      "rejectedValue": "123",
      "message": "Invalid Egyptian mobile number format"
    },
    {
      "field": "fullName",
      "rejectedValue": "",
      "message": "Full name is mandatory"
    }
  ]
}
```

### 3.2 Error Code Mapping

| Exception Class | HTTP Status | Response Error Key |
|---|---|---|
| `MethodArgumentNotValidException` | `400 Bad Request` | `VALIDATION_FAILED` |
| `IllegalArgumentException` | `400 Bad Request` | `INVALID_ARGUMENT` |
| `BadCredentialsException` | `401 Unauthorized` | `AUTHENTICATION_FAILED` |
| `AccessDeniedException` | `403 Forbidden` | `ACCESS_DENIED` |
| `ResourceNotFoundException` | `404 Not Found` | `RESOURCE_NOT_FOUND` |
| `DuplicateResourceException` | `409 Conflict` | `DUPLICATE_RESOURCE` |
| `InvalidStateTransitionException` | `422 Unprocessable Entity` | `INVALID_STATE_TRANSITION` |
| `IdempotencyConflictException` | `409 Conflict` | `IDEMPOTENCY_CONFLICT` |
| `Exception` (catch-all) | `500 Internal Server Error` | `INTERNAL_SERVER_ERROR` |

---

## 4. Feature Endpoints & DTO Contracts

### 4.1 Authentication (`/api/v1/auth`)

#### `POST /api/v1/auth/login`
- **Purpose:** Authenticate staff user and obtain JWT tokens.
- **Authorization:** Public
- **Request (`LoginRequest`):**
  ```json
  {
    "username": "01012345678", // Phone or Email
    "password": "Password123!"
  }
  ```
- **Response `200 OK` (`AuthResponse`):**
  ```json
  {
    "accessToken": "eyJhbGciOiJIUzI1NiIsIn...",
    "refreshToken": "d7a8e9f0-1b2c-3d4e-5f6a...",
    "expiresInSeconds": 900,
    "user": {
      "id": "e2f3a4b5-c6d7-8e9f-0a1b-2c3d4e5f6a7b",
      "name": "Ali Hassan",
      "role": "RECEPTIONIST",
      "orgId": "11111111-2222-3333-4444-555555555555",
      "branchId": "99999999-8888-7777-6666-555555555555"
    }
  }
  ```

#### `POST /api/v1/auth/refresh`
- **Purpose:** Rotate access token using valid refresh token.
- **Request:** `{"refreshToken": "uuid"}`
- **Response `200 OK`:** Same as `AuthResponse`.

#### `POST /api/v1/auth/logout`
- **Purpose:** Invalidate refresh token.
- **Response:** `204 No Content`.

---

### 4.2 Member Management (`/api/v1/members`)

#### `POST /api/v1/members`
- **Purpose:** Register a new gym member.
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')`
- **Request (`CreateMemberRequest`):**
  ```json
  {
    "fullName": "Ahmed Tarek",
    "phone": "01012345678",
    "gender": "MALE",
    "birthDate": "1994-05-12",
    "branchId": "99999999-8888-7777-6666-555555555555",
    "source": "INSTAGRAM",
    "notes": "Interested in morning workouts",
    "whatsappOptIn": true
  }
  ```
- **Validation:**
  - `fullName`: `@NotBlank`, `@Size(min = 2, max = 100)`
  - `phone`: `@NotBlank`, regex `@Pattern(regexp = "^(010|011|012|015)[0-9]{8}$")`
  - `branchId`: `@NotNull`
  - `whatsappOptIn`: `@NotNull`
- **Response `201 Created` (`MemberResponse`):**
  ```json
  {
    "id": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "fullName": "Ahmed Tarek",
    "phone": "01012345678",
    "gender": "MALE",
    "birthDate": "1994-05-12",
    "branchId": "99999999-8888-7777-6666-555555555555",
    "source": "INSTAGRAM",
    "status": "ACTIVE",
    "whatsappOptIn": true,
    "hasActiveSubscription": false,
    "createdAt": "2026-11-03T10:00:00Z"
  }
  ```

#### `GET /api/v1/members`
- **Purpose:** Search and list members with pagination.
- **Query Params:** `search`, `status`, `branchId`, `page`, `size`, `sort`
- **Response `200 OK`:** `PageResponse<MemberSummaryResponse>`.

#### `POST /api/v1/members/import`
- **Purpose:** Batch upload Excel sheet of existing members and historical subscriptions.
- **Payload:** `multipart/form-data` with `file: members.xlsx`
- **Response `200 OK` (`ImportResultResponse`):**
  ```json
  {
    "totalRows": 350,
    "successCount": 348,
    "failureCount": 2,
    "errors": [
      {"rowNumber": 45, "phone": "01000", "error": "Invalid phone format"},
      {"rowNumber": 112, "phone": "01122334455", "error": "Duplicate phone number in file"}
    ]
  }
  ```

---

### 4.3 Membership Plans (`/api/v1/plans`)

#### `POST /api/v1/plans`
- **Purpose:** Create a membership plan template.
- **Authorization:** `hasRole('GYM_OWNER')`
- **Request (`CreatePlanRequest`):**
  ```json
  {
    "name": "Gold 3-Month Membership",
    "type": "QUARTERLY",
    "durationDays": 90,
    "sessionCount": null,
    "priceMinor": 180000, // 1,800.00 EGP
    "freezeMaxDays": 14,
    "graceDays": 5,
    "timeRestriction": null
  }
  ```
- **Response `201 Created` (`PlanResponse`).**

---

### 4.4 Subscriptions (`/api/v1/subscriptions`)

#### `POST /api/v1/subscriptions`
- **Purpose:** Create a new subscription for a member (initial state `PENDING_PAYMENT`).
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')`
- **Request (`CreateSubscriptionRequest`):**
  ```json
  {
    "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "planId": "33333333-4444-5555-6666-777777777777",
    "startDate": "2026-11-05" // Optional, defaults to current date
  }
  ```
- **Response `201 Created` (`SubscriptionResponse`):**
  ```json
  {
    "id": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "planName": "Gold 3-Month Membership",
    "priceMinorSnapshot": 180000,
    "priceFormatted": "1800.00 EGP",
    "startDate": "2026-11-05",
    "endDate": "2027-02-03",
    "status": "PENDING_PAYMENT",
    "sessionsRemaining": null,
    "frozenDaysUsed": 0
  }
  ```

#### `POST /api/v1/subscriptions/{id}/renew`
- **Purpose:** Renew subscription before or after expiration.
- **Request (`RenewSubscriptionRequest`):**
  ```json
  {
    "planId": "33333333-4444-5555-6666-777777777777",
    "overrideStartDate": null
  }
  ```
- **Response `200 OK` (`SubscriptionResponse`).**

#### `POST /api/v1/subscriptions/{id}/freeze`
- **Purpose:** Freeze an active subscription.
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')`
- **Request (`FreezeSubscriptionRequest`):**
  ```json
  {
    "days": 7,
    "reason": "Traveling for work"
  }
  ```
- **Response `200 OK` (`SubscriptionResponse`).**

---

### 4.5 Payments & Collections (`/api/v1/payments`)

#### `POST /api/v1/payments`
- **Purpose:** Record a member payment against a subscription.
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')`
- **Headers:** `Idempotency-Key: <unique-uuid>`
- **Request (`RecordPaymentRequest`):**
  ```json
  {
    "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "amountMinor": 180000,
    "method": "INSTAPAY",
    "reference": "TXN-987654321",
    "notes": "Paid full amount via InstaPay"
  }
  ```
- **Response `201 Created` (`PaymentResponse`):**
  ```json
  {
    "id": "55555555-6666-7777-8888-999999999999",
    "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "amountMinor": 180000,
    "amountFormatted": "1800.00 EGP",
    "method": "INSTAPAY",
    "reference": "TXN-987654321",
    "receiptNumber": "KM-2026-00421",
    "status": "COMPLETED",
    "createdAt": "2026-11-03T11:00:00Z"
  }
  ```

#### `POST /api/v1/payments/{id}/refund`
- **Purpose:** Process a refund (creates reverse accounting record).
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')`
- **Request (`RefundRequest`):**
  ```json
  {
    "amountMinor": 180000,
    "reason": "Member relocated to Alexandria"
  }
  ```
- **Response `200 OK` (`RefundResponse`).**

---

### 4.6 Check-In (`/api/v1/checkins`)

#### `POST /api/v1/checkins`
- **Purpose:** Record member attendance.
- **Request (`CheckInRequest`):**
  ```json
  {
    "identifier": "01012345678", // Phone number, Member Code, or QR Payload
    "method": "PHONE_SEARCH" // NAME_SEARCH, PHONE_SEARCH, STATIC_QR
  }
  ```
- **Response `200 OK` (`CheckInResultResponse`):**
  ```json
  {
    "checkInId": "44444444-5555-6666-7777-888888888888",
    "status": "SUCCESS", // SUCCESS, EXPIRING_SOON_ALERT, DUPLICATE_WARNING
    "memberName": "Ahmed Tarek",
    "planName": "Gold 3-Month Membership",
    "daysRemaining": 4,
    "message": "Welcome Ahmed! Your subscription expires in 4 days."
  }
  ```
- **Error Response `422 Unprocessable Entity` (e.g., Frozen / Expired):**
  ```json
  {
    "status": "FROZEN_BLOCKED",
    "message": "Subscription is frozen until 2026-11-10. Reason: Traveling for work."
  }
  ```

---

### 4.7 WhatsApp Message Queue (`/api/v1/notifications`)

#### `GET /api/v1/notifications/queue`
- **Purpose:** Fetch pending messages for reception staff to send.
- **Query Params:** `dueDate`, `template`
- **Response `200 OK` (`List<MessageQueueItemResponse>`):**
  ```json
  [
    {
      "id": "11112222-3333-4444-5555-666677778888",
      "memberName": "Mohamed Samir",
      "memberPhone": "01099887766",
      "template": "RENEWAL_REMINDER",
      "renderedText": "أهلاً يا محمد، اشتراكك في جولدز جيم بيخلص يوم 10 نوفمبر...",
      "waMeLink": "https://wa.me/201099887766?text=%D8%A3%D9%87%D9%84%D8%A7%D9%8B...",
      "dueDate": "2026-11-03",
      "status": "PENDING"
    }
  ]
  ```

#### `PATCH /api/v1/notifications/{id}/sent`
- **Purpose:** Receptionist confirms message has been transmitted via WhatsApp.
- **Response `200 OK`:** Updated message status (`SENT_CONFIRMED`).

---

### 4.8 Retention & Risk Score (`/api/v1/risk`)

#### `GET /api/v1/risk/at-risk`
- **Purpose:** Retrieve list of members flagged at risk of churn.
- **Authorization:** `hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')`
- **Response `200 OK` (`List<AtRiskMemberResponse>`):**
  ```json
  [
    {
      "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
      "fullName": "Karim Adel",
      "phone": "01234567890",
      "score": 85,
      "riskLevel": "CRITICAL",
      "reasons": [
        "Absent for 13 days (typical frequency: 3x/week)",
        "Attendance dropped by 70% in last 14 days",
        "Subscription expires in 4 days"
      ],
      "recommendedAction": "Contact member today with personalized renewal offer",
      "subscriptionValueMinor": 70000,
      "subscriptionValueFormatted": "700.00 EGP"
    }
  ]
  ```

---

### 4.9 Owner Dashboard & KPIs (`/api/v1/dashboard`)

#### `GET /api/v1/dashboard/summary`
- **Purpose:** Main executive dashboard metrics.
- **Authorization:** `hasRole('GYM_OWNER')`
- **Response `200 OK` (`DashboardSummaryResponse`):**
  ```json
  {
    "renewalsThisWeek": {
      "expiringCount": 18,
      "renewedCount": 7,
      "pendingCount": 11,
      "totalValueMinor": 1260000,
      "totalValueFormatted": "12,600.00 EGP"
    },
    "atRiskSummary": {
      "criticalCount": 8,
      "highCount": 14,
      "totalAtRiskValueFormatted": "15,400.00 EGP"
    },
    "revenueMonthToDateMinor": 8400000,
    "revenueMonthToDateFormatted": "84,000.00 EGP",
    "renewalRatePercentage": 64.5,
    "monthlyChurnPercentage": 7.2
  }
  ```
