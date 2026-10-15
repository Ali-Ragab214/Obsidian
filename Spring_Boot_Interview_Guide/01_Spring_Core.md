# Section 01: Spring Core (IoC, DI, Beans & ApplicationContext)

> **الملف ده مبني على مذكراتك في `Spring Core/` و أسئلة الإنترفيو رقم 61 و 62 و 72.**  
> تم ترتيب الأسئلة من الأسهل للأصعب مع أمثلة كود كاملة، مقارنات، وجزء System Design في النهاية.

---

### Q1: What is Inversion of Control (IoC), and why does Spring use it?
**Level:** Junior  
**Related to:** Q2 (Dependency Injection), Q3 (What is a Bean), Q13 (BeanFactory vs ApplicationContext)

**Answer:**
في الـ Java التقليدية (من غير فريم وورك)، أنت كمبرمج كنت بتتحكم في كل دورة حياة الـ Objects:
```java
// أنت اللي بتعمل new وأنت اللي رابط الكلاسات ببعض
Engine engine = new PetrolEngine();
Car car = new Car(engine);
```
لو مشروعك فيه 500 كلاس و 200 dependency، هتلاقي نفسك بتكتب آلاف الـ `new` وتتحكم يدوياً في إيه يتبني قبل إيه، وده بيعمل **Tight Coupling** رهيب وفوضى في الكود.

**Inversion of Control (انعكاس التحكم):**
المفهوم ده معناه: **انعكاس السيطرة**. بدل ما أنت كمطور اللي تنشئ وتدير الـ Objects بنفسك (`new`)، بنسلّم التحكم ده لـ Container خارجي وهو **Spring IoC Container**.
> تشبيه المطبخ 🍳 من مذكراتك:
> - **بدون Spring:** أنت اللي بتطبخ، تقطع، تشتري الأدوات، وتنضف المطبخ.
> - **مع Spring (IoC):** فيه Chef جاهز، يجهز الأدوات، يشتري المكونات، ويقدملك الطبق جاهز للاستخدام مباشرة!

**الفوائد الأساسية للـ IoC:**
1. **Loose Coupling:** الكلاسات مش معتمدة على implementations محددة بل على Interfaces.
2. **Easy Testing:** تقدر تعمل Mock لأي dependency بسهولة في الـ Unit Tests.
3. **Maintainability:** تغيير أي implementation مش بيجبرك تعدل في كل الكلاسات المستهلكة له.

```
بدون Spring (Developer in Control):
Developer ──► new PetrolEngine() ──► new Car(engine)

مع Spring (Inversion of Control):
Developer ──► يكتب الكلاسات والـ Annotations فقط
Spring IoC Container ──► ينشئ الـ Objects ──► يحقن الـ Dependencies
```

**Short answer (30 seconds version):**
> "IoC stands for Inversion of Control. It is a design principle where the control of object creation, lifecycle management, and dependency wiring is transferred from the developer to a container (Spring IoC Container). This promotes loose coupling, testability, and clean architecture."

---

### Q2: What is Dependency Injection (DI) and what is the difference between Loose Coupling and Tight Coupling?
**Level:** Junior  
**Related to:** Q1 (IoC), Q6 (Constructor vs Setter vs Field Injection), Q7 (@Autowired)

**Answer:**
**DI (حقن التبعيات)** هو الـ **Design Pattern** الفعلي اللي Spring بيطبّق بيه مبدأ الـ **IoC**.
بدل ما الـ Object ينشئ الـ dependencies بتاعته جواه، الـ dependencies بتجيله "محقونة" من الخارج (عادة عبر الـ Constructor).

#### Tight Coupling vs Loosely Coupled (من مذكراتك):
**1. Tight Coupling (ترابط قوي - سيء ❌):**
```java
class Car {
    // Car مربوطة مباشرة بـ PetrolEngine وعارفة تفاصيله!
    PetrolEngine engine = new PetrolEngine(); 
    void drive() { engine.start(); }
}
```
*المشكلة:* لو حبيت تحول لـ `DieselEngine` أو `ElectricEngine` لازم تدخل تعدل كود كلاس `Car` نفسه!

