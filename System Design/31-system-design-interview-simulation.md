---
# 🔥 31 — System Design Interview Simulation (Backend — Java / Spring Boot)

> **إزاي تستخدم الملف ده؟**
> ده محاكاة انترفيو كاملة مبنية على أول 9 ملفات في الفولدر.
> كل سؤال جاي من concept أو مثال أو discussion موجود فعلاً في الملفات.
> ذاكر سؤال → جاوب بصوت عالي → قارن إجابتك بالـ Model Answer.

---

# 🎯 1) Interview Overview — نظرة سريعة

**Candidate Profile:**

- Backend Engineer — Java / Spring Boot
- شغال CRUD حقيقي (Controller / Service / Repository)
- مشروع عملي: E-commerce / Dream Shop (Categories → Products → Orders)
- الهدف: يعدي System Design Interview لشركة كبيرة

**Interview Format:**

- 60–90 دقيقة
- 9 أقسام (كل قسم = ملف من الملفات التسعة)
- كل قسم: Easy → Mid → Senior
- أنواع الأسئلة في كل قسم:
  - Conceptual (يعني ايه؟ ليه؟ امتى لأ؟ X vs Y؟)
  - Deep dive / Follow-up (طب لو الترافيك ضرب 10x؟ لو النود وقعت؟)
  - Scenario-based (صمّم / شخص مشكلة حقيقية)
  - Java / Spring Boot (نفّذها إزاي؟ annotation ايه؟ config ايه؟)

---

## 🗺️ Mapping — كل قسم جاي من أنهي ملف؟

| # | Interview Section | Source File / Topic |
|---|---|---|
| 1 | Priorities — أولويات مذاكرة System Design | `0-Priorities.md` |
| 2 | Roadmap — خارطة الطريق من Zero لـ Senior | `1-Road Map.md` |
| 3 | Intro — يعني ايه System Design؟ | `2-Intro.md` |
| 4 | Software Architecture & System Requirements | `3-Lecture 1 — Introduction to Software Architecture & System Requirements.md` |
| 5 | Feature Requirements — جمع المتطلبات صح | `4-Feature Requirements.md` |
| 6 | Use Case vs Feature — الفرق اللي بيوقع ناس كتير | `4.1-Use Case vs Feature.md` |
| 7 | Quality Attributes (NFRs) — الجودة هي اللي بتوقع السيستم | `5-Quality attributes (Non Functional reqs).md` |
| 8 | System Constraints — القيود اللي بتبني عليها | `6-System Constraints.md` |
| 9 | Performance — Latency و Throughput و P99 | `7-Performance.md` |

📌 **ملحوظة:**
كل المصطلحات هنا بنفس أسامي الملفات (Functional, Non-Functional, Actors, Use Cases, User Flows, Events, Trade-offs, Loose Coupling, P95/P99, Degradation Point...).

---

# 📌 القسم 1 — Priorities (من ملف `0-Priorities.md`)

> الملف ده كان List مركزة: Load Balancer — Caching (Redis) — Database Indexing — SQL vs NoSQL — ACID — CAP — Replication & Sharding — Message Queues (Kafka/RabbitMQ) — Distributed Transactions & Saga — Microservices Patterns.
> القسم ده بيثبّت الصورة الكبيرة قبل ما نغوص.

---

### Q1.1 — لو بقولك System Design Priorities ايه اللي تذاكره الأول؟ رتّبهم ليه؟

**Question:**
رتب أولويات مذاكرة System Design من الليست دي وقولي ليه هتبدأ بده؟

**Model Answer:**
هبدأ بـ Load Balancer + Caching + Indexing لأنهم بيحلوا 80% من مشاكل الأداء في أي Monolith، بعدين SQL vs NoSQL + ACID + CAP عشان اختيار الداتا الصح، بعدين Replication & Sharding + Queues + Saga + Microservices لأنهم Distributed Systems بجد.

**Key Points:**

- لازم يذكر إن الترتيب = impact مقابل complexity
- يربط كل item بمشكلة حقيقية (بطء → Cache/Index، ضغط → LB، نمو داتا → Sharding)

**Common Mistakes:**

- يبدأ بـ Saga و Microservices قبل الأساسيات ❌
- يحفظ الليست من غير ما يقول امتى تستخدم كل واحدة

**Level:** Junior

---

### Q1.2 — امتى Load Balancer يبقى أهم من Caching؟

**Question:**
عندك API بطيء. امتى تحط Load Balancer وامتى تحط Redis Cache؟

**Model Answer:**
لو المشكلة Throughput (سيرفر واحد مش ملاحق requests) → Load Balancer + Horizontal Scaling. لو المشكلة Repeated Reads لنفس الداتا (GET /categories) → Redis Cache. غالباً بنعمل الاتنين مع بعض.

**Key Points:**

- يفرق بين throughput problem و repeated-read problem
- يذكر Stateless requirement عشان LB يشتغل صح

**Common Mistakes:**

- فاكر إن LB بيسرّع الـ response نفسه ❌ (هو بيوزع الحمل مش بيقلل زمن DB)

**Level:** Junior

---

### Q1.3 — Database Indexing امتى يبقى الحل وامتى يبقى المشكلة؟

**Question:**
ليه الـ Indexing في أولوياتك؟ وامتى تبعد عنه؟

**Model Answer:**
Index بيسرّع Reads (WHERE / JOIN / ORDER BY) لكن بيبطّأ Writes وبياخد مساحة. لو workload قراءة تقيلة (Products search) → Index. لو Writes تقيلة (Orders عالية) وكتابة كتير → Indexes كتير هتقتلك.

**Key Points:**

- Read-heavy vs Write-heavy trade-off
- لازم يذكر مثال: `CREATE INDEX idx_product_category ON product(category_id)`

**Common Mistakes:**

- يعمل index على كل column ❌

**Level:** Junior

---

### Q1.4 — SQL vs NoSQL — تختار ايه لـ Orders؟

**Question:**
عندك E-commerce فيه Orders وفلوس. SQL ولا NoSQL؟ ليه؟

**Model Answer:**
Orders → SQL (MySQL/PostgreSQL) عشان ACID و Relations و consistency الفلوس مفيهاش هزار. Catalog/views أو Cart sessions ممكن NoSQL (Mongo/Redis) عشان flexible schema وسرعة.

```java
@Entity
public class Order {
  @Id @GeneratedValue
  private Long id;
  @ManyToOne(fetch = FetchType.LAZY)
  private User user;
  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}
```

**Key Points:**

- Money = ACID = SQL
- Catalog / unstructured = NoSQL candidate

**Common Mistakes:**

- يقول Mongo لكل حاجة عشان trendy ❌
- ينسى Relations (1-M / M-M) و Normalization

**Level:** Mid

---

### Q1.5 — ACID امتى تكسرها بإيدك؟

**Question:**
تعرف ACID؟ طب اديني case تكسر فيها ACID عمداً.

**Model Answer:**
ACID = Atomicity, Consistency, Isolation, Durability. بكسرها لما أحتاج Scale عالي (Analytics, Likes, Views) فأقبل Eventual Consistency وأستخدم NoSQL أو Saga بدل Distributed Transaction.

**Key Points:**

- يعرف الحروف الأربعة صح
- يقول trade-off: consistency مقابل availability/scale

**Common Mistakes:**

- فاكر إن أي microservice لازم يبقى ACID عبر السيرفسز ❌

**Level:** Mid

---

### Q1.6 — CAP Theorem ببساطة — هتضحي بإيه؟

**Question:**
اشرح CAP في سطرين وقولي في E-commerce هتختار ايه؟

**Model Answer:**
Consistency + Availability + Partition Tolerance — وقت الـ Partition لازم تختار CP أو AP. فلوس ومخزون → CP (Consistency). Feed و reviews → AP (Availability).

**Key Points:**

- P اجباري في Distributed System
- مثال عملي لكل اختيار

**Common Mistakes:**

- يقول هنحقق التلاتة مع بعض ❌
- يتلخبط بين CAP و ACID

**Level:** Mid

---

### Q1.7 — Replication vs Sharding — ايه الفرق؟

**Question:**
الفرق بين Replication و Sharding؟ وامتى تستخدم كل واحد؟

**Model Answer:**
Replication = نسخ نفس الداتا (لله High Availability + Read Scale). Sharding = تقسيم الداتا (لله Write Scale + حجم كبير). Reads كتير → Replicas. داتا ضخمة / Writes عالية → Shards.

```
Primary (write) → Replica1 (read)
                → Replica2 (read)

Shard1: users 1-1M | Shard2: users 1M-2M
```

**Key Points:**

- Replication ≠ Scaling للكتابة
- Sharding بيدخل complexity (resharding, cross-shard queries)

**Common Mistakes:**

- فاكر إن Replication بيحل مشكلة حجم الداتا ❌

**Level:** Mid

---

### Q1.8 — Kafka ولا RabbitMQ؟

**Question:**
امتى تختار Kafka وامتى RabbitMQ؟

**Model Answer:**
Kafka → High throughput, event log, replay, Orders pipeline. RabbitMQ → Traditional queue, routing معقد, low latency task distribution (notifications, jobs). في مشروعك: Orders → Kafka، Emails → RabbitMQ/BullMQ.

**Key Points:**

- Throughput + retention مقابل routing + simplicity
- يذكر Event-Driven Architecture

**Common Mistakes:**

- يختار من غير ما يسأل عن throughput و retention

**Level:** Mid

---

### Q1.9 — Distributed Transactions & Saga — ليه مش 2PC؟

**Question:**
عندك Order → Payment → Stock في 3 سيرفسز. هتعمل Distributed Transaction إزاي؟

**Model Answer:**
مش هستخدم 2PC (blocking وغالي). هستخدم Saga: Choreography (events) أو Orchestration (Saga orchestrator) مع compensating actions (لو الدفع فشل → رجّع المخزون + Cancel Order).

```
OrderService → PaymentService → StockService
     ↑________ compensate ________|
```

**Key Points:**

- يعرف Compensating Transaction
- يفرق Choreography vs Orchestration

**Common Mistakes:**

- يقترح `@Transactional` عبر microservices ❌

**Level:** Senior

---

### Q1.10 — Microservices Pattern واحد تشرحه بعمق: API Gateway

**Question:**
يعني ايه API Gateway؟ وليه مش نخلي الـ Client يكلم السيرفسز مباشرة؟

**Model Answer:**
بوابة واحدة: Routing, Auth (JWT), Rate Limiting, Aggregation. بيقلل coupling من الـ Client وبيوحد cross-cutting concerns. في Spring Cloud:

```java
// application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
```

**Key Points:**

- Single entry + Auth + Rate limit
- خطر: Single point of failure → لازم HA

**Common Mistakes:**

- يحط فيه Business Logic ❌

**Level:** Mid

---

### Q1.11 — Circuit Breaker و Retry — امتى تستخدمهم؟

**Question:**
Payment Gateway بيقع أحياناً. هتتعامل إزاي؟

**Model Answer:**
Retry مع Exponential Backoff + Jitter للأخطاء المؤقتة، و Circuit Breaker (Resilience4j) عشان مفضلش أخبط في سيرفس واقع. Fallback يرجع response degraded.

```java
@CircuitBreaker(name = "payment", fallbackMethod = "fallback")
@Retry(name = "payment")
public PaymentResponse pay(Order o) { /* call Stripe */ }

public PaymentResponse fallback(Order o, Throwable t) {
  return PaymentResponse.pending();
}
```

**Key Points:**

- Retry للـ transient فقط مش لكل error
- Fallback + metrics

**Common Mistakes:**

- Retry infinite من غير backoff → يموّت السيرفس الواقع ❌

**Level:** Senior

---

### Q1.12 — Scenario: من Monolith لـ Microservices — هتفك ازاي؟

**Question:**
عندك Dream Shop monolith (Categories, Products, Orders). جالك 10x traffic. هتفككه إزاي؟

**Model Answer:**
مش هفكك مرة واحدة. الأول: Scale Monolith (LB + Cache + Index + Read Replicas). بعدين أفصل أول سيرفس تقيلة (Orders → Queue + DB لوحدها)، بعدين Products (Cache-heavy). أخلي Contracts ثابتة وأراقب.

**Key Points:**

- Strangler pattern + Monolith first
- يحدد bounded contexts

**Common Mistakes:**

- يقول هنحول microservices من أول يوم ❌

**Level:** Senior

---

### Q1.13 — Scenario: صمم Product Details Page تستحمل Flash Sale

**Question:**
عندك Flash Sale: مليون user بيفتحوا نفس المنتج. صممها بالـ Priorities اللي فوق.

**Model Answer:**
CDN للصور + Redis Cache للمنتج (TTL قصير) + Read Replicas + Rate Limiting + Queue للـ Orders عشان منحرقش DB. المخزون في Redis decrement atomic وبعدين confirm في DB عبر Saga.

**Key Points:**

- Cache + CDN + Queue + Atomic stock
- يذكر Hot key problem

**Common Mistakes:**

- يخلي كل request يضرب DB ❌

**Level:** Senior

---

### Q1.14 — Spring Boot: اربط Cache + Index + LB في مشروع واحد

**Question:**
وريني عملياً في Spring Boot هتطبق Caching و Indexing إزاي؟

**Model Answer:**
JPA Index عبر `@Index` أو migration، و Cache عبر `spring-boot-starter-data-redis` + `@Cacheable`.

```java
@Table(indexes = @Index(name="idx_product_cat", columnList="category_id"))
@Entity
public class Product { /* ... */ }

@Service
public class CategoryService {
  @Cacheable(value="categories", key="#id")
  public Category getById(Long id) { return repo.findById(id).orElseThrow(); }

  @CacheEvict(value="categories", key="#id")
  public void update(Long id, Category c) { /* ... */ }
}
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```

**Key Points:**

- Cache-Aside + Evict على Write
- Index migration مش بس annotation

**Common Mistakes:**

- ينسى Evict فيبقى Stale Cache ❌
- يكاش كل حاجة من غير TTL

**Level:** Mid

---

### Q1.15 — Deep Dive: لو Redis وقع والـ DB اتخنقت — Plan B ايه؟

**Question:**
افترض Redis وقع وقت الـ Peak. ايه اللي هيحصل؟ وايه الـ Fallback؟

**Model Answer:**
هيحصل Cache Stampede / Thundering Herd على DB → Degradation ثم Crash. الحل: Fallback to DB مع Rate Limit + Request Coalescing + Short TTL in-memory (Caffeine) كـ L1 + Circuit Breaker على Redis client.

```java
Cache<String, Category> local = Caffeine.newBuilder()
  .expireAfterWrite(30, TimeUnit.SECONDS)
  .maximumSize(10_000).build();
```

**Key Points:**

- L1 + L2 cache
- Stampede protection

**Common Mistakes:**

- مفيش fallback plan ❌

**Level:** Senior

---
# 📌 القسم 2 — Roadmap (من ملف `1-Road Map.md`)

> الملف ده مقسّم الرحلة: Basics → DB → Performance → Scalability → Patterns → Messaging → Security → Real Systems.
> و رابطه بمشروعك: Categories → DB ، Products → Cache ، Orders → Queue.
> الأسئلة هنا بتقيس هل فاهم الطريق ولا حافظ كلمات.

---

### Q2.1 — اشرح 3-Tier Architecture على مشروعك

**Question:**
يعني ايه 3-Tier Architecture؟ وطبقها على Categories API عندك.

**Model Answer:**
Client → Controller → Service → DB. الـ Controller بيستقبل HTTP، الـ Service فيه الـ Business Logic، والـ Repository بيكلم الداتا. ده بيفصل المسؤوليات وبيسهل التست.

```java
@RestController @RequestMapping("/api/categories")
public class CategoryController {
  private final CategoryService service;
  @GetMapping public List<Category> all() { return service.findAll(); }
}
```

**Key Points:**

- لازم يذكر فصل المسؤوليات
- يربطها بمشروعه فعلاً

**Common Mistakes:**

- يحط Business Logic في الـ Controller ❌

**Level:** Junior

---

### Q2.2 — Monolith vs Microservices — امتى تختار ايه؟

**Question:**
Startup صغيرة وعايزة MVP في شهرين. Monolith ولا Microservices؟

