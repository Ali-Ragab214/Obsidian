# 👥 SkillHub — Roles & Permissions

---

## نظرة عامة

السيستم فيه 3 roles، كل واحد بيتسجل أو بيتعمل بطريقة مختلفة وليه صلاحيات محددة.

```
STUDENT     → بيسجل بنفسه، يشوف الكورسات، يدفع عشان يدخل المدفوع
INSTRUCTOR  → بيسجل بنفسه، ينشئ ويدير كورساته بس
ADMIN       → بيتعمل manually في الـ DB أو بـ endpoint مخصوص محمي
```

---

## 🔐 التسجيل والـ Default Role

### Register Endpoint — `/api/auth/register`

الـ Request بيجي فيه `role` كـ String:

```json
{
  "name": "Ahmed",
  "email": "ahmed@email.com",
  "password": "123456",
  "role": "STUDENT"   // أو "INSTRUCTOR"
}
```

**القاعدة في الـ AuthService:**

- لو الـ role جه `STUDENT` أو `INSTRUCTOR` → تمام، بنقبله
- لو الـ role جه `ADMIN` → نرمي `BadRequestException` فوراً
- لو الـ role مجاش خالص → `Default = STUDENT`

```java
// في AuthServiceImpl
if (request.getRole() == Role.ADMIN) {
    throw new BadRequestException("Cannot register as ADMIN");
}
Role role = request.getRole() != null ? request.getRole() : Role.STUDENT;
```

### إزاي بنعمل Admin؟

إما manually في الـ DB:

```sql
INSERT INTO users (name, email, password, role, enabled)
VALUES ('Admin', 'admin@skillhub.com', '<encoded_password>', 'ADMIN', true);
```

أو بـ endpoint مخصوص محمي بـ `@PreAuthorize("hasRole('ADMIN')")` — بس ده Phase تاني.

---

## 🎓 STUDENT — الصلاحيات

### بيشوف إيه بدون Enrollment؟

|المحتوى|متاح؟|
|---|---|
|قائمة الكورسات المنشورة (title, description, price, level)|✅|
|الكورسات المجانية (price = 0) كاملة|✅|
|Preview Lessons (`is_preview = true`)|✅|
|محتوى الكورس المدفوع (الدروس)|❌ محتاج Enrollment|
|Reviews على الكورسات|✅ يشوف بس|

### بعد الـ Enrollment (أو المجاني):

|العملية|متاح؟|
|---|---|
|مشاهدة كل الدروس في الكورس|✅|
|تحديث الـ `progress_percentage`|✅|
|كتابة Review على الكورس|✅ (مرة واحدة بس لكل كورس)|
|تعديل أو حذف Review بتاعته|✅|

### ممنوع تماماً:

- إنشاء أو تعديل أي كورس
- الوصول لبيانات Student تاني
- أي endpoint خاص بالـ Admin أو الـ Instructor

---

## 👨‍🏫 INSTRUCTOR — الصلاحيات

### إدارة الكورسات:

|العملية|القيد|
|---|---|
|إنشاء كورس جديد|✅ بيبقى `DRAFT` تلقائياً|
|تعديل كورس|✅ بس اللي هو عامله بس `instructor_id = me`|
|حذف كورس|✅ بس لو `DRAFT` — المنشور محتاج موافقة Admin (اختياري)|
|نشر الكورس `PUBLISHED`|✅|
|أرشفة الكورس `ARCHIVED`|✅|
|إضافة / تعديل / حذف Sections|✅ بس في كورساته|
|إضافة / تعديل / حذف Lessons|✅ بس في كورساته|

### يشوف إيه؟

|البيانات|متاح؟|
|---|---|
|قائمة طلابه (اللي اشتركوا في كورساته)|✅|
|إجمالي الـ Enrollments لكورساته|✅|
|الـ Reviews على كورساته|✅|
|بيانات Instructor تاني أو طلاب تانيين|❌|

### ممنوع تماماً:

- الاشتراك في كورسات كـ Student (مش في الـ MVP — ممكن تضيفه بعدين)
- تعديل كورس مش بتاعه → `UnauthorizedException`
- أي endpoint خاص بالـ Admin

---

## 🛡️ ADMIN — الصلاحيات

الـ Admin شايف كل حاجة وعنده full control:

|العملية||
|---|---|
|إدارة كل الـ Users (تفعيل / تعطيل)|✅|
|حذف أي كورس أو Review مخالف|✅|
|تغيير الـ CourseStatus لأي كورس|✅|
|مشاهدة كل الـ Enrollments والـ Reviews|✅|
|ترقية User لـ Instructor أو تخفيضه|✅|

---

## 🔒 Security Rules — ملخص للـ SecurityConfig

```java
.requestMatchers(HttpMethod.POST, "/api/auth/**").permitAll()
.requestMatchers(HttpMethod.GET,  "/api/courses/**").permitAll()        // يشوف الكل
.requestMatchers(HttpMethod.GET,  "/api/lessons/{id}").authenticated()  // بس بعد login

// STUDENT
.requestMatchers(HttpMethod.POST, "/api/enrollments/**").hasRole("STUDENT")
.requestMatchers(HttpMethod.POST, "/api/reviews/**").hasRole("STUDENT")

// INSTRUCTOR
.requestMatchers(HttpMethod.POST,   "/api/courses/**").hasRole("INSTRUCTOR")
.requestMatchers(HttpMethod.PUT,    "/api/courses/**").hasRole("INSTRUCTOR")
.requestMatchers(HttpMethod.DELETE, "/api/courses/**").hasRole("INSTRUCTOR")

// ADMIN
.requestMatchers("/api/admin/**").hasRole("ADMIN")

.anyRequest().authenticated()
```

---

## ⚠️ Business Rules مهمة تتطبق في الـ Service Layer

```
1. Student يحاول يدخل lesson في كورس مدفوع بدون enrollment → 403
2. Instructor يحاول يعدل كورس مش بتاعه → 403
3. Student يحاول يكتب review تاني على نفس الكورس → 400
4. أي حد يحاول يسجل كـ ADMIN → 400
5. Lesson مش is_preview في كورس المستخدم معملش فيه enroll → 403
```

---

## 📌 التعديلات المطلوبة على الـ Domain

### User Entity — أضيف `enabled` field

```java
private Boolean enabled = true; // الـ Admin يقدر يعطله
```

### Enrollment Entity — أضيف منطق التحقق

في الـ `EnrollmentService` قبل ما تدي access للـ Lessons:

```java
boolean isEnrolled = enrollmentRepository
    .existsByStudentIdAndCourseId(studentId, courseId);

if (!isEnrolled && course.getPrice() > 0) {
    throw new UnauthorizedException("Enroll in the course first");
}
```

---

## 🗺️ ملخص بصري

```
Register
   ├── role = STUDENT     → account جاهز فوراً
   ├── role = INSTRUCTOR  → account جاهز فوراً
   └── role = ADMIN       → ❌ مرفوض

Browse Courses (بدون login)
   └── يشوف العنوان والوصف والسعر بس

Login كـ STUDENT
   ├── يشوف كل الكورسات المنشورة
   ├── يدخل المجاني مباشرة
   ├── يشوف الـ Preview lessons
   └── المدفوع → يدفع → Enrollment → وصول كامل

Login كـ INSTRUCTOR
   ├── ينشئ كورسات (DRAFT)
   ├── يضيف Sections و Lessons
   ├── ينشر الكورس
   └── يشوف إحصائيات كورساته بس

Login كـ ADMIN
   └── full access على كل حاجة
```