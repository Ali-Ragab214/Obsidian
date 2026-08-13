آه. خلينا نبني **Feature كاملة `course`** كأنها Production Spring Boot project، ونشوف **مين بيحقن مين وليه**.

هستخدم هنا:

- **Feature-based organization**
    
- **Clean Architecture**
    
- **CQRS**
    
- **Repository Pattern**
    
- **Constructor Injection**
    
- Domain مستقل عن Spring/JPA
    
- كل Use Case له Handler مستقل
    

---

# 1. شكل الـ Module

```text
course/
├── api/
│   ├── CourseController.java
│   ├── request/
│   │   ├── CreateCourseRequest.java
│   │   └── UpdateCourseRequest.java
│   └── response/
│       └── CourseResponse.java
│
├── application/
│   ├── command/
│   │   ├── create/
│   │   │   ├── CreateCourseCommand.java
│   │   │   └── CreateCourseHandler.java
│   │   │
│   │   ├── update/
│   │   │   ├── UpdateCourseCommand.java
│   │   │   └── UpdateCourseHandler.java
│   │   │
│   │   └── publish/
│   │       ├── PublishCourseCommand.java
│   │       └── PublishCourseHandler.java
│   │
│   └── query/
│       └── get/
│           ├── GetCourseQuery.java
│           └── GetCourseHandler.java
│
├── domain/
│   ├── entity/
│   │   └── Course.java
│   │
│   ├── repository/
│   │   └── CourseRepository.java
│   │
│   ├── service/
│   │   └── CourseDomainService.java
│   │
│   └── exception/
│       ├── CourseNotFoundException.java
│       └── CourseAlreadyPublishedException.java
│
└── infrastructure/
    ├── persistence/
    │   ├── CourseEntity.java
    │   └── CourseJpaRepository.java
    │
    ├── mapper/
    │   └── CourseMapper.java
    │
    └── adapter/
        └── CourseRepositoryImpl.java
```

---

# 2. أول حاجة: الـ Domain

الـ Domain هو **قلب الـ Feature**.

هو لا يعرف:

- Spring
    
- JPA
    
- PostgreSQL
    
- REST
    
- DTO
    

---

## `Course.java`

```java
package com.example.course.domain.entity;

public class Course {

    private Long id;
    private String title;
    private String description;
    private CourseStatus status;

    public Course(
            Long id,
            String title,
            String description,
            CourseStatus status
    ) {
        this.id = id;
        this.title = title;
        this.description = description;
        this.status = status;
    }

    public static Course create(
            String title,
            String description
    ) {
        return new Course(
                null,
                title,
                description,
                CourseStatus.DRAFT
        );
    }

    public void update(
            String title,
            String description
    ) {
        if (status == CourseStatus.PUBLISHED) {
            throw new IllegalStateException(
                    "Published course cannot be updated"
            );
        }

        this.title = title;
        this.description = description;
    }

    public void publish() {
        if (status == CourseStatus.PUBLISHED) {
            throw new CourseAlreadyPublishedException();
        }

        this.status = CourseStatus.PUBLISHED;
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }

    public CourseStatus getStatus() {
        return status;
    }
}
```

و:

```java
public enum CourseStatus {
    DRAFT,
    PUBLISHED,
    ARCHIVED
}
```

لاحظ هنا:

**الـ Entity نفسها عندها business behavior.**

مش:

```java
course.setStatus(PUBLISHED);
```

لكن:

```java
course.publish();
```

وده أقرب للـ Domain Model الصح.

---

# 3. Repository Interface

`CourseRepository.java`

```java
package com.example.course.domain.repository;

public interface CourseRepository {

    Course save(Course course);

    Optional<Course> findById(Long id);

    boolean existsByTitle(String title);
}
```

لاحظ:

```text
Domain
  ↓
CourseRepository
```

الـ Domain/Application بيقول:

> أنا محتاج repository.

لكن مش عارف implementation إيه.

---

# 4. Infrastructure

هنا بقى Spring + JPA.

## `CourseEntity.java`

```java
@Entity
@Table(name = "courses")
public class CourseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String description;

    @Enumerated(EnumType.STRING)
    private CourseStatus status;

    // getters/setters
}
```

دي **مش Domain Entity**.

دي Database representation.

---

# 5. `CourseJpaRepository`

```java
public interface CourseJpaRepository
        extends JpaRepository<CourseEntity, Long> {

    boolean existsByTitle(String title);
}
```

Spring Data JPA هيعمل implementation لها.

---

# 6. الـ Mapper

`CourseMapper.java`

