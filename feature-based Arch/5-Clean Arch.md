تمام. خلينا نمسك **Clean Architecture نفسها** من أهم قاعدة فيها، وبعدها نربطها بالـ `Course` اللي كنا بنتكلم عنه.

![Image](https://images.openai.com/static-rsc-4/gaPhWXFbKiRYiumE_qx9gqLO29qAFjCGOOYxxOPTaJSsw-_lFbrF_QqmhxSDalNX4p24OHp8PqUctgZD_OlShS_wd3biPFmuEdomunAM8khsoAk70BoJhrFN2iTxQCfRp-4dI1N0d99FoBNTt-DZDhdjXuqDF1Ix2-5EWOP6hQKnolK8WnvU9s-h7Kz55yvw?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/BG28Gniq3v6f3ChuJo7dpM77R2QFD8BezBmUQnB8AwEbPDGe99JfQi1gRPlh5CMpyjmItj-mY-8z51NwdNPtS_pK7DcpcWvke7CXk_S8cpWfKFtM5TNl0ijBXCJI25O5saCadE4b18l89BGGxGJmnOj4Y0AYJ3BnKy56ZGeYSZljWtAxbzc5oHlu9UPLbRoo?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/CW7Tj2F-HySb6iB2l-1cpt9OURV3YuFh-daSNudPhyePcHO31TVzj_FNuKJ_s1bTV24DNtAPwHYFDIYprbOo7-bAGnXy0TsYFvMF0uuVJGCu0STd_pkyASx-iaXAYbINYfcBE0c6s0U8BTe2W_DawgnaCoqu_zwLWGefb0pmJJH9FU12UJ6rFMb5CBmTEBIA?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pDqujSFfdzb37oWpOx7x69-jh8XYNQ30CPUbHsTOPP0-XGYUiZgW8unKiFkcwmvGGa3NUMtpXE0ib2enkuPbiANJ5IvbTMB5ihReH1NDKeNmcgyZMXIrKYeBTS2aJh61SG-odI8VS3ld0AM7UKim_ZcpesX86FZpJyOL-xdmZ6pnHy-IToZfX1EIgkTdNgK2?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/H4BQxQoMm4Hy3Wcrue2aJWwfC0w_FC9GJyOj38fsWXMdgFV2cZoHcfziIn27Wn-0RyUvR-Ha3ni_2iHrrarCasd02O-A0CVextDuwsX2b0Zt8WIijiDen_k0DF8s8k2ClQ7YXRdraeFyGnaX_iozjrHhS5S_DaYXEil3Aa9yLNvqPiyzDvwtOcBJykIuSysW?purpose=fullsize)

## 1. الطبقات الأساسية

في الـ Clean Architecture عندك بشكل مبسط:

```text
┌─────────────────────────────────────────────┐
│           Frameworks & Infrastructure       │
│       Spring / JPA / PostgreSQL / Web       │
│                                             │
│   ┌─────────────────────────────────────┐   │
│   │          Interface Adapters         │   │
│   │ Controllers / Mappers / Adapters    │   │
│   │                                     │   │
│   │   ┌─────────────────────────────┐   │   │
│   │   │       Application           │   │   │
│   │   │  Use Cases / Handlers       │   │   │
│   │   │                             │   │   │
│   │   │   ┌─────────────────────┐   │   │   │
│   │   │   │       Domain        │   │   │   │
│   │   │   │ Entities / Rules    │   │   │   │
│   │   │   └─────────────────────┘   │   │   │
│   │   └─────────────────────────────┘   │   │
│   └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

لكن **الأهم من أسماء الطبقات هو اتجاه الـ dependency**.

---

# 2. القاعدة الذهبية: Dependencies بتدخل لجوه

احفظ الجملة دي:

> **Dependencies must point inward.**

يعني:

```text
Infrastructure
      ↓
Interface Adapters
      ↓
Application
      ↓
Domain
```

كل ما تنزل لجوه، الطبقة **مش عارفة** الطبقات اللي بره.

لكن الطبقة اللي بره ممكن تعتمد على اللي جواها.

### مثال:

```text
Controller
    ↓
CreateCourseHandler
    ↓
Course
```

صح.

لكن:

```text
Course
    ↓
CourseJpaRepository
```

❌ غلط.

لأن الـ Domain بقى عارف Infrastructure.

---

# 3. مين يعتمد على مين؟

بالنسبة للـ structure بتاعتنا:

```text
                    ┌───────────────────┐
                    │   Infrastructure  │
                    │                   │
                    │ JPA               │
                    │ PostgreSQL        │
                    │ RepositoryImpl    │
                    └─────────┬─────────┘
                              │
                              ↓
                    ┌───────────────────┐
                    │       API         │
                    │                   │
                    │ Controllers       │
                    │ Requests          │
                    │ Responses         │
                    └─────────┬─────────┘
                              │
                              ↓
                    ┌───────────────────┐
                    │   Application     │
                    │                   │
                    │ Commands          │
                    │ Queries           │
                    │ Handlers          │
                    └─────────┬─────────┘
                              │
                              ↓
                    ┌───────────────────┐
                    │      Domain       │
                    │                   │
                    │ Course             │
                    │ Business Rules    │
                    │ Repository Ports  │
                    └───────────────────┘
```

لكن هنا محتاج أصحح نقطة صغيرة من الرسم:

**API و Infrastructure مش لازم يكونوا بيعتمدوا على بعض.**

الأدق:

```text
                       DOMAIN
                    ┌───────────┐
                    │  Course   │
                    │   Rules   │
                    │ Repository│
                    │ Interface │
                    └─────▲─────┘
                          │
                          │
                    ┌─────┴─────┐
                    │Application│
                    │ Handlers  │
                    └─────▲─────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             │                         │
       ┌─────┴──────┐           ┌──────┴──────┐
       │    API      │           │Infrastructure│
       │ Controllers │           │ JPA / DB    │
       └─────────────┘           └──────────────┘
```

---

# 4. طب الـ Business Logic يتحط فين؟

دي أهم نقطة في سؤالك.

مش كل Logic في Application.

لازم تفرق بين:

### Domain Business Logic

و

### Application Logic

---

## Domain Business Logic

دي القواعد اللي **تمثل طبيعة الـ business نفسها**.

مثلاً Course:

> الـ Course المنشور مينفعش يتعمله Publish مرة تانية.

```java
public class Course {

    private CourseStatus status;

    public void publish() {

        if (status == CourseStatus.PUBLISHED) {
            throw new CourseAlreadyPublishedException();
        }

        status = CourseStatus.PUBLISHED;
    }
}
```

دي **Domain Logic**.

ليه؟

لأن القاعدة دي تخص الـ Course نفسه، ومش فارق معانا مين استدعاها:

```text
REST API
   أو
Kafka
   أو
CLI
   أو
Scheduled Job
```

الـ rule نفسها تفضل:

```text
Published Course
       ↓
Cannot publish again
```

---

# 5. طيب Application Logic بيبقى إيه؟

Application Layer مسؤولة عن **تنسيق الـ Use Case**.

مثلاً:

```java
@Service
public class PublishCourseHandler {

    private final CourseRepository courseRepository;

    public Course handle(PublishCourseCommand command) {

        Course course = courseRepository
                .findById(command.courseId())
                .orElseThrow(CourseNotFoundException::new);

        course.publish();

        return courseRepository.save(course);
    }
}
```

هنا الـ Handler بيعمل:

```text
1. Get Course
2. Check if exists
3. Ask Course to publish
4. Save Course
```

ده اسمه **Application Orchestration**.

لكن هو **مش اللي بيقرر** هل Course ينفع يتعمله Publish أم لا.

الـ Domain هو اللي بيقرر:

```java
course.publish();
```

---

# 6. الفرق مهم جدًا

### Application:

```text
"إزاي أنفذ Use Case؟"
```

### Domain:

```text
"إيه القواعد اللي تحكم الـ Business؟"
```

مثلاً:

```text
Publish Course
```

الـ Application يقول:

```text
هات الـ Course
        ↓
استدعي publish()
        ↓
Save
```

والـ Domain يقول:

```text
هل ينفع Publish؟
        ↓
Yes → Published
No  → Exception
```

---

# 7. طيب `CourseDomainService` نستخدمه إمتى؟

مش كل Business Logic لازم يتحط في Domain Service.

الأفضل الأول تحاول تحطه داخل الـ Entity نفسها.

مثلاً:

```java
course.publish();
course.archive();
course.update();
```

ممتاز.

لكن لو عندك Rule **بتحتاج أكتر من Entity**:

```text
Course
Instructor
Enrollment
```

مثلاً:

> Instructor cannot publish more than 10 courses unless he is verified.

هنا الـ rule مش تخص `Course` لوحده.

ممكن تعمل:

```java
public class CourseDomainService {

    public void validatePublishing(
            Course course,
            Instructor instructor
    ) {
        // complex business rule
    }
}
```

يبقى:

```text
Entity
    ↓
Rules related mainly to itself

Domain Service
    ↓
Rules involving multiple domain objects
```

---

# 8. والـ Repository Interface مكانه فين؟

دي نقطة هتربط كل الكلام ببعضه.

عندنا:

```java
public interface CourseRepository {

    Course save(Course course);

    Optional<Course> findById(Long id);
}
```

الـ Application محتاج Repository.

لكن الـ Domain/Application **مش عايز يعرف** إننا بنستخدم JPA.

لذلك:

```text
Application
     ↓
CourseRepository (interface)
```

والـ Infrastructure تعمل:

```java
@Repository
public class CourseRepositoryImpl
        implements CourseRepository {
}
```

وبالتالي:

```text
             APPLICATION
                  │
                  ↓
        CourseRepository
             interface
                  ↑
                  │ implements
                  │
        CourseRepositoryImpl
                  │
                  ↓
       CourseJpaRepository
                  │
                  ↓
              PostgreSQL
```

وده تطبيق **Dependency Inversion Principle**.

---

# 9. مين يشوف مين؟

احفظ الجدول ده:

|Layer|تقدر تعتمد على|
|---|---|
|**Domain**|نفسه فقط + abstractions داخل الـ Domain|
|**Application**|Domain|
|**API**|Application + Domain models حسب التصميم|
|**Infrastructure**|Application + Domain|
|**JPA**|Infrastructure|

والأهم:

```text
Domain ❌ → Application
Domain ❌ → API
Domain ❌ → JPA
Domain ❌ → Spring

Application ❌ → JPA
Application ❌ → PostgreSQL
Application ❌ → Controller

Infrastructure ✅ → Domain
Infrastructure ✅ → Application
API ✅ → Application
```

---

# 10. بالنسبة لمشروعك تحديدًا

يبقى عندك:

```text
course/
│
├── api/
│   └── CourseController
│
├── application/
│   ├── command/
│   │   ├── CreateCourseHandler
│   │   ├── UpdateCourseHandler
│   │   └── PublishCourseHandler
│   │
│   └── query/
│       └── GetCourseHandler
│
├── domain/
│   ├── entity/
│   │   └── Course
│   │
│   ├── repository/
│   │   └── CourseRepository
│   │
│   └── service/
│       └── CourseDomainService
│
└── infrastructure/
    ├── persistence/
    │   ├── CourseEntity
    │   └── CourseJpaRepository
    │
    └── adapter/
        └── CourseRepositoryImpl
```

والـ dependency:

```text
                      ┌──────────────────┐
                      │   Course Domain  │
                      │                  │
                      │ Course           │
                      │ Business Rules  │
                      │ CourseRepository│
                      └────────▲─────────┘
                               │
                               │ depends on
                               │
                      ┌────────┴─────────┐
                      │   Application    │
                      │                  │
                      │ Commands         │
                      │ Queries          │
                      │ Handlers         │
                      └───────▲──────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
              ┌─────┴─────┐     ┌──────┴─────────┐
              │    API     │     │ Infrastructure │
              │ Controller │     │                │
              │ REST       │     │ JPA            │
              └───────────┘     │ PostgreSQL     │
                                │ RepositoryImpl │
                                └────────────────┘
```

### أهم 3 قواعد تحفظهم للـ Interview:

**1. Dependency Rule**

> **Dependencies point inward.**

**2. Domain**

> **The Domain contains the core business rules and should be independent of frameworks, databases, and UI.**

**3. Application**

> **The Application layer orchestrates use cases but should not contain infrastructure-specific logic.**

ولو عايز تلخص الـ Clean Architecture كلها في سطر واحد:

> **Outer layers know about inner layers, but inner layers don't know about outer layers.**

وده هو السبب الحقيقي اللي خلانا من البداية نعمل `Course` منفصل عن `CourseEntity`، و`CourseRepository` منفصل عن `CourseJpaRepository`.


        ┌───────────────┐
        │      API      │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │  Application  │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │    Domain     │
        └───────────────┘
                ↑
                │
        ┌───────┴───────┐
        │ Infrastructure │
        └───────────────┘





