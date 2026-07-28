تمام، دي من أهم أجزاء أي Interview Backend Java/Spring. هديك **الزتونة** لكل Topic بحيث تقدر تجاوب في الانترفيو حتى لو السؤال اتلف ودار.

---

# 1) Dependency Injection (DI) & IoC Container

## الزتونة

### الأول يعني إيه IoC؟

قبل Spring كنت بتعمل كده:

```java
UserRepository repo = new UserRepository();
UserService service = new UserService(repo);
```

يعني أنت بنفسك مسؤول عن إنشاء الـ Objects وربطها ببعض.

Spring قالك:

> "سيب الموضوع عليا."

هو اللي هيعمل الـ Objects وهيحقنها لبعض.

وده اسمه

> **Inversion of Control (IoC)**

يعني الـ Control انتقل منك إلى Spring Container.

---

### طيب يعني ايه Dependency Injection؟

لو عندي

```java
UserService
```

بيعتمد على

```java
UserRepository
```

بدل ما يعمل

```java
new UserRepository();
```

Spring يبعتهوله.

يعني

Before

```java
UserRepository repo = new UserRepository();
```

After

```java
@Service
class UserService{

    private final UserRepository repo;

    UserService(UserRepository repo){
        this.repo = repo;
    }

}
```

Spring يشوف Constructor ويقول:

"أنا عندي Bean من UserRepository"

ويروح يحقنه.

وده اسمه

Dependency Injection.

---

### أنواع الـ Injection

#### Constructor Injection ✅ (الأفضل)

```java
public UserService(UserRepository repo)
```

الأفضل لأنه

- Immutable
    
- Testable
    
- Required dependencies
    

---

#### Setter Injection

```java
setRepo(...)
```

للـ Optional dependencies.

---

#### Field Injection

```java
@Autowired
private UserRepository repo;
```

مش مفضل.

---

### IoC Container بيعمل إيه؟

هو المسؤول عن

- إنشاء الـ Beans
    
- تخزينها
    
- حقنها
    
- تدميرها
    

---

### Interview Questions

### الفرق بين IoC و DI؟

الإجابة الذهبية

> IoC هو المبدأ، أما Dependency Injection فهو أشهر طريقة Spring بيطبق بيها الـ IoC.

---

### ليه Constructor Injection أفضل؟

- immutable
    
- testing
    
- required dependency
    
- يمنع Null
    

---

### يعني إيه Bean؟

أي Object Spring هو اللي أنشأه.

---

### هل كل Object Bean؟

لا.

لو عملت

```java
new User()
```

مش Bean.

---

# 2) Spring Boot Auto Configuration

وده سؤال بيتكرر كتير جدًا.

---

## الزتونة

زمان كنت بتكتب

٢٠٠ سطر Config.

Spring Boot قالك:

أنا هعملهم أوتوماتيك.

---

مثلاً

أول ما تضيف

```xml
spring-boot-starter-web
```

تلاقي

- DispatcherServlet
    
- Tomcat
    
- Jackson
    
- ObjectMapper
    

كلهم اشتغلوا.

أنت معملتش حاجة.

---

### بيعملها إزاي؟

ببساطة

لما البرنامج يبدأ

```
@SpringBootApplication
```

فيها

```
@EnableAutoConfiguration
```

ودي تبدأ تبحث داخل

```
META-INF/spring/
```

عن ملفات Auto Configuration.

كل Starter بيجيب Config خاص بيه.

---

مثلاً

لو لقت

```
DataSource.class
```

تقول

```
اعمل DataSource Bean
```

---

لو لقت

```
Tomcat
```

تقول

```
اعمل Embedded Server
```

---

كل ده باستخدام

```
@Conditional
```

---

أشهرهم

```
@ConditionalOnClass
```

لو الكلاس موجود.

---

```
@ConditionalOnMissingBean
```

لو المستخدم معملش Bean.

---

```
@ConditionalOnProperty
```

لو Property موجود.

---

### لو أنا عملت Bean؟

Spring يقول

"تمام"

مش هيعمل واحد تاني.

---

### مثال

لو كتبت

```java
@Bean
ObjectMapper mapper()
```