**Model Answer:**
Monolith. أسرع، أرخص، Deploy واحد، Debugging أسهل. Microservices لما التيم يكبر والـ Domains تبقى واضحة والـ Scale مختلف لكل سيرفس.

| Monolith | Microservices |
|---|---|
| Deploy واحد | Deploy مستقل |
| سهل في الأول | صعب Ops |
| Scale كله مع بعض | Scale لكل سيرفس |

**Key Points:**

- Business Constraint (وقت/فلوس) بيحسم الاختيار
- Monolith first استراتيجية مشهورة

**Common Mistakes:**

- يختار Microservices عشان شكلها جامد ❌

**Level:** Junior

---

### Q2.3 — Client-Server Model — فين الـ Stateless؟

**Question:**
يعني ايه Stateless Server؟ وليه مهم للـ Roadmap؟

**Model Answer:**
السيرفر ميخزنش Session في الميموري، كل request معاه كل اللي محتاجه (JWT مثلاً). ده بيخلي أي سيرفر يخدم أي request → سهل تعمل Horizontal Scaling + LB.

**Key Points:**

- Session in Redis/DB مش في السيرفر
- JWT example

**Common Mistakes:**

- يخزن cart/session في HashMap في السيرفر ❌

**Level:** Junior

---

### Q2.4 — HTTP/HTTPS و REST — صمم صح

**Question:**
صمم REST APIs للـ Products والـ Orders بشكل صح.

**Model Answer:**
Resources بأسماء جمع، HTTP verbs صح، Status codes صح، Pagination.

```
GET    /api/products?category=5&page=0&size=20  → 200
POST   /api/orders                               → 201
GET    /api/orders/{id}                          → 200 / 404
```

```java
@GetMapping("/api/products")
public Page<ProductDto> list(@PageableDefault(size=20) Pageable p) {
  return service.list(p);
}
```

**Key Points:**

- Plural nouns + verbs + codes
- Pagination إجباري لليستات الكبيرة

**Common Mistakes:**

- `GET /getProducts` أو `POST /deleteOrder` ❌

**Level:** Junior

---

### Q2.5 — DNS ببساطة — بيحصل ايه لما تكتب amazon.com؟

**Question:**
احكيلي بيحصل ايه من أول ما أكتب URL لحد ما الـ Response يرجع.

**Model Answer:**
DNS Resolve → TCP + TLS handshake → Request يروح LB → API Gateway → Service → Cache/DB → Response. الـ DNS بيحول الاسم لـ IPى
**Common Mistakes:**

- ينسى DNS و TLS تماماً ❌

**Level:** Junior

---

### Q2.6 — Latency vs Throughput — ايه الفرق؟

**Question:**
الفرق بين Latency و Throughput؟ ومثال من مشروعك؟

**Model Answer:**
Latency = زمن request واحد (ms). Throughput = عدد requests في الثانية (RPS). مثال: `GET /products` بياخد 120ms (latency) والسيرفر بيخدم 500 RPS (throughput).

**Key Points:**

- وحدات القياس مختلفة
- تحسين واحد ممكن يحسّن التاني

**Common Mistakes:**

- يستخدمهم كأنهم نفس الحاجة ❌

**Level:** Junior

---

### Q2.7 — MySQL vs MongoDB — امتى تستخدم كل واحد؟

**Question:**
في مشروع E-commerce عندك: Users/Orders و Product Reviews. هتخزنهم فين؟

**Model Answer:**
Users/Orders → MySQL (Relations + ACID + Joins). Reviews / Product attributes المتغيرة → MongoDB (flexible schema). وممكن Hybrid.

**Key Points:**

- Structured + transactional = SQL
- Semi-structured + evolve fast = NoSQL

**Common Mistakes:**

- يختار على أساس الشهرة مش الـ Use Case ❌

**Level:** Mid

---


### Q2.8 — Indexing و Sharding و Replication — حطهم في جملة واحدة مفيدة

**Question:**
اربط Indexing و Replication و Sharding في سيناريو نمو واحد.

**Model Answer:**
أبدأ بـ Index على `category_id` عشان البحث سريع، لما الـ Reads تزيد أعمل Read Replicas، لما الداتا تبقى تيرا والـ Writes تخنق أعمل Sharding حسب `user_id`.

**Key Points:**

- الترتيب الزمني للنمو
- كل حل لمشكلة مختلفة

**Common Mistakes:**

- يبدأ بـ Sharding من أول يوم ❌

**Level:** Mid

---

### Q2.9 — Normalization — normalized ولا denormalized؟

**Question:**
Normalized schema دايماً صح؟ امتى تعمل Denormalization؟

**Model Answer:**
Normalization تمنع التكرار وتحافظ على Consistency (Orders). Denormalization للقراءة السريعة (Product + Category name مع بعض عشان منعملش Join كل مرة). في الـ Read-heavy بنضحي بشوية تكرار مقابل سرعة.

**Key Points:**

- Write correctness مقابل Read speed
- مثال: تخزين `categoryName` مع Product للعرض

**Common Mistakes:**

- Denormalize من غير ما يفكر في Stale Data ❌

**Level:** Mid

---

### Q2.10 — Caching: Redis و In-Memory — امتى تستخدم ايه؟

**Question:**
الفرق بين Redis و In-Memory Cache (Caffeine)؟ وهتستخدمهم إزاي مع بعض؟

**Model Answer:**
Caffeine (L1) سريع جداً بس Local لكل instance. Redis (L2) مشترك بين الكل. الاستراتيجية: L1 TTL قصير (30s) + L2 TTL أطول (5min) + DB. Categories الثابتة → Redis. Config صغيرة → Caffeine.

```java
@Cacheable(value="products", key="#id")
public ProductDto get(Long id) { return repo.find(id); }
```

**Key Points:**

- L1 + L2 strategy
- TTL و Eviction

**Common Mistakes:**

- يكاش من غير TTL فيتملي الرام ❌

**Level:** Mid

---

### Q2.11 — CDN — للصور والفيديوهات ليه مش S3 مباشرة؟

**Question:**
عندك صور منتجات على S3. ليه محتاج CDN؟

**Model Answer:**
S3 في Region واحد → Latency عالية للبعيد. CDN (CloudFront) بيكاش الصور في Edges قريبة من اليوزر → أسرع وأرخص (تقليل origin hits). الصور لا تتغير كتير فمثالية للكاش.

```
User (Cairo) → Edge Cairo (HIT) ✅
             → S3 Frankfurt (MISS, أول مرة بس)
```

**Key Points:**

- Edge caching + TTL طويل للـ static
- Signed URLs للصور الخاصة

**Common Mistakes:**

- يطلع صور dynamic من CDN من غير invalidation plan ❌

**Level:** Mid

---

### Q2.12 — Vertical vs Horizontal Scaling + Stateless vs Stateful + Auto Scaling

**Question:**
اشرح التلاتة دول مع بعض في سيناريو واحد: ترافيك 10x فجأة.

**Model Answer:**
Vertical = كبّر السيرفر (سريع بس له سقف). Horizontal = زوّد سيرفرات (محتاج Stateless + LB). Auto Scaling = زوّد/قلل أوتوماتيك حسب CPU/RPS. الحل: Stateless services + LB + HPA في k8s.

```yaml
# HPA مثال
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: product-service }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: product-service }
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } }
```

**Key Points:**

- لازم Stateless عشان Horizontal ينفع
- Vertical له limit مادي ومالي

**Common Mistakes:**

- يعمل Horizontal لسيرفس Stateful من غير Sticky/Shared session ❌

**Level:** Senior

---

### Q2.13 — Patterns: API Gateway vs Circuit Breaker vs Rate Limiting vs Retry

**Question:**
اربط الأربعة patterns دول في طلب واحد: `POST /orders`.

**Model Answer:**
Request → API Gateway (Auth + Rate Limit 100/min) → OrderService → (Retry + Circuit Breaker على Payment) → Success/Fallback. كل pattern بيحمي طبقة مختلفة.

**Key Points:**

- Gateway على الـ Edge، CB/Retry بين السيرفسز
- Fallback إجباري مع CB

**Common Mistakes:**

- Retry على كل error حتى `400 Bad Request` ❌

**Level:** Senior

---

### Q2.14 — Messaging: Kafka + Event-Driven على مشروعك

**Question:**
طبق Event-Driven على مشروعك: Order اتعمل. ايه الـ Events والـ Consumers؟

**Model Answer:**
`OrderCreated` → (StockService ينقص، NotificationService يبعت، AnalyticsService يسجل). الـ Producer ميستناش حد → Loose coupling.

```java
@EventListener
public void on(OrderCreatedEvent e) {
  kafkaTemplate.send("order.created", e.orderId().toString(), e);
}
```

**Key Points:**

- Async + Loose coupling + Replay
- Idempotency للـ Consumers (مهم وهنرجعله)

**Common Mistakes:**

- يخلي الـ Order يستنى الـ Email يتبعت synchronously ❌

**Level:** Senior

---

### Q2.15 — Security: JWT و OAuth و Hashing — صمم login صح

**Question:**
صمم Login + Authorization بـ Spring Security: JWT فين؟ OAuth امتى؟ والباسورد يتخزن إزاي؟

**Model Answer:**
Password → BCrypt hash في DB (عمره ما يتخزن plain). Login → Access Token (قصير) + Refresh Token (أطول، في HttpOnly cookie). OAuth2 (Google login) لما مش عايز تدير باسوردات. كل request محمي بـ JWT filter + RBAC.

```java
@Bean
public PasswordEncoder encoder() { return new BCryptPasswordEncoder(); }

http.authorizeHttpRequests(a -> a
  .requestMatchers("/api/admin/**").hasRole("ADMIN")
  .requestMatchers("/api/**").authenticated()
  .anyRequest().permitAll());
```

**Key Points:**

- Hash + Salt (BCrypt) مش Encryption
- Short-lived access + refresh rotation

**Common Mistakes:**

- يخزن JWT في localStorage من غير ما يفكر في XSS أو يخزن باسورد plain ❌
- Access token يعيش شهور

**Level:** Senior

---

# 📌 القسم 3 — Intro (من ملف `2-Intro.md`)

> الملف ده بيجاوب: يعني ايه System Design؟ الفرق بين API وسيستم كامل (Amazon/Instagram)؟ LLD vs HLD؟ وازاي `createCategory()` تتحول لـ Gateway + Service + DB + Cache + Auth + Logging.
> الأسئلة هنا سهلة في الأول وبتتقل بسرعة.

---

### Q3.1 — يعني ايه System Design في سطرين؟

**Question:**
اشرح لحد non-technical يعني ايه System Design.

**Model Answer:**
إزاي نصمم سيستم كامل يشتغل بكفاءة ويتحمل ضغط ويكبر مع الوقت — مش مجرد كود، لكن Architecture والداتا بتمشي إزاي والسيستم هيستحمل كام user.

> **System Design = تصميم السيستم كله مش الكود بس**

**Key Points:**

- لازم يذكر: efficiency + تحمل ضغط + نمو
- يفرق بين method/class و architecture

**Common Mistakes:**

- يشرح Design Patterns ويفتكر كده System Design ❌

**Level:** Junior

---

### Q3.2 — Backend vs System Design — `POST /categories` vs Amazon

**Question:**
عملت `POST /categories` (Controller/Service/Repo). ليه ده مش System Design؟ وامتى يبقى System Design؟

**Model Answer:**
ده LLD / implementation لفيتشر واحدة. يبقى System Design لما أسأل: مليون user؟ DB واحدة ولا Distributed؟ Cache؟ LB؟ Failover؟ Microservices ولا Monolith؟ Amazon = Services + DB + Cache + S3 + CDN + Auth.

**Key Points:**

- Feature مقابل System
- يذكر 2-3 أسئلة System Design على نفس الـ API

**Common Mistakes:**

- فاكر إن CRUD نضيف = System Design ❌

**Level:** Junior

---

### Q3.3 — LLD vs HLD — اديني مثال على نفس الفيتشر

**Question:**
خد فيتشر Likes في Instagram ووريني LLD و HLD لكل واحدة.

**Model Answer:**
LLD: Classes (`Like`, `LikeService`), Design Patterns (Observer للنوتيفيكيشن). HLD: Likes Service + DB (sharded by post) + Redis counter + Queue للـ writes + CDN. LLD اللي بعمله دلوقتي، HLD هو System Design الحقيقي.

| LLD | HLD |
|---|---|
| Classes/Patterns | Services/LB/DB/Cache |
| Single process | Distributed |

**Key Points:**

- مثال concrete لكل نوع
- يعرف مكانه الحالي (LLD) والخطوة الجاية (HLD)

**Common Mistakes:**

- يعكسهم ❌

**Level:** Junior

---

### Q3.4 — SQL ولا NoSQL ولا Distributed؟

**Question:**
امتى تقول Database واحدة كفاية وامتى تقول لازم Distributed؟

**Model Answer:**
واحدة كفاية: ترافيك قليل، داتا صغيرة، joins كتير، budget قليل. Distributed: داتا ضخمة، writes عالية، availability عالية، regions متعددة. العلامات: DB CPU 80%+ باستمرار، حجم > مئات GB، Single point of failure بيخوفك.

**Key Points:**

- يذكر إشارات عملية مش كلام نظري
- يذكر تكلفة التعقيد

**Common Mistakes:**

- يوزع من أول يوم من غير داعي ❌

**Level:** Mid

---

### Q3.5 — لو عندي مليون user — Cache و LB و CDN هيشتغلوا إزاي مع بعض؟

**Question:**
ارسملي request واحد معدي على CDN و LB و Cache و DB.

**Model Answer:**

```
User → CDN (static? HIT → return)
     → LB (اختار server)
     → API (Auth)
     → Redis (HIT? return)
     → DB (MISS → fill cache)
```

الـ CDN للصور، الـ LB للتوزيع، الـ Cache لتقليل ضرب DB.

**Key Points:**

- الترتيب الصح
- HIT/MISS في كل طبقة

**Common Mistakes:**

- يحط الـ LB بعد الـ DB ❌

**Level:** Mid

---

### Q3.6 — Horizontal ولا Vertical؟

**Question:**
سيرفرك جاب آخره. هتكبره (Vertical) ولا تزود سيرفرات (Horizontal)؟

**Model Answer:**
Vertical سريع ومفيهوش تعقيد بس غالي وله سقف و Single point of failure. Horizontal أرخص على المدى الطويل و Fault-tolerant بس محتاج Stateless + LB + Shared state (Redis/DB). للـ 1M user → Horizontal.

**Key Points:**

- Trade-off: simplicity مقابل scalability/availability
- يذكر Stateless شرط

**Common Mistakes:**

- يقول Vertical بيكمل للأبد ❌

**Level:** Mid

---

### Q3.7 — لو Server وقع — Backup و Failover فين؟

**Question:**
لو سيرفر وقع الساعة 3 الفجر، اليوزر هيحس بحاجة؟ ليه؟

**Model Answer:**
لو فيه LB + Health checks + Replicas + DB failover (Primary/Secondary) → لا، الترافيك يتحول تلقائياً. لو Single server + Single DB → أيوة، Downtime كامل. الـ Reliability بتتبني بـ Redundancy في كل طبقة.

**Key Points:**

- Health check + Auto failover
- DB replication مش App بس

**Common Mistakes:**

- يأمن الـ App وينسى الـ DB كـ SPOF ❌

**Level:** Mid

---

### Q3.8 — Instagram HLD — صممها في 5 دقايق

**Question:**
صمم Instagram (Users, Posts, Likes, Feed) High-Level بسرعة.

**Model Answer:**
Users Service + Posts Service + Likes Service + (Postgres للـ metadata + S3 للصور + CDN + Redis للـ feed/counters + Queue لرفع الصور). الـ Upload async: Upload → S3 → Queue (resize) → CDN.

```
App → Gateway → Users/Posts/Likes
              → Postgres / Redis
              → S3 → CDN
```

**Key Points:**

- يفصل Hot storage (S3) عن Metadata (DB)
- Async للصور

**Common Mistakes:**

- يخزن الصور كـ BLOB في DB ❌

**Level:** Mid

---

### Q3.9 — من `createCategory()` لـ System حقيقي

**Question:**
حوّل `createCategory()` لكلام System Design: ايه الـ Components اللي ناقصة؟