**2. Loosely Coupled (ترابط ضعيف - ممتاز ✅):**
بنعمل Interface وسيط، والـ Car بتعتمد على الـ Interface مش على الـ Class:
```java
// 1. Interface يمثل الـ Contract (What the service can do)
public interface Engine {
    void start();
}

// 2. Implementations تمثل التفاصيل (How it does it)
@Component
public class PetrolEngine implements Engine {
    @Override
    public void start() { System.out.println("Petrol Engine Started 🔥"); }
}

// 3. Car معتمدة على الـ Interface فقط وتستقبل الـ Engine من الخارج
@Component
public class Car {
    private final Engine engine; // Loose Coupling!

    public Car(Engine engine) { // Dependency Injection!
        this.engine = engine;
    }

    public void drive() {
        engine.start();
    }
}
```

| المقارنة | Tightly Coupled ❌ | Loosely Coupled (مع Spring DI) ✅ |
| :--- | :--- | :--- |
| **إنشاء الـ Dependency** | الكلاس بينشئ الـ dependency بـ `new` | بتجيله جاهزة من الـ Container |
| **الاعتمادية** | يعتمد على Concrete Class | يعتمد على Interface |
| **سهولة التعديل** | تعديل الـ engine يكسر كود Car | تضيف engine جديد بدون لمس Car نهائياً |
| **قابلية الاختبار (Testing)** | شبه مستحيل تعمل Mock | سهل جداً تمرر MockEngine في الـ Constructor |

**Gotcha / Common Mistake:**
كثير من الناس بيخلطوا ويقولوا "أنا عملت Loose Coupling بـ Interface وخلاص مش محتاج Spring".
الحقيقة: الـ Interface بيحل مشكلة الارتباط فقط، لكنك مازلت هتضطر تكتب `new` يدوي وتربط الكائنات. Spring بيحل المشكلتين معاً: Loose Coupling + Automatic Wiring!

**Short answer (30 seconds version):**
> "Dependency Injection is a pattern implementing IoC, where an object receives its dependencies from an external assembler rather than creating them internally. Loose coupling means decoupling classes using interfaces, allowing us to change implementations or swap them with mocks without touching the dependent class."

---

### Q3: What is a Spring Bean, and how is it managed?
**Level:** Junior  
**Related to:** Q1 (IoC), Q4 (Bean Scopes), Q5 (Bean Lifecycle)

**Answer:**
ببساطة شديدة كما هو مدون في مذكراتك:
> **Bean = Any Java Object created, configured, and managed by the Spring IoC Container.**

أي كلاس عادي في جافا (POJO) مش بيبقى Bean إلا لما Spring يمسكه ويديره في الـ ApplicationContext.

```java
// مجرد Java Object عادي في الـ Heap مش متسجل في Spring:
UserService u = new UserService(); 

// Spring Bean:
@Service
public class UserService {
    // Spring هو اللي أنشأه، ومخزنه في الـ Container، ويحقنه لأي كلاس يطلبه
}
```

