# Payment — خطة التنفيذ

> **الـ Phase:** 2 · **المدة:** 5–6 أيام · **الترتيب:** بعد `subscription`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-PAY-001 — Migrations + Entities

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-001 |
| **المخرجات** | `V10__create_payments.sql`, `V11__create_document_sequences.sql`, `Payment`, `DocumentSequence` |
| **معيار القبول** | RLS + Unique + CHECKs + Grants + Immutability Trigger |

**خطوات:**
1. `payments` بكل الأعمدة + `org_id`.
2. `CHECK (amount_minor > 0)`.
3. `CHECK (method IN ('CASH','INSTAPAY','BANK_TRANSFER','ONLINE'))`.
4. `CHECK (status IN ('COMPLETED','REVERSED'))`.
5. `UNIQUE (org_id, idempotency_key)` + `UNIQUE (org_id, receipt_number)`.
6. `document_sequences` + PK مركّب.
7. `app.next_document_number(...)`.
8. `SELECT app.enable_tenant_rls(...)` للاتنين.
9. `app.guard_payment_update()` Trigger.
10. `REVOKE DELETE` + `GRANT UPDATE (status, reversed_at, reversed_by)`.
11. Indexes `(subscription_id, status)` + `(org_id, created_at DESC)` + `(shift_id) WHERE shift_id IS NOT NULL`.

---

### TSK-PAY-002 — Idempotency

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-001 |
| **المخرجات** | `IdempotencyService` + `IdempotencyConflictException` |
| **معيار القبول** | 10 طلبات متزامنة بنفس الـ Key → دفعة واحدة |

**خطوات:**
1. Validate الـ Header (إلزامي، 8–100 حرف).
2. `pg_advisory_xact_lock(hashtextextended(orgId + ':' + key, 0))`.
3. `findByOrgIdAndIdempotencyKey`.
4. لو موجود → Replay Response + Header `Idempotent-Replay: true`.
5. Unique Constraint كـ Safety Net.
6. Test تحت 10 Threads.

---

### TSK-PAY-003 — Document Numbering

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-001 |
| **المخرجات** | `DocumentNumberService` |
| **معيار القبول** | 100 دفعة متزامنة → 100 رقم مختلف ومتسلسل |

**خطوات:**
1. `nextReceiptNumber(orgId, cairoDate)`.
2. Format `KM-%d-%06d`.
3. `Propagation.MANDATORY`.
4. Test على سنتين مختلفتين (Reset سنوي).
5. Test على Org‑ين (كل واحدة تبدأ من 1).

---

### TSK-PAY-004 — Payment Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-002, TSK-PAY-003 |
| **المخرجات** | `PaymentService`, `PaymentServiceImpl`, `PaymentQueryService` |
| **معيار القبول** | Full / Partial / Overpay / Replay / Invalid State |

**خطوات:**
1. `record` — تحقق من الاشتراك + العضو.
2. حساب `paidTotal` + `outstanding`.
3. رفض الدفع الزائد (PAY-02).
4. توليد `receiptNumber`.
5. حفظ `Payment`.
6. `eventPublisher.publishEvent(PaymentRecordedEvent)`.
7. `auditService.record(PAYMENT_RECORDED)`.
8. `listPayments` + `getPayment` + `getPaidTotal` + `getOutstanding`.
9. `@PreAuthorize` على كل Method.

---

### TSK-PAY-005 — Subscription Listener

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-SUB-004, TSK-PAY-004 |
| **المخرجات** | `PaymentRecordedListener` (في `subscription`) |
| **معيار القبول** | `totalPaidMinor >= priceSnapshot` → `ACTIVE` |

**خطوات:**
1. `@EventListener` (Synchronous).
2. `activateIfFullyPaid(subscriptionId, totalPaidMinor)`.
3. Idempotent — لو `ACTIVE` → تجاهل.
4. Test: Partial لا يفعّل، Full يفعّل، Replay لا يفعّل مرتين.

> **ملاحظة:** الـ Listener مملوك لـ `subscription`، مش `payment`. المهمة هنا للتكامل فقط.

---

### TSK-PAY-006 — Receipt PDF

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-004 |
| **المخرجات** | `ReceiptPdfService`, `ReceiptPdfGenerator` |
| **معيار القبول** | PDF يفتح + عربي سليم + كل الحقول |

**خطوات:**
1. Embed `Noto Naskh Arabic`.
2. `PdfWriter` + `Document`.
3. Header الجيم + الفرع.
4. بيانات العضو + الاشتراك.
5. المبلغ + المتبقي + طريقة الدفع.
6. Footer + رقم الإيصال.
7. `GET /payments/{id}/receipt` → `application/pdf`.
8. Visual Test يدوي + Snapshot Test للحجم.

---

### TSK-PAY-007 — Collections

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-004 |
| **المخرجات** | `listCollections` + `GET /payments/collections` |
| **معيار القبول** | متبقي صحيح + Aging Buckets + Branch Scope |

**خطوات:**
1. Query الاشتراكات `PENDING_PAYMENT` مع `paidTotal`.
2. احسب `outstanding` + `daysOutstanding`.
3. رتّب بالأقدم.
4. فلترة الفرع عبر `MemberQueryService` (Batch).
5. Enrich بـ `memberName` / `planName`.
6. Pagination.

---

### TSK-PAY-008 — API + Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-PAY-004..007 |
| **المخرجات** | `PaymentController` + 5 Endpoints + Tests |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

**خطوات:**
1. `POST /payments` + Header.
2. `GET /payments`.
3. `GET /payments/{id}`.
4. `GET /payments/{id}/receipt`.
5. `GET /payments/collections`.
6. كل الاختبارات.

---

## 2. الترتيب

```text
TSK-PAY-001 Migrations + Entities
      ↓
TSK-PAY-002 Idempotency
      ↓
TSK-PAY-003 Document Numbering
      ↓
TSK-PAY-004 Payment Service
      ↓
TSK-PAY-005 Subscription Listener (تكامل)
      ↓
TSK-PAY-006 Receipt PDF
      ↓
TSK-PAY-007 Collections
      ↓
TSK-PAY-008 API + Tests
```

---

## 3. معايير القبول النهائية

- [ ] `payments` + `document_sequences` + RLS.
- [ ] `Idempotency-Key` إلزامي وفعّال.
- [ ] 10 طلبات متزامنة → دفعة واحدة.
- [ ] `receiptNumber` متسلسل لكل Org/سنة.
- [ ] دفع كامل يفعّل الاشتراك في نفس الـ Transaction.
- [ ] دفع جزئي لا يفعّل.
- [ ] دفع زائد مرفوض.
- [ ] `PaymentRecordedEvent` فيه `totalPaidMinor`.
- [ ] PDF يفتح وبالعربي.
- [ ] Collections صحيحة.
- [ ] Audit + Immutability.
- [ ] كل الاختبارات خضراء.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| Double Submit | Idempotency + Advisory Lock |
| Receipt مكرر | `document_sequences` + Unique |
| دفعة بدون تفعيل | Event Synchronous + Transaction واحدة |
| تعديل مبلغ يدوي | Trigger + No Setter |
| PDF عربي مكسور | Visual Test + HTML Fallback |
| Collections بطيء | Index + Pagination |
| Overpay يسبب فلوس معلقة | رفض + Refund P1 |
| Manager يشوف مالية Org | Branch Scope + C-14 |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
