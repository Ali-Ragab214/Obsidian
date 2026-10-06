---

# 📊 تقرير تحليل الفجوات - System Design Gap Analysis Report

> [!abstract]
> التقرير ده معمول عشان يقيم هل النوتس اللي عندك في فولدر [[System Design]] كافية لمهندس **Junior Software Engineer** يفهم System Design ويعدي انترفيو جونيور ولا لأ.
> اللغة والأسلوب نفس ستايل النوتس بتاعتك (عربي مصري + إنجليزي تكنيكال + إيموجي + جداول + أمثلة).

---

# 🎯 1. الحكم النهائي - Overall Verdict

## ❓ هل المحتوى كافي لـ Junior؟

### **الإجابة: جزئيًا - Partially ❌➡️✅**

> [!important]
> عندك **أساس قوي جدًا** في حوالي 60-65% من اللي مطلوب لجونيور، خصوصًا في:
> Availability, Fault Tolerance, Performance, Load Balancer, Databases/ACID, Replication, Rate Limiting, Message Queues, Idempotency
>
> لكن **لسه ناقصك حاجات أساسية بتتسأل حرفيًا في كل انترفيو جونيور** زي: Caching Strategies, CAP Theorem, Security (JWT/OAuth), Back-of-Envelope Estimation, Consistent Hashing, وتصميم أنظمة مشهورة (URL Shortener, Chat...).

**ليه جزئيًا؟**
- ✅ اللي موجود مكتوب حلو جدًا، بالعربي بسيط + أمثلة عملية + كود Java/Spring + رسومات ASCII/mermaid
- ✅ يغطي النظري اللي الشركات الكبيرة بتسأل فيه عن Scalability/Availability
- ❌ ناقص التطبيق العملي (مفيش ولا System Design كامل محلول)
- ❌ ناقص مواضيع Hot Topics للجونيور: Cache, CAP, Auth, Estimation, CDN, Consistent Hashing
- ❌ خطوة الـ Step-by-Step للانترفيو موجودة كصورة بس (`[[17-SystemDesign step by step]]`) من غير شرح نصي

> **الخلاصة:** لو دخلت انترفيو دلوقتي هتعرف تجاوب أسئلة نظري كويس، بس هتقع في أسئلة زي "صمم URL Shortener" أو "إيه الفرق بين Cache-Aside و Write-Through" أو "احسب Storage لـ Instagram".

---

# 📚 2. ملخص اللي موجود حاليًا - What Exists

عندك **35 ملف** في فولدر `System Design` (مفيش subfolders). التقسيمة كالتالي:

## 🟢 الأساسيات والنظري العام (7 ملفات)

| الملف | بيغطي إيه باختصار |
|---|---|
| [[0-Priorities]] | لستة أولويات 10 مواضيع (Roadmap مختصر) |
| [[1-Road Map]] | خارطة طريق 8 مراحل من Zero لـ FAANG + خطة 7 أسابيع + ربط بمشروع Dream Shop |
| [[2-Intro]] | يعني إيه System Design؟ الفرق بين LLD/HLD + مثال Instagram |
| [[3-Lecture 1 — Introduction to Software Architecture & System Requirements]] | تعريف Architecture + SDLC + أنواع المتطلبات (Functional/Non-Functional/Constraints) |
| [[4-Feature Requirements]] + [[4.1-Use Case vs Feature]] | طريقة جمع المتطلبات الصح (Actors/Use Cases/User Flows/Sequence Diagram) |
| [[5-Quality attributes (Non Functional reqs)]] | Quality Attributes + ليه الأنظمة بتفشل + Feasibility (Technical/Financial/Time) |
| [[6-System Constraints]] | أنواع القيود (Technical/Business/Legal) + Loosely Coupled Architecture |

## 🔵 Performance & Scalability & Reliability (6 ملفات)

| الملف | بيغطي إيه |
|---|---|
| [[7-Performance]] | Latency (Processing vs Waiting) + Throughput + P95/P99 vs Average + Degradation Point + HikariCP |
| [[8-Scalability]] | Vertical vs Horizontal vs Team Scalability + Degradation Point + Load Balancer مقدمة + Microservices ليه ظهرت |
| [[9-Availability]] | تعريف Availability + Nines (99% → 99.999%) + MTBF/MTTR + Outage vs Downtime + Spring Actuator |
| [[10-Fault tolerance & High Availability]] | معادلة Fault Tolerance = Prevention+Detection+Recovery + SPOF/Redundancy/Replication + Active-Active vs Active-Passive + Health Checks + Failover + Mermaid كتير |
| [[11-SLI, SLO, SLA]] | SLI (قياس) vs SLO (هدف) vs SLA (عقد) + Error Budget + مثال Delivery 25 دقيقة |
| [[16-Software Architecture and patterns]] | Coupling/Cohesion/SoC + 4 أنماط (Layered/Clean/Microservices/Event-Driven) + 4 Design Patterns (Repository/Factory/Strategy/Observer) |

## 🟣 Databases (5 ملفات)

