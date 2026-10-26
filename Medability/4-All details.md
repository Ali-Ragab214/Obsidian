# منصة توافر الأدوية — Blueprint v2

**Feature-Based Architecture · Spring Boot 4**

لـ علي رجب · 24 أكتوبر 2026 · v2 = الفكرة والـ schema بتوع v1 بعد إعادة تصميمهم على Feature-Based وعلى طريقة شغلك: Controller → Service → Repository → DB + DTOs + Mappers + Unified Response.

## 0. إزاي تستخدم الملف ده

- ده **مرجع واحد كامل**: سلّمه لأي AI مع الـ Rules Block (قسم 19) وهو يقدر يولّد أي feature بنفس الستايل.
- كل قسم **مستقل** ومتعلّم برقم، فتقدر تقول للـ AI: «نفّذ 8.4 حسب قواعد 5 و6».
- **الجداول والـ Business Rules (BR) = مصدر الحقيقة.** الكود اللي جوه الملف = **قوالب** الـ AI يكمّلها، مش كود نهائي.
- كل قرار اتغيّر عن v1 عليه علامة **[معدّل]**، والجديد **[جديد]** (قسم 2).
- اللي لسه محتاج قرارك في قسم 18 ومعاه اقتراحي.
- تحليل السوق والمنافسين والمخاطر القانونية في v1 لسه سارية ومتكررتش هنا.

## 1. الفكرة في دقيقة

منصة بتخلّي المريض يلاقي الدواء الناقص (أو بديله العلمي المطابق) في أقرب صيدلية عندها مخزونه، في ثواني بدل ما يلف على 10–15 صيدلية. بتربط المرضى بالصيدليات (B2C) والصيدليات ببعضها عبر الـ requests (B2B).

- **نوع المشروع:** Portfolio مؤثر (driven-impact) مش startup — السوق زحمة (شفاء وغيرها).
- **القلب الحقيقي:** جودة بيانات التوافر، مش Elasticsearch ولا AI.
- **الـ MVP:** كتالوج + بدائل حتمية بـ SQL · بحث عربي/إنجليزي بيتحمّل الأخطاء الإملائية · بحث جغرافي · توثيق الصيدليات · تحديث المخزون (CSV + يدوي) مع TTL · Availability Requests · بلاغات المستخدمين · JWT + أدوار.
- **بعد الـ MVP (اختياري):** Elasticsearch بعد benchmark · OCR/AI للروشتة · تكامل POS.
- **خارج النطاق:** التوصيل والدفع · أي نصيحة أو قرار طبي · الأدوية المخدرة والمضبوطة.
- **الأدوار:** `PATIENT` · `PHARMACY_STAFF` · `ADMIN`.

## 2. سجل القرارات (Decision Log)

|   |   |   |   |
|---|---|---|---|
|#|القرار|السبب|الحالة|
|D1|**Package-by-Feature** جوه Modular Monolith (مش Layered على مستوى المشروع)|كل feature = vertical slice بتملك طبقاتها وجداولها؛ أسهل في الفهم والاختبار، وممكن تتفصل لـ service بعدين|[جديد]|
|D2|Spring Modulith لفحص الحدود + الـ events بين الـ features|يكسر الـ build لو حصل cycle أو تسريب اعتماديات|[v1 + توسعة]|
|D3|**Spring Boot 4.1.x + Java 21**|Boot 3.5 خلّص الـ OSS support في 30 يونيو 2026، فالمشروع الجديد يبدأ على 4.x|[معدّل] (v1 كان Boot 3)|
|D4|**Derived queries هي الأساس**؛ JPQL (`@Query`) للمتوسط؛ native SQL بس لـ PostGIS / pg_trgm / upsert وفي repository معزول ومتوثّق|derived queries مبتفهمش geography ولا trigram|[جديد]|
|D5|DTOs = Java **records** · Mapper = **MapStruct** · ممنوع نعرض Entity في الـ API|فصل الـ API عن الـ DB + أمان|[جديد]|
|D6|**Unified Response** `ApiResponse<T>` + `GlobalExceptionHandler` + `ErrorCode` لكل feature|شكل واحد للنجاح والخطأ|[جديد]|
|D7|`search` اتدمجت جوه `catalog`|بتقرا جداول الكتالوج؛ فصلها بيكسر الـ ownership. بديل Elasticsearch بيتبدّل بـ property|[معدّل]|
|D8|موديول `availability` في v1 اتقسم لتلاتة: `pharmacy` + `inventory` + `availability`|كل واحدة ليها ownership واضح؛ availability = طبقة تجميع (orchestration)|[معدّل]|
|D9|عمليات الصيدلية على `/pharmacies/me/**` بدل `/{id}`|الصيدلية بتتحدد من الـ JWT → مفيش IDOR|[معدّل]|
|D10|حالة التوافر الفعلية بتتحسب **وقت القراءة** من `expires_at` (مفيش job بيحدّث الصفوف)|أبسط وأدق|[جديد]|
|D11|Cache للكتالوج والبدائل بس — **ممنوع** للتوافر|الثقة أهم من السرعة|[v1]|
|D12|البدائل = SQL حتمي بـ `formulation_key`؛ الـ AI اختياري وآخر مرحلة|الـ LLM بيهلوس والغلط في الدواء مكلف|[v1]|
|D13|IDs = `BIGINT identity` · الأوقات `Instant` / `timestamptz` (UTC) · الـ API تحت `/api/v1`|بساطة + مفيش لخبطة timezone (مصر فيها DST)|[جديد]|
|D14|Flyway يملك الـ schema: `ddl-auto=validate` و`open-in-view=false`|منع تغييرات صامتة في الـ DB|[جديد]|

## 3. الـ Tech Stack والإصدارات

|   |   |   |
|---|---|---|
|الطبقة|الأداة|ملاحظة|
|Language|Java 21 LTS|25 LTS مدعومة لو حبيت|
|Framework|Spring Boot 4.1.x|Spring Framework 7 · Spring Security 7 · Hibernate 7|
|Build|Maven|أو Gradle — اختار واحد وخلّيك عليه|
|Modularity|Spring Modulith 2.x|verify + events (راجع توافقه مع إصدار Boot وقت البدء)|
|Web|Spring Web MVC + Bean Validation||
|Persistence|Spring Data JPA + `hibernate-spatial` (JTS)||
|Database|PostgreSQL 16+ مع PostGIS · `pg_trgm` · `unaccent`||
|Migrations|Flyway||
|Mapping|MapStruct|مع Lombok لازم `lombok-mapstruct-binding`|
|Boilerplate|Lombok (محدود)|قسم 5.5|
|Security|Spring Security + JWT|access 15 دقيقة + refresh 7 أيام|
|Batch|Spring Batch|استيراد المخزون والكتالوج|
|Cache / Rate limit|Redis + Bucket4j||
|API Docs|springdoc-openapi|الإصدار المتوافق مع Boot 4|
|Observability|Actuator + Micrometer|Prometheus اختياري|
|Testing|JUnit 5 · Mockito · AssertJ · Testcontainers (PostGIS + Redis)||
|Infra|Docker Compose · GitHub Actions||

**ملاحظات Boot 4 (مهمة للـ AI لأن أغلب الأمثلة القديمة على Boot 3):**

- ولّد الـ `pom.xml` من start.spring.io. أسماء الـ starters اتقسّمت/اتغيّرت في Boot 4، ماتكتبهاش من الذاكرة.
- **Jackson 3:** الـ packages بقت `tools.jackson.*` بدل `com.fasterxml.jackson.*` (الـ annotations زي `@JsonInclude` لسه في `com.fasterxml.jackson.annotation`).
- في الـ tests استخدم `@MockitoBean` بدل `@MockBean`.
- **Spring Batch 6:** راجع الـ API الحالي لتشغيل الـ jobs قبل ما تعتمد على أمثلة قديمة.
- أي حل من StackOverflow أو مقال قديم يتراجع على توافق Boot 4 قبل الاستخدام.

## 4. الـ Architecture: Feature-Based

### 4.1 المبدأ

في الـ Layered القديم كان عندك `controller/` و`service/` و`repository/` على مستوى المشروع كله. في **Feature-Based** بنقلب الترتيب: كل feature (مثلاً `catalog`) بتحتوي **كل** طبقاتها جواها. الطبقات لسه موجودة والاتجاه ثابت (Controller → Service → Repository → DB) بس بقت **جوه الـ feature**.

- **التغيير في feature واحدة** بيحصل في package واحد، مش متفرّق على 6 أماكن.
- **الحدود واضحة:** كل feature بتملك جداولها، وغيرها ميقدرش يوصل لها غير من الباب الرسمي (الـ service interface).
- **الاختبار والفصل:** تقدر تختبر feature لوحدها، ولو كبر المشروع تقدر تفصلها لـ service مستقل.

### 4.2 شجرة المشروع الكاملة

```
pharmacy-availability/
├── pom.xml
├── docker-compose.yml
├── .github/workflows/ci.yml
└── src/
    ├── main/
    │   ├── java/com/aliragab/pharmacyfinder/     # الـ base package (مثال، غيّره)
    │   │   ├── PharmacyFinderApplication.java
    │   │   ├── common/                           # Shared Kernel — مالوش اعتماد على أي feature
    │   │   │   ├── api/         # ApiResponse, PageResponse, FieldErrorDto, ApiPaths
    │   │   │   ├── exception/   # AppException, ErrorCode, CommonErrorCode, GlobalExceptionHandler
    │   │   │   ├── config/      # JpaAuditingConfig, ClockConfig, AsyncConfig, CacheConfig, OpenApiConfig
    │   │   │   ├── persistence/ # AuditableEntity
    │   │   │   ├── security/    # Role, AuthenticatedUser, SecurityUtils
    │   │   │   ├── geo/         # Coordinates, GeoFactory
    │   │   │   ├── text/        # ArabicTextNormalizer
    │   │   │   └── web/         # RequestIdFilter, RateLimitFilter
    │   │   ├── identity/        # المستخدمين + JWT
    │   │   ├── catalog/         # الأدوية + البدائل + البحث + استيراد الكتالوج
    │   │   ├── pharmacy/        # الصيدلية + التوثيق + البحث الجغرافي
    │   │   ├── inventory/       # المخزون + استيراد CSV
    │   │   ├── availability/    # بحث التوافر + الطلبات + البلاغات
    │   │   ├── notification/    # إشعار الصيدليات
    │   │   └── ai/              # اختياري — آخر مرحلة
    │   └── resources/
    │       ├── application.yml  (+ application-dev / -test / -prod)
    │       └── db/migration/    # V1__...sql, V2__...sql
    └── test/java/...            # نفس هيكل الـ packages + integration/
```

### 4.3 تشريح feature واحدة

```
catalog/
├── package-info.java      # @ApplicationModule
├── controller/            # DrugController, DrugSearchController, CatalogAdminController
├── service/               # DrugService, DrugSearchService  ← interfaces = الـ Public API
│   └── impl/              # DrugServiceImpl ...            ← internal
├── repository/            # DrugRepository (derived) + DrugSearchRepository (native، معزول)
├── entity/                # Drug, ActiveIngredient, Manufacturer, DrugIngredient + enums
├── dto/
│   ├── request/           # CreateDrugRequest, DrugSearchRequest
│   └── response/          # DrugResponse, DrugSummaryResponse
├── mapper/                # DrugMapper (MapStruct)
├── exception/             # DrugNotFoundException, CatalogErrorCode
├── event/                 # events اللي الـ feature بتنشرها (لو فيه)
└── config/                # @ConfigurationProperties أو beans خاصة بالـ feature (لو فيه)
```

لو الـ feature صغيرة (زي `notification`) قلّل الـ sub-packages، بس **الاتجاه ثابت:** controller → service → repository.

### 4.4 الـ Features: بتملك إيه وبتكشف إيه

|   |   |   |   |
|---|---|---|---|
|Feature|بتملك (Tables)|Public API (اللي غيرها ينادي عليه)|بتعتمد على|
|`identity`|`app_user`, `refresh_token`|لا شيء تقريبًا (الباقي بيعتمد على `AuthenticatedUser` من common)|common|
|`catalog`|`active_ingredient`, `manufacturer`, `drug`, `drug_ingredient`, `catalog_import_job`|`DrugService`: getSummary, getSummaries, findIdsByCodes, findAlternatives · `DrugSearchService`|common|
|`pharmacy`|`pharmacy`, `pharmacy_staff`|`PharmacyService`: getSummaries, findVerifiedNearby, getPharmacyIdByStaffUser, getContact|common|
|`inventory`|`pharmacy_stock`, `stock_import_job`, `stock_import_error`|`StockService`: findStocks(pharmacyIds, drugIds), upsert(command)|common, catalog, pharmacy|
|`availability`|`availability_request`, `availability_request_target`, `availability_response`, `availability_report`|events: `AvailabilityRequestCreatedEvent`, `AvailabilityRequestRespondedEvent`|common, catalog, pharmacy, inventory|
|`notification`|`notification`|لا شيء (بتسمع events بس)|common, availability (events)، pharmacy|
|`ai`|—|`PrescriptionOcrService`|common, catalog|

