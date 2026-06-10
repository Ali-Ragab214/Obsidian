# Introduction to Software Architecture and Patterns

---

## ما هي الـ Software Architecture؟

تخيل إنك بتبني عمارة — مش هتبدأ تبني من غير خريطة ورسومات هندسية. الـ Software Architecture هي نفس الفكرة بالظبط.

> هي القرارات الكبيرة اللي بتحدد **شكل النظام كله** — إزاي الأجزاء بتتكلم مع بعض، فين الـ data بتتحفظ، وإزاي النظام بيتوسع.

---

## ليه مهمة؟

```
بدون Architecture                    مع Architecture
─────────────────                    ───────────────
✅ بتبني بسرعة في الأول              ✅ بتبني صح من الأول
❌ بعد 6 شهور الكود كارثة            ✅ سهل تضيف features
❌ صعب تضيف features جديدة           ✅ سهل تـ scale
❌ bug في حتة بتأثر على كل حاجة      ✅ الـ bugs معزولة
❌ مستحيل تعمل testing               ✅ سهل الـ testing
```

---

## المفاهيم الأساسية

---

### 1. Coupling و Cohesion

دول أهم مفهومين في الـ Architecture.

**Coupling** = قد إيه الـ components مرتبطة ببعض

```java
// ❌ High Coupling — كارثة
public class OrderService {
    // مرتبط بكل حاجة
    private MySQLDatabase db = new MySQLDatabase();
    private SmtpEmailSender email = new SmtpEmailSender();
    private StripePayment payment = new StripePayment();

    // لو غيرت الـ database هتكسر كل حاجة
}

// ✅ Low Coupling — صح
public class OrderService {
    private final OrderRepository repository;  // interface
    private final EmailService emailService;   // interface
    private final PaymentService payment;      // interface

    // تقدر تغير الـ implementation بدون ما تلمس OrderService
}
```

**Cohesion** = قد إيه الحاجات اللي جوه الـ class مترابطة مع بعض

```java
// ❌ Low Cohesion — class بتعمل كل حاجة
public class UserManager {
    public void createUser() { }
    public void sendEmail() { }      // مش مكانها هنا
    public void generateReport() { } // مش مكانها هنا
    public void processPayment() { } // مش مكانها هنا
}

// ✅ High Cohesion — كل class بتعمل حاجة واحدة
public class UserService {
    public void createUser() { }
    public void updateUser() { }
    public void deleteUser() { }
    // كلها متعلقة بالـ User بس
}
```

> **القاعدة:** Low Coupling + High Cohesion = Architecture كويسة
> كل الميثودات تتعلق بإدارة المستخدمين فقط. يعني مش هعمل انشاء للكورس هنا 
> عندما ترى `UserService` تعرف مباشرة أنه خاص بالمستخدمين.

---

### 2. Separation of Concerns

كل جزء في النظام مسؤول عن حاجة واحدة بس.

```
┌─────────────────────────────────────────┐
│              Client Request             │
└─────────────────┬───────────────────────┘
                  │
         ┌────────▼────────┐
         │   Controller    │  ← بس بيستقبل الـ request
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │    Service      │  ← بس بيعمل الـ business logic
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │   Repository    │  ← بس بيتكلم مع الـ database
         └────────┬────────┘
                  │
         ┌────────▼────────┐
         │    Database     │
         └─────────────────┘
```

---

## Architectural Patterns الرئيسية

---

### 1. Layered Architecture (N-Tier)

الأكثر شيوعاً وده اللي انت بتستخدمه في Spring Boot.

```
┌──────────────────────────────┐
│      Presentation Layer      │  Controllers, REST APIs
├──────────────────────────────┤
│      Business Layer          │  Services, Business Logic
├──────────────────────────────┤
│      Persistence Layer       │  Repositories, DAOs
├──────────────────────────────┤
│      Database Layer          │  PostgreSQL, MySQL
└──────────────────────────────┘
```

**مثال في Spring Boot:**

