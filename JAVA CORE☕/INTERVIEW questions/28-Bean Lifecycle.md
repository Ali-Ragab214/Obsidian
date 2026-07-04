ممتاز، دول من أشهر أسئلة الـ Spring Interviews، وغالبًا بييجوا ورا بعض لأنهم مرتبطين ببعض جدًا.

---

#  Explain the Bean Lifecycle

## أولًا: يعني إيه Bean؟

الـ Bean هو ببساطة:

> Object يتم إنشاؤه وإدارته بواسطة Spring Container.

مثال:

```java
@Service
public class UserService {
}
```

الـ `UserService` هنا Bean.

---

# دورة حياة الـ Bean

```text
1. Spring starts
        ↓
2. Bean Definition is loaded
        ↓
3. Bean instance is created
        ↓
4. Dependencies are injected
        ↓
5. Initialization callbacks execute
        ↓
6. Bean becomes ready to use
        ↓
7. Application runs
        ↓
8. Context shuts down
        ↓
9. Destroy callbacks execute
```

---

# Diagram

```text
┌───────────────────────────────┐
│ Spring Container Starts       │
└──────────────┬────────────────┘
               ↓
      Bean Definition Loaded
               ↓
      Bean Instance Created
               ↓
      Dependency Injection
               ↓
      @PostConstruct
      afterPropertiesSet()
      initMethod()
               ↓
         Bean Ready
               ↓
        Application Runs
               ↓
      @PreDestroy
      destroy()
      destroyMethod()
               ↓
     Bean Removed From Memory
```

---

# Step 1: Bean Definition Loading

Spring يعمل Scan للكلاسات:

```java
@Service
@Repository
@Component
@Controller
@Configuration
```

ويخزن معلومات عنها داخل الـ Container.

---

# Step 2: Bean Instantiation

Spring يعمل:

```java
UserService service = new UserService();
```

لكن مش أنت اللي بتعمل `new`.

---

# Step 3: Dependency Injection

مثال:

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring يعمل:

```java
UserRepository repo = new UserRepository();
UserService service = new UserService(repo);
```

---

# Step 4: Initialization Phase

بعد الـ Injection مباشرة.

هنا Spring ينفذ:

## 1) `@PostConstruct`

```java
@PostConstruct
public void init() {
    System.out.println("Bean initialized");
}
```

---

## 2) `InitializingBean`

```java
public class UserService implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Initialized");
    }
}
```

---

## 3) initMethod

```java
@Bean(initMethod = "init")
public UserService userService() {
    return new UserService();
}
```

---

# Step 5: Bean Ready

الـ Bean جاهز للاستخدام:

```java
@Autowired
private UserService service;
```

---

# Step 6: Destruction Phase

عند إغلاق التطبيق:

Spring ينفذ:

## 1)

```java
@PreDestroy
public void cleanup() {
}
```

---

## 2)

```java
public class UserService implements DisposableBean {

    @Override
    public void destroy() {
    }
}
```

---

## 3)

```java
@Bean(destroyMethod = "close")
```

---

# أشهر سؤال Interview

## What is the order?

الإجابة:

```text
Constructor
↓
Dependency Injection
↓
@PostConstruct
↓
Bean Ready
↓
@PreDestroy
```

---

# مثال كامل

```java
@Service
public class UserService {

    public UserService() {
        System.out.println("Constructor");
    }

    @PostConstruct
    public void init() {
        System.out.println("PostConstruct");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Destroy");
    }
}
```

Output:

```text
Constructor
PostConstruct
Application Running
Destroy
```

---

---

# 42) How are Beans Created?

في Spring عندك 3 طرق رئيسية.

---

# 1) Stereotype Annotations (الأكثر استخدامًا)

مثال:

```java
@Service
public class UserService {
}
```

أو:

```java
@Repository
public class UserRepository {
}
```

أو:

```java
@Controller
public class UserController {
}
```

Spring يعمل Component Scan ويحولهم Beans.

---

## أشهر الـ Stereotypes

|Annotation|الاستخدام|
|---|---|
|`@Component`|Generic Bean|
|`@Service`|Business Logic|
|`@Repository`|Database Layer|
|`@Controller`|MVC Controller|
|`@RestController`|REST API|
|`@Configuration`|Configuration Class|

---

# Diagram

```text
@ComponentScan
      ↓
Find Classes
      ↓
Create Bean
      ↓
Store in Container
```

---

# 2) Using `@Bean`

تستخدم لما:

- الكلاس من Library خارجية
    
- لا تستطيع تعديل الكلاس وإضافة `@Component`
    

مثال:

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Spring يسجل الـ ObjectMapper كـ Bean.

---

## Diagram

```text
@Configuration
      ↓
@Bean Method
      ↓
Create Object
      ↓
Register Bean
```

---

# مثال مشهور جدًا

```java
@Bean
public BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

ثم:

```java
@Service
public class UserService {

    private final BCryptPasswordEncoder encoder;

    public UserService(BCryptPasswordEncoder encoder) {
        this.encoder = encoder;
    }
}
```

---

# 3) XML Configuration (قديم جدًا)

قبل Spring Boot كان يتم إنشاء الـ Beans هكذا:

```xml
<bean id="userService"
      class="com.example.UserService"/>
```

---

أو:

```xml
<bean id="repo"
      class="com.example.UserRepository"/>
```

---

ثم:

```xml
<property name="repository" ref="repo"/>
```

---

الآن نادر جدًا استخدامه.

---

# مقارنة

|Method|مستخدم اليوم؟|الأفضل|
|---|---|---|
|Stereotypes|✅ نعم|✅|
|@Bean|✅ نعم|✅|
|XML|❌ نادر جدًا|❌|

---

# متى أستخدم `@Bean` بدل `@Component`؟

## استخدم `@Component`

لو الكلاس بتاعك أنت:

```java
@Service
class UserService {}
```

---

## استخدم `@Bean`

لو الكلاس من مكتبة خارجية:

```java
@Bean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

---

# إجابة إنترفيو قصيرة

## Bean Lifecycle

> Spring creates the bean, injects its dependencies, calls initialization callbacks such as `@PostConstruct`, keeps it available during the application lifecycle, and finally invokes destruction callbacks like `@PreDestroy` when the context shuts down.

---

## How are Beans Created?

> Beans can be created using stereotype annotations such as `@Service` and `@Component`, Java configuration using `@Bean`, or XML configuration, although XML is rarely used in modern Spring applications.