**Model Answer:**
`createCategory()` لوحدها method. كسيستم: API Gateway (Auth/Rate limit) → Category Service → (Postgres + Redis invalidate) + Auth check (Admin only) + Logging/Metrics + Validation.

```java
@PreAuthorize("hasRole('ADMIN')")
@CacheEvict(value="categories", allEntries=true)
public Category create(CreateCategoryDto dto) { /* validate + save */ }
```

**Key Points:**

- Auth + Validation + Cache invalidation + Observability
- الفرق بين method و system

**Common Mistakes:**

- ينسى Auth و Logging تماماً ❌

**Level:** Mid

---

### Q3.10 — Microservices ولا Monolith لـ Dream Shop؟

**Question:**
Dream Shop بتاعك: هتعمله Microservices من الأول؟ دافع عن اختيارك.

**Model Answer:**
لأ. هبدأ Monolith modular (Categories/Products/Orders packages مفصولة) + LB + Cache. لما Orders تبقى bottleneck أفصلها الأول. Microservices من الأول = DevOps و Monitoring و Distributed transactions من غير داعي.

**Key Points:**

- Modular monolith كخطوة ذكية
- معيار الفصل: scale مختلف / تيم مختلف / deploy مستقل

**Common Mistakes:**

- Microservices عشان الـ CV ❌

**Level:** Mid

---

### Q3.11 — Reliability: حساب سريع — 99.9% يعني ايه؟

**Question:**
يعني ايه 99.9% availability؟ مسموح تقع قد ايه في الشهر؟

**Model Answer:**
99.9% = حوالي 43 دقيقة downtime في الشهر (~8.7 ساعة في السنة). 99.99% = ~4.3 دقيقة شهرياً. عشان كده كل 9 زيادة بتكلف Redundancy و Multi-AZ.

| Availability | Downtime/شهر |
|---|---|
| 99% | ~7 ساعات |
| 99.9% | ~43 دقيقة |
| 99.99% | ~4.3 دقيقة |

**Key Points:**

- يعرف يحسبها أو حافظ الأرقام المشهورة
- يربطها بالتكلفة

**Common Mistakes:**

- يقول 99% و 99.9% زي بعض ❌

**Level:** Mid

---

### Q3.12 — Scenario: Amazon Homepage وقعت في Black Friday — شخص المشكلة

**Question:**
الـ Homepage بطيئة جداً وقت التخفيضات. هتشخص إزاي؟ وهتحل إزاي؟

**Model Answer:**
أقيس الأول (P95, DB slow queries, Cache hit ratio, CPU). غالباً: Cache misses + DB overload + Hot product. الحل: Pre-warm Cache، Read Replicas، Rate limit + Queue للـ checkout، CDN للـ static، Degrade gracefully (أخفي recommendations التقيلة).

**Key Points:**

- Measure first مش تخمين
- Degradation plan

**Common Mistakes:**

- يزوّد سيرفرات عمياني من غير ما يعرف الـ Bottleneck ❌

**Level:** Senior

---

### Q3.13 — Scenario: صمم URL Shortener صغير بس صح

**Question:**
صمم bit.ly مبسط: Functional و HLD سريع.

**Model Answer:**
`POST /shorten {url}` → `abc123`، `GET /{id}` → redirect 302. DB (SQL) mapping id↔url + Redis للـ hot links + Base62. الـ Read 100x الـ Write فالكاش هنا ذهبي.

```
POST /api/shorten → Service (gen id) → DB + Cache
GET /abc123 → Cache HIT? → 302 else DB
```

**Key Points:**

- Read-heavy → Cache-first
- ID generation (counter + Base62 / hash + collision handling)

**Common Mistakes:**

- ينسى Expiry و Abuse/Spam ❌

**Level:** Senior

---

### Q3.14 — Spring Boot: Category Service كـ HLD مصغر

**Question:**
وريني Category Service في Spring Boot بعيون System Design (مش CRUD بس).

**Model Answer:**

```java
@RestController
@RequestMapping("/api/categories")
public class CategoryController {
  private final CategoryService service;
  @GetMapping("/{id}")
  public CategoryDto get(@PathVariable Long id) { return service.getCached(id); }
}

@Service
public class CategoryService {
  @Cacheable(value="categories", key="#id")
  public CategoryDto getCached(Long id) { return repo.findDto(id); }
}
```

+ Spring Actuator للـ health/metrics + Micrometer + RateLimiter (Resilience4j) + Request ID logging.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

**Key Points:**

- CRUD + Cache + Observability + Resilience
- Actuator إجباري

**Common Mistakes:**

- Controller بيرجع Entity مباشرة من غير DTO ❌

**Level:** Senior

---

### Q3.15 — Follow-up: لو طلب Amazon بقى 10x فجأة — ايه أول 3 حاجات تعملهم؟

**Question:**
الترافيك ضرب 10x فجأة والـ Latency ولعت. أول 3 actions؟

**Model Answer:**
1) احمي الـ DB (زود Cache TTL مؤقتاً + Rate limit + Queue للـ writes). 2) وسّع أفقياً (Auto scale + Read replicas). 3) Degrade غير الأساسي (اقفل recommendations الثقيلة). وبعد الأزمة: Postmortem + Load test.

**Key Points:**

- Protect → Scale → Degrade
- Postmortem culture

**Common Mistakes:**

- يعمل Deploy لفيتشر جديدة وقت الأزمة ❌

**Level:** Senior

---
# 📌 القسم 4 — Software Architecture & System Requirements (من ملف `3-Lecture 1`)

> الملف ده أساس الانترفيو: Why structure matters — تعريف Architecture — SDLC — Small vs Large — أهمية Requirement Gathering — التلات أنواع (FR/NFR/Constraints) — مثال Ride Sharing.
> اللي مش حافظ التلات أنواع بالنص بيقع هنا.

---

### Q4.1 — Why Structure Matters؟

**Question:**
ليه أي سيستم كبير لازم يكون له Structure؟ ايه اللي يحصل من غيره؟

**Model Answer:**
لأن السيستم = Components + Communication + Constraints. من غير Structure → عشوائي، غير قابل للتوسع أو الصيانة، أي تعديل يكسر حاجة تانية.

**Key Points:**

- يذكر التلاتة: components, communication, constraints
- يربطها بالصيانة والتوسع

**Common Mistakes:**

- كلام عام من غير ما يذكر components/communication ❌

**Level:** Junior

---

### Q4.2 — عرّف Software Architecture بطريقة تعجب الـ Interviewer

**Question:**
What is Software Architecture؟

**Model Answer:**
> A high-level description of the system structure, its components, and how they communicate to satisfy requirements and constraints.

الكلمات المفتاحية: High-level (مش كود)، Structure، Components (Services/Modules/Layers)، Communication (APIs/Events)، Requirements & Constraints (سبب التصميم). يعني بنصمم شكل النظام قبل كتابة الكود.

**Key Points:**

- يقول التعريف الإنجليزي + يفكك كلماته
- يوضح إن التصميم قبل الكود

**Common Mistakes:**

- يعرّفها إنها Design Patterns بس ❌

**Level:** Junior

---

### Q4.3 — SDLC High-Level — فين System Design؟

**Question:**
احكي SDLC و System Design قاعد فين فيه؟

**Model Answer:**
```
Design → Implementation → Testing → Deployment
```
الـ Design: فهم متطلبات + اختيار Architecture + تكنولوجيا. Implementation: كود + modules + patterns. Testing: Unit/Integration/Performance. Deployment: نشر + Monitoring + Scaling. الـ System Design هو مرحلة الـ Design.

**Key Points:**

- الترتيب + مثال لكل مرحلة
- Monitoring/Scaling في Deployment

**Common Mistakes:**

- ينسى Testing و Monitoring ❌

**Level:** Junior

---

### Q4.4 — Small Task vs Large Scale System

**Question:**
ايه الفرق بين تاسك صغير و Large Scale System من ناحية المتطلبات؟

**Model Answer:**

| Small Task | Large Scale System |
|---|---|
| Method/Algorithm | Full distributed system |
| Clear requirements | Ambiguous requirements |
| Small scope | Huge scope |
| Low risk | High risk |

الكبير: Big scope + High abstraction + High ambiguity → عشان كده Requirement Gathering حياة أو موت.

**Key Points:**

- الجدول ده بالنص مهم
- يذكر ambiguity

**Common Mistakes:**

- يتعامل مع سيستم كبير كأنه CRUD واضح ❌

**Level:** Junior

---

### Q4.5 — ليه Requirement Gathering أهم مرحلة؟

**Question:**
ليه الغلط في المتطلبات كارثة؟

**Model Answer:**
لأن أي خطأ هنا = بناء system غلط + تكلفة تعديل عالية + مشاكل scalability/performance + تأخير. القاعدة الذهبية:

> Fixing a bug in requirements is 100x cheaper than fixing it after deployment.

**Key Points:**

- يذكر 2-3 مخاطر concrete
- يقول القاعدة الـ 100x

**Common Mistakes:**

- يستعجل الكود قبل ما يفهم المطلوب ❌

**Level:** Junior

---

### Q4.6 — التلات أنواع — قولهم في 30 ثانية

**Question:**
What are the types of system requirements؟

**Model Answer:**

```
1) Functional Requirements
2) Non-Functional Requirements
3) System Constraints
```

> Functional describe what the system does. Non-functional describe how well it performs. Constraints define limitations and boundaries.

**Key Points:**

- يقول التلاتة + جملة لكل واحدة بالإنجليزي
- ده سؤال حفظ بس لازم يتقال صح

**Common Mistakes:**

- يتلخبط بين NFR و Constraints ❌

**Level:** Junior

---

### Q4.7 — Functional Requirements — اديني 5 من مشروعك

**Question:**
اديني Functional Requirements حقيقية من مشروع Services بتاعك.

**Model Answer:**
User can register / login / book a service / pay online / cancel booking. Admin can approve jobs. الـ Functional = Features = what system DOES.

**Key Points:**

- أمثلة بصيغة Actor + Action
- يفرقها عن الجودة

**Common Mistakes:**

- يقول "السيستم سريع" على إنها Functional ❌

**Level:** Junior

---

### Q4.8 — Non-Functional Requirements — اشهر 7

**Question:**
قول أشهر NFRs ومعنى كل واحدة في سطر.

**Model Answer:**

| Attribute | المعنى |
|---|---|
| Scalability | يتحمل ملايين المستخدمين |
| Availability | يشتغل 24/7 |
| Performance | سرعة الاستجابة |
| Security | حماية البيانات |
| Reliability | يقلل الفشل |
| Maintainability | سهل التعديل |
| Usability | سهل الاستخدام |

شركات FAANG بتركز هنا أكتر من الفيتشرز.

**Key Points:**

- يحفظ 5 على الأقل + مثال measurable لكل واحدة
- يعرف إن ده أهم جزء في الانترفيو

**Common Mistakes:**

- يقول كلام عام (سريع وآمن) من غير أرقام ❌

**Level:** Mid

---

### Q4.9 — System Constraints — 3 أمثلة من مشروع التخرج

**Question:**
اديني Constraints حقيقية تقيد تصميمك.

**Model Answer:**
لازم نستخدم AWS (مش حر) + لازم MySQL (تكامل مع legacy) + Deadline 3 شهور + Budget محدود + لازم يشتغل Mobile فقط. الـ Constraints بتأثر مباشرة على الـ Architecture.

**Key Points:**

- مثال Technical + Business + وقت
- يوضح تأثير كل قيد على الاختيار

**Common Mistakes:**

- يعتبر Constraints حاجة سلبية بس ❌ (هي نقطة بداية بتقلل الاختيارات)

**Level:** Mid

---

### Q4.10 — مثال Ride Sharing — فرّق التلاتة

**Question:**
خد Ride Sharing App وفرّق FR و NFR و Constraints.

**Model Answer:**
FR (Features): User requests ride، Driver accepts ride. NFR (Quality): Handle 1M users، Response < 2s، 99.99% availability. Constraints: Must use Google Maps API، Must launch in 3 months.

**Key Points:**

- مثال لكل نوع بأرقام في NFR
- الأرقام إجباري (1M, 2s, 99.99%)

**Common Mistakes:**

- NFR من غير أرقام ❌

**Level:** Mid

---

### Q4.11 — Scenario: عميل بيقول "عايز زي Uber" — هترد بإيه؟

**Question:**
عميل قالك "عايز نسخة زي Uber". أول 10 دقايق في الميتنج هتعمل ايه؟

**Model Answer:**
مش هتكلم تكنولوجيا. هسأل: مين الـ Actors؟ ايه الـ Use Cases الأساسية؟ كام user متوقع؟ Response target؟ Budget و Deadline؟ لازم تستخدم ايه إجباري؟ وبعدين أقسم FR/NFR/Constraints وأتفق على MVP scope.

**Key Points:**

- أسئلة Actors + Scale + Constraints قبل أي architecture
- MVP scoping

**Common Mistakes:**

- يقوله هنستخدم Microservices و Kafka من أول جملة ❌

**Level:** Senior

---

### Q4.12 — Scenario: متطلبات غامضة — هتتعامل إزاي مع Ambiguity؟

**Question:**
المتطلبات ambiguous والعميل نفسه مش عارف عايز ايه. هتتصرف إزاي؟

**Model Answer:**
أحوّل الغموض لـ Use Cases + User Flows + أسئلة measurable. أعمل Prototype/MVP صغير، وأثبّت Assumptions كتابةً، وأراجع كل أسبوع. الغموض بيتكسر بالسيناريوهات الحقيقية مش بالأسئلة العامة.

**Key Points:**

- Assumptions documented
- Iterative clarification

**Common Mistakes:**

- يفترض من دماغه ويكمل ❌

**Level:** Senior

---

### Q4.13 — Trade-off: عايزين كل حاجة — هترد بإيه؟

**Question:**
العميل عايز: سريع + رخيص + يتحمل ملايين + يتسلم في شهر. هترد بإيه؟

**Model Answer:**
مستحيل تحقق كل الـ Quality Attributes بأقصى درجة (No single architecture can provide all). هفهمه الـ Trade-offs: سرعة التسليم = Monolith + فيتشرز أقل. ملايين اليوزرز = وقت وتكلفة. وهخليه يرتّب الأولويات.

**Key Points:**

- No free lunch
- Prioritization مع العميل

**Common Mistakes:**

- يوعده بكل حاجة ❌

**Level:** Senior

---

### Q4.14 — Spring Boot: FR/NFR/Constraints في user story واحدة

**Question:**
حوّل User Story دي لـ FR/NFR/Constraints ووريني أثرها على الكود: "اليوزر يبحث عن منتج ويلاقي نتيجة بسرعة".

**Model Answer:**
FR: Search products. NFR: P95 < 300ms + 1000 RPS. Constraint: Must use MySQL + Redis المتاحين. الأثر: لازم Index على name + Cache للنتائج المشهورة + Pagination.

```java
@Query("SELECT p FROM Product p WHERE p.name LIKE %:q%")
Page<Product> search(@Param("q") String q, Pageable p);

// + @Cacheable للكلمات المشهورة + index على name
```

**Key Points:**

- يفكك الجملة لتلاتة
- يترجمها لقرارات تقنية

**Common Mistakes:**

- يترجم "بسرعة" لكود من غير رقم ❌

**Level:** Senior

---

### Q4.15 — Deep Dive: لو NFRs اتغيرت بعد 6 شهور — التصميم هيستحمل؟

**Question:**
صممت على 10k users، فجأة بقوا 1M. التصميم بتاعك هيستحمل؟ وكنت هتصمم differently إزاي لو كنت عارف؟

**Model Answer:**
لو بنيت Tightly Coupled + Stateful + Single DB → هيقع. لو بنيت Stateless + LB-ready + Cache layer + Read replicas ready → هتوسع بسهولة. الدرس: صمم للـ NFRs المتوقعة + 10x headroom في الـ Critical paths، وخلي الـ Bottlenecks معروفة ومكتوبة.

**Key Points:**

- Headroom planning
- يعرف bottlenecks بتاعته مسبقاً

**Common Mistakes:**

- يقول صممت لـ infinite scale من الأول (over-engineering) ❌

**Level:** Senior

---

# 📌 القسم 5 — Feature Requirements (من ملف `4-Feature Requirements.md`)

