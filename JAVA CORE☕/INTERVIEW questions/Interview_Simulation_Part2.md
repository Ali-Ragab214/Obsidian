تمام، دلوقتي إحنا في **Java Backend Interview**، وأنا الـ Interviewer.

الجزء ده **Part2: Spring + JPA/Hibernate + DB/SQL**.

نفس النظام: سؤال -> إجابة مثالية -> تفاصيل -> follow-up -> هو بيختبر إيه.

---

# Section 1: Spring Core

---

# السؤال الأول

> **What is a Spring Bean? What is IoC and DI?**

إجابة الانترفيو:

> **A Bean is a class instance created and managed by Spring's IoC container. IoC inverts control of creation to the container, and DI implements it by injecting dependencies instead of `new`.**

يعني ببساطة...

من غير Spring:

```java
class Car {
    Engine e = new Engine(); // Car لازقة في Engine، tight coupling
}
```

عايز ت test؟ عايز تغير Engine؟ لازم تعدل الكود.

مع Spring:

```java
@Service
class Car {
    private final Engine engine;
    Car(Engine engine) { this.engine = engine; } // حد يجيبهولي من بره
}
```

```
Class + new = plain object
Class + Spring management = Bean
```

## أنواع الـ Injection

1. **Constructor (المفضل):**

```java
Car(Engine e) { this.e = e; }
```

2. **Setter:**

```java
@Autowired
void setEngine(Engine e) { this.e = e; }
```

3. **Field:**

```java
@Autowired Engine e; // سريع بس وحش في التست ومش immutable
```

ليه Constructor أحسن؟

- `final` + immutable reference.
- valid state من أول لحظة.
- سهل test (new Car(mock)).
- thread-safe للـ reference.

## IoC vs DI؟

- **IoC** = المبدأ (مين بيخلق الـ objects؟ الـ container مش أنت).
- **DI** = التنفيذ (إزاي؟ حقن عبر ctor/setter/field).

**Follow-up:**

> **What is ApplicationContext? Difference with BeanFactory?**

> **ApplicationContext is the IoC container: creates beans, injects, manages lifecycle, plus events/i18n/resources. BeanFactory is the minimal lazy version.**

الـ Context بيتخلق هنا:

```java
SpringApplication.run(App.class, args);
```

تخيله مدير الشركة: بيخلق الموظفين (Beans)، يوزع الشغل (DI)، ويتابع مين دخل ومين خرج (lifecycle).

**Follow-up:**

> **How are beans created? What is the lifecycle?**

3 طرق:

```java
@Service / @Component / @Repository // auto-scan (الأشهر)
@Bean في @Configuration // للـ external libs زي RestTemplate
// XML (legacy، نادر)
```

الـ lifecycle:

```text
Definition -> Instantiation -> DI -> init (@PostConstruct) -> Ready -> destroy (@PreDestroy)
```

```java
@PostConstruct void init() { // بعد الحقن }
@PreDestroy void down() { // قبل القفل }
```

**What the interviewer is really testing:** فاهم ليه بنستخدم Spring أصلاً (coupling/testing) ولا حافظ annotations.

---

# السؤال التاني

> **What is the difference between `@Component`, `@Service`, `@Repository`, `@Controller`? Are they stereotypes?**

إجابة الانترفيو:

> **Yes, all are stereotypes via meta-annotation `@Component`. They make classes beans, plus extras: `@Repository` adds exception translation, `@Service` marks business logic.**

القاعدة:

```text
كل Service/Repo/Controller هو Component، والعكس مش شرط.
```

مثال شركة:

```
@Component = موظف
@Service = محاسب (business)
@Repository = مخزن (DB + ترجمة exceptions)
@Controller = استقبال (HTTP)
```

**Follow-up:**

> **What about `@RestController`? Which of `@Controller`, `@ResponseBody`, `@Component` are part of it?**

```java
@RestController = @Controller + @ResponseBody
@Controller = @Component (transitively)
```

يعني `@RestController` فيها التلاتة منطقياً، لكن `@ResponseBody` لوحدها مش Bean، دي marker بتقول "رجع JSON مش View".

```java
@RestController
class UserController {
    @GetMapping("/u") User get() { return user; } // JSON تلقائي
}
```

