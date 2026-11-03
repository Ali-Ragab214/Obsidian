# Kammel — Testing Strategy & Quality Assurance

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) · [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md) · [08_BUSINESS_LOGIC_AND_STATE_MACHINES.md](./08_BUSINESS_LOGIC_AND_STATE_MACHINES.md)

---

## 1. Testing Philosophy & Priority Hierarchy

In a financial, multi-tenant SaaS application, software defects carry asymmetric consequences. A UI cosmetic glitch is an inconvenience; a tenant data leak or financial double-charge is catastrophic.

### The Kammel Test Priority Hierarchy:
1. **Tier 1 (Non-Negotiable - 100% Pass in CI):** Multi-Tenant Data Isolation & RLS Security Tests.
2. **Tier 2 (Critical Business Accuracy):** Financial Transactions, Idempotency, and Subscription State Transitions.
3. **Tier 3 (Operational Continuity):** Attendance Check-In Verification, Background Scheduler Expirations.
4. **Tier 4 (Standard Coverage):** General CRUD operations, DTO mapping, pagination, and sorting.

```
          / \
         / E2E \           <-- Few smoke tests (User login to check-in)
        /-------\
       / Integr. \         <-- Testcontainers PostgreSQL + RLS + Isolation
      /-----------\
     / WebMvc Slice\       <-- Controller validation & RBAC security
    /---------------\
   /   Unit Tests    \     <-- Domain logic, State Machine, Money arithmetic
  /-------------------\
```

---

## 2. Testing Frameworks & Tooling

| Framework / Tool | Usage & Purpose |
|---|---|
| **JUnit 5 (Jupiter)** | Core test engine, parameterized tests, and assertions. |
| **Mockito** | Unit testing mocking framework for isolated service-layer unit tests. |
| **AssertJ** | Fluent, readable assertions (`assertThat(sub.getStatus()).isEqualTo(ACTIVE)`). |
| **MockMvc** | Controller testing without booting a full embedded web server. |
| **Testcontainers** | Boots a real, containerized PostgreSQL 16 instance during integration tests. *(Never use H2 in tests; H2 lacks PostgreSQL RLS and custom session variables)*. |
| **Spring Security Test** | `@WithMockUser` and custom `@WithMockKammelUser` security context helpers. |

---

## 3. Tier 1: Multi-Tenant Data Isolation Test Suite

This automated integration test runs on every pull request. If any endpoint reveals or mutates cross-tenant data, the build is blocked immediately.