> المحاضرة كلها عن: ليه الطريقة التقليدية (اسأل العميل عن كل حاجة) فاشلة؟ والطريقة الأقوى (Use Cases + User Flows) — 3 خطوات: Actors → Use Cases → User Flows (Events = Action + Data) — وبعدين UML/Sequence — وأخيراً الترجمة لـ Endpoints و DB.
> ده اللي بيحوّل الأفكار لـ Backend.

---

### Q5.1 — ليه "قولنا كل اللي انت عايزه" طريقة فاشلة؟

**Question:**
ليه مينفعش نروح للعميل نقوله احكيلنا كل اللي نفسك فيه ونمشي نبرمج؟

**Model Answer:**
لأن العميل مش فاهم تقنياً، مش عارف كل السيناريوهات والاستثناءات، بيفتكر وينسى. النتيجة: نظام مليان مشاكل بعد التسليم. للـ Complex systems الطريقة دي ❌.

**Key Points:**

- يذكر 2-3 أسباب (تقني/سيناريوهات/استثناءات)
- يفرق Complex vs Simple

**Common Mistakes:**

- يلوم العميل بدل ما يغير الطريقة ❌

**Level:** Junior

---

### Q5.2 — Use Cases + User Flows — ايه هما؟

**Question:**
ايه الطريقة الأقوى لجمع المتطلبات؟ اشرحها في دقيقة.

**Model Answer:**
بدل الطلبات العامة → نحوّلها لمواقف حقيقية (Use Cases) ثم نفصّص كل موقف خطوة بخطوة (User Flows). الطريقة تعتمد على حاجتين: 1) Use Cases 2) User Flows.

**Key Points:**

- الفرق بين طلب عام وموقف حقيقي
- يذكر الرقمين

**Common Mistakes:**

- يشرح واحدة وينسى التانية ❌

**Level:** Junior

---

### Q5.3 — يعني ايه Use Case؟

**Question:**
يعني ايه Use Case؟ واديني مثال من تطبيق خدمات.

**Model Answer:**
> Situation / Scenario in which our system is used.

يعني سيناريو حقيقي بيستخدم فيه المستخدم النظام. بدل "عايزين Booking system" → (Customer books service، Worker accepts، Customer cancels، Customer pays). بقينا بنفكر في مواقف مش Features عامة.

**Key Points:**

- التعريف + مثال تحويل من عام لمواقف
- 3 أمثلة على الأقل

**Common Mistakes:**

- يعرّفها إنها زرار أو endpoint ❌

**Level:** Junior

---

### Q5.4 — يعني ايه User Flow؟

**Question:**
يعني ايه User Flow؟ وايه الفرق بينه وبين Use Case؟

**Model Answer:**
> Step-by-step graphical representation of each use case.

Use Case = السيناريو، User Flow = خطوات السيناريو. مثال Book Service: Customer opens app → Select category → Select service → System fetches nearby workers → Customer sends request → Worker accepts/rejects.

**Key Points:**

- Use Case مقابل Flow
- مثال بـ 5+ خطوات مرتبة

**Common Mistakes:**

- يبدلهم ❌

**Level:** Junior

---

### Q5.5 — الخطوة 1: مين الـ Actors؟

**Question:**
في تطبيق خدمات (Customer/Worker/Admin) مين الـ Actors؟ وهل لازم يكون إنسان؟

**Model Answer:**
Customer 👤 + Worker 👷 + Admin 👨‍💼 + Payment Gateway 💳 (أي System خارجي Actor). لأ، مش شرط إنسان — أي external system يعتبر Actor.

> first we need to identify all actors/users in the system

**Key Points:**

- يذكر external system كـ Actor
- 3-4 actors للمثال

**Common Mistakes:**

- ينسى الـ Payment/Maps/Notification services ❌

**Level:** Junior

---

### Q5.6 — الخطوة 2: هات كل الـ Use Cases

**Question:**
هات Use Cases لكل Actor في تطبيق الخدمات.

**Model Answer:**
Customer: Register/Login/Choose service/Send booking/Cancel/Pay. Worker: Register/Accept/Reject/Complete. Admin: Approve workers/Manage categories/Manage users.

> capture and describe all possible use cases

**Key Points:**

- تغطية التلاتة Actors
- صيغة فعل واضح

**Common Mistakes:**

- يدمج أدوار بعض (Admin بيحجز خدمة؟) ❌

**Level:** Junior

---

### Q5.7 — الخطوة 3: فصّص Use Case واحد لـ Flow

**Question:**
خد Book Service وفصّصه User Flow كامل.

**Model Answer:**
1. Customer opens app 2. Select category 3. Select service 4. System fetches nearby workers 5. Customer sends booking request 6. Worker receives notification 7. Worker accepts/rejects. كل خطوة اسمها Event.

> expand each use case through flow of events

**Key Points:**

- ترتيب منطقي + يذكر كلمة Event
- يشمل الـ failure branch (reject)

**Common Mistakes:**

- Flow خطي سعيد فقط من غير reject/cancel ❌

**Level:** Mid

---

### Q5.8 — كل Event = Action + Data — وريني

**Question:**
يعني ايه كل Event فيه Action و Data؟ اديني مثال Backend.

**Model Answer:**
> each event contain: action and data.

مثال: "Customer sends booking request" → Action: Send request، Data: customerId, serviceId, location, time. وهنا بدأنا ندخل Backend 🔥 (الداتا دي هي الـ DTO والـ DB columns).

```java
public record BookingRequest(
  Long customerId, Long serviceId,
  Double lat, Double lng, Instant time
) {}
```

**Key Points:**

- يفصل Action عن Data
- يترجمها لـ DTO

**Common Mistakes:**

- يقول Action بس وينسى الداتا ❌

**Level:** Mid

---

### Q5.9 — Sequence Diagram — ارسم Book Service كلاماً

**Question:**
حوّل الـ Flow ده لـ Sequence Diagram (مين كلم مين وبأي ترتيب؟).

**Model Answer:**
Customer → App → BookingService → Worker (notification) → Payment → App → Customer. الرسم بيبين ترتيب الرسائل والتفاعل ومين بدأ وردود الفعل. مهم للـ Backend Design.

```
Customer -> App: book(serviceId)
App -> BookingService: POST /bookings
BookingService -> WorkerApp: notify()
WorkerApp -> BookingService: accept()
BookingService -> PaymentService: charge()
```

> UML used mostly for software design, used to represent interactions between entities.

**Key Points:**

- الترتيب + الاتجاه
- يذكر فايدته للـ Backend

**Common Mistakes:**

- يرسم Class Diagram ويقول عليه Sequence ❌

**Level:** Mid

---

### Q5.10 — من Flow لـ Endpoints — ترجم

**Question:**
ترجم Use Case (Book Service) لـ APIs حقيقية.

**Model Answer:**
الأفكار → Endpoints، الـ Flows → DB، الـ Events → APIs. مثال: `POST /bookings` + `GET /workers/nearby?lat=&lng=` + `PATCH /bookings/{id}/accept`. كل Event تقريباً endpoint أو message.

**Key Points:**

- 3 endpoints على الأقل بأفعال صح
- يوضح mapping القاعدة

**Common Mistakes:**

- endpoint واحد يعمل كل حاجة ❌

**Level:** Mid

---

### Q5.11 — Scenario: صمم Ride Completion زي Uber

**Question:**
صمم Ride Completion (Customer/App/Driver/Payment) كـ Actors + Flow + Events.

**Model Answer:**
Actors: Customer/App/Driver/Payment. Flow: Customer ends ride → App calculates fare → Payment charges → Driver gets payout → Both get receipt. Events كلها Action+Data (rideId, fare, paymentMethod).

**Key Points:**

- يغطي الـ money path
- يذكر receipt/notification

**Common Mistakes:**

- ينسى الـ failure (الدفع فشل → ايه؟) ❌

**Level:** Senior

---

### Q5.12 — Scenario: عميل نسي Cancel و Refund — هتكتشفها إزاي؟

**Question:**
العميل قال "عايزين Booking" ونسي يقول Cancel و Refund. هتكتشف النواقص دي إزاي بالطريقة دي؟

**Model Answer:**
لما أمشي Actor-by-Actor وأسأل "كل Actor ممكن يعمل ايه؟" + أمشي الـ Happy path وأسأل "و لو لغى؟ و لو الدفع فشل؟ و لو الـ Worker مجاش؟" → النواقص بتظهر لوحدها. الـ Use Cases بتكشف الـ edge cases.

**Key Points:**

- يسأل عن الـ negative paths عمداً
- Checklist: cancel/refund/timeout/no-show

**Common Mistakes:**

- يستنى العميل يفتكر لوحده ❌

**Level:** Senior

---

### Q5.13 — Deep Dive: لو الـ Flow اتغير بعد ما بنيت الـ APIs — هتحمي نفسك إزاي؟

**Question:**
بنيت APIs والعميل غيّر الـ Flow. هتتصرف إزاي من غير ما تهد كل حاجة؟

**Model Answer:**
لو الـ APIs مبنية على Events صغيرة (Accept/Cancel/Pay منفصلة) + Versioning + Loose coupling → التغيير سهل. لو endpoint واحد كبير → كارثة. الدرس: صمم granular + versioned من الأول.

```
PATCH /v1/bookings/{id}/accept
PATCH /v1/bookings/{id}/cancel
```

**Key Points:**

- Granular APIs + Versioning
- Backward compatibility

**Common Mistakes:**

- يكسر contract القديم عادي ❌

**Level:** Senior

---

### Q5.14 — Spring Boot: حوّل Booking Flow لكود حقيقي

**Question:**
حوّل Booking Flow لكود Spring Boot (Controller + DTO + Service).

**Model Answer:**

```java
@RestController @RequestMapping("/api/bookings")
public class BookingController {
  @PostMapping public ResponseEntity<BookingDto> create(@Valid @RequestBody BookingRequest req) {
    return ResponseEntity.status(201).body(service.create(req));
  }
  @PatchMapping("/{id}/accept") public BookingDto accept(@PathVariable Long id) {
    return service.accept(id);
  }
}

@Service @Transactional
public class BookingService {
  public BookingDto create(BookingRequest req) {
    // validate customer + service + nearby workers → save PENDING → publish event
    return mapper.toDto(repo.save(booking));
  }
}
```

**Key Points:**

- DTOs من الـ Events (Action+Data)
- Status codes + Validation + Transaction

**Common Mistakes:**

- Entity في الـ Request مباشرة ❌
- نسي `@Transactional` لعملية multi-step

**Level:** Senior

---

### Q5.15 — Follow-up: من Flow لـ DB Schema — صمم جدول Booking

**Question:**
من الـ Events اللي فوق صمم جدول Bookings (columns + relations).

**Model Answer:**

```java
@Entity
public class Booking {
  @Id @GeneratedValue private Long id;
  @ManyToOne private User customer;
  @ManyToOne private Worker worker;
  @ManyToOne private Service service;
  @Enumerated(EnumType.STRING) private BookingStatus status; // PENDING/ACCEPTED/CANCELLED/DONE
  private Double lat; private Double lng;
  private Instant scheduledAt; private Instant createdAt;
}
```

كل Data في الـ Events بقت column، وكل Actor بقى relation.

**Key Points:**

- Status enum + Timestamps + Relations
- يربط كل column بـ Event

**Common Mistakes:**

- يخزن lat/lng كـ String ❌ أو ينسى الـ status

**Level:** Senior

---

# 📌 القسم 6 — Use Case vs Feature (من ملف `4.1-Use Case vs Feature.md`)

> ملف صغير بس بيوقع ناس كتير: Use Case = عملية/هدف الـ Actor عايز يحققه. Feature = capability موجودة. مثال E-commerce (Place Order) ومثال YouTube. والسلسلة الذهبية: Requirements → Use Cases → APIs → Components → DB → Scaling → Architecture.

---

### Q6.1 — عرّف Use Case في سطر

**Question:**
يعني ايه Use Case في System Design؟

**Model Answer:**
> وصف لعملية أو هدف يريد الـ User أو Actor أن يحققه باستخدام الـ System.

بنسأل: "المستخدم عايز يعمل ايه بالـ system؟" مش "السيستم فيه ايه؟"

**Key Points:**

- وجهة نظر الـ Actor مش السيستم
- هدف/عملية مش زرار

**Common Mistakes:**

- يعرّفها إنها endpoint أو class ❌

**Level:** Junior

---

### Q6.2 — Feature vs Use Case — الفرق في مثال واحد

**Question:**
ايه الفرق بين Feature و Use Case؟ واديني مثال Payment.

**Model Answer:**
Feature = capability موجودة. Use Case = السيناريو/الهدف اللي الـ Actor بيستخدم فيه الـ capability. مثال:

```
Feature: Payment System
Use Cases:
- Customer pays for an order
- Customer gets a refund
- Admin verifies a payment
```

**Key Points:**

- واحدة Feature قصادها 3 Use Cases
- الصياغة Actor + فعل

**Common Mistakes:**

- يستخدمهم كأنهم مترادفات ❌

**Level:** Junior

---

### Q6.3 — E-commerce: هات Use Cases حقيقية

**Question:**
هات Use Cases لـ E-commerce System.

**Model Answer:**
Register، Login، Search Products، Add to Cart، Place Order، Pay، Track Order، Cancel Order. كل واحدة هدف مستقل الـ User عايز يحققه.

**Key Points:**

- 5+ use cases بصيغة أفعال
- تغطي lifecycle كامل (browse → buy → track → cancel)

**Common Mistakes:**

- يقول "Database" أو "API" كـ Use Case ❌

**Level:** Junior

---

### Q6.4 — فصّص Place Order خطوة بخطوة

**Question:**
فصّص Use Case (Place Order) زي ما الملف عمل.

**Model Answer:**

```text
Actor: Customer
Use Case: Place Order
1. Customer chooses products
2. System validates product availability
3. Customer provides shipping address
4. System calculates total price
5. Customer pays
6. System creates the order
7. System sends confirmation
```

**Key Points:**

- الترتيب + تمييز خطوات اليوزر من السيستم
- يشمل validation + confirmation

**Common Mistakes:**

- ينسى validation و confirmation ❌

**Level:** Junior

---

### Q6.5 — YouTube Actors — هاتهم

**Question:**
لو بنصمم YouTube مين الـ Actors؟

**Model Answer:**
User + Creator + Admin (+ Payment/Transcoding Service كـ external actors). كل Actor له أهداف مختلفة تماماً.

**Key Points:**

- يفرق User (يتفرج) عن Creator (يرفع)
- يذكر external system

**Common Mistakes:**

- يدمج User و Creator ❌

**Level:** Junior

---

### Q6.6 — YouTube Use Cases — هاتهم متقسمين

**Question:**
هات Use Cases لكل Actor في YouTube.

**Model Answer:**

```text
User:
  - Sign up / Watch / Search / Like / Subscribe
Creator:
  - Upload / Edit / Delete / View analytics
Admin:
  - Moderate content / Ban user
```

**Key Points:**

- متقسمين حسب Actor
- يغطي CRUD + domain actions (Subscribe/Ban)

**Common Mistakes:**

- Use Cases عامة من غير تقسيم ❌

**Level:** Junior

---

### Q6.7 — ليه بنستخدم Use Cases قبل الـ Architecture؟

**Question:**
ليه بنحدد Use Cases قبل ما نرسم Architecture؟

**Model Answer:**
لأنها بتحدد ايه اللي السيستم المفروض يعمله قبل ما نقرر هيتعمل إزاي. بعدها نسأل: "ايه الـ components/services اللي محتاجها عشان أحقق الـ Use Cases دي؟" فالـ Architecture بتطلع من الاحتياج مش العكس.

**Key Points:**

- What قبل How
- Architecture driven by needs

**Common Mistakes:**

- يرسم architecture وبعدين يدور على use cases ليها ❌

**Level:** Mid

---

### Q6.8 — السلسلة الذهبية — قولها وطبقها

**Question:**
قول السلسلة الذهبية وطبقها على YouTube Upload.

**Model Answer:**
`Requirements → Use Cases → APIs → Components/Services → Database → Scaling → Architecture`. مثال Upload: Requirement (Creator يرفع فيديو) → Use Case (Upload video) → APIs (`POST /videos`, `GET /videos/{id}/status`) → Components (Upload Service + Transcoder + S3 + Queue) → DB (metadata) → Scaling (async + CDN) → Architecture.

