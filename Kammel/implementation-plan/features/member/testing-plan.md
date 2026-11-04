# Member — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| تسجيل سريع | < 2 ثانية |
| منع التكرار | Phone Unique per Org |
| Soft Delete | بيانات محفوظة + مش ظاهرة |
| Import دقيق | Row-by-Row Errors |
| Trainer Scoping | عملاؤه فقط |
| Tenant Isolation | صفر تسريب |

---

## 2. Unit Tests

| # | الاختبار | المتوقع |
|---|---|---|
| UT-MEM-01 | `Member.softDelete(now)` | `deletedAt` + `INACTIVE` |
| UT-MEM-02 | `activate` / `deactivate` | الحالة صح |
| UT-MEM-03 | Code Format `M-%06d` | `M-000001` |
| UT-MEM-04 | Phone Normalization `+2010...` | `010...` |
| UT-MEM-05 | Phone غير صالح | `ConstraintViolation` |
| UT-MEM-06 | `whatsappOptIn` false → true | `whatsappOptInAt` متظبط |
| UT-MEM-07 | Import Row Validation | Error Code صح |
| UT-MEM-08 | Date Parsing `dd/MM/yyyy` | LocalDate صح |

---

## 3. Repository Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RT-MEM-01 | `findByOrgIdAndPhoneAndDeletedAtIsNull` | Optional |
| RT-MEM-02 | نفس Phone في Org تانية | مسموح |
| RT-MEM-03 | Soft-Deleted Phone | مسموح تسجيل جديد |
| RT-MEM-04 | `findByQrToken` | Optional |
| RT-MEM-05 | `findByMemberCode` | Optional |
| RT-MEM-06 | Trainer Filter | أعضاء الكابتن فقط |
| RT-MEM-07 | `MAX(member_code)` | الرقم الصح |

---

## 4. API Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-MEM-01 | تسجيل عضو صحيح | `201` + Code + QR |
| AT-MEM-02 | Phone مكرر | `409` + `memberId` |
| AT-MEM-03 | Phone غير صالح | `400` + `details` |
| AT-MEM-04 | Branch من Org تانية | `404` |
| AT-MEM-05 | Trainer غير موجود | `422` |
| AT-MEM-06 | Search بالـ Phone | نتيجة واحدة |
| AT-MEM-07 | Search بالاسم | نتائج مطابقة |
| AT-MEM-08 | Search بالكود | نتيجة واحدة |
| AT-MEM-09 | Update Phone | يشتغل + Audit |
| AT-MEM-10 | Update `memberCode` | مرفوض/متجاهل |
| AT-MEM-11 | Soft Delete | `204` + مش ظاهر |
| AT-MEM-12 | Delete بـ Receptionist | `403` |
| AT-MEM-13 | Lookup Phone | `200` |
| AT-MEM-14 | Lookup غير موجود | `404` |

---

## 5. Import Tests

| # | السيناريو | المتوقع |
|---|---|---|
| IM-MEM-01 | Dry Run 350 صف | تقرير بدون كتابة |
| IM-MEM-02 | Commit 350 صف | 348 عضو + تقرير |
| IM-MEM-03 | Phone مكرر في الملف | صفوف مكررة = خطأ |
| IM-MEM-04 | Phone مكرر في DB | `DUPLICATE_IN_DB` |
| IM-MEM-05 | Branch غير موجود | `BRANCH_NOT_FOUND` |
| IM-MEM-06 | `plan_name` غير موجود | `PLAN_NOT_FOUND` |
| IM-MEM-07 | `end_date < start_date` | `INVALID_DATE_RANGE` |
| IM-MEM-08 | ملف `.xlsm` | `400` |
| IM-MEM-09 | ملف 20MB | `413` |
| IM-MEM-10 | 5000 صف | < 3s |
| IM-MEM-11 | Commit مع خطأ DB | Rollback كامل |
| IM-MEM-12 | Import مع Plan | Subscription `ACTIVE` + Event |

---

## 6. Concurrency Tests

```java
@Test
void concurrentMemberCreation_generatesUniqueCodes() throws Exception {
    int threads = 20;
    ExecutorService pool = Executors.newFixedThreadPool(threads);
    CountDownLatch start = new CountDownLatch(1);
    Set<String> codes = ConcurrentHashMap.newKeySet();

    for (int i = 0; i < threads; i++) {
        final int n = i;
        pool.submit(() -> {
            start.await();
            MemberResponse m = createMember("0100000" + String.format("%04d", n));
            codes.add(m.memberCode());
            return null;
        });
    }
    start.countDown();
    pool.shutdown();
    pool.awaitTermination(30, SECONDS);

    assertThat(codes).hasSize(threads);
}
```

| # | الاختبار | المتوقع |
|---|---|---|
| CT-MEM-01 | 20 إنشاء متزامن | 20 كود فريد |
| CT-MEM-02 | نفس Phone متزامن | واحد ينجح + واحد `409` |
| CT-MEM-03 | QR Tokens | فريدة |

---

## 7. Isolation & Scoping Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IT-MEM-01 | `GET /members` كـ Org A | أعضاء A فقط |
| IT-MEM-02 | `GET /members/{B}` | `404` |
| IT-MEM-03 | `PATCH /members/{B}` | `404` |
| IT-MEM-04 | `DELETE /members/{B}` | `404` |
| IT-MEM-05 | Receptionist في فرع A2 | أعضاء فرعه فقط |
| IT-MEM-06 | Trainer | عملاؤه فقط |
| IT-MEM-07 | Trainer يفتح عضو غيره | `404` |
| IT-MEM-08 | Raw JDBC بدون Context | صفر صفوف |

---

## 8. Status Sync Tests

| # | السيناريو | المتوقع |
|---|---|---|
| SS-MEM-01 | `SubscriptionActivated` | Member `ACTIVE` |
| SS-MEM-02 | آخر اشتراك `Expired` | Member `INACTIVE` |
| SS-MEM-03 | اشتراك تاني نشط | Member يفضل `ACTIVE` |
| SS-MEM-04 | Soft Delete | `INACTIVE` |
| SS-MEM-05 | Event مكرر | Idempotent |

---

## 9. PII / Security

| # | الاختبار | المتوقع |
|---|---|---|
| SC-MEM-01 | Log تسجيل عضو | مفيش Phone/Name |
| SC-MEM-02 | Body فيه `orgId` | متجاهل |
| SC-MEM-03 | Receptionist Delete | `403` |
| SC-MEM-04 | Trainer Import | `403` |
| SC-MEM-05 | Photo URL خارجي | مسموح (بدون Fetch) |

---

## 10. Performance

| المسار | الهدف |
|---|---|
| `POST /members` | P95 < 300ms |
| `GET /members?search=` | P95 < 300ms |
| `POST /members/import` (500 صف) | < 3s |
| `GET /members/lookup` | P95 < 100ms |

---

## 11. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-003 | IM-MEM-03, IM-MEM-04 |
| BR-011 | AT-MEM-06 |
| BR-012 | RT-MEM-02, RT-MEM-03, CT-MEM-02 |
| BR-013 | AT-MEM-01 |
| BR-037 | IT-MEM-06, IT-MEM-07 |

---

## 12. Definition of Done

- [ ] Unit + Repo + API + Import Tests.
- [ ] Concurrency Test للأكواد.
- [ ] Isolation + Trainer Scoping.
- [ ] Status Sync.
- [ ] PII Test.
- [ ] 500 صف < 3s.
- [ ] كل الاختبارات مستقرة.
