# هيكل المشروع (Target Project Structure)

> **المرجع:** [`../../Kammel_Project_Documentation/03_SYSTEM_ARCHITECTURE.md`](../../Kammel_Project_Documentation/03_SYSTEM_ARCHITECTURE.md)  
> **الملفات المرتبطة:** [shared-foundation](./shared-foundation.md) · [feature-dependencies](./feature-dependencies.md)

---

## 1. الفلسفة

المشروع **Feature-Based Architecture (Vertical Slice Architecture)**، مش Layer-Based.

```text
❌ مرفوض: controller/ service/ repository/ entity/   (كل حاجة متفرقة)
✅ معتمد: features/member/{api,application,domain,infrastructure}
```

كل Feature:

- مالكة الـ Entities والـ Enums والـ Domain Rules بتاعتها.
- مالكة الـ DTOs والـ Validation بتاعتها.
- مالكة الـ Repositories والـ Services والـ Controllers بتاعتها.
- مالكة الـ Tests بتاعتها.
- بتتكلم مع الـ Features التانية من خلال **Service Interfaces** أو **Integration Events** فقط.

---

## 2. الهيكل العام (Package Root: `com.kammel`)

```text
com.kammel/
├── KammelApplication.java
│
├── shared/                              # Cross-Cutting Concerns فقط
│   ├── domain/
│   │   ├── BaseEntity.java
│   │   ├── BaseAuditableEntity.java
│   │   └── TenantScoped.java
│   ├── api/
│   │   ├── ApiResponse.java
│   │   ├── PageResponse.java
│   │   └── ErrorResponse.java
│   ├── exception/
│   │   ├── GlobalExceptionHandler.java
│   │   ├── BusinessException.java
│   │   ├── ResourceNotFoundException.java
│   │   ├── DuplicateResourceException.java
│   │   ├── InvalidStateTransitionException.java
│   │   ├── TenantMismatchException.java
│   │   └── IdempotencyConflictException.java
│   ├── security/
│   │   ├── TenantContext.java
│   │   ├── UserPrincipal.java
│   │   ├── JwtService.java
│   │   ├── JwtAuthenticationFilter.java
│   │   ├── SecurityUtils.java
│   │   ├── BranchSecurity.java
│   │   ├── TenantAwareJpaTransactionManager.java
│   │   └── TenantContextCleanupFilter.java
│   ├── persistence/
│   │   ├── TenantTransactionRunner.java
│   │   └── RlsSessionInitializer.java
│   ├── config/
│   │   ├── SecurityConfig.java
│   │   ├── JpaConfig.java
│   │   ├── SchedulingConfig.java
│   │   ├── OpenApiConfig.java
│   │   ├── JacksonConfig.java
│   │   └── AsyncConfig.java
│   ├── validation/
│   │   ├── ValidPhone.java
│   │   └── PhoneValidator.java
│   ├── util/
│   │   ├── MoneyUtil.java
│   │   ├── PhoneUtil.java
│   │   └── DateUtil.java
│   └── contracts/
│       └── events/
│           ├── PaymentRecordedEvent.java
│           ├── SubscriptionActivatedEvent.java
│           ├── SubscriptionRenewedEvent.java
│           └── SubscriptionExpiredEvent.java
│
├── features/
│   ├── tenancy/
│   ├── identity/
│   ├── member/
│   ├── plan/
│   ├── subscription/
│   ├── payment/
│   ├── checkin/
│   ├── risk/
│   ├── notification/
│   ├── reporting/
│   ├── audit/
│   ├── shift/                           # P1
│   ├── lead/                            # P1
│   └── member-portal/                   # P1
│
└── ai/                                  # P2/P3 فقط — مش في الـ MVP
    ├── churn/
    └── assistant/
```

---

## 3. التشريح الداخلي لأي Feature

```text
features/{feature}/
├── api/
│   ├── {Feature}Controller.java
│   ├── request/
│   │   └── Create{Entity}Request.java
│   ├── response/
│   │   └── {Entity}Response.java
│   └── mapper/
│       └── {Entity}WebMapper.java
│
├── application/
│   ├── {Feature}Service.java            # Public Interface
│   ├── {Feature}ServiceImpl.java        # Implementation
│   ├── {Feature}EventListener.java      # Integration Events
│   ├── {Feature}QueryService.java       # Read-only contract للـ Features التانية
│   └── mapper/
│       └── {Entity}Mapper.java
│
├── domain/
│   ├── {Entity}.java
│   ├── enums/
│   ├── vo/                              # Value Objects لو محتاجة
│   ├── exception/
│   └── event/
│
└── infrastructure/
    ├── {Entity}Repository.java
    ├── {Entity}Specifications.java
    ├── adapter/                         # External clients
    └── config/                          # Feature-specific config
```

### مسؤوليات الطبقات

| الطبقة | المسؤولية | ممنوع |
|---|---|---|
| `api/` | HTTP + Validation + DTO Mapping | Business Logic / Repository Calls |
| `application/` | Use Cases + Transactions + Authorization | JPA Entities تخرج للخارج |
| `domain/` | Entities + Rules + Enums + Domain Exceptions | Spring Web / JPA Repositories |
| `infrastructure/` | JPA + External Adapters | Business Decisions |

> **ملاحظة:** `domain/` هنا مش Pure Domain Model بمعنى DDD الكامل. هي مكان الـ JPA Entities وقواعدها. مش هنضيف Aggregate/Repository Interfaces بشكل مبالغ فيه لمشروع مطور واحد.

---

## 4. مثال كامل: `features/subscription`