### 4.5 قواعد الاعتماد (Dependency Rules)

```
 identity ─┐
 catalog ──┼──► common
 pharmacy ─┘
 inventory    ──► catalog, pharmacy
 availability ──► catalog, pharmacy, inventory
 notification ──► availability (events بس), pharmacy
 ai           ──► catalog
```

|   |   |
|---|---|
|#|القاعدة|
|R1|feature تنادي feature تانية **بس** عن طريق الـ `service` interface (مش الـ impl)، وبترجع DTOs/Views (records) — **مش Entities**.|
|R2|ممنوع import لـ entity / repository / mapper / controller بتاعة feature تانية.|
|R3|الربط بين الـ features **بالـ ID بس** (`Long drugId`). مفيش `@ManyToOne` عابر للـ features. الـ FK على مستوى الـ DB مسموح.|
|R4|ممنوع الـ cycles. لو A محتاجة B وB محتاجة A → استخدم Event.|
|R5|التواصل fire-and-forget بالـ Events: records في `event/` جوه الـ feature الناشرة، أسماؤها بصيغة الماضي (`StockUpdatedEvent`).|
|R6|`common` مفيهوش business logic ومالوش علاقة بأي feature.|
|R7|الـ `@Transactional` في الـ service بس. المناداة عبر features جوه نفس الـ transaction مسموحة، لكن الأحداث الحساسة (إشعارات) تتبعت **after-commit**.|
|R8|استثناء واحد معلن: Repository native جوه feature يقدر يقرا جدول feature تانية **read-only** لو الأداء اتثبت بالقياس إنه محتاج join واحد. يتوثّق بتعليق `// CROSS-FEATURE READ` وسبب.|

**الـ Enforcement (مش بالنية):**

- Test بـ `ApplicationModules.of(PharmacyFinderApplication.class).verify()` يكسر الـ build لو فيه cycle أو وصول لـ internal.
- كل feature فيها `package-info.java` بـ `@ApplicationModule`، والـ packages المسموح تتكشف (`service`, `dto`, `event`) تتعلّم بـ `@NamedInterface` — اطلب من الـ AI يكتبها حسب docs إصدار Modulith المستخدم.
- (اختياري) قواعد ArchUnit: الـ controller مايعتمدش على repository · الـ entity مايعتمدش على dto · الـ `impl` مايتناداش من برّه الـ feature.

### 4.6 رحلة الـ request

```
HTTP Request
  → Filters:  RequestId → RateLimit → JWT Authentication
  → Controller:  @Valid على الـ DTO، يحوّل path/query، ينادي service، يلفّ النتيجة في ApiResponse
  → Service:  business rules + @Transactional + مناداة services تانية + Mapper
  → Repository:  derived query / JPQL / native (استثناء)
  → PostgreSQL (+ PostGIS)
  ← Entity ← Mapper → Response DTO ← ApiResponse<T> ← ResponseEntity

أي Exception في أي مرحلة → GlobalExceptionHandler → ApiResponse(success=false, code, message, errors)
```

## 5. قواعد الطبقات والـ Conventions

### 5.1 خريطة الـ Stereotype Annotations

|   |   |   |
|---|---|---|
|Annotation|بتتحط على|مكانها|
|`@RestController` + `@RequestMapping`|الـ Controllers|`<feature>/controller`|
|`@Service`|الـ service impl|`<feature>/service/impl`|
|`@Repository`|الـ repositories المخصصة (native/custom) بس — الـ Spring Data interfaces مش محتاجاها|`<feature>/repository`|
|`@Component`|أي bean مساعد (Normalizer, Generator, Listener...)|حسب الـ feature|
|`@Configuration` + `@ConfigurationProperties`|الإعدادات|`<feature>/config` أو `common/config`|
|`@RestControllerAdvice`|GlobalExceptionHandler|`common/exception`|
|`@Entity` / `@MappedSuperclass`|الـ entities|`<feature>/entity`|
|`@Mapper` (MapStruct)|الـ mappers|`<feature>/mapper`|
|`@ApplicationModuleListener`|listeners للـ events بين الـ features|`<feature>/service/listener`|

### 5.2 الـ Controller

- **رفيع جدًا:** لا business logic، لا `@Transactional`، لا اتصال بـ repository.
- بيرجع دايمًا `ResponseEntity<ApiResponse<T>>`. مفيش try/catch — الأخطاء بتروح للـ GlobalExceptionHandler.
- `@Valid` على الـ request body، و`@Validated` على الـ class لو فيه constraints على `@RequestParam`.
- الـ paths من `ApiPaths` (constants)، والـ HTTP status: 200 للقراءة والتعديل · 201 للإنشاء · 202 للـ async import. مش بنستخدم 204 عشان الـ envelope الموحد (بنرجع 200 مع `data` فاضية).
- الصلاحيات بـ `@PreAuthorize` على الـ method، والـ docs بـ `@Tag` و`@Operation`.
- المستخدم الحالي بييجي من `@AuthenticationPrincipal AuthenticatedUser`.

### 5.3 الـ Service

- **Interface** (العقد العام) + **`ServiceImpl`** في `service/impl` بـ `@Service` و`@RequiredArgsConstructor`، والـ impl يكون package-private.
- `@Transactional(readOnly = true)` على مستوى الـ class، و`@Transactional` على methods الكتابة.
- بياخد Request DTO أو primitives، ويرجّع Response DTO/View. **الـ Entity مبتخرجش من الـ feature أبدًا.**
- هنا بتتحط: الـ Business Rules، فحص الـ ownership، التنسيق مع services تانية، نشر الـ events.
- بيرمي exceptions من `<feature>/exception` مش `RuntimeException` عام.
- الوقت من `Clock` bean (مش `Instant.now()` مباشرة) عشان الـ tests تتحكم في الزمن.

### 5.4 الـ Repository — Derived Queries أولًا

سلّم الأولويات:

1. **Derived query** هو الافتراضي: `findByCode`, `existsByLicenseNo`, `findByIdInAndActiveTrue`...
2. لو الاسم طال (أكتر من 3 conditions) أو فيه join → `@Query` JPQL مع projection.
3. **Native SQL بس في:** PostGIS (`ST_DWithin`, `ST_Distance`) · pg_trgm (`similarity`, `%`) · upsert (`ON CONFLICT`) · bulk updates. ويتحط في repository منفصل بإسم واضح (`DrugSearchRepository`, `PharmacyGeoRepository`, `StockUpsertRepository`) وعليه Javadoc بسبب الاستثناء.

قواعد إضافية: النتائج المخصصة بترجع **Projection** (record/interface) مش `Object[]` · ممنوع `findAll()` من غير pagination على الجداول الكبيرة (`drug`, `pharmacy_stock`) · الـ fetch الافتراضي LAZY وبنعالج الـ N+1 بـ `@EntityGraph` أو projections · كل native query ليه `@DataJpaTest` على Testcontainers PostGIS.

```java
Optional<Drug> findByCodeAndActiveTrue(String code);
boolean existsByLicenseNo(String licenseNo);
Page<Pharmacy> findByVerificationStatus(VerificationStatus status, Pageable pageable);
List<StockImportError> findByJobIdOrderByRowNumber(Long jobId);
List<Drug> findByFormulationKeyAndIdNotAndControlledFalseAndActiveTrue(String key, Long id);
```

### 5.5 الـ Entity (و Lombok)

- `@Getter @Setter @NoArgsConstructor(access = PROTECTED)`. **ممنوع `@Data`** و`@EqualsAndHashCode` الافتراضي على entities. `equals/hashCode` على الـ id أو بلاش.
- `@Enumerated(EnumType.STRING)` دايمًا · الأوقات `Instant` · الـ audit (`createdAt`, `updatedAt`) من `AuditableEntity` بـ `@EnableJpaAuditing`.
- العلاقات جوه الـ feature بس (`@ManyToOne(fetch = LAZY)`)، وبين الـ features بالـ ID (R3).
- حقل boolean زي `is_controlled` بيتسمّى `controlled` في الـ entity عشان derived queries (`...ControlledFalse...`).
- Geography: `org.locationtech.jts.geom.Point` مع `@JdbcTypeCode(SqlTypes.GEOGRAPHY)`، وافتكر **x = longitude و y = latitude**. لو الـ mapping عمل مشاكل → fallback: native INSERT/UPDATE بـ `ST_MakePoint(lng, lat)`.
- Lombok مسموح: `@Getter/@Setter/@RequiredArgsConstructor/@Slf4j`. مش مسموح على الـ DTOs (records).

### 5.6 الـ DTOs

- كلها `record`. الأسماء: `<Action><Entity>Request` · `<Entity>Response` · `<Entity>SummaryResponse`.
- Validation على الـ request records: `@NotBlank` `@Size` `@Pattern` `@Positive` `@DecimalMin/@DecimalMax`. الرسايل بالعربي من `messages.properties`.
- الـ Response فيها اللي الـ client محتاجه بس: مفيش `passwordHash`، ولا `licenseNo` للمريض.
- الـ Views اللي بتتبادل بين الـ features (`DrugSummaryResponse`) = records صغيرة في `dto/response` وهي جزء من الـ Public API.

### 5.7 الـ Mappers (MapStruct)

- كل mappers بتورّث config مشتركة (`CommonMapperConfig`) فيها `componentModel = spring` و`unmappedTargetPolicy = ERROR` → أي حقل منسي يكسر الـ compile.
- ثلاث أنواع methods: `toResponse(Entity)` · `toEntity(CreateRequest)` · `updateEntity(UpdateRequest, @MappingTarget Entity)`.
- الـ mapper مفيهوش business logic ولا repository calls. أي lookup الـ service يعمله ويبعت النتيجة.

### 5.8 قواعد التسمية

|                   |                                                           |                                   |
| ----------------- | --------------------------------------------------------- | --------------------------------- |
| العنصر            | القاعدة                                                   | مثال                              |
| Feature package   | lowercase مفرد                                            | `catalog`, `inventory`            |
| Controller        | `<X>Controller`                                           | `DrugSearchController`            |
| Service           | interface `<X>Service` + impl `<X>ServiceImpl`            | `DrugService`                     |
| Repository        | `<Entity>Repository`                                      | `DrugRepository`                  |
| DTO               | `Create<X>Request` · `<X>Response` · `<X>SummaryResponse` | `DrugResponse`                    |
| Mapper            | `<Entity>Mapper`                                          | `DrugMapper`                      |
| Exception         | `<X>NotFoundException`                                    | `DrugNotFoundException`           |
| ErrorCode enum    | `<Feature>ErrorCode`                                      | `CatalogErrorCode.DRUG_NOT_FOUND` |
| Event             | `<X><PastVerb>Event`                                      | `StockUpdatedEvent`               |
| Endpoint          | `/api/v1/<kebab-plural>`                                  | `/api/v1/import-jobs`             |
| JSON fields       | camelCase                                                 | `lastConfirmedAt`                 |
| DB tables/columns | snake_case مفرد                                           | `pharmacy_stock.expires_at`       |
| DB constraints    | `pk_` `fk_` `uq_` `chk_` `idx_` + table + cols            | `idx_pharmacy_location`           |
| Tests             | `<Class>Test` + methods `should_X_when_Y`                 |                                   |
| Config props      | `app.<feature>.<key>`                                     | `app.availability.ttl.csv`        |

## 6. الـ Unified API Response ومعالجة الأخطاء

### 6.1 الشكل

كل endpoint (نجاح أو فشل) بيرجع نفس الـ envelope. الحقول الـ null مش بتظهر (`NON_NULL`).

```json
{ "success": true, "code": "OK", "message": "تمت العملية بنجاح",
  "data": { "id": 12 }, "timestamp": "2026-10-24T10:15:30Z" }

{ "success": false, "code": "VALIDATION_ERROR", "message": "بيانات غير صالحة",
  "errors": [ { "field": "lat", "message": "خط العرض مطلوب" } ],
  "timestamp": "2026-10-24T10:15:30Z" }
```

`code` ثابت ومقروء للـ client (`DRUG_NOT_FOUND`)، و`message` للعرض (عربي)، و`errors` في الـ validation بس.