| الملف | بيغطي إيه |
|---|---|
| [[14-Relational Databases & ACID]] | ليه SQL؟ Table/Row/Column + PK/FK + Relations 1-1/1-M/M-M + Normalization 1NF-3NF vs Denormalization + Indexes B-Tree + Joins + ACID + Isolation Levels + Scaling (Replica/Partitioning/Sharding) |
| [[15-ACID]] | شرح ACID بالتفصيل بأمثلة بنكية + Isolation والـ Locks (`SELECT FOR UPDATE`) + Dirty/Phantom Reads + WAL + Saga Pattern للـ Microservices |
| [[18-DB Reblication]] | Primary-Replica + Sync vs Async + Read Scaling + Failover + Geographic Latency |
| [[19-Stale Read]] | Stale Read = نتيجة Replication Lag + Read-Your-Writes + Read-After-Write Consistency |
| [[21-Replication Types, Lag , Data Consistency Across Shards]] | Single/Multi-Leader/Leaderless (Quorum) + أسباب Lag + Consistency Across Shards + 2PC vs Eventual Consistency + Shard Key |

## 🟡 Networking & Load Balancing (4 ملفات)

| الملف | بيغطي إيه |
|---|---|
| [[13-DNS & Load Balancer]] | ليه محتاج LB + فوائده 4 + أنواع LB (DNS/Hardware/Software/GSLB) + مقارنة كاملة |
| [[13.1 LB]] | موسوعة LB 921 سطر: Definition + Algorithms (Round Robin/Least Connections/Weighted/IP Hash) + Health Checks + Sticky Sessions + SSL Termination + L4 vs L7 + Reverse Proxy + Microservices + Interview Q&A |
| [[13.1.0 AWS]] | تطبيق AWS عملي: EC2 + ALB + Auto Scaling + Target Groups |
| [[12-Api Design]] | API كـ Black Box/Contract + أنواع APIs (Public/Private/Partner) + Best Practices (Encapsulation/Idempotency/Pagination/Async Polling) |

> ⚠️ ملاحظة: [[13-DNS & Load Balancer]] و [[13.1 LB]] مكررين بنسبة 80% — نفس المحتوى بترتيب مختلف.

## 🟠 Advanced Topics (6 ملفات)

| الملف | بيغطي إيه |
|---|---|
| [[22-Rate Limiting]] | 581 سطر ممتاز: ليه؟ + By IP/User/Key/Plan + Redis للعدادات + 4 خوارزميات (Fixed/Sliding/Token Bucket/Leaky Bucket) + 429 + Bucket4j/Resilience4j |
| [[23-Message Queues]] | Producer/Broker/Consumer + Async/Decoupling/Load Leveling + Push vs Pull + 5 Patterns + Delivery Guarantees (At Most/At Least/Exactly Once) + ACK/Retry/DLQ + Kafka vs RabbitMQ |
| [[20-Failover & Leader Election & split brain]] | Failover + Election via etcd/ZooKeeper + Split Brain + Old Primary بيرجع Replica |

## 🔴 Idempotency Cluster (6 ملفات - مغطى بعمق زيادة عن اللزوم)

| الملف | بيغطي إيه |
|---|---|
| [[24-Idempotency]] | ملخص خام: Safe vs Idempotent Methods + Payment double charge |
| [[25-Idempotency exp]] | 894 سطر شرح مفصل + UUID + HandlerInterceptor + مشاكل HashSet |
| [[26-Idempotency Key]] | الفرق بين Operation vs Request + مين يولد UUID (Frontend) + Receipt analogy |
| [[27-Idempotency Response Miss]] | Response ضاع = المشكلة الحقيقية (Wi-Fi/Timeout/LB) |
| [[28-Idempotency to record video]] | سكريبت فيديو مختصر لنفس الموضوع |
| [[29-Idempotency Code]] | 754 سطر مراجعة كود Spring سطر بسطر + 8 أخطاء (payements typo/header mismatch/HashSet مش thread-safe) |
| [[30-Api Versioning]] | 581 سطر: Breaking Change + 4 طرق Versioning (URL/Header/Query/Accept) + Lifecycle + API Gateway |
| [[z-REST]] | مش شرح REST، ده Gap Analysis قديم انت كاتبه بنفسك بيلخص النواقص (CAP/Consistent Hashing/Indexing...) |

---

# 📊 3. جدول التغطية الكامل - Coverage Table

> [!tip]
> Covered well = موجود ومشروح كويس بالأمثلة
> Covered partially = موجود بس سطحي أو ناقص تفاصيل مهمة
> Missing = مش موجود خالص أو سطر واحد بس

