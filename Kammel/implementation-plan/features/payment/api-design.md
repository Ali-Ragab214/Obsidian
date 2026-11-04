# Payment — تصميم الـ API

> **Base Path:** `/api/v1/payments`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| POST | `/payments` | Owner, Manager, Receptionist | تسجيل دفع |
| GET | `/payments` | Owner, Manager | قائمة المدفوعات |
| GET | `/payments/{id}` | Owner, Manager, Receptionist | تفاصيل دفعة |
| GET | `/payments/{id}/receipt` | Owner, Manager, Receptionist | إيصال PDF |
| GET | `/payments/collections` | Owner, Manager | طابور التحصيل |

---

## 2. `POST /payments`

**Headers:**

| Header | إلزامي | الوصف |
|---|:---:|---|
| `Authorization` | ✅ | `Bearer <JWT>` |
| `Idempotency-Key` | ✅ | 8–100 حرف — فريد لكل Org |
| `Content-Type` | ✅ | `application/json` |

**Request:**
```json
{
  "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
  "amountMinor": 180000,
  "method": "INSTAPAY",
  "reference": "TXN-987654321",
  "notes": "Full payment"
}
```

**Validation:**

| الحقل | القاعدة |
|---|---|
| `subscriptionId` | `@NotNull` + تابع للـ Org + مش `CANCELLED`/`EXPIRED` |
| `amountMinor` | `@Positive` + `<= outstanding` |
| `method` | `@NotNull` + من الـ Enum |
| `reference` | `@Size(max = 100)` |
| `notes` | `@Size(max = 500)` |

**Response `201 Created`:**
```json
{
  "success": true,
  "message": "Payment recorded",
  "data": {
    "id": "55555555-6666-7777-8888-999999999999",
    "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "memberName": "Ahmed Tarek",
    "amountMinor": 180000,
    "amountFormatted": "1,800.00 EGP",
    "paidTotalMinor": 180000,
    "outstandingMinor": 0,
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

**Replay `200 OK`:**
```json
{
  "success": true,
  "message": "Payment already recorded",
  "data": { "...": "نفس الدفعة الأصلية" },
  "timestamp": "2026-11-03T11:00:05Z"
}
```

**Headers في الـ Replay:**
```http
Idempotent-Replay: true
```

**Errors:**

| الحالة | Error | متى |
|---|---|---|
| `400` | `VALIDATION_FAILED` | مبلغ ≤ 0 / Method غلط |
| `404` | `SUBSCRIPTION_NOT_FOUND` | اشتراك مش موجود / Org تانية |
| `409` | `IDEMPOTENCY_CONFLICT` | نفس الـ Key بجسم مختلف / Race |
| `422` | `PAYMENT_EXCEEDS_BALANCE` | المبلغ > المتبقي |
| `422` | `SUBSCRIPTION_ALREADY_PAID` | المتبقي = 0 |
| `422` | `SUBSCRIPTION_NOT_PAYABLE` | `CANCELLED` / `EXPIRED` |

---

## 3. `GET /payments`

**Query:** `subscriptionId`, `memberId`, `method`, `status`, `from`, `to`, `page`, `size`, `sort`.

**Response `200 OK`:** `PageResponse<PaymentSummaryResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "id": "55555555-...",
        "subscriptionId": "77777777-...",
        "memberId": "a0b1c2d3-...",
        "memberName": "Ahmed Tarek",
        "amountMinor": 180000,
        "amountFormatted": "1,800.00 EGP",
        "method": "INSTAPAY",
        "receiptNumber": "KM-2026-000421",
        "status": "COMPLETED",
        "createdAt": "2026-11-03T11:00:00Z"
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
- Manager يشوف مدفوعات فرعه فقط.
- Receptionist → `403` على القائمة (بيستخدم `GET /{id}` لدفعة سجلها).

---

