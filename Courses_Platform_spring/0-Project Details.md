--# 🎓 SkillHub Learning Platform API

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
│           AuthController             ← Phase 4
│           CourseController           ← Phase 4
│           EnrollmentController       ← Phase 4
│           ReviewController           ← Phase 4
│
├── application
│   ├── dto
│   │     ├── request
│   │     │     RegisterRequest        ← Phase 3
│   │     │     LoginRequest           ← Phase 3
│   │     │     CourseRequest          ← Phase 3
│   │     │     SectionRequest         ← Phase 3
│   │     │     LessonRequest          ← Phase 3
│   │     │     EnrollmentRequest      ← Phase 3
│   │     │     ReviewRequest          ← Phase 3
│   │     │
│   │     └── response
│   │           UserResponse           ← Phase 3
│   │           AuthResponse           ← Phase 3
│   │           CourseResponse         ← Phase 3
│   │           SectionResponse        ← Phase 3
│   │           LessonResponse         ← Phase 3
│   │           EnrollmentResponse     ← Phase 3
│   │           ReviewResponse         ← Phase 3
│   │
│   ├── services
│   │     ├── auth
│   │     │     AuthService            ← Phase 3
│   │     │     AuthServiceImpl        ← Phase 3
│   │     ├── course
│   │     │     CourseService          ← Phase 3
│   │     │     CourseServiceImpl      ← Phase 3
│   │     ├── content
│   │     │     ContentService         ← Phase 3
│   │     │     ContentServiceImpl     ← Phase 3
│   │     ├── enrollment
│   │     │     EnrollmentService      ← Phase 3
│   │     │     EnrollmentServiceImpl  ← Phase 3
│   │     └── review
│   │           ReviewService          ← Phase 3
│   │           ReviewServiceImpl      ← Phase 3
│   │
│   ├── mapper
│   │     UserMapper                   ← Phase 3
│   │     CourseMapper                 ← Phase 3
│   │     SectionMapper                ← Phase 3
│   │     LessonMapper                 ← Phase 3
│   │     EnrollmentMapper             ← Phase 3
│   │     ReviewMapper                 ← Phase 3
│   │
│   ├── security
│   │     JwtService                   ← Phase 2
│   │     JwtFilter                    ← Phase 2
│   │     CustomUserDetailsService     ← Phase 2
│   │     SecurityConfig               ← Phase 2
│   │
│   ├── exceptions
│   │     GlobalExceptionHandler       ← Phase 2
│   │     ResourceNotFoundException    ← Phase 2
│   │     BadRequestException          ← Phase 2
│   │     UnauthorizedException        ← Phase 2
│   │
│   └── responses
│         ApiResponse                  ← Phase 2
│
├── domain
│   ├── models
│   │     BaseEntity                   ← Phase 1 ✅
│   │     User                         ← Phase 1 ✅
│   │     Course                       ← Phase 1 ✅
│   │     Section                      ← Phase 1 ✅
│   │     Lesson                       ← Phase 1 ✅
│   │     Enrollment                   ← Phase 1 ✅
│   │     Review                       ← Phase 1 ✅
│   │
│   └── enums
│         Role                         ← Phase 1 ✅
│         CourseLevel                  ← Phase 1 ✅
│         CourseStatus                 ← Phase 1 ✅
│
├── infrastructure
│   ├── repository
│   │     UserRepository               ← Phase 2
│   │     CourseRepository             ← Phase 2
│   │     SectionRepository            ← Phase 2
│   │     LessonRepository             ← Phase 2
│   │     EnrollmentRepository         ← Phase 2
│   │     ReviewRepository             ← Phase 2
│   │
│   └── config
│         JpaConfig                    ← Phase 2
│         SwaggerConfig                ← Phase 2
│
└── shared
    ├── utils
    │     SecurityUtils                ← Phase 2
    └── constants
          AppConstants                 ← Phase 2
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

---











تفضل يا غالي، ده الـ **Project Structure** الكامل بعد ما دمجنا فيه أجزاء الـ **Security** ووزعناها صح على طبقات الـ **Clean Architecture**، عشان تضمن إن الـ Business logic (الـ Domain) يفضل نضيف ومعزول عن أي مكتبات خارجية (مكتبات الـ JWT مثلاً مكانها الـ Infrastructure).

