تمام، هجاوبك كأنك في إنترفيو على Spring Framework وSpring Boot.

---

# 1) `@Component`

## ما هي؟

تقول لـ Spring:

> "أنشئ Object من هذا الكلاس وضعه داخل الـ IoC Container."

بمعنى أن الكلاس يصبح **Spring Bean**.

---

## تستخدم فين؟

تستخدم على مستوى الكلاس (**Class Level Annotation**).

---

## مثال

```java
@Component
public class EmailService {

    public void sendEmail() {
        System.out.println("Email Sent");
    }
}
```

عند تشغيل التطبيق:

```text
Spring Container
    ↓
creates EmailService Bean
```

---

## متى تستخدمها؟

عندما تريد أن يدير Spring عمر الكائن.

غالبًا تستخدم مع:

- Services
    
- Helpers
    
- Utilities
    

---

## ملاحظة مهمة

الـ annotations التالية كلها تعتبر نوعًا من `@Component`:

- `@Service`
    
- `@Repository`
    
- `@Controller`
    
- `@RestController`
    

يعني:

```java
@Service
public class UserService {}
```

تعادل تقريبًا:

```java
@Component
public class UserService {}
```

لكن `@Service` أوضح من ناحية المعنى.

---

# 2) `@Bean`

## ما هي؟

تقول لـ Spring:

> "استدع هذا الـ method وخذ الـ object الذي يرجعه وسجله كـ Bean."

---

## تستخدم فين؟

- **Method Level Annotation**
    
- داخل كلاس عليه `@Configuration`.
    

---

## مثال

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

Spring ينفذ:

```java
RestTemplate rt = restTemplate();
```

ثم يضيفه للـ Container.

---

## لماذا لا نستخدم Component هنا؟

لأنك لا تملك كلاس `RestTemplate`.

أنت تنشئه بنفسك.

---

## أشهر استخدامات `@Bean`

- RestTemplate
    
- PasswordEncoder
    
- ModelMapper
    
- ObjectMapper
    
- SecurityFilterChain
    

مثال:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

# 3) `@Configuration`

## ما هي؟

تعرف الكلاس على أنه:

> كلاس مسؤول عن تعريف الـ Beans.

---

## تستخدم فين؟

- **Class Level Annotation**
    

---

## مثال

```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

## بدون Configuration؟

في بعض الحالات سيعمل، لكن Spring لن يتعامل مع الكلاس على أنه مصدر إعدادات رسمي.

---

# 4) `@ComponentScan`

## ما هي؟

تطلب من Spring البحث عن:

```text
@Component
@Service
@Repository
@Controller
```

داخل Packages معينة.

---

## تستخدم فين؟

- Class Level
    

---

## مثال

```java
@ComponentScan(
    basePackages = {
        "com.skillhub.services",
        "com.skillhub.repositories"
    }
)
```

---

## ماذا يحدث بدونها؟

في Spring Boot:

```java
@SpringBootApplication
```

تحتوي ضمنيًا على:

```java
@ComponentScan
```

لذلك غالبًا لا نكتبها يدويًا.

---

# 5) `@Autowired`

## ما هي؟

تطلب من Spring حقن Dependency.

---

## تستخدم فين؟

- Field Level
    
- Constructor Level
    
- Setter Level
    

---

## Constructor Injection (المفضل)

```java
@Service
public class UserService {

    private final EmailService emailService;

    @Autowired
    public UserService(
            EmailService emailService) {

        this.emailService = emailService;
    }
}
```

في الإصدارات الحديثة:

```java
@Service
public class UserService {

    private final EmailService emailService;

    public UserService(
            EmailService emailService) {

        this.emailService = emailService;
    }
}
```

Spring سيحقنه تلقائيًا.

---

## Field Injection

```java
@Autowired
private EmailService emailService;
```

ممكن لكنه غير مفضل.

---

## Setter Injection

```java
@Autowired
public void setEmailService(
        EmailService emailService) {

    this.emailService = emailService;
}
```

---

# لماذا Constructor Injection أفضل؟

- يسمح باستخدام `final`.
    
- أسهل في الاختبار.
    
- يجعل الـ dependencies واضحة.
    
- يمنع إنشاء Object غير مكتمل.
    

---

# 6) `@Value`

## ما هي؟

تستخدم لقراءة القيم من:

- application.properties
    
- application.yml
    
- Environment Variables
    

---

## تستخدم فين؟

غالبًا Field Level.

---

## مثال

```properties
app.name=SkillHub
```

```java
@Value("${app.name}")
private String appName;
```

القيمة تصبح:

```text
SkillHub
```

---

## مثال آخر

```properties
jwt.secret=my-secret-key
```

```java
@Value("${jwt.secret}")
private String secret;
```

---

# مقارنة سريعة

|Annotation|المستوى|الوظيفة|
|---|---|---|
|`@Component`|Class Level|إنشاء Bean من كلاس|
|`@Bean`|Method Level|إنشاء Bean من Method|
|`@Configuration`|Class Level|تعريف كلاس إعدادات|
|`@ComponentScan`|Class Level|البحث عن Components|
|`@Autowired`|Field / Constructor / Setter|حقن Dependency|
|`@Value`|Field / Constructor Parameter|قراءة القيم من Properties|

---

# `@Component` vs `@Bean`

|`@Component`|`@Bean`|
|---|---|
|Class Level|Method Level|
|Spring ينشئ الـ Object تلقائيًا|أنت تنشئ الـ Object بنفسك|
|يستخدم مع الكلاسات التي تملكها|يستخدم مع كائنات مكتبات خارجية أو إعدادات خاصة|

مثال:

```java
@Service
public class UserService {}
```

استخدم:

```java
@Component
```

أما:

```java
new BCryptPasswordEncoder()
```

فاستخدم:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

# `@Configuration` vs `@Component`

|`@Configuration`|`@Component`|
|---|---|
|كلاس إعدادات|كلاس Business Logic|
|يحتوي غالبًا على Beans|يحتوي على Methods للعمل|
|يستخدم مع Config Classes|يستخدم مع Services و Helpers|

---

# سؤال إنترفيو مشهور

### لماذا `@Service` أفضل من `@Component`؟

لأن:

- الاثنان يسجلان Bean داخل الـ Container.
    
- لكن `@Service` يوضح أن الكلاس يحتوي Business Logic.
    
- يجعل الكود أكثر وضوحًا وتنظيمًا.
    

---

# سؤال أصعب قليلًا

### ما الفرق بين IoC و Dependency Injection؟

- **IoC** هو المبدأ العام:
    
    > الـ Container هو الذي يتحكم في إنشاء الـ Objects.
    
- **Dependency Injection** هو إحدى طرق تنفيذ IoC:
    
    > يقوم الـ Container بحقن الـ Dependencies بدلًا من إنشائها يدويًا.
    

دي إجابة إنترفيو قوية جدًا وغالبًا تعجب أي مهندس Spring.