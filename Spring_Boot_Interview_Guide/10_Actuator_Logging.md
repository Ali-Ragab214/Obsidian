# Section 10: Actuator, Logging & Caching (Redis)

> **الملف ده مبني على مذكراتك في مجلدات `Logging/logback.md`، ومجلد `Redis/` لعمليات الـ Caching مع Spring Boot.**  
> يغطي: مراقبة صحة التطبيق عبر Spring Boot Actuator، مستويات الـ Logging ومعمارية Logback، وتفعيل الـ Caching باستخدام Redis (`@Cacheable`, `@CacheEvict`, `@CachePut`).

---

### Q1: What is Spring Boot Actuator? What are the key production endpoints?
**Level:** Junior / Mid  
**Related to:** Section 02 - Q1 (Production-Ready Features), System Design

**Answer:**
**Spring Boot Actuator** هي مكتبة رسمية توفر مجموعة من واجهات الـ HTTP و JMX الجاهزة لـ **مراقبة وفحص صحة وإدارة التطبيق في بيئة الإنتاج** (Production Observability & Monitoring) بدون الحاجة لكتابة كود يدوي.

#### إضافة التبعية في `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### أشهر الـ Endpoints الأساسية:
1. **`/actuator/health`:**
   - يفحص هل التطبيق يعمل وسليم (`"status": "UP"`).
   - يفحص تلقائياً الاتصال بقاعدة البيانات، والـ Redis، ومساحة الديسك، وخادم البريد.
2. **`/actuator/info`:**
   - يعرض معلومات عامة عن الإصدار، اسم التطبيق، وتفاصيل الـ Git Commit.
3. **`/actuator/metrics`:**
   - يوفر إحصائيات دقيقة جداً: استهلاك الرام (JVM Heap/Non-heap)، عدد الاتصالات المفتوحة في HikariCP، عدد استدعاءات كل REST Endpoint ومتوسط زمن استجابتها.
4. **`/actuator/env`:**
   - يعرض متغيرات البيئة وخصائص الـ Properties المحملة في التطبيق.
5. **`/actuator/loggers`:**
   - يسمح بقراءة وتعديل مستويات الـ Logging وقت التشغيل بدون إعادة تشغيل السيرفر!

#### تأمين الـ Actuator (Security Best Practice ⚠️):
افتراضياً، لحماية البيانات الحساسة، Spring Boot لا يفتح سوى `/health`. لو أردت كشف نقاط أخرى، يجب تحديدها في `application.yml`:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info,metrics" # لا تضع "*" في بيئة الإنتاج أبداً!
  endpoint:
    health:
      show-details: when-authorized # لا تكشف تفاصيل فشل الداتابيز إلا للمسؤولين