## 4. `GET /payments/{id}`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "55555555-...",
    "subscriptionId": "77777777-...",
    "memberId": "a0b1c2d3-...",
    "memberName": "Ahmed Tarek",
    "memberCode": "M-000123",
    "planName": "Gold 3-Month",
    "amountMinor": 90000,
    "amountFormatted": "900.00 EGP",
    "paidTotalMinor": 90000,
    "outstandingMinor": 90000,
    "method": "CASH",
    "reference": null,
    "receiptNumber": "KM-2026-000422",
    "status": "COMPLETED",
    "notes": "Deposit",
    "receivedBy": "b2c3d4e5-...",
    "receivedByName": "Mona Salah",
    "subscriptionStatus": "PENDING_PAYMENT",
    "createdAt": "2026-11-03T12:00:00Z",
    "reversedAt": null
  }
}
```

**Errors:** `404 PAYMENT_NOT_FOUND`.

---

## 5. `GET /payments/{id}/receipt`

**Response `200 OK`:**

```http
Content-Type: application/pdf
Content-Disposition: inline; filename="KM-2026-000421.pdf"
```

**Body:** PDF Binary.

**Rules:**
- Receptionist يقدر يطبع إيصال دفعة سجلها هو.
- Manager/Owner يقدر يطبع أي إيصال في نطاقه.
- الـ PDF بيتولد عند الطلب — مفيش Cache في MVP.

**Errors:** `404`, `500 RECEIPT_GENERATION_FAILED`.

---

## 6. `GET /payments/collections`

**Query:** `branchId` (Owner فقط), `agingBucket` (`0-7`, `8-14`, `15-30`, `31+`), `search`, `page`, `size`, `sort`.

**Response `200 OK`:** `PageResponse<CollectionItemResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "subscriptionId": "77777777-...",
        "memberId": "a0b1c2d3-...",
        "memberName": "Ahmed Tarek",
        "memberPhone": "01012345678",
        "planName": "Gold 3-Month",
        "priceMinorSnapshot": 180000,
        "priceFormatted": "1,800.00 EGP",
        "paidTotalMinor": 90000,
        "paidTotalFormatted": "900.00 EGP",
        "outstandingMinor": 90000,
        "outstandingFormatted": "900.00 EGP",
        "daysOutstanding": 12,
        "agingBucket": "8-14",
        "startDate": "2026-10-22"
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
- الاشتراكات `PENDING_PAYMENT` فقط.
- Manager → فرعه فقط.
- الترتيب الافتراضي: `daysOutstanding DESC`.

---

## 7. الـ DTOs

```java
public record RecordPaymentRequest(
    @NotNull UUID subscriptionId,
    @Positive long amountMinor,
    @NotNull PaymentMethod method,
    @Size(max = 100) String reference,
    @Size(max = 500) String notes
) {}

public record PaymentResponse(
    UUID id,
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberCode,
    String planName,
    long amountMinor,
    String amountFormatted,
    long paidTotalMinor,
    long outstandingMinor,
    PaymentMethod method,
    String reference,
    String receiptNumber,
    PaymentStatus status,
    String notes,
    UUID receivedBy,
    String receivedByName,
    String subscriptionStatus,
    Instant createdAt,
    Instant reversedAt
) {}

public record PaymentSummaryResponse(
    UUID id,
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    long amountMinor,
    String amountFormatted,
    PaymentMethod method,
    String receiptNumber,
    PaymentStatus status,
    Instant createdAt
) {}

public record CollectionItemResponse(
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberPhone,
    String planName,
    long priceMinorSnapshot,
    String priceFormatted,
    long paidTotalMinor,
    String paidTotalFormatted,
    long outstandingMinor,
    String outstandingFormatted,
    int daysOutstanding,
    String agingBucket,
    LocalDate startDate
) {}
```

---

## 8. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PaymentResponse record(RecordPaymentRequest request, @RequestHeader("Idempotency-Key") String key);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
PageResponse<PaymentSummaryResponse> list(PaymentFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PaymentResponse get(UUID id);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
ResponseEntity<byte[]> receipt(UUID id);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
PageResponse<CollectionItemResponse> collections(CollectionFilter filter, Pageable pageable);
```

- Branch Scope على `member.branch_id`.
- Trainer → `403` في كل الـ Endpoints.
- Receptionist → `403` على `GET /payments` و`/collections`.

---

## 9. أمثلة أخطاء

**`422 PAYMENT_EXCEEDS_BALANCE`:**
```json
{
  "timestamp": "2026-11-03T12:00:00Z",
  "status": 422,
  "error": "PAYMENT_EXCEEDS_BALANCE",
  "message": "Amount 2000.00 EGP exceeds outstanding balance 1800.00 EGP",
  "path": "/api/v1/payments",
  "traceId": "...",
  "details": []
}
```

**`409 IDEMPOTENCY_CONFLICT`:**
```json
{
  "timestamp": "2026-11-03T12:00:00Z",
  "status": 409,
  "error": "IDEMPOTENCY_CONFLICT",
  "message": "Idempotency key already used with a different request",
  "path": "/api/v1/payments",
  "traceId": "...",
  "details": []
}
```

---

## 10. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