```java
// Presentation Layer
@RestController
public class CourseController {
    private final CourseService courseService;

    @GetMapping("/courses")
    public ResponseEntity<?> getCourses() {
        return ResponseEntity.ok(courseService.getAllCourses());
    }
}

// Business Layer
@Service
public class CourseServiceImpl implements CourseService {
    private final CourseRepository repository;

    public List<CourseResponse> getAllCourses() {
        return repository.findAll()
                .stream()
                .map(mapper::toResponse)
                .toList();
    }
}

// Persistence Layer
@Repository
public interface CourseRepository extends JpaRepository<Course, Long> { }
```

**المميزات والعيوب:**

```
✅ سهل الفهم والتعلم
✅ منظم وواضح
✅ كويس للـ small/medium projects
❌ بيتبطأ مع الـ scale الكبير
❌ الـ layers ممكن تبقى مرتبطة ببعض
```

---

### 2. Clean Architecture

ده اللي بتستخدمه في الـ CoursePilot بتاعك — الأقوى والأنظف.

```
        ┌─────────────────────────┐
        │      Frameworks &       │
        │    Drivers (outer)      │  ← Spring, DB, UI
        ├─────────────────────────┤
        │    Interface Adapters   │  ← Controllers, Presenters
        ├─────────────────────────┤
        │    Application Rules    │  ← Use Cases, Services
        ├─────────────────────────┤
        │     Enterprise Rules    │  ← Entities, Domain Models
        └─────────────────────────┘
              ↑ Dependency Rule
        الـ dependencies بتروح للداخل بس
```

**القاعدة الذهبية:**

```
الـ inner layers مش بتعرف الـ outer layers
الـ Domain مش بيعرف Spring
الـ Service مش بيعرف الـ Controller
```

**مثال:**

```java
// Domain Layer — مش عارف Spring خالص
public class Course {
    private Long id;
    private String title;
    private CourseStatus status;

    public void publish() {
        if (this.status != CourseStatus.DRAFT)
            throw new IllegalStateException("Only draft courses can be published");
        this.status = CourseStatus.PUBLISHED;
    }
}

// Application Layer — بيعرف الـ Domain بس
public class PublishCourseUseCase {
    private final CourseRepository repository; // interface مش implementation

    public void execute(Long courseId) {
        Course course = repository.findById(courseId)
                .orElseThrow(() -> new CourseNotFoundException(courseId));
        course.publish();
        repository.save(course);
    }
}

// Infrastructure Layer — بيعرف كل حاجة
@Repository
public class CourseRepositoryImpl implements CourseRepository {
    private final JpaCourseRepository jpaRepository;
    // هنا Spring و JPA
}
```

---

### 3. Microservices Architecture

بدل ما تعمل app واحدة كبيرة، بتقسمها لـ services صغيرة مستقلة.

```
                    ┌─────────────┐
                    │  API Gateway │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼──────┐  ┌────────▼──────┐  ┌───────▼──────┐
│ User Service │  │Course Service │  │Payment Service│
│   :8081      │  │    :8082      │  │    :8083      │
└───────┬──────┘  └────────┬──────┘  └───────┬──────┘
        │                  │                  │
   ┌────▼────┐        ┌────▼────┐        ┌────▼────┐
   │Users DB │        │Courses  │        │Payments │
   │         │        │   DB    │        │   DB    │
   └─────────┘        └─────────┘        └─────────┘
```

**مثال واقعي — Netflix:**

```
مش عندهم app واحدة
عندهم 700+ microservice
كل service بتتعمل deploy لوحدها
لو الـ Recommendations Service وقعت
الـ Streaming لسه شغال
```

```
✅ كل service تـ scale لوحدها
✅ لو service وقعت مش كل حاجة بتقع
✅ تقدر تستخدم لغات مختلفة لكل service
❌ معقد جداً
❌ محتاج infrastructure كبير
❌ الـ debugging صعب
```

---

### 4. Event-Driven Architecture

الـ services بتتكلم مع بعض عن طريق **events** مش عن طريق direct calls.

```
User Service                    Course Service
─────────────                   ──────────────
user.registered event ──────►   بيسمع الـ event
                                بيعمل welcome course enrollment

Payment Service                 Email Service
───────────────                 ─────────────
payment.completed event ──────► بيبعت confirmation email
```

**مثال بسيط:**