| # | الموضوع | الحالة | ملاحظات | الملف المرتبط |
|---|---|---|---|---|
| **A. Fundamentals** | | | | |
| 1 | Client-Server Model + 3-Tier Architecture | 🟡 Covered partially | موجود في [[1-Road Map]] و [[2-Intro]] و [[16-Software Architecture and patterns]] بس مفيش شرح عميق لـ Request lifecycle | [[1-Road Map]] [[2-Intro]] |
| 2 | HTTP/HTTPS, REST, Verbs, Status Codes | 🟡 Covered partially | REST مذكور في Roadmap بس مفيش ملف مخصص. [[12-Api Design]] بيغطي API design بس مش HTTP نفسه. `z-REST.md` اسمه REST بس محتواه تحليل نواقص | [[12-Api Design]] |
| 3 | DNS, IP, Ports, TCP/IP Basics | 🟢 Covered well | DNS مشروح ممتاز جدًا في [[13-DNS & Load Balancer]] و [[13.1 LB]] مع Resolution Flow و TTL | [[13-DNS & Load Balancer]] [[13.1 LB]] |
| 4 | Latency vs Throughput | 🟢 Covered well | ممتاز في [[7-Performance]] مع P95/P99 و Long Tail | [[7-Performance]] |
| **B. Scalability** | | | | |
| 5 | Vertical vs Horizontal Scaling | 🟢 Covered well | مشروح بالتفصيل مع مميزات/عيوب + تكلفة في [[8-Scalability]] و [[13.1 LB]] | [[8-Scalability]] |
| 6 | Load Balancing (Types + Algorithms) | 🟢 Covered well | موسوعة في [[13.1 LB]] (4 أنواع + 4 Algorithms + Health Checks + L4/L7) | [[13.1 LB]] [[13-DNS & Load Balancer]] |
| 7 | Stateless vs Stateful + Session Management | 🔴 Missing | مذكور كعنوان في [[1-Road Map]] بس مفيش ملف بيشرحه. Sticky Sessions مذكورة سريعًا في [[13.1 LB]] | — |
| 8 | Auto Scaling | 🟡 Covered partially | سطر واحد في [[13.1.0 AWS]] (Auto Scaling) + عنوان في [[1-Road Map]] | [[13.1.0 AWS]] |
| **C. Databases** | | | | |
| 9 | SQL vs NoSQL (أنواع NoSQL) | 🟡 Covered partially | عنوان في [[1-Road Map]] + فقرة صغيرة في [[14-Relational Databases & ACID]] (امتى متستخدمش SQL) بس مفيش مقارنة Document/Key-Value/Column/Graph | [[14-Relational Databases & ACID]] |
| 10 | Normalization / Denormalization | 🟢 Covered well | مشروح بأمثلة 1NF-3NF + Denormalization لـ 5 Joins في [[14-Relational Databases & ACID]] | [[14-Relational Databases & ACID]] |
| 11 | Indexing (B-Tree, Hash, Internals) | 🟡 Covered partially | `CREATE INDEX` و `O(log n)` و تشبيه فهرس الكتاب في [[14-Relational Databases & ACID]] بس مفيش B-Tree/B+Tree Internals | [[14-Relational Databases & ACID]] |
| 12 | ACID + Transactions + Isolation Levels | 🟢 Covered well | ممتاز جدًا في [[14-Relational Databases & ACID]] و [[15-ACID]] مع أمثلة بنكية + Locks + WAL | [[14-Relational Databases & ACID]] [[15-ACID]] |
| 13 | Replication (Sync/Async, Lag, Types) | 🟢 Covered well | مغطى بعمق في [[18-DB Reblication]] + [[19-Stale Read]] + [[21-Replication Types, Lag , Data Consistency Across Shards]] | [[18-DB Reblication]] [[21-Replication Types, Lag , Data Consistency Across Shards]] |
| 14 | Sharding / Partitioning (Strategies, Shard Key, Hot Spots) | 🟡 Covered partially | Sharding مذكور في [[14-Relational Databases & ACID]] و [[21-Replication Types, Lag , Data Consistency Across Shards]] بس مفيش تفصيل Range/Hash/Directory + Hot Spots + Resharding | [[21-Replication Types, Lag , Data Consistency Across Shards]] |
| **D. Consistency & Theorems** | | | | |
| 15 | CAP Theorem | 🔴 Missing | موجود في [[0-Priorities]] كعنوان بس مفيش ملف بيشرحه. مذكور كـ Missing في [[z-REST]] | — |
| 16 | Consistency Models (Strong/Eventual/Read-Your-Writes) + BASE | 🟡 Covered partially | Eventual و Read-After-Write مذكورين في [[19-Stale Read]] و [[21-Replication Types, Lag , Data Consistency Across Shards]] بس BASE مش مشروح | [[19-Stale Read]] |
| 17 | Consistent Hashing | 🔴 Missing | مذكور كـ Missing في [[z-REST]] + عنوان في Roadmap بس مفيش شرح | — |
| **E. Caching** | | | | |
| 18 | Caching Strategies (Cache-Aside, Write-Through, Write-Behind, Read-Through) | 🔴 Missing | Redis مذكور ككلمة في [[1-Road Map]] و [[0-Priorities]] بس مفيش ملف Caching مخصص | — |
| 19 | Eviction Policies (LRU, LFU, TTL) | 🔴 Missing | مش موجود خالص | — |
| 20 | CDN | 🔴 Missing | عنوان في [[1-Road Map]] بس مفيش شرح | — |
| 21 | Redis Use Cases | 🟡 Covered partially | مذكور سريعًا في [[7-Performance]] (Cache) و [[22-Rate Limiting]] (عدادات Redis) | [[7-Performance]] |
| **F. Messaging & Async** | | | | |
| 22 | Message Queues (Kafka vs RabbitMQ) + Patterns | 🟢 Covered well | ممتاز في [[23-Message Queues]] (637 سطر + جدول مقارنة + DLQ + Ordering) | [[23-Message Queues]] |
| 23 | Async Processing + Event-Driven Architecture | 🟢 Covered well | في [[23-Message Queues]] و [[16-Software Architecture and patterns]] (Event-Driven مع `@EventListener`) | [[23-Message Queues]] [[16-Software Architecture and patterns]] |
| **G. Architecture Styles** | | | | |
| 24 | Monolith vs Microservices | 🟡 Covered partially | مقدمة في [[1-Road Map]] و [[8-Scalability]] (Team Scalability) و [[16-Software Architecture and patterns]] (700 microservices Netflix) بس مفيش مقارنة Trade-offs عميقة | [[16-Software Architecture and patterns]] |
| 25 | API Gateway | 🟡 Covered partially | رسمة في [[13.1 LB]] و [[30-Api Versioning]] + عنوان في Roadmap بس مفيش ملف مخصص يشرح Rate Limiting/Auth/Routing في Gateway | [[13.1 LB]] |
| 26 | Service Discovery + Service Mesh | 🔴 Missing | مش موجود خالص | — |
| **H. Security & Reliability** | | | | |
| 27 | Rate Limiting | 🟢 Covered well | موسوعة 581 سطر في [[22-Rate Limiting]] مع 4 خوارزميات + كود Bucket4j | [[22-Rate Limiting]] |
| 28 | Authentication/Authorization (JWT, OAuth2, Session vs Token, SSO) | 🔴 Missing | عنوان في [[1-Road Map]] بس مفيش ملف. [[22-Rate Limiting]] ذكر Login بس | — |
| 29 | Security Basics (HTTPS, Hashing, Encryption) | 🟡 Covered partially | Hashing مذكور في Roadmap + SSL Termination في [[13.1 LB]] | [[13.1 LB]] |
| 30 | Availability + Fault Tolerance + Failover | 🟢 Covered well | ممتاز في [[9-Availability]] (Nines + MTBF) + [[10-Fault tolerance & High Availability]] (موسوعة) + [[20-Failover & Leader Election & split brain]] | [[10-Fault tolerance & High Availability]] [[20-Failover & Leader Election & split brain]] |
| 31 | Monitoring / Observability (Logging, Metrics, Tracing) | 🟡 Covered partially | SLI/SLO/SLA ممتاز في [[11-SLI, SLO, SLA]] + Prometheus/Grafana سريعًا في [[7-Performance]] بس مفيش Logging/Tracing مفصل | [[11-SLI, SLO, SLA]] |
| **I. Estimation & Practice** | | | | |
| 32 | Back-of-Envelope Estimation (QPS, Storage, Bandwidth) | 🔴 Missing | مش موجود خالص. مذكور كـ Missing في [[z-REST]] | — |
| 33 | Common Design Problems (URL Shortener, Chat, Notification...) | 🔴 Missing | عناوين في [[1-Road Map]] (Instagram/WhatsApp/URL Shortener) بس مفيش ولا تصميم محلول | — |
| **J. Patterns & Interview Method** | | | | |
| 34 | Design Patterns (SOLID, CQRS, Circuit Breaker, Retry, Bulkhead, Saga, 2PC) | 🟡 Covered partially | Circuit Breaker/Retry عنوان في Roadmap بس مش مشروح. Saga و 2PC مذكورين سريعًا في [[15-ACID]] و [[21-Replication Types, Lag , Data Consistency Across Shards]]. Repository/Factory/Strategy/Observer في [[16-Software Architecture and patterns]] | [[15-ACID]] [[16-Software Architecture and patterns]] |
| 35 | Step-by-Step Interview Approach (Requirements → Estimation → Design → Deep Dive) | 🟡 Covered partially | [[17-SystemDesign step by step]] فيه صورتين بس من غير نص. المتطلبات مشروحة كويس في [[4-Feature Requirements]] بس مفيش framework كامل للانترفيو | [[17-SystemDesign step by step]] [[4-Feature Requirements]] |
| 36 | Idempotency + API Versioning | 🟢 Covered well | Idempotency مغطى بزيادة (6 ملفات!) + [[30-Api Versioning]] ممتاز | [[25-Idempotency exp]] [[30-Api Versioning]] |

