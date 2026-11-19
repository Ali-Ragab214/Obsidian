---
title: API Documentation
cssclasses: [rtl]
tags: [api, rest, openapi, springdoc]
section: 10
---

# 10. API Documentation
⬅️ [[09 - Stock Movement Lifecycle]] | ➡️ [[11 - AI Capabilities]]

## 10.1 Conventions

- **Base path:** `/api/v1`
- **Content type:** `application/json`
- **Authentication:** `Authorization: Bearer <JWT>` لكل الـ endpoints ما عدا الـ public list في `SecurityConfig`.
- **OpenAPI UI:** `/swagger-ui.html` (springdoc default) و `/v3/api-docs`.

> [!note] حاليًا مفيش custom OpenAPI metadata ولا security scheme ولا examples.

## 10.2 ✅ الـ Endpoints المنفَّذة

الـ **category endpoints** بس هي المكشوفة:

| Method | Path | Request | Response |
|---|---|---|---|
| `POST` | `/api/v1/categories/create` | `CategoryRequest` | `201 Created` |
| `PUT` | `/api/v1/categories/update/{id}` | `CategoryRequest` | `204 No Content` |
| `GET` | `/api/v1/categories/{id}` | — | `200` + `CategoryResponse` |
| `GET` | `/api/v1/categories?page=0&size=10` | Query params | `200` + `PageResponse<CategoryResponse>` |
| `DELETE` | `/api/v1/categories/delete/{id}` | — | `204 No Content` |

**`CategoryRequest`**
```json
{
  "name": "Beverages",
  "description": "Soft drinks, water, and juices"
}
```

**`CategoryResponse`**
```json
{
  "id": "6f1c2a9e-7b1d-4f0e-9a2c-3d4e5f6a7b8c",
  "name": "Beverages",
  "description": "Soft drinks, water, and juices"
}
```

**مثال request** (التوكن الصالح مش ممكن تاخده من الـ API حاليًا):
```bash
curl -X POST http://localhost:8080/api/v1/categories/create \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Beverages","description":"Soft drinks, water, and juices"}'
```

## 10.3 الـ Pagination Envelope

`PageResponse<T>` بيحتوي:

| Field | Type | ملاحظات |
|---|---|---|
| `content` | array | عناصر الصفحة |
| `page` | int | Zero-based page index |
| `size` | int | حجم الصفحة المطلوب |
| `totalElements` | long | إجمالي الصفوف المطابقة |
| `totalPages` | int | إجمالي الصفحات |
| `hasNext` | boolean | |
| `hasPrevious` | boolean | |
| `isFirst` | boolean | JavaBean naming ممكن يتسيريلايز كـ `first`؛ **اتحقق من الـ JSON contract الفعلي** |
| `isLast` | boolean | ممكن يتسيريلايز كـ `last`؛ اتحقق |

> [!warning] مفيش **حد أقصى لحجم الصفحة** ولا **sort parameter** ولا filtering. `page` أو `size` سالب بيطلّع framework exception بيتحوّل حاليًا لـ **HTTP 500**.

## 10.4 Error Format

`GlobalExceptionHandler` بيرجّع:
```json
{
  "timestamp": "2026-11-18T20:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Category with id ... not found",
  "path": "/api/v1/categories/..."
}
```

### القيود الحالية
- مفيش **machine-readable error code**.
- مفيش **field-level error list**.
- مفيش **correlation/trace id**.
- `BusinessException` (بما فيها `InvalidRequestException` و `UnauthorizedException`) **مش متعامل معاها صراحةً** وبتقع في **HTTP 500**.
- `EntityNotFoundException` و `DuplicateKeyException` و `IllegalStateException` و malformed JSON و type mismatches كلها برضو بتقع في 500.
- رسائل الـ validation بتتجمّع في string واحد.
- مفيش تمييز بين 401 و 403 على مستوى الـ security layer.

> [!tip] التوصية
> اعتمد **RFC 9457 `ProblemDetail`** بحقول: `type`, `title`, `status`, `detail`, `instance`, `code`, `errors[]`, `traceId`.

## 10.5 فجوات تصميم الـ API

- **Non-resource paths:** `/create` و `/update/{id}` و `/delete/{id}` عبارة عن **RPC-style**. الأفضل: `POST /categories` و `PUT /categories/{id}` و `DELETE /categories/{id}`.
- مفيش **`Location` header** عند الإنشاء.
- مفيش **response body** عند الإنشاء، فالـ clients مش بتعرف الـ id المولَّد.
- **Hard delete** للـ categories/products رغم وجود عمود `deleted`.
- مفيش filtering ولا search ولا sorting على الـ list endpoints.
- مفيش حد أقصى لحجم الصفحة.
- مفيش **idempotency** لأي write.
- مفيش **ETag / If-Match** للـ updates.
- مفيش **API versioning policy** صريحة.
- مفيش **security scheme** في الـ OpenAPI، فزر "Authorize" في Swagger مش متظبط.
- مفيش example values أو schema descriptions.
- مفيش `problem+json` content type.

## 10.6 💡 الـ API Surface المقترحة

> [!warning] تتضاف **بعد** ما الـ tenant isolation والـ stock ledger يبقوا موجودين.

| المجال | الـ Endpoints المقترحة |
|---|---|
| **Authentication** | `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `GET /auth/me` |
| **Tenants** | `POST /tenants`, `GET /tenants`, `GET /tenants/{id}`, `POST /tenants/{id}/approve`, `POST /tenants/{id}/suspend`, `POST /tenants/{id}/activate` |
| **Users** | `GET /users`, `POST /users`, `GET /users/{id}`, `PUT /users/{id}`, `DELETE /users/{id}`, `POST /users/{id}/enable`, `POST /users/{id}/disable`, `POST /invitations` |
| **Categories** | `GET /categories`, `POST /categories`, `GET /categories/{id}`, `PUT /categories/{id}`, `DELETE /categories/{id}` |
| **Products** | `GET /products`, `POST /products`, `GET /products/{id}`, `PUT /products/{id}`, `DELETE /products/{id}` |
| **Warehouses** | `GET /warehouses`, `POST /warehouses`, `PUT /warehouses/{id}`, `DELETE /warehouses/{id}` |
| **Stock** | شوف [[09 - Stock Movement Lifecycle#9.12 Stock APIs المقترحة]] |
| **Reservations** | `POST /reservations`, `GET /reservations`, `DELETE /reservations/{id}` |
| **Replenishment** | `GET /reorder-policies`, `PUT /reorder-policies/{warehouseId}/{productId}`, `GET /alerts/low-stock` |
| **Reporting** | `GET /reports/stock-summary`, `GET /reports/movements`, `GET /reports/valuation`, `GET /reports/turnover`, `POST /reports/export` |
| **AI** | شوف [[11 - AI Capabilities]] (§11.12) |

## 10.7 توصيات الـ OpenAPI

- ضيف **OpenAPI bean** فيه title و version و description و contact و license (الـ license حاليًا **TBD**).
- ضيف **SecurityScheme** لـ bearer JWT وطبّقه globally ما عدا login/swagger endpoints.
- جمّع الـ operations بـ **tags** (`Auth`, `Tenants`, `Products`, `Stock`, `Reports`, `AI`).
- وثّق الـ error responses بـ **ProblemDetail schema** مشترك.
- ضيف request/response **examples**.
- خلّي الـ schema المولَّدة متزامنة مع integration tests أو **contract snapshot**.

---
⬅️ [[09 - Stock Movement Lifecycle]] | ➡️ [[11 - AI Capabilities]]
