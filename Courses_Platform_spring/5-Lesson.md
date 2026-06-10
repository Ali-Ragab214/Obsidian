بناءً على مشروعك، دي الـ methods اللي هتحتاجها:

---

## CRUD الأساسي

|Method|Description|
|---|---|
|`createLesson(Long sectionId, LessonRequest request)`|إنشاء lesson جديدة لـ section معينة|
|`updateLesson(Long lessonId, LessonRequest request)`|تعديل بيانات lesson|
|`deleteLesson(Long lessonId)`|حذف lesson|
|`getLessonById(Long lessonId)`|جيب lesson بالـ ID|

---

## Methods خاصة بالـ Section

|Method|Description|
|---|---|
|`getLessonsBySection(Long sectionId, Pageable pageable)`|كل lessons الـ section مع pagination|
|`reorderLessons(Long sectionId, List<Long> lessonIds)`|إعادة ترتيب الـ lessons عن طريق الـ orderIndex|

---

## Methods خاصة بالـ Preview

|Method|Description|
|---|---|
|`getPreviewLessons(Long sectionId)`|جيب الـ lessons اللي `isPreview = true` بس|

---

## ⚠️ مش هتحتاجها دلوقتي

- `getLessonsByCourse` — دي هتبقى في الـ **CourseService** مش هنا
- `markAsCompleted` — دي هتبقى في الـ **EnrollmentService** أو **ProgressService**

---

عايز أكتبلك الـ Service كاملة؟