**What the interviewer is really testing:** فاهم الـ meta-annotations ولا بتستخدمهم بالبركة.

---

# السؤال التالت

> **Difference between `@Component` (class-level) and `@Bean` (method-level)?**

```java
@Component
class EmailService { } // كلاسك أنت، Spring يلقطه بالـ scan
```

```java
@Configuration
class AppConfig {
    @Bean
    RestTemplate rest() { return new RestTemplate(); } // كلاس جاهز مش بتاعك
}
```

القاعدة: كلاسك -> `@Component`، كلاس مكتبة خارجية -> `@Bean` في `@Configuration`.

**Follow-up: `@Value` vs `@ConfigurationProperties`؟**

- `@Value("${x}")` لحاجة واحدة.
- `@ConfigurationProperties(prefix="app")` لمجموعة related (cleaner).

**What the interviewer is really testing:** بيفرق بين auto-scan و manual bean definition.

---

# السؤال الرابع

> **What did Spring Boot add over traditional Spring? What are Starters?**

إجابة الانترفيو:starter dpendency,auto config,embeded server 

> **Boot is Framework + AutoConfiguration + Starters + Embedded server + Actuator, with convention-over-configuration so you run `java -jar` without XML.**

قبل:

```xml
<!-- DispatcherServlet + DataSource + JPA + Transaction + 200 سطر -->
```

بعد:

```java
@SpringBootApplication // 3 في 1
class App { public static void main(String[] a) { SpringApplication.run(App.class,a); } }
```

الـ 3 في 1:

```
@SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
```

## Starters

Starter = شنطة dependencies جاهزة:

```xml
spring-boot-starter-web <!-- MVC + Jackson + Tomcat + Validation -->
spring-boot-starter-data-jpa
spring-boot-starter-security
```

من غير parent كنت هتكتب versions بإيدك، مع `starter-parent` الـ versions متظبطة.

**Follow-up:**

> **What happens if you remove `@SpringBootApplication` from Main?**

تبقى `main()` عادية:

- مفيش context ولا scan ولا beans.
- مفيش auto-config (DataSource/JPA/Redis مش هيتظبطوا).
- مفيش embedded Tomcat.

**Follow-up:**

> **What happens internally when Boot starts? (14 خطوة مختصرة)**

```text
JVM -> main -> SpringApplication.run
 -> نوع App (servlet/reactive)
 -> ApplicationContext يتخلق
 -> properties تتحمل
 -> scan + BeanDefinitions
 -> AutoConfig (@ConditionalOnClass...)
 -> singletons + DI
 -> @PostConstruct
 -> embedded Tomcat + DispatcherServlet
 -> ReadyEvent (Started!)
```

**What the interviewer is really testing:** عارف Boot وفرت عليك إيه فعلاً ولا فاكرها Framework مختلفة.

---

# Section 2: Spring AOP + Proxy + Transactions (أهم حتة Senior)

---

# السؤال الأول

> **What is Spring Proxy? JDK vs CGLIB?**

بص يا علي، لو فهمت الـ Proxy هتفهم `@Transactional` و `@Async` و `@Cacheable` و AOP كلهم.

إجابة الانترفيو:

> **A Proxy is an object between client and real bean that adds behavior before/after. JDK proxy if bean implements interface, else CGLIB subclass.**

الشكل الطبيعي:

```text
Client -> Real Object (مباشر)
```

مع Spring:

```text
Client -> Proxy -> Real Object
              + open tx / check auth / log
```

مثال:

```java
@Service
class PaymentService {
    public void pay() { System.out.println("pay..."); }
}
```

الـ Proxy شبه كده منطقياً:

```java
class Proxy extends PaymentService {
    void pay() {
        openTx();
        super.pay();
        commitTx();
    }
}
```

- لو فيه interface -> **JDK dynamic proxy**.
- لو مفيش -> **CGLIB subclass**.

**What the interviewer is really testing:** دي البوابة بتاعة السؤال الجاي، لو وقعت هنا هتقع في اللي بعده.

---

# السؤال التاني

> **What happens when a `@Transactional` method calls another `@Transactional` method in the same bean? And why?**

ده أشهر Senior Spring سؤال.

