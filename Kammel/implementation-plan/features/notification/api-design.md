# Notification — تصميم الـ API

> **Base Path:** `/api/v1/notifications`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/notifications/queue` | Owner, Manager, Receptionist | طابور اليوم |
| GET | `/notifications/{id}` | Owner, Manager, Receptionist | تفاصيل رسالة |
| PATCH | `/notifications/{id}/opened` | Owner, Manager, Receptionist | فتح `wa.me` |
| PATCH | `/notifications/{id}/sent` | Owner, Manager, Receptionist | تأكيد الإرسال |
| PATCH | `/notifications/{id}/cancel` | Owner, Manager | إلغاء |
| GET | `/notifications/attribution` | Owner | تقرير الـ ROI |

---

## 2. `GET /notifications/queue`

**Query:** `dueDate` (Default اليوم), `template`, `status` (Default `PENDING`), `branchId` (Owner فقط), `page`, `size`, `sort`.

**Response `200 OK` — داخل النافذة:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "id": "11112222-3333-4444-5555-666677778888",
        "memberId": "a0b1c2d3-...",
        "memberName": "Mohamed Samir",
        "memberPhone": "01099887766",
        "branchId": "99999999-...",
        "branchName": "Maadi",
        "template": "RENEWAL_REMINDER",
        "renderedText": "أهلاً يا محمد 👋\nاشتراكك في جولدز جيم هينتهي يوم 2026-11-10...",
        "waMeLink": "https://wa.me/201099887766?text=%D8%A3%D9%87%D9%84%D8%A7...",
        "status": "PENDING",
        "dueDate": "2026-11-03",
        "openedAt": null,
        "sentAt": null,
        "attributedSubscriptionId": null,
        "createdAt": "2026-11-03T04:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 18,
    "totalPages": 1,
    "first": true,
    "last": true
  },
  "timestamp": "2026-11-03T09:15:00Z"
}
```

**Response `200 OK` — بره النافذة (قبل 9 أو بعد 9 مساءً):**
```json
{
  "success": true,
  "message": "Queue is available from 09:00 to 21:00 Africa/Cairo",
  "data": {
    "content": [],
    "page": 0,
    "size": 20,
    "totalElements": 0,
    "totalPages": 0,
    "first": true,
    "last": true,
    "availableAt": "2026-11-04T09:00:00+02:00"
  },
  "timestamp": "2026-11-03T22:30:00Z"
}
```

**Rules:**
- `dueDate` Default اليوم (Cairo).
- `status` Default `PENDING` + `OPENED` (اللي لسه محتاج تأكيد).
- Manager/Receptionist → فرعهم فقط.
- بره النافذة → `content: []` + `availableAt`.
- Trainer → `403`.

---

## 3. `GET /notifications/{id}`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "11112222-...",
    "memberId": "a0b1c2d3-...",
    "memberName": "Mohamed Samir",
    "memberPhone": "01099887766",
    "memberCode": "M-000789",
    "branchId": "99999999-...",
    "branchName": "Maadi",
    "template": "GRACE_EXPOSURE",
    "renderedText": "أهلاً يا محمد،\nاشتراكك انتهى، لكن إحنا في فترة السماح...",
    "waMeLink": "https://wa.me/201099887766?text=...",
    "status": "OPENED",
    "dueDate": "2026-11-03",
    "openedAt": "2026-11-03T09:20:00Z",
    "sentAt": null,
    "sentBy": null,
    "sentByName": null,
    "attributedSubscriptionId": null,
    "createdAt": "2026-11-03T04:00:00Z"
  }
}
```

**Errors:** `404 MESSAGE_NOT_FOUND`, `403` لو فرع تاني.

---

## 4. `PATCH /notifications/{id}/opened`

**Request:** لا يوجد Body.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Message opened",
  "data": {
    "id": "11112222-...",
    "status": "OPENED",
    "openedAt": "2026-11-03T09:20:00Z"
  }
}
```

**Rules:**
- `PENDING → OPENED` فقط.
- لو `OPENED` بالفعل → `200` Idempotent.
- لو `SENT_CONFIRMED` / `CANCELLED` → `409`.
- بره 9–21 → `422 OUTSIDE_SEND_WINDOW`.
- العضو `whatsappOptIn = false` → `422 MEMBER_OPTED_OUT`.

**Errors:** `404`, `409 INVALID_MESSAGE_STATE`, `422 OUTSIDE_SEND_WINDOW`, `422 MEMBER_OPTED_OUT`.

