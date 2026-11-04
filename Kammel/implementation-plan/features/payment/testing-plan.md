# Payment — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Idempotency | صفر دفعات مكررة تحت التزامن |
| الدقة المالية | كل الحسابات `Long` بالقروش |
| الترقيم | تسلسل فريد لكل Org/سنة |
| التفعيل | دفع كامل يفعّل الاشتراك |
| الجزئي | المتبقي صحيح دائماً |
| Immutability | ممنوع تعديل/حذف |
| العزل | صفر تسريب بين الـ Orgs |

---

## 2. Idempotency Tests

| # | السيناريو | المتوقع |
|---|---|---|
| ID-PAY-01 | نفس الـ Key مرتين متتاليتين | دفعة واحدة + `200` Replay |
| ID-PAY-02 | 10 Threads بنفس الـ Key | دفعة واحدة + 9 Replay |
| ID-PAY-03 | نفس الـ Key في Org مختلفة | دفعتين — مسموح |
| ID-PAY-04 | Key ناقص | `400 VALIDATION_FAILED` |
| ID-PAY-05 | Key > 100 حرف | `400` |
| ID-PAY-06 | Key مختلف + نفس الدفعة | دفعتين — مسموح |
| ID-PAY-07 | Replay بعد يوم | نفس الدفعة |
| ID-PAY-08 | Replay بجسم مختلف | `409` (P1) / تجاهل (MVP) |

```java
@Test
void concurrentSameIdempotencyKey_createsOnePayment() throws Exception {
    int threads = 10;
    ExecutorService pool = Executors.newFixedThreadPool(threads);
    CountDownLatch start = new CountDownLatch(1);

    List<Future<PaymentResponse>> futures = new ArrayList<>();
    for (int i = 0; i < threads; i++) {
        futures.add(pool.submit(() -> {
            start.await();
            return paymentService.record(request, "key-123");
        }));
    }

    start.countDown();
    Set<UUID> ids = new HashSet<>();
    for (Future<PaymentResponse> f : futures) ids.add(f.get().id());

    assertThat(ids).hasSize(1);
    assertThat(paymentRepository.count()).isEqualTo(1);
}
```

---

## 3. Financial Tests

| # | السيناريو | المتوقع |
|---|---|---|
| FI-PAY-01 | سعر 180000 + دفع 180000 | `outstanding = 0` + `ACTIVE` |
| FI-PAY-02 | سعر 180000 + دفع 90000 | `outstanding = 90000` + `PENDING_PAYMENT` |
| FI-PAY-03 | 90000 + 90000 | `outstanding = 0` + `ACTIVE` |
| FI-PAY-04 | دفع 200000 على 180000 | `422 PAYMENT_EXCEEDS_BALANCE` |
| FI-PAY-05 | دفع على اشتراك مدفوع | `422 SUBSCRIPTION_ALREADY_PAID` |
| FI-PAY-06 | مبلغ 0 | `400` |
| FI-PAY-07 | مبلغ سالب | `400` |
| FI-PAY-08 | 3 دفعات جزئية | مجموع صحيح + `ACTIVE` عند الاكتمال |
| FI-PAY-09 | `MoneyUtil` | `70000 → 700.00` — مفيش `double` |
| FI-PAY-10 | مبلغ كبير `Long.MAX_VALUE` | مرفوض بـ Validation |

---

## 4. Receipt Number Tests

| # | السيناريو | المتوقع |
|---|---|---|
| RC-PAY-01 | أول دفعة في Org | `KM-2026-000001` |
| RC-PAY-02 | تاني دفعة | `KM-2026-000002` |
| RC-PAY-03 | دفعة في سنة جديدة | `KM-2027-000001` |
| RC-PAY-04 | Org تانية | تبدأ من `000001` |
| RC-PAY-05 | 100 دفعة متزامنة | 100 رقم مختلف |
| RC-PAY-06 | Transaction تفشل | الرقم لا يُستهلك |
| RC-PAY-07 | رقم > 999999 | 7 أرقام بدون كسر |
| RC-PAY-08 | `receipt_number` Unique | DB يرفض التكرار |

---

## 5. Subscription Activation Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AC-PAY-01 | دفع كامل | `PaymentRecordedEvent` + `ACTIVE` |
| AC-PAY-02 | دفع جزئي | `PENDING_PAYMENT` + مفيش `ACTIVATED` |
| AC-PAY-03 | دفعة تانية تكمل | `ACTIVE` مرة واحدة |
| AC-PAY-04 | Replay | مفيش تفعيل جديد |
| AC-PAY-05 | Listener Exception | الـ Transaction Rollback — مفيش Payment |
| AC-PAY-06 | `totalPaidMinor` في الـ Event | يساوي مجموع الدفعات |
| AC-PAY-07 | اشتراك `CANCELLED` | `422` |
| AC-PAY-08 | اشتراك `EXPIRED` | `422` |
| AC-PAY-09 | اشتراك `GRACE` | دفع مسموح |
| AC-PAY-10 | اشتراك `FROZEN` | دفع مسموح |

