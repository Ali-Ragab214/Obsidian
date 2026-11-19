---
title: Testing
cssclasses: [rtl]
tags: [testing, testcontainers, ci, quality]
section: 14
---

# 14. Testing
⬅️ [[13 - Performance and Scalability]] | ➡️ [[15 - Deployment Instructions]]

## 14.1 الوضع الحالي

- **test واحد بس:** `ApplicationTests.contextLoads()`.
- بيستخدم `@SpringBootTest`، يعني محتاج **application context كامل + database + JWT keys صالحة + migrations شغّالة**.
- `./mvnw test` **مش هيشتغل** لأن الـ main sources مابتعملش compile.
- مفيش **test profile** ولا **Testcontainers** ولا fixtures ولا MockMvc tests ولا CI.
- **التغطية الحالية:** عمليًا **صفر** على الـ business. مفيش نسبة مئوية معلنة لأنها مش قابلة للقياس في build مكسور.

## 14.2 ليه ده مهم هنا؟

أخطر المناطق هي بالظبط اللي **سهل نغلط فيها وصعب نلاحظها يدويًا**:

- 🔒 **Tenant isolation.**
- 📉 **مخزون سالب تحت الـ concurrency.**
- 🔄 **Atomicity وـ deadlocks في الـ transfers.**
- 🔁 **Idempotent retries.**
- 🛂 **Authorization و privilege escalation.**
- 🏗️ **فشل الـ provisioning والـ rollback.**

دي محتاجة **automated tests**، مش فحص يدوي بـ curl.

## 14.3 الـ Test Pyramid الموصى بيه

| الطبقة | الأدوات | بتختبر إيه |
|---|---|---|
| **Unit** | JUnit 5, AssertJ, Mockito | Domain invariants: balance delta, insufficient stock, transfer validation, role rules, reorder formula, forecast math |
| **Slice** | `@WebMvcTest`, `@DataJpaTest` | Controller contracts, validation, error mapping, repository queries والـ indexes |
| **Integration** | `@SpringBootTest` + **Testcontainers PostgreSQL** | Flyway migrations, tenant provisioning, real SQL, transaction boundaries |
| **Security** | Spring Security Test, MockMvc | 401/403, role matrix, tenant isolation, JWT tampering/expiry, deleted/suspended users |
| **Concurrency** | JUnit + executor/Testcontainers | `OUT` متوازية, transfers, lock ordering, deadlock retries, idempotency |
| **Contract** | springdoc snapshot / OpenAPI diff | استقرار API schema |
| **Architecture** | **ArchUnit** | مفيش controller → repository access؛ الـ domain مايعتمدش على web؛ وجود tenant checks |
| **End-to-end** | RestAssured أو Playwright (اختياري) | Flows حرجة: login → create product → receive → issue → transfer → report |

## 14.4 ✅ Test Cases الأساسية

### 🔒 Tenant isolation
1. Tenant A يعمل category؛ Tenant B يعمل list للـ categories **وماشوفهاش**.
2. Tenant B يطلب category/product/movement بتاعة A بالـ id ويستلم **404/403**، **مش بيانات**.
3. Tenant B يحط id بتاع A في filter أو AI tool argument ومايجيش **أي بيانات**.
4. غياب الـ tenant context **بيرفض الـ request**.
5. Tenant **suspended** بيترفض حتى لو معاه token صالح.

### 📦 Stock correctness
1. `IN` بيزوّد الـ balance **بالظبط** بالكمية.
2. `OUT` بينقّص ويرفض لو المخزون **مش كفاية**.
3. الـ transfer بينقل المخزون **atomically**؛ الفشل بيسيب المخزنين **من غير تغيير**.
4. الـ reversal بيرجّع الـ balance **ومابيعدّلش الحركة الأصلية**.
5. **50 طلب `OUT` متزامن** بـ 10 وحدات → **10 نجاحات بالظبط** و balance **مش سالب**.
6. تكرار الـ idempotency key **مابيعملش double-post**.

### 🛂 Security
1. من غير token → **401**؛ role غلط → **403**.
2. مستخدم الـ tenant **مايقدرش** يعيّن `ROLE_PLATFORM_ADMIN`.
3. حقول الـ password **عمرها ماتظهر** في الـ JSON.
4. Token منتهي/متلاعَب فيه → **401**.
5. مستخدم محذوف/معطّل **مايقدرش** يتصدّق أو يستخدم token موجود.
6. Payloads الـ **schema injection** بتترفض أثناء التسجيل.

## 14.5 الـ CI Pipeline المقترح

1. `./mvnw -q verify` (compile + unit + slice + integration بـ Testcontainers).
2. **Static analysis** (SpotBugs / Error Prone / Checkstyle أو ما يعادلها) و **ArchUnit**.
3. **Dependency و container vulnerability scanning**؛ ولّد **SBOM**.
4. **Flyway migration validation** على PostgreSQL container نظيف.
5. **OpenAPI diff / contract check**.
6. Build وـ sign للـ container image.
7. اختياري: deploy لبيئة **staging** وشغّل smoke tests.

## 14.6 سياسة الـ Coverage

حدّد target **بعد أول vertical slice شغّال**، مثلًا:

- **Domain/stock logic:** تغطية عالية مع invariant و concurrency tests.
- **Security-critical paths:** **كل branch** مختبر.
- **Controllers:** contract و error-path tests.
- **Overall line coverage:** **TBD** — ماتحسّنش على رقم قبل ما الـ critical tests تتوفر.

---
⬅️ [[13 - Performance and Scalability]] | ➡️ [[15 - Deployment Instructions]]
