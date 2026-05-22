# 🎓 SkillHub Learning Platform API

## منصة إدارة كورسات وتعلم

Backend احترافي مبني بـ: **Spring Boot + Clean Architecture + JPA + Security**

---

# 🧠 فكرة النظام باختصار

منصة تعلم مثل Udemy:

- المدرس ينشئ كورسات ودروس
- الطالب يشترك ويتعلم
- تقييمات + تقدم الطالب
- Admin يدير المنصة

---

# 👥 ROLES

```text
STUDENT     -> يشترك في الكورسات ويتعلم
INSTRUCTOR  -> ينشئ كورسات ودروس
ADMIN       -> إدارة المنصة
```

---

# 🧱 BASE ENTITY (الجديد ✨)

قبل ما نشوف الجداول، هنعمل **MappedSuperclass** مشتركة بين كل الـ Entities:

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

وفي الـ `JpaConfig` لازم تضيف:

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig { }
```

كل Entity دلوقتي هترث منها:

```java
public class Course extends BaseEntity { ... }
public class User extends BaseEntity { ... }
// وهكذا
```

---

# 🗄️ DATABASE DESIGN

## 🟦 User Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|name|String|
|email|String UNIQUE|
|password|String|
|role|ENUM|
|enabled|Boolean|

---

## 🟩 Course Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|title|String|
|description|TEXT|
|price|Double|
|level|ENUM|
|status|ENUM|
|instructor_id|FK → users|

---

## 🟨 Section Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|title|String|
|order_index|int|
|course_id|FK|

---

## 🟥 Lesson Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|title|String|
|video_url|String|
|duration|int|
|is_preview|Boolean|
|section_id|FK|

---

## 🟪 Enrollment Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|student_id|FK → users|
|course_id|FK → courses|
|progress_percentage|int|

> `enrolled_at` اتغير لـ `createdAt` الموروث ✅

---

## 🟫 Review Table

_يرث من BaseEntity: `id`, `createdAt`, `updatedAt`_

|column|type|
|---|---|
|rating|int|
|comment|TEXT|
|student_id|FK|
|course_id|FK|

> `created_at` اتغير لـ `createdAt` الموروث ✅

User
 ├── @OneToMany → courses    (instructor)
 ├── @OneToMany → enrollments (student)
 └── @OneToMany → reviews    (student)

Course
 ├── @ManyToOne → User       (instructor)
 ├── @OneToMany → sections
 ├── @OneToMany → enrollments
 └── @OneToMany → reviews

Section
 ├── @ManyToOne → Course
 └── @OneToMany → lessons

Lesson
 └── @ManyToOne → Section

Enrollment
 ├── @ManyToOne → User  (student)
 └── @ManyToOne → Course

Review
 ├── @ManyToOne → User  (student)
 └── @ManyToOne → Course

---

# 🔗 العلاقات بين الجداول

```text
User (Instructor) 1 -------- * Course
Course 1 -------- * Section
Section 1 -------- * Lesson
User (Student) * -------- * Course  via Enrollment
Course 1 -------- * Review
User (Student) 1 -------- * Review
```

---

# 🧩 ENUMS

```
Role       -> STUDENT, INSTRUCTOR, ADMIN
CourseLevel -> BEGINNER, INTERMEDIATE, ADVANCED
CourseStatus -> DRAFT, PUBLISHED, ARCHIVED
```

---

# 🏗️ PROJECT STRUCTURE الكامل

```
skillhub
│
├── api
│   └── controllers
│           AuthController
│           CourseController
│           EnrollmentController
│           ReviewController
│
├── application
│   ├── dto
│   │     request/
│   │     response/
│   │
│   ├── services
│   │     ├── auth
│   │     │     AuthService / AuthServiceImpl
│   │     ├── course
│   │     │     CourseService / CourseServiceImpl
│   │     ├── content
│   │     │     ContentService / ContentServiceImpl
│   │     ├── enrollment
│   │     │     EnrollmentService / EnrollmentServiceImpl
│   │     └── review
│   │           ReviewService / ReviewServiceImpl
│   │
│   ├── mapper
│   │     UserMapper
│   │     CourseMapper
│   │     EnrollmentMapper
│   │
│   ├── security
│   │     JwtService
│   │     JwtFilter
│   │     CustomUserDetailsService
│   │     SecurityConfig
│   │
│   ├── exceptions
│   │     GlobalExceptionHandler
│   │     ResourceNotFoundException
│   │     BadRequestException
│   │     UnauthorizedException
│   │
│   └── responses
│         ApiResponse
│
├── domain
│   ├── models
│   │     BaseEntity        ⬅️ الجديد ✨
│   │     User
│   │     Course
│   │     Section
│   │     Lesson
│   │     Enrollment
│   │     Review
│   │
│   └── enums
│         Role
│         CourseLevel
│         CourseStatus
│
├── infrastructure
│   ├── repository
│   │     UserRepository
│   │     CourseRepository
│   │     EnrollmentRepository
│   │     ReviewRepository
│   │
│   └── config
│         JpaConfig         ⬅️ فيها @EnableJpaAuditing
│         SwaggerConfig
│
└── shared
    ├── utils
    └── constants
```

---

# 🔐 AUTH SYSTEM

**Register** — encode password → save user → return JWT

**Login** — validate credentials → generate JWT

---

# 📦 UNIFIED API RESPONSE

```json
{
  "success": true,
  "message": "Course fetched",
  "data": {},
  "timestamp": ""
}
```

---

# 🧨 EXCEPTION HANDLING

Global Handler يمسك:

- `ResourceNotFoundException` → 404
- `BadRequestException` → 400
- `UnauthorizedException` → 401
- `MethodArgumentNotValidException` → 422

---

# 📅 ROADMAP التنفيذ

|Phase|المحتوى|
|---|---|
|**1**|Project Setup + Structure + BaseEntity|
|**2**|User + Security + JWT|
|**3**|Course CRUD|
|**4**|Sections + Lessons|
|**5**|Enrollment|
|**6**|Reviews|
|**7**|Pagination + Filtering|
|**8**|Polish + Swagger|

---

> الفايدة الكبيرة من `BaseEntity` إنك بقيت بتكتب الـ `id` و `createdAt` و `updatedAt` **مرة واحدة بس** — وكل Entity بترث منها تلقائياً بدون تكرار 🎯