Plaintext

```
skillhub
│
├── domain
│   ├── models
│   │     ├── BaseEntity                   ← (Phase 1 ✅)
│   │     ├── User                         ← (يرث من BaseEntity ويعمل Implement لـ UserDetails)
│   │     ├── Course                       ← (Phase 1 ✅)
│   │     ├── Section                      ← (Phase 1 ✅)
│   │     ├── Lesson                       ← (Phase 1 ✅)
│   │     ├── Enrollment                   ← (Phase 1 ✅)
│   │     └── Review                       ← (Phase 1 ✅)
│   │
│   └── enums
│         ├── Role                         ← (STUDENT, INSTRUCTOR, ADMIN)
│         ├── CourseLevel                  ← (BEGINNER, INTERMEDIATE, ADVANCED)
│         └── CourseStatus                 ← (DRAFT, PUBLISHED, ARCHIVED)
│
├── application
│   ├── security
│   │     ├── JwtService                   ← (Interface لتوليد وفك الـ Tokens)
│   │     └── CustomUserDetailsService     ← (تحميل الـ User من الداتابيز لـ Spring Security)
│   │
│   ├── dto
│   │     ├── request
│   │     │     ├── RegisterRequest        ← (بيانات التسجيل: الإسم، الإيميل، الباسورد، الـ Role)
│   │     │     └── LoginRequest           ← (الإيميل والباسورد)
│   │     └── response
│   │           ├── AuthResponse           ← (يرجع الـ Token وبيانات المستخدم الأساسية)
│   │           └── UserResponse           
│   │
│   ├── services
│   │     └── auth
│   │           ├── AuthService            ← (Interface لعمليات الـ Login/Register)
│   │           └── AuthServiceImpl        
│   │
│   ├── mapper
│   │     └── UserMapper                   ← (تحويل الـ User لـ DTOs والعكس)
│   │
│   └── exceptions
│         ├── GlobalExceptionHandler       ← (يمسك الـ BadCredentials والـ AccessDenied)
│         ├── ResourceNotFoundException    ← (404)
│         ├── BadRequestException          ← (400)
│         └── UnauthorizedException        ← (401)
│
├── infrastructure
│   ├── repository
│   │     └── UserRepository               ← (السند والأساس للـ Security عشان يتأكد من الإيميل)
│   │
│   ├── security
│   │     ├── JwtServiceImpl               ← (الـ Implementation الفعلي بمكتبة io.jsonwebtoken)
│   │     ├── JwtAuthFilter                ← (الـ Filter اللي بيقرا الـ Bearer Token من الـ Header)
│   │     └── SecurityConfig               ← (قفل وفتح الـ Endpoints وتشفير الباسوردات)
│   │
│   └── config
│         ├── JpaConfig                    ← (تفعيل الـ @EnableJpaAuditing)
│         └── SwaggerConfig                ← (إضافة زرار الـ Authorize Bearer في الـ UI)
│
├── api
│   └── controllers
│         └── AuthController               ← (Endpoints: /api/v1/auth/register & /login)
│
└── shared
    ├── utils
    │     └── SecurityUtils                ← (هيجيب لك الـ Current User ID من الـ SecurityContextHolder)
    └── constants
          └── AppConstants                 
```

### 🔄 مسار الـ Request بيمشي إزاي في الـ Security ديجرام ده؟

1. **الـ Client** بيبعت الـ Request ومعاه الـ `Authorization: Bearer <token>` في الـ Header.
    
2. أول حاجة تستقبله في الـ Infrastructure هي الـ **`JwtAuthFilter`**، تقرا الـ Token وتبعته للـ **`JwtService`** عشان تتأكد إنه سليم ومش منتهي الصلاحية.
    
3. الـ Filter يكلم الـ **`CustomUserDetailsService`** عشان يتأكد إن الـ User ده لسه موجود ونشط في الـ **`UserRepository`**.
    
4. لو كله تمام، الـ Filter بيحط الـ User جوه الـ **`SecurityContextHolder`** ويعدي الـ Request للـ Controller.
    