```

**Short answer (30 seconds version):**
> "Spring Boot Actuator provides production-ready operational endpoints for monitoring application health (/actuator/health), metrics (JVM and HTTP throughput via /actuator/metrics), and configurations. Sensitive endpoints must be restricted in production via proper Spring Security controls."

---

### Q2: How do you implement a Custom Health Indicator in Spring Boot?
**Level:** Mid  
**Related to:** Q1 (Actuator)

**Answer:**
إذا كان تطبيقك يعتمد على خدمة خارجية حرجة (Third-party Payment API أو File Storage)، يمكنك إنشاء كلاس مخصص ليفحصه Actuator ويحدد بناءً عليه هل التطبيق `UP` أم `DOWN`:

```java
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean isGatewayReachable = checkPaymentGateway();

        if (isGatewayReachable) {
            return Health.up()
                    .withDetail("PaymentService", "Available")
                    .withDetail("ResponseTimeMs", 45)
                    .build();
        } else {
            return Health.down()
                    .withDetail("PaymentService", "Gateway Timeout")
                    .withDetail("Error", "HTTP 503 Connection Refused")
                    .build();
        }
    }

    private boolean checkPaymentGateway() {
        // تنفيذ Ping أو Socket check خفيف
        return true; 
    }
}
```

عند طلب `/actuator/health`، سيظهر الـ Custom Indicator ضمن مكونات النظام:
```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "diskSpace": { "status": "UP" },
    "paymentGateway": {
      "status": "UP",
      "details": {
        "PaymentService": "Available",
        "ResponseTimeMs": 45
      }
    }
  }
}
```

**Short answer (30 seconds version):**
> "You implement a custom health check by creating a Spring bean implementing the HealthIndicator interface and overriding the health() method. Return Health.up() or Health.down() enriched with key-value diagnostic details to seamlessly integrate into /actuator/health."

---

### Q3: Logging in Spring Boot: SLF4J vs Logback and Log Levels.
**Level:** Junior / Mid  
**Related to:** Section 06 (Exception Handling)

**Answer:**
مأخوذة من مذكراتك بملف `Logging/logback.md`:

#### 1. ما الفرق بين SLF4J و Logback؟
- **SLF4J (Simple Logging Facade for Java):**
  - هو مجرد **Logging Abstraction / Interface** (مواصفة).
  - لا يقوم بطباعة الـ Logs بنفسه، بل يوفر واجهة كود موحدة (`log.info("Hello {}", name)`).
  - الفائدة: يسمح بتبديل مكتبة الـ Logging التحتية بدون تغيير سطر واحد في كودك البرمجي.
- **Logback:**
  - هو الـ **Implementation الفعلي** وهو المحرك الافتراضي المدمج في Spring Boot.

#### 2. مستويات الـ Logging (Log Levels) من الأقل للأعلى خطورة:
1. **`TRACE`:** تفاصيل منخفضة المستوى ودقيقة جداً (لتتبع خطوات خوارزمية معينة).
2. **`DEBUG`:** معلومات تفصيلية مفيدة للمطور أثناء التطوير المحلي (مثل استعلامات SQL، باراميترز الميثودز).
3. **`INFO` (الافتراضي في الإنتاج):** أحداث النظام الهامة العادية (مثل: بدء تشغيل السيرفر، تسجيل دخول مستخدم، اكتمال طلب أوردر).
4. **`WARN`:** تحذير من شيء غير متوقع حدث ولكن التطبيق استمر في العمل بنجاح (مثل: قرب امتلاء الذاكرة، محاولة دخول فاشلة).
5. **`ERROR`:** خطأ جسيم منع تنفيذ العملية (مثل: فشل الاتصال بقاعدة البيانات، رمي استثناء غير معالج).

#### ضبط المستويات في `application.yml`:
```yaml
logging:
  level:
    root: INFO # المستوى العام للتطبيق والمكتبات
    com.example.myapp: DEBUG # مستوى كود شركتك الداخلي فقط
    org.hibernate.SQL: DEBUG # طباعة استعلامات الـ SQL في الـ Console
  file:
    name: logs/app.log # حفظ الـ Logs في ملف على السيرفر
```

#### الاستخدام في كود جافا عبر Lombok:
```java
@Service
@Slf4j // يولد Logger logger = LoggerFactory.getLogger(OrderService.class);
public class OrderService {
    public void processOrder(Long id) {
        log.info("Processing order id: {}", id); // استخدام Parameterized logging يوفر استهلاك الذاكرة!
    }
}
```

**Short answer (30 seconds version):**
> "SLF4J is a logging facade decoupling our code from specific log engines, while Logback is the default high-performance logging implementation in Spring Boot. The hierarchy spans TRACE, DEBUG, INFO (production standard), WARN, and ERROR, configured granularly per package."

---

### Q4: Spring Caching with Redis: `@Cacheable`, `@CachePut`, and `@CacheEvict`.
**Level:** Mid / Senior  
**Related to:** System Design Caching

**Answer:**
مأخوذة من مذكراتك بمجلد `Redis/`:

الـ **Cache (الذاكرة المؤقتة)** هو وسيلة لتخزين نتائج الاستعلامات الثقيلة والمتكررة في الذاكرة السريعة (RAM / Redis) بدلاً من استنزاف قاعدة البيانات في كل طلب.

#### 1. تفعيل الـ Caching في Spring Boot:
- إضافة التبعيات:
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-cache</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```
- تفعيل الأنوتيشن في كلاس الإعدادات:
  ```java
  @Configuration
  @EnableCaching // تفعيل محرك الـ Cache في Spring
  public class CacheConfig { }
  ```