```java
@Component
public class CourseMapper {

    public CourseEntity toEntity(Course course) {

        CourseEntity entity = new CourseEntity();

        entity.setId(course.getId());
        entity.setTitle(course.getTitle());
        entity.setDescription(course.getDescription());
        entity.setStatus(course.getStatus());

        return entity;
    }

    public Course toDomain(CourseEntity entity) {

        return new Course(
                entity.getId(),
                entity.getTitle(),
                entity.getDescription(),
                entity.getStatus()
        );
    }
}
```

وظيفته:

```text
Domain ↔ JPA
```

---

# 7. Repository Adapter

هنا أهم جزء.

`CourseRepositoryImpl.java`

```java
@Repository
@RequiredArgsConstructor
public class CourseRepositoryImpl
        implements CourseRepository {

    private final CourseJpaRepository jpaRepository;
    private final CourseMapper mapper;

    @Override
    public Course save(Course course) {

        CourseEntity entity =
                mapper.toEntity(course);

        CourseEntity saved =
                jpaRepository.save(entity);

        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Course> findById(Long id) {

        return jpaRepository.findById(id)
                .map(mapper::toDomain);
    }

    @Override
    public boolean existsByTitle(String title) {
        return jpaRepository.existsByTitle(title);
    }
}
```

هنا الـ dependency graph:

```text
CourseRepositoryImpl
       │
       ├── CourseJpaRepository
       │
       └── CourseMapper
```

---

# 8. Application Layer

هنا بنحط **Use Cases**.

مثلاً Create Course.

## `CreateCourseCommand`

```java
public record CreateCourseCommand(
        String title,
        String description
) {
}
```

ده مجرد **data**.

مش:

```java
@Service
```

ولا:

```java
@Component
```

---

# 9. `CreateCourseHandler`

هنا الـ logic الخاص بالـ use case.

```java
@Service
@RequiredArgsConstructor
public class CreateCourseHandler {

    private final CourseRepository courseRepository;

    public Course handle(CreateCourseCommand command) {

        if (courseRepository.existsByTitle(command.title())) {
            throw new IllegalArgumentException(
                    "Course title already exists"
            );
        }

        Course course = Course.create(
                command.title(),
                command.description()
        );

        return courseRepository.save(course);
    }
}
```

لاحظ حاجة مهمة جدًا:

الـ Handler مش واخد:

```text
CourseJpaRepository
```

ولا:

```text
CourseEntity
```

ولا:

```text
CourseMapper
```

هو واخد:

```text
CourseRepository
```

لأن Application layer مش المفروض تعرف infrastructure details.

---

# 10. Update

`UpdateCourseCommand`

```java
public record UpdateCourseCommand(
        Long courseId,
        String title,
        String description
) {
}
```

`UpdateCourseHandler`

```java
@Service
@RequiredArgsConstructor
public class UpdateCourseHandler {

    private final CourseRepository courseRepository;

    public Course handle(UpdateCourseCommand command) {

        Course course = courseRepository
                .findById(command.courseId())
                .orElseThrow(CourseNotFoundException::new);

        course.update(
                command.title(),
                command.description()
        );

        return courseRepository.save(course);
    }
}
```

لاحظ الـ flow:

```text
Controller
    ↓
UpdateCourseHandler
    ↓
CourseRepository
    ↓
CourseRepositoryImpl
    ↓
CourseJpaRepository
    ↓
Database
```

---

# 11. Publish

`PublishCourseCommand`

```java
public record PublishCourseCommand(
        Long courseId
) {
}
```

`PublishCourseHandler`

```java
@Service
@RequiredArgsConstructor
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

هنا الـ business rule:

```java
course.publish();
```

موجود في **Domain**.

لكن orchestration بتاع الـ use case موجود في **Application**.

وده فصل مهم جدًا.

---

# 12. Query

`GetCourseQuery`

```java
public record GetCourseQuery(
        Long courseId
) {
}
```

`GetCourseHandler`

```java
@Service
@RequiredArgsConstructor
public class GetCourseHandler {

    private final CourseRepository courseRepository;

    public Course handle(GetCourseQuery query) {

        return courseRepository
                .findById(query.courseId())
                .orElseThrow(CourseNotFoundException::new);
    }
}
```

---

# 13. API Layer

## Request

```java
public record CreateCourseRequest(
        @NotBlank String title,
        @NotBlank String description
) {
}
```

وده **HTTP DTO**.

---

# 14. Response

```java
public record CourseResponse(
        Long id,
        String title,
        String description,
        CourseStatus status
) {
}
```

---

# 15. Controller

```java
@RestController
@RequestMapping("/api/courses")
@RequiredArgsConstructor
public class CourseController {