```text
features/subscription/
├── api/
│   ├── SubscriptionController.java
│   ├── request/
│   │   ├── CreateSubscriptionRequest.java
│   │   ├── RenewSubscriptionRequest.java
│   │   ├── FreezeSubscriptionRequest.java
│   │   └── CancelSubscriptionRequest.java
│   └── response/
│       ├── SubscriptionResponse.java
│       └── SubscriptionEventResponse.java
│
├── application/
│   ├── SubscriptionService.java
│   ├── SubscriptionServiceImpl.java
│   ├── SubscriptionQueryService.java
│   ├── SubscriptionStateMachine.java
│   └── SubscriptionEventListener.java
│
├── domain/
│   ├── Subscription.java
│   ├── SubscriptionEvent.java
│   ├── SubscriptionStatus.java
│   ├── SubscriptionEventType.java
│   └── exception/
│       └── InvalidSubscriptionTransitionException.java
│
└── infrastructure/
    ├── SubscriptionRepository.java
    ├── SubscriptionEventRepository.java
    └── SubscriptionSpecifications.java
```

---

## 5. قواعد الحدود بين الـ Features

1. **ممنوع** Feature تستدعي Repository بتاع Feature تانية.
2. **ممنوع** JPA Relationship بين Entity في Feature وEntity في Feature تانية — استخدم `UUID` Reference.
3. **مسموح** JPA Relationships داخل نفس الـ Feature فقط.
4. **ممنوع** Circular Dependencies — الـ Graph لازم يكون DAG.
5. **ممنوع** Entity يخرج من الـ Feature — الـ Contracts Records/DTOs فقط.
6. **الأحداث** اللي بتعبر الـ Features تتسجل في `shared/contracts/events`.

### أمثلة صح/غلط

```java
// ❌ ممنوع: checkin يستدعي SubscriptionRepository مباشرة
subscriptionRepository.findById(subId);

// ✅ صح: checkin يستدعي SubscriptionService
subscriptionService.findAccessSubscription(memberId);

// ❌ ممنوع: Payment Entity فيها @ManyToOne Subscription من Feature تانية
@ManyToOne private Subscription subscription;

// ✅ صح: UUID Reference
@Column(name = "subscription_id") private UUID subscriptionId;
```

---

## 6. اختبار الحدود بـ Spring Modulith

```java
class ArchitectureBoundaryTests {
    ApplicationModules modules = ApplicationModules.of(KammelApplication.class);

    @Test
    void verifyModularity() {
        modules.verify();
    }
}
```

> الاختبار ده بيفشل الـ Build لو حصل Dependency غير مسموح أو Cycle.

---

## 7. الـ Naming Conventions

| العنصر | النمط | مثال |
|---|---|---|
| Controller | `{Feature}Controller` | `MemberController` |
| Service Interface | `{Feature}Service` | `SubscriptionService` |
| Service Impl | `{Feature}ServiceImpl` | `SubscriptionServiceImpl` |
| Query Contract | `{Feature}QueryService` | `CheckInQueryService` |
| Repository | `{Entity}Repository` | `PaymentRepository` |
| Entity | Singular | `Member`, `CheckIn` |
| Request DTO | `{Action}{Entity}Request` | `CreateMemberRequest` |
| Response DTO | `{Entity}Response` | `SubscriptionResponse` |
| Mapper | `{Entity}Mapper` | `PaymentMapper` |
| Exception | `{Problem}Exception` | `InvalidStateTransitionException` |
| DB Table | `snake_case` جمع | `members`, `subscription_events` |
| DB Column | `snake_case` | `price_minor_snapshot` |

---

## 8. الأماكن اللي **مش** هنعملها

| الحاجة | السبب |
|---|---|
| `common/` عملاق فيه Business Logic | يخالف الـ Feature-Based |
| `AbstractBaseService<T>` | تعقيد بلا قيمة |
| Generic Repository مخصص | Spring Data كافي |
| CQRS كامل | Over-engineering لمشروع مطور واحد |
| Microservices | تعقيد شبكة وTransactions |
| Kafka | Spring Events كافية |
| Redis في MVP | مش Needed |
| MapStruct لكل حاجة | Manual Mappers أوضح وأقل إعداداً؛ MapStruct اختياري |

---

## 9. إعدادات المشروع (Project Setup)

| العنصر | الاختيار |
|---|---|
| Build Tool | Maven (أو Gradle — اختيار واحد والالتزام بيه) |
| Java | 21 LTS |
| Spring Boot | 3.x (آخر Stable) |
| DB | PostgreSQL 16 |
| Migrations | Flyway |
| Testing | JUnit 5 + AssertJ + Mockito + Testcontainers |
| API Docs | SpringDoc OpenAPI |
| PDF | OpenPDF |
| Excel | Apache POI |
| Scheduling Lock | ShedLock JDBC |
| Mapping | Manual Mappers (MapStruct اختياري) |
| Container | Docker + Docker Compose |
| CI | GitHub Actions |

---

## 10. الـ Profiles

| Profile | الاستخدام | ملاحظات |
|---|---|---|
| `dev` | تطوير محلي | PostgreSQL Docker + Debug Logging |
| `test` | CI | Testcontainers PostgreSQL 16 — ممنوع H2 |
| `staging` | تجربة قبل الإنتاج | بيانات تجريبية |
| `prod` | الإنتاج | HTTPS + Logging Info + Monitoring |

---

## 11. الملفات الأساسية في الـ Repository

```text
kammel/
├── pom.xml
├── docker-compose.yml
├── Dockerfile
├── .github/workflows/ci.yml
├── src/main/java/com/kammel/...
├── src/main/resources/
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-prod.yml
│   ├── db/migration/
│   └── templates/whatsapp/
└── src/test/java/com/kammel/...
```

> **قاعدة:** مفيش `application-secret.yml` في الـ Git. كل الـ Secrets من Environment Variables.
