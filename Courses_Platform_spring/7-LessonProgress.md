## ازاي تهندلها في الـ Backend

---

### الـ Endpoint

```
POST /api/progress/{lessonId}/complete    → Mark as Complete
DELETE /api/progress/{lessonId}/complete  → Mark as Incomplete
```

---

### الـ Logic

**Mark as Complete:**

```
POST /api/progress/{lessonId}/complete
        │
        ▼
احفظ LessonProgress record
        │
        ▼
احسب Progress واحدّث Enrollment
```

**Mark as Incomplete:**

```
DELETE /api/progress/{lessonId}/complete
        │
        ▼
امسح LessonProgress record
        │
        ▼
احسب Progress واحدّث Enrollment
```

---

### الـ Repository

```java
public interface LessonProgressRepository extends JpaRepository<LessonProgress, Long> {

    // هل خلص الـ Lesson دي؟
    boolean existsByStudentIdAndLessonId(Long studentId, Long lessonId);

    // امسح الـ record
    void deleteByStudentIdAndLessonId(Long studentId, Long lessonId);

    // كام Lesson خلص في الـ Course دي؟
    long countByStudentIdAndLessonSectionCourseId(Long studentId, Long courseId);
}
```

---

### حساب الـ Progress

```java
// كام Lesson في الـ Course؟
long totalLessons = lessonRepository.countBySectionCourseId(courseId);

// كام Lesson خلص؟
long completedLessons = lessonProgressRepository
    .countByStudentIdAndLessonSectionCourseId(studentId, courseId);

// الـ Progress
int progress = (int) ((completedLessons * 100) / totalLessons);

// حدّث الـ Enrollment
enrollment.setProgress(progress);
enrollmentRepository.save(enrollment);
```

---

### ببساطة

|Action|Backend|
|---|---|
|ضغط Complete|`save` record في الـ DB|
|ضغط Incomplete|`delete` record من الـ DB|
|في الحالتين|احسب Progress وحدّث الـ Enrollment|

---

نبدأ نكتب الـ `LessonProgressRepository` و `EnrollmentRepository` دلوقتي؟

## الـ Flow الكامل لـ LessonProgress

---

## هل هتستخدمها في الـ EnrollmentServiceImpl؟

**لأ** — هتبقى **service منفصلة** ✅

```
EnrollmentService     → بيتعامل مع الاشتراك في الـ Course
LessonProgressService → بيتعامل مع إتمام الـ Lessons
```

---

## الـ Flow الكامل

### 1. Mark as Complete

```
Student ضغط "Mark as Complete" على Lesson
        │
        ▼
LessonProgressService.markAsComplete(lessonId, studentEmail)
        │
        ├── جيب الـ Student من الـ DB
        │
        ├── جيب الـ Lesson من الـ DB
        │       └── مش موجودة → 404
        │
        ├── الـ Student مشترك في الـ Course؟
        │       └── لأ → 403
        │
        ├── خلص الـ Lesson دي قبل كده؟
        │       └── آه → 409
        │
        ├── احفظ LessonProgress record ✅
        │
        └── احسب Progress وحدّث Enrollment
                │
                ├── كام Lesson في الـ Course؟
                ├── كام Lesson خلص؟
                └── progress = completed/total × 100
```

### 2. Mark as Incomplete

```
Student ضغط "Mark as Incomplete"
        │
        ▼
LessonProgressService.markAsIncomplete(lessonId, studentEmail)
        │
        ├── جيب الـ Student
        │
        ├── الـ Record موجود أصلاً؟
        │       └── لأ → 404
        │
        ├── امسح الـ LessonProgress record
        │
        └── احسب Progress وحدّث Enrollment
```

### 3. Get Completed Lessons

```
Student عايز يشوف الـ Lessons اللي خلصها في Course
        │
        ▼
LessonProgressService.getCompletedLessons(courseId, studentEmail)
        │
        └── ارجع List<LessonResponse> للـ Lessons المكتملة
```

---

## العلاقة بين الـ Services

```
LessonProgressService
        │
        │ بيحتاج
        ▼
EnrollmentRepository  ← للتحقق إن الـ Student مشترك
LessonRepository      ← للتحقق إن الـ Lesson موجودة
LessonProgressRepository ← للحفظ والمسح
EnrollmentRepository  ← لتحديث الـ Progress
```

---

## الـ Services والـ Repositories

```
LessonProgressService (Interface)
├── markAsComplete(lessonId, studentEmail)
├── markAsIncomplete(lessonId, studentEmail)
└── getCompletedLessons(courseId, studentEmail)

LessonProgressServiceImpl
├── LessonProgressRepository
├── LessonRepository
└── EnrollmentRepository  ← بس للقراءة وتحديث الـ Progress
```

---

## الـ Endpoints

```
POST   /api/progress/{lessonId}/complete   → markAsComplete
DELETE /api/progress/{lessonId}/complete   → markAsIncomplete
GET    /api/progress/{courseId}/completed  → getCompletedLessons
```

---

## ملخص العلاقات

```
EnrollmentService
└── مسؤول عن: enroll, unenroll, getMyEnrollments
└── مش بيتعامل مع LessonProgress مباشرة

LessonProgressService
└── مسؤول عن: markAsComplete, markAsIncomplete
└── بيحدّث Enrollment.progress بعد كل عملية
└── بيتحقق من الـ Enrollment قبل أي عملية
```

---

نبدأ بكتابة الـ Repositories الأول؟



آه بالظبط ✅

---

## الترتيب الصح

```
1. Enrollment (كامل)
   ├── EnrollmentRepository
   ├── EnrollmentRequest
   ├── EnrollmentResponse
   ├── EnrollmentMapper
   ├── EnrollmentService
   ├── EnrollmentServiceImpl
   └── EnrollmentController

        ↓ بعد ما تخلص

2. LessonProgress
   ├── LessonProgressRepository
   ├── LessonProgressService
   ├── LessonProgressServiceImpl
   └── LessonProgressController
```

---

## ليه الترتيب ده؟

لأن الـ `LessonProgressService` هيحتاج الـ `EnrollmentRepository` عشان يتحقق إن الـ Student مشترك — فلازم الـ Enrollment يبقى جاهز الأول.

---

نبدأ بالـ `EnrollmentRepository` دلوقتي؟