```java
@Service
class OrderService {
    @Transactional
    public void place() {
        saveOrder();
        this.audit(); // self-invocation!
    }
    @Transactional(propagation = REQUIRES_NEW)
    public void audit() { ... }
}
```

الإجابة:

> **It stays in one transaction. Because `this.audit()` bypasses the proxy, so `REQUIRES_NEW` is ignored.**

ليه؟

```text
Client -> Proxy(place) -> Real.place()
                              |
                         this.audit() (مباشر، مفيش Proxy!)
```

الـ Proxy مشافش الـ call التاني أصلاً.

الحل: افصل في Bean تاني:

```java
@Service
class AuditService {
    @Transactional(REQUIRES_NEW)
    public void audit() { ... }
}
```

```text
Client -> Proxy(place) -> Real.place() -> Proxy(audit) -> Real.audit()
                                        (suspend + new tx)
```

**Follow-up: Propagation و Isolation؟**

- `REQUIRED` (default): ادخل في الموجودة أو اعمل جديدة.
- `REQUIRES_NEW`: علق القديمة واعمل جديدة (للـ audit).
- Isolation: `READ_COMMITTED` default في Postgres، `REPEATABLE_READ` في MySQL.

**What the interviewer is really testing:** لبست في الـ self-invocation في شغل حقيقي ولا مذاكر نظري.

---

# السؤال التالت

> **How do you handle exceptions globally in Spring Boot?**

الغلط:

```java
// try-catch في كل Controller + 500 خام للمستخدم
```

الصح:

```java
@RestControllerAdvice
class ApiErrors {
    @ExceptionHandler(NotFound.class)
    ResponseEntity<ErrorResponse> handle(NotFound e) {
        return ResponseEntity.status(404).body(new ErrorResponse(e.getMessage()));
    }
    @ExceptionHandler(Exception.class)
    ResponseEntity<ErrorResponse> fallback(Exception e) {
        return ResponseEntity.status(500).body(new ErrorResponse("unexpected"));
    }
}
```

- **What:** centralized mapping من exception لـ HTTP status + body نضيف.
- **Why:** بدل تكرار و raw stacktrace.
- **When NOT:** الـ validation البسيطة (`@Valid`) ليها handler جاهز، متعيدش اختراعه.
- **How Spring:** الـ DispatcherServlet بيبعت الـ exception للـ Advice قبل ما يرجع response.

اعمل base exception واحدة (`DomainNotFound`) وورث منها عشان تتوسع.

**What the interviewer is really testing:** اشتغلت REST API بجد ولا لسه try-catch في كل حتة.

---

# Section 3: JPA / Hibernate

---

# السؤال الأول

> **What is the difference between JPA and Hibernate?**

إجابة الانترفيو:

> **JPA is the spec (`jakarta.persistence`, `EntityManager`), Hibernate is the implementation (Session, dirty-checking, caches, HQL). Code to JPA, Hibernate runs underneath.**

```java
// أنت بتكتب:
@Entity class User { ... }
userRepo.save(u); // Spring Data -> EntityManager -> Hibernate Session -> JDBC -> DB
```

الـ Stack:

```text
Spring Data -> EntityManager (JPA) -> Session (Hibernate) -> JDBC -> Driver -> DB
```

**What the interviewer is really testing:** أول سؤال JPA، لو قلت "هما نفس الحاجة" بتقع.

---

# السؤال التاني

> **Explain JPA relations: `mappedBy`, `@JoinColumn`, Cascade, Fetch?**

```java
@Entity class Dept { @OneToMany(mappedBy="dept") List<Emp> emps; }
@Entity class Emp { @ManyToOne User u; @JoinColumn(name="dept_id") Dept dept; }
```

- **Owner side** هو اللي عنده الـ FK (`Emp`)، التاني `mappedBy` (غير مالك).
- `@JoinColumn(name="...")` اسم عمود الـ FK.
- **Cascade:** هل العملية تسمع في اللي related؟ (`PERSIST/REMOVE/ALL`).
- **Fetch:** `LAZY` (حمّل عند الاستخدام) vs `EAGER` (حمّل فوراً).

Defaults اللي لازم تحفظها:

```
@ManyToOne / @OneToOne -> EAGER (خطر N+1)
@OneToMany / @ManyToMany -> LAZY
```

