# Subscription — تصميم الـ API

> **Base Path:** `/api/v1/subscriptions`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/subscriptions` | Owner, Manager, Receptionist | إنشاء |
| GET | `/subscriptions` | Owner, Manager, Receptionist | قائمة |
| GET | `/subscriptions/{id}` | Owner, Manager, Receptionist | تفاصيل |
| POST | `/subscriptions/{id}/renew` | Owner, Manager, Receptionist | تجديد |
| POST | `/subscriptions/{id}/freeze` | Owner, Manager | تجميد |
| POST | `/subscriptions/{id}/unfreeze` | Owner, Manager | فك تجميد |
| POST | `/subscriptions/{id}/cancel` | Owner, Manager | إلغاء |
| GET | `/subscriptions/{id}/events` | Owner, Manager, Receptionist | Timeline |

---

## 2. `POST /subscriptions`

**Request:**
```json
{
  "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
  "planId": "33333333-4444-5555-6666-777777777777",
  "startDate": "2026-11-05"
}
```

**Validation:**
- `memberId`: `@NotNull` + Member نشط + تابع للـ Org.
- `planId`: `@NotNull` + Plan نشط.
- `startDate`: اختياري — Default اليوم (Cairo).

**Response `201 Created`:**
```json
{
  "success": true,
  "message": "Subscription created",
  "data": {
    "id": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "memberId": "a0b1c2d3-...",
    "planId": "33333333-...",
    "planName": "Gold 3-Month Membership",
    "priceMinorSnapshot": 180000,
    "priceFormatted": "1,800.00 EGP",
    "startDate": "2026-11-05",
    "endDate": "2027-02-03",
    "status": "PENDING_PAYMENT",
    "sessionsRemaining": null,
    "frozenDaysUsed": 0,
    "expiringSoon": false
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

**Errors:** `404 MEMBER_NOT_FOUND`, `422 PLAN_INACTIVE`, `422 MEMBER_INACTIVE`.

---

## 3. `GET /subscriptions`

**Query:** `memberId`, `status`, `planId`, `expiringSoon`, `page`, `size`, `sort`.

**Response:** `PageResponse<SubscriptionSummaryResponse>`.

```json
{
  "id": "77777777-...",
  "memberId": "a0b1c2d3-...",
  "memberName": "Ahmed Tarek",
  "planName": "Gold 3-Month",
  "status": "ACTIVE",
  "startDate": "2026-11-05",
  "endDate": "2027-02-03",
  "daysRemaining": 92,
  "expiringSoon": false,
  "sessionsRemaining": null
}
```

> `memberName` بيتجاب من `MemberQueryService` (Batch) — مش Join.

---

## 4. `GET /subscriptions/{id}`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "77777777-...",
    "memberId": "a0b1c2d3-...",
    "memberName": "Ahmed Tarek",
    "planId": "33333333-...",
    "planName": "Gold 3-Month",
    "priceMinorSnapshot": 180000,
    "priceFormatted": "1,800.00 EGP",
    "startDate": "2026-11-05",
    "endDate": "2027-02-03",
    "status": "ACTIVE",
    "daysRemaining": 92,
    "expiringSoon": false,
    "sessionsRemaining": null,
    "frozenDaysUsed": 0,
    "frozenAt": null,
    "scheduledUnfreezeDate": null,
    "graceEndsAt": null,
    "createdAt": "2026-11-03T10:00:00Z"
  }
}
```

**Errors:** `404 RESOURCE_NOT_FOUND`.

---

## 5. `POST /subscriptions/{id}/renew`

**Request:**
```json
{
  "planId": "33333333-4444-5555-6666-777777777777",
  "overrideStartDate": null,
  "reason": null
}
```

**السلوك:**

| حالة القديم | `new.startDate` | القديم |
|---|---|---|
| `ACTIVE` و`today <= endDate` | `old.endDate` | يفضل `ACTIVE` |
| `GRACE` | `today` | `EXPIRED` |
| `EXPIRED` | `today` | — |
| `PENDING_PAYMENT` / `FROZEN` | ❌ رفض | — |

**Response `201 Created`:** `SubscriptionResponse` للاشتراك **الجديد**، مع:
```json
{
  "renewalOf": "77777777-8888-9999-aaaa-bbbbbbbbbbbb"
}
```

**Rules:**
- `planId` اختياري — Default نفس الباقة.
- `priceMinorSnapshot` من الباقة الحالية.
- `overrideStartDate` للـ Manager/Owner فقط + `reason` إلزامي + Audit.
- Audit `SUBSCRIPTION_RENEWED`.

**Errors:** `422 INVALID_STATE_TRANSITION`, `422 PLAN_INACTIVE`.

---

## 6. `POST /subscriptions/{id}/freeze`

**Request:**
```json
{
  "days": 7,
  "reason": "Traveling for work"
}
```

**Validation:**
- `days`: `@Min(1)`.
- `reason`: `@NotBlank`, ≤ 500.
- `frozenDaysUsed + days <= plan.freezeMaxDays`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Subscription frozen",
  "data": {
    "id": "77777777-...",
    "status": "FROZEN",
    "frozenAt": "2026-11-03T12:00:00Z",
    "scheduledUnfreezeDate": "2026-11-10",
    "frozenDaysUsed": 0
  }
}
```

