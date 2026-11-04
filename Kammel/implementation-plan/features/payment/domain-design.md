# Payment — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.payment.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. `Payment`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL` | RLS |
| `subscriptionId` | `UUID` | `NOT NULL` | FK — ممنوع `CASCADE` |
| `memberId` | `UUID` | `NOT NULL` | Denormalized من الاشتراك |
| `amountMinor` | `long` | `> 0` | بالقروش |
| `method` | `PaymentMethod` | `NOT NULL` | 4 قيم |
| `reference` | `String` | `NULL`, ≤ 100 | رقم عملية InstaPay/التحويل |
| `receivedBy` | `UUID` | `NOT NULL` | الموظف اللي سجل |
| `shiftId` | `UUID` | `NULL` | **بدون FK في MVP** (SC-20) |
| `status` | `PaymentStatus` | `NOT NULL` | `COMPLETED` / `REVERSED` |
| `idempotencyKey` | `String` | `NOT NULL`, ≤ 100 | Unique `(org_id, key)` |
| `receiptNumber` | `String` | `NOT NULL`, ≤ 30 | Unique `(org_id, receipt_number)` |
| `notes` | `String` | `NULL`, ≤ 500 | — |
| `reversedAt` | `Instant` | `NULL` | P1 |
| `reversedBy` | `UUID` | `NULL` | P1 |
| `createdAt` | `Instant` | `NOT NULL` | UTC |

```java
@Entity
@Table(name = "payments")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Payment extends BaseEntity implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(name = "subscription_id", nullable = false)
    private UUID subscriptionId;

    @Column(name = "member_id", nullable = false)
    private UUID memberId;

    @Column(name = "amount_minor", nullable = false)
    private long amountMinor;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private PaymentMethod method;

    @Column(length = 100)
    private String reference;

    @Column(name = "received_by", nullable = false)
    private UUID receivedBy;

    @Column(name = "shift_id")
    private UUID shiftId;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private PaymentStatus status;

    @Column(name = "idempotency_key", nullable = false, length = 100)
    private String idempotencyKey;

    @Column(name = "receipt_number", nullable = false, length = 30)
    private String receiptNumber;

    @Column(length = 500)
    private String notes;

    @Column(name = "reversed_at")
    private Instant reversedAt;

    @Column(name = "reversed_by")
    private UUID reversedBy;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt;

    public static Payment record(
        UUID orgId, UUID subscriptionId, UUID memberId, long amountMinor,
        PaymentMethod method, String reference, UUID receivedBy, UUID shiftId,
        String idempotencyKey, String receiptNumber, String notes, Instant now
    ) {
        // Validation + إنشاء الكيان
    }

    public void reverse(UUID actorId, Instant now) {
        if (status != PaymentStatus.COMPLETED) {
            throw new InvalidPaymentStateException(status);
        }
        this.status = PaymentStatus.REVERSED;
        this.reversedBy = actorId;
        this.reversedAt = now;
    }
}
```

> **ممنوع Setters** على `amountMinor` / `subscriptionId` / `receiptNumber` / `idempotencyKey`.
> التغيير الوحيد المسموح: `COMPLETED → REVERSED` (P1) — وده بيتحقق في الـ DB Trigger كمان.

---

## 2. الـ Enums

```java
public enum PaymentMethod {
    CASH,
    INSTAPAY,
    BANK_TRANSFER,
    ONLINE
}

public enum PaymentStatus {
    COMPLETED,
    REVERSED
}
```

> **ملاحظة:** مفيش `PENDING` في الـ MVP — الدفع بيتسجل بعد ما يتم فعلياً. الـ Gateway Webhooks مؤجلة P2.

---

## 3. `DocumentSequence`

| الحقل | النوع | القيود |
|---|---|---|
| `orgId` | `UUID` | PK جزء |
| `documentType` | `String` | PK جزء — `RECEIPT` |
| `year` | `int` | PK جزء |
| `nextValue` | `long` | `> 0` |

```java
@Entity
@Table(name = "document_sequences")
@IdClass(DocumentSequenceId.class)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class DocumentSequence {

    @Id
    @Column(name = "org_id")
    private UUID orgId;

    @Id
    @Column(name = "document_type", length = 30)
    private String documentType;

    @Id
    @Column(name = "year")
    private int year;

    @Column(name = "next_value", nullable = false)
    private long nextValue;
}
```

