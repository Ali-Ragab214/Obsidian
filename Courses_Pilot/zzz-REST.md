بص يا علي، بما إنك **خلصت Course + Section + Lesson** فأنت عملت أصعب جزء في المشروع. من وجهة نظري أنت دلوقتي خلصت حوالي **70% من SkillHub**.

اللي باقي هو الـ Business Features الحقيقية. وأنا هديك البلان اللي أنا كنت همشي بيها لو ده مشروعي.

---

# الوضع الحالي

```text
✅ Domain
✅ Security (JWT)
✅ Auth
✅ Course
✅ Section
✅ Lesson
✅ DTOs الخاصة بيهم
✅ Mappers الخاصة بيهم
✅ Controllers الخاصة بيهم
✅ Repositories الخاصة بيهم
```

---

# المرحلة القادمة

```text
Enrollment
↓
Review
↓
SecurityUtils
↓
Swagger (DONE)
↓
Testing
↓
Docker
↓
Redis
↓
Deployment
```

---

# Phase 1: Enrollment ⭐⭐⭐⭐⭐

دي أهم حاجة دلوقتي.

---

## الفكرة

Student يعمل تسجيل في كورس.

```text
Student
    ↓
Enrollment
    ↓
Course
```

---

## السيناريو

Student:

```http
POST /api/courses/{courseId}/enroll
```

↓

يتأكد إن:

```text
✓ الكورس موجود
✓ المستخدم Student
✓ المستخدم مش مسجل بالفعل
```

↓

يحفظ Enrollment.

---

## المطلوب

### DTO

```java
EnrollmentResponse
```

بصراحة غالبًا مش محتاج:

```java
EnrollmentRequest
```

لأن الـ courseId هيجي من الـ URL.

---

### Mapper

```java
EnrollmentMapper
```

---

### Repository

```java
existsByUserIdAndCourseId()

findByUserId()

findByUserIdAndCourseId()
```

---

### Service

```java
EnrollmentResponse enroll(Long courseId,String email);

void cancelEnrollment(Long courseId,String email);

PaginationResponse<EnrollmentResponse> getMyEnrollments(...);
```

---

### Controller

```http
POST   /api/courses/{courseId}/enroll

DELETE /api/courses/{courseId}/enroll

GET    /api/enrollments/me
```

---

## Business Rules

لازم تطبق:

```java
if(alreadyEnrolled)
    throw new BadRequestException(...);
```

---

# Phase 2: Review ⭐⭐⭐⭐

بعد Enrollment مباشرة.

---

## الفكرة

Student يعمل تقييم للكورس.

---

## السيناريو

```http
POST /api/courses/{courseId}/reviews
```

↓

يتأكد:

```text
✓ Student
✓ Enrolled
✓ لم يقيم من قبل
```

↓

يحفظ Review.

---

## DTO

```java
ReviewRequest

ReviewResponse
```

---

## Mapper

```java
ReviewMapper
```

---

## Repository

```java
existsByUserIdAndCourseId()

findByCourseId()

findByUserIdAndCourseId()
```

---

## Service

```java
ReviewResponse createReview(...);

PaginationResponse<ReviewResponse> getCourseReviews(...);

void deleteReview(...);
```

---

## Controller

```http
POST   /api/courses/{courseId}/reviews

GET    /api/courses/{courseId}/reviews

DELETE /api/reviews/{id}
```

---

## Business Rules

مثال:

```java
if(!enrolled)
    throw new UnauthorizedException(...);

if(alreadyReviewed)
    throw new BadRequestException(...);
```

---

# Phase 3: SecurityUtils ⭐⭐⭐

بعد ما Enrollment وReview يخلصوا.

بدل:

```java
Authentication authentication
```

في كل Controller.

اعمل:

```java
public class SecurityUtils {

    public static String getCurrentUsername() {
        Authentication authentication =
                SecurityContextHolder.getContext().getAuthentication();

        return authentication.getName();
    }
}
```

---

واستخدم:

```java
String email = SecurityUtils.getCurrentUsername();
```

---

# Phase 4: Swagger ⭐⭐

بعد ما كل APIs تشتغل.

راجع:

```text
Auth
Course
Section
Lesson
Enrollment
Review
```

---

جرّب كل حاجة من:

```text
http://localhost:8080/swagger-ui/index.html
```

---

# Phase 5: Testing ⭐⭐⭐⭐⭐

دي أهم حاجة بعد الـ Business Logic.

---

## ابدأ بـ Service Tests

### EnrollmentServiceTest

اختبر:

```text
✓ enroll success

✓ already enrolled

✓ course not found
```

---

### ReviewServiceTest

اختبر:

```text
✓ review success

✓ not enrolled

✓ duplicate review
```

---

### LessonServiceTest

اختبر:

```text
✓ create

✓ update

✓ delete
```

---

استخدم:

```java
Mockito

JUnit 5
```

---

# Phase 6: Docker ⭐⭐⭐

بعد ما المشروع كله يبقى Stable.

---

## Dockerfile

للـ Spring Boot.

---

## docker-compose.yml

هيحتوي:

```yaml
postgres
redis
skillhub-app
```

---

هتتعلم:

```text
Volumes

Networks

Environment Variables
```

---

# Phase 7: Redis ⭐⭐⭐

اعمله Enhancement.

---

## Cache

على:

```java
getCourseById()

getAllCourses()
```

---

باستخدام:

```java
@EnableCaching

@Cacheable

@CacheEvict
```

---

مثال:

```java
@Cacheable(value = "courses", key = "#id")
public CourseResponse getCourseById(Long id) {
    ...
}
```

---

# Phase 8: Deployment ⭐⭐

آخر خطوة.

---

انشر المشروع باستخدام Docker على:

- Railway
    
- Render
    
- VPS
    

---

# البلان النهائية

```text
1. Enrollment
2. Review
3. SecurityUtils
4. Swagger Testing
5. Unit Testing
6. Docker + Docker Compose
7. Redis Cache
8. Deployment
```

---

## لو كنت مكانك

أنا كنت هعمل كده الأسبوع ده:

```text
اليوم 1:
Enrollment كامل

اليوم 2:
Review كامل

اليوم 3:
Swagger + SecurityUtils

اليوم 4:
Unit Testing

اليوم 5:
Docker + Compose

اليوم 6:
Redis

اليوم 7:
Deployment
```

وبكده SkillHub هيبقى مشروع قوي جدًا فيه:

- Spring Boot
    
- Spring Security + JWT
    
- JPA
    
- Validation
    
- Global Exception Handling
    
- Swagger
    
- Unit Testing
    
- Docker
    
- Redis
    
- Deployment
    

وده مستوى ممتاز جدًا لمشروع Portfolio أو حتى تتكلم عنه بثقة في أي إنترفيو Backend Java/Spring Boot.