**Key Points:**

- الترتيب بالنص
- مثال لكل حلقة

**Common Mistakes:**

- ينط من Use Case لـ DB مباشرة ❌

**Level:** Mid

---

### Q6.9 — Use Case ولا Endpoint؟ — الفرق الحاسم

**Question:**
هل Use Case = Endpoint؟ ليه لأ؟

**Model Answer:**
لأ. Use Case هدف business من وجهة نظر Actor، ممكن يتنفذ بعدة endpoints (Place Order = validate + address + pay + create). والـ Endpoint تفصيلة implementation ممكن تتغير من غير ما الهدف يتغير.

**Key Points:**

- One-to-many (use case → endpoints)
- Business مقابل Technical

**Common Mistakes:**

- يصمم endpoint لكل use case 1:1 عمياني ❌ (أحياناً صح وأحياناً غلط)

**Level:** Mid

---

### Q6.10 — Conceptual: امتى Feature واحدة تطلع 5 Use Cases؟

**Question:**
اديني Feature واحدة وطلع منها 5 Use Cases.

**Model Answer:**
Feature: Cart. Use Cases: Add to cart / Remove / Update quantity / View cart / Checkout from cart / Save for later. كل واحدة هدف مختلف لنفس الـ capability.

**Key Points:**

- يفرّع صح
- كل use case independent goal

**Common Mistakes:**

- يكرر نفس الهدف بصياغات مختلفة ❌

**Level:** Mid

---

### Q6.11 — Scenario: عميل قال "عايز Search" — حوّلها لـ Use Cases

**Question:**
العميل قال "عايز Search حلو". هتحوّلها لإيه؟

**Model Answer:**
مش هاخدها كـ Feature واحدة. هسأل: مين بيبحث؟ (User/Admin) بيدور على ايه؟ (Products/Orders/Users) محتاج filters؟ محتاج autocomplete؟ فتطلع Use Cases: User searches products by name/category، Admin searches orders by status/date، System suggests autocomplete.

**Key Points:**

- يكسر العام لمواقف
- يذكر filters/autocomplete كـ use cases فرعية

**Common Mistakes:**

- يعمل endpoint بحث واحد generic ويفتكر خلص ❌

**Level:** Senior

---

### Q6.12 — Scenario: صمم Subscribe + Notifications كـ Use Cases

**Question:**
صمم Subscribe to channel بالكامل: Actors + Use Cases + APIs + Components.

**Model Answer:**
Actor: User/Creator/System. Use Cases: Subscribe/Unsubscribe/Get notified on new video. APIs: `POST /channels/{id}/subscribe`, `DELETE ...`, `GET /subscriptions`. Components: Subscription Service + Notification Service + Queue (fan-out). DB: subscription table (user_id, channel_id).

**Key Points:**

- يمشي السلسلة كاملة
- يذكر fan-out problem (creator مشهور → ملايين notifications)

**Common Mistakes:**

- Notification synchronous ❌

**Level:** Senior

---

### Q6.13 — Deep Dive: لو Use Case واحد بيحتاج 3 سيرفسز — مين المسؤول؟

**Question:**
Place Order محتاج Order + Payment + Stock. هتقسم المسؤولية إزاي؟

**Model Answer:**
كل سيرفس مسؤول عن domain بتاعه، والتنسيق عبر Saga Orchestrator أو Events. الـ Use Case الواحد مش معناه سيرفس واحد — معناه flow عبر سيرفسز بـ contracts واضحة و compensations.

**Key Points:**

- Bounded contexts
- Orchestration مقابل Choreography

**Common Mistakes:**

- سيرفس واحد يعمل كل حاجة (distributed monolith) ❌

**Level:** Senior

---

### Q6.14 — Spring Boot: نفّذ Place Order كـ Use Case (مش CRUD)

**Question:**
نفّذ Place Order كـ Use Case كامل في Spring Boot (validation + total + payment + confirmation).

**Model Answer:**

```java
@Service @Transactional
public class OrderService {
  public OrderDto placeOrder(PlaceOrderRequest req) {
    var products = validateAvailability(req.items()); // 1
    var total = pricing.calculate(products, req.address()); // 2
    payment.charge(req.paymentToken(), total); // 3
    var order = repo.save(mapper.toEntity(req, total)); // 4
    events.publish(new OrderCreatedEvent(order.getId())); // 5 confirmation async
    return mapper.toDto(order);
  }
}
```

كل خطوة في الـ Use Case بقت سطر/سيرفس.

**Key Points:**

- الترتيب + Transaction boundaries
- Events للـ side effects

**Common Mistakes:**

- كل حاجة في Controller ❌
- ينسى idempotency للدفع (هنتكلم عنها بعدين بس لازم يلمح ليها)

**Level:** Senior

---

### Q6.15 — Follow-up: ازاي Use Cases تكشف الـ Scaling Needs بدري؟

**Question:**
ازاي من الـ Use Cases أعرف هحتاج Scaling فين من قبل ما أبني؟

**Model Answer:**
أبص على frequency و data size لكل Use Case: Watch video (ملايين reads → CDN + Cache) مقابل Upload (قليل writes بس تقيل → Queue + async transcoding). الـ Hot use cases هي اللي بتحدد الـ Architecture والـ Scaling مش العكس.

| Use Case | Pattern | Scaling Need |
|---|---|---|
| Watch | Read-heavy | CDN + Cache |
| Upload | Write-heavy slow | Queue + Workers |
| Search | Compute-heavy | Index + Sharding |

**Key Points:**

- Read/write ratio لكل use case
- Hot path identification

**Common Mistakes:**

- يصمم نفس الـ scaling لكل الـ use cases ❌

**Level:** Senior

---
# 📌 القسم 7 — Quality Attributes / NFRs (من ملف `5-Quality attributes`)

> أهم ملف في التسعة 🔥 الفكرة: مش النظام بيعمل ايه ❌ لكن بيعمله ازاي وبجودة قد ايه ✅ — الأنظمة بتقع عشان الجودة مش الفيتشرز — Architecture بيتبني عشان Quality مش Features — لازم measurable & testable — مفيش architecture بيدي كل حاجة (Trade-offs) — وفي الآخر Feasibility بأنواعها الأربعة.
> أي انترفيو FAANG بيتلعب هنا.

---

### Q7.1 — يعني ايه Quality Attributes؟

**Question:**
يعني ايه Quality Attributes؟ وايه الفرق بينها وبين Functional؟

**Model Answer:**
Functional = سلوك السيستم (User يعمل Login / يبحث / يحجز). Quality (NFR) = جودة تنفيذ الفيتشر (السرعة/الأمان/الاستقرار/قابلية التطوير). مش بيعمل ايه ❌ لكن بيعمله ازاي وبجودة قد ايه ✅.

**Key Points:**

- مثال Functional قصاده NFR لنفس الفيتشر
- يذكر 3-4 attributes

**Common Mistakes:**

- يعرّف NFR إنها فيتشرز إضافية ❌

**Level:** Junior

---

### Q7.2 — ليه الأنظمة بتتفشل؟

**Question:**
سيستم فيه كل الفيتشرز ومع ذلك فشل. ايه الأسباب المشهورة؟

**Model Answer:**
isn't fast enough (بطيء) / doesn't scale (مش بيستحمل) / slow to develop (التطوير صعب) / hard to maintain (الصيانة صعبة) / not secure enough (مخترق). أمثلة: موقع ينهار وقت التخفيضات، أبلكيشن بيفتح ببطء، كل تعديل يبوظ السيستم. ده كله Quality مش Features.

> systems are redesigned not because of functional requirements

**Key Points:**

- يحفظ 4-5 أسباب + مثال واقعي لكل واحدة
- الجملة المحفوظة فوق

**Common Mistakes:**

- يفتكر الـ redesign بيحصل عشان فيتشر ناقصة ❌

**Level:** Junior

---

### Q7.3 — Architecture بيتبني عشان ايه؟

**Question:**
ليه بنقول Architecture بيتبني عشان Quality مش Features؟

**Model Answer:**
لأن الـ Login ممكن يتعمل بـ 100 طريقة، لكن لو عايزه سريع وآمن و Scalable هتختار Architecture مختلفة تماماً (Cache + Hashing + Stateless + LB). الفيتشر واحدة والجودة هي اللي بتحدد الشكل.

> they have direct correlation with the architecture of our system

**Key Points:**

- مثال واحد (Login/Search) يتنفذ بطريقتين حسب الجودة
- العلاقة المباشرة بين quality و architecture

**Common Mistakes:**

- يصمم architecture من الفيتشرز بس ❌

**Level:** Junior

---

### Q7.4 — فكك الجملة دي: "when user clicks search … within 100ms"

**Question:**
في الجملة دي فين الـ Functional وفين الـ Non-Functional؟

**Model Answer:**
"when user clicks search" → Functional ✔️ (السلوك). "within 100ms" → Non-Functional / Performance ✔️. الجملة فيها الاتنين، والـ 100ms هي اللي هتحدد Cache/Index/Async.

> within at most 100 milliseconds → performance

**Key Points:**

- يفكك الجملة صح
- يترجم الرقم لقرار تقني

**Common Mistakes:**

- يقول الجملة كلها Functional ❌

**Level:** Junior

---

### Q7.5 — Availability بالأرقام — 99% vs 99.9% vs 99.99%

**Question:**
يعني ايه "online store must be available 99.9% of times"؟

**Model Answer:**
يعني يفضل شغال 99.9% من الوقت (≈43 دقيقة downtime شهرياً). 99% = 7 ساعات، 99.99% = 4 دقايق. كل 9 زيادة = Redundancy و Multi-AZ وتكلفة. الـ Availability لازم رقم مش "شغال دايماً وخلاص".

**Key Points:**

- الأرقام + التكلفة
- يذكر Redundancy كحل

**Common Mistakes:**

- "100% availability" ❌ (مفيش حاجة كده)

**Level:** Junior

---

### Q7.6 — Deployability — "team can deploy twice a week" يعني ايه؟

**Question:**
ليه "الفريق يقدر يعمل deploy مرتين في الأسبوع" يعتبر Quality Attribute؟

**Model Answer:**
دي Deployability = سرعة نشر التحديثات. لو السيستم صعب يتنشر (Monolith ضخم، تست بطيء، deploy يدوي) → Architecture سيئة حتى لو شغال. الحل: CI/CD + Automated tests + Independent deploys.

**Key Points:**

- يربطها بالـ Architecture (coupling يبطئ النشر)
- يذكر CI/CD

**Common Mistakes:**

- يفتكرها مشكلة process بس ملهاش علاقة بالتصميم ❌

**Level:** Mid

---

### Q7.7 — لازم تكون measurable & testable — حوّل الكلام العام لأرقام

**Question:**
حوّل دول لأرقام تتقاس: "سيستم سريع / آمن / مستقر".

**Model Answer:**
❌ سريع → ✅ P95 < 200ms. ❌ آمن → ✅ 2FA + encryption + OWASP compliance + لا تخزين plain. ❌ مستقر → ✅ uptime 99.9% + error rate < 0.1%.

> must be measurable & testable 🏆

**Key Points:**

- كل صفة برقم + أداة قياس
- 3 تحويلات صح

**Common Mistakes:**

- يفضل يتكلم عام من غير أرقام ❌

**Level:** Mid

---

### Q7.8 — مفيش Architecture مثالي — اشرح Trade-offs

**Question:**
ليه مستحيل نحقق كل الـ Quality Attributes بأقصى درجة؟

**Model Answer:**
> no single software architecture can provide all quality attributes.

مثال: Secure جداً (Encryption في كل حاجة) → أبطأ 🐢. Fast جداً ⚡ → ممكن تقلل security/checks. لازم Trade-offs — قلب الـ System Design.

**Key Points:**

- مثال Security vs Performance
- يذكر كلمة Trade-off صراحة

**Common Mistakes:**

- يوعد بكل حاجة ❌

**Level:** Mid

---

### Q7.9 — Trade-off عملي: Consistency vs Latency في المخزون

**Question:**
عايز مخزون دقيق 100% وكمان سريع جداً. هتختار ايه؟

**Model Answer:**
مفيش الاتنين مع بعض للأبد. مخزون الفلوس → Consistency (Strong checks + DB lock → أبطأ شوية). عداد المشاهدات → Latency (Redis incr → eventual). للـ E-commerce: حجز المخزون synchronous، وعرض "المتبقي" cached بثواني.

**Key Points:**

- يفرق critical (money) عن non-critical (views)
- حل hybrid

**Common Mistakes:**

- Strong consistency لكل حاجة فيقتل الأداء ❌

**Level:** Senior

---

### Q7.10 — Feasibility — يعني ايه؟ وليه قبل ما نوافق؟

**Question:**
يعني ايه Feasibility؟ وليه لازم ندرسها قبل ما نقول "هنعمل السيستم"؟

**Model Answer:**
> make sure system capable of delivering what client asks.

هل ينفع أصلاً نعمله (تقنياً/مالياً/زمنياً/تشغيلياً)؟ قبل ما توعد لازم تتأكد إن الوعد ممكن.

**Key Points:**

- التعريف + مثال "وعد مستحيل"
- 4 الأنواع (السؤال الجاي)

**Common Mistakes:**

- يوافق على أي طلب من غير دراسة ❌

**Level:** Mid

---

### Q7.11 — Technical Feasibility — مثال 10 مليون من أول يوم

**Question:**
عميل عايز تطبيق يخدم 10 مليون من أول يوم. هترد بإيه تقنياً؟

**Model Answer:**
أسأل: عندك Cloud؟ LB؟ Budget سيرفرات؟ خبرة distributed؟ لو لأ → غير قابل تقنياً بالشكل ده. البديل: MVP لـ 100k + Architecture تتوسع (Stateless + Cache + Read replicas ready).

**Key Points:**

- Checklist تقني
- بديل متدرج

**Common Mistakes:**

- يقول "هنعملها" من غير ما يسأل عن الإمكانيات ❌

**Level:** Mid

---

### Q7.12 — Financial + Time Feasibility — مثالين يحفظوك

**Question:**
اديني مثال Financial ومثال Time يخلوا السيستم غير قابل للتنفيذ.

**Model Answer:**
Financial: النظام يكلف 1M$ والعميل معاه 50k 😅 → نقلل scope (Monolith + Single region). Time: "عايز نسخة زي Uber خلال شهر" 🚨 مستحيل → MVP (Ride request + accept فقط) ثم مراحل.

**Key Points:**

- رقمين concrete
- بديل لكل حالة

**Common Mistakes:**

- يضغط التيم عشان يلحق المستحيل فيطلع سيستم ميت ❌

**Level:** Mid

---

### Q7.13 — Operational Feasibility — السيستم الممتاز اللي محدش عرف يستخدمه

**Question:**
يعني ايه Operational Feasibility؟ واديني مثال فشل.

**Model Answer:**
هل الناس هتعرف تستخدمه؟ نظام معقد لموظفين غير تقنيين → هيفشل حتى لو ممتاز تقنياً. الحل: Usability + Training + Pilot. الجودة تشمل سهولة التشغيل مش الكود بس.

**Key Points:**

- مثال domain معقد (مخازن/مستشفى)
- حلول غير تقنية (تدريب/تبسيط)

**Common Mistakes:**

- يهتم بالتقنية وينسى المستخدم ❌

**Level:** Mid

---

### Q7.14 — Scenario: اكتب NFRs حقيقية لمشروع التخرج

**Question:**
اكتب 6 NFRs measurable لمشروع Services بتاعك.

**Model Answer:**
Search P95 < 300ms / Booking success 99.9% / Support 50k concurrent / Deploy twice/week / Passwords BCrypt + HTTPS everywhere / Booking flow usable في 3 خطوات. كل واحدة برقم وتتقاس.

**Key Points:**

- 6 بأرقام + أداة قياس (Prometheus/Load test/OWASP)
- تغطي performance/availability/security/deployability/usability

**Common Mistakes:**

- NFRs عامة من غير أرقام ❌

**Level:** Senior

---

### Q7.15 — Spring Boot: NFRs بتتحقق إزاي في الكود والـ Infra؟

**Question:**
وريني NFR واحد بيتحقق إزاي عملياً في Spring Boot.