- الـ Counter بيتزود **ذرّياً** في الـ DB عبر `app.next_document_number(...)`.
- ممنوع `SELECT ... FOR UPDATE` من الـ Java ثم `UPDATE` — الدالة بتعمل `INSERT ... ON CONFLICT DO UPDATE` في خطوة واحدة.
- `document_sequences` مملوكة لـ `payment` لأنها أول مستهلك، وأي مستند جديد في المستقبل يستخدم نفس الـ Function.

---

## 4. ترقيم الإيصالات

### 4.1 التنسيق

```text
KM-{YYYY}-{000001}
```

- `KM` — Kammel.
- `YYYY` — سنة الدفع بتوقيت `Africa/Cairo` (مش UTC).
- `000001` — 6 أرقام على الأقل، وبتزيد طبيعي لو تعدّت `999999`.

**مثال:** دفعة يوم `2026-11-03` → `KM-2026-000421`.

### 4.2 التسلسل

```java
@Component
public class DocumentNumberService {

    private final EntityManager em;

    @Transactional(propagation = Propagation.MANDATORY)
    public String nextReceiptNumber(UUID orgId, LocalDate cairoDate) {
        long value = (Long) em.createNativeQuery(
                "SELECT app.next_document_number(:orgId, 'RECEIPT', :year)")
            .setParameter("orgId", orgId)
            .setParameter("year", cairoDate.getYear())
            .getSingleResult();

        return "KM-%d-%06d".formatted(cairoDate.getYear(), value);
    }
}
```

- `MANDATORY` — لازم تكون جوه Transaction موجودة.
- الـ Row Lock بيتم داخل الـ Function، فمفيش تكرار تحت التزامن.
- لو الـ Transaction اتعملت Rollback، الزيادة بترجع معاها — يعنى ممكن يحصل **Gap** لو الـ Sequence اتستهلكت في Transaction فشلت؟ لا: لأن الزيادة جزء من نفس الـ Transaction، فالـ Rollback بيرجّعها. الـ Gap ممكن يحصل فقط لو فيه Transaction منفصلة اتعملت Commit — وده مش هيحصل في التصميم ده.
- **ممنوع** استخدام `SEQUENCE` عادية — لأنها Global ومش Per-Org.

---

## 5. حساب المتبقي

```text
paidTotalMinor     = Σ payments.amount_minor
                     WHERE subscription_id = :id
                       AND status = 'COMPLETED'

outstandingMinor   = max(0, price_minor_snapshot - paidTotalMinor)
```

### 5.1 قواعد الدفع

| الحالة | السلوك |
|---|---|
| `amountMinor < outstanding` | دفع جزئي — الاشتراك يفضل `PENDING_PAYMENT`. |
| `amountMinor == outstanding` | دفع كامل — `PaymentRecordedEvent` بيفعّل الاشتراك. |
| `amountMinor > outstanding` | ❌ `422 PAYMENT_EXCEEDS_BALANCE` (PAY-02). |
| `outstanding == 0` | ❌ `422 SUBSCRIPTION_ALREADY_PAID`. |
| `subscription.status = CANCELLED` | ❌ `422 SUBSCRIPTION_CANCELLED`. |
| `subscription.status = EXPIRED` | ❌ `422 SUBSCRIPTION_EXPIRED` — لازم تجديد الأول. |

> **ليه ممنوع الدفع الزائد؟** لأن مفيش `Credit Balance` أو `Discount` Entity في الـ MVP. أي زيادة هتكون فلوس بدون مرجع محاسبي. الـ Refund/Adjustment في P1.

### 5.2 الـ Overpayment والـ Add-ons

لو الجيم بيبيع `PT Sessions` أو `Towel Card` منفصلين، دول **مش** جزء من سعر الاشتراك. لازم يتعملوا كـ `Plan` منفصل أو `Payment` مرتبط باشتراك منفصل — مش دفع زائد على نفس الاشتراك.

---

## 6. الـ Idempotency

### 6.1 المشكلة

الاستقبال بيدوس "حفظ" مرتين، أو الشبكة قطعت والعميل أعاد المحاولة. من غير حماية → دفعتين بنفس الفلوس.

### 6.2 الحل — Advisory Lock + Unique Constraint