نصيحة: خلي كله LAZY واعمل fetch صريح عند الحاجة.

Many-to-Many بجدول وسيط فيه columns زيادة؟ اعمل Entity وسيط (`Enrollment`) بدل `@ManyToMany` مباشرة.

**Follow-up: What is N+1? How to fix without EAGER?**

```java
findAll(); // 1 query للـ users
u.getOrders(); // N queries (لو 1000 user = 1001!)
```

الحل مش EAGER (هيحمل دايماً حتى لو مش محتاج).

الحل:

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();

@EntityGraph(attributePaths="orders")
List<User> findAll();
```

واحد query بدل 1001، والـ mapping يفضل LAZY.

**What the interviewer is really testing:** دي اللي بتفرق Backend شغال production من مذاكر نظري.

---

# السؤال التالت

> **What is `EntityManager`? What is First-Level Cache? What is Dirty Checking?**

التلاتة حاجة واحدة من 3 زوايا.

## EntityManager

هو الـ gateway بتاع JPA للـ Persistence Context:

```java
persist() // جديد -> Managed
find() // هات من Context أو DB
merge() // Detached -> Managed
remove() // -> Removed
flush() // ابعت SQL دلوقتي
clear()/detach() // اخرج من الـ Context
```

## First-Level Cache

Cache تلقائي per-Session جوه الـ Persistence Context:

```java
em.find(User.class,1); // SELECT
em.find(User.class,1); // من الـ Cache، مفيش SELECT
```

بيتمسح مع قفل الـ Session. مختلف عن Second-Level (shared across sessions، محتاج config).

## Dirty Checking

الـ Context محتفظ بالـ Entity + Snapshot:

```java
@Transactional
void rename(Long id) {
    User u = repo.findById(id).orElseThrow();
    u.setName("Ali"); // مفيش save()!
} // عند commit: قارن -> dirty -> UPDATE تلقائي
```

```text
Managed Entity + Snapshot
 ↓ flush/commit
قارن -> dirty؟ آه -> UPDATE
```

- **When:** فقط للـ Managed جوه transaction.
- **When NOT:** لو Detached أو readOnly مش هيحصل.
- **Cost:** مقارنة كل flush، للـ bulk استخدم query مباشرة.

**What the interviewer is really testing:** فاهم ليه أحياناً بتنسى `save()` والحاجة بتتحفظ لوحدها.

---

# السؤال الرابع

> **What is JDBC? Why `PreparedStatement`?**

```java
Connection c = DriverManager.getConnection(url,user,pass);
PreparedStatement ps = c.prepareStatement("SELECT * FROM users WHERE id=?");
ps.setLong(1, id);
ResultSet rs = ps.executeQuery();
```

ليه Prepared مش Statement؟

```java
// Statement: "SELECT * FROM users WHERE name='" + input + "'"
// لو input = ' OR '1'='1 -> SQL Injection!
```

الـ Prepared بيعمل bind parameters، أمان + أداء.

الـ Stack الحقيقي: `Spring Data -> Hibernate -> JDBC -> Driver -> DB`.

⚠️ ده سؤال قديم، بيتسأل كـ security follow-up مش كـ Core.

---

# Section 4: DB / Backend Misc

---

# السؤال الأول

> **Pagination: offset vs cursor? Why paginate in DB not memory?**

```sql
-- Offset (سهل، للصفحات الأولى):
SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 40; -- page 3