---

# 🚨 4. المواضيع الناقصة - Missing Topics (مرتبة حسب الأولوية)

## 🔴 أولوية عالية - High Priority (لازم تذاكرها قبل أي انترفيو جونيور)

| # | الموضوع الناقص | ليه مهم جدًا لجونيور؟ |
|---|---|---|
| 1 | **Caching Strategies كاملة** (Cache-Aside, Write-Through, Write-Behind, Read-Through + Invalidation + Eviction LRU/LFU/TTL) | بتتسأل في 90% من انترفيوهات Backend. عندك Redis خبرة عملية من SkillHub بس ناقص النظري. أي سؤال "صمم Instagram" هيبدأ بـ Cache |
| 2 | **CAP Theorem + BASE** | Framework التفكير في أي Distributed System. الانترفيور هيقولك "تختار Consistency ولا Availability؟" لازم تجاوب بـ CAP |
| 3 | **Consistent Hashing** | أساس Sharding و Load Balancing الحديث + Cache Distribution. مذكور في [[z-REST]] كناقص |
| 4 | **Authentication at Scale (JWT, OAuth2, Session vs Token, SSO, Refresh Token)** | أي System Design فيه Users لازم تشرح Auth. بتتسأل كتير "إزاي تعمل Login scalable؟" |
| 5 | **Back-of-Envelope Estimation** | أول 10 دقايق في أي انترفيو System Design. حساب QPS/Storage/Bandwidth/Memory. من غيرها تصميمك هيبقى عشوائي |
| 6 | **تصميم أنظمة مشهورة محلولة (URL Shortener, Chat System, Notification System, News Feed, Rate Limiter Design)** | ده اللي الانترفيو كله عنه! Roadmap عندك ذاكرهم كعناوين بس مفيش ولا واحد محلول خطوة بخطوة |
| 7 | **CDN + كيف بيشتغل** | أي System فيه صور/فيديو (Instagram/YouTube) لازم تذكر CDN. عنوان بس في [[1-Road Map]] |

## 🟡 أولوية متوسطة - Medium Priority (مهمة بس ممكن تتأجل أسبوع)