**Model Answer:**
مثال Performance (P95 < 200ms لـ GET /products): Index + `@Cacheable` + Pagination + Micrometer metrics + Load test (k6). ومثال Security: BCrypt + HTTPS + Rate limit.

```java
@Timed(value="products.search")
@Cacheable(value="search", key="#q + '-' + #page")
public Page<ProductDto> search(String q, Pageable page) { /* indexed query */ }
```

```yaml
management:
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true
```

**Key Points:**

- NFR → code + config + measurement
- Timer + percentiles

**Common Mistakes:**

- يحقق NFR بالكلام من غير metric ❌

**Level:** Senior

---

# 📌 القسم 8 — System Constraints (من ملف `6-System Constraints`)

> الجزء اللي بييجي بعد FR و NFR: Constraint = قرار اتاخد قبلك بيقيد حريتك. هي pillars (أساسات) + نقطة بداية مش عائق. الأنواع: Technical / Business / Legal. وبعدها أهم ربط: لو متقيد بتكنولوجيا → متعملهاش Tightly Coupled → اعمل Loose Coupling (PaymentGateway Interface مثال).
> ده سؤال Senior مفضل.

---

### Q8.1 — يعني ايه System Constraint؟

**Question:**
يعني ايه System Constraint؟

**Model Answer:**
> A system constraint is a decision already made for us restricting our freedom.

قيود = حاجات انت مش حر تختارها، قرارات اتاخدت قبل التصميم. مثال: "عايزين السيستم بـ .NET" → خلاص مينفعش تقول Node.js ❌.

**Key Points:**

- التعريف الإنجليزي + مثال
- الفرق بين اختيار وقيد

**Common Mistakes:**

- يتعامل مع القيد كأنه اقتراح ❌

**Level:** Junior

---

### Q8.2 — ليه Constraints هي Pillars؟

**Question:**
ليه بنقول Constraints هي pillars للـ Architecture؟

**Model Answer:**
> System constraints are pillars for software architecture.

النظام كله بيتبني فوقها زي أساسات المبنى. لو تجاهلتها → التصميم كله غلط. وبدل ما تشوفها قيود شوفها نقطة بداية — بتقلل 100 اختيار لـ 3 وبتخليك تبدأ أسرع.

**Key Points:**

- أساسات + تجاهلها = redesign
- الجانب الإيجابي (تقليل الاختيارات)

**Common Mistakes:**

- يصمم وبعدين يكتشف القيد ❌

**Level:** Junior

---

### Q8.3 — التلات أنواع — قولهم بأمثلة

**Question:**
ايه أنواع Constraints؟ واديني مثال لكل نوع.

**Model Answer:**
1) Technical (لغة/DB/Hardware مفروضة) 2) Business (Budget/Deadline/Users) 3) Legal (GDPR/Banking). مثال: لازم Java + Oracle (T) + Budget 5k$ (B) + بيانات أوروبا في أوروبا (L).

**Key Points:**

- التلاتة + مثال لكل واحدة
- يحفظهم بالترتيب ده

**Common Mistakes:**

- ينسى Legal تماماً ❌

**Level:** Junior

---

### Q8.4 — Technical: Legacy Constraint

**Question:**
شركة عندها Java + Oracle قديم وعايزة خدمة جديدة. ينفع تستخدم MongoDB؟

**Model Answer:**
غالباً لأ — لازم تتكامل مع القديم (Legacy constraint). الحل: تبني Service جديدة تتكلم مع Legacy عبر APIs/Events، ولو محتاج Mongo لسبب قوي → اعمله isolated + Anti-corruption layer.

**Key Points:**

- Integration constraint
- Anti-corruption layer

**Common Mistakes:**

- يفرض تكنولوجيا جديدة تكسر القديم ❌

**Level:** Mid

---

### Q8.5 — Technical: Cloud Provider + Offline

**Question:**
عميل قال "AWS فقط" + "الأب لازم يشتغل Offline". ايه تأثير ده على تصميمك؟

**Model Answer:**
AWS فقط → S3 بدل GCS، RDS/Dynamo بدل غيرهم. Offline → Local DB (SQLite/Room) + Sync strategy (last-write-wins أو CRDT) + Queue للعمليات لحد ما النت يرجع. القيدين بيغيروا DB و Architecture تماماً.

**Key Points:**

- كل قيد وأثره
- Sync strategy للـ offline

**Common Mistakes:**

- يصمم online-only ويتجاهل القيد ❌

**Level:** Mid

---

### Q8.6 — Business: Budget محدود (5000$)

**Question:**
Startup معاها 5000$ Cloud بس. هتصمم زي Netflix (Multi-Region)؟

**Model Answer:**
طبعاً لأ. Business Constraint يحكم: Monolith + Cheap hosting + Single region + Minimal infra + Managed services (RDS بدل self-hosted). الـ Fancy architecture تموّت الشركة قبل ما تبدأ.

**Key Points:**

- Cost-driven design
- Managed > self-hosted للصغير

**Common Mistakes:**

- Over-engineering على حساب البقاء ❌

**Level:** Mid

---

### Q8.7 — Business: Deadline 3 شهور — Monolith أولاً

**Question:**
العميل عايز MVP بعد 3 شهور. Microservices ولا Monolith؟

**Model Answer:**
Monolith أولاً → بعدين نفككه. Microservices محتاج وقت + DevOps + Monitoring. الـ MVP: أهم Use Cases بس + Deploy واحد سريع. وده بيتقال في كل كورسات System Design.

**Key Points:**

- Time-to-market يحسم
- Strangler later

**Common Mistakes:**

- يبدأ distributed فيزنق نفسه ❌

**Level:** Mid

---

### Q8.8 — Business: "لازم يدعم مليون من أول يوم"

**Question:**
القيد ده (1M من أول يوم) هيغيّر ايه في تصميمك؟

**Model Answer:**
هيغيّر Scalability و Infra: Stateless + LB + Cache + Read replicas + Load testing قبل اللانش + Auto scaling. وممكن يغيّر Budget نفسه (لازم فلوس لكده). القيد ده بيحوّل NFR لالتزام تعاقدي.

**Key Points:**

- يترجمه لـ components + tests
- يربطه بالـ Budget

**Common Mistakes:**

- ياخده كلام تسويقي من غير ما يصمم له ❌

**Level:** Mid

---

### Q8.9 — Legal: GDPR و Banking

**Question:**
اديني مثالين Legal يفرضوا Architecture.

**Model Answer:**
GDPR: بيانات الأوروبي في أوروبا → AWS EU Region + Data residency + Right to delete. Banking: Encryption قوي + Audit logs + Retention سنوات. القانون بيفرض Region و Storage و Logging.

**Key Points:**

- Region + Delete + Audit
- Retention policy

**Common Mistakes:**

- يخزن كل حاجة في region واحدة رخيصة ❌

**Level:** Mid

---

### Q8.10 — Tightly Coupled يعني ايه؟

**Question:**
يعني ايه Tightly Coupled؟ واديني مثال سيء.

**Model Answer:**
مربوط بتكنولوجيا واحدة جامد — لو غيرتها السيستم ينهار. ❌ مثال: الكود كله MySQL queries مخصوصة، لو نقلنا PostgreSQL → كارثة.

**Key Points:**

- تعريف + مثال + نتيجة التغيير
- يذكر vendor-specific code

**Common Mistakes:**

- يفتكر الـ coupling مشكلة كود بس ❌ (هي architecture)

**Level:** Mid

---

### Q8.11 — Loose Coupling — الحل الذهبي

**Question:**
ايه الحل؟ واشرح مثال Stripe الشهير.

**Model Answer:**
نحط Layer abstraction. ❌ غلط: `PaymentService → Stripe مباشرة`. ✅ صح:

```
PaymentService → PaymentGateway Interface → Stripe
                                               → PayPal
```

لو غيرنا Stripe → نغير implementation فقط. وده اسمه Loose Coupling.

> If limited to database or third-party service, don't tightly couple system to it. 💎

**Key Points:**

- الجملة الذهبية + الرسم + Interface
- فايدة تغيير المزود

**Common Mistakes:**

- Interface من غير ما يفصل الـ vendor code فعلاً ❌

**Level:** Mid

---

### Q8.12 — ليه Loose Coupling مهم مع Constraints؟

**Question:**
ليه الكلام عن Loose Coupling جاي بعد Constraints مباشرة؟

**Model Answer:**
لأن القيود ممكن تتغير: اليوم AWS+MySQL+Stripe، بكرة Azure+PostgreSQL+PayPal 😅. لو tightly coupled → كارثة. لو loosely coupled → تغيير بسيط. المرونة بتتبني وقت القيد مش بعده.

**Key Points:**

- مثال اليوم/بكرة
- Change cost

**Common Mistakes:**

- يبني على القيد الحالي كأنه أبدي ❌

**Level:** Senior

---

### Q8.13 — Scenario: غيّروا DB من MySQL لـ PostgreSQL فجأة — هتنجو؟

**Question:**
بعد سنة العميل قال هننقل PostgreSQL. تصميمك هينجو؟ وكنت هتعمل ايه من الأول؟

**Model Answer:**
لو استخدمت JPA/JPQL + Repository abstraction + Flyway migrations + مفيش native queries كتير → النقل أيام. لو native MySQL في كل حتة → شهور. كنت هعمل: Repository interfaces + تجنب vendor-specific + اختبارات على الاتنين في CI.

```java
public interface PaymentGateway { PaymentResult charge(Money m); }
// StripePaymentGateway implements PaymentGateway
// PayPalPaymentGateway implements PaymentGateway
```

**Key Points:**

- Abstraction + Migration strategy
- Test على الاتنين

**Common Mistakes:**

- `SELECT ... LIMIT` بلهجة vendor في كل الكود ❌

**Level:** Senior

---

### Q8.14 — Spring Boot: Loose Coupling عملي (JPA + Stripe + Config)

**Question:**
وريني Loose Coupling في Spring Boot: DB + Payment + Config.

**Model Answer:**

```java
public interface NotificationSender { void send(String to, String msg); }

@Component("sms")
public class SmsSender implements NotificationSender { /* ... */ }

@Service
public class BookingService {
  private final NotificationSender sender;
  public BookingService(@Qualifier("sms") NotificationSender s) { this.sender = s; }
  // تغيير المزود = تغيير Qualifier/Config بس
}
```

```yaml
app:
  payment:
    provider: stripe # غيّرها لـ paypal من غير كود
```

+ JPA (مش native) + Profiles (`dev/prod`) + Externalized config.

**Key Points:**

- Interface + DI + Config
- Profiles

**Common Mistakes:**

- `@Autowired` لـ implementation مباشرة ❌
- Secrets في الكود

**Level:** Senior

---

### Q8.15 — Follow-up:abstraction زيادة عن اللزوم؟ امتى Loose Coupling يبقى Over-engineering؟

**Question:**
امتى الـ abstraction يبقى غلط؟

**Model Answer:**
لو التكنولوجيا مستقرة ومفيش احتمال تغيير (Postgres لسنين) + التيم صغير → طبقة زيادة = تعقيد وتكلفة من غير فايدة. القاعدة: اعمل abstraction على الحدود المتغيرة (Payment/SMS/Cloud storage) مش على كل سطر. YAGNI + Bounded abstraction.

**Key Points:**

- يعرف امتى يقول لأ
- حدود متغيرة فقط

**Common Mistakes:**

- Interface لكل class عمياني ❌

**Level:** Senior

---

# 📌 القسم 9 — Performance (من ملف `7-Performance.md`)

> ملف الأداء: Response Time (Latency) = Processing + Waiting — Throughput — العلاقة بينهم — Average كذاب و Max مش عملي والحل Percentiles (P50/P95/P99) — Histogram و Long Tail — وبعدين Backend عملي: Measure First + قلل Waiting (DB/Index/Pagination/N+1 + Redis + Async/BullMQ + Pooling) + Scalability — وأخيراً Degradation Point + مثال HikariCP.
> ده أكتر قسم فيه أرقام.

---

### Q9.1 — يعني ايه Performance؟ ايه أهم metric؟

**Question:**
لما نقول سيستم سريع ولا بطيء بنقيس ايه؟

**Model Answer:**
حاجتين: 1) Response Time (Latency) → الوقت اللي اليوزر بيستناه 2) Throughput → السيستم يخدم كام request في الثانية. الاتنين أهم metrics في أي System Design.

**Key Points:**

- التعريفين + الوحدات (ms و RPS)
- مثال لكل واحدة

**Common Mistakes:**

- يقيس CPU بس ويفتكر كده performance ❌

**Level:** Junior

---

### Q9.2 — Latency = Processing + Waiting — اشرح

**Question:**
فكك الـ Response Time لحتتين واديني مثال.

**Model Answer:**
Processing (المفيد: Validate + Query + Logic + Build response) + Waiting (الخطير: مستني DB/thread/network/queue). مثال `GET /doctors`: Processing 50ms + DB waiting 300ms + Network 100ms = 450ms. المعالجة 50 بس 😅 والباقي انتظار.

> أغلب latency مش Processing بل Waiting.

**Key Points:**

- الحتتين + مثال بالأرقام
- Waiting أكبر سبب

**Common Mistakes:**

- يحسّن الـ logic ويسيب الـ DB waiting ❌

**Level:** Junior

---

### Q9.3 — القاعدة الذهبية: قلل WAITING مش PROCESSING

**Question:**
لو عايز تسرّع سيستم هتبدأ منين؟

**Model Answer:**
> لو عايز تسرّع السيستم → قلل WAITING مش PROCESSING.

وده سبب وجود Cache و Queues و Async و Indexing. الـ Processing غالباً صغير، الـ Waiting هو الوحش.

**Key Points:**

- يذكر 3 حلول للـ waiting
- مثال: Cache بدل DB كل مرة

**Common Mistakes:**

- Micro-optimizations في الكود ويسيب N+1 ❌

**Level:** Junior

---

### Q9.4 — Throughput — احسبها

**Question:**
سيرفر عنده 10 threads وكل request بياخد 1 ثانية. الـ Throughput كام؟ ولو قللنا latency للنص؟

**Model Answer:**
10 req/sec. لو latency بقت 0.5s → Throughput = 20 req/sec 🔥. لو Response time يطول → Throughput يقل، لأن السيرفر بيقعد أطول مع كل request.

**Key Points:**

- الحسبة + العلاقة العكسية
- يذكر threads كعامل

**Common Mistakes:**

- يفتكر Throughput ثابت ملوش علاقة بالـ latency ❌

**Level:** Junior

---

### Q9.5 — Average كذاب — اثبت بالأرقام

**Question:**
ليه Average مقياس سيء للأداء؟

**Model Answer:**
مثال: `100ms, 120ms, 110ms, 90ms, 5s 😱` → المتوسط ≈1.1s لكن أغلب اليوزرز بياخدوا 100ms! المتوسط كذاب 😅 لأن outlier واحد بيبوظه.

```
Response time = 450ms (مثال الملف)
```

**Key Points:**

- مثال بالأرقام
- Outlier effect

**Common Mistakes:**

- يعتمد على Average في الـ SLA ❌

**Level:** Junior

---

### Q9.6 — Max مش عملي — ليه؟ والحل ايه؟

**Question:**
طب ليه منستخدمش Max (أسوأ حالة)؟

**Model Answer:**
أي request بطيء واحد يخرب الأرقام ومش عملي. الحل: Percentiles — P50 (median: 50% أسرع من كده) + P95 (95% أسرع) + P99 (99% أسرع). الشركات الكبيرة بتستخدمهم.

**Key Points:**

- يعرف التلاتة
- مثال: `P95 = 300ms` يعني 95% أقل من 300ms

**Common Mistakes:**

- يتلخبط بين P95 و Average ❌

**Level:** Mid

---

### Q9.7 — Histogram و Long Tail — اشرح الرسم بكلام

**Question:**
الرسم (Histogram) بيقول ايه؟ ويعني ايه Long Tail؟

**Model Answer:**
معظم الطلبات 100-200ms، شوية 500ms، قليل جداً 2s. الطلبات البطيئة دي = Long Tail 🐍 — أخطر حاجة لأن اليوزر اللي بياخد 2s هيشتكي 😅 حتى لو المتوسط حلو.