-- Cursor (للـ large/scale):
SELECT * FROM users WHERE id > 1000 ORDER BY id LIMIT 20;
```

ليه في DB؟

لو عندك مليون row وحملتهم في memory عشان تاخد 20، قتلت الـ Heap والـ GC.

- **Offset:** بسيط لكن بطيء في الصفحات البعيدة + duplicates لو حد insert أثناء التصفح.
- **Cursor:** ثابت وسريع للـ infinite scroll (Twitter/GitHub/Stripe).

في Spring:

```java
Page<User> p = repo.findAll(PageRequest.of(0,20,Sort.by("id")));
```

**What the interviewer is really testing:** اشتغلت high-traffic ولا بتجيب كل الداتا وت filter في Java.

---

# السؤال التاني (Spring عملي)

> **What is `@Builder.Default` in Lombok? Why needed?**

المفاجأة:

```java
@Builder
class User {
    boolean active = true;
    List<String> roles = new ArrayList<>();
}
```

```java
User u = User.builder().build();
// u.active = false! roles = null! (الـ initializer اتطنش)
```

ليه؟

الـ Builder بيبني عبر constructor/package، مش عبر الـ field initializer.

الحل:

```java
@Builder.Default
boolean active = true;
@Builder.Default
List<String> roles = new ArrayList<>();
```

Use cases: flags/status/collections/counters/dates.

⚠️ سؤال Lombok عملي، الشركات اللي بتستخدم Lombok بتسأله.

---

# السؤال التالت

> **Servlet vs Spring? Web Server vs App Server?**

## Servlet vs Spring

- **Servlet:** الـ spec/API اللي شغال جوه Tomcat (`HttpServlet/doGet`)، لكل endpoint servlet مجهدة.
- **Spring MVC:** طبقة فوقه (`DispatcherServlet -> Controller -> Service -> Repo`)، routing و DI و validation جاهزين.

Spring مش بديل، هو layer فوق Servlet (ما عدا WebFlux reactive).

## Web vs App

- **Web (Nginx/Apache):** static files + HTTP.
- **App (WildFly/GlassFish):** business + DB + tx + security (Java EE كامل).
- **Tomcat:** servlet container + web server، مش EE كامل.

الـ Boot بي embed الـ Tomcat وتشغل `java -jar` من غير سيرفر خارجي.

⚠️ أسئلة deployment، اعرفها في سطرين.

---

# السؤال الرابع (Security - Added, مكنش عندك نص)

> **How does Spring Security + JWT work? [Added by agent (not in my original files)]**

عندك كان صورة فقط (`71-How Security work.md`) من غير نص، فدي خلاصة عملية:

```text
Login -> Authenticate (UserDetailsService + PasswordEncoder)
 -> Issue JWT (header.payload.signature)
 -> Client يبعت Authorization: Bearer <token>
 -> Filter يفك التوقيع -> SecurityContext
 -> @PreAuthorize("hasRole('ADMIN')") يسمح/يمنع
```

- **What:** stateless auth للـ REST.
- **Why:** من غير sessions، scale أفقي سهل.
- **When NOT:** للـ server-rendered بسيط session كفاية.
- **How:**

```java
@Bean SecurityFilterChain chain(HttpSecurity h) throws Exception {
    return h.csrf(csrf -> csrf.disable())
        .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
        .authorizeHttpRequests(a -> a.requestMatchers("/auth/**").permitAll().anyRequest().authenticated())
        .build();
}
```

**What the interviewer is really testing:** فاهم الـ filter chain ولا فاكرها magic.

---

# ملاحظات سريعة (❌/⚠️ مش مستاهلة عمق)

- **DB vs DBMS** (`13-...md`): DB = الداتا نفسها، DBMS = السوفتوير (MySQL/Postgres). تعريف بدائي، مش سؤال Java.
- **DELETE/TRUNCATE/DROP:** DELETE=DML صف صف + WHERE + rollback، TRUNCATE=DDL كل الصفوف سريع + reset identity، DROP=DDL يدمر الجدول. سؤال DB عام.
- **`System.exit(1)`:** بيقفل JVM فوراً + shutdown hooks، 0=success. معلومة API.
- **امتداد bytecode `.class`:** بديهية.
- **Dictionary:** obsolete، استخدم Map. متذاكرهوش.
- **Servlet/WebServer/JDBC التفصيلية:** اعرف سطرين بس إلا لو داخل DevOps/DB interview.

---

# الإجابة الذهبية (Part2 - 30 ثانية)

> **Spring manages beans via IoC with constructor injection for immutability, proxies add tx and AOP with self-invocation as the classic trap, Boot adds auto-config and starters over Framework. Global advice centralizes REST errors, pagination belongs in DB with cursor for scale. JPA is the spec with Hibernate underneath, relations use owner plus mappedBy with LAZY by default, N+1 is fixed by JOIN FETCH or EntityGraph, and dirty checking plus first-level cache live inside the persistence context.**

خش على `Interview_Simulation_OOP.md`، ده أهم ملف.