| # | الموضوع الناقص | ليه مهم؟ |
|---|---|---|
| 8 | **SQL vs NoSQL بالتفصيل + أنواع NoSQL الأربعة** | [[14-Relational Databases & ACID]] ذكر SQL بس. لازم تعرف امتى تستخدم Document (MongoDB) vs Key-Value (Redis) vs Column (Cassandra) vs Graph (Neo4j) |
| 9 | **Stateless vs Stateful Architecture + Session Management** | عشان تفهم ليه بنخلي السيرفرات Stateless مع Load Balancer (Sticky Sessions مش حل scalable) |
| 10 | **API Gateway بالتفصيل + Service Discovery** | موجود كرسمة بس في [[13.1 LB]]، محتاج شرح Auth/Rate Limiting/Routing/Aggregation في Gateway + Eureka/Consul للـ Discovery |
| 11 | **Database Indexing Internals (B-Tree vs B+Tree vs Hash Index)** | [[14-Relational Databases & ACID]] قال `O(log n)` بس الانترفيو ممكن يسأل "ليه B+Tree؟" |
| 12 | **Sharding Strategies بالتفصيل (Range/Hash/Directory + Shard Key + Hot Spots + Resharding + Cross-Shard Queries)** | [[21-Replication Types, Lag , Data Consistency Across Shards]] لمسه بس، محتاج ملف مخصص زي ما [[z-REST]] اقترح |
| 13 | **Resilience Patterns (Circuit Breaker, Retry with Exponential Backoff, Bulkhead, Timeout)** | عناوين في [[1-Road Map]] بس مفيش شرح. بتتسأل "لو Payment Service وقع هتعمل إيه؟" |
| 14 | **Observability (Logging, Metrics, Distributed Tracing - ELK, Prometheus, Jaeger)** | [[11-SLI, SLO, SLA]] غطى جزء بس، محتاج Logging/Tracing اللي الشركات الكبيرة بتركز عليه |

## 🟢 أولوية منخفضة - Low Priority (لو عايز تبقى مميز)

| # | الموضوع الناقص | ليه؟ |
|---|---|---|
| 15 | **SOLID Principles + CQRS + Event Sourcing** | مفيد لـ LLD أكتر من HLD، بس بيدي انطباع كويس |
| 16 | **WebSockets vs SSE vs Long Polling vs Webhooks** | [[12-Api Design]] ذكرهم سريعًا في Async، بس محتاج مقارنة عملية للـ Chat/Notification |
| 17 | **GraphQL vs REST vs gRPC** | [[12-Api Design]] ذكر مقارنة كـ Obsidian link بس مفيش محتوى |
| 18 | **Security المتقدم (Encryption at Rest/In Transit, DDoS Protection)** | للمستوى الأعلى من جونيور |

---

# 🔧 5. مواضيع موجودة بس محتاجة عمق أو أمثلة أكتر

| الموضوع | فين موجود | إيه اللي ناقصه؟ | اقتراح تحسين |
|---|---|---|---|
| **HTTP/REST Fundamentals** | [[12-Api Design]] | مفيش شرح HTTP Verbs/Status Codes/Headers/Cookies/REST Constraints | زود ملحق في [[12-Api Design]] يشرح `GET/POST/PUT/PATCH/DELETE + 200/201/400/401/429/500 + Statelessness` |
| **SQL vs NoSQL** | [[14-Relational Databases & ACID]] | فقرة واحدة "امتى متستخدمش SQL" | اعمل مقارنة جدولية + أمثلة: E-commerce → SQL، Chat → NoSQL (Cassandra), Cache → Redis |
| **Indexing** | [[14-Relational Databases & ACID]] | تشبيه فهرس الكتاب جميل بس ناقص رسمة B+Tree + مثال `EXPLAIN ANALYZE` | زود مثال عملي `SELECT * FROM orders WHERE user_id=5` قبل وبعد Index مع أرقام |
| **Sharding** | [[21-Replication Types, Lag , Data Consistency Across Shards]] | مذكور كتعريف بس | محتاج ملف منفصل يشرح Hot Spot (مثلاً Celebrity User) + Resharding |
| **Stateless/Stateful** | [[13.1 LB]] (Sticky Sessions) | سطرين بس | اشرح ليه Stateless + JWT هو الحل الصح مع LB، و Sticky Sessions مش scalable |
| **API Gateway** | [[13.1 LB]] + [[30-Api Versioning]] | رسمة بس | اشرح Gateway كـ Single Entry Point: Auth + Rate Limiting + Routing + Aggregation |
| **Load Balancer** | [[13-DNS & Load Balancer]] و [[13.1 LB]] | **مكرر 80%** — نفس المحتوى في ملفين! | ادمجهم في ملف واحد مرجعي واحذف التكرار، أو خلّي [[13-DNS & Load Balancer]] مختصر و [[13.1 LB]] هو المرجع |
| **Availability Nines** | [[9-Availability]] | جدول Nines مش كامل (سطور ناقصة) | كمل الجدول: `99% = 3.65 days/year, 99.9% = 8.76h, 99.99% = 52min, 99.999% = 5min` |
| **Performance Percentiles** | [[7-Performance]] | ممتاز بس ناقص مثال حسابي لـ P99 مع 1M users | المثال موجود بس ممكن تضيف Histogram رسمة أوضح |
| **Microservices** | [[16-Software Architecture and patterns]] | ذكر Netflix 700 service بس مفيش Trade-offs (Complexity, Distributed Transactions) | زود جدول Monolith vs Microservices (متى تستخدم كل واحد) |
| **Saga / 2PC** | [[15-ACID]] + [[21-Replication Types, Lag , Data Consistency Across Shards]] | سطرين بس | محتاج مثال E-commerce: Order→Payment→Inventory مع Compensation |
| **Step-by-Step Interview** | [[17-SystemDesign step by step]] | صورتين بس من غير شرح نصي قابل للبحث | حول الصور لنص مكتوب: `1. Requirements 2. Estimation 3. High-Level Design 4. Deep Dive 5. Trade-offs` |
| **Idempotency** | 6 ملفات! | **مغطى بزيادة عن اللزوم** → 6 ملفات لنفس الموضوع | ادمج [[24-Idempotency]]+[[28-Idempotency to record video]] كملخص، وخلّي [[25-Idempotency exp]]+[[29-Idempotency Code]] هما المرجع |