**Key Points:**

- يوصف التوزيع
- خطورة الـ tail

**Common Mistakes:**

- يتجاهل الـ tail عشان "نسبة صغيرة" ❌

**Level:** Mid

---

### Q9.8 — ليه P95/P99 أهم من المتوسط؟ احسبها على مليون

**Question:**
ليه الشركات بتركز على P95/P99؟

**Model Answer:**
لأن اليوزرز مش Average. لو عندك 1M: الـ 1% = 10,000 يوزر 😳 متضايقين لو P99 بطيء. الـ P99 البطيء = آلاف الشكاوى حتى لو المتوسط ممتاز.

**Key Points:**

- الحسبة (1% من مليون)
- User impact

**Common Mistakes:**

- "P99 حلو بس مش مهم" ❌

**Level:** Mid

---

### Q9.9 — Measure First — هتقيس إزاي قبل ما تحسّن؟

**Question:**
قبل ما تحسّن الأداء هتعمل ايه؟ وبإيه؟

**Model Answer:**
لازم أقيس الأول: Logging response time + Monitoring (Prometheus/Grafana) + APM. في Nest: Interceptor يحسب زمن كل request. في Spring: Actuator + Micrometer + Timed.

```java
@Timed(value="bookings.create")
public BookingDto create(...) { /* ... */ }
```

**Key Points:**

- لا تحسين من غير قياس
- 2 أدوات على الأقل

**Common Mistakes:**

- يحسّن بالتخمين ❌

**Level:** Mid

---

### Q9.10 — قلل Waiting (DB): Indexes + Pagination + N+1

**Question:**
اديني 3 حلول DB للأداء ووريني N+1 بتتحل إزاي.

**Model Answer:**
Indexes + Pagination (متجيبش 100k row مرة واحدة) + تجنب N+1 (بدل 1 + N queries → Join/Fetch). مثال N+1: تجيب orders ثم لكل order تجيب items (101 query!) → حلها `JOIN FETCH` أو `@EntityGraph`.

```java
// ❌ N+1
orders.forEach(o -> o.getItems().size());
// ✅
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.user.id = :uid")
List<Order> findWithItems(@Param("uid") Long uid);
```

**Key Points:**

- 3 حلول + كود N+1
- Pagination إجباري

**Common Mistakes:**

- `findAll()` على جدول ضخم ❌

**Level:** Mid

---

### Q9.11 — قلل Waiting (Cache + Async): `GET /categories` و Emails

**Question:**
`GET /categories` بيضرب DB كل مرة، والـ Emails بتأخر الـ Response. الحل؟

**Model Answer:**
Categories ثابتة → Redis (`Cache frequently requested data`) مش DB كل مرة. Emails/Notifications/Image processing → Queue (BullMQ/Kafka) async بدل ما اليوزر يستنى.

```
GET /categories → Redis HIT ✅ (TTL 5min)
POST /orders → save + publish email job → return 201 فوراً
```

**Key Points:**

- Cache للـ static reads
- Async للحاجات غير الفورية

**Common Mistakes:**

- يكاش الداتا المتغيرة بسرعة من غير invalidation ❌

**Level:** Mid

---

### Q9.12 — Connection Pooling — ليه منفتحش connection لكل request؟

**Question:**
ليه Connection Pooling مهم؟ وايه علاقته بالأداء؟

**Model Answer:**
فتح connection جديد غالي (TCP + Auth). الـ Pool (HikariCP) بيحافظ على connections جاهزة → Waiting أقل. لو الـ Pool صغير والترافيك كبير → Requests تستنى → Degradation.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
```

**Key Points:**

- Cost of new connections
- Pool sizing

**Common Mistakes:**

- Pool ضخم جداً فيموّت الـ DB ❌

**Level:** Mid

---

### Q9.13 — Degradation Point — يعني ايه؟

**Question:**
يعني ايه Degradation Point؟ واديني مثال.

**Model Answer:**
النقطة اللي لما النظام يوصلها يبدأ الأداء يتراجع بشكل ملحوظ — لسه شغال بس ببطء أو errors جزئية (قبل الـ crash الكامل). مثال: API شغال تمام على 1000 RPS، عند 5000 يبدأ response يتأخر و errors تزيد.

**Key Points:**

- التعريف + مثال برقم
- الفرق بين degradation و crash

**Common Mistakes:**

- يفتكر النظام يا شغال يا واقع مفيش وسط ❌

**Level:** Senior

---

### Q9.14 — أسباب الـ Degradation الأربعة + مثال HikariCP

**Question:**
ايه أسباب الـ Degradation؟ واشرح مثال HikariCP المشهور.

**Model Answer:**
CPU/Memory للـ limit / DB connections اتخنقت / Thread pool اتملى / Cache miss زادت. مثال الملف: HikariCP فيه 10 connections وجالك 50 request → الباقي يستنى → Degradation. الحل: Pool sizing صح + Queue + Scale + Cache.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      connection-timeout: 2000
```

**Key Points:**

- 3 أسباب على الأقل + المثال
- حل لكل سبب

**Common Mistakes:**

- يزوّد Pool من غير ما يبص على DB capacity ❌

**Level:** Senior

---

### Q9.15 — Scenario: P99 ولّع فجأة — شخص وحل (Spring Boot)

**Question:**
P50 تمام (120ms) بس P99 ولّع (3s). هتشخص إزاي؟ وهتحل إزاي في Spring Boot؟

**Model Answer:**
P50 حلو + P99 وحش = Long Tail (GC pauses؟ Slow queries لقليل من اليوزرز؟ Downstream بطيء؟ Cold cache؟). أشوف: Slow query log + APM traces للـ slow requests + GC logs + Cache hit ratio للـ tail. الحل: Index للـ slow queries + Timeout + Async للـ downstream + Warm cache + Bulkhead (Resilience4j) عشان البطيء ميخنقش الكل.

```java
@Bulkhead(name="search", type=Bulkhead.Type.SEMAPHORE)
@TimeLimiter(name="search")
public CompletableFuture<List<Product>> search(String q) { /* ... */ }
```

**Key Points:**

- يفرق Tail عن general slowness
- Traces مش averages

**Common Mistakes:**

- يبص على Average ويقول "كله تمام" ❌

**Level:** Senior

---
# 🎯 Full System Design Round — 4 مسائل كاملة

> المسائل دي بتجمع كل اللي فوق: FR/NFR/Constraints + Actors/Use Cases + Quality + Performance.
> امشي بالـ 8 خطوات بالترتيب زي أي انترفيو حقيقي.
> الدياجرامات بنفس ستايل الملفات (Text arrows + boxes) عشان تبقى شبههم.

---

## 🏗️ Problem 1 — URL Shortener (زي bit.ly)

### 1) Requirements

**Functional (Features):**

- User shortens long URL → gets short link
- User opens short link → redirect (302)
- Creator views clicks count (basic analytics)
- Admin deletes abusive links

**Actors / Use Cases:**

```text
Actors: User, Creator, Admin
Use Cases:
  User: shorten URL, open short link
  Creator: view stats
  Admin: ban abusive URL
```

**Non-Functional (Quality — measurable ✔️):**

- 100M URLs/month writes، Reads 10x Writes (Read-heavy)
- Redirect P95 < 100ms
- Availability 99.9%
- Durability: لا نضيع mapping أبداً

**Constraints:**

- Must use MySQL المتاحة + Redis
- Budget صغير → Single region الأول
- Legal: منع spam/abuse

---

### 2) Capacity Estimation

- Writes: 100M/month ≈ 40/sec average، Peak 200/sec
- Reads: 10x → 400/sec avg، Peak 2000/sec
- Storage 5 سنين: 6B URLs × 500 bytes ≈ 3TB (DB) — سهلة
- Bandwidth: reads صغيرة (302) → قليلة

📌 يعني: Read-heavy → Cache-first design.

---

### 3) API Design

```
POST /api/v1/shorten { "url": "https://...", "ttlDays": 30 } → 201 { "short": "abc123" }
GET  /{id} → 302 Location: longURL
GET  /api/v1/stats/{id} → { clicks: 1234 }
DELETE /api/v1/admin/urls/{id} → 204 (ADMIN only)
```

```java
@PostMapping("/api/v1/shorten")
public ShortenResponse shorten(@Valid @RequestBody ShortenRequest r) { /* ... */ }

@GetMapping("/{id}")
public void redirect(@PathVariable String id, HttpServletResponse res) throws IOException {
  String longUrl = service.resolve(id); // cache first
  res.setStatus(302); res.setHeader("Location", longUrl);
}
```

---

### 4) Database & Schema

```java
@Entity @Table(indexes = @Index(name="idx_short", columnList="shortId", unique=true))
public class UrlMapping {
  @Id @GeneratedValue private Long id;
  @Column(unique=true) private String shortId; // Base62 7 chars
  @Column(length=2048) private String longUrl;
  private Instant createdAt; private Instant expiresAt;
  private Long clicks;
}
```

- ID gen: Counter (Redis INCR) + Base62 → قصير و unique من غير collision checks كتير
- البديل: Hash (MD5 أول 7) + collision handling — أبطأ

---

### 5) High-Level Architecture

```
User → CDN? (لا، dynamic) → LB → Gateway (Rate Limit 100/min + Auth)
     → Shortener Service → Redis (hot links, TTL) → MySQL (mapping)
     → Kafka (click events) → Analytics Worker → Stats DB
```

- Write path: `POST → gen ID → save DB → fill cache`
- Read path: `GET → Redis HIT? return : DB → fill cache → 302`

---

### 6) Deep Dive

- **Hot links:** 1% من اللينكات عليها 90% الترافيك → Redis + L1 Caffeine + Pre-warm
- **ID collision:** Counter+Base62 مفيش collision أصلاً. لو Hash → check + retry
- **Abuse:** Rate limit + URL blacklist + Expiry

---

### 7) Bottlenecks & Solutions

| Bottleneck | الحل |
|---|---|
| DB read overload | Redis 90%+ hit ratio + Read replicas |
| Counter SPOF | Redis cluster + fallback range allocator |
| Click analytics تخنق Write | Kafka async (مستنيش اليوزر) |

---

### 8) Trade-offs

- Counter (مرتب وقصير) مقابل Random (موزع بس collision) → اخترنا Counter
- Strong consistency للـ mapping مقابل Eventual للـ clicks → clicks تتأخر ثواني عادي
- 302 (count كل مرة + تسمح بتغيير) مقابل 301 (cacheable أكتر بس صعب تغيّر) → 302

---

## 🏗️ Problem 2 — Rate Limiter (100 req/min لكل يوزر)

### 1) Requirements

**Functional:**

- User يقدر يعمل N requests في window
- System يرفض الزيادة بـ `429 Too Many Requests` + `Retry-After`
- Admin يغيّر limits لكل endpoint

**Use Cases:**

```text
Actor: User (calls API), Admin (configures limits)
  User: call API within limit / gets rejected over limit
  Admin: set rule (100/min for /orders)
```

**NFR:**

- Decision P99 < 5ms (على الـ hot path!)
- Availability 99.99% (لو وقع → يا نعدي يا نمنع؟ لازم قرار)
- Accuracy: لا false reject كتير

**Constraints:**

- Must work مع Horizontal Scaling (مش in-memory لوكال بس)
- Redis متاح

---

### 2) Capacity

- 10k RPS × check واحد Redis = 10k ops/sec — Redis single node يشيلها مستريح
- Memory: 1M users × 100 bytes ≈ 100MB — صغيرة

---

### 3) API Design

- مش API لوحده — Middleware/Filter + Headers:

```
GET /api/orders → 200 + X-RateLimit-Remaining: 42
                → 429 + Retry-After: 12 (لو خلص)
```

```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {
  protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain c)
    throws IOException, ServletException {
    if (!limiter.allow(req.getHeader("X-User-Id"), req.getRequestURI())) {
      res.setStatus(429); res.setHeader("Retry-After", "60"); return;
    }
    c.doFilter(req, res);
  }
}
```

---

### 4) DB/Schema (Redis)

- Token Bucket أو Sliding Window في Redis (Lua script atomic):

```
Key: rl:{user}:{endpoint}:{minute} → counter + expire 60s
```

- Rules في MySQL: `(endpoint, limit, window)` → تكاش في Redis

---

### 5) Architecture

```
User → LB → Gateway (RateLimitFilter → Redis Lua) → Service
                                          ↓ 429
                                        Return
```

---

### 6) Deep Dive — Algorithm

| Algorithm | Pros | Cons |
|---|---|---|
| Fixed Window | سهل | Burst على الحدود |
| Sliding Window | عادل | أغلى شوية |
| Token Bucket | يسمح بburst محسوب | أعقد |

✅ هنستخدم Sliding Window (Lua atomic) للـ fairness.

```lua
-- pseudo
local c = redis.call('INCR', KEYS[1])
if c == 1 then redis.call('EXPIRE', KEYS[1], 60) end
return c <= tonumber(ARGV[1])
```

---

### 7) Bottlenecks

- Redis SPOF → Cluster + Fail-open vs Fail-closed قرار: للـ checkout (fail-closed آمن)، للـ feed (fail-open متاح)
- Hot user واحد → Sharding حسب user_id

---

### 8) Trade-offs

- Precision (Sliding) مقابل Cost (Fixed) → Sliding للمدفوع، Fixed للداخلي
- Fail-open (Availability) مقابل Fail-closed (Security) → حسب الـ endpoint

---

## 🏗️ Problem 3 — Notification Service (Email/SMS/Push)

### 1) Requirements

**Functional:**

- System يبعت notification (order confirm / OTP / promo)
- User يختار قنواته (email/push)
- Admin يشوف delivery status

**Actors/Use Cases:**

```text
Actors: OrderService (producer), User (receiver), Admin
  OrderService: enqueue notification
  User: receive + opt-out
  Admin: view status / retry failed
```

Flow:

```
OrderCreated → Enqueue → Worker sends → User receives → Status tracked
```

كل Event = Action + Data: `{ userId, channel, template, payload, idempotencyKey }`

**NFR:**

- OTP P95 < 5s، Promo ممكن دقايق (priorities)
- Durability: منضيعش OTP
- Throughput: 5k/sec peak (promo)

**Constraints:**

- SMS provider واحد إجباري (Technical) + Budget يمنع Multi-provider غالي + GDPR (opt-in)

---

### 2) Capacity

- 10M/day ≈ 120/sec avg، Peak 5k/sec (promo blast)
- Retention 90 يوم للـ status ≈ مئات GB — Partition حسب date

---

### 3) API

```
POST /api/v1/notifications { userId, channel, template, data, idempotencyKey } → 202 { id }
GET  /api/v1/notifications/{id} → { status: QUEUED/SENT/FAILED }
POST /api/v1/users/{id}/preferences { email: true, sms: false }
```

Idempotency إجباري (نفس key مرتين = نفس الرسالة مرة واحدة).

---

### 4) DB

```java
@Entity
public class Notification {
  @Id private String id; // UUID
  private Long userId; private String channel; private String template;
  @Enumerated(EnumType.STRING) private NStatus status;
  @Column(unique=true) private String idempotencyKey;
  private int attempts; private Instant createdAt;
}
```

- Queue: Kafka topics حسب priority (`notify.otp` vs `notify.promo`)

---

### 5) Architecture

```
OrderService → Gateway → Notify API → Kafka (otp/promo topics)
                                        → Workers → SMS/Email providers
                                        → Status DB + Retry Queue (DLQ)
```

---

### 6) Deep Dive

- **Priority queues:** OTP topic بworkers أكتر + timeout صغير
- **Retry:** Exponential backoff (1s, 5s, 30s) + DLQ بعد 5 محاولات
- **Loose coupling:** `NotificationSender` interface → SMS/Email/Push implementations (نفس مثال Stripe في الملف!)

```java
public interface NotificationSender { void send(Notification n); }
```

---

### 7) Bottlenecks

- Provider بطيء → Bulkhead + Timeout + Circuit Breaker (Resilience4j) + Fallback channel (SMS فشل → Push)
- Promo blast يموّت OTP → عزل topics/workers + Rate limit للـ promo

---

### 8) Trade-offs

- At-least-once (ممكن duplicate → لازم idempotent consumer) مقابل At-most-once (ممكن ضياع) → At-least-once + dedupe
- Sync (سهل بس بطيء) مقابل Async (سريع بس معقد) → Async + status API