#### 2. الأنوتيشنز الثلاثة الأساسية للـ Cache:

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    // 1. @Cacheable: ابحث في الكاش أولاً؛ لو وجدته ارجعه فوراً بدون تنفيذ كود الميثود والـ DB!
    // لو لم تجده في الكاش، نفذ الميثود واحفظ النتيجة في Redis للمرات القادمة.
    @Cacheable(value = "products", key = "#id")
    public ProductDto getProductById(Long id) {
        System.out.println("Fetching from Database... (Slow)");
        return productRepository.findById(id)
                .map(ProductMapper::toDto)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    // 2. @CachePut: ينفذ الميثود دائماً ويحدث القيمة المخزنة في الكاش بالبيانات الجديدة
    @CachePut(value = "products", key = "#dto.id")
    public ProductDto updateProduct(ProductDto dto) {
        Product updated = productRepository.save(ProductMapper.toEntity(dto));
        return ProductMapper.toDto(updated);
    }

    // 3. @CacheEvict: يحذف العنصر من الكاش (لتفادي قراءة بيانات قديمة Stale Data بعد الحذف)
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }

    // حذف كل محتويات كاش المنتجات دفعة واحدة
    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductCache() { }
}
```

| Annotation | هل تنفذ الميثود دائماً؟ | ماذا تفعل بالكاش؟ |
| :--- | :--- | :--- |
| **`@Cacheable`** | لا (تتجاوزها لو وُجدت القيمة في الكاش) | تقرأ من الكاش، أو تضيف إليه إذا كان مفقوداً |
| **`@CachePut`** | ✅ نعم تنفذها دائماً | تحدث القيمة في الكاش بالبيانات الجديدة |
| **`@CacheEvict`** | ✅ نعم | تمسح القيمة من الكاش لإبطالها |

**Short answer (30 seconds version):**
> "@Cacheable intercepts execution to serve cached results if present, populating the cache on a miss. @CachePut always executes the method and updates the cached entry. @CacheEvict purges stale entries from the cache upon resource mutation or deletion."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What is Cache Stampede (Thundering Herd Problem) and how do you mitigate it? (Using distributed locks or probabilistic early expiration).*
2. *What is the difference between Cache-Aside, Read-Through, and Write-Through patterns? (Spring's `@Cacheable` implements Cache-Aside).*
3. *Why must cached Java objects implement `Serializable` when using Redis? (Redis stores binary streams or JSON, requiring serialization).*

---

## 🔗 System Design Connection: Prometheus, Grafana & Distributed Caching

### Real-world System Design Application:
في المعماريات المتقدمة، يتم ربط **Actuator** و **Redis** بمنظومة المراقبة المركزية:

```
 Kubernetes Cluster
  ┌────────────────────────────────────────────────────────┐
  │  Spring Boot Pods                                      │
  │  Actuator /actuator/prometheus ──(Scraped every 15s)──►│ Prometheus (Time-Series DB)
  │                                                        │         │
  │  Writes Cache Items                                    │         ▼
  │         │                                              │      Grafana
  │         ▼                                              │   (Alerts & Dashboards)
  │  Redis Cluster (In-Memory Cache)                       │
  └────────────────────────────────────────────────────────┘
```

**System Design Rules:**
1. **Prometheus Scraping:** Actuator يقوم بعمل Export للمقاييس بصيغة يفهمها Prometheus عبر مكتبة `micrometer-registry-prometheus`، ليعرض في Grafana عدد الـ Requests و الـ CPU Usage ونسبة الـ Error Rate.
2. **Cache Eviction Policies (TTL & LRU):** في Redis، نحدد دائماً **TTL (Time to Live)** لكل مفتاح (مثلاً ساعة واحدة) مع تفعيل سياسة **LRU (Least Recently Used)** لمنع امتلاء الرام وانفجار السيرفر.
