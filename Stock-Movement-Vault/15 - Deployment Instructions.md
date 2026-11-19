---
title: Deployment Instructions
cssclasses: [rtl]
tags: [deployment, docker, setup, operations]
section: 15
---

# 15. Deployment Instructions
⬅️ [[14 - Testing]] | ➡️ [[16 - Project Roadmap]]

## 15.1 المتطلبات

- **Java 21**.
- **Docker و Docker Compose** (لـ PostgreSQL).
- **OpenSSL** (لتوليد JWT keys).
- Shell شبيه بـ Unix؛ مستخدمين Windows يقدروا يستخدموا **WSL** أو Maven wrapper.

## 15.2 Local Setup

```bash
# 1. Start PostgreSQL
docker compose up -d postgres

# 2. Generate a JWT key pair (the committed files are empty)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 \
  -out src/main/resources/certs/private_key.pem
openssl rsa -pubout \
  -in src/main/resources/certs/private_key.pem \
  -out src/main/resources/certs/public_key.pem

# 3. Build
./mvnw -DskipTests compile

# 4. Run
./mvnw spring-boot:run
```

الـ database بتتعرّض على الـ port المحدد في `.env` (`DB_PORT=5434` افتراضيًا). التطبيق بيقرا نفس الـ variable.

> [!warning] **ماتعملش commit للمفاتيح المولَّدة**
> الأوامر دي بتكتب في `src/main/resources/certs/` **للتطوير المحلي بس**. `JwtTokenService` بيحمّل المفاتيح حاليًا من الـ **classpath**؛ في الـ production لازم تعمل **mount للأسرار**، ولو لزم تعدّل الـ loader عشان يقرا من filesystem/secret path خارجي. تعديل الـ loader ده **TBD**.

## 15.3 الـ Configuration

| Variable | Default | الغرض |
|---|---|---|
| `DB_HOST` | `localhost` | PostgreSQL host |
| `DB_PORT` | `5432` في `application.yml`، و`5434` في `.env` | PostgreSQL port |
| `DB_NAME` | `saas-app-db` | اسم الـ database |
| `DB_USER` | `postgres` | Database user |
| `DB_PASSWORD` | `postgres` | Database password |
| `SERVER_PORT` | `8080` | HTTP port |
| `SPRING_PROFILES_ACTIVE` | `dev` | الـ Active profile |
| `JWT_PRIVATE_KEY_PATH` | `certs/private_key.pem` | مكان مفتاح التوقيع (classpath) |
| `JWT_PUBLIC_KEY_PATH` | `certs/public_key.pem` | مكان مفتاح التحقق (classpath) |
| `JWT_ACCESS_TOKEN_EXPIRATION` | `86400000` | عمر الـ access token بالـ milliseconds |

> [!note] مفيش `.env.example` لسه؛ ضيف واحد بـ placeholders.

## 15.4 🚧 الـ Blockers المعروفة (لازم تتصلح قبل ما التطبيق يشتغل)

| Blocker | التأثير | الإصلاح |
|---|---|---|
| `ProvisioningServiceImpl.java:47` syntax error وconstructor ناقص | **فشل الـ compile** | صلّح الـ statement والـ constructor |
| **PEM files فاضية** في `JwtTokenService` | فشل الـ startup | ولّد المفاتيح؛ حمّلها من secrets في الـ production |
| **Common migration فاضية** | جداول `tenants` / `users` مفقودة | اكتب الـ common `V1` migration |
| **مفيش tenant schema routing** | استعلامات الـ business بتروح `public` | نفّذ الـ resolver/provider ([[06 - Multi-Tenancy Strategy]]) |
| **مفيش `AuditorAware`** | الـ inserts بـ `created_by NOT NULL` بتفشل | ضيف auditor provider |
| **مفيش login endpoint** | مفيش token يتاخد | ضيف auth APIs |

## 15.5 جاهزية Docker والـ Deployment

### الوضع الحالي
- `docker-compose.yml` بيشغّل **PostgreSQL 17.5** بس.
- مفيش `Dockerfile` ومفيش application service في Compose.
- مفيش production profile ولا health check ولا resource limit ولا secrets management.
- مفيش **CI/CD pipeline**.

### 💡 الـ Production Topology الموصى بيها
1. ابنِ **OCI image** بـ Spring Boot Maven plugin (`./mvnw spring-boot:build-image`) أو **multi-stage Dockerfile**.
2. شغّل التطبيق كـ **non-root user** بـ read-only filesystem لو ممكن.
3. احقن أسرار الـ database والـ JWT من **secret manager**.
4. حطّه ورا **load balancer بينهي الـ TLS**.
5. ضيف **health/readiness probes** مدعومة بـ Actuator.
6. شغّل **Flyway** كخطوة deployment صريحة أو عند الـ startup مع lock.
7. استخدم **managed PostgreSQL** بـ backups و **point-in-time recovery**.
8. اظبط **JVM container memory limits** وفعّل heap sizing الواعي بالـ container.
9. عطّل **DevTools** و `show-sql` والـ verbose SQL formatting في الـ production.

## 15.6 ✅ Production Checklist

- [ ] الـ Build والـ tests بتنجح في CI.
- [ ] الـ Common والـ tenant migrations كاملة ومختبرة على database نظيفة.
- [ ] **Tenant isolation tests** بتنجح لكل business endpoint.
- [ ] الـ Authentication و refresh و logout و revocation شغّالين.
- [ ] **Authorization matrix** متفرضة ومختبرة.
- [ ] الأسرار **مش** في VCS أو الـ images.
- [ ] TLS و **CORS allowlist** و security headers و rate limits متظبطين.
- [ ] Actuator health/metrics مكشوفين للشبكات المصرّح لها بس.
- [ ] الـ Backups وإجراءات الـ restore **مختبرة**.
- [ ] الـ Logging **structured** وماحتواش أسرار أو PII زيادة.
- [ ] الـ Load tests بتغطي **posting concurrency** واستعلامات التقارير.
- [ ] فيه **runbooks** لفشل الـ provisioning و deadlocks وحوادث الـ database.
- [ ] الـ License وإشعارات الأطراف الثالثة محددة.

## 15.7 العمليات (Operations)

- **Migrations:** شغّلها بـ lock؛ **ماتعدّلش migrations اتطبّقت**؛ اعمل backup قبل أي تغيير هدّام.
- **Provisioning:** خلّيه **idempotent وقابل للمراقبة والإعادة**؛ اعرض الـ status للـ platform admins.
- **Backups:** restore لكل tenant لو schema-per-tenant؛ غير كده أدوات row-level export.
- **Incident response:** correlation IDs و audit logs ومسار تصعيد موثّق.
- **Key rotation:** ادعم **مفاتيح عامة متداخلة** أثناء تدوير مفتاح توقيع الـ JWT.
- **Data deletion:** نفّذ وسجّل عملية **offboarding للـ tenant**.

---
⬅️ [[14 - Testing]] | ➡️ [[16 - Project Roadmap]]