> [!tip]
> نصيحة: متكررش موضوع واحد في 6 ملفات (Idempotency) وتسيب مواضيع أهم فاضية (Caching/CAP). الأفضل تدمج المكرر وتستغل الوقت في النواقص.

---

# 🐛 6. أخطاء، تناقضات، ونقاط غير واضحة

## ❌ أخطاء إملائية / Typo

| الملف | الخطأ | الصح |
|---|---|---|
| [[18-DB Reblication]] | `Reblication` في اسم الملف | `Replication` |
| [[29-Idempotency Code]] و [[25-Idempotency exp]] | `payements` (بـ e زيادة) في `@RequestMapping("/api/v1/payements")` | `payments` — وده كسر الـ Interceptor اللي بيراقب `/api/v1/payments` (من غير e) |
| [[29-Idempotency Code]] | `request.getHeader("universal unique identifier")` | المفروض `Idempotency-Key` أو `ID_UUID` — الاسم مش consistent مع `@RequestHeader String uuid` |
| [[14-Relational Databases & ACID]] | `00.0%` في مثال Availability | `99.9%` |

## ⚠️ تناقضات أو شرح محتاج توضيح

1.  **Load Balancer مكرر:** [[13-DNS & Load Balancer]] (375 سطر) و [[13.1 LB]] (921 سطر) بيشرحوا نفس الأنواع الأربعة بنفس الأمثلة (SkillHub/Netflix). ده بيلخبط — أي واحد هو المرجع؟
2.  **Header Name مش Consistent في Idempotency:** في [[25-Idempotency exp]] بتقول `Idempotency-Key: 123-abc` وفي [[29-Idempotency Code]] بتقول `ID_UUID` وفي الكود `universal unique identifier`. الانترفيور هيسألك "إيه الاسم الصح؟" — الجواب: `Idempotency-Key` هو الـ Standard (Stripe/AWS).
3.  **HashSet vs Redis:** [[25-Idempotency exp]] و [[29-Idempotency Code]] بيستخدموا `private static Set<String> cache = new HashSet<>()` كحل، وبعدين بيقولوا ده غلط (بيضيع مع Restart + مش Thread-Safe + Multi-Server problem). الشرح ممتاز بس لازم توضح من الأول إن ده **Demo فقط** والصح هو `Redis + TTL` أو `DB Table` — حاليًا القارئ ممكن يفتكر HashSet هو الحل.
4.  **Synchronous Replication:** في [[18-DB Reblication]] بتقول Sync = "100% consistent بس أبطأ" — ده صح بس محتاج توضيح إن Sync في Production نادرًا بيكون على كل Replicas (بيكون Semi-Sync أو Quorum) عشان الـ Latency.
5.  **Availability 100% مستحيل:** في [[9-Availability]] بتقول "ليه مش ممكن نوصل 100%؟" وده صح، بس في [[10-Fault tolerance & High Availability]] بتقول Active-Active "كل السيرفرات شغالة" — محتاج توضح إن حتى Active-Active مش 100% (Network Partition ممكن يوقع الكل).
6.  **Idempotency Methods:** في [[24-Idempotency]] بتقول `HEAD, OPTIONS, TRACE` Idempotent — صح نظريًا بس نادرًا ما بتتسأل في انترفيو Backend. الأهم تركز على `GET (Safe) / PUT/DELETE (Idempotent) / POST (Not Idempotent)`.
7.  **z-REST.md:** اسمه `z-REST` بس محتواه مش عن REST خالص — هو Gap Analysis قديم. الاسم بيلخبط، الأفضل تسميه `z-Gap-Analysis-Old` أو تدمجه مع التقرير ده.
8.  **Frontmatter وهمي:** كل الملفات فيها `---` كـ Horizontal Rule مش كـ YAML Frontmatter. مفيش Tags ولا Title. ده مش خطأ بس لو عايز Obsidian Graph يبقى أقوى، زود `tags: [system-design]` في المستقبل.

## 💡 نقاط غير واضحة محتاجة أمثلة

- **Sharding في [[14-Relational Databases & ACID]] و [[21-Replication Types, Lag , Data Consistency Across Shards]]:** بتقول "Shard Key" بس مفيش مثال عملي إزاي تختاره (مثلاً `user_id` vs `order_id` والـ Hot Spot لو Celebrity).
- **Eventual Consistency:** مذكور كحل بس مفيش مثال يوضح الفرق بين Strong vs Eventual بالأرقام (مثلاً "بعد 2 ثانية كل Replicas هتبقى Consistent").
- **Token Bucket في [[22-Rate Limiting]]:** الشرح ممتاز بس ناقص رسمة توضح Refill Rate vs Burst.