    private final CreateCourseHandler createCourseHandler;
    private final UpdateCourseHandler updateCourseHandler;
    private final PublishCourseHandler publishCourseHandler;
    private final GetCourseHandler getCourseHandler;

    @PostMapping
    public CourseResponse create(
            @Valid @RequestBody CreateCourseRequest request
    ) {

        CreateCourseCommand command =
                new CreateCourseCommand(
                        request.title(),
                        request.description()
                );

        Course course =
                createCourseHandler.handle(command);

        return toResponse(course);
    }

    @GetMapping("/{id}")
    public CourseResponse get(
            @PathVariable Long id
    ) {

        Course course =
                getCourseHandler.handle(
                        new GetCourseQuery(id)
                );

        return toResponse(course);
    }

    @PostMapping("/{id}/publish")
    public CourseResponse publish(
            @PathVariable Long id
    ) {

        Course course =
                publishCourseHandler.handle(
                        new PublishCourseCommand(id)
                );

        return toResponse(course);
    }

    private CourseResponse toResponse(Course course) {

        return new CourseResponse(
                course.getId(),
                course.getTitle(),
                course.getDescription(),
                course.getStatus()
        );
    }
}
```

---

# 16. طب شوف الـ Dependency Graph كله

دي أهم حاجة عايزك تحفظها:

```text
                    API
                     │
                     ↓
              ┌──────────────┐
              │  Controller  │
              └──────┬───────┘
                     │
                     ↓
              APPLICATION
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
 CreateHandler UpdateHandler PublishHandler
        │            │            │
        └────────────┼────────────┘
                     ↓
              CourseRepository
                «interface»
                     ↑
                     │
              CourseRepositoryImpl
                     │
             ┌───────┴────────┐
             ↓                ↓
    CourseJpaRepository   CourseMapper
             │
             ↓
           JPA
             │
             ↓
         PostgreSQL
```

والـ Domain:

```text
                 DOMAIN
                   ↑
          Application depends on it

       ┌─────────────────────┐
       │      Course         │
       │                     │
       │  publish()          │
       │  update()           │
       │  create()           │
       └─────────────────────┘

       ┌─────────────────────┐
       │ CourseRepository    │
       │     interface       │
       └─────────────────────┘
```

---

# 17. أهم نقطة: مين `@Service` ومين لأ؟

|Class|Annotation|السبب|
|---|---|---|
|`CourseController`|`@RestController`|API|
|`CreateCourseHandler`|`@Service`|Application Use Case|
|`UpdateCourseHandler`|`@Service`|Application Use Case|
|`PublishCourseHandler`|`@Service`|Application Use Case|
|`GetCourseHandler`|`@Service`|Application Use Case|
|`Course`|**None**|Pure Domain|
|`CourseDomainService`|حسب التصميم|Domain Service|
|`CourseRepository`|**None**|Port/Interface|
|`CourseRepositoryImpl`|`@Repository`|Infrastructure Adapter|
|`CourseJpaRepository`|Spring Data|JPA Repository|
|`CourseEntity`|`@Entity`|Database model|
|`CourseMapper`|`@Component`|Mapping|

---

# 18. وهل الـ Controller هيبقى عنده 10 Dependencies؟

ممكن يحصل، وده **مش مثالي**.

لو Course عندك فيه:

```text
Create
Update
Delete
Publish
Archive
Enroll
Unenroll
Review
...
```

ممكن Controller يوصل لـ:

```java
private final CreateCourseHandler createHandler;
private final UpdateCourseHandler updateHandler;
private final PublishCourseHandler publishHandler;
private final ArchiveCourseHandler archiveHandler;
private final DeleteCourseHandler deleteHandler;
```

وده مقبول إلى حد ما، لكن لو بدأ يكبر جدًا، ممكن تستخدم **Facade / Application Service** أو تقسّم الـ API Controllers حسب الـ use cases.

مثلاً:

```text
CourseCommandController
CourseQueryController
CoursePublishingController
```

خصوصًا لو الـ feature كبيرة.

---

# 19. طب هل كل Handler لازم يبقى فيه Repository؟

**لا.**

دي نقطة مهمة جدًا.

الـ dependency injection مش:

> "كل Handler ياخد كل حاجة."

لكن:

> **Inject only what the use case actually needs.**

مثلاً:

```text
CreateCourseHandler
    ↓
CourseRepository

PublishCourseHandler
    ↓
CourseRepository

SearchCoursesHandler
    ↓
CourseReadRepository

SendCourseNotificationHandler
    ↓