---

## 5. `PATCH /notifications/{id}/sent`

**Request:** لا يوجد Body.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Message marked as sent",
  "data": {
    "id": "11112222-...",
    "status": "SENT_CONFIRMED",
    "sentAt": "2026-11-03T09:22:00Z",
    "sentBy": "b2c3d4e5-..."
  }
}
```

**Rules:**
- `OPENED → SENT_CONFIRMED` فقط.
- لو `SENT_CONFIRMED` بالفعل → `200` Idempotent.
- لو `PENDING` → `409` — لازم `opened` الأول.
- بره 9–21 → `422 OUTSIDE_SEND_WINDOW`.
- Audit `MESSAGE_SENT`.
- `MessageSentEvent` بعد الـ Commit.

**Errors:** `404`, `409 INVALID_MESSAGE_STATE`, `422`.

---

## 6. `PATCH /notifications/{id}/cancel`

**Request:**
```json
{
  "reason": "Member asked us to stop messaging"
}
```

**Validation:**
- `reason`: `@NotBlank`, ≤ 500.
- Manager/Owner فقط.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Message cancelled",
  "data": {
    "id": "11112222-...",
    "status": "CANCELLED"
  }
}
```

**Rules:**
- `PENDING` / `OPENED` → `CANCELLED`.
- `SENT_CONFIRMED` → `409`.
- Audit `MESSAGE_CANCELLED` + Reason.

---

## 7. `GET /notifications/attribution`

**Query:** `from`, `to`, `branchId` (اختياري), `template`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "from": "2026-10-01",
    "to": "2026-10-31",
    "sentCount": 180,
    "attributedCount": 42,
    "attributedValueMinor": 9800000,
    "attributedValueFormatted": "98,000.00 EGP",
    "attributionRate": 23.3,
    "byTemplate": [
      {
        "template": "RENEWAL_REMINDER",
        "sentCount": 120,
        "attributedCount": 34,
        "attributedValueMinor": 8120000,
        "attributedValueFormatted": "81,200.00 EGP"
      },
      {
        "template": "MISS_YOU",
        "sentCount": 60,
        "attributedCount": 8,
        "attributedValueMinor": 1680000,
        "attributedValueFormatted": "16,800.00 EGP"
      }
    ]
  }
}
```

**Rules:**
- Owner فقط.
- `from` / `to` إلزاميين — أقصى مدى 366 يوم.
- `attributionRate` = `attributedCount / sentCount * 100` — `0` لو `sentCount = 0`.
- `attributedValueMinor` = مجموع `price_minor_snapshot` للتجديدات المربوطة.
- Manager → `403` (C-14: مش بيشوف Org Revenue).

---

## 8. الـ DTOs

```java
public record MessageQueueItemResponse(
    UUID id,
    UUID memberId,
    String memberName,
    String memberPhone,
    UUID branchId,
    String branchName,
    MessageTemplate template,
    String renderedText,
    String waMeLink,
    MessageStatus status,
    LocalDate dueDate,
    Instant openedAt,
    Instant sentAt,
    UUID sentBy,
    String sentByName,
    UUID attributedSubscriptionId,
    Instant createdAt
) {}

public record MessageStatusResponse(
    UUID id,
    MessageStatus status,
    Instant openedAt,
    Instant sentAt,
    UUID sentBy
) {}

public record CancelMessageRequest(
    @NotBlank @Size(max = 500) String reason
) {}

public record AttributionResponse(
    LocalDate from,
    LocalDate to,
    long sentCount,
    long attributedCount,
    long attributedValueMinor,
    String attributedValueFormatted,
    double attributionRate,
    List<TemplateAttributionResponse> byTemplate
) {}

public record TemplateAttributionResponse(
    MessageTemplate template,
    long sentCount,
    long attributedCount,
    long attributedValueMinor,
    String attributedValueFormatted
) {}
```

---

## 9. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PageResponse<MessageQueueItemResponse> queue(MessageFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
MessageQueueItemResponse get(UUID id);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
MessageStatusResponse markOpened(UUID id);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
MessageStatusResponse markSent(UUID id);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
MessageStatusResponse cancel(UUID id, CancelMessageRequest request);

@PreAuthorize("hasRole('GYM_OWNER')")
AttributionResponse attribution(LocalDate from, LocalDate to, UUID branchId, MessageTemplate template);
```

- Branch Scope جوه الـ Service.
- Trainer → `403`.

---

## 10. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