#### كيف يتعرف Spring على الـ Beans؟
1. **عبر الـ Stereotype Annotations:** لما تحط `@Component` أو `@Service` أو `@Repository` أو `@Controller`. الـ Component Scanning بيلاقيهم ويعمل منهم Beans.
2. **عبر الـ `@Bean` داخل `@Configuration` Class:** لما تكون بتعرف كلاس خارجي (Third-Party Library) مش مسموحلك تعدل على السورس كود بتاعه لتحط عليه `@Component`.

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(); // Spring هياخد الـ instance دي ويخليها Bean عنده
    }
}
```

**Short answer (30 seconds version):**
> "A Spring Bean is a Java object that is instantiated, assembled, and managed by the Spring IoC Container. Unlike regular objects created via the 'new' keyword, Beans participate in Spring's lifecycle, dependency injection, and AOP proxying."

---

### Q4: What are Spring Bean Scopes? Explain Singleton vs Prototype in detail.
**Level:** Junior / Mid  
**Related to:** Q3 (Bean), Q5 (Lifecycle)

**Answer:**
الـ **Bean Scope** يحدد: **كم نسخة (Instances) هيتم إنشاؤها من الـ Bean ده؟ ودورة حياة كل نسخة عاملة إزاي؟**

#### الـ Scopes المتاحة في Spring:
1. **Singleton (الافتراضي - Default):** نسخة واحدة فقط لكل Spring IoC Container.
2. **Prototype:** نسخة جديدة تماماً في كل مرة تطلب فيها الـ Bean (`context.getBean()` أو كـ dependency).
3. **Request (Web-aware):** نسخة جديدة لكل HTTP Request (خاص بتطبيقات الويب).
4. **Session (Web-aware):** نسخة واحدة تدوم طوال فترة الـ HTTP Session للمستخدم.
5. **Application (Web-aware):** نسخة واحدة طوال دورة حياة الـ `ServletContext`.
6. **Websocket (Web-aware):** نسخة مرتبطة بدورة حياة اتصال الـ WebSocket.

#### مقارنة تفصيلية: Singleton vs Prototype

```java
// Car.java
@Component
@Scope("prototype") // لو شلت السطر ده هيرجع الافتراضي: Singleton
public class Car {
    public Car() {
        System.out.println("Car instance created!");
    }
}
```

لو طلبنا الـ Bean مرتين:
```java
Car car1 = context.getBean(Car.class);
Car car2 = context.getBean(Car.class);
System.out.println(car1 == car2);
```

**لو كان Prototype:**
- الأوت بوت:
  ```
  Car instance created!
  Car instance created!
  false
  ```
- كل مرجع بيشير لـ Object مختلف في الـ Heap.

**لو كان Singleton (Default):**
- الأوت بوت:
  ```
  Car instance created!
  true
  ```
- نفس الـ Object بيتم إعادة استخدامه.

| وجه المقارنة | Singleton (Default) | Prototype |
| :--- | :--- | :--- |
| **عدد الـ Instances** | كائن واحد فقط لكل Container | كائن جديد عند كل استدعاء |
| **وقت الإنشاء** | Eager (عند بدء تشغيل التطبيق افتراضياً) | Lazy (عند طلبه فقط) |
| **استهلاك الذاكرة** | خفيف وموفر للذاكرة | عالي لو تكرر طلبه |
| **الاستخدام المثالي** | Stateless Beans (Services, Repositories, Controllers) | Stateful Beans (تحمل بيانات متغيرة تخص مهمة معينة) |
| **إدارة دورة الحياة والتدمير** | Spring يديره بالكامل حتى استدعاء الـ `destroy()` | Spring ينشئه ويحقنه فقط، ولا ينفذ له الـ `PreDestroy`! |

> ⚠️ **سؤال تريكي مشهور (Gotcha):**
> ماذا يحدث لو حقنت Prototype Bean داخل Singleton Bean؟
> **الإجابة:** الـ Singleton Bean بيتم إنشاؤه مرة واحدة فقط عند بدء التطبيق، وبالتالي الـ Prototype Bean هيتحقن **مرة واحدة فقط** ومش هيتجدد مع كل استدعاء للـ Singleton! 
> الحل: استخدام `@Lookup` method injection أو `ObjectProvider<T>`.

**Short answer (30 seconds version):**
> "Spring provides several bean scopes, the most common being Singleton (default) and Prototype. Singleton means one shared instance per Spring container, perfect for stateless components like Services and Repositories. Prototype creates a brand-new instance every time the bean is requested, ideal for stateful objects."

---

### Q5: What is the Spring Bean Lifecycle? [ADDED - not in my notes]
**Level:** Mid  
**Related to:** Q3 (Bean), Q4 (Scopes), Q18 (Lifecycle Callbacks)

**Answer:**
دورة حياة الـ Bean بتمر بمراحل دقيقة من لحظة قراءة الـ Class وحتى تدميره:

```
1. Instantiation (Constructor call: new Object())
       │
2. Populate Properties (Dependency Injection: Setters / Fields)
       │
3. Aware Interfaces (BeanNameAware, BeanFactoryAware, ApplicationContextAware)
       │
4. BeanPostProcessor: postProcessBeforeInitialization()
       │