NotificationService
```

كل Use Case له dependencies الخاصة به.

وده أحد أسباب إن **Use Case per Handler** أفضل من `CourseService` ضخم.

---

## وأخيرًا: الشكل اللي أنصحك تمشي عليه

بالنسبة لمشروع Spring Boot كبير زي الـ **SkillHub** بتاعك، أنا أميل للشكل ده:

```text
feature/
├── api/
├── application/
│   ├── command/
│   │   └── use-case/
│   └── query/
│       └── use-case/
├── domain/
│   ├── entity/
│   ├── repository/
│   ├── service/
│   └── exception/
└── infrastructure/
    ├── persistence/
    ├── mapper/
    └── adapter/
```

والقاعدة الذهبية:

> **Controller → Handler → Domain/Repository Interface → Infrastructure Adapter → JPA → Database**

وليس:

> Controller → Service → JPA Repository مباشرة.

ده اللي بيخلي الـ **Feature-Based Clean Architecture** بتاعتك فعلاً لها قيمة، مش مجرد folders متقسمة بشكل حلو.



أيوه، دول **أهم نقطتين في الـ architecture كلها**. لو فهمتهم هتفهم ليه عاملين Domain + Infrastructure أصلًا.

---

# 1. ليه عندنا `Course.java` و `CourseEntity.java`؟

ببساطة:

> **`Course` = Business Object**
> 
> **`CourseEntity` = Database Object**

يعني الاتنين ممكن يمثلوا نفس الـ Course، لكن **كل واحد له مسؤولية مختلفة**.

---

## أولًا: `Course.java`

ده **Domain Entity**.

وظيفته إنه يمثل الـ Course من وجهة نظر الـ **Business**.

مثلاً:

```java
public class Course {

    private Long id;
    private String title;
    private String description;
    private CourseStatus status;

    public void publish() {
        if (status == CourseStatus.PUBLISHED) {
            throw new CourseAlreadyPublishedException();
        }

        this.status = CourseStatus.PUBLISHED;
    }

    public void archive() {
        if (status != CourseStatus.PUBLISHED) {
            throw new IllegalStateException(
                "Only published courses can be archived"
            );
        }

        this.status = CourseStatus.ARCHIVED;
    }
}
```

لاحظ إن عنده **Business Rules**.

مثلاً:

> مينفعش Course Published يتعمله Publish تاني.

أو:

> مينفعش Archive لـ Course لسه Draft.

دي قواعد Business، ومكانها الطبيعي هو الـ **Domain**.

---

## طب إيه اللي مش موجود في `Course.java`؟

مش عايزينه يعرف إن البيانات بتتخزن في PostgreSQL.

يعني مش عايزين:

```java
@Entity
@Table(name = "courses")
```

ولا:

```java
@ManyToOne
```

ولا:

```java
@Column
```

ولا:

```java
@GeneratedValue
```

لأن دي كلها **Database/Persistence concerns**.

---

# ثانيًا: `CourseEntity.java`

دي بتمثل الـ Course في **Database**.

مثلاً:

```java
@Entity
@Table(name = "courses")
public class CourseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String description;

    @Enumerated(EnumType.STRING)
    private CourseStatus status;

    // getters & setters
}
```

هنا أنت بتقول لـ JPA:

> "يا JPA، الـ object ده بيتخزن في جدول `courses`."

فهي مهتمة بـ:

- Database mapping
    
- Columns
    
- Relationships
    
- JPA annotations
    
- Persistence
    

---

# طب ليه ما نستخدمش `CourseEntity` وخلاص؟

ده سؤال Interview ممتاز.

تقدر تعمل:

```java
@Entity
public class Course {

    @Id
    private Long id;

    private String title;

    public void publish() {
        // business logic
    }
}
```

وده **هيشتغل عادي جدًا**.

لكن كده عملت coupling بين:

```text
Business Logic
      +
JPA
```

يعني الـ Domain بتاعك أصبح يعرف Spring/JPA.

وده بيخالف جزء مهم من Clean Architecture:

> **The domain should not depend on infrastructure details.**

---

## مثال يوضح المشكلة

تخيل بعد سنتين قررت تنقل من:

```text
PostgreSQL + JPA
```

إلى:

```text
MongoDB
```

لو عندك:

```java
@Entity
public class Course {
}
```

فالـ Domain نفسه مربوط بـ JPA.

لكن لو عندك:

```text
Course.java
CourseEntity.java
```

ممكن تغير:

```text
CourseEntity
     ↓
