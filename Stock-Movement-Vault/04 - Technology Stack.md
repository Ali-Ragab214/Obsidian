---
title: Technology Stack
cssclasses: [rtl]
tags: [stack, dependencies, java, spring-boot]
section: 4
---

# 4. Technology Stack
⬅️ [[03 - Core Features]] | ➡️ [[05 - Architecture Overview]]

## 4.1 ✅ التقنيات المستخدمة فعلًا

| Technology | Version / source | الغرض في المشروع |
|---|---|---|
| **Java** | 21 (`pom.xml`) | Language و runtime |
| **Spring Boot** | 4.1.2 (parent) | Application framework و auto-configuration |
| **Spring Web MVC** | `spring-boot-starter-webmvc` | REST controllers |
| **Spring Data JPA** | `spring-boot-starter-data-jpa` | Repositories و persistence |
| **Hibernate** | Managed by Spring Boot | ORM |
| **Spring Security** | `spring-boot-starter-security` | Filter chain و password encoding وتفعيل method-security |
| **Bean Validation** | `spring-boot-starter-validation` | Request validation (مستخدمة جزئيًا) |
| **Flyway** | `spring-boot-starter-flyway`, `flyway-database-postgresql` | Common و per-tenant migrations |
| **PostgreSQL** | 17.5 (Docker image) + JDBC driver | الـ Primary database |
| **JJWT** | 0.12.6 (`jjwt-api`, `jjwt-impl`, `jjwt-jackson`) | إنشاء والتحقق من JWT |
| **Lombok** | Managed by Spring Boot | تقليل الـ boilerplate |
| **springdoc-openapi** | 3.2.0 (`springdoc-openapi-starter-webmvc-ui`) | OpenAPI / Swagger UI |
| **Spring Boot DevTools** | Runtime, optional | Local development |
| **Maven Wrapper** | `.mvn/wrapper` | Reproducible builds |
| **Docker Compose** | `docker-compose.yml` | Local PostgreSQL |
| **JUnit / Spring Boot Test starters** | `spring-boot-starter-*test` | Testing (test واحد بس) |

## 4.2 🚫 تقنيات **مش موجودة عمدًا**

> [!warning] دي موجودة في رؤية المنتج لكنها **مش** في الـ codebase. ماينفعش تتوصف كأنها implemented:
> - **Redis** أو **Caffeine** أو أي cache provider.
> - **Spring AI** أو **Ollama** أو **OpenAI** أو **LangChain4j** أو أي model client.
> - **Vector databases** (pgvector, Qdrant, Pinecone, ...).
> - **Kafka** أو **RabbitMQ** أو أي message broker.
> - **Spring Batch** أو **Quartz** أو أي scheduling framework.
> - **Actuator** أو metrics backend أو **OpenTelemetry** أو distributed tracing.
> - **Testcontainers** أو **RestAssured** أو أدوات mutation/coverage.
> - **MapStruct** (الـ mappers مكتوبة يدويًا).
> - **Liquibase** (المستخدم Flyway).
> - **Kubernetes manifests** أو **Helm charts** أو **Terraform** أو CI pipeline.
> - Billing/subscription provider (**Stripe**, **Paddle**, ...).

## 4.3 🔍 ملاحظات على الـ Dependencies

> [!caution] Spring Boot 4.1.2 جديد جدًا
> اتأكد إن **springdoc 3.2.0** و **JJWT 0.12.6** و الـ test starters متوافقين تمامًا قبل ما تعتمد عليهم في الـ production. محاولة الـ compile حلّت الـ dependencies بنجاح، لكن **توافق الـ OpenAPI UI وقت الـ runtime ماتأكدش منه**.

- **JJWT** مقسّم صح: API و implementation و Jackson modules.
- الـ test dependencies (`spring-boot-starter-data-jpa-test`, `...validation-test`, `...-webmvc-test`) هي **modular test starters** بتاعة Spring Boot 4. **مفيهاش Testcontainers**.
- مفيش cache أو messaging أو AI dependencies. أي اقتراح محتاجهم **بيضيف operational surface جديد**.
- `spring-boot-devtools` هي runtime dependency. اتأكد إنها **بتتستبعد من production artifacts** (الـ Spring Boot Maven plugin عادةً بيستبعدها وقت بناء executable jar، بس راجع).

---
⬅️ [[03 - Core Features]] | ➡️ [[05 - Architecture Overview]]