هيستخدم بتاعك.

---

### Interview Questions

كيف يعمل Auto Configuration؟

الجواب:

- Starter
    
- AutoConfiguration
    
- Conditional annotations
    
- Bean registration
    

---

ليه Spring Boot سريع؟

لأنه بيعتمد على

Convention over Configuration.

---

# 3) Spring Data JPA

---

## الزتونة

هي Layer فوق Hibernate.

بدل ما تكتب DAO

تكتب

```java
interface UserRepository
```

وخلاص.

---

### Repository

```java
interface UserRepository extends JpaRepository<User,Long>
```

تاخد

- save
    
- delete
    
- findAll
    
- findById
    

مجانًا.

---

### Derived Queries

```java
findByEmail()
```

Spring يولد SQL.

---

### JPQL

```java
@Query()
```

---

### Relationships

OneToOne

OneToMany

ManyToOne

ManyToMany

لازم تبقى فاهم مين الـ Owner ومين الـ Inverse Side.

---

### Lazy Loading

يعني

مش يحمل الـ Relation.

إلا لما أطلبها.

---

### Eager Loading

يحملها فورًا.

---

### ليه Lazy أفضل؟

لأنه يمنع تحميل بيانات ملهاش لازمة.

---

### أشهر مشكلة

N+1

---

سببها

```
Lazy
```

مع Loop.

---

حلولها

- Fetch Join
    
- EntityGraph
    
- DTO Projection
    

---

### Interview Questions

ليه OneToMany Lazy؟

لمنع تحميل آلاف Records.

---

ليه ManyToOne Eager؟

لأنها غالبًا Object واحد.

(لكن كثير من الفرق تغيّره إلى Lazy لتقليل الاستعلامات غير الضرورية.)

---

ما هو N+1؟

سؤال ثابت.

---

# 4) Spring Security

---

## الزتونة

هو المسؤول عن حماية التطبيق.

---

يقسم إلى

Authentication

مين أنت؟

---

Authorization

مسموحلك تعمل إيه؟

---

### Authentication

المستخدم يدخل

```
username
password
```

Spring يتحقق.

---

لو صح

ينشئ

```
Authentication Object
```

ويحطه داخل

```
SecurityContext
```

---

### Authorization

بعدها يبدأ يسأل

هل عنده

```
ROLE_ADMIN
```

ولا

```
ROLE_USER
```

---

### JWT

بدل Session

نستخدم Token.

---

Flow

Login

↓

Server

↓

Generate JWT

↓

Client

↓

Authorization Header

↓

Server validates

↓

يفتح الطلب.

---

### JWT يتكون من

Header

Payload

Signature

---

### ليه Signature؟

عشان يمنع التعديل.

---

### Interview Questions

Authentication vs Authorization؟

أشهر سؤال.

---

ليه JWT Stateless؟

لأن السيرفر لا يحتفظ بحالة المستخدم، وكل المعلومات اللازمة للتحقق موجودة داخل الـ Token (أو يمكن التحقق منها من خلاله).

---

ليه JWT أفضل للـ REST APIs؟

لأنه Stateless وسهل الاستخدام مع الخدمات الموزعة.

---

# 5) Transaction Management

---

## الزتونة

Transaction

يعني

كل العمليات تنجح

أو

كلها تفشل.

---

مثال

تحويل فلوس

خصم

إضافة

---

لو الخصم نجح والإضافة فشلت

يبقى لازم Rollback.

---

### @Transactional

```java
@Transactional
```

Spring يعمل

```
Begin

↓

Commit

↓

Rollback
```

---

### Propagation

بيعرف

لو فيه Transaction موجودة

أعمل إيه؟

---

Required

استخدم الحالية.

---

Requires_New

اعمل واحدة جديدة.

---

Mandatory

لازم الحالية موجودة.

---

Never

ارفض لو فيه واحدة.

---

Supports

اشتغل بيها لو موجودة.

---

### Isolation

لمنع مشاكل الـ Concurrent Transactions.

أشهرهم

Read Committed

Repeatable Read

Serializable

---

### أشهر مشاكل

Dirty Read

