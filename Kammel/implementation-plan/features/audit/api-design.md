# Audit — تصميم الـ API

> **Base Path:** `/api/v1/audit-logs`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/audit-logs` | `GYM_OWNER` | قائمة مع فلاتر |
| GET | `/audit-logs/{id}` | `GYM_OWNER` | تفاصيل سجل |
| GET | `/audit-logs/entities/{entityType}/{entityId}` | `GYM_OWNER` | تاريخ كيان |

> **مفيش** `POST` / `PATCH` / `DELETE` — الـ Audit بيتكتب من الـ Services فقط.

---

## 2. `GET /audit-logs`

**Query Params:**

| Param | النوع | ملاحظات |
|---|---|---|
| `actorId` | `UUID` | اختياري |
| `action` | `String` | اختياري — من الكتالوج |
| `entityType` | `String` | اختياري |
| `entityId` | `UUID` | اختياري |
| `from` | `Instant` | اختياري — ISO-8601 |
| `to` | `Instant` | اختياري |
| `page` / `size` / `sort` | — | default `createdAt,desc` |

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "id": "11111111-2222-3333-4444-555555555555",
        "actorType": "USER",
        "actorId": "b2c3d4e5-...",
        "actorName": "Mostafa Ibrahim",
        "action": "SUBSCRIPTION_FROZEN",
        "entityType": "SUBSCRIPTION",
        "entityId": "77777777-...",
        "reason": "Traveling for work",
        "ipAddress": "197.xx.xx.xx",
        "createdAt": "2026-11-03T12:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1,
    "first": true,
    "last": true
  },
  "timestamp": "2026-11-03T12:05:00Z"
}
```

**Rules:**
- الـ `orgId` من الـ Context — ممنوع Query Param.
- الـ `actorName` بيتجاب من `UserQueryService` (Batch) — مش Join مباشر.
- لو الـ User اتحذف/اتعطل → الاسم لسه بيظهر (Snapshot مش متخزن، لكن User موجود).

---

## 3. `GET /audit-logs/{id}`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "id": "11111111-...",
    "actorType": "USER",
    "actorId": "b2c3d4e5-...",
    "actorName": "Mostafa Ibrahim",
    "action": "SUBSCRIPTION_FROZEN",
    "entityType": "SUBSCRIPTION",
    "entityId": "77777777-...",
    "beforeState": {
      "status": "ACTIVE",
      "endDate": "2026-12-10"
    },
    "afterState": {
      "status": "FROZEN",
      "endDate": "2026-12-17"
    },
    "reason": "Traveling for work",
    "ipAddress": "197.xx.xx.xx",
    "userAgent": "Mozilla/5.0 ...",
    "traceId": "9f8e7d6c5b4a3210",
    "createdAt": "2026-11-03T12:00:00Z"
  }
}
```

**Errors:** `404 RESOURCE_NOT_FOUND` — لو السجل تابع لـ Org تانية.

---

## 4. `GET /audit-logs/entities/{entityType}/{entityId}`

**Response `200 OK`:** `List<AuditLogSummaryResponse>` مرتبة `createdAt DESC`.

**الاستخدام:** تاريخ كيان معين — مثلاً كل عمليات اشتراك.

**Errors:**
- `400 INVALID_ARGUMENT` — `entityType` غير معروف.
- `404 RESOURCE_NOT_FOUND`.

---

## 5. الـ DTOs

```java
public record AuditLogSummaryResponse(
    UUID id,
    ActorType actorType,
    UUID actorId,
    String actorName,
    String action,
    String entityType,
    UUID entityId,
    String reason,
    String ipAddress,
    Instant createdAt
) {}

public record AuditLogDetailResponse(
    UUID id,
    ActorType actorType,
    UUID actorId,
    String actorName,
    String action,
    String entityType,
    UUID entityId,
    JsonNode beforeState,
    JsonNode afterState,
    String reason,
    String ipAddress,
    String userAgent,
    String traceId,
    Instant createdAt
) {}

public record AuditLogFilter(
    UUID actorId,
    String action,
    String entityType,
    UUID entityId,
    Instant from,
    Instant to
) {}
```

---

## 6. الصلاحيات

```java
@PreAuthorize("hasRole('GYM_OWNER')")
PageResponse<AuditLogSummaryResponse> list(AuditLogFilter filter, Pageable pageable);

@PreAuthorize("hasRole('GYM_OWNER')")
AuditLogDetailResponse get(UUID id);
```

- ممنوع Manager / Receptionist / Trainer.
- الـ Owner بس — حتى لو عنده أكتر من فرع.

---

## 7. الأداء

| Query | Index |
|---|---|
| حسب التاريخ | `(org_id, created_at DESC)` |
| حسب الكيان | `(org_id, entity_type, entity_id, created_at DESC)` |
| حسب المستخدم | `(org_id, actor_id, created_at DESC)` |

- `size` أقصى 100.
- الفترة الافتراضية لو مش محددة: آخر 30 يوم.
- ممنوع `SELECT *` بدون Pagination.

---

## 8. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
