# Audit — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. الأهداف

| الهدف | المعيار |
|---|---|
| Immutability | `UPDATE`/`DELETE` مرفوضين |
| Coverage | كل عملية حساسة لها Audit |
| Tenant Isolation | Org A مش شايف Audit Org B |
| No PII | مفيش Phone/Password/Token في السجلات |
| Transaction | Audit بيفشل مع العملية |

---

## 2. Unit Tests

| # | الاختبار | المتوقع |
|---|---|---|
| UT-AUD-01 | `AuditLog.create(cmd)` | كل الحقول من الـ Context |
| UT-AUD-02 | `recordSystem` | `actorType = SYSTEM`, `actorId = null` |
| UT-AUD-03 | `record` بدون `TenantContext` | `IllegalStateException` |
| UT-AUD-04 | `reason` مطلوب لـ Freeze | Exception |
| UT-AUD-05 | JSON Snapshot | Serialization صحيح |
| UT-AUD-06 | `AuditContext.currentIp` مع `X-Forwarded-For` | أول IP فقط |

---

## 3. Immutability Tests

```java
@Test
void updateAuditLogIsRejected() {
    AuditLog log = fixtures.auditLog(orgId);

    assertThatThrownBy(() -> jdbcTemplate.update(
        "UPDATE audit_log SET reason = 'hacked' WHERE id = ?", log.getId()))
        .hasMessageContaining("append-only");
}

@Test
void deleteAuditLogIsRejected() {
    assertThatThrownBy(() -> jdbcTemplate.update(
        "DELETE FROM audit_log WHERE id = ?", log.getId()))
        .hasMessageContaining("append-only");
}

@Test
void appRoleCannotUpdateAuditLog() {
    // Connect as kammel_app
    assertThatThrownBy(() -> stmt.executeUpdate("UPDATE audit_log SET reason = 'x'"))
        .isInstanceOf(SQLException.class);
}
```

| # | الاختبار | المتوقع |
|---|---|---|
| IM-AUD-01 | `UPDATE audit_log` | Exception `55000` |
| IM-AUD-02 | `DELETE audit_log` | Exception `55000` |
| IM-AUD-03 | `kammel_app` UPDATE | Permission Denied |
| IM-AUD-04 | `kammel_app` DELETE | Permission Denied |
| IM-AUD-05 | Insert صحيح | ينجح |

---

## 4. Coverage Tests (كل Feature)

| # | العملية | Action |
|---|---|---|
| CV-AUD-01 | Freeze | `SUBSCRIPTION_FROZEN` |
| CV-AUD-02 | Unfreeze | `SUBSCRIPTION_UNFROZEN` |
| CV-AUD-03 | Cancel | `SUBSCRIPTION_CANCELLED` |
| CV-AUD-04 | Payment | `PAYMENT_RECORDED` |
| CV-AUD-05 | Reversal | `PAYMENT_REVERSED` |
| CV-AUD-06 | Check-in Override | `CHECKIN_OVERRIDDEN` |
| CV-AUD-07 | Member Delete | `MEMBER_DELETED` |
| CV-AUD-08 | Plan Update | `PLAN_UPDATED` |
| CV-AUD-09 | User Role Change | `USER_ROLE_CHANGED` |
| CV-AUD-10 | Password Reset | `PASSWORD_RESET` |
| CV-AUD-11 | Login Failed | `LOGIN_FAILED` |
| CV-AUD-12 | Refresh Reuse | `REFRESH_TOKEN_REUSE` |

---

## 5. Transaction Tests

| # | السيناريو | المتوقع |
|---|---|---|
| TR-AUD-01 | Audit فشل أثناء Freeze | Subscription Rollback |
| TR-AUD-02 | Audit فشل أثناء Payment | Payment Rollback |
| TR-AUD-03 | `recordBestEffort` فشل | العملية تنجح + Log |
| TR-AUD-04 | Scheduler Expiration | `SYSTEM` Audit |

---

## 6. API Tests

| # | السيناريو | المتوقع |
|---|---|---|
| AT-AUD-01 | Owner `GET /audit-logs` | `200` |
| AT-AUD-02 | Manager `GET /audit-logs` | `403` |
| AT-AUD-03 | Filter بـ `action` | نتائج مطابقة |
| AT-AUD-04 | Filter بـ `from`/`to` | نتائج في الفترة |
| AT-AUD-05 | `GET /audit-logs/{otherOrg}` | `404` |
| AT-AUD-06 | `GET /audit-logs/entities/...` | تاريخ مرتب |
| AT-AUD-07 | `POST /audit-logs` | `405` |
| AT-AUD-08 | Pagination | `PageResponse` صحيح |
| AT-AUD-09 | `size=500` | `400` |

---

## 7. Tenant Isolation Tests

```java
@Test
void auditLogsAreIsolated() {
    fixtures.auditLog(orgA, "MEMBER_CREATED", memberA);
    fixtures.auditLog(orgB, "MEMBER_CREATED", memberB);

    String tokenA = fixtures.login(orgA);

    restTemplate.exchange("/api/v1/audit-logs", GET, auth(tokenA), ...)
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$.data.content[*].entityId").value(not(hasItem(memberB.getId())));
}
```

---

## 8. PII Tests

| # | الاختبار | المتوقع |
|---|---|---|
| PII-AUD-01 | Audit Payment | مفيش Phone/Name |
| PII-AUD-02 | Audit Login Failed | مفيش Password |
| PII-AUD-03 | Application Log | مفيش PII |
| PII-AUD-04 | Error Response | مفيش Stack Trace |
| PII-AUD-05 | Sentry Event | مفيش PII (Scrubbing) |

---

## 9. Performance

| العملية | الهدف |
|---|---|
| `record` | < 10ms |
| `GET /audit-logs` (100k rows) | < 300ms |
| `GET /audit-logs/entities/...` | < 100ms |

---

## 10. Definition of Done

- [ ] Unit + Immutability + Coverage + API Tests.
- [ ] Isolation Test.
- [ ] PII Test.
- [ ] Transaction Test.
- [ ] كل Action في الكتالوج له Test.
- [ ] مفيش `UPDATE`/`DELETE` ممكن على `audit_log`.
