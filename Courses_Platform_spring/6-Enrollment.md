بص يا علي، بعد ما قريت اللي أنت خلصته وفكرة المشروع، أنا شايف إنك وصلت لمرحلة ممتازة جدًا. لو كنت Team Lead للمشروع ده، فدي الخطة اللي هحطها ليك من دلوقتي لحد النسخة الأولى (MVP) من SkillHub.

---

# أنت دلوقتي فين؟

```text
✅ Domain Design
✅ JWT + Security
✅ Auth
✅ Course Module
✅ Section Module
✅ Lesson Module
```

يعني أنت خلصت الـ Content Management System الخاص بالـ Instructor.

دلوقتي هنبدأ ندخل في الجزء اللي بيخلي المنصة "منصة تعليم" فعلًا.

---

# المرحلة القادمة: Enrollment Module ⭐⭐⭐⭐⭐

دي أهم مرحلة حاليًا.

---

## الفكرة

الطالب يسجل في كورس.

```text
Student
   ↓
Enrollment
   ↓
Course
```

---

## السيناريو الكامل

### Student يضغط Enroll

```http
POST /api/courses/{courseId}/enroll
```

↓

JWT يحدد مين الطالب.

↓

السيستم ينفذ اللوجيك.

---

# اللوجيك المطلوب

## 1) التأكد إن المستخدم موجود

أنت بالفعل بتجيب الإيميل من الـ JWT.

يبقى:

```text
email
↓
UserRepository
↓
User student
```

---

## 2) التأكد إنه Student

```text
لو Role != STUDENT

↓

401 أو 403
```

لأن الـ Instructor مينفعش يسجل في كورس.

---

## 3) التأكد إن الكورس موجود

```text
CourseRepository

findById(courseId)
```

لو مش موجود:

```text
404
```

---

## 4) التأكد إن الكورس منشور

ودي نقطة ناس كتير بتنسيها.

```text
Course.status == PUBLISHED
```

لو:

```text
DRAFT
ARCHIVED
```

يبقى ممنوع التسجيل.

---

## 5) التأكد إنه مش مسجل بالفعل

ودي أهم Rule.

```text
existsByStudentIdAndCourseId(...)
```

لو True:

```text
BadRequestException

"You are already enrolled"
```

---

## 6) إنشاء Enrollment

```text
student
course
progress = 0
createdAt = تلقائي
```

---

## 7) حفظه

```text
save(enrollment)
```

---

## 8) إرجاع EnrollmentResponse

يحتوي مثلاً على:

```text
EnrollmentId
CourseTitle
StudentName
Progress
EnrolledAt
```

---

# APIs المطلوبة

## التسجيل

```http
POST /api/courses/{courseId}/enroll
```

---

## إلغاء التسجيل

```http
DELETE /api/courses/{courseId}/enroll
```

---

# لوجيك الإلغاء

## التأكد من:

```text
✓ الطالب موجود
✓ الكورس موجود
✓ مسجل بالفعل
```

↓

احذف الـ Enrollment.

---

## جلب كورساتي

```http
GET /api/enrollments/me
```

---

# اللوجيك

```text
JWT
↓
current student
↓
findByStudent(...)
↓
Pagination
↓
EnrollmentResponse
```

---

## ممكن تضيف

```text
page
size
sort
```

من البداية.

---

# مرحلة اختيارية بعد Enrollment

## منع الوصول للدروس

الفكرة:

لو Lesson ليست Preview.

```text
isPreview == false
```

↓

لازم يكون الطالب مسجل.

---

يعني:

```text
Student غير مسجل
↓
يشوف Preview فقط

Student مسجل
↓
يشوف كل الدروس
```

ودي من أجمل الـ Business Rules في SkillHub.
# طيب في SkillHub بتاعك هتتطبق إزاي؟

في الـ Lesson Entity:

```
private Boolean isPreview = false;
```

ولما تجيب Lesson:

```
لو isPreview == true    اسمح لأي حد يشوفها.لو isPreview == false    اتأكد إن فيه Enrollment بين الطالب والكورس.
```

يعني اللوجيك هيبقى:

```
Lesson↓هل Preview؟    ↓ نعم        Return Lesson    ↓ لا        هل المستخدم مسجل؟            ↓ نعم                Return Lesson            ↓ لا                403 Forbidden
```