5. جوه الـ Controller أو السيرفيس، لما تحب تعرف مين الـ User اللي بيكلمك عشان تربطه بكورس أو باشتراك، بتنادي على الـ **`SecurityUtils`** تجيب لك بياناته علطول بدون ما تسأله عنها في الـ Request body.
    

لو الـ Structure كده تمام في دماغك ومستعد، قولي عشان نبدأ نكتب كود الـ `User` وجزء الـ Configuration وتشفير الباسوردات علطول!




- DTOs
- Mapper
- Auth Service
- User Registration
- Login
- Password Encoding
- JWT
skillhub
│
├── domain  [done]
│   ├── models  [done]
│   │   ├── BaseEntity  [done]
│   │   ├── User  [done]
│   │   ├── Course  [done]
│   │   ├── Section  [done]
│   │   ├── Lesson  [done]
│   │   ├── Enrollment  [done]
│   │   └── Review  [done]
│   │
│   └── enums  [done]
│       ├── Role  [done]
│       ├── CourseLevel  [done]
│       └── CourseStatus  [done]
│
├── application
│   ├── dto
│   │   ├── request  [done/partial]
│   │   │   ├── RegisterRequest  [done]
│   │   │   ├── LoginRequest  [done]
│   │   │   ├── CourseRequest  [done/partial]
│   │   │   ├── SectionRequest  [done/partial]
│   │   │   ├── LessonRequest  [done/partial]
│   │   │   ├── EnrollmentRequest  [pending]
│   │   │   └── ReviewRequest  [pending]
│   │   │
│   │   └── response  [done/partial]
│   │       ├── AuthResponse  [done]
│   │       ├── UserResponse  [done]
│   │       ├── CourseResponse  [done/partial]
│   │       ├── SectionResponse  [done/partial]
│   │       ├── LessonResponse  [done/partial]
│   │       ├── EnrollmentResponse  [pending]
│   │       └── ReviewResponse  [pending]
│   │
│   ├── mapper  [done/partial]
│   │   ├── UserMapper  [done]
│   │   ├── CourseMapper  [done/partial]
│   │   ├── SectionMapper  [done/partial]
│   │   ├── LessonMapper  [done/partial]
│   │   ├── EnrollmentMapper  [pending]
│   │   └── ReviewMapper  [pending]
│   │
│   ├── services
│   │   ├── auth  [done/partial]
│   │   │   ├── AuthService  [done]
│   │   │   └── AuthServiceImpl  [done]
│   │   │
│   │   ├── course  [pending]
│   │   ├── content  [pending]
│   │   ├── enrollment  [pending]
│   │   └── review  [pending]
│   │
│   ├── security  [done]
│   │   ├── JwtService  [done]
│   │   ├── JwtFilter  [done]
│   │   ├── CustomUserDetailsService  [done]
│   │   └── SecurityConfig  [done]
│   │
│   ├── exceptions  [done]
│   │   ├── GlobalExceptionHandler  [done]
│   │   ├── ResourceNotFoundException  [done]
│   │   ├── BadRequestException  [done]
│   │   └── UnauthorizedException  [done]
│   │
│   └── responses  [done]
│       └── ApiResponse  [done]
│
├── api
│   └── controllers
│       ├── AuthController  [done]
│       ├── CourseController  [pending/partial]
│       ├── SectionController  [pending]
│       ├── LessonController  [pending]
│       ├── EnrollmentController  [pending]
│       └── ReviewController  [pending]
│
├── infrastructure
│   ├── repository  [done/partial]
│   │   ├── UserRepository  [done]
│   │   ├── CourseRepository  [done]
│   │   ├── SectionRepository  [done]
│   │   ├── LessonRepository  [done]
│   │   ├── EnrollmentRepository  [pending]
│   │   └── ReviewRepository  [pending]
│   │
│   ├── config  [done/partial]
│   │   ├── JpaConfig  [done]
│   │   └── SwaggerConfig  [pending]
│   │
│   └── security  [done]
│       ├── JwtServiceImpl  [done]
│       └── JwtAuthFilter  [done]
│
├── shared
│   ├── utils  [pending]
│   │   └── SecurityUtils  [pending]
│   └── constants  [pending]
│       └── AppConstants  [pending]
│
└── resources
    ├── application.properties  [done]
    └── db
        └── migration
            └── V2__update_lessons_table.sql  [done]