```java
@Transactional
public PaymentResponse record(RecordPaymentRequest request, String idempotencyKey) {
    UUID orgId = TenantContext.requireOrgId();
    UUID actorId = SecurityUtils.getCurrentUserId();

    // 1) قفل منطقي على (org, key) — بيمنع Racing Requests
    em.createNativeQuery(
            "SELECT pg_advisory_xact_lock(hashtextextended(:key, 0))")
        .setParameter("key", orgId + ":" + idempotencyKey)
        .getSingleResult();

    // 2) لو المفتاح موجود → Replay
    Optional<Payment> existing = repository
        .findByOrgIdAndIdempotencyKey(orgId, idempotencyKey);
    if (existing.isPresent()) {
        return mapper.toResponse(existing.get(), true);
    }

    // 3) تحقق + قفل الاشتراك
    SubscriptionSummary sub = subscriptionQueryService
        .getSubscriptionForUpdate(request.subscriptionId());

    // 4) حساب المتبقي
    long paid = paymentQueryService.getPaidTotal(sub.id());
    long outstanding = sub.priceMinorSnapshot() - paid;

    // 5) رقم الإيصال + الحفظ
    ...
}
```

- `pg_advisory_xact_lock` بتتحرر تلقائياً عند Commit/Rollback.
- الـ Unique Index `(org_id, idempotency_key)` هو **آخر خط دفاع**.
- لو الـ Unique Constraint اترمت رغم القفل (مثلاً Bug) → `IdempotencyConflictException` → `409`.
- الـ Replay بيرجع **نفس** `PaymentResponse` مع Header `Idempotent-Replay: true` و HTTP `200 OK`.

### 6.3 قواعد الـ Key

| القاعدة | التفصيل |
|---|---|
| إلزامي | Header `Idempotency-Key` — مفيش Default. |
| الطول | 8–100 حرف. |
| الشكل | UUID مستحسن، لكن أي String مقبول. |
| النطاق | لكل Org — نفس الـ Key في Org تانية مسموح. |
| الصلاحية | للأبد — مفيش Expiry في MVP. |
| الـ Replay | نفس الـ Body مطلوب؛ لو الـ Body مختلف → `409 IDEMPOTENCY_CONFLICT`. |

> **الـ Body Hash:** بنخزن `SHA-256` للـ Request في `notes`؟ لا — ممنوع تلويث الحقول. الـ MVP بيعتبر نفس الـ Key = نفس العملية، ومش بيقارن الـ Body. الـ `409` بيتحقق منه في P1 لو أضفنا `request_hash`.

---

## 7. الـ Event

```java
public record PaymentRecordedEvent(
    UUID paymentId,
    UUID orgId,
    UUID subscriptionId,
    UUID memberId,
    long amountMinor,
    long totalPaidMinor,
    PaymentMethod method,
    Instant occurredAt
) {}
```

### 7.1 قواعد النشر

1. **Synchronous** — `@EventListener` مش `@TransactionalEventListener`.
2. بيتنشر **بعد** حفظ الـ Payment وقبل الـ Commit.
3. `totalPaidMinor` = مجموع الدفعات `COMPLETED` **بعد** إضافة الدفعة الحالية.
4. لو الـ Listener رفض (مثلاً Subscription مش موجود) → الـ Transaction كلها Rollback.
5. الـ Listener لازم يكون Idempotent — لو الاشتراك `ACTIVE` بالفعل، يتجاهل.

```java
eventPublisher.publishEvent(new PaymentRecordedEvent(
    payment.getId(), orgId, payment.getSubscriptionId(), payment.getMemberId(),
    payment.getAmountMinor(), paidTotalAfter, payment.getMethod(), now
));
```

---

## 8. الـ Query Service

```java
public interface PaymentQueryService {

    PaymentResponse getPayment(UUID paymentId);

    PageResponse<PaymentSummary> listPayments(PaymentFilter filter, Pageable pageable);

    long getPaidTotal(UUID subscriptionId);

    long getOutstanding(UUID subscriptionId, long priceMinorSnapshot);

    PageResponse<CollectionItem> listCollections(CollectionFilter filter, Pageable pageable);
}

public record PaymentSummary(
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

public record CollectionItem(
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberPhone,
    String planName,
    long priceMinorSnapshot,
    long paidTotalMinor,
    long outstandingMinor,
    int daysOutstanding,
    LocalDate startDate
) {}
```

- `getPaidTotal` بتستخدم `SUM` — مش بتحمّل كل الدفعات.
- `listCollections` بتجيب الاشتراكات اللي `outstanding > 0` من `subscription` + `payment`، وبتفلتر الفرع عبر `MemberQueryService` (Batch).
- ممنوع ترجع `Payment` Entity.

---

## 9. الـ Command Service

```java
public interface PaymentService {

    PaymentResponse record(RecordPaymentRequest request, String idempotencyKey);

    // P1
    PaymentResponse reverse(UUID paymentId, ReversePaymentRequest request);
}
```