### 6.2 الكود (common/api)

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public record ApiResponse<T>(boolean success, String code, String message,
                             T data, List<FieldErrorDto> errors, Instant timestamp) {

    public static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>(true, "OK", "تمت العملية بنجاح", data, null, Instant.now());
    }
    public static <T> ApiResponse<T> ok(String message, T data) { /* ... */ }
    public static <T> ApiResponse<T> created(T data)            { /* code = CREATED */ }
    public static ApiResponse<Void> error(ErrorCode ec, String message) { /* success = false */ }
    public static ApiResponse<Void> validationError(List<FieldErrorDto> errors) { /* ... */ }
}

public record FieldErrorDto(String field, String message) {}

public record PageResponse<T>(List<T> items, int page, int size,
                              long totalElements, int totalPages, boolean hasNext) {
    public static <E, T> PageResponse<T> from(Page<E> page, Function<E, T> mapper) { /* ... */ }
}
```

الـ endpoints اللي فيها pagination بترجع `ApiResponse<PageResponse<T>>`. الحد الأقصى لـ `size` = 50 (`spring.data.web.pageable.max-page-size`).

### 6.3 ErrorCode والـ Exceptions

```java
// common/exception
public interface ErrorCode {
    String code();                 // DRUG_NOT_FOUND
    HttpStatus status();           // 404
    String defaultMessage();       // بالعربي
}

public enum CommonErrorCode implements ErrorCode {
    VALIDATION_ERROR, MALFORMED_REQUEST, UNAUTHORIZED, FORBIDDEN,
    NOT_FOUND, CONFLICT, FILE_TOO_LARGE, RATE_LIMITED, INTERNAL_ERROR
    // كل constant ليه (HttpStatus, defaultMessage)
}

public class AppException extends RuntimeException {
    private final ErrorCode errorCode;
    public AppException(ErrorCode errorCode, String detailForLog) { /* ... */ }
}

// catalog/exception  — كل feature عندها enum بتاعها (عشان الـ features متتشبكش ببعض)
public enum CatalogErrorCode implements ErrorCode { DRUG_NOT_FOUND, DUPLICATE_DRUG_CODE, CONTROLLED_DRUG_NOT_ALLOWED }

public class DrugNotFoundException extends AppException {
    public DrugNotFoundException(Long id) { super(CatalogErrorCode.DRUG_NOT_FOUND, "drugId=" + id); }
}
```

### 6.4 الـ GlobalExceptionHandler (`@RestControllerAdvice`)

|   |   |   |
|---|---|---|
|Exception|HTTP|code|
|`AppException`|`errorCode.status()`|`errorCode.code()`|
|`MethodArgumentNotValidException`|400|`VALIDATION_ERROR` (errors من الـ binding)|
|`ConstraintViolationException` / `HandlerMethodValidationException`|400|`VALIDATION_ERROR`|
|`HttpMessageNotReadableException`|400|`MALFORMED_REQUEST`|
|`MethodArgumentTypeMismatchException` / `MissingServletRequestParameterException`|400|`VALIDATION_ERROR`|
|`AccessDeniedException`|403|`FORBIDDEN`|
|`DataIntegrityViolationException` / `ObjectOptimisticLockingFailureException`|409|`CONFLICT` (من غير رسالة الـ DB)|
|`MaxUploadSizeExceededException`|413|`FILE_TOO_LARGE`|
|أي `Exception` تانية|500|`INTERNAL_ERROR` (رسالة عامة + log بـ requestId)|

- الـ 401/403 اللي بتحصل في الـ filter chain بتطلع بنفس الـ envelope عن طريق `AuthenticationEntryPoint` و`AccessDeniedHandler` (في `identity`).
- ممنوع تسريب stack trace أو رسالة SQL أو أسماء جداول للـ client.

## 7. Reference Slice: `catalog` / Drug (القالب اللي الـ AI يكرّره)

ده vertical slice كامل بحجم صغير. أي feature جديدة بتتبني **بنفس الشكل**. (الـ write slice بالـ validation والـ `@MappingTarget` متوضح في 8.3.)

**Entity**

```java
@Entity
@Table(name = "drug")
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Drug extends AuditableEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false, unique = true) private String code;
    @Column(name = "trade_name_en", nullable = false) private String tradeNameEn;
    @Column(name = "trade_name_ar") private String tradeNameAr;
    @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "manufacturer_id") private Manufacturer manufacturer;
    @Column(name = "dosage_form") private String dosageForm;
    @Column(name = "official_price") private BigDecimal officialPrice;
    @Column(name = "is_controlled", nullable = false) private boolean controlled;
    @Column(name = "formulation_key") private String formulationKey;
    @Column(nullable = false) private boolean active = true;
    // + name_en_norm, name_ar_norm, search_text (بتتحسب وقت الاستيراد بـ ArabicTextNormalizer)
}
```

**Repository (derived)**

```java
public interface DrugRepository extends JpaRepository<Drug, Long> {
    boolean existsByCode(String code);
    List<Drug> findByIdInAndActiveTrue(Collection<Long> ids);
    List<Drug> findByCodeIn(Collection<String> codes);
    List<Drug> findByFormulationKeyAndIdNotAndControlledFalseAndActiveTrue(String key, Long id);
}
```

**DTOs**

```java
public record DrugResponse(Long id, String code, String tradeNameEn, String tradeNameAr,
        String manufacturerName, String dosageForm, String packSize, BigDecimal officialPrice) {}

public record DrugSummaryResponse(Long id, String tradeNameEn, String tradeNameAr,
        String dosageForm, BigDecimal officialPrice) {}      // ← View بتتبادل مع الـ features التانية

public record AlternativesResponse(List<DrugSummaryResponse> alternatives, String disclaimer) {}
```

**Mapper**

```java
@Mapper(config = CommonMapperConfig.class)
public interface DrugMapper {
    @Mapping(target = "manufacturerName", source = "manufacturer.nameEn")
    DrugResponse toResponse(Drug drug);
    DrugSummaryResponse toSummary(Drug drug);
}
```

**Service — العقد العام + الـ impl**

```java
public interface DrugService {
    DrugResponse getById(Long id);
    List<DrugSummaryResponse> findAlternatives(Long drugId);
    List<DrugSummaryResponse> getSummaries(Collection<Long> ids);       // للـ features التانية
    Map<String, Long> findIdsByCodes(Collection<String> codes);         // للـ inventory (import)
}

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class DrugServiceImpl implements DrugService {
    private final DrugRepository drugRepository;
    private final DrugMapper drugMapper;

    @Override
    public DrugResponse getById(Long id) {
        return drugMapper.toResponse(getVisible(id));
    }

    @Override
    @Cacheable(cacheNames = CacheNames.DRUG_ALTERNATIVES, key = "#drugId")
    public List<DrugSummaryResponse> findAlternatives(Long drugId) {
        Drug drug = getVisible(drugId);
        return drugRepository
                .findByFormulationKeyAndIdNotAndControlledFalseAndActiveTrue(drug.getFormulationKey(), drugId)
                .stream().map(drugMapper::toSummary).toList();
    }

    private Drug getVisible(Long id) {          // BR-C3: المخدر/الموقوف = مش موجود للمستخدم
        return drugRepository.findById(id)
                .filter(d -> d.isActive() && !d.isControlled())
                .orElseThrow(() -> new DrugNotFoundException(id));
    }
    // getSummaries, findIdsByCodes ...
}
```

**Controller**

```java
@RestController
@RequestMapping(ApiPaths.DRUGS)          // /api/v1/drugs
@RequiredArgsConstructor
@Tag(name = "Catalog")
public class DrugController {
    private final DrugService drugService;

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<DrugResponse>> getById(@PathVariable Long id) {
        return ResponseEntity.ok(ApiResponse.ok(drugService.getById(id)));
    }