---

# 📁 7. ملفات جديدة مقترحة + ترتيب المذاكرة

## 🗂️ ملفات جديدة تقترح تنشئها (بنفس ستايل تسميتك)

> كل اسم مقترح بنفس فورمات `رقم-الموضوع.md` عشان يمشي مع ترتيبك الحالي.

| # | اسم الملف المقترح | بيغطي إيه | الأولوية |
|---|---|---|---|
| 1 | `31-CAP Theorem & BASE.md` | CAP (Consistency/Availability/Partition Tolerance) + PACELC + BASE vs ACID + أمثلة CP (Postgres) vs AP (Cassandra) | 🔴 High |
| 2 | `32-Caching Strategies & Redis.md` | Cache-Aside/Write-Through/Write-Behind/Read-Through + Invalidation + LRU/LFU/TTL + Redis vs Memcached + أمثلة SkillHub `GET /products` | 🔴 High |
| 3 | `33-CDN.md` | إيه هو CDN + كيف بيشتغل (Edge Servers) + Cache Hit/Miss + أمثلة CloudFront/Cloudflare | 🔴 High |
| 4 | `34-Consistent Hashing.md` | Hash Ring + Virtual Nodes + ليه أحسن من Modulo + علاقة بـ Sharding و Cache | 🔴 High |
| 5 | `35-Authentication at Scale (JWT & OAuth2).md` | JWT Structure (Header/Payload/Signature) + OAuth2 Flow + Session vs Token + Refresh Token + SSO | 🔴 High |
| 6 | `36-Back-of-Envelope Estimation.md` | أرقام تحفظها (1M QPS, 1KB per request) + حساب Storage/Bandwidth/Memory + مثال URL Shortener بالأرقام | 🔴 High |
| 7 | `37-System Design - URL Shortener.md` | أول تصميم كامل: Requirements → Estimation → API Design → DB Design (Base62) → Cache → Scaling | 🔴 High |
| 8 | `38-System Design - Chat System (WhatsApp).md` | WebSockets + Message Queue + Presence + Fanout + Storage | 🟡 Medium |
| 9 | `39-System Design - Notification System.md` | Push/Pull + Queue + Retry + Rate Limiting | 🟡 Medium |
| 10 | `40-Stateless vs Stateful & Session Management.md` | ليه Stateless + JWT vs Sticky Sessions + Session Store (Redis) | 🟡 Medium |
| 11 | `41-API Gateway & Service Discovery.md` | Gateway Responsibilities + Service Discovery (Eureka/Consul) + Load Balancing مع Discovery | 🟡 Medium |
| 12 | `42-Resilience Patterns (Circuit Breaker, Retry, Bulkhead).md` | Circuit Breaker States (Closed/Open/Half-Open) + Exponential Backoff + Bulkhead + أمثلة Resilience4j | 🟡 Medium |
| 13 | `43-Sharding Deep Dive.md` | Range/Hash/Directory + Shard Key Selection + Hot Spots (Celebrity Problem) + Cross-Shard Queries + Resharding | 🟡 Medium |
| 14 | `44-Observability (Logging, Metrics, Tracing).md` | ELK + Prometheus/Grafana + Jaeger/Zipkin + علاقتها بـ SLI/SLO | 🟢 Low |
| 15 | `45-SQL vs NoSQL Deep Dive.md` | مقارنة جدولية + أنواع NoSQL الأربعة + متى تستخدم كل واحدة + أمثلة عملية | 🟢 Low |

---

## 🗓️ ترتيب المذاكرة المقترح (4 أسابيع بعد اللي عندك)

> مبني على خطة [[1-Road Map]] الأسبوع 7+، بس بترتيب أولويات الانترفيو.

### الأسبوع 1: سد الفجوات الحرجة

1.  [[31-CAP Theorem & BASE]] → افهم ليه بنختار CP ولا AP
2.  [[32-Caching Strategies & Redis]] → أهم موضوع للجونيور
3.  [[34-Consistent Hashing]] → اربطه بـ Caching و Sharding

### الأسبوع 2: الأمان والتقدير

4.  [[35-Authentication at Scale (JWT & OAuth2)]] → أي System فيه Login
5.  [[36-Back-of-Envelope Estimation]] → احفظ الأرقام (1M, 1GB, QPS...)
6.  [[33-CDN]] → سريع (يوم واحد)

### الأسبوع 3: التطبيق العملي (الأهم!)

7.  [[37-System Design - URL Shortener]] → صممه بنفسك خطوة بخطوة
8.  [[38-System Design - Chat System (WhatsApp)]] → WebSockets + Queue
9.  [[39-System Design - Notification System]] → Fanout

### الأسبوع 4: التقوية

10. [[40-Stateless vs Stateful & Session Management]]
11. [[41-API Gateway & Service Discovery]]
12. [[42-Resilience Patterns (Circuit Breaker, Retry, Bulkhead)]]
13. راجع [[17-SystemDesign step by step]] وحوله لنص + حل 2-3 أنظمة كمان (News Feed, Search Autocomplete)

> [!success]
> بعد الـ 4 أسابيع دول هتبقى غطيت 95% من اللي مطلوب لجونيور، والـ 35 ملف اللي عندك + 15 الجديد هيبقوا بورتفوليو قوي جدًا.

---