---

## 🏗️ Problem 4 — E-commerce Orders (Dream Shop — مشروعك 👈)

### 1) Requirements

**Functional (Use Cases):**

```text
Actors: Customer, Worker/Admin, Payment Gateway 💳
  Customer: search products / add to cart / place order / pay / track / cancel
  Admin: manage categories / approve / refund
  Payment: charge / refund webhook
```

**NFR (measurable):**

- Search P95 < 300ms، Place order P95 < 800ms
- Availability 99.9% (checkout)، Consistency قوية للفلوس والمخزون
- 50k concurrent وقت Sale

**Constraints:**

- Must: MySQL + Redis المتاحين + AWS region واحدة (Budget) + 3 شهور MVP
- Legal: تخزين بيانات الدفع ممنوع (PCI → Stripe tokens بس)

---

### 2) Capacity

- 1M products، 100k orders/day (≈1.2/sec avg، Peak 100/sec sale)
- Reads 20x Writes على Catalog → Cache-first
- Storage: orders 100k/day × 1KB × 3 سنين ≈ 100GB — معقول

---

### 3) API

```
GET  /api/v1/products?category=5&page=0&size=20 → 200
POST /api/v1/cart/items { productId, qty } → 201
POST /api/v1/orders { cartId, addressId, paymentToken, idempotencyKey } → 201
GET  /api/v1/orders/{id} → 200 (tracking)
PATCH /api/v1/orders/{id}/cancel → 200
POST /api/v1/payments/webhook → 200 (Stripe callback)
```

Idempotency key إجباري لمنع double charge (F5 مرتين!).

---

### 4) DB & Schema

```java
@Entity public class Product {
  @Id @GeneratedValue private Long id;
  private String name; private BigDecimal price; private int stock;
  @ManyToOne(fetch=LAZY) private Category category;
}
@Entity public class Order {
  @Id @GeneratedValue private Long id;
  @ManyToOne private User user;
  @OneToMany(cascade=ALL) private List<OrderItem> items;
  @Enumerated(EnumType.STRING) private OrderStatus status;
  @Column(unique=true) private String idempotencyKey;
  private BigDecimal total; private Instant createdAt;
}
```

- Index: `idx_product_cat`, `idx_order_user`, unique على idempotency
- Stock: Redis atomic decrement للـ hot sale + confirm في DB (Saga)

---

### 5) Architecture

```
User → CDN (صور) → LB → Gateway (Auth JWT + RateLimit)
     → Catalog Service → Redis + MySQL + Search index
     → Order Service → MySQL + Redis stock + Kafka (OrderCreated)
     → Payment (Stripe via Gateway Interface) + Stock + Notify consumers
```

- Upload صور → S3 → CDN (نفس Instagram pattern في الملف)
- Monolith modular الأول → افصل Orders لما تخنق (Strangler)

---

### 6) Deep Dive — Checkout Saga

```
Order (PENDING) → Payment charge → Stock reserve → Confirm
      ↓ fail              ↓ fail          ↓ fail
   Cancelled ←————— compensate ——————————┘
```

- Orchestration (Saga orchestrator) عشان خطوات الفلوس حساسة
- Webhook من Stripe → verify signature → update order (idempotent!)

```java
@Service @Transactional
public class CheckoutService {
  public OrderDto checkout(CheckoutRequest r) {
    if (repo.existsByIdempotencyKey(r.key())) return existing(r.key()); // dedupe
    // reserve stock (Redis) → charge (Stripe) → save order → publish event
  }
}
```

---

### 7) Bottlenecks

| Bottleneck | الحل |
|---|---|
| Hot product cache stampede | Pre-warm + L1/L2 + Request coalescing |
| Stock oversell | Redis Lua atomic decrement |
| DB writes peak | Kafka buffer + Async confirm + Read replicas |
| HikariCP exhaustion (10 → 50 concurrent) | Pool sizing + Timeout + Bulkhead |

---

### 8) Trade-offs

- Monolith (سرعة MVP) مقابل Microservices (scale مستقل) → Monolith modular + فصل تدريجي
- Strong consistency (orders/money) مقابل Eventual (views/reviews) → Hybrid
- Sync checkout (بسيط) مقابل Async (يتحمل peak) → Sync للـ MVP + Queue للـ sale
- SQL (orders) مقابل NoSQL (reviews flexible) → Hybrid حسب الملف

---
# 💬 Behavioral & Experience Questions — 9 أسئلة (STAR)

> الـ Interviewer هنا عايز يسمع قرارات حقيقية: Trade-off عملته، Incident عيشته، Constraint غيّر تصميمك.
> جاوب STAR: Situation → Task → Action → Result.

---

### B1 — احكي عن قرار architecture أخدته وطلع صح (أو غلط)

**Question:**
احكي عن قرار تصميم مهم أخدته في مشروعك.

**Model Answer (STAR):**
**S:** Dream Shop كان Monolith و `GET /products` بيضرب DB كل مرة والـ P95 وصل 900ms.
**T:** كان لازم أنزل تحت 300ms من غير ما أغيّر DB (Constraint).
**A:** حطيت Redis `@Cacheable` + Index على `category_id` + Pagination 20.
**R:** P95 نزل لـ 140ms والـ DB CPU نزل من 85% لـ 30%. اتعلمت: Measure → Cache → Index قبل أي حل كبير.

**Key Points:** رقم قبل/بعد + Constraint احترمته
**Common Mistakes:** يحكي فيتشر من غير رقم أو قرار ❌

---

### B2 — احكي عن Production Incident (السيرفر وقع / الدنيا ولعت)

**Question:**
احكي عن مرة السيستم وقع أو كان بطيء جداً في Production.

**Model Answer (STAR):**
**S:** يوم demo المشروع 50 يوزر دخلوا مع بعض والـ login علّق (HikariCP pool 10 خلص).
**T:** أرجّع السيستم بسرعة وأمنع تكرارها.
**A:** زودت Pool لـ 20 مؤقتاً + حطيت Rate limit + كشفت إن N+1 في تحميل اليوزر، صلحتها بـ `JOIN FETCH`، وزودت Actuator + logging للـ slow queries.
**R:** استقر في 10 دقايق، وبعدها P95 ثابت. الدرس: Degradation Point لازم يتعرف قبل الـ Peak مش بعده.

---

### B3 — اتعاملت مع متطلبات غامضة إزاي؟

**Question:**
احكي عن مرة العميل/الدكتور قال كلام عام وانت حوّلته لسيستم.

**Model Answer (STAR):**
**S:** اتقال "عايزين Booking system" بس.
**T:** أطلع scope واضح من غير ما أبني غلط.
**A:** حددت Actors (Customer/Worker/Admin/Payment) ثم Use Cases لكل واحد ثم Flows (Book → Accept → Pay) وكل Event كتبت Action+Data، ورجعت أكدت الـ edge cases (Cancel/Refund/No-show).
**R:** اكتشفنا 6 use cases ناقصة قبل الكود، وطلعت الـ Endpoints والـ Schema من الـ Flows مباشرة.

---

### B4 — Trade-off صعب: سرعة ضد جودة

**Question:**
احكي عن مرة ضحيت بحاجة عشان حاجة (Trade-off).

**Model Answer (STAR):**
**S:** كان لازم نسلم في 3 أسابيع والعميل عايز Search سريع + Filters كتير.
**T:** أوازن Time مقابل Performance.
**A:** أجلت Full-text search (Elastic) وعملت `LIKE + Index + Cache` للكلمات المشهورة، وكتبت Assumption: لو البحث زاد 10x هننقل Elastic.
**R:** سلمنا في المعاد والـ P95 كان 250ms. الدرس: No single architecture gives all — اختار ووثّق اللي أجّلته.

---

### B5 — Constraint قيّدك (Budget / Legacy / Legal)

**Question:**
احكي عن قيد فرض عليك تصميم معين.

**Model Answer (STAR):**
**S:** الكلية فرضت MySQL + Deploy على سيرفر واحد (Budget/Technical constraints).
**T:** أطلع بأحسن availability بالمتاح.
**A:** بدل ما أشتكي: Monolith + Redis + Backup يومي + Health checks + Loose coupling (Repository interfaces) عشان لو نقلنا Postgres بعدين.
**R:** السيستم استحمل التقييم كامل من غير downtime. الدرس: Constraints = نقطة بداية مش عذر.

---

### B6 — مشكلة N+1 أو Cache Stampede حقيقية

**Question:**
احكي عن مشكلة أداء شخصتها بالأرقام.

**Model Answer (STAR):**
**S:** صفحة المنتجات P50 حلو (150ms) بس P99 وحش (2.5s) — Long Tail.
**T:** أعرف مين اللي بيتأخر وليه.
**A:** قست بـ Actuator + logs: لقيت category مشهورة cache-miss + N+1 في الصور. عملت Pre-warm + `JOIN FETCH` + Caffeine L1.
**R:** P99 نزل لـ 400ms. الدرس: Average كذاب — بص على P95/P99.

---

### B7 — خلاف مع حد على تصميم (Monolith vs Microservices)

**Question:**
احكي عن خلاف تقني واتحل إزاي.

**Model Answer (STAR):**
**S:** زميلي عايز Microservices من أول يوم وأنا شايف Monolith.
**T:** نوصل لقرار مبني على constraints مش آراء.
**A:** حطينا Budget/Deadline/Team size على الترابيزة + عملنا Modular Monolith (packages مفصولة) مع contract واضح يسمح بالفصل بعدين.
**R:** خلصنا أسرع بشهر، وفصلنا Orders لوحدها لما احتاجت scale. الدرس: Monolith first استراتيجية مش كسل.

---

### B8 — Deploy خوّفك — عملت ايه؟

**Question:**
احكي عن Deploy كان مخيف وازاي أمنته.

**Model Answer (STAR):**
**S:** أول deploy كان manual وبيوقع 10 دقايق.
**T:** أوصل لـ deploy آمن ومتكرر (Deployability NFR).
**A:** عملت CI (build+tests) + Flyway migrations + Health endpoint + Rollback plan + Deploy بالليل + Smoke tests.
**R:** بقينا نعمل deploy مرتين في الأسبوع من غير خوف. الدرس: Deployability quality attribute زيها زي الأداء.

---

### B9 — لو هتعيد مشروعك من الأول — هتغيّر ايه؟

**Question:**
لو هتبدأ Dream Shop من الصفر بخبرتك دلوقتي — هتعمل ايه differently؟

**Model Answer (STAR):**
**S:** بدأت CRUD مباشرة من غير Use Cases و NFRs.
**T:** أحدد كان المفروض اعمل ايه.
**A:** كنت هبدأ: Actors/Use Cases/Flows → FR/NFR بأرقام (P95, RPS, 99.9%) → Constraints → Schema + Cache + Idempotency للـ orders من الأول + Observability (Actuator/Prometheus) من أول يوم.
**R:** كان هيوفر إعادة كتابة الـ checkout مرتين. الدرس: Design قبل Implementation أرخص 100x.

---

# ❓ Questions the Candidate Can Ask the Interviewer — 5 أسئلة ذكية

> اسأل أسئلة تبين إنك بتفكر System مش كود بس.

1. ايه أكبر Bottleneck عندكم دلوقتي — Latency ولا Throughput؟ وبتقيسوه بـ P95/P99 ولا Average؟
2. الـ Architecture هنا Monolith ولا Microservices؟ وايه الـ Trade-off اللي خلاكم تختاروا كده؟ وايه اللي نفسكم تغيّروه؟
3. ايه الـ Constraints اللي بتحكمكم (Cloud/Budget/Legal زي GDPR)؟ وازاي بتتعاملوا مع Loose Coupling مع الـ vendors؟
4. الـ On-call والـ Incidents ماشية إزاي؟ آخر Degradation حصل كان سببه ايه واتعلمتوا منه ايه؟
5. الـ Deployability عندكم عاملة إزاي (CI/CD frequency)؟ وايه معايير الـ NFRs اللي أي سيرفس جديدة لازم تحققها قبل الـ launch؟

---

# ⚡ Quick Review — المراجعة الأخيرة قبل الانترفيو

## 1) Cheat Sheet — Concept / When to Use / Drawback

| Concept | When to Use | Drawback / Trade-off |
|---|---|---|
| Load Balancer | Throughput عالي / HA | Single point لو واحد + Session مشكلة لو Stateful |
| Redis Cache | Repeated reads / Hot data | Stale + Invalidation + Memory cost |
| Caffeine (In-Memory) | L1 سريع / Config صغيرة | Local لكل instance (inconsistent) |
| CDN | صور/فيديو static | Invalidation + مش للـ dynamic |
| DB Index | WHERE/JOIN/ORDER BY تقيل | يبطئ Writes + مساحة |
| Replication | Read scale + HA | Lag + Stale reads + مش للـ writes |
| Sharding | حجم ضخم / Write scale | Resharding + Cross-shard صعب |
| SQL (MySQL) | فلوس/Relations/ACID | Scale أصعب |
| NoSQL (Mongo) | Flexible / Scale writes | No ACID قوي / Joins ضعيفة |
| Kafka | High throughput events / Replay | Ops معقد |
| RabbitMQ | Routing / Jobs خفيفة | Throughput أقل من Kafka |
| API Gateway | Auth/Rate/Aggregation على Edge | SPOF لو مش HA + مش Business Logic |
| Circuit Breaker | Downstream بيقع | Fallback إجباري + Tuning |
| Retry + Backoff | Transient errors بس | يقتل الواقع لو من غير limit |
| Saga | Multi-service transaction | Compensations معقدة (مفيش rollback سحري) |
| Stateless | عايز Horizontal Scale | محتاج external state (Redis/DB) |
| JWT | Auth موزع / Mobile | Revoke صعب + Size + XSS لو localStorage |
| P95/P99 | SLA حقيقي | أصعب من Average (محتاج metrics) |
| Loose Coupling (Interface) | Vendor/DB ممكن يتغيّر | Over-engineering لو overused |

---

## 2) Most Frequently Asked — أكتر أسئلة بتتكرر من الـ 9 ملفات 🔥

1. Functional vs Non-Functional vs Constraints — فرّق بالأمثلة والأرقام
2. Use Case vs Feature — عرّف + مثال Payment/YouTube
3. Actors → Use Cases → Flows (Action+Data) → Endpoints/DB — امشي السلسلة
4. Latency = Processing + Waiting — وقلل Waiting
5. Average كذاب — استخدم P95/P99 — اشرح Long Tail على مليون يوزر
6. SQL vs NoSQL — فلوس = SQL، flexible = NoSQL
7. Replication vs Sharding — نسخ مقابل تقسيم
8. CAP — اختار CP للفلوس و AP للـ feed
9. Monolith first — امتى تفك لـ Microservices؟
10. Loose Coupling (Stripe Interface) — وليه بعد Constraints؟
11. Degradation Point + HikariCP — ايه اللي بيحصل عند الـ Peak؟
12. Trade-offs — مفيش architecture بيدي كل حاجة + Feasibility بأنواعها

---

## 3) Pre-Interview Checklist — ✅ قبل ما تدخل

- [ ] أعرف أقول التلات أنواع (FR/NFR/Constraints) بالإنجليزي في 30 ثانية
- [ ] معايا مثال Ride Sharing و E-commerce و YouTube جاهزين
- [ ] أعرف أرسم `User → CDN → LB → Gateway → Service → Redis → DB` من غير تفكير
- [ ] حافظ أرقام Availability (99.9% = 43min/month) ومعنى P95/P99
- [ ] معايا 2 كود Spring جاهزين: `@Cacheable + Index` و `CircuitBreaker + Retry`
- [ ] أعرف أحكي Incident واحد و Trade-off واحد بالأرقام (STAR)
- [ ] أسأل في الآخر سؤالين عن Bottlenecks و Constraints عندهم
- [ ] نمت كويس 😄 — الـ System Design تفكير مش حفظ

---

> 💯 الخلاصة (زي كل الملفات):
> **System Design = تفكير في السيستم كله مش كود بس — What قبل How — قيس قبل ما تحسّن — مفيش حل ببلاش (Trade-offs) — وصمم على القيود بمرونة (Loose Coupling).**
> بالتوفيق يا علي 🔥

