# استراتيجية الاختبارات (Testing Strategy)

> **المرجع:** [`../Kammel_Project_Documentation/10_TESTING_STRATEGY.md`](../Kammel_Project_Documentation/10_TESTING_STRATEGY.md)  
> **الملفات المرتبطة:** [requirements-traceability](./architecture/requirements-traceability.md) · [security-strategy](./architecture/security-strategy.md) · [database-strategy](./architecture/database-strategy.md)

---

## 1. الفلسفة

> **الاختبار مش خطوة بعد الكود — هو تعريف "خلصت".**

| المبدأ | التطبيق |
|---|---|
| **Test What Matters** | المال، العزل، الـ State Machine، الـ Idempotency — أولوية قصوى. |
| **Test Pyramid** | Unit كتير، Integration متوسط، E2E قليل. |
| **Real Database** | Testcontainers PostgreSQL 16 — **ممنوع H2**. |
| **Tenant First** | كل Integration Test بيتشغل في Tenant Scope واضح. |
| **No Real PII** | بيانات وهمية بأرقام مصرية صالحة الشكل فقط. |
| **Fast Feedback** | Unit < 30 ثانية، Integration < 5 دقائق. |
| **Deterministic** | ممنوع `Thread.sleep` أو Time-dependent Tests بدون Clock Injection. |

---

## 2. هرم الاختبارات

```text
        ┌─────────────────────┐
        │  E2E / Pilot (5%)   │  سيناريوهات كاملة على Staging
        ├─────────────────────┤
        │ Integration (25%)   │  API + DB + Security + Tenant
        ├─────────────────────┤
        │ Unit (70%)          │  Domain Rules + Services
        └─────────────────────┘
```

| المستوى | الأداة | السرعة | ماذا يغطي |
|---|---|---|---|
| **Unit** | JUnit 5 + AssertJ + Mockito | مللي ثانية | Domain Logic، State Machine، الحسابات |
| **Repository** | `@DataJpaTest` + Testcontainers | ثواني | Queries، Indexes، Constraints |
| **Web Slice** | `@WebMvcTest` + MockMvc | ثواني | Validation، Envelope، Error Mapping |
| **Integration** | `@SpringBootTest` + Testcontainers | دقائق | End-to-End داخل التطبيق + RLS |
| **Architecture** | Spring Modulith | ثواني | Boundaries + Cycles |
| **Performance** | k6 / Gatling | دقائق | Latency + Throughput |
| **UAT** | Manual Script | أيام | Pilot حقيقي |

---

## 3. الأدوات والإعداد

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@TestConfiguration
public class TestcontainersConfig {

    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgres() {
        return new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("kammel_test")
            .withUsername("kammel_app")
            .withPassword("test");
    }
}
```

**قواعد:**
- Container واحد لكل Test Suite (Reuse) — مش لكل Test.
- Flyway بتشتغل قبل كل Suite.
- كل Test بيتشغل في Transaction وRollback (ما عدا اختبارات الـ Concurrency).
- `@DirtiesContext` ممنوع إلا لو لازم — بطيء.

---

## 4. أنواع الاختبارات بالتفصيل

### 4.1 Unit Tests

| المجال | أمثلة |
|---|---|
| **Money** | `MoneyUtil.toMajor(70000) == 700.00` — ممنوع `double`. |
| **Subscription State Machine** | كل Transition صالحة/غير صالحة. |
| **Renewal** | قبل الانتهاء + بعد الانتهاء + Grace. |
| **Freeze** | حساب الأيام + الحد الأقصى. |
| **Risk Formula** | كل Signal + الوزن + الحد الأقصى 100. |
| **Cooldown** | 3 أيام بين الرسائل. |
| **Time Window** | 9ص–9م القاهرة. |
| **Phone Validation** | `010/011/012/015` + 8 أرقام. |

```java
@Test
void renewBeforeExpiry_extendsFromCurrentEndDate() {
    Subscription sub = subscription().endDate(LocalDate.of(2026, 12, 10)).build();
    Plan plan = plan().durationDays(30).build();

    Subscription renewed = stateMachine.renew(sub, plan, LocalDate.of(2026, 12, 5));

    assertThat(renewed.getEndDate()).isEqualTo(LocalDate.of(2027, 1, 9));
}
```

### 4.2 Repository Tests

- كل Query مخصصة (Search، Collections، At-Risk).
- كل Unique/Check Constraint.
- Partial Indexes (Soft Delete).
- `@DataJpaTest` + `@AutoConfigureTestDatabase(replace = NONE)`.

### 4.3 Web Slice Tests

- Validation Errors → `400` + `details[]`.
- Envelope Shape → `success`, `message`, `data`, `timestamp`.
- Error Mapping → كل Exception للـ HTTP الصح.
- Auth → `401`/`403` بدون Context.

### 4.4 Integration Tests

- `@SpringBootTest(webEnvironment = RANDOM_PORT)` + `TestRestTemplate`/`MockMvc`.
- تغطي:
  - تسجيل عضو → اشتراك → دفع → تفعيل → Check-in → Risk → Notification.
  - Freeze → Unfreeze → تمديد.
  - Renewal قبل/بعد الانتهاء.
  - Partial Payments.
  - Excel Import (Dry Run + Commit).
  - Audit Log بعد كل عملية حساسة.

### 4.5 Architecture Tests

```java
class ArchitectureBoundaryTests {