MongoCourseDocument
```

والـ `Course` يفضل زي ما هو.

---

# الخلاصة في النقطة الأولى

```text
Course
│
├── Business rules
├── Business behavior
├── Domain state
└── Pure Java

CourseEntity
│
├── JPA annotations
├── Database mapping
├── Relationships
└── Persistence concerns
```

والـ Mapper هو اللي بيربطهم:

```text
Course
  ↕
CourseMapper
  ↕
CourseEntity
```

---

# 2. ليه عندنا `CourseRepository` و `CourseJpaRepository`؟

نفس الفكرة بالضبط.

عندنا:

```text
CourseRepository
```

وده **Domain/Application abstraction**.

وعندنا:

```text
CourseJpaRepository
```

وده **Infrastructure/JPA implementation detail**.

---

# `CourseRepository`

ده Interface أنت اللي بتكتبه:

```java
public interface CourseRepository {

    Course save(Course course);

    Optional<Course> findById(Long id);

    boolean existsByTitle(String title);
}
```

لاحظ:

**مفيش JPA هنا.**

مفيش:

```java
extends JpaRepository
```

ومفيش:

```java
CourseEntity
```

لأن الـ Domain مش المفروض يعرف JPA.

---

# `CourseJpaRepository`

ده بتاع Spring Data:

```java
public interface CourseJpaRepository
        extends JpaRepository<CourseEntity, Long> {

    boolean existsByTitle(String title);
}
```

ده معناه:

> "أنا بتعامل مع `CourseEntity` في الـ Database باستخدام JPA."

---

# طب مين بيربط الاتنين؟

`CourseRepositoryImpl`.

```java
@Repository
@RequiredArgsConstructor
public class CourseRepositoryImpl
        implements CourseRepository {

    private final CourseJpaRepository jpaRepository;
    private final CourseMapper mapper;

    @Override
    public Course save(Course course) {

        CourseEntity entity =
                mapper.toEntity(course);

        CourseEntity saved =
                jpaRepository.save(entity);

        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Course> findById(Long id) {

        return jpaRepository.findById(id)
                .map(mapper::toDomain);
    }
}
```

وده أهم جزء في الموضوع.

---

# شوف الـ Flow

لما تعمل:

```java
courseRepository.save(course);
```

الـ Handler شايف:

```text
CourseRepository
```

فقط.

لكن اللي بيحصل تحت:

```text
CreateCourseHandler
        ↓
CourseRepository
        ↓
CourseRepositoryImpl
        ↓
CourseMapper
        ↓
CourseEntity
        ↓
CourseJpaRepository
        ↓
JPA
        ↓
PostgreSQL
```

---

# ليه كل التعقيد ده؟

عشان الـ Application يقول:

> **أنا محتاج أحفظ Course.**

مش:

> أنا محتاج أستخدم PostgreSQL + JPA + CourseEntity.

الـ Application يهتم بالـ **what**:

```text
CourseRepository
```

والـ Infrastructure تهتم بالـ **how**:

```text
CourseRepositoryImpl
       ↓
JPA
       ↓
PostgreSQL
```

وده تطبيق مباشر لـ:

### Dependency Inversion Principle

بدل:

```text
Application
     ↓
JPA
```

بقى:

```text
Application
     ↓
Interface
     ↑
Infrastructure
```

---

# طيب ليه ما نخليش `CourseRepository` نفسه extends JpaRepository؟

لو عملت:

```java
public interface CourseRepository
        extends JpaRepository<CourseEntity, Long> {
}
```

فأنت كده خليت الـ abstraction بتاعتك تعتمد على JPA.

يعني:

```text
Domain/Application
       ↓
    Spring Data JPA
```

وبالتالي فقدت جزء مهم من الـ Clean Architecture.

---

# احفظ الاتنين دول بالشكل ده

### `Course` vs `CourseEntity`

> **Course represents the business/domain model, while CourseEntity represents the persistence/database model.**

### `CourseRepository` vs `CourseJpaRepository`

> **CourseRepository is an abstraction/port used by the application, while CourseJpaRepository is a Spring Data JPA repository used by the infrastructure to access the database.**

والاتنين مع بعض بيمشوا على نفس الفكرة:

```text
        DOMAIN                    INFRASTRUCTURE

       Course                  CourseEntity
          ↕                         ↕
       Mapper                    Mapper
          ↕                         ↕
CourseRepository  ← Adapter →  CourseJpaRepository
    (interface)                 (Spring/JPA)
```

**الفكرة الأساسية كلها:**  
إحنا بنفصل **Business Logic عن Persistence Logic**.  
الـ Domain يعرف الـ business، والـ Infrastructure تعرف JPA/Database.