5. Initialization (@PostConstruct ──► InitializingBean's afterPropertiesSet() ──► custom init-method)
       │
6. BeanPostProcessor: postProcessAfterInitialization() (هنا بيتعمل الـ AOP Dynamic Proxies!)
       │
7. Bean is Ready for Use! (تطبيقك بيشتغل بيه)
       │
8. Container Shutdown
       │
9. Destruction (@PreDestroy ──► DisposableBean's destroy() ──► custom destroy-method)
```

#### كود يوضح الـ Callbacks الأساسية:
```java
@Component
public class DatabaseConnector implements InitializingBean, DisposableBean {

    public DatabaseConnector() {
        System.out.println("1. Constructor: Bean Instantiated");
    }

    @PostConstruct
    public void customInit() {
        System.out.println("2. @PostConstruct: Executed after DI is complete");
    }

    @Override
    public void afterPropertiesSet() {
        System.out.println("3. InitializingBean interface callback");
    }

    @PreDestroy
    public void customDestroy() {
        System.out.println("4. @PreDestroy: Cleanup connections before shutdown");
    }

    @Override
    public void destroy() {
        System.out.println("5. DisposableBean interface destroy callback");
    }
}
```

> 💡 **معلومة للإنترفيو:** الـ Dynamic Proxies للـ AOP (زي `@Transactional` و `@Async`) بيتم إنشاؤها في مرحلة `BeanPostProcessor.postProcessAfterInitialization()`. لو استدعيت الـ method في الـ Constructor مش هتكون متغطية بالـ Proxy!

**Short answer (30 seconds version):**
> "The Bean lifecycle consists of: Instantiation, Populating Properties (DI), Aware interfaces execution, Pre-initialization BeanPostProcessors, Initialization (@PostConstruct and init methods), Post-initialization BeanPostProcessors (AOP proxying), and finally Destruction (@PreDestroy) upon container shutdown."

---

### Q6: Constructor Injection vs Setter Injection vs Field Injection — Why is Constructor Injection recommended?
**Level:** Junior / Mid  
**Related to:** Q2 (DI), Q7 (@Autowired)

**Answer:**
في Spring فيه 3 طرق لحقن التبعيات:

#### 1. Field Injection (غير محبذة ❌):
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // Field Injection
}
```
- **عيوبها:**
  - مستحيل تخلي الحقل `final` (Immutable).
  - صعب جداً في الـ Unit Testing (مضطر تستخدم Reflection أو `@InjectMocks` لأنك متعرفش تباصيها بالـ `new`).
  - بتخفي التبعيات وممكن تؤدي لـ Circular Dependency خفية بسهولة.

#### 2. Setter Injection (استخدامات خاصة ⚠️):
```java
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
- مناسبة فقط للـ **Optional Dependencies** اللي التطبيق يقدر يشتغل بدونها.

#### 3. Constructor Injection (الـ Best Practice الرسمية ✅):
```java
@Service
public class UserService {
    private final UserRepository userRepository; // Immutable!

    // بدءاً من Spring 4.3، لو فيه Constructor واحد مش محتاج تكتب @Autowired أصلاً!
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```
أو بالاختصار باستخدام Lombok (مذكورة في كورس الملاحظات):
```java
@Service
@RequiredArgsConstructor // بتولد Constructor لجميع الحقول الـ final تلقائياً
public class UserService {
    private final UserRepository userRepository;
}
```

#### مقارنة سريعة:
| المعيار | Constructor Injection ✅ | Field Injection ❌ | Setter Injection ⚠️ |
| :--- | :--- | :--- | :--- |
| **Immutability (`final`)** | متاح (`final fields`) | غير متاح نهائياً | غير متاح |
| **Unit Testing بدون Spring** | سهل جداً (`new UserService(mockRepo)`) | صعب ويحتاج Reflection | ممكن عن طريق استدعاء الـ setter |
| **اكتشاف الـ Circular Dependency** | في الـ Startup فورا (Fail-fast) | ممكن يحصل Run-time error متأخر | يسمح بها |
| **توصية Spring الرسمية** | الموصى بها دائماً | مرفوضة في الكود الحديث | للتبعيات الاختيارية فقط |

**Short answer (30 seconds version):**
> "Constructor injection is the official best practice because it enforces immutability via 'final' fields, prevents NullPointerExceptions by ensuring required dependencies are provided at creation, makes unit testing without Spring effortless, and fails fast on circular dependencies."

---

### Q7: What does `@Autowired` do, and how does Spring resolve dependencies?
**Level:** Junior  
**Related to:** Q6 (Constructor Injection), Q8 (@Qualifier vs @Primary)

**Answer:**
`@Autowired` هي الـ Annotation المسؤولة عن توجيه Spring للبحث عن Bean مناسب في الـ ApplicationContext وحقنه تلقائياً.

#### كيف يبحث Spring عن الـ Bean المناسب؟ (Resolution Steps):
1. **أولاً: البحث بالنوع (By Type):**
   - يدور Spring في الـ Container: هل فيه Bean بيعمل `implements` للـ Interface ده أو من نفس الـ Type؟
2. **لو لقى Bean واحد فقط:** يقوم بحقنه فوراً بنجاح.
3. **لو لقى أكثر من Bean لنفس الـ Type (Ambiguity):**
   - يفحص هل فيه واحد منهم متعلم بـ `@Primary`؟ لو وجد، يختاره.
   - يدور لو فيه `@Qualifier("beanName")` محدد عند مكان الحقن.
   - يدور على تطابق اسم الحقل/المتغير مع اسم الـ Bean (Resolution by Name).
   - لو معرفش يحدد، يرمي الاستثناء الشهير: `NoUniqueBeanDefinitionException`.
4. **لو ملقاش ولا Bean:**
   - يرمي `NoSuchBeanDefinitionException`. (إلا لو كانت `@Autowired(required = false)` فبيسبها `null`).

**Short answer (30 seconds version):**
> "@Autowired enables automatic dependency injection by Spring. Spring first attempts to resolve dependencies by type. If multiple candidate beans of the same type exist, it resolves ambiguity using @Primary, @Qualifier, or falling back to matching the bean name."

---

### Q8: What is the difference between `@Primary` and `@Qualifier`?
**Level:** Junior / Mid  
**Related to:** Q7 (@Autowired), Q2 (Loose Coupling)

**Answer:**
لما يكون عندك Interface واحد وليه كذا Implementation (زي كود الـ `Engine` في مذكراتك: `PetrolEngine` و `ElectricEngine`)، Spring بيحتاج توجيه عشان يعرف يقرر مين اللي يتحقن.

```java
public interface Engine { void start(); }

@Component
public class PetrolEngine implements Engine { ... }

@Component
public class ElectricEngine implements Engine { ... }
```

#### 1. `@Primary` (Default Choice / أولوية عامة):
توضع على مستوى الـ **Class (Implementation)** عشان تقول لـ Spring: "لو حد طلب `Engine` ومحددش اسم معين، اختارني أنا كخيار افتراضي".
```java
@Component
@Primary // الخيار الافتراضي
public class ElectricEngine implements Engine { ... }

@Component
public class Car {
    private final Engine engine;
    public Car(Engine engine) { // هيتحقن تلقائياً ElectricEngine بسبب @Primary
        this.engine = engine;
    }
}
```

#### 2. `@Qualifier` (Specific Choice / اختيار محدد بالاسم):
توضع عند **مكان الحقن (Injection Point)** لتجاوز أي خيار افتراضي وتحديد الـ Bean بالاسم:
```java
@Component
public class Car {
    private final Engine engine;

    // بنقول لـ Spring بالاسم: هات petrolEngine حتى لو غيره واخد @Primary!
    public Car(@Qualifier("petrolEngine") Engine engine) {
        this.engine = engine;
    }
}
```

| المعيار | `@Primary` | `@Qualifier` |
| :--- | :--- | :--- |
| **مكان الاستخدام** | على الـ Bean Class أو `@Bean` method | عند نقطة الاستهلاك (Constructor/Field/Param) |
| **الوظيفة** | يحدد الـ Bean الافتراضي في حالة التعدد | يحدد بدقة اسم الـ Bean المراد حقنه |
| **الأولوية** | أقل أولوية (يتم تجاوزه لو وُجد `@Qualifier`) | أعلى أولوية دائماً |

**Short answer (30 seconds version):**
> "@Primary sets a default bean among multiple candidates of the same type at the declaration level. @Qualifier is specified at the injection point to select a specific bean by its name, overriding @Primary."

---

### Q9: What are Stereotype Annotations in Spring? Why do we use them instead of just `@Component`?
**Level:** Junior / Mid  
**Related to:** Q10 (@Component vs @Service vs @Repository vs @Controller)

**Answer:**
مأخوذة نصاً من سؤال الإنترفيو رقم 61 في مذكراتك:
كلمة **Stereotype** معناها **تصنيف أو هوية أو دور (Role)**.
في الشركة، كل الناس موظفين، لكن علي شغال Backend Developer، ومحمد HR، وسارة Manager. كلهم موظفين بس كل واحد ليه Role.

في Spring: كل الكلاسات في النهاية عبارة عن **Beans** مسجلة في الـ ApplicationContext، لكن الفريم وورك والمطورين محتاجين يعرفوا وظيفة كل كلاس:
```
                 @Component  (الأب العام)
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   @Service      @Repository     @Controller
                                      │
                               @RestController
```

#### ليه منكتبش `@Component` على كل حاجة وخلاص؟
لو كتبت `@Component` على كله:
1. الكود هيشتغل تكنيكياً، لكنك بتفقد **الـ Architecture Clarity و الـ Readability** (مش هتعرف بمجرد النظر كود الـ DB من كود الـ Business logic).
2. بتفقد الميزات المتخصصة اللي بيضيفها كل Annotation:
   - `@Repository` بتفعل ميزة **Automatic Exception Translation**.
   - `@Controller` بتفعل الـ Routing واستقبال طلبات الـ HTTP والتعامل مع Views أو Rest Responses.
   - Pointcuts في الـ AOP بتعتمد على الـ Stereotypes لاستهداف طبقة الـ Service مثلاً لتطبيق Transaction أو Security.

> 🌟 **Golden Rule from your notes:**
> *"Every `@Service`, `@Repository`, and `@Controller` is a `@Component`, but not every `@Component` is a `@Service`, `@Repository`, or `@Controller`."*

**Short answer (30 seconds version):**
> "Stereotype annotations are specialized markers derived from @Component that assign architectural roles to classes (Web, Business, Data layers). While @Component is generic, specialized stereotypes improve readability, clarify layer responsibilities, and unlock layer-specific features like exception translation in @Repository."

---

### Q10: What is the exact difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?
**Level:** Junior / Mid  
**Related to:** Q9 (Stereotypes), Q11 (@RestController)

**Answer:**
جميعها تُقرأ أثناء الـ Component Scanning وتُسجل كـ Beans، والفرق تفصيلياً:

1. **`@Component`:**
   - الـ General-purpose stereotype.
   - يُستخدم للكلاسات العامة والمساعدة التي لا تنتمي لطبقة معمارية محددة مثل: `JwtUtils`, `EmailSender`, `FileUploader`.

2. **`@Service`:**
   - يمثل الـ **Service Layer (Business Logic)**.
   - حالياً من الناحية التقنية لا يضيف معالجة إضافية عن `@Component` سوى دلالته المعمارية وإمكانية استهدافه بالـ AOP.

3. **`@Repository` (مهمة جداً في الإنترفيو!):**
   - تمثل الـ **Data Access Layer (DAO / Database)**.
   - **الميزة التقنية الإضافية:** بتفعل ميزة **Automatic Exception Translation** عبر `PersistenceExceptionTranslationPostProcessor`.
   - لو حصل خطأ خاص بقاعدة البيانات (زي `SQLException` من Postgres أو MySQL)، Spring بيترجمه تلقائياً لـ Spring's unified `DataAccessException` hierarchy، وده بيمنع تسريب تفاصيل DB معينة لباقي التطبيق.

4. **`@Controller`:**
   - تمثل الـ **Presentation Layer (Spring MVC)**.
   - تُستخدم للتعامل مع الـ HTTP Requests وإرجاع Views (HTML/Thymeleaf/JSP).

**Short answer (30 seconds version):**
> "@Component is the generic root stereotype for any bean. @Service marks business logic components. @Repository marks data access components and enables automatic exception translation from vendor-specific SQLExceptions to Spring's DataAccessException hierarchy. @Controller handles HTTP web requests in Spring MVC."

---

### Q11: Explain `@RestController` decomposition: Which annotations compose it and why?
**Level:** Junior  
**Related to:** Q9, Q10

**Answer:**
مأخوذة مباشرة من ملف الإنترفيو رقم 62 بمذكراتك:

لو فتحت كود `@RestController` في Spring:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller       // 1. Controller Stereotype
@ResponseBody     // 2. ResponseBody Marker
public @interface RestController {
}
```

إذن `@RestController` = **`@Controller` + `@ResponseBody`**.

#### ما وظيفة كل جزء؟
1. **`@Controller`:** يرث `@Component` (بـ Meta-Annotation)، وده اللي بيسجل الكلاس كـ Spring Bean ويخليه يستقبل الـ HTTP Requests.
2. **`@ResponseBody`:** مش stereotype annotation، هو **Marker Annotation**. بيقول لـ Spring: "القيمة الراجعة من الميثود دي مش اسم صفحة HTML/View، دي عبارة عن Data خالصة حطها مباشرة في الـ HTTP Response Body (وحولها لـ JSON عبر `HttpMessageConverter` مثل Jackson)".

| Annotation | هل يحتوي على `@Component`؟ | الوظيفة |
| :--- | :--- | :--- |
| `@Controller` | ✅ نعم (مباشرة) | تسجيل Bean والتعامل كـ Web Controller |
| `@RestController` | ✅ نعم (بالوراثة من `@Controller`) | دمج الـ Web Controller مع إرجاع JSON تلقائي |
| `@ResponseBody` | ❌ لا | توجيه البيانات للـ Response Body كـ JSON |

**Short answer (30 seconds version):**
> "@RestController is a composite meta-annotation combining @Controller and @ResponseBody. @Controller registers the class as a Spring web bean, while @ResponseBody instructs Spring to serialize the return object directly into the HTTP response body as JSON or XML rather than rendering a view template."

---

### Q12: What is the difference between `@Component` and `@Bean`?
**Level:** Junior / Mid  
**Related to:** Q3 (Bean), Q9 (Stereotypes)

**Answer:**
السؤال ده بيتكرر في كل إنترفيو تقريباً:

#### 1. `@Component` (Class-level Auto-detection):
- يوضع على **Class**.
- Spring هو اللي بيكتشف الكلاس ده تلقائياً عبر الـ **Component Scan** ويعمل منه `new`.
- ينفع فقط مع **الكود الخاص بك** (كود عندك سورس كود بتاعه وتقدر تكتب فوقه annotation).

#### 2. `@Bean` (Method-level Explicit Declaration):
- يوضع على **Method** داخل كلاس متعلم بـ `@Configuration`.
- أنت اللي بتكتب كود إنشاء الـ Object بنفسك داخل الميثود (`return new SomeClass()`)، ولكن النتيجة الراجعة بتتسجل كـ Bean عند Spring.
- **الاستخدام الإجباري:** مع كلاسات الـ **Third-party libraries** (مكتبات خارجية جاية من Maven/Gradle) زي `RestTemplate`, `ObjectMapper`, `ModelMapper`, أو لو محتاج Custom initialization logic معقد قبل تسليم الـ Object للـ Container.

```java
// مثال: مكتبة خارجية من Stripe أو Jackson
@Configuration
public class ThirdPartyConfig {

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper; // Spring يدير الـ Object ده الآن
    }
}
```

| وجه المقارنة | `@Component` | `@Bean` |
| :--- | :--- | :--- |
| **المكان** | Class-level | Method-level داخل `@Configuration` |
| **طريقة الإنشاء** | تلقائية عبر Component Scan | يدوية برمجياً عبر الـ Method body |
| **مع الـ Third-party code** | ❌ مستحيل (مش هتقدر تعدل الـ .class في المكتبة) | ✅ الحل القياسي |
| **التخصيص (Customization)** | محدود | مرن جداً وتقدر تعمل أي منطق قبل الإرجاع |

**Short answer (30 seconds version):**
> "@Component is a class-level annotation for auto-detecting and managing our own classes via component scanning. @Bean is a method-level annotation used within @Configuration classes to explicitly instantiate third-party library classes or objects requiring complex manual setup."

---

### Q13: What is the difference between `BeanFactory` and `ApplicationContext`?
**Level:** Mid  
**Related to:** Q1 (IoC), Q3 (Bean), Q4 (Scopes)

**Answer:**
كلاهما يمثل الـ **Spring IoC Container**، ولكن:

```
BeanFactory (الواجهة الأساسية البسيطة - Base Interface)
     ▲
     │ extends
ApplicationContext (الحاوية المتقدمة المستخدمة عملياً)
```

1. **`BeanFactory`:**
   - الحاوية الأساسية والأبسط (Root interface).
   - بيستخدم **Lazy Loading**: يعني مش بينشئ الـ Singleton Beans إلا لما تطلبها أول مرة (`getBean()`).
   - خفيف جداً ومصمم للأجهزة ذات الموارد المحدودة للغاية (Embedded devices / Mobile).

2. **`ApplicationContext`:**
   - يرث `BeanFactory` ويضيف عليه ميزات الـ Enterprise الحديثة.
   - بيستخدم **Eager Loading**: بينشئ كل الـ Singleton Beans ويفحص الـ Dependencies وقت الـ Startup (وده بيكشف أي خطأ تهيئة فوراً Fail-fast).
   - يدعم مزايا متقدمة لا غنى عنها:
     - Internationalization (i18n) عبر `MessageSource`.
     - Event Publishing (`ApplicationEventPublisher`).
     - AOP Integration سهل.
     - Web-aware features (`WebApplicationContext`).

| وجه المقارنة | `BeanFactory` | `ApplicationContext` |
| :--- | :--- | :--- |
| **نوع الـ Loading للـ Singletons** | Lazy Loading (عند الطلب) | Eager Loading (وقت الـ Startup) |
| **اكتشاف الأخطاء** | متأخر (عند استدعاء الميثود) | Fail-fast (عند بدء تشغيل البرنامج) |
| **الميزات الإضافية** | DI و Bean lifecycle أساسي فقط | i18n, Events, AOP, Web Application Context |
| **الاستخدام العملي اليوم** | نادراً جداً | المعيار القياسي دائماً (De facto standard) |

**Short answer (30 seconds version):**
> "BeanFactory is the fundamental IoC container providing basic DI and lazy loading of beans. ApplicationContext is a superset of BeanFactory that loads singleton beans eagerly at startup for fail-fast error detection and adds enterprise capabilities like event publishing, i18n, and seamless AOP support."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What happens if two beans have circular dependencies using constructor injection? How does Spring handle it?*
2. *Can we use `@Autowired` on private fields without getters/setters? How does it work under the hood? (Reflection API)*
3. *What is the difference between `@Resource`, `@Inject`, and `@Autowired`?*
4. *How can you inject a Prototype bean into a Singleton bean correctly without stale state? (`@Lookup` / `ObjectFactory`)*

---

## 🔗 System Design Connection: Loose Coupling & Plugin Architectures

### Real-world System Design Application:
في الأنظمة الكبيرة (Enterprise Systems)، مبدأ **Loose Coupling + DI** هو الأساس لبناء أنظمة تقبل التوسع (Extensible Architectures) مثل **Payment Gateways** أو **Notification Engines**.

تخيل نظام E-Commerce عالمي يحتاج دعم وسائل دفع متعددة: Stripe, PayPal, Fawry, Vodafone Cash.
لو بنيت الكود معتمداً على أسماء الكلاسات مباشرة، في كل مرة تضيف وسيلة دفع جديدة هتضطر تعدل في الـ Checkout Service.

باستخدام مبدأ Spring Core (Interface + Strategy Pattern + Spring Auto-collection):
```java
public interface PaymentStrategy {
    PaymentProvider getProvider();
    void pay(BigDecimal amount);
}

@Service
public class CheckoutService {
    // Spring تلقائياً بيجمع كل الـ Beans اللي بتعمل implement للـ Interface في Map!
    private final Map<PaymentProvider, PaymentStrategy> paymentStrategies;

    public CheckoutService(List<PaymentStrategy> strategies) {
        this.paymentStrategies = strategies.stream()
            .collect(Collectors.toMap(PaymentStrategy::getProvider, Function.identity()));
    }

    public void processOrder(PaymentProvider provider, BigDecimal amount) {
        PaymentStrategy strategy = paymentStrategies.get(provider);
        if (strategy == null) throw new UnsupportedOperationException("Provider not supported");
        strategy.pay(amount);
    }
}
```

```
                  ┌──────────────────────┐
                  │   CheckoutService    │
                  └──────────┬───────────┘
                             │  delegates via Interface
                             ▼
                 ┌────────────────────────┐
                 │    PaymentStrategy     │
                 └───────────┬────────────┘
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ StripeStrategy  │ │ PayPalStrategy  │ │  FawryStrategy  │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```
**الفائدة:** إضافة أي وسيلة دفع جديدة في المستقبل يتطلب كتابة كلاس جديد بـ `@Component` فقط، بدون تعديل سطر واحد في كود الدفع الأصلي (**Open/Closed Principle**).