    ApplicationModules modules = ApplicationModules.of(KammelApplication.class);

    @Test
    void verifiesModularStructure() {
        modules.verify();
    }

    @Test
    void noFeatureDependsOnReporting() {
        // Reporting is a leaf
    }
}
```

### 4.6 Migration Tests

| الاختبار | المعيار |
|---|---|
| Clean Migrate | Flyway من DB فاضية |
| Validate | Checksums |
| RLS Coverage | كل جدول فيه `org_id` عنده Policy |
| Immutability | `UPDATE`/`DELETE` على الجداول المحظورة → Exception |
| Sequence | 20 Threads → أرقام فريدة |

---

## 5. اختبار عزل المستأجرين (Tenant Isolation Suite) — إجباري

> **أي Feature تخرج للـ Production بدون الاختبار ده = مرفوضة.**

### 5.1 السيناريو الأساسي

```java
@Test
void tenantA_cannotReadTenantBData() {
    // given
    UUID orgA = fixtures.createOrg("alpha");
    UUID orgB = fixtures.createOrg("beta");
    Member memberB = fixtures.createMember(orgB, "01011112222");

    // when
    String tokenA = fixtures.login(orgA, "01000000001", "Password123!");

    // then
    restTemplate.exchange("/api/v1/members/" + memberB.getId(), GET, auth(tokenA), ...)
        .expectStatus().isNotFound();          // 404 مش 403

    restTemplate.exchange("/api/v1/members?search=...", GET, auth(tokenA), ...)
        .expectBody()
        .jsonPath("$.data.content[*].id").value(not(hasItem(memberB.getId())));
}
```

### 5.2 المصفوفة الإلزامية

| الاختبار | المتوقع |
|---|---|
| `GET /members` كـ Org A | كل النتائج `org_id = A` |
| `GET /members/{B}` | `404` |
| `PATCH /members/{B}` | `404` |
| `DELETE /members/{B}` | `404` |
| `POST /subscriptions` بـ `memberId` من B | `404` |
| `POST /payments` بـ `subscriptionId` من B | `404` |
| `POST /checkins` بعضو من B | `404` |
| `GET /audit-logs` كـ Org A | كل النتائج `org_id = A` |
| `GET /notifications/queue` كـ A | كل النتائج `org_id = A` |
| Raw JDBC كـ `kammel_app` بدون Context | صفر صفوف |
| Raw JDBC بـ Context A | صفوف A فقط |
| `UPDATE ... WHERE org_id = B` بـ Context A | 0 Rows Affected |

### 5.3 Raw RLS Test

```java
@Test
void rlsBlocksCrossTenantUpdate() {
    try (Connection c = dataSource.getConnection()) {
        c.createStatement().execute("SET LOCAL app.current_org = '" + orgA + "'");
        int updated = c.createStatement().executeUpdate(
            "UPDATE members SET full_name = 'Hacked' WHERE org_id = '" + orgB + "'");
        assertThat(updated).isZero();
    }
}
```

---

## 6. اختبارات التزامن (Concurrency Tests)

| السيناريو | الإعداد | المعيار |
|---|---|---|
| **Payment Idempotency** | 10 Threads بنفس `Idempotency-Key` | Payment واحد فقط + نفس الـ Response |
| **Receipt Numbers** | 20 Threads تسجل دفع | 20 رقم فريد ومتسلسل بدون Gaps |
| **Session Deduction** | 5 Threads على اشتراك بـ 3 جلسات | 3 نجاح + 2 رفض `NO_SESSIONS_REMAINING` |
| **Subscription Renewal** | 2 Threads تجديد نفس الاشتراك | واحد ينجح + واحد يرفض/Idempotent |
| **Freeze** | طلبين تجميد متزامنين | واحد ينجح + واحد `INVALID_STATE_TRANSITION` |
| **Check-in Duplicate** | طلبين في نفس اللحظة | 2 Check-ins (Warning مش Block) |

```java
@Test
void concurrentPaymentsWithSameKey_createOnePayment() throws Exception {
    int threads = 10;
    ExecutorService pool = Executors.newFixedThreadPool(threads);
    CountDownLatch start = new CountDownLatch(1);
    List<Future<ResponseEntity<PaymentResponse>>> futures = new ArrayList<>();

    for (int i = 0; i < threads; i++) {
        futures.add(pool.submit(() -> {
            start.await();
            return postPayment(SAME_KEY, SAME_BODY);
        }));
    }
    start.countDown();

    Set<UUID> ids = futures.stream().map(f -> f.get().getBody().data().id())
        .collect(toSet());

    assertThat(ids).hasSize(1);
    assertThat(paymentRepository.count()).isEqualTo(1);
}
```

---

## 7. اختبارات الأمان

- كل بند في [Security Test Checklist](./architecture/security-strategy.md#11-قائمة-اختبارات-الأمان-security-test-checklist) له Test.
- `@WithMockUser` ممنوع في اختبارات الصلاحيات الحقيقية — استخدم JWT حقيقي.
- اختبارات Branch Scoping وTrainer Scoping إجبارية.
- اختبارات Rate Limiting (Caffeine) — استخدم Clock قابل للتحكم.

---

## 8. اختبارات الـ Jobs والـ Scheduler

| الاختبار | الطريقة |
|---|---|
| Expiration Job | Clock ثابت + بيانات End Date مختلفة |
| Grace Transition | `grace_days > 0` / `= 0` |
| Freeze Auto-Unfreeze | `scheduled_unfreeze_date` في الماضي |
| Risk Job | بيانات حضور مصطنعة + توقع Score |
| Notification Job | Cooldown + Opt-in + Time Window |
| ShedLock | تشغيل متزامن على Instance‑ين → مرة واحدة |
| Tenant Scope | Org‑ين → كل واحد بياناته فقط |

> **ممنوع** اختبار Job بـ `Thread.sleep`. استخدم `Clock` + استدعاء مباشر للـ Method.

---

## 9. اختبارات الأداء (Performance)

| المسار | الهدف | الأداة |
|---|---|---|
| Check-in | P95 < 500ms (حد أقصى 1s) | k6 |
| Member Search | P95 < 300ms | k6 |
| Dashboard Summary | P95 < 300ms | k6 |
| Payment Recording | P95 < 400ms | k6 |
| Excel Import (500 rows) | < 3s | Integration |
| Login | P95 < 300ms | k6 |

**شروط:**
- على Staging ببيانات تحاكي 500 عضو / 20,000 Check-in.
- `EXPLAIN ANALYZE` لأي Query > 100ms.
- ممنوع N+1 — `hibernate.generate_statistics` في الـ Test.

---

## 10. بيانات الاختبار (Test Fixtures)

```java
public final class TestFixtures {