Non-repeatable Read

Phantom Read

---

### Interview Questions

ليه نستخدم @Transactional؟

لضمان Atomicity واتساق البيانات.

---

إيه الفرق بين Required و Requires_New؟

Required ينضم للمعاملة الحالية، أما Requires_New فيوقف الحالية مؤقتًا ويبدأ معاملة مستقلة.

---

# 6) Spring AOP

---

## الزتونة

AOP

يعني

تفصل الـ Cross Cutting Concerns

عن الـ Business Logic.

---

زي

Logging

Security

Caching

Transactions

Monitoring

---

بدل ما تكرر

```
log
```

في 200 Method.

اعمل

Aspect.

---

### المصطلحات

Aspect

الكلاس.

---

Advice

الكود.

---

Pointcut

مكان التنفيذ.

---

Join Point

الميثود.

---

Advisor

Pointcut + Advice

---

### أنواع Advice

Before

After

AfterReturning

AfterThrowing

Around

---

Around

هو الأقوى.

لأنه يقدر

يشغل

الميثود

أو

يمنعها

أو

يغير النتيجة.

---

### Interview Questions

الفرق بين Aspect و Advice؟

Aspect هو الكلاس الذي يحتوي على منطق الـ AOP، أما Advice فهو الكود الذي يُنفذ عند نقطة معينة.

---

ليه Around أقوى؟

لأنه يتحكم في تنفيذ الميثود بالكامل (قبلها، وبعدها، ويمكنه منع التنفيذ أو تعديل النتيجة).

---

# 7) Bean Lifecycle & Scopes

---

## الزتونة

أي Bean يمر بمراحل:

```
Instantiation

↓

Dependency Injection

↓

@PostConstruct

↓

Ready

↓

Used

↓

@PreDestroy

↓

Destroyed
```

---

### Scope

Singleton ✅

واحدة فقط طوال عمر التطبيق.

---

Prototype

كل Request لـ Bean ينشئ نسخة جديدة.

---

Request

نسخة لكل HTTP Request.

---

Session

نسخة لكل User Session.

---

Application

نسخة لكل ServletContext.

---

### Singleton

هو الـ Default.

---

### Prototype

Spring ينشئ الـ Bean ويحقنه، لكنه لا يدير مرحلة تدميره بعد ذلك.

---

### Interview Questions

ما هو الـ Default Scope؟

Singleton.

---

متى تستخدم Prototype؟

عندما تحتاج كائنًا جديدًا في كل مرة يتم طلبه، مثل كائنات تحمل حالة (Stateful Objects).

---

# 🔥 أهم 15 سؤال انترفيو من كل التوبيكس

1. ما الفرق بين IoC و Dependency Injection؟
    
2. لماذا Constructor Injection أفضل من Field Injection؟
    
3. كيف يعمل Spring Boot Auto Configuration داخليًا؟
    
4. ما هي `@ConditionalOnMissingBean`؟
    
5. ما الفرق بين Lazy Loading و Eager Loading؟
    
6. ما هي مشكلة N+1 وكيف تحلها؟
    
7. ما الفرق بين Authentication و Authorization؟
    
8. كيف يعمل JWT من لحظة تسجيل الدخول حتى تنفيذ الطلب؟
    
9. لماذا يعتبر JWT Stateless؟
    
10. ماذا تفعل `@Transactional`؟
    
11. ما الفرق بين `Propagation.REQUIRED` و `REQUIRES_NEW`؟
    
12. ما هي مستويات الـ Isolation وما المشكلات التي تمنعها؟
    
13. ما الفرق بين Aspect و Advice و Pointcut و Advisor؟
    
14. اشرح دورة حياة الـ Bean داخل Spring Container.
    
15. ما الفرق بين Singleton و Prototype Scope؟
    

إذا كنت داخل على **Senior Junior / Mid-Level Spring Boot Interview**، فهذه الموضوعات تمثل تقريبًا **70–80% من الأسئلة النظرية**، لكن أنصح أيضًا بفهم **"كيف تعمل من الداخل"** وليس مجرد حفظ التعريفات، لأن هذا ما يميز الإجابات القوية في المقابلات.