---

## 10. التحصيل (Collections)

### 10.1 التعريف

```text
Collection = Subscription
             WHERE status = 'PENDING_PAYMENT'
               AND outstandingMinor > 0
```

- الاشتراك `ACTIVE` مستبعد لأن تفعيله بيحتاج `paidTotal >= priceSnapshot`.
- `GRACE` مستبعد لأن المتبقي عليه صفر (لو فيه متبقي كان يفضل `PENDING_PAYMENT`).
- `daysOutstanding` = `today(Cairo) - subscription.created_at(Cairo)`.

### 10.2 الـ Aging Buckets

| Bucket | المعنى |
|---|---|
| `0–7` | جديد — تذكير ودّي. |
| `8–14` | متأخر — مكالمة. |
| `15–30` | متأخر جداً — Manager. |
| `31+` | خطر — Owner. |

> **ملاحظة:** الـ Collections بتقرأ من `subscriptions` + `payments` — Read-only. ممنوع تعدل حالة الاشتراك من هنا.

---

## 11. الإيصال PDF

### 11.1 المحتوى

- اسم الجيم + الفرع + رقم الهاتف.
- رقم الإيصال + التاريخ.
- اسم العضو + كود العضو.
- الباقة + الفترة.
- المبلغ المدفوع + المتبقي (لو جزئي).
- طريقة الدفع + المرجع.
- الموظف اللي سجل.
- ملاحظة: "هذا الإيصال لا يُعد بديلاً عن الفاتورة الضريبية".

### 11.2 التنفيذ

```java
public interface ReceiptPdfService {
    byte[] generate(UUID paymentId);
}
```

- **OpenPDF** (`com.github.librepdf:openpdf`).
- خط عربي مضمّن: `Noto Naskh Arabic` من `src/main/resources/fonts`.
- `BaseFont.IDENTITY_H` + `PdfPCell.setRunDirection(PdfWriter.RUN_DIRECTION_RTL)`.
- لو النص العربي فيه مشكلة Shaping → نستخدم `ArabicShaping` + `Bidi` من `icu4j` قبل الرسم.
- الـ Endpoint بيرجع `application/pdf` + `Content-Disposition: inline; filename="KM-2026-000421.pdf"`.
- الملف **مش متخزن** — بيتولد عند الطلب (Stateless).

> **مخاطرة:** دعم العربي في PDF محتاج اختبار بصري. لو ظهرت مشاكل، البديل المؤقت هو HTML Print View في الـ Frontend، والـ PDF يتأجل P1.

---

## 12. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-PAY-01 | `amountMinor > 0` | DB CHECK + DTO |
| INV-PAY-02 | `memberId` = `subscription.memberId` | Service |
| INV-PAY-03 | `receiptNumber` فريد لكل Org | DB Unique |
| INV-PAY-04 | `idempotencyKey` فريد لكل Org | DB Unique |
| INV-PAY-05 | الحقول المالية Immutable | Trigger + No Setter |
| INV-PAY-06 | ممنوع `DELETE` | Grants + Trigger |
| INV-PAY-07 | `status` من `COMPLETED` لـ `REVERSED` فقط | Trigger + Service |
| INV-PAY-08 | `totalPaidMinor` يشمل الدفعة الحالية | Service |
| INV-PAY-09 | الدفع لا يعدل `subscription.status` | Arch Test |
| INV-PAY-10 | `orgId` من الـ Context | Service |
| INV-PAY-11 | `receivedBy` من الـ Context | Service |
| INV-PAY-12 | كل دفع له Audit | Service |
| INV-PAY-13 | ممنوع دفع على اشتراك مش تابع للـ Org | RLS + Query Service |

---

## 13. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `SubscriptionNotFoundException` | 404 | اشتراك مش موجود / Org تانية |
| `PaymentNotFoundException` | 404 | دفعة مش موجودة |
| `SubscriptionNotPayableException` | 422 | `CANCELLED` / `EXPIRED` |
| `PaymentExceedsBalanceException` | 422 | المبلغ > المتبقي |
| `SubscriptionAlreadyPaidException` | 422 | المتبقي = 0 |
| `IdempotencyConflictException` | 409 | نفس الـ Key بجسم مختلف / Unique Race |
| `InvalidPaymentStateException` | 422 | Reversal على `REVERSED` |
| `ReceiptGenerationException` | 500 | فشل PDF |

---

## 14. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [database-strategy](../../architecture/database-strategy.md#6-ترقيم-المستندات-document-sequences)