### Sample Implementation (`TenantIsolationIntegrationTest.java`):

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class TenantIsolationIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("kammel_test")
            .withUsername("kammel_app")
            .withPassword("secret");

    @Autowired private MockMvc mockMvc;
    @Autowired private TestDataFactory testDataFactory;

    private TenantCredentials gymAlpha;
    private TenantCredentials gymBeta;

    @BeforeEach
    void setupTenants() {
        // Provision two independent gym organizations with test data
        gymAlpha = testDataFactory.createTenantWithStaffAndMembers("Gym Alpha");
        gymBeta = testDataFactory.createTenantWithStaffAndMembers("Gym Beta");
    }

    @Test
    @DisplayName("Gym Alpha user must never read Gym Beta members")
    void testTenantCannotReadOtherTenantMembers() throws Exception {
        UUID betaMemberId = gymBeta.getFirstMemberId();

        mockMvc.perform(get("/api/v1/members/" + betaMemberId)
                .header("Authorization", "Bearer " + gymAlpha.getAccessToken()))
                .andExpect(status().isNotFound()); // Must return 404, never 200 or 403
    }

    @Test
    @DisplayName("Gym Alpha user querying member list receives only Gym Alpha records")
    void testMemberListStrictlyScoped() throws Exception {
        mockMvc.perform(get("/api/v1/members")
                .header("Authorization", "Bearer " + gymAlpha.getAccessToken()))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content[*].branchId").value(everyItem(isIn(gymAlpha.getBranchIds()))));
    }

    @Test
    @DisplayName("Gym Alpha cannot renew a Gym Beta subscription")
    void testCannotMutateOtherTenantSubscription() throws Exception {
        UUID betaSubId = gymBeta.getFirstSubscriptionId();

        mockMvc.perform(post("/api/v1/subscriptions/" + betaSubId + "/renew")
                .header("Authorization", "Bearer " + gymAlpha.getAccessToken())
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"planId\": \"" + gymAlpha.getFirstPlanId() + "\"}"))
                .andExpect(status().isNotFound());
    }
}
```

---

## 4. Tier 2: Financial & Payment Tests

### 4.1 Payment Idempotency Test
```java
@Test
@DisplayName("Duplicate payment requests with identical Idempotency-Key are processed exactly once")
void testPaymentIdempotency() throws Exception {
    String idempotencyKey = UUID.randomUUID().toString();
    RecordPaymentRequest request = new RecordPaymentRequest(subscriptionId, 70000L, PaymentMethod.CASH, null, "Test");

    // First request - succeeds and creates payment
    MvcResult firstResult = mockMvc.perform(post("/api/v1/payments")
            .header("Authorization", "Bearer " + gymToken)
            .header("Idempotency-Key", idempotencyKey)
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andReturn();

    // Second request with same key - returns existing result without second insertion
    MvcResult secondResult = mockMvc.perform(post("/api/v1/payments")
            .header("Authorization", "Bearer " + gymToken)
            .header("Idempotency-Key", idempotencyKey)
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isOk())
            .andReturn();

    // Verify DB count
    assertThat(paymentRepository.countBySubscriptionId(subscriptionId)).isEqualTo(1);
}
```

### 4.2 Partial Payments & Auto-Activation Test
```java
@Test
@DisplayName("Partial payments accumulate until full price snapshot is reached, then activate subscription")
void testPartialPaymentActivation() {
    Subscription sub = subscriptionService.createSubscription(memberId, monthlyPlanId); // Price: 70,000 piasters
    assertThat(sub.getStatus()).isEqualTo(SubscriptionStatus.PENDING_PAYMENT);

    // Pay 30,000 piasters
    paymentService.recordPayment(sub.getId(), 30000L, PaymentMethod.CASH);
    sub = subscriptionRepository.findById(sub.getId()).orElseThrow();
    assertThat(sub.getStatus()).isEqualTo(SubscriptionStatus.PENDING_PAYMENT);

    // Pay remaining 40,000 piasters
    paymentService.recordPayment(sub.getId(), 40000L, PaymentMethod.CASH);
    sub = subscriptionRepository.findById(sub.getId()).orElseThrow();
    assertThat(sub.getStatus()).isEqualTo(SubscriptionStatus.ACTIVE);
}
```

---

## 5. Tier 3: State Machine & Transition Tests

An exhaustive parameterized test verifies all valid transitions succeed and all invalid transitions throw `InvalidStateTransitionException`.

```java
@ParameterizedTest
@CsvSource({
    "PENDING_PAYMENT, CANCELLED, true",
    "ACTIVE, FROZEN, true",
    "ACTIVE, GRACE, true",
    "ACTIVE, EXPIRED, true",
    "FROZEN, ACTIVE, true",
    "GRACE, EXPIRED, true",
    "EXPIRED, ACTIVE, false",   // Invalid! Must create new subscription
    "CANCELLED, ACTIVE, false", // Invalid!
    "PENDING_PAYMENT, FROZEN, false", // Invalid!
    "FROZEN, FROZEN, false"     // Invalid!
})
void testSubscriptionStateTransitions(SubscriptionStatus from, SubscriptionStatus to, boolean expectedValid) {
    Subscription sub = createSubscriptionInState(from);

    if (expectedValid) {
        assertThatCode(() -> subscriptionStateMachine.transition(sub, to, testActor, "Reason"))
                .doesNotThrowAnyException();
        assertThat(sub.getStatus()).isEqualTo(to);
    } else {
        assertThatThrownBy(() -> subscriptionStateMachine.transition(sub, to, testActor, "Reason"))
                .isInstanceOf(InvalidStateTransitionException.class);
    }
}
```

---

## 6. Continuous Integration (CI) Quality Gates

The GitHub Actions pipeline enforces the following gates before merging to `main`:
1. **Compilation & Checkstyle:** Zero compile warnings or lint failures.
2. **Unit & Slice Tests:** 100% pass rate.
3. **Testcontainers Integration Tests:** Full PostgreSQL migration chain applied cleanly via Flyway; 100% multi-tenant isolation tests pass.
4. **Code Coverage:** Minimum 85% line coverage on domain and service layers; 100% branch coverage on financial calculation classes (`PaymentService`, `MoneyUtil`).