# 🔍 8. كلمات مفتاحية ومصادر لكل موضوع ناقص

| الموضوع الناقص | كلمات تبحث بيها (عربي/إنجليزي) | مصدر مقترح سريع |
|---|---|---|
| **CAP Theorem** | `CAP theorem explained, PACELC, BASE vs ACID, CP vs AP database` | ByteByteGo - CAP Theorem (يوتيوب) + Martin Kleppmann Ch.9 |
| **Caching Strategies** | `Cache aside vs write through vs write behind, cache invalidation, LRU LFU, Redis cache patterns` | ByteByteGo - Caching Strategies + Redis Docs |
| **Consistent Hashing** | `Consistent hashing virtual nodes, hash ring, consistent hashing vs modulo` | ByteByteGo - Consistent Hashing + System Design Interview Book Ch.5 |
| **JWT/OAuth2** | `JWT structure header payload signature, OAuth2 authorization code flow, session vs JWT, refresh token` | Web Dev Simplified - JWT + OAuth2 Illustrated |
| **Back-of-Envelope** | `Back of envelope estimation system design, QPS storage bandwidth calculation, powers of two` | System Design Interview (Alex Xu) Appendix + ByteByteGo Cheat Sheet |
| **URL Shortener Design** | `System design URL shortener, base62, tinyurl design` | ByteByteGo + Alex Xu Ch.8 |
| **Chat System Design** | `System design chat whatsapp, websocket vs long polling, fanout` | ByteByteGo - Chat System + Alex Xu Ch.12 |
| **Notification System** | `System design notification system, push vs pull, fanout` | ByteByteGo - Notification System |
| **CDN** | `CDN how it works, edge server, cache hit miss, CloudFront` | Cloudflare Learning - What is CDN? |
| **Stateless vs Stateful** | `Stateless vs stateful architecture, sticky sessions vs JWT, session store redis` | MDN - Stateless + ByteByteGo |
| **API Gateway** | `API gateway pattern, service discovery eureka consul, gateway vs load balancer` | Nginx - API Gateway + Microservices.io |
| **Circuit Breaker** | `Circuit breaker pattern, retry exponential backoff, bulkhead pattern Resilience4j` | Martin Fowler - Circuit Breaker + Resilience4j Docs |
| **Sharding Strategies** | `Database sharding strategies, shard key hot spot, range vs hash sharding, resharding` | ByteByteGo - Sharding + [[z-REST]] اللي عندك |
| **Observability** | `Observability logging metrics tracing, ELK stack, Prometheus Grafana, Jaeger` | Google SRE Book - Monitoring |
| **SQL vs NoSQL Types** | `SQL vs NoSQL, document key-value column graph database, when to use mongodb vs cassandra` | MongoDB Docs + Cassandra Docs |

---

# ✨ ملاحظات أخيرة عن الجودة

> [!tip]
> **إيه الحلو في النوتس بتاعتك؟**
> - اللغة بسيطة ومصرية وبتوصل المعلومة بسرعة (أحسن من إنجليزي أكاديمي)
> - أمثلة من مشروعك (SkillHub/Dream Shop) بتربط النظري بالعملي
> - كود Spring Boot حقيقي + رسومات ASCII/mermaid بتخلي المعلومة تثبت
> - تغطية Idempotency و Rate Limiting و SLI/SLO بمستوى Senior مش Junior

> [!warning]
> **إيه اللي محتاج تحسين سريع؟**
> - ادمج الملفات المكررة (LB + Idempotency) عشان متتشتتش
> - حول صور [[17-SystemDesign step by step]] لنص قابل للبحث
> - صحح Typos (`Reblication`, `payements`, `00.0%`)
> - أي موضوع جديد اعمله بنفس الستايل: عنوان بإيموجي + `---` + شرح عربي + كود + جدول + خلاصة للانترفيو

---

## 🔗 ملفات للمراجعة السريعة

لو عايز تراجع قبل الانترفيو، ابدأ بدول (ترتيب الأهمية):

1. [[7-Performance]] (Latency/Throughput/P99)
2. [[10-Fault tolerance & High Availability]] (SPOF/Redundancy)
3. [[13.1 LB]] (LB Types/Algorithms)
4. [[14-Relational Databases & ACID]] + [[15-ACID]] (ACID/Isolation)
5. [[18-DB Reblication]] + [[21-Replication Types, Lag , Data Consistency Across Shards]] (Replication)
6. [[22-Rate Limiting]] (Algorithms)
7. [[23-Message Queues]] (Kafka vs RabbitMQ)
8. [[11-SLI, SLO, SLA]] (SRE Basics)
9. [[30-Api Versioning]] (Versioning)
10. [[25-Idempotency exp]] (Idempotency)

> وبعدها كمل النواقص اللي فوق 👆

---

> [!abstract]
> **الخلاصة النهائية:** النوتس بتاعتك **ممتازة كأساس (60-65%)** وتشرح النظري بأسلوب سهل وممتع، بس **مش كافية لوحدها لانترفيو جونيور** من غير ما تكمل الـ 7 مواضيع الـ High Priority + تحل 2-3 أنظمة كاملة (URL Shortener أهم واحد). لو كملتهم بالترتيب اللي فوق، هتبقى جاهز بنسبة 95% لأي انترفيو Junior System Design في شركات زي Vodafone/Fawry/Startups.

*تاريخ التقرير: 2026-10-04 | مبني على قراءة كاملة لـ 35 ملف في فولدر `System Design`*

