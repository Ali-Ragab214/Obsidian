# Notification — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Queue Generation | Trigger صحيح + أولوية |
| Cooldown | 3 أيام |
| Time Window | 9–21 Cairo |
| Opt-in | صفر رسائل لـ Opt-out |
| Lifecycle | 3 حالات + Audit |
| Attribution | 7 أيام |
| Isolation | صفر تسريب |

---

## 2. Queue Generation Tests

| # | السيناريو | المتوقع |
|---|---|---|
| QG-NOT-01 | `ACTIVE` + `endDate = today+7` | `RENEWAL_REMINDER` |
| QG-NOT-02 | `ACTIVE` + `today+3` | `RENEWAL_REMINDER` |
| QG-NOT-03 | `ACTIVE` + `today+1` | `RENEWAL_REMINDER` |
| QG-NOT-04 | `ACTIVE` + `today+5` | مفيش رسالة |
| QG-NOT-05 | `GRACE` | `GRACE_EXPOSURE` |
| QG-NOT-06 | `CRITICAL` + بعيد عن الانتهاء | `MISS_YOU` |
| QG-NOT-07 | `HIGH` + بعيد | `MISS_YOU` |
| QG-NOT-08 | `MEDIUM` | مفيش رسالة |
| QG-NOT-09 | `GRACE` + `CRITICAL` | `GRACE_EXPOSURE` (أولوية) |
| QG-NOT-10 | `RENEWAL` + `CRITICAL` | `RENEWAL_REMINDER` (أولوية) |
| QG-NOT-11 | Opt-out | مفيش رسالة |
| QG-NOT-12 | عضو محذوف | مفيش رسالة |
| QG-NOT-13 | عضو غير نشط | مفيش رسالة |
| QG-NOT-14 | Job مرتين نفس اليوم | رسالة واحدة |
| QG-NOT-15 | Org‑ين | كل واحدة طابورها |

```java
@Test
void graceTakesPriorityOverMissYou() {
    fixtures.memberWithGraceSubscription("Ahmed");
    fixtures.criticalRiskScore("Ahmed", 90);

    job.run();

    MessageQueueItem msg = messageRepository.findAll().getFirst();
    assertThat(msg.getTemplate()).isEqualTo(MessageTemplate.GRACE_EXPOSURE);
}
```

---

## 3. Cooldown Tests

| # | السيناريو | المتوقع |
|---|---|---|
| CD-NOT-01 | رسالة قبل 2 يوم | Skip |
| CD-NOT-02 | رسالة قبل 3 يوم بالظبط | مسموح |
| CD-NOT-03 | رسالة قبل 4 يوم | مسموح |
| CD-NOT-04 | رسالة `CANCELLED` قبل يوم | مسموح |
| CD-NOT-05 | رسالة `FAILED` قبل يوم | مسموح (P1) |
| CD-NOT-06 | رسالة `PENDING` النهارده | Skip |
| CD-NOT-07 | رسالة `SENT_CONFIRMED` قبل يوم | Skip |
| CD-NOT-08 | Template مختلف قبل يومين | Skip (Global) |

---

## 4. Time Window Tests

| # | الوقت (Cairo) | المتوقع |
|---|---|---|
| TW-NOT-01 | 08:59 | Queue فاضي + `availableAt` |
| TW-NOT-02 | 09:00 | Queue ظاهر |
| TW-NOT-03 | 20:59 | Queue ظاهر |
| TW-NOT-04 | 21:00 | Queue فاضي |
| TW-NOT-05 | 22:30 | `PATCH opened` → `422` |
| TW-NOT-06 | 08:00 | `PATCH sent` → `422` |
| TW-NOT-07 | 12:00 | `PATCH opened/sent` → `200` |
| TW-NOT-08 | DST (لو اتغير) | `Africa/Cairo` |

> استخدم `Clock.fixed(...)` — ممنوع الوقت الحقيقي في الـ Tests.

---

## 5. Lifecycle Tests

| # | From | Action | To | المتوقع |
|---|---|---|---|---|
| LC-NOT-01 | `PENDING` | Open | `OPENED` | ✅ + `openedAt` |
| LC-NOT-02 | `OPENED` | Open | `OPENED` | ✅ Idempotent |
| LC-NOT-03 | `OPENED` | Sent | `SENT_CONFIRMED` | ✅ + `sentAt` + `sentBy` |
| LC-NOT-04 | `SENT_CONFIRMED` | Sent | `SENT_CONFIRMED` | ✅ Idempotent |
| LC-NOT-05 | `PENDING` | Sent | — | ❌ `409` |
| LC-NOT-06 | `SENT_CONFIRMED` | Cancel | — | ❌ `409` |
| LC-NOT-07 | `PENDING` | Cancel | `CANCELLED` | ✅ |
| LC-NOT-08 | `OPENED` | Cancel | `CANCELLED` | ✅ |
| LC-NOT-09 | `CANCELLED` | Open | — | ❌ `409` |

