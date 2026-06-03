بما إن مشروعك **Skill Hub Learning Platform** وفيه أدوار مثل Admin و Instructor، فالأفضل تفكر في الصلاحيات من منظور "مين يدير المنصة؟" و"مين يدير محتواه؟".

## 👑 Admin

الـ Admin مسؤول عن إدارة المنصة كلها.

### صلاحيات مقترحة:

#### إدارة الكورسات

- عرض جميع الكورسات.
    
- حذف أي كورس.
    
- تعديل أي كورس.
    
- الموافقة على الكورسات قبل النشر (لو عندك workflow).
    

#### إدارة المدرسين

- عرض جميع المدرسين.
    
- تفعيل/إيقاف حساب Instructor.
    
- حذف Instructor.
    

#### إدارة الطلاب

- عرض المستخدمين.
    
- تعطيل الحسابات المخالفة.
    

#### إدارة التصنيفات

- إنشاء Category.
    
- تعديل Category.
    
- حذف Category.
    

#### الإحصائيات

- عدد الكورسات.
    
- عدد الطلاب.
    
- عدد المدرسين.
    
- عدد التسجيلات.
    

---

## 👨‍🏫 Instructor

الـ Instructor مسؤول فقط عن المحتوى الخاص به.

### صلاحيات مقترحة:

#### إدارة كورساته

- إنشاء كورس.
    
- تعديل كورساته فقط.
    
- حذف كورساته فقط.
    
- عرض كورساته.
    

#### إدارة الدروس

- إضافة Lesson.
    
- تعديل Lesson.
    
- حذف Lesson.
    

#### إدارة المحتوى

- رفع فيديوهات.
    
- رفع ملفات PDF.
    
- إضافة Quiz.
    

#### متابعة الطلاب

- رؤية الطلاب المسجلين في كورساته.
    
- رؤية التقدم الخاص بهم.
    

---

# مثال عملي على Security

نفترض عندك Roles:

```java
ROLE_ADMIN
ROLE_INSTRUCTOR
```

---

## Create Course

```java
POST /api/courses
```

مسموح لـ:

```java
.hasAnyRole("ADMIN", "INSTRUCTOR")
```

---

## Delete Course

لو الـ Admin فقط:

```java
DELETE /api/courses/{id}
```

```java
.hasRole("ADMIN")
```

---

## Get All Courses

غالبًا:

```java
GET /api/courses
```

ممكن:

```java
.permitAll()
```

أو

```java
.hasAnyRole("ADMIN", "INSTRUCTOR")
```

حسب طبيعة المنصة.

---

## Categories

```java
POST /api/categories
PUT /api/categories/{id}
DELETE /api/categories/{id}
```

غالبًا:

```java
.hasRole("ADMIN")
```

---

# نقطة مهمة جدًا

في المشاريع الحقيقية لا تعتمد على الـ Role فقط.

مثال:

```java
PUT /api/courses/5
```

الـ Instructor المفروض يعدل **كورسه هو فقط**.

يعني حتى لو عنده Role:

```java
ROLE_INSTRUCTOR
```

لازم داخل الـ Service تعمل تحقق:

```java
if (!course.getInstructor().getId().equals(currentInstructorId)) {
    throw new AccessDeniedException("You can only update your own courses");
}
```

لأن Security Config تعرف:

> "مين يقدر يدخل الـ Endpoint"

لكن الـ Service تعرف:

> "هل يملك هذا الـ Resource أم لا"

وده فرق مهم جدًا.

---

لو كنت مكانك كنت أبدأ بـ:

### ADMIN

- CRUD Categories
    
- CRUD Courses
    
- Manage Instructors
    
- Dashboard
    

### INSTRUCTOR

- Create Course
    
- Update Own Course
    
- Delete Own Course
    
- Manage Lessons
    
- View Own Students
    

وده كافي جدًا كـ MVP قوي لمنصة تعليمية.