---

# المرحلة اللي بعدها: Review ⭐⭐⭐⭐

ودي مرتبطة بالـ Enrollment.

---

## الشرط الأساسي

لازم يكون:

```text
Student
AND
Enrolled
```

---

## السيناريو

```http
POST /api/courses/{courseId}/reviews
```

---

# اللوجيك

## التأكد إن الكورس موجود

↓

## التأكد إن المستخدم Student

↓

## التأكد إنه مسجل

```text
EnrollmentRepository
```

↓

## التأكد إنه لم يراجع من قبل

```text
existsByStudentIdAndCourseId(...)
```

↓

لو عمل Review بالفعل:

```text
400

"You have already reviewed this course"
```

↓

احفظ الـ Review.

---

# APIs الخاصة بالـ Review

## إنشاء تقييم

```http
POST /api/courses/{courseId}/reviews
```

---

## عرض تقييمات كورس

```http
GET /api/courses/{courseId}/reviews
```

---

## حذف تقييم

```http
DELETE /api/reviews/{id}
```

---

# مين يقدر يحذف؟

عندك اختيارين:

## الأول

```text
صاحب التقييم فقط
```

---

## الثاني

```text
ADMIN
أو
صاحب التقييم
```

وأنا أفضل الثاني.

---

# المرحلة التالية: Pagination & Filtering ⭐⭐⭐

بعد ما Enrollment وReview يخلصوا.

راجع كل الـ GETs.

مثلاً:

```http
GET /courses?page=0&size=10
```

```http
GET /courses?level=BEGINNER
```

```http
GET /courses?keyword=spring
```

---

# المرحلة التالية: Swagger ⭐⭐

اختبر السيناريوهات الحقيقية.

## Instructor

```text
Register
Login
Create Course
Create Section
Create Lesson
```

---

## Student

```text
Register
Login
Enroll
Review
Browse Courses
```

---

# المرحلة التالية: SecurityUtils ⭐⭐

اعمل Utility تجيب المستخدم الحالي.

بدل ما كل Controller يبقى فيه:

```text
Authentication authentication
```

---

# المرحلة التالية: Unit Testing ⭐⭐⭐⭐⭐

ودي هتفرق جدًا في مستوى المشروع.

اختبر:

## EnrollmentService

```text
Enroll Success

Already Enrolled

Course Not Found

Course Not Published
```

---

## ReviewService

```text
Review Success

Not Enrolled

Duplicate Review
```

---

# المرحلة التالية: Docker ⭐⭐⭐

اعمل:

```text
Dockerfile
docker-compose
```

للـ:

```text
Spring Boot
PostgreSQL
Redis
```

---

# المرحلة التالية: Redis ⭐⭐⭐

أول استخدام ليه في SkillHub:

## Cache

لـ:

```text
Get Course By Id

Get All Courses
```

باستخدام:

```text
@Cacheable
@CacheEvict
```

---

# المرحلة الأخيرة: Deployment ⭐⭐

انشر المشروع.

مثلاً:

```text
Render
Railway
VPS
```

---

# الخطة النهائية للمشروع

```text
[الآن]
Enrollment
↓
Review
↓
Pagination & Filtering
↓
Swagger End-to-End Testing
↓
SecurityUtils
↓
Unit Testing
↓
Docker + Compose
↓
Redis Cache
↓
Deployment
```

## ملاحظتين مهمين جدًا

### 1) بالنسبة لـ EnrollmentRequest

أنا شخصيًا **مش هعمله**.

لأن:

```http
POST /courses/{courseId}/enroll
```

كل البيانات موجودة بالفعل:

- الـ Student من JWT.
    
- الـ Course من Path Variable.
    

فمش محتاج Request Body أصلًا.

---

### 2) بعد ما تخلص Enrollment وReview

أنا أنصحك تضيف Feature صغيرة قوية جدًا:

```text
Course Progress Tracking
```

يعني كل مرة الطالب يخلص Lesson:

```http
POST /lessons/{lessonId}/complete
```

↓

يتحدث:

```text
Enrollment.progressPercentage
```

ودي Feature موجودة في Udemy فعلًا، وهتخلي SkillHub مشروع أقوى بكتير في الـ CV وفي الإنترفيو.