---

## 6. Opt-in Tests

| # | السيناريو | المتوقع |
|---|---|---|
| OI-NOT-01 | `whatsappOptIn = true` | رسالة |
| OI-NOT-02 | `whatsappOptIn = false` | مفيش رسالة |
| OI-NOT-03 | Opt-out بعد التوليد | `PATCH` → `422` |
| OI-NOT-04 | Opt-in تاني | رسائل جديدة |
| OI-NOT-05 | رقم غير مصري | Job يتخطى + Log |

---

## 7. Rendering + Link Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RL-NOT-01 | `01012345678` | `https://wa.me/201012345678?...` |
| RL-NOT-02 | Arabic Text | URL Encoded |
| RL-NOT-03 | Emojis | Encoded |
| RL-NOT-04 | متغير ناقص | Exception |
| RL-NOT-05 | `{unknown}` | Exception |
| RL-NOT-06 | Link Length | ≤ DB Limit (`TEXT`) |
| RL-NOT-07 | `memberName` | موجود في النص |
| RL-NOT-08 | `gymName` | موجود في النص |
| RL-NOT-09 | `GRACE_EXPOSURE` | `graceEndDate` صح |
| RL-NOT-10 | `MISS_YOU` | `daysAbsent` صح |

---

## 8. Immutability Tests

| # | الاختبار | المتوقع |
|---|---|---|
| IM-NOT-01 | `UPDATE rendered_text` | DB Exception |
| IM-NOT-02 | `UPDATE template` | DB Exception |
| IM-NOT-03 | `UPDATE member_id` | DB Exception |
| IM-NOT-04 | `UPDATE due_date` | DB Exception |
| IM-NOT-05 | `DELETE` | DB Exception |
| IM-NOT-06 | `UPDATE status PENDING → SENT_CONFIRMED` | DB Trigger يرفض |
| IM-NOT-07 | `UPDATE status OPENED → SENT_CONFIRMED` | مسموح |
| IM-NOT-08 | Service يعدل النص | مفيش Setter |

---

## 9. Attribution Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-NOT-01 | رسالة `SENT_CONFIRMED` قبل 5 أيام + تجديد | Attributed |
| AT-NOT-02 | رسالة قبل 8 أيام | Not Attributed |
| AT-NOT-03 | رسالة `OPENED` | Not Attributed |
| AT-NOT-04 | رسالة `PENDING` | Not Attributed |
| AT-NOT-05 | رسالتين — الأحدث | الأحدث فقط |
| AT-NOT-06 | تجديدين | رسالة واحدة لكل تجديد |
| AT-NOT-07 | Attribution مرتين | ثابت |
| AT-NOT-08 | Value | `price_minor_snapshot` |
| AT-NOT-09 | Attribution Rate | حساب صحيح |
| AT-NOT-10 | `byTemplate` | Breakdown صحيح |

---

## 10. Isolation + Audit

| # | الاختبار | المتوقع |
|---|---|---|
| IT-NOT-01 | Queue كـ Org A | A فقط |
| IT-NOT-02 | Message من Org B | `404` |
| IT-NOT-03 | Manager فرع B | `404` |
| IT-NOT-04 | Raw JDBC بدون Context | صفر صفوف |
| AU-NOT-01 | `SENT_CONFIRMED` | `MESSAGE_SENT` |
| AU-NOT-02 | `CANCELLED` | `MESSAGE_CANCELLED` |
| AU-NOT-03 | Open | مفيش Audit |
| AU-NOT-04 | Actor | المستخدم الحالي |

---

## 11. Performance

| المسار | الهدف |
|---|---|
| Job 500 عضو | < 2 دقيقة |
| `GET /queue` | P95 < 300ms |
| `PATCH /sent` | P95 < 200ms |
| Attribution Listener | < 100ms |
| `GET /attribution` | P95 < 500ms |

---

## 12. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-033 | LC-NOT-02/03/05 |
| BR-034 | CD-NOT-01..08 |
| BR-035 | TW-NOT-01..08 |
| BR-036 | OI-NOT-01..05 |
| NOT-04 | CD-NOT-08 |
| NOT-08 | AT-NOT-01/02 |
| NOT-09 | QG-NOT-14 |

---

## 13. Definition of Done

- [ ] كل Triggers مغطاة.
- [ ] Cooldown + Time Window.
- [ ] Opt-in.
- [ ] Lifecycle + Audit.
- [ ] Rendering + Link.
- [ ] Immutability.
- [ ] Attribution + Stats.
- [ ] Isolation + Performance.
- [ ] مفيش رسالة تتبعت تلقائياً.