---

## 6. Immutability Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IM-PAY-01 | `UPDATE amount_minor` | DB Exception |
| IM-PAY-02 | `UPDATE subscription_id` | DB Exception |
| IM-PAY-03 | `UPDATE receipt_number` | DB Exception |
| IM-PAY-04 | `UPDATE idempotency_key` | DB Exception |
| IM-PAY-05 | `DELETE payment` | DB Exception |
| IM-PAY-06 | `UPDATE status COMPLETED → REVERSED` | مسموح (P1) |
| IM-PAY-07 | `UPDATE status REVERSED → COMPLETED` | مرفوض |
| IM-PAY-08 | Service يحاول تعديل المبلغ | مفيش Setter — Compile Error |

---

## 7. Collections Tests

| # | السيناريو | المتوقع |
|---|---|---|
| CL-PAY-01 | اشتراك مدفوع بالكامل | مش ظاهر |
| CL-PAY-02 | اشتراك بمتبقي | ظاهر |
| CL-PAY-03 | Aging `0-7` | Bucket صح |
| CL-PAY-04 | Aging `31+` | Bucket صح |
| CL-PAY-05 | Manager فرع A | اشتراكات A فقط |
| CL-PAY-06 | Owner | كل الفروع |
| CL-PAY-07 | ترتيب | الأقدم أولاً |
| CL-PAY-08 | بحث بالاسم/الهاتف | فلترة صحيحة |

---

## 8. Receipt PDF Tests

| # | الاختبار | المتوقع |
|---|---|---|
| PD-PAY-01 | `GET /receipt` | `200` + `application/pdf` |
| PD-PAY-02 | محتوى PDF | رقم الإيصال + الاسم + المبلغ |
| PD-PAY-03 | عربي | الحروف متصلة ومش معكوسة |
| PD-PAY-04 | دفعة جزئية | المتبقي ظاهر |
| PD-PAY-05 | دفعة مش موجودة | `404` |
| PD-PAY-06 | Receptionist | يقدر يطبع |
| PD-PAY-07 | Trainer | `403` |
| PD-PAY-08 | PDF > 1 صفحة | Pagination سليمة |

---

## 9. Isolation Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-PAY-01 | `POST /payments` على اشتراك Org B | `404` |
| IT-PAY-02 | `GET /payments/{B}` | `404` |
| IT-PAY-03 | `GET /collections` كـ Org A | A فقط |
| IT-PAY-04 | نفس `Idempotency-Key` في Org‑ين | مسموح |
| IT-PAY-05 | Raw JDBC بدون Context | صفر صفوف |
| IT-PAY-06 | Manager فرع B | `404` |

---

## 10. Audit Tests

| # | العملية | Audit |
|---|---|---|
| AU-PAY-01 | تسجيل دفع | `PAYMENT_RECORDED` |
| AU-PAY-02 | Replay | مفيش Audit جديد |
| AU-PAY-03 | Reversal (P1) | `PAYMENT_REVERSED` |
| AU-PAY-04 | فشل الدفع | مفيش Audit |
| AU-PAY-05 | `beforeState` / `afterState` | JSON صحيح |

---

## 11. Performance

| المسار | الهدف |
|---|---|
| `POST /payments` | P95 < 300ms |
| `GET /payments` | P95 < 200ms |
| `GET /payments/{id}` | P95 < 150ms |
| `GET /collections` (5000 اشتراك) | P95 < 400ms |
| Receipt PDF | < 1s |
| 100 دفعة متزامنة | < 5s إجمالي |

---

## 12. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-019 | FI-PAY-09 |
| BR-020 | ID-PAY-01..08 |
| BR-021 | IM-PAY-05 |
| BR-022 | FI-PAY-02/03/08 |
| BR-023 | RC-PAY-01..08 |
| BR-024 | P1 — Shift Integration |
| PAY-02 | FI-PAY-04 |
| PAY-07 | AC-PAY-05 |
| C-16 | IM-PAY-06/07 |
| C-18 | RC-PAY-05/06 |

---

## 13. Definition of Done

- [ ] Idempotency Tests خضراء.
- [ ] Financial Tests خضراء.
- [ ] Receipt Numbering تحت التزامن.
- [ ] Activation Tests خضراء.
- [ ] Immutability Tests خضراء.
- [ ] Collections + PDF.
- [ ] Isolation + Audit.
- [ ] مفيش `double` في أي حساب مالي.
