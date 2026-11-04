# Tenancy — خطة الاختبارات

> **المرتبط:** [`README.md`](./README.md) · [testing-strategy](../../testing-strategy.md) · [requirements-traceability](../../architecture/requirements-traceability.md)

---

## 1. أهداف الاختبار

| الهدف | المعيار |
|---|---|
| عزل الفروع | Manager مش شايف فرع تاني |
| عزل المستأجرين | Org A مش شايف Org B |
| سلامة الـ Provisioning | Org + Branch + Owner في Transaction واحدة |
| منع تعطيل آخر فرع | `422` |
| منع الحذف | مفيش Endpoint + DB Grants |

---

## 2. Unit Tests

| # | الاختبار | المتوقع |
|---|---|---|
| UT-TEN-01 | `Organization.rename("  ")` | `BusinessException` |
| UT-TEN-02 | `Organization.rename("Gold")` | الاسم يتحدث + Trim |
| UT-TEN-03 | `Organization.suspend()` | `status = SUSPENDED` |
| UT-TEN-04 | `Branch.deactivate()` | `active = false` |
| UT-TEN-05 | `Branch.activate()` | `active = true` |
| UT-TEN-06 | Slug Regex صالح/غير صالح | قبول/رفض |
| UT-TEN-07 | Timezone غير صالح | `BusinessException` |

---

## 3. Repository Tests

| # | الاختبار | المتوقع |
|---|---|---|
| RT-TEN-01 | `findBySlug` موجود/غير موجود | Optional صحيح |
| RT-TEN-02 | `existsBySlug` | true/false |
| RT-TEN-03 | `countByOrgIdAndActiveTrue` | العدد الصح |
| RT-TEN-04 | `findAllByOrgIdOrderByNameAsc` | ترتيب أبجدي |
| RT-TEN-05 | `findAllActiveIds` | Org‑ات `ACTIVE` فقط |
| RT-TEN-06 | Insert Branch بـ `org_id` غير موجود | FK Exception |

---

## 4. API / Integration Tests

| # | السيناريو | المتوقع |
|---|---|---|
| IT-TEN-01 | Owner يعدل اسم الـ Org | `200` + Audit |
| IT-TEN-02 | Receptionist يعدل الـ Org | `403` |
| IT-TEN-03 | Owner ينشئ فرع | `201` + RLS Context صح |
| IT-TEN-04 | Body فيه `orgId` | يتجاهل / `400` |
| IT-TEN-05 | Owner يعطّل آخر فرع | `422` |
| IT-TEN-06 | Owner يعطّل فرع من 2 | `200` |
| IT-TEN-07 | Manager يفتح فرعه | `200` |
| IT-TEN-08 | Manager يفتح فرع تاني | `403` |
| IT-TEN-09 | Owner يفتح أي فرع | `200` |
| IT-TEN-10 | `DELETE /branches/{id}` | `405` |
| IT-TEN-11 | `GET /organization` بدون Token | `401` |
| IT-TEN-12 | Org `SUSPENDED` Login | `401` |

---

## 5. Tenant Isolation Tests (إجباري)

```java
@Test
void branchesAreIsolatedBetweenOrganizations() {
    UUID orgA = fixtures.createOrg("alpha");
    UUID orgB = fixtures.createOrg("beta");
    fixtures.createBranch(orgB, "Beta Main");

    String tokenA = fixtures.login(orgA);

    restTemplate.exchange("/api/v1/branches", GET, auth(tokenA), BranchPage.class)
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$.data.content[*].name").value(not(hasItem("Beta Main")));
}

@Test
void rawJdbcWithoutContextReturnsNoBranches() {
    try (Connection c = dataSource.getConnection()) {
        ResultSet rs = c.createStatement().executeQuery("SELECT count(*) FROM branches");
        rs.next();
        assertThat(rs.getInt(1)).isZero();
    }
}

@Test
void rawJdbcCannotUpdateOtherTenantBranch() {
    try (Connection c = dataSource.getConnection()) {
        c.createStatement().execute("SET LOCAL app.current_org = '" + orgA + "'");
        int updated = c.createStatement().executeUpdate(
            "UPDATE branches SET name = 'Hacked' WHERE org_id = '" + orgB + "'");
        assertThat(updated).isZero();
    }
}
```

---

## 6. Security Tests

| # | الاختبار | المتوقع |
|---|---|---|
| ST-TEN-01 | Receptionist `POST /branches` | `403` |
| ST-TEN-02 | Trainer `GET /branches/{other}` | `403`/`404` |
| ST-TEN-03 | Branch من Org تانية | `404` |
| ST-TEN-04 | JWT فيه `org_id` تانية | البيانات مش بتتغير |
| ST-TEN-05 | Body فيه `orgId` | مرفوض/متجاهل |

---

## 7. Migration Tests

| # | الاختبار | المتوقع |
|---|---|---|
| MT-TEN-01 | Clean Migrate | نجاح |
| MT-TEN-02 | RLS Coverage على `branches` | Policy موجودة |
| MT-TEN-03 | `organizations` بدون RLS | مقصود |
| MT-TEN-04 | `slug` Unique | Duplicate → Exception |
| MT-TEN-05 | `users.role` CHECK | قيمة غلط → Exception |
| MT-TEN-06 | `DELETE branches` كـ `kammel_app` | Permission Denied |

---

## 8. Provisioning Tests

| # | الاختبار | المتوقع |
|---|---|---|
| PT-TEN-01 | Provisioning ناجح | Org + Branch + Owner |
| PT-TEN-02 | Slug مكرر | `409` / فشل واضح |
| PT-TEN-03 | Phone Owner مكرر | فشل |
| PT-TEN-04 | Rollback عند فشل أي خطوة | مفيش Org ناقصة |
| PT-TEN-05 | Audit `ORGANIZATION_PROVISIONED` | Record موجود |

---

## 9. Performance

| المسار | الهدف |
|---|---|
| `GET /branches` (5 فروع) | < 100ms |
| `GET /organization` | < 50ms |
| Provisioning | < 2s |

---

## 10. ربط بالـ BRs

| BR | الاختبار |
|---|---|
| BR-001 | PT-TEN-01, IT-TEN-05 |
| BR-004 | IT-TEN-09, IT-TEN-07 |
| BR-008 | IT-TEN-07, ST-TEN-02 |
| BR-009 | IT-TEN-08, ST-TEN-03 |

---

## 11. Definition of Done

- [ ] Unit + Repository + API + Isolation Tests.
- [ ] RLS Coverage Test.
- [ ] Provisioning Test.
- [ ] مفيش Test معتمد على بيانات Pilot حقيقية.
- [ ] كل الاختبارات مستقرة في CI.