    public static OrganizationBuilder org(String slug) { ... }
    public static MemberBuilder member(UUID orgId) { ... }
    public static SubscriptionBuilder activeSubscription(UUID orgId, UUID memberId) { ... }
    public static PaymentBuilder payment(UUID orgId, UUID subscriptionId) { ... }
}
```

**قواعد:**
- Builder Pattern — ممنوع Constructors طويلة.
- أرقام موبايل مصرية صالحة الشكل فقط (`01000000001`).
- أسماء وهمية.
- ممنوع نسخ بيانات Pilot الحقيقية للـ Test.
- كل Fixture بترجع Entity محفوظة (Persisted) أو Builder واضح.

---

## 11. التغطية (Coverage)

| النطاق | الحد الأدنى |
|---|---|
| Domain / State Machine | **90%** |
| Services | **80%** |
| Overall Line Coverage | **70%** |
| Critical Paths (Payment, Subscription, RLS) | **100% Branches** |

**أدوات:** JaCoCo — الـ Build بيفشل تحت الحد.

> **ملاحظة:** التغطية مش هدف في حد ذاتها. Test بيغطي سطر بدون Assert مفيد = مفيش قيمة.

---

## 12. CI Pipeline

```yaml
# .github/workflows/ci.yml (مبسط)
jobs:
  build:
    steps:
      - checkout
      - setup-java 21
      - cache maven
      - run: ./mvnw -B verify
        # يشمل:
        # 1. Compile
        # 2. Unit Tests
        # 3. Architecture Tests
        # 4. Integration Tests (Testcontainers)
        # 5. Tenant Isolation Suite
        # 6. JaCoCo Check
        # 7. SpotBugs / Checkstyle
        # 8. Secret Scan