```java
// Publisher
@Service
public class UserService {
    private final ApplicationEventPublisher eventPublisher;

    public void registerUser(RegisterRequest request) {
        User user = userRepository.save(newUser);

        // بدل ما تكال Email Service مباشرة
        eventPublisher.publishEvent(new UserRegisteredEvent(user.getEmail()));
    }
}

// Listener
@Component
public class EmailNotificationListener {

    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        emailService.sendWelcomeEmail(event.getEmail());
    }
}
```

---

## Design Patterns الأساسية

---

### 1. Repository Pattern

بتعزل الـ data access logic عن الـ business logic.

```java
// بدل ما الـ Service تتكلم مع الـ DB مباشرة
@Service
public class CourseService {
    // ✅ بتتعامل مع interface مش implementation
    private final CourseRepository repository;

    public Course findById(Long id) {
        return repository.findById(id)
                .orElseThrow(() -> new CourseNotFoundException(id));
    }
}
```

---

### 2. Factory Pattern

بتخلي الـ object creation في مكان واحد.

```java
public class CourseFactory {
    public static Course createDraftCourse(CourseRequest request, User instructor) {
        return Course.builder()
                .title(request.getTitle())
                .description(request.getDescription())
                .price(request.getPrice())
                .status(CourseStatus.DRAFT)
                .instructor(instructor)
                .createdAt(LocalDateTime.now())
                .build();
    }
}

// في الـ Service
Course course = CourseFactory.createDraftCourse(request, currentUser);
```

---

### 3. Strategy Pattern

بتغير الـ behavior في الـ runtime بدون ما تغير الكود.

```java
// مثال: طرق الدفع المختلفة
public interface PaymentStrategy {
    void pay(BigDecimal amount);
}

public class StripePayment implements PaymentStrategy {
    public void pay(BigDecimal amount) { /* Stripe logic */ }
}

public class PayPalPayment implements PaymentStrategy {
    public void pay(BigDecimal amount) { /* PayPal logic */ }
}

// في الـ Service
public class PaymentService {
    private PaymentStrategy strategy;

    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void processPayment(BigDecimal amount) {
        strategy.pay(amount); // بتشتغل مع أي strategy
    }
}
```

---

### 4. Observer Pattern

نفس فكرة الـ Event-Driven — entity بتخطر entities تانية لما حاجة تحصل.

```java
// Spring بيعمل ده تلقائياً بالـ @EventListener
// لما course اتنشر → خطر كل الـ enrolled students
@EventListener
public void onCoursePublished(CoursePublishedEvent event) {
    enrollmentService.notifyStudents(event.getCourseId());
}
```

---

## إزاي تختار الـ Architecture الصح؟

```
┌─────────────────────────────────────────────────────┐
│ السؤال                         الإجابة              │
├─────────────────────────────────────────────────────┤
│ Project صغير + team صغير    → Layered Architecture  │
│ Project متوسط + growing     → Clean Architecture    │
│ Project كبير + teams كتير  → Microservices          │
│ Real-time + events كتير     → Event-Driven          │
└─────────────────────────────────────────────────────┘
```

---

## الـ Trade-offs — مفيش Architecture مثالية

```
Simplicity     ◄────────────────► Scalability
     │                                  │
Layered                          Microservices
(سهل بس مش بيتسكيل)          (بيتسكيل بس معقد)


Consistency    ◄────────────────► Availability
     │                                  │
Strong ACID                      Eventual Consistency
(بيانات صح دايماً)              (ممكن يتأخر شوية)
```

---

## ملخص

```
Architecture = القرارات الكبيرة اللي صعب تتغير بعدين

المبادئ الأساسية:
├── Low Coupling     → الأجزاء مش مرتبطة ببعض
├── High Cohesion    → كل class عندها مسؤولية واحدة
└── Separation of Concerns → كل layer بتعمل حاجة واحدة

الـ Patterns الرئيسية:
├── Layered      → الأبسط — كويس للبداية
├── Clean        → الأنظف — كويس للـ production
├── Microservices → الأقوى — للـ scale الكبير
└── Event-Driven  → للـ real-time systems
```

> **الحقيقة:** مش في Architecture صح أو غلط — في Architecture **مناسب للمشكلة** بتاعتك. الـ Senior Engineer مش بيختار الـ Architecture الأعقد — بيختار الـ **أبسط اللي بيحل المشكلة**.