    @GetMapping("/{id}/alternatives")
    public ResponseEntity<ApiResponse<AlternativesResponse>> alternatives(@PathVariable Long id) {
        var body = new AlternativesResponse(drugService.findAlternatives(id), MedicalDisclaimer.TEXT);
        return ResponseEntity.ok(ApiResponse.ok(body));
    }
}
```

**الـ Tests بتاعة الـ slice:** `DrugServiceImplTest` (Mockito: `should_throw_not_found_when_drug_is_controlled`) · `DrugMapperTest` · `DrugRepositoryTest` (`@DataJpaTest` + Testcontainers) · `DrugControllerTest` (`@WebMvcTest` — بيتأكد من شكل الـ envelope).

`MedicalDisclaimer.TEXT` ثابت نصي في `common/api`: «أي تبديل للدواء يجب أن يكون بمراجعة الطبيب المعالج أو الصيدلي.» — بيتضاف لأي response فيه بدائل (BR-S1).

## 8. تفاصيل كل Feature

كل feature تحت بنفس الترتيب: المسؤولية ← الجداول ← الـ Public API ← الـ Business Rules ← التدفقات ← الـ Events. قاعدة الأرقام: `BR-<حرف>` = Business Rule.

### 8.0 قواعد السلامة العامة (BR-S) — بتتطبق على كل الـ features

|   |   |
|---|---|
|#|القاعدة|
|BR-S1|أي response فيه بدائل لازم يحتوي `disclaimer` (`MedicalDisclaimer.TEXT`).|
|BR-S2|الأدوية المخدرة/المضبوطة (`controlled = true`) مستبعدة من: البحث، التفاصيل، البدائل، التوافر، واستيراد المخزون.|
|BR-S3|مفيش نصيحة طبية: لا جرعات ولا «ينفع/مينفعش». المنصة بتعرض توافر وبدائل مسجلة رسميًا بس.|
|BR-S4|السعر من `drug.official_price` فقط، والصيدلية مبتعدّلوش.|
|BR-S5|موقع المريض وتليفونه بيانات حساسة: مبتتسجلش في الـ logs، ومبتظهرش للصيدلية (الطلب للصيدلية من غير هوية المريض)، وإحداثيات الـ requests بتتمسح بعد 30 يوم.|
|BR-S6|مفيش صيدلية بتظهر في أي نتيجة إلا لو `VERIFIED`.|
|BR-S7|أي حالة توافر تتعرض ومعاها وقت آخر تأكيد وحالة التأكيد (`ConfirmationState`). مفيش حالة قديمة تتعرض كأنها مؤكدة.|

### 8.1 `identity`

**المسؤولية:** التسجيل، الدخول، الـ JWT، الأدوار، وإنتاج `AuthenticatedUser` لباقي النظام. **الجداول:** `app_user`, `refresh_token`.

**الـ Services:** `AuthService` (register, login, refresh, logout) · `UserService` (getMe). Internal: `JwtService`, `JwtAuthenticationFilter`, `SecurityConfig`, `RestAuthenticationEntryPoint`, `RestAccessDeniedHandler`.

**DTOs:** `RegisterRequest(fullName, email, phone, password, role)` · `LoginRequest(identifier, password)` · `RefreshRequest(refreshToken)` · `AuthResponse(accessToken, refreshToken, expiresInSeconds, user)` · `UserResponse(id, fullName, email, phone, role)`.

|   |   |
|---|---|
|#|القاعدة|
|BR-U1|التسجيل الذاتي للدورين `PATIENT` و`PHARMACY_STAFF` بس. الـ `ADMIN` بيتعمل من bootstrap runner بيقرا `app.admin.email/password` من env (لو مفيش ADMIN).|
|BR-U2|لازم email أو phone (E.164 زي `+2010...`)، وكلمة السر 8 حروف على الأقل. التخزين بـ BCrypt.|
|BR-U3|الـ JWT claims: `sub=userId`, `role`, `iat`, `exp`. الـ `pharmacyId` **مش** في التوكن (بيتجاب من DB عند الحاجة عشان ميبقاش قديم).|
|BR-U4|الـ refresh token بيتخزن **hash** بس (SHA-256) وبيتعمله rotation: كل refresh بيلغي القديم. استخدام token ملغي = إلغاء كل tokens المستخدم.|
|BR-U5|endpoints الـ `/auth/**` عليها rate limit أشد من الباقي.|

### 8.2 `catalog`

**المسؤولية:** الأدوية، المواد الفعالة، الشركات، البدائل، البحث، واستيراد الكتالوج (ADMIN). **الجداول:** `active_ingredient`, `manufacturer`, `drug`, `drug_ingredient`, `catalog_import_job`. ملحوظة: `search` اتدمجت هنا (D7).

**الـ Public API:** `DrugService` (قسم 7) + `DrugSearchService.search(query, pageable)`.

|   |   |
|---|---|
|#|القاعدة|
|BR-C1|**`formulation_key`** = SHA-256 لـ: (المواد الفعالة مرتبة بالـ id، كل واحدة `ingredientId:strength:unit`) + `\|` + الشكل الدوائي المطبّع. الـ strength بيتحول لـ `mg` (g ← ×1000، mcg ← ÷1000) ويتشال الأصفار الزايدة (5.0 = 5). الشكل الدوائي بيتوحّد (tablet/tab/أقراص = `TABLET`).|
|BR-C2|نفس المادة بتركيز أو شكل دوائي مختلف **مش** بديل (أقراص ≠ شراب). البديل = نفس الـ `formulation_key`.|
|BR-C3|البدائل بتستبعد `controlled` و`active = false` ونفس الدواء.|
|BR-C4|**التطبيع العربي** (`ArabicTextNormalizer.normalize`): أ/إ/آ ← ا · ى ← ي · ة ← ه · شيل التشكيل والتطويل · الأرقام الهندية (٠-٩) ← 0-9 · lowercase + `unaccent` للإنجليزي · trim ودمج المسافات. **نفس الدالة** وقت الكتابة (import) ووقت البحث.|
|BR-C5|`drug.code` ثابت وفريد، والاستيراد upsert بيه.|

**البحث (`GET /drugs/search?q=&page=&size=`):**

- `q` من 2 لـ 100 حرف. بيتطبّع بـ BR-C4.
- الـ interface `DrugSearchService` ليه implementation افتراضية `PostgresDrugSearchService` (`app.search.engine=postgres`) وبديل مستقبلي `ElasticDrugSearchService` (`elasticsearch`) بـ `@ConditionalOnProperty`.
- الـ Postgres impl (native في `DrugSearchRepository`): فلتر `active AND NOT is_controlled AND (name_en_norm % :q OR name_ar_norm % :q OR search_text ILIKE ...)` وترتيب بـ score = `GREATEST(similarity(en), similarity(ar))` + boost للتطابق الكامل (+1.0) والبداية (+0.5).
- حد التشابه الافتراضي 0.3 ممكن ينزل لـ 0.2 للعربي؛ بيتظبط بـ `SET LOCAL pg_trgm.similarity_threshold` جوه الـ transaction بعد القياس.
- **معيار الانتقال لـ Elasticsearch:** golden set من ~50 استعلام فيهم أخطاء إملائية. لو الـ recall@10 أقل من الهدف أو الـ p95 أعلى من 200ms على الداتا الحقيقية → ابدأ ES. غير كده فضّل Postgres.

**استيراد الكتالوج (ADMIN):** `POST /admin/catalog/import` (multipart CSV) ← Batch job `catalogImportJob`: reader ← validate ← normalize + `formulation_key` ← upsert بالـ `code` (chunk=500). الأعمدة: `code, trade_name_en, trade_name_ar, manufacturer, dosage_form, pack_size, official_price, is_controlled, ingredients` (حيث `ingredients` = `nameEn:strength:unit;...`). بعد الـ job: امسح cache البدائل و`ANALYZE drug`. الـ job قابل للإعادة (idempotent).

### 8.3 `pharmacy`

**المسؤولية:** بيانات الصيدلية، ربط الموظف، التوثيق، والبحث الجغرافي. **الجداول:** `pharmacy`, `pharmacy_staff`.

```java
public interface PharmacyService {            // الـ Public API
    PharmacySummaryResponse getSummary(Long id);
    List<PharmacySummaryResponse> getSummaries(Collection<Long> ids);
    List<NearbyPharmacy> findVerifiedNearby(Coordinates c, int radiusM, int limit);
    Long getPharmacyIdByStaffUser(Long userId);          // يرمي PharmacyNotFoundForUserException
    PharmacyContact getContact(Long pharmacyId);
    // + onboarding/verification للـ controllers
}
```

**دورة الحياة:** موظف يسجّل (identity) ← `POST /pharmacies` (بيتعمل `PENDING` ويتربط بالموظف) ← ADMIN يراجع الترخيص ← `VERIFIED` أو `REJECTED` (مع سبب).

|   |   |
|---|---|
|#|القاعدة|
|BR-P1|الصيدلية بتظهر للعامة `VERIFIED` بس (BR-S6).|
|BR-P2|`license_no` فريد. موظف واحد = صيدلية واحدة (MVP).|
|BR-P3|تعديل `license_no` أو الموقع بعد التوثيق بيرجّع الحالة لـ `PENDING`.|
|BR-P4|الإحداثيات: lat في [-90, 90] وlng في [-180, 180] **وجوه مربع مصر التقريبي** (lat 21.5–32، lng 24.5–37) بـ validator مخصص `@EgyptCoordinates`.|
|BR-P5|التليفون E.164. `opening_hours` JSON اختياري بصيغة `{sat:[[09:00,23:00]], ...}` أو `is_24h = true`.|

**الـ Write Slice (قالب فيه validation + `@MappingTarget`):**

```java
public record CreatePharmacyRequest(
        @NotBlank @Size(max = 150) String name,
        @NotBlank @Pattern(regexp = "^[A-Za-z0-9-/]{4,40}$") String licenseNo,
        @NotBlank @Pattern(regexp = "^[+]?[0-9]{8,15}$") String phone,
        @NotBlank @Size(max = 300) String address,
        @NotBlank String governorate, String area,
        @NotNull @DecimalMin("21.5") @DecimalMax("32.0") Double lat,
        @NotNull @DecimalMin("24.5") @DecimalMax("37.0") Double lng,
        boolean is24h, Map<String, List<String>> openingHours) {}

@Mapper(config = CommonMapperConfig.class, uses = GeoMapper.class)   // GeoMapper: (lat,lng) ← Point
public interface PharmacyMapper {
    Pharmacy toEntity(CreatePharmacyRequest request);           // location من GeoMapper
    PharmacyResponse toResponse(Pharmacy pharmacy);
    void updateEntity(UpdatePharmacyRequest request, @MappingTarget Pharmacy pharmacy);
}

// service.impl
@Override @Transactional
public PharmacyResponse create(Long staffUserId, CreatePharmacyRequest request) {
    if (staffRepository.existsByUserId(staffUserId)) throw new PharmacyAlreadyExistsException();
    if (pharmacyRepository.existsByLicenseNo(request.licenseNo())) throw new DuplicateLicenseException();
    Pharmacy p = pharmacyMapper.toEntity(request);
    p.setVerificationStatus(VerificationStatus.PENDING);
    pharmacyRepository.save(p);
    staffRepository.save(new PharmacyStaff(staffUserId, p.getId()));
    return pharmacyMapper.toResponse(p);
}

// controller
@PostMapping
@PreAuthorize("hasRole('PHARMACY_STAFF')")
public ResponseEntity<ApiResponse<PharmacyResponse>> create(
        @AuthenticationPrincipal AuthenticatedUser user, @Valid @RequestBody CreatePharmacyRequest request) {
    return ResponseEntity.status(HttpStatus.CREATED)
            .body(ApiResponse.created(pharmacyService.create(user.userId(), request)));
}
```

**البحث الجغرافي (native في `PharmacyGeoRepository`، استثناء موثّق):** `ST_DWithin` على `geography` مع GiST index، والمسافة بالمتر بـ `ST_Distance`، والترتيب بالأقرب. **انتبه:** `ST_MakePoint(lng, lat)` — الـ lng الأول. الـ aliases لازم تطابق أسماء الـ projection.

```sql
SELECT p.id, p.name, p.address, p.phone, p.is_24h,
       ST_Distance(p.location, pt.g) AS distance_m,
       ST_Y(p.location::geometry) AS lat, ST_X(p.location::geometry) AS lng
FROM pharmacy p
CROSS JOIN (SELECT ST_SetSRID(ST_MakePoint(:lng, :lat), 4326)::geography AS g) pt
WHERE p.verification_status = 'VERIFIED' AND ST_DWithin(p.location, pt.g, :radiusM)
ORDER BY distance_m LIMIT :limit
```

### 8.4 `inventory`

**المسؤولية:** مخزون كل صيدلية (status + TTL) واستيراده. **الجداول:** `pharmacy_stock` (PK = `pharmacy_id + drug_id`), `stock_import_job`, `stock_import_error`.

```java
public interface StockService {               // الـ Public API
    List<StockView> findStocks(Collection<Long> pharmacyIds, Collection<Long> drugIds);
    void upsert(UpsertStockCommand cmd);      // (pharmacyId, drugId, status, quantity?, source)
}
// StockView(pharmacyId, drugId, status, source, updatedAt, expiresAt)  ← من غير quantity (داخلي)
```

|   |   |
|---|---|
|#|القاعدة|
|BR-I1|upsert على `(pharmacy_id, drug_id)`. آخر كتابة بتكسب.|
|BR-I2|`expires_at = updated_at + TTL(source)` والـ TTL من `app.availability.ttl.*` (قسم 12).|
|BR-I3|حالة «غير مؤكد» بتتحسب وقت القراءة (D10) — مفيش job بيعدّل الصفوف.|
|BR-I4|الـ status من الصف لو موجود، وإلا بيتحسب من الـ quantity: 0 ← `OUT` · ≤ `app.inventory.low-threshold` (افتراضي 5) ← `LOW` · غير كده `AVAILABLE`. الـ quantity داخلي ومبيظهرش للمرضى.|
|BR-I5|سبب رفض الصف: `UNKNOWN_OR_RESTRICTED_CODE` (كود مش موجود أو الدواء مخدر/موقوف — `findIdsByCodes` بترجّع الأدوية المرئية بس) · `INVALID_STATUS` · `INVALID_QUANTITY` · `MISSING_VALUE`. تكرار نفس الكود في الملف = آخر صف بيكسب.|
|BR-I6|**نجاح جزئي:** الصفوف الصالحة بتتطبق والفاشلة بتتسجل (أقصى 1000 خطأ) والـ job بيبقى `COMPLETED` مع `rows_failed`. الملف التالف أو الـ header الغلط أو تجاوز الحدود = `FAILED` من غير تطبيق أي صف.|
|BR-I7|الحدود: ملف ≤ 5MB و≤ 20,000 صف · CSV بـ UTF-8 (BOM مقبول) · xlsx في مرحلة لاحقة (Apache POI reader).|
|BR-I8|الصيدلية لازم تكون `VERIFIED` عشان تحدّث المخزون، وإلا 403 `PHARMACY_NOT_VERIFIED`.|

**الـ CSV Template:** `drug_code,status,quantity` (status: AVAILABLE/LOW/OUT — واحد منهم أو quantity على الأقل). بيتنزّل من `GET /pharmacies/me/stock/template`.

**تدفق الاستيراد:** `POST /pharmacies/me/stock/import` ← تحقق سريع من الملف وحفظه مؤقتًا (`app.inventory.import-dir`) ← سجل `stock_import_job (PENDING)` ← تشغيل الـ Batch job غير متزامن ← **202** + `ImportJobResponse(jobId, status)`. الصيدلية بتتابع بـ `GET .../import-jobs/{jobId}` وتشوف الأخطاء من `.../errors`.

**الـ Batch job:** Reader (`FlatFileItemReader`) ← Processor (تحقق syntactic: status/quantity) ← Writer: **لكل chunk (500)** استدعاء واحد لـ `DrugService.findIdsByCodes` + upsert دفعة واحدة بـ JDBC + تسجيل الصفوف الفاشلة. الملف بيتمسح في نهاية الـ job.

```sql
INSERT INTO pharmacy_stock (pharmacy_id, drug_id, status, quantity, source, updated_at, expires_at)
VALUES (:pharmacyId, :drugId, :status, :quantity, :source, :now, :expiresAt)
ON CONFLICT (pharmacy_id, drug_id) DO UPDATE SET
  status = EXCLUDED.status, quantity = EXCLUDED.quantity, source = EXCLUDED.source,
  updated_at = EXCLUDED.updated_at, expires_at = EXCLUDED.expires_at;
```

(Native في `StockUpsertRepository` — استثناء موثّق. لو Spring Batch تقيل على الـ MVP: fallback = `@Async` service بيعمل `JdbcTemplate.batchUpdate` على chunks.)

### 8.5 `availability`

**المسؤولية:** الواجهة اللي المريض بيشوفها: البحث عن التوافر، طلبات الـ broadcast، والبلاغات. **هي طبقة تجميع:** مبتملكش بيانات دواء ولا صيدلية ولا مخزون، بتنادي `catalog` و`pharmacy` و`inventory`. **الجداول:** `availability_request`, `availability_request_target`, `availability_response`, `availability_report`.

**(أ) بحث التوافر — `GET /availability?drugId&lat&lng&radiusM`**

1. تحقق: الإحداثيات + `radiusM` (افتراضي 3000، من 500 لـ 10000) — BR-A1.
2. `DrugService.getById(drugId)` (404 لو مخدر/مش موجود) ثم `findAlternatives(drugId)`.
3. `PharmacyService.findVerifiedNearby(coords, radiusM, 100)`. لو فاضي ← نتيجة فاضية + `suggestRequest = true`.
4. `StockService.findStocks(pharmacyIds, [drugId] + altIds)`.
5. بلاغات إيجابية حديثة من `availability_report` للـ (pharmacy, drug) خلال `report-ttl`.
6. لكل (pharmacy, drug) احسب `ConfirmationState` (الجدول تحت) وتجاهل `OUT` والمجهول.
7. رتّب: **(isAlternative، المسافة)** — الدواء الأصلي الأول بالأقرب، وبعده البدائل بالأقرب. حد أقصى 30 نتيجة.
8. لو في بدائل في الرد ← `disclaimer` (BR-S1).

|   |   |   |
|---|---|---|
|ConfirmationState|الشرط|يتعرض|
|`CONFIRMED`|stock حالته AVAILABLE/LOW و`expires_at > now`|«متوفر — آخر تأكيد من X دقيقة»|
|`UNCONFIRMED`|stock حالته AVAILABLE/LOW و`expires_at <= now`|«غير مؤكد — آخر تحديث X» + زر إرسال طلب|
|`USER_REPORTED`|مفيش stock صالح، وفي ≥ 1 بلاغ «لقيته هنا» خلال `report-ttl`|«بلاغ مستخدم» (وزن أقل)|
|(مش بيظهر)|`OUT` صالح أو مفيش معلومة|—|

```json
{ "success": true, "code": "OK", "data": {
  "drug": { "id": 12, "tradeNameEn": "..." },
  "results": [ {
     "pharmacy": { "id": 5, "name": "...", "distanceM": 420, "lat": 30.04, "lng": 31.23 },
     "drug": { "id": 12, "tradeNameEn": "..." }, "alternative": false,
     "stockStatus": "AVAILABLE", "confirmationState": "CONFIRMED",
     "lastConfirmedAt": "2026-10-24T09:40:00Z", "minutesSinceConfirmed": 35 } ],
  "suggestRequest": false,
  "disclaimer": "..." } }
```

لو الأداء بقى مشكلة بعد القياس: استبدل الخطوات 3–5 بـ native join واحد داخل `availability` كاستثناء R8 موثّق.

**(ب) الطلبات (Broadcast)**

|   |   |
|---|---|
|#|القاعدة|
|BR-A1|البحث: الافتراضي 3 كم، الحد 0.5–10 كم.|
|BR-A2|الطلب: أقصى 3 طلبات `OPEN` للمريض الواحد، والـ TTL 30 دقيقة (`app.availability.request-ttl`)، وأقصى 15 صيدلية مستهدفة (الأقرب).|
|BR-A3|الصيدلية بترد مرة واحدة على الطلب (409 `ALREADY_RESPONDED`)، ولازم تكون ضمن الـ targets والطلب `OPEN` وغير منتهي.|
|BR-A4|الرد `ALTERNATIVE` لازم `alternativeDrugId` يكون من بدائل الدواء المطلوب (يتحقق منه عبر `DrugService.findAlternatives`).|
|BR-A5|أثر الرد (في نفس الـ transaction عبر `StockService.upsert` بـ source = `REQUEST_REPLY`): `AVAILABLE` ← المطلوب AVAILABLE · `OUT` ← المطلوب OUT · `ALTERNATIVE` ← المطلوب OUT + البديل AVAILABLE. أول رد إيجابي بيخلّي الطلب `FULFILLED`.|
|BR-A6|الصيدلية بتشوف طلبات مفتوحة من غير هوية المريض ومن غير موقعه الدقيق (BR-S5).|
|BR-A7|حالة `EXPIRED` بتتحسب lazily عند القراءة، و`@Scheduled` كل ساعة بيقفل المنتهي ويمسح الإحداثيات القديمة (> 30 يوم).|

**التدفق:** `POST /availability/requests` ← تحقق + BR-A2 ← اختيار الـ targets ← حفظ request + targets ← نشر `AvailabilityRequestCreatedEvent(requestId, drugId, pharmacyIds)` بعد الـ commit ← **201**. المريض بيتابع بـ `GET /availability/requests/{id}` (polling؛ SSE اختياري بعدين).

**(ج) البلاغات — `POST /availability/reports` `{pharmacyId, drugId, found}`**

- **BR-A8:** بلاغ واحد لكل (user, pharmacy, drug) كل 6 ساعات، والصيدلية `VERIFIED` والدواء مرئي، والمريض مسجّل.
- البلاغ بيأثر وقت القراءة بس (يحسب `USER_REPORTED`)، ومبيكتبش على `pharmacy_stock`.
- (مرحلة 2 اختيارية) `DISPUTED`: بلاغان سلبيان أو أكتر من مستخدمين مختلفين خلال ساعتين على صيدلية حالتها `CONFIRMED` ← تظهر «غير مؤكد».

### 8.6 `notification`

**المسؤولية:** توصيل إشعارات الطلبات للصيدليات. **الجدول:** `notification`.

- `NotificationChannel` (Strategy): `type()` و`send(message)`. التنفيذات: `InAppChannel` (بيكتب صف في `notification` والموظف بيقراه — **الافتراضي للـ MVP**) · `LoggingChannel` (dev) · لاحقًا `PushChannel` / `WhatsAppChannel`.
- الـ listener: `@ApplicationModuleListener` على `AvailabilityRequestCreatedEvent` ← لكل `pharmacyId`: `PharmacyService.getContact()` ← بناء الرسالة ← إرسال. فشل صيدلية واحدة بيتسجل ومبيوقفش الباقي، ومبيأثرش على إنشاء الطلب (async بعد commit).
- لو الـ event فشل معالجته، سجل الـ event publication في Modulith بيسمح بإعادة المحاولة.
- Endpoints (اختيارية في MVP): `GET /pharmacies/me/notifications` · `PATCH /pharmacies/me/notifications/{id}/read`.

### 8.7 `ai` (اختياري — آخر مرحلة)

- **الاستخدام الوحيد المقبول:** OCR لصورة الروشتة أو العلبة لاستخراج اسم الدواء. بعدها البحث والبدائل يكملوا بـ SQL حتمي (D12).
- `PrescriptionOcrService.extractDrugCandidates(image)` ← قايمة أسماء مرشّحة ← بتتمرر لـ `DrugSearchService` ← المستخدم **يختار ويأكّد** من القايمة.
- الـ LLM مبياخدش قرار طبي ومبيختارش بدائل. لو استُخدم LLM للاستخراج، الـ system prompt بتاع v1 بيتطبق حرفيًا.
- Guardrails: حد حجم الصورة · الصورة مبتتخزنش · أي بيانات شخصية على الروشتة مبتتسجلش · الـ provider ورا interface `OcrProvider` عشان يتبدّل.

## 9. قاعدة البيانات — Flyway Migrations

الملفات في `src/main/resources/db/migration` بتسمية `V<رقم>__<feature>_<وصف>.sql`. قواعد: الـ migration **مبتتعدّلش** بعد ما تتطبق (أي تغيير = ملف جديد) · أي entity بيورّث `AuditableEntity` لازم جدوله فيه `created_at` و`updated_at` · الـ enums مخزّنة `VARCHAR` + `CHECK`.

|   |   |
|---|---|
|الملف|المحتوى|
|`V1__common_extensions.sql`|PostGIS · pg_trgm · unaccent|
|`V2__identity_tables.sql`|app_user · refresh_token|
|`V3__catalog_tables.sql`|active_ingredient · manufacturer · drug · drug_ingredient · catalog_import_job|
|`V4__pharmacy_tables.sql`|pharmacy · pharmacy_staff|
|`V5__inventory_tables.sql`|pharmacy_stock · stock_import_job · stock_import_error|
|`V6__availability_tables.sql`|request · target · response · report|
|`V7__notification_tables.sql`|notification|
|`V8__batch_schema.sql`|انسخ `schema-postgresql.sql` من `spring-batch-core` بنفس إصدار Batch المستخدم، وخلّي `spring.batch.jdbc.initialize-schema=never`|
|`V9__modulith_events.sql`|جدول `event_publication` الخاص بـ Spring Modulith (من الـ docs بتاعة الإصدار)|

```sql
-- V1__common_extensions.sql
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS unaccent;

-- V2__identity_tables.sql   (الـ email بيتخزن lowercase من الـ service)
CREATE TABLE app_user (
  id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  full_name     VARCHAR(120) NOT NULL,
  email         VARCHAR(255),
  phone         VARCHAR(20),
  password_hash VARCHAR(100) NOT NULL,
  role          VARCHAR(20)  NOT NULL,
  enabled       BOOLEAN      NOT NULL DEFAULT TRUE,
  created_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ  NOT NULL DEFAULT now(),
  CONSTRAINT uq_app_user_email UNIQUE (email),
  CONSTRAINT uq_app_user_phone UNIQUE (phone),
  CONSTRAINT chk_app_user_role CHECK (role IN ('PATIENT','PHARMACY_STAFF','ADMIN')),
  CONSTRAINT chk_app_user_contact CHECK (email IS NOT NULL OR phone IS NOT NULL)
);

CREATE TABLE refresh_token (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id    BIGINT      NOT NULL REFERENCES app_user(id) ON DELETE CASCADE,
  token_hash CHAR(64)    NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  revoked    BOOLEAN     NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT uq_refresh_token_hash UNIQUE (token_hash)
);
CREATE INDEX idx_refresh_token_user ON refresh_token(user_id);

-- V3__catalog_tables.sql
CREATE TABLE active_ingredient (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name_en    VARCHAR(200) NOT NULL,
  name_ar    VARCHAR(200),
  name_norm  VARCHAR(200) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT uq_active_ingredient_norm UNIQUE (name_norm)
);

CREATE TABLE manufacturer (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name_en    VARCHAR(200) NOT NULL,
  name_ar    VARCHAR(200),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT uq_manufacturer_name UNIQUE (name_en)
);

CREATE TABLE drug (
  id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  code            VARCHAR(50)  NOT NULL,
  trade_name_en   VARCHAR(200) NOT NULL,
  trade_name_ar   VARCHAR(200),
  name_en_norm    VARCHAR(200) NOT NULL,
  name_ar_norm    VARCHAR(200),
  search_text     TEXT         NOT NULL,   -- أسماء + مواد فعالة + شركة (مطبّعة)
  manufacturer_id BIGINT REFERENCES manufacturer(id),
  dosage_form     VARCHAR(50),
  pack_size       VARCHAR(50),
  official_price  NUMERIC(10,2),
  is_controlled   BOOLEAN      NOT NULL DEFAULT FALSE,
  active          BOOLEAN      NOT NULL DEFAULT TRUE,
  formulation_key CHAR(64),
  created_at      TIMESTAMPTZ  NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ  NOT NULL DEFAULT now(),
  CONSTRAINT uq_drug_code UNIQUE (code),
  CONSTRAINT chk_drug_price CHECK (official_price IS NULL OR official_price >= 0)
);
CREATE INDEX idx_drug_formulation_key ON drug(formulation_key) WHERE active AND NOT is_controlled;
CREATE INDEX idx_drug_name_en_trgm    ON drug USING GIN (name_en_norm gin_trgm_ops);
CREATE INDEX idx_drug_name_ar_trgm    ON drug USING GIN (name_ar_norm gin_trgm_ops);
CREATE INDEX idx_drug_search_trgm     ON drug USING GIN (search_text gin_trgm_ops);

CREATE TABLE drug_ingredient (
  drug_id        BIGINT NOT NULL REFERENCES drug(id) ON DELETE CASCADE,
  ingredient_id  BIGINT NOT NULL REFERENCES active_ingredient(id),
  strength_value NUMERIC(12,4),
  strength_unit  VARCHAR(10),               -- بعد التطبيع (mg / ml / IU / %)
  PRIMARY KEY (drug_id, ingredient_id)
);
CREATE INDEX idx_drug_ingredient_ingredient ON drug_ingredient(ingredient_id);

CREATE TABLE catalog_import_job (
  id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  file_name    VARCHAR(255) NOT NULL,
  status       VARCHAR(12)  NOT NULL DEFAULT 'PENDING',
  rows_total   INTEGER NOT NULL DEFAULT 0,
  rows_failed  INTEGER NOT NULL DEFAULT 0,
  error_sample JSONB,                        -- أول 100 خطأ
  created_by   BIGINT REFERENCES app_user(id),
  started_at   TIMESTAMPTZ, finished_at TIMESTAMPTZ,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT chk_catalog_job_status CHECK (status IN ('PENDING','RUNNING','COMPLETED','FAILED'))
);

-- V4__pharmacy_tables.sql
CREATE TABLE pharmacy (
  id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name                VARCHAR(150) NOT NULL,
  license_no          VARCHAR(40)  NOT NULL,
  phone               VARCHAR(20)  NOT NULL,
  address             VARCHAR(300) NOT NULL,
  governorate         VARCHAR(60)  NOT NULL,
  area                VARCHAR(80),
  location            geography(Point,4326) NOT NULL,
  opening_hours       JSONB,
  is_24h              BOOLEAN NOT NULL DEFAULT FALSE,
  verification_status VARCHAR(10) NOT NULL DEFAULT 'PENDING',
  rejection_reason    VARCHAR(300),
  verified_by         BIGINT REFERENCES app_user(id),
  verified_at         TIMESTAMPTZ,
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT uq_pharmacy_license UNIQUE (license_no),
  CONSTRAINT chk_pharmacy_status CHECK (verification_status IN ('PENDING','VERIFIED','REJECTED'))
);
CREATE INDEX idx_pharmacy_location ON pharmacy USING GIST (location);
CREATE INDEX idx_pharmacy_status   ON pharmacy(verification_status);

CREATE TABLE pharmacy_staff (
  user_id     BIGINT NOT NULL UNIQUE REFERENCES app_user(id) ON DELETE CASCADE,   -- MVP: صيدلية واحدة للموظف
  pharmacy_id BIGINT NOT NULL REFERENCES pharmacy(id) ON DELETE CASCADE,
  PRIMARY KEY (user_id, pharmacy_id)
);

-- V5__inventory_tables.sql   (pharmacy_stock مش بيورّث AuditableEntity: عنده updated_at بس)
CREATE TABLE pharmacy_stock (
  pharmacy_id BIGINT NOT NULL REFERENCES pharmacy(id) ON DELETE CASCADE,
  drug_id     BIGINT NOT NULL REFERENCES drug(id),
  status      VARCHAR(10) NOT NULL,
  quantity    INTEGER,
  source      VARCHAR(20) NOT NULL,
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at  TIMESTAMPTZ NOT NULL,
  PRIMARY KEY (pharmacy_id, drug_id),
  CONSTRAINT chk_stock_status   CHECK (status IN ('AVAILABLE','LOW','OUT')),
  CONSTRAINT chk_stock_source   CHECK (source IN ('CSV','API','MANUAL','REQUEST_REPLY')),
  CONSTRAINT chk_stock_quantity CHECK (quantity IS NULL OR quantity >= 0)
);
CREATE INDEX idx_stock_drug_active ON pharmacy_stock(drug_id) WHERE status <> 'OUT';

CREATE TABLE stock_import_job (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  pharmacy_id BIGINT NOT NULL REFERENCES pharmacy(id) ON DELETE CASCADE,
  file_name   VARCHAR(255) NOT NULL,
  status      VARCHAR(12)  NOT NULL DEFAULT 'PENDING',
  rows_total  INTEGER NOT NULL DEFAULT 0,
  rows_failed INTEGER NOT NULL DEFAULT 0,
  started_at  TIMESTAMPTZ, finished_at TIMESTAMPTZ,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT chk_stock_job_status CHECK (status IN ('PENDING','RUNNING','COMPLETED','FAILED'))
);
CREATE INDEX idx_stock_job_pharmacy ON stock_import_job(pharmacy_id, created_at DESC);

CREATE TABLE stock_import_error (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  job_id     BIGINT NOT NULL REFERENCES stock_import_job(id) ON DELETE CASCADE,
  row_number INTEGER NOT NULL,
  raw_line   VARCHAR(500),
  reason     VARCHAR(40) NOT NULL
);
CREATE INDEX idx_stock_error_job ON stock_import_error(job_id, row_number);

-- V6__availability_tables.sql
CREATE TABLE availability_request (
  id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  patient_id BIGINT NOT NULL REFERENCES app_user(id),
  drug_id    BIGINT NOT NULL REFERENCES drug(id),
  location   geography(Point,4326) NOT NULL,
  radius_m   INTEGER NOT NULL,
  status     VARCHAR(12) NOT NULL DEFAULT 'OPEN',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at TIMESTAMPTZ NOT NULL,
  CONSTRAINT chk_request_radius CHECK (radius_m BETWEEN 500 AND 10000),
  CONSTRAINT chk_request_status CHECK (status IN ('OPEN','FULFILLED','EXPIRED','CANCELLED'))
);
CREATE INDEX idx_request_patient_status ON availability_request(patient_id, status);
CREATE INDEX idx_request_open_expiry    ON availability_request(expires_at) WHERE status = 'OPEN';

CREATE TABLE availability_request_target (
  request_id  BIGINT NOT NULL REFERENCES availability_request(id) ON DELETE CASCADE,
  pharmacy_id BIGINT NOT NULL REFERENCES pharmacy(id),
  notified_at TIMESTAMPTZ,
  PRIMARY KEY (request_id, pharmacy_id)
);
CREATE INDEX idx_target_pharmacy ON availability_request_target(pharmacy_id);

CREATE TABLE availability_response (
  id                  BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  request_id          BIGINT NOT NULL,
  pharmacy_id         BIGINT NOT NULL,
  status              VARCHAR(12) NOT NULL,
  alternative_drug_id BIGINT REFERENCES drug(id),
  responded_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT fk_response_target FOREIGN KEY (request_id, pharmacy_id)
    REFERENCES availability_request_target(request_id, pharmacy_id),
  CONSTRAINT uq_response_once   UNIQUE (request_id, pharmacy_id),
  CONSTRAINT chk_response_status CHECK (status IN ('AVAILABLE','OUT','ALTERNATIVE')),
  CONSTRAINT chk_response_alt   CHECK ((status = 'ALTERNATIVE') = (alternative_drug_id IS NOT NULL))
);

CREATE TABLE availability_report (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id     BIGINT NOT NULL REFERENCES app_user(id),
  pharmacy_id BIGINT NOT NULL REFERENCES pharmacy(id),
  drug_id     BIGINT NOT NULL REFERENCES drug(id),
  found       BOOLEAN NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_report_lookup ON availability_report(pharmacy_id, drug_id, created_at DESC);
CREATE INDEX idx_report_dedupe ON availability_report(user_id, pharmacy_id, drug_id, created_at DESC);

-- V7__notification_tables.sql
CREATE TABLE notification (
  id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  pharmacy_id  BIGINT NOT NULL REFERENCES pharmacy(id) ON DELETE CASCADE,
  type         VARCHAR(40) NOT NULL,
  reference_id BIGINT,
  payload      JSONB,
  channel      VARCHAR(20) NOT NULL,
  status       VARCHAR(10) NOT NULL DEFAULT 'PENDING',
  created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
  read_at      TIMESTAMPTZ,
  CONSTRAINT chk_notification_status CHECK (status IN ('PENDING','SENT','FAILED','READ'))
);
CREATE INDEX idx_notification_pharmacy ON notification(pharmacy_id, created_at DESC);
```

بعد ما الـ schema يشتغل: اتأكد بـ `EXPLAIN ANALYZE` إن الـ queries الأساسية (بحث trigram، `ST_DWithin`، البدائل) بتستخدم الـ indexes، وسجّل النتيجة في الـ README.

## 10. الـ API Catalog

الـ base path: `/api/v1`. **Public*** = متاح من غير تسجيل (قرار 18.3) لكن عليه rate limit. كل الردود بالـ envelope (قسم 6) إلا تنزيل الـ CSV template (ملف مباشر؛ أخطاؤه بالـ envelope).

|   |   |   |   |   |
|---|---|---|---|---|
|Method|Path|الدور|Request ← Response|Status|
|POST|`/auth/register`|Public|`RegisterRequest` ← `AuthResponse`|201|
|POST|`/auth/login`|Public|`LoginRequest` ← `AuthResponse`|200|
|POST|`/auth/refresh`|Public|`RefreshRequest` ← `AuthResponse`|200|
|POST|`/auth/logout`|Auth|`RefreshRequest` ← فاضي|200|
|GET|`/users/me`|Auth|← `UserResponse`|200|
|GET|`/drugs/search?q&page&size`|Public*|← `PageResponse<DrugSearchResult>`|200|
|GET|`/drugs/{id}`|Public*|← `DrugResponse`|200|
|GET|`/drugs/{id}/alternatives`|Public*|← `AlternativesResponse`|200|
|POST|`/admin/catalog/import`|ADMIN|multipart ← `ImportJobResponse`|202|
|GET|`/admin/catalog/import-jobs/{jobId}`|ADMIN|← `ImportJobResponse`|200|
|POST|`/pharmacies`|STAFF|`CreatePharmacyRequest` ← `PharmacyResponse`|201|
|GET|`/pharmacies/me`|STAFF|← `PharmacyResponse`|200|
|PUT|`/pharmacies/me`|STAFF|`UpdatePharmacyRequest` ← `PharmacyResponse`|200|
|GET|`/pharmacies/nearby?lat&lng&radiusM`|Public*|← `List<NearbyPharmacyResponse>`|200|
|GET|`/admin/pharmacies?status&page&size`|ADMIN|← `PageResponse<PharmacyAdminResponse>`|200|
|PATCH|`/admin/pharmacies/{id}/verification`|ADMIN|`VerifyPharmacyRequest(decision, reason?)` ← `PharmacyAdminResponse`|200|
|POST|`/pharmacies/me/stock/import`|STAFF|multipart ← `ImportJobResponse`|202|
|GET|`/pharmacies/me/stock/import-jobs/{jobId}`|STAFF|← `ImportJobResponse`|200|
|GET|`/pharmacies/me/stock/import-jobs/{jobId}/errors`|STAFF|← `PageResponse<ImportErrorResponse>`|200|
|GET|`/pharmacies/me/stock?page&size`|STAFF|← `PageResponse<StockItemResponse>`|200|
|PUT|`/pharmacies/me/stock/{drugId}`|STAFF|`UpdateStockRequest(status, quantity?)` ← `StockItemResponse`|200|
|GET|`/pharmacies/me/stock/template`|STAFF|← ملف CSV|200|
|GET|`/availability?drugId&lat&lng&radiusM`|Public*|← `AvailabilitySearchResponse`|200|
|POST|`/availability/requests`|PATIENT|`CreateAvailabilityRequest(drugId, lat, lng, radiusM)` ← `RequestResponse`|201|
|GET|`/availability/requests/{id}`|PATIENT (owner)|← `RequestDetailsResponse`|200|
|POST|`/availability/reports`|PATIENT|`CreateReportRequest(pharmacyId, drugId, found)` ← `ReportResponse`|201|
|GET|`/pharmacies/me/requests?status`|STAFF|← `List<PharmacyRequestResponse>`|200|
|POST|`/pharmacies/me/requests/{requestId}/respond`|STAFF|`RespondRequest(status, alternativeDrugId?)` ← فاضي|200|
|GET|`/pharmacies/me/notifications?page&size`|STAFF|← `PageResponse<NotificationResponse>`|200|
|PATCH|`/pharmacies/me/notifications/{id}/read`|STAFF|← فاضي|200|
|GET|`/actuator/health`|Public|—|200|

ملاحظة: `/swagger-ui` و`/v3/api-docs` في `dev` بس. أي endpoint بيرجع PII أو بيكتب لازم يكون مش Public.

## 11. الأمان (Security)

- **الـ chain:** stateless (`SessionCreationPolicy.STATELESS`) · CSRF مقفول (JWT في header) · CORS من properties · `@EnableMethodSecurity` والصلاحيات بـ `@PreAuthorize`.
- **الـ public paths** بتتقرا من `app.security.public-paths` في الـ yml عشان `identity` متعرفش مسارات الـ features التانية. أي مسار تاني = `authenticated()`.
- **الـ principal:** `JwtAuthenticationFilter` بيبني `AuthenticatedUser(userId, role)` من الـ claims من غير DB hit، ويحطه في `SecurityContext` بـ authority `ROLE_<role>`.
- **الـ Ownership:** عمليات الصيدلية على `/pharmacies/me/**` (D9) والـ service بتجيب `pharmacyId` من `getPharmacyIdByStaffUser(userId)` — مفيش `{pharmacyId}` من الـ client. موارد المريض بتتفحص بـ `patientId == currentUser.userId()` ولو غلط → **404** مش 403 (منع التخمين).
- **JWT:** HS256 بسر ≥ 256-bit من env (`APP_JWT_SECRET`) ومبيتعملوش commit. access = 15 دقيقة، refresh = 7 أيام. (RS256 اختياري لاحقًا.)
- **كلمات السر:** BCrypt بـ strength 10–12. رسالة فشل الدخول واحدة عامة (من غير تحديد إن كان الإيميل مش موجود).
- **Rate limiting (`RateLimitFilter` بـ Bucket4j + Redis):** public 60 طلب/دقيقة/IP · `/auth/**` 10/دقيقة/IP · authenticated 120/دقيقة/مستخدم. الزيادة = 429 بالـ envelope + `Retry-After`.
- **الـ uploads:** تحقق من الامتداد **والمحتوى** (مش `Content-Type` بس) · حد 5MB (`spring.servlet.multipart.max-file-size`) · اسم ملف مؤقت عشوائي ومفيش path traversal · الملف بيتمسح بعد الـ job.
- **الـ Input:** DTO validation على كل مدخل · الـ native queries بـ bind parameters فقط (`:name`) ومفيش string concat · مفيش bind مباشر على Entities (يمنع mass assignment).
- **الخصوصية:** BR-S5 · الـ logs بتخفي التليفون/الإيميل ومفيهاش lat/lng · `X-Request-Id` لتتبع الـ request من غير PII.
- **الأسرار:** env vars أو `.env` (في `.gitignore`) أو CI secrets. مفيش أسرار في `application.yml`.
- **قايمة OWASP سريعة:** IDOR (D9 + 404) · Mass assignment (records) · Brute force (rate limit) · Enumeration (رسالة عامة) · Injection (bind params) · Sensitive data exposure (DTOs بس).

## 12. الـ Configuration والـ Infra

### 12.1 `application.yml` (الأساس)

```yaml
spring:
  application:
    name: pharmacy-availability
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/pharmacy}
    username: ${DB_USER:pharmacy}
    password: ${DB_PASSWORD:pharmacy}
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate          # Flyway هو اللي بيملك الـ schema
    properties:
      hibernate.jdbc.time_zone: UTC
  flyway:
    enabled: true
  batch:
    jdbc:
      initialize-schema: never    # الـ schema جاي من V8
    job:
      enabled: false              # الـ jobs بتتشغل من الـ API مش وقت الـ startup
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: 6379
    web:
      pageable:
        default-page-size: 20
        max-page-size: 50
  servlet:
    multipart:
      max-file-size: 5MB
      max-request-size: 6MB

app:
  admin:
    email: ${ADMIN_EMAIL:}
    password: ${ADMIN_PASSWORD:}
  security:
    jwt:
      secret: ${APP_JWT_SECRET}     # إجباري من env
      access-ttl: 15m
      refresh-ttl: 7d
    cors-allowed-origins: ${CORS_ORIGINS:http://localhost:3000}
    public-paths:
      - /api/v1/auth/**
      - /api/v1/drugs/**
      - /api/v1/pharmacies/nearby
      - /api/v1/availability
      - /actuator/health
  rate-limit:
    public-per-minute: 60
    auth-per-minute: 10
    user-per-minute: 120
  search:
    engine: postgres                # postgres | elasticsearch
    min-query-length: 2
    similarity-threshold: 0.3
  inventory:
    low-threshold: 5
    max-rows: 20000
    import-dir: ${java.io.tmpdir}/pharmacy-imports
  availability:
    default-radius-m: 3000
    min-radius-m: 500
    max-radius-m: 10000
    max-results: 30
    max-targets: 15
    max-open-requests-per-patient: 3
    request-ttl: 30m
    report-ttl: 3h
    report-dedupe-window: 6h
    request-location-retention: 30d
    ttl:                            # عمر معلومة المخزون حسب المصدر (BR-I2)
      csv: 36h
      api: 6h
      manual: 12h
      request-reply: 2h
```

- كل feature عندها `@ConfigurationProperties` record خاص بيها في `<feature>/config` (مع `@ConfigurationPropertiesScan` على الـ Application class). مثال: `AvailabilityProperties`، و`StockTtlProperties` (مملوك لـ `inventory` وبيربط على `app.availability.ttl` — الـ prefix اسمه availability لأنه بيعبّر عن عمر معلومة التوافر، لكن الـ `availability` feature نفسها مبتقراهوش).
- الـ profiles: `dev` (swagger شغال، SQL logging، `LoggingChannel`) · `test` (Testcontainers) · `prod` (من غير swagger، JSON logs، actuator مقيّد).

### 12.2 `docker-compose.yml`

```yaml
services:
  postgres:
    image: postgis/postgis:16-3.4
    environment:
      POSTGRES_DB: pharmacy
      POSTGRES_USER: pharmacy
      POSTGRES_PASSWORD: pharmacy
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7
    ports:
      - 6379:6379
volumes:
  pgdata:
```

### 12.3 الـ Dependencies والـ Build

- **من start.spring.io:** Web · Validation · Data JPA · PostgreSQL Driver · Flyway · Security · Data Redis · Batch · Actuator · Lombok · Spring Modulith · Testcontainers.
- **إضافات يدوية (راجع الإصدار المتوافق مع Boot 4):** `hibernate-spatial` · MapStruct (+ processor + `lombok-mapstruct-binding`) · springdoc-openapi · Bucket4j (Redis) · مكتبة JWT (الأسهل: `spring-security-oauth2-jose` أو jjwt — اختار واحدة).
- ترتيب الـ annotation processors في الـ compiler plugin: **Lombok ← lombok-mapstruct-binding ← MapStruct**.
- الـ Java toolchain 21، وفعّل `-parameters` (الـ Boot parent بيعمله) عشان `@Cacheable(key = "#id")` و`@PathVariable`.

## 13. الـ Caching والـ Async والـ Observability

### 13.1 الـ Cache (Redis)

|   |   |   |   |
|---|---|---|---|
|Cache|المفتاح|TTL|بيتمسح لما|
|`drugAlternatives`|`drugId`|6 ساعات|استيراد الكتالوج يخلص|
|`drugDetails` (اختياري)|`drugId`|6 ساعات|استيراد الكتالوج يخلص|
|**ممنوع:** التوافر، المخزون، نتائج `/availability`|—|—|—|

- أسماء الـ caches في `CacheNames` (constants) جوه `common`، والـ `RedisCacheManager` بـ TTL لكل cache في `CacheConfig`.
- الـ cached values records → اختار serializer بيتعامل معاها ومتوافق مع Jackson 3 (أو خلّيها `Serializable`). اختبر الـ serialize/deserialize صراحة.

### 13.2 الـ Async والـ Scheduling

- `AsyncConfig`: executor باسم `importExecutor` (core 2 / max 4 / queue 50) لتشغيل الـ Batch imports، وexecutor للـ `@ApplicationModuleListener`. (اختياري: `spring.threads.virtual.enabled=true` مع Java 21.)
- `@EnableScheduling`: `ExpiredRequestsCleaner` كل ساعة (BR-A7). لو اتشغّل أكتر من instance → ShedLock.
- الـ events بتتنشر **بعد الـ commit** (`@ApplicationModuleListener` بيعمل ده)، وفشل الـ listener مبيكسرش الـ request الأصلي.

### 13.3 الـ Observability

- Actuator: `health`, `info`, `metrics`, `prometheus` (الأخيرين مقيّدين/internal). Health يشمل DB وRedis.
- Micrometer metrics مخصصة: `stock_import_rows_total{result}` · `availability_search_duration` · `availability_request_created_total` · `availability_response_total{status}` · `notification_sent_total{channel,result}`.
- Logging: `@Slf4j` · MDC بـ `requestId` · JSON في `prod` · **ممنوع PII** (BR-S5). مستويات: INFO للأحداث التجارية، WARN للرفض/الحدود، ERROR للأخطاء غير المتوقعة بس.

## 14. استراتيجية الاختبار

|   |   |   |   |
|---|---|---|---|
|النوع|الأدوات|بيغطي|مثال|
|Unit|JUnit 5 · Mockito · AssertJ|قواعد الـ service (BR)، الـ mappers، الـ normalizer، `FormulationKeyGenerator`، حساب `ConfirmationState`|`ConfirmationStateCalculatorTest`|
|Repository|`@DataJpaTest` + Testcontainers PostGIS|derived queries + الـ native (geo, trigram, upsert)|`PharmacyGeoRepositoryTest`|
|Web slice|`@WebMvcTest` + `@MockitoBean`|شكل الـ envelope، الـ validation، status codes، الأدوار|`PharmacyControllerTest`|
|Integration|`@SpringBootTest` + Testcontainers (PostGIS + Redis)|الـ flows الكاملة|`AvailabilityFlowIT`|
|Architecture|Modulith `verify()` + ArchUnit|حدود الـ features وقواعد الطبقات|`ModularityTest`|
|Search quality|golden set (CSV)|الأخطاء الإملائية العربي/الإنجليزي|`DrugSearchQualityTest`|
|Performance (اختياري)|k6 أو Gatling|p95 للبحث والتوافر|—|

**قواعد:** أسماء الـ methods `should_X_when_Y` · قاعدة `AbstractIntegrationTest` بـ container singleton (image PostGIS معرّف `asCompatibleSubstituteFor(postgres)` و`@ServiceConnection`) · `Clock.fixed` في الـ tests بتاعة الـ TTL · test data builders بدل الـ setters المتكررة.

**السيناريوهات الإلزامية:**

1. الـ normalizer: أ/إ/آ ← ا · ة ← ه · تشكيل · أرقام هندية.
2. `formulation_key`: نفس المواد بترتيب مختلف ← نفس المفتاح · 5mg ≠ 10mg · g↔mg متطابقين · شراب ≠ أقراص.
3. البدائل بتستبعد المخدر ونفس الدواء.
4. الجغرافيا: داخل/خارج النطاق · ترتيب بالأقرب · غير الموثقة مبتظهرش.
5. الـ TTL: `CONFIRMED` ← `UNCONFIRMED` بعد الانتهاء (Clock ثابت).
6. الاستيراد: نجاح جزئي · كود مجهول/مخدر مرفوض · تكرار = آخر صف · ملف تالف = `FAILED`.
7. دورة الطلب: أقصى 3 مفتوحة · رد مرتين = 409 · `ALTERNATIVE` من غير بديل صحيح مرفوض · أثر الرد على المخزون.
8. الأمان: الموظف مبيلمسش صيدلية غيره (`/me`) · المريض مبيشوفش طلب غيره (404) · الأدوار.
9. كل شكل خطأ بالـ envelope (400/401/403/404/409/413/429/500).
10. الـ rate limit بيرجع 429 مع `Retry-After`.

## 15. خطة التنفيذ (Phases + Definition of Done)

|   |   |   |
|---|---|---|
|المرحلة|الشغل|Definition of Done|
|**P0 — تحضير**|حسم أسئلة قسم 18 · تحديد dataset الأدوية وترخيصه · ERD · repo + README · اسم المشروع|ملف قرارات متكتب + dataset متحدد|
|**P1 — الأساس**|مشروع Boot + base package · `common` كاملة (ApiResponse, PageResponse, exceptions, GlobalExceptionHandler, AuditableEntity, Clock, CommonMapperConfig, ArabicTextNormalizer) · Docker Compose · V1 · profiles · قاعدة Testcontainers · `ModularityTest` · CI|`verify` أخضر في CI · `/actuator/health` شغال|
|**P2 — identity**|V2 · register/login/refresh/logout/me · SecurityConfig + JWT filter · entry point وdenied handler بالـ envelope · admin bootstrap|اختبارات الأدوار تمر · 401/403 بالـ envelope|
|**P3 — catalog**|V3 · `FormulationKeyGenerator` · import job · البحث (Postgres) · details + alternatives · golden-set test|dataset كامل متستورد · بحث بأخطاء إملائية يعدي الـ golden set|
|**P4 — pharmacy** (بالتوازي مع P3)|V4 · create/update/me · admin verification · nearby (PostGIS) · `@EgyptCoordinates`|`EXPLAIN` بيثبت استخدام GiST · غير الموثقة مبتظهرش|
|**P5 — inventory**|V5 · manual PUT · CSV import بـ Batch · تقرير الأخطاء · TTL · template|استيراد 20k صف في زمن مقبول (هدف < 30 ثانية) والفاشلة متسجلة|
|**P6 — availability + notification**|V6/V7 · بحث التوافر + `ConfirmationState` · requests/respond + events + `InAppChannel` · reports · scheduler التنضيف|`AvailabilityFlowIT` الكامل يعدي|
|**P7 — الجودة والتشغيل**|Redis cache للبدائل · rate limiting · metrics · springdoc · README + Postman collection · benchmark أولي · seed للـ demo|تشغيل بأمر واحد (`docker compose up` + run) · أرقام p95 متسجلة|
|**P8 — اختياري**|Elasticsearch (بعد benchmark) · OCR/AI · POS API · Push/WhatsApp · Frontend|بحسب القرار|

**الاعتماديات:** P2 قبل P4 و P5 و P6 · P3 و P4 مستقلين · P5 يحتاج P3 + P4 · P6 يحتاج P3 + P4 + P5.

## 16. الأفخاخ المعروفة (Gotchas)

1. **ترتيب الإحداثيات:** `ST_MakePoint(lng, lat)` والـ JTS `Point` فيه `x = lng` و`y = lat`. أشهر غلطة في المشروع ده.
2. **`geography` مش `geometry`:** المسافات بالمتر. للـ `ST_X/ST_Y` اعمل cast `::geometry`.
3. **`open-in-view=false`:** أي entity بتطلع برّه الـ service هتعمل `LazyInitializationException`. عشان كده الـ mapping لـ DTO بيحصل **جوه** الـ service وجوه الـ transaction.
4. **`ddl-auto=validate`:** أي اختلاف بين entity وcolumn بيوقّع الـ startup. ده المطلوب.
5. **`IDENTITY` ids** بتعطّل JDBC batching في Hibernate — عشان كده الاستيرادات بتستخدم Batch/JDBC writers مش `saveAll`.
6. **الوقت:** `Instant` و`timestamptz` UTC، وممنوع `LocalDateTime` للقيم المخزنة. مصر فيها DST، فتحويل `Africa/Cairo` بيحصل وقت العرض بس.
7. **Boot 4:** Jackson 3 (`tools.jackson.*`) · `@MockitoBean` · أسماء starters جديدة · راجع توافق springdoc وModulith وBucket4j مع الإصدار قبل ما تعتمد.
8. **MapStruct + Lombok:** ترتيب الـ processors مهم، و`unmappedTargetPolicy = ERROR` بيكشف الحقول المنسية وقت الـ compile.
9. **N+1:** استخدم `@EntityGraph` أو projections، والـ fetch الافتراضي LAZY.
10. **Trigram مع العربي:** النصوص القصيرة بتدّي similarity واطي، فظبط الـ threshold بالـ golden set بعد التطبيع. و`unaccent` للاتيني بس — العربي بيعتمد على `ArabicTextNormalizer`.
11. **Native projections:** الـ aliases لازم تطابق أسماء الـ properties (استخدم quoted aliases أو record + constructor mapping لو الـ case اتلخبط).
12. **الـ multipart:** احفظ الملف في مكان مؤقت **قبل** ما ترجع 202، لأن الـ request stream بيقفل. وامسحه بعد الـ job.
13. **Spring Batch:** كل تشغيل بـ `JobParameters` فريدة · `spring.batch.job.enabled=false` · الـ API بتاع التشغيل اتغيّر في Batch 6 فراجعه.
14. **الـ Events:** فشل الـ listener مبيرجعش تلقائيًا إلا لو سجل الـ event publication في Modulith متفعّل. اختبر سيناريو الفشل.
15. **تطبيع الإدخال قبل الـ uniqueness:** email بـ lowercase وphone بـ E.164 قبل أي فحص تكرار.
16. **المخدر في كل مسار:** أي query جديدة على `drug` لازم تفلتر `controlled = false` (BR-S2). اعمل لها test.
17. **`quantity` داخلي:** مفيش DTO بيرجع للمريض فيه `quantity`.
18. **الرسائل:** النصوص العربية في `messages.properties` عبر `MessageSource` مش hard-coded (التزم بيها لو هتدعم لغتين).

## 17. Checklist لأي Feature جديدة

- Migration `V<n>__<feature>_*.sql` (جداول + indexes + constraints)
- Entities (ترث `AuditableEntity`) + enums بـ `STRING`
- Repository: derived أولًا · native معزول وموثّق ومعاه `@DataJpaTest`
- DTOs (records): requests بالـ validation + responses
- Mapper (MapStruct بـ `CommonMapperConfig`)
- Service interface + impl (package-private) + `@Transactional` + الـ BRs
- Exceptions + `<Feature>ErrorCode`
- Controller: `ResponseEntity<ApiResponse<T>>` + `@PreAuthorize` + `@Valid` + springdoc
- Events (لو في تواصل async مع feature تانية)
- `package-info.java` بـ `@ApplicationModule` (+ `@NamedInterface` للـ packages المكشوفة)
- `@ConfigurationProperties` record + مفاتيح yml (لو في إعدادات)
- Tests: unit + repository + web slice + (IT للـ flow)
- `ModularityTest` أخضر
- تحديث الـ OpenAPI/README

## 18. الأسئلة المفتوحة واقتراحاتي

أسئلة v1 + اللي اتضافت. كل صف فيه **اقتراحي الافتراضي** عشان تقدر تبدأ من غير ما تستنى، وتعدّله لما تقرر.

|   |   |   |   |
|---|---|---|---|
|#|السؤال|اقتراحي|بيأثر على|
|18.1|مصدر dataset الأدوية المصرية وترخيصه؟|ابدأ بـ seed تجريبي (200–500 دواء) للتطوير، وبالتوازي دوّر على dataset منشور (هيئة الدواء المصرية أو مصدر مفتوح) وراجع الترخيص والـ attribution قبل أي نشر. أنا مقدرش أأكدلك ترخيص أي dataset.|P3|
|18.2|الإشعارات: WhatsApp / SMS / Push؟|MVP: in-app (polling) + `LoggingChannel`. Push لو في موبايل app. WhatsApp/SMS بتكلّف وتحتاج مزوّد → بعد الـ MVP.|notification|
|18.3|المريض لازم يسجّل؟|لا للبحث والبدائل و`nearby` (Public + rate limit). نعم للـ requests والبلاغات (anti-abuse).|security|
|18.4|الواجهة؟|API + Swagger + Postman collection للـ MVP، وfrontend بعدين لو الوقت سمح.|scope|
|18.5|نخزّن quantity ولا status بس؟|status أساسي، وquantity اختياري وداخلي (BR-I4).|inventory|
|18.6|قيمة الـ TTL؟|csv 36h · manual 12h · api 6h · request-reply 2h · report 3h. كلها في الـ yml. راجعها مع صيدلي حقيقي لو قدرت.|inventory|
|18.7|توثيق الصيدلية؟|يدوي من ADMIN بمراجعة `license_no` (ورفع صورة الترخيص في مرحلة لاحقة).|pharmacy|
|18.8|أول محافظات؟|القاهرة الكبرى (القاهرة + الجيزة) للـ demo والـ seed.|data|
|18.9|اسم المشروع والـ base package؟|placeholder `com.aliragab.pharmacyfinder` لحد ما تختار اسم.|P1|
|18.10|Modulith ولا ArchUnit؟|الاتنين: Modulith للحدود والـ events، وArchUnit لقواعد الطبقات.|architecture|
|18.11|Maven ولا Gradle؟|Maven.|P1|
|18.12|لغة الرسائل؟|عربي في MVP، و`Accept-Language` لاحقًا.|common|

## 19. الـ AI Rules Block والـ Prompt Pack

### 19.1 الـ Rules Block (الصقه في أول أي جلسة مع الـ AI)

```text
You are implementing the Pharmacy Availability Platform backend. Follow these rules strictly.

STACK: Java 21, Spring Boot 4.1.x, Spring Data JPA (Hibernate 7), PostgreSQL + PostGIS + pg_trgm,
Flyway, MapStruct, Lombok (limited), Spring Security + JWT, Spring Batch, Redis, Spring Modulith.

ARCHITECTURE: package-by-feature under one base package.
Features: identity, catalog, pharmacy, inventory, availability, notification, ai (optional).
Shared code lives only in common.
Inside a feature: controller -> service (interface) -> service/impl -> repository -> entity,
plus dto/request, dto/response, mapper, exception, event, config.
DEPENDENCIES: a feature may call another feature ONLY through its service interface and DTO/view
records. Never import another feature's entity, repository, mapper or controller. Link features by
ID only (no cross-feature @ManyToOne). No cycles (use events). common has no business logic and
depends on no feature.

LAYERS:
- Controller: thin; returns ResponseEntity<ApiResponse<T>>; @Valid on bodies; @PreAuthorize for
  roles; no try/catch, no repository access, no @Transactional.
- Service: interface + package-private impl in service/impl; @Service @RequiredArgsConstructor;
  class-level @Transactional(readOnly = true), write methods @Transactional; holds business rules;
  throws feature exceptions; uses the injected Clock; never returns entities.
- Repository: derived queries first; JPQL @Query for > 3 conditions or joins; native SQL ONLY for
  PostGIS, pg_trgm, upsert/bulk, in a dedicated documented repository, bind params, with a
  @DataJpaTest.
- Entity: @Getter @Setter @NoArgsConstructor(access = PROTECTED); never @Data; enums as STRING;
  Instant for time; extends AuditableEntity.
- DTOs: Java records with validation on requests; never expose entities or secrets.
- Mapper: MapStruct with CommonMapperConfig (unmappedTargetPolicy = ERROR); no business logic.

RESPONSES: every endpoint returns ApiResponse<T> (success, code, message, data, errors, timestamp).
Errors go through GlobalExceptionHandler with ErrorCode enums (CommonErrorCode + one per feature).
Never leak stack traces or SQL.

CONVENTIONS: /api/v1/<kebab-plural>; JSON camelCase; DB snake_case singular; Flyway owns the
schema (ddl-auto=validate, open-in-view=false); Instant/timestamptz in UTC; BIGINT identity ids.

SAFETY INVARIANTS: exclude controlled drugs everywhere (BR-S2); only VERIFIED pharmacies are
visible (BR-S6); always return the disclaimer with alternatives (BR-S1); never cache availability;
never log PII; availability state is computed at read time from expires_at.

BOOT 4: Jackson 3 (tools.jackson.*), @MockitoBean, generate dependencies from start.spring.io,
verify third-party library compatibility.

WHEN IN DOUBT: follow the Reference Slice (section 7) and the feature spec (section 8).
Ask before deviating and list every assumption you make.
```

### 19.2 قوالب الـ Prompts

**P1 — تنفيذ feature كاملة:**

```text
Read the Rules Block and Blueprint v2 sections 4.5, 5, 6, 7, 8.<n> and 9 (this feature's tables).
Implement feature <name> end to end: migration, entities, repositories, DTOs, mappers, service
interface + impl, controller, exceptions + error codes, config, and tests (unit, repository, web
slice). Output files grouped by path. Do not modify other features; call them only via their
service interfaces. List assumptions at the end.
```

**P2 — توسيع قسم لخطة تنفيذ (من غير كود):**

```text
Expand section <n> into an implementation plan: class list with responsibilities, method
signatures, call sequence, edge cases, and test cases. Do not write code yet.
```

**P3 — كتابة اختبارات:**

```text
Write tests for <class> covering business rules <BR ids>. Use Testcontainers (PostGIS), a fixed
Clock, AssertJ, and names like should_X_when_Y. Include the mandatory scenarios from section 14
that apply.
```

**P4 — مراجعة كود:**

```text
Review this code against the Rules Block and sections 4.5 and 5. Report violations as a table
(rule, file, fix). Do not rewrite unless I ask.
```

**P5 — Debug:**

```text
Here is the stack trace, the relevant code, and the Boot 4 notes. Find the root cause, propose the
smallest fix, and mention any Boot 3 vs Boot 4 difference that matters.
```

**طريقة الشغل مع الـ AI:** جلسة جديدة لكل feature، وابعت **الـ Rules Block + الأقسام المتعلقة بالـ feature بس** (مش الملف كله) عشان الـ context يفضل نضيف. نفّذ بترتيب الـ phases في قسم 15، وبعد كل feature شغّل `ModularityTest` والـ tests قبل ما تكمّل.

## 20. إزاي تقدّمه في الـ CV والإنترفيو

- **Feature-based modular monolith** والحدود متفحوصة تلقائيًا بـ Modulith في الـ CI — دي بتثبت إنك فاهم الـ boundaries مش بس الـ folders.
- **قرارات مبنية على قياس:** بدأت بـ Postgres (trigram + normalization) ومدخلتش Elasticsearch غير بعد benchmark على golden set.
- **مشكلة تقنية حقيقية:** بحث عربي بهمزات وأخطاء إملائية من غير محرك بحث خارجي.
- **Data trust model:** TTL + `ConfirmationState` + broadcast requests + بلاغات المستخدمين، لأن نجاح المشروع بيعتمد على جودة بيانات التوافر.
- **Spring Batch import** بنجاح جزئي وتقرير للصفوف الفاشلة، و**PostGIS** للبحث الجغرافي بـ GiST.
- **Testcontainers + PostGIS + CI** وAPI موثّقة بـ OpenAPI.
- قول بصراحة إنه **portfolio** مش startup، وإنك عارف المنافسين (شفاء وغيرها).