**Errors:** `422 FREEZE_LIMIT_EXCEEDED`, `422 INVALID_STATE_TRANSITION`.

---

## 7. `POST /subscriptions/{id}/unfreeze`

**Request:**
```json
{ "reason": "Returned early" }
```

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Subscription unfrozen",
  "data": {
    "id": "77777777-...",
    "status": "ACTIVE",
    "endDate": "2027-02-06",
    "frozenDaysUsed": 3
  }
}
```

**Rules:**
- `actualDays = today - frozenAt` (Cairo).
- `endDate += actualDays`.
- `frozenDaysUsed += actualDays`.
- Audit `SUBSCRIPTION_UNFROZEN`.

**Errors:** `422 INVALID_STATE_TRANSITION`.

---

## 8. `POST /subscriptions/{id}/cancel`

**Request:**
```json
{ "reason": "Member relocated" }
```

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Subscription cancelled",
  "data": {
    "id": "77777777-...",
    "status": "CANCELLED",
    "cancelledAt": "2026-11-03T12:00:00Z"
  }
}
```

**Rules:**
- `reason` إلزامي.
- ممنوع من `EXPIRED`.
- مفيش Refund في الـ MVP.
- Audit `SUBSCRIPTION_CANCELLED`.

---

## 9. `GET /subscriptions/{id}/events`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": [
    {
      "id": "evt-1",
      "type": "CREATED",
      "fromStatus": null,
      "toStatus": "PENDING_PAYMENT",
      "actorId": "b2c3d4e5-...",
      "reason": null,
      "metadata": { "planId": "33333333-..." },
      "createdAt": "2026-11-03T10:00:00Z"
    },
    {
      "id": "evt-2",
      "type": "ACTIVATED",
      "fromStatus": "PENDING_PAYMENT",
      "toStatus": "ACTIVE",
      "actorId": null,
      "reason": null,
      "metadata": { "paymentId": "55555555-..." },
      "createdAt": "2026-11-03T11:00:00Z"
    }
  ]
}
```

**Errors:** `404`.

---

## 10. الـ DTOs

```java
public record CreateSubscriptionRequest(
    @NotNull UUID memberId,
    @NotNull UUID planId,
    LocalDate startDate
) {}

public record RenewSubscriptionRequest(
    UUID planId,
    LocalDate overrideStartDate,
    @Size(max = 500) String reason
) {}

public record FreezeSubscriptionRequest(
    @Min(1) int days,
    @NotBlank @Size(max = 500) String reason
) {}

public record CancelSubscriptionRequest(
    @NotBlank @Size(max = 500) String reason
) {}

public record SubscriptionResponse(
    UUID id,
    UUID memberId,
    String memberName,
    UUID planId,
    String planName,
    long priceMinorSnapshot,
    String priceFormatted,
    LocalDate startDate,
    LocalDate endDate,
    SubscriptionStatus status,
    Integer daysRemaining,
    boolean expiringSoon,
    Integer sessionsRemaining,
    int frozenDaysUsed,
    Instant frozenAt,
    LocalDate scheduledUnfreezeDate,
    LocalDate graceEndsAt,
    UUID renewalOf,
    Instant createdAt
) {}

public record SubscriptionEventResponse(
    UUID id,
    SubscriptionEventType type,
    SubscriptionStatus fromStatus,
    SubscriptionStatus toStatus,
    UUID actorId,
    String reason,
    JsonNode metadata,
    Instant createdAt
) {}
```

---

## 11. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
SubscriptionResponse create(CreateSubscriptionRequest request);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
SubscriptionResponse freeze(UUID id, FreezeSubscriptionRequest request);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
SubscriptionResponse cancel(UUID id, CancelSubscriptionRequest request);
```

- Branch Scope: الاشتراك تابع لعضو في فرع المستخدم.
- Trainer → `403`.

---

## 12. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