```

| البوابة | الشرط |
|---|---|
| Compile | صفر أخطاء |
| Unit | 100% Pass |
| Integration | 100% Pass |
| Isolation | 100% Pass — **Build Fails لو فشل** |
| Coverage | ≥ الحد |
| Arch | مفيش Cycles |
| Secret Scan | صفر Findings |
| Migration | Clean Migrate ناجح |

---

## 13. تسمية الاختبارات

```text
{Method}_{Scenario}_{ExpectedResult}
```

أمثلة:
- `renew_beforeExpiry_extendsFromCurrentEndDate`
- `freeze_exceedingMaxDays_throwsBusinessRuleViolation`
- `payment_sameIdempotencyKey_returnsOriginal`
- `checkIn_frozenSubscription_returns422`

**ترتيب:** `given / when / then` — أو `// arrange / act / assert`.

---

## 14. Definition of Done للاختبارات

أي Feature متعتبرش خلصت غير لما:

- [ ] Unit Tests لكل Domain Rule.
- [ ] Repository Test لكل Query مخصصة.
- [ ] Integration Test للـ Happy Path.
- [ ] Integration Test لكل Error Path أساسي.
- [ ] **Tenant Isolation Test** للـ Endpoints الجديدة.
- [ ] Security Test للصلاحيات.
- [ ] Concurrency Test لو فيه Counter/Sequence.
- [ ] Migration Test.
- [ ] Coverage ≥ الحد.
- [ ] الـ Tests مستقرة (مفيش Flaky).
- [ ] Test IDs مربوطة بـ BR في [requirements-traceability](./architecture/requirements-traceability.md).

---

## 15. سياسة الـ Flaky Tests

1. أي Test فشل عشوائياً → `@Disabled` مع Issue + Deadline 48 ساعة.
2. ممنوع `Thread.sleep` — استخدم `Awaitility`.
3. ممنوع الاعتماد على ترتيب الـ Tests.
4. ممنوع الوقت الحقيقي — `Clock` ثابت.
5. الـ Concurrency Tests تتشغل في CI منفصلة عن الـ Suite العادية.

---

## 16. اختبارات الـ Pilot (UAT)

| السيناريو | المعيار |
|---|---|
| Receptionist يسجل عضو جديد | < 2 دقيقة |
| تسجيل دفع + طباعة إيصال | < 1 دقيقة |
| Check-in بالبحث/QR | < 5 ثواني |
| تجديد عضو منتهي | < 2 دقيقة |
| صاحب الجيم يفتح Dashboard | < 3 ثواني |
| 14 يوم متواصل بدون رجوع للدفتر | 0 أيام رجوع |
| 70% من الحضور متسجل | تقرير أسبوعي |

---

## 17. مرتبط بـ

- [requirements-traceability](./architecture/requirements-traceability.md) — BR ↔ Test.
- [security-strategy](./architecture/security-strategy.md) — Security Checklist.
- [database-strategy](./architecture/database-strategy.md) — Migration/RLS Tests.
- [testing-plan](./features/) — خطة كل Feature.
