سأحدد لك تصميم `Section` كاملًا داخل المشروع الحالي: الـ DTOs، الـ service methods، الـ controller endpoints، والعلاقات مع `Course`، مع الاستثناءات والحالات المتوقعة.

تمام، هديك **خطة Section كاملة ومضبوطة على مشروعك الحالي** — من غير كود، لكن بشكل عملي جدًا:  
إيه يتحط في `DTOs`، وإيه في `Service` و`ServiceImpl`، وإيه في `Controller`، وإزاي تربطه بـ `Course`، وإيه الـ exceptions اللي لازم تتحقق، وإيه الـ endpoints الصح.

---
#  IMPORTANT 
## اللي هيظهر في الـ Body

```json
{
  "title": "Introduction to Spring Boot"
}
```

بس كده — الـ `orderIndex` اختياري.

---

## هل صح تسيب الـ orderIndex اختياري؟

**أيوه ✅ وده الأصح** — وإليك السبب:

---

### الحل الأذكى — الـ Backend يحسبها تلقائياً

```java
// في الـ ServiceImpl
public SectionResponse addSection(Long courseId, SectionRequest request, String email) {
    Course course = courseRepository.findById(courseId)
            .orElseThrow(() -> new ResourceNotFoundException("Course not found"));

    // تحقق إن الـ instructor صاحب الـ course
    if (!course.getInstructor().getEmail().equals(email)) {
        throw new UnauthorizedException("Not your course");
    }

    // ✅ احسب الـ orderIndex تلقائياً
    int nextOrderIndex = sectionRepository.countByCourseId(courseId);

    Section section = Section.builder()
            .title(request.getTitle())
            .orderIndex(request.getOrderIndex() != null 
                    ? request.getOrderIndex()  // لو بعتها استخدمها
                    : nextOrderIndex)           // لو مبعتهاش احسبها
            .course(course)
            .build();

    return sectionMapper.toResponse(sectionRepository.save(section));
}
```

```java
// في الـ Repository
int countByCourseId(Long courseId);
```

---

### النتيجة

```
Section 1 → orderIndex = 0  (تلقائي)
Section 2 → orderIndex = 1  (تلقائي)
Section 3 → orderIndex = 2  (تلقائي)
```

الـ Instructor مش محتاج يفكر فيها خالص. 🎯
---

## ✅ Checklist التنفيذ
- [ ] تثبيت شكل `Section` داخل الـ domain
- [ ] تحديد الـ DTOs المطلوبة
- [ ] تحديد مسؤوليات `SectionService`
- [ ] تحديد مسؤوليات `SectionServiceImpl`
- [ ] تحديد endpoints في `SectionController`
- [ ] تحديد قواعد الربط مع `Course`
- [ ] تحديد الـ exceptions والحالات الخاصة
- [ ] ترتيب الأولويات لو هتبدأ تكتب

---

# 1) شكل `Section` في المشروع عندك

من الملف `Section.java` واضح إن الـ section عندك فيه:

- `title`
- `orderIndex`
- `course`
- `lessons`

وده معناه إن الـ `Section` عندك **تابعة لكورس**، والـ course هو الـ parent.

### يعني مهم جدًا:
أي عملية على section لازم تكون مرتبطة بـ:
- `courseId`
- أو `sectionId` مع فحص إنه تابع للكورس المطلوب

---

# 2) الـ DTOs اللي لازم تعملها لمرحلة Section

## A) `SectionRequest`
ده الـ request الأساسي لإضافة/تعديل section.

### يحتوي على:
- `title`
- `orderIndex`

### وممكن كمان يكون فيه:
- `courseId`  
لكن الأفضل معماريًا:
- لو الـ endpoint خاص بكورس معين، خليه في الـ path
- وماتحطش `courseId` في الـ body إلا لو محتاجه صراحة

### الشكل الأفضل:
- `POST /courses/{courseId}/sections`
- body فيه:
  - `title`
  - `orderIndex`

---

## B) `SectionResponse`
ده اللي هيرجع للعميل بعد الإنشاء أو الجلب.

### يحتوي على:
- `id`
- `title`
- `orderIndex`
- `courseId`
- `courseTitle` لو حبيت
- `createdAt`
- `updatedAt`

### لو عايز response أنضف:
يرجع أيضًا:
- عدد الـ lessons
- أو list مختصر للـ lessons لاحقًا

لكن في المرحلة دي الأفضل تبدأ بسيط:
- section data فقط
- بدون lessons details

---

## C) Optional DTOs حسب احتياجك
ممكن تحتاج لاحقًا:
- `SectionUpdateRequest` لو عايز تفصل create عن update
- `SectionOrderUpdateRequest` لو هتعمل reorder مستقل

لكن كبداية:
- `SectionRequest`
- `SectionResponse`
كفاية جدًا.

---

# 3) `SectionService` المفروض يحتوي على إيه؟

هنا بنحدد الـ contract فقط.

## أهم الميثودز المقترحة:

### 1. `createSection`
- ينشئ section جديدة داخل course معين

### 2. `updateSection`
- يعدل title أو orderIndex أو الاثنين

### 3. `getSectionById`
- يرجع section واحدة

### 4. `getSectionsByCourseId`
- يرجع كل sections الخاصة بكورس معين مرتبة

### 5. `deleteSection`
- يحذف section محددة

### 6. `reorderSections`
- لو هتسمح بتغيير ترتيب sections

---

# 4) `SectionServiceImpl` هتعمل فيه إيه؟

دي أهم جزء عملي.

---

## أولًا: `createSection`

### اللي يحصل:
1. تجيب `Course` بالـ `courseId`
2. لو مش موجود → `ResourceNotFoundException`
3. تتأكد إن الـ user المسموح له هو:
   - instructor صاحب الكورس
   - أو admin
4. تنشئ `Section`
5. تربطها بالكورس
6. تحفظها
7. ترجع `SectionResponse`

### Validation هنا:
- title مش فاضي
- title length مناسب
- orderIndex غير سالب
- course موجود
- course status يسمح بإضافة section؟  
  - لو أنت عايز rules stricter:
    - ممكن تسمح فقط لو `DRAFT`
    - أو تسمح في `PUBLISHED` لو ده مناسب

### مهم جدًا:
لو الكورس `ARCHIVED`:
- غالبًا تمنع أي تعديل
- وترجع `BadRequestException`

---

## ثانيًا: `updateSection`

### اللي يحصل:
1. تجيب section بالـ `sectionId`
2. لو مش موجود → `ResourceNotFoundException`
3. تجيب الـ course الخاص بيها
4. تتأكد من صلاحية المستخدم
5. تعدل الحقول المطلوبة فقط
6. تحفظ
7. ترجع response

### القواعد:
- لو title موجود في request → حدّثه
- لو orderIndex موجود → حدّثه
- ما تمسحش course relation
- ما تغيرش lessons هنا

---

## ثالثًا: `getSectionById`

### اللي يحصل:
1. تجيب section
2. لو مش موجود → `ResourceNotFoundException`
3. ترجع response

### ممكن تضيف rule:
- لازم section تبقى تابعة لنفس courseId لو endpoint nested:
  - `/courses/{courseId}/sections/{sectionId}`

ده أفضل عشان يمنع user يجيب section غلط من course غلط.

---

## رابعًا: `getSectionsByCourseId`

### اللي يحصل:
1. تتأكد إن course موجود
2. تجيب sections المرتبة بـ `orderIndex`
3. ترجع list

### هنا `SectionRepository` عندك أصلًا فيه:
- `findByCourseIdOrderByOrderIndexAsc`

وده ممتاز جدًا.  
يعني الريبو جاهز تقريبًا للجزء ده.

---

## خامسًا: `deleteSection`

### اللي يحصل:
1. تجيب section
2. تتأكد من صلاحية المستخدم
3. لو section فيها lessons:
   - بما إن `orphanRemoval = true` في `Section.lessons`
   - الحذف هيشيل lessons تلقائيًا
4. تحذف section
5. ترجع success response

### مهم:
لو عندك business rule تمنع حذف section فيها lessons:
- ساعتها تعمل check قبل الحذف وترمي `BadRequestException`
لكن من تصميمك الحالي:
- الحذف التلقائي منطقي

---

## سادسًا: `reorderSections`
دي optional، لكن أنصحك تجهز لها لو المشروع كبير.

### اللي تعملها:
- تستقبل ترتيب جديد
- تحدث `orderIndex` لكل section
- تتحقق إن كل section تخص نفس course
- تحفظ batch

### لو مش هتعملها الآن:
مفيش مشكلة، بس خلي `orderIndex` موجود من البداية لأنه هيفيدك جدًا.

---

# 5) `SectionController` لازم فيه إيه؟

هنا الـ API layer.

## أفضل REST design لمرحلة Section:

---

### 1. إنشاء Section داخل Course
`POST /api/v1/courses/{courseId}/sections`

#### body:
- `title`
- `orderIndex`

#### response:
- `SectionResponse`

---

### 2. جلب كل Sections لكورس
`GET /api/v1/courses/{courseId}/sections`

#### response:
- list of `SectionResponse`

---

### 3. جلب Section واحدة
`GET /api/v1/courses/{courseId}/sections/{sectionId}`

#### response:
- `SectionResponse`

---

### 4. تحديث Section
`PUT /api/v1/courses/{courseId}/sections/{sectionId}`

#### body:
- `title`
- `orderIndex`

#### response:
- `SectionResponse`

---

### 5. حذف Section
`DELETE /api/v1/courses/{courseId}/sections/{sectionId}`

#### response:
- success message فقط

---

### 6. ترتيب Sections
لو عملتها:
`PATCH /api/v1/courses/{courseId}/sections/reorder`

#### body:
- list of section ids with new order
أو list of sections with orderIndex

---

# 6) العلاقات مع `Course` لازم تراعيها إزاي؟

دي أهم نقطة عندك.

## Section تبقى تابعة لـ Course
يعني:
- ماينفعش تعمل section من غير course
- ماينفعش section تتنقل بين courses إلا لو أنت عامل use case خاص

## في الخدمة لازم تعمل:
- `courseRepository.findById(courseId)`
- لو مش موجود → `ResourceNotFoundException`
- then set course in section

## لو جبت section by id:
لازم تتأكد:
- `section.getCourse().getId().equals(courseId)`
عشان تمنع references غلط.

---

# 7) الصلاحيات والـ Access Control

دي مهمة جدًا في مشروعك لأن عندك roles.

## مين يقدر يعمل Section؟

### Instructor:
- ينشئ section داخل كورسه
- يعدل section في كورسه
- يحذف section في كورسه

### Admin:
- يقدر يعمل كل حاجة

### Student:
- يقرأ فقط

---

## القاعدة العملية:
### على مستوى `SecurityConfig`
ممكن تسيب endpoints اللى فيها POST/PUT/DELETE على sections:
- `hasAnyRole("INSTRUCTOR", "ADMIN")`

لكن ده **مش كفاية لوحده**.

### لازم داخل service:
- تتأكد إن instructor هو صاحب course فعلًا
- أو admin

لأن security config مش هيعرف ownership.

---

# 8) الـ Exceptions اللي لازم تتعامل معاها

## A) `ResourceNotFoundException`
تستخدمها لو:
- course مش موجود
- section مش موجود

### أمثلة:
- courseId غلط
- sectionId غلط

---

## B) `BadRequestException`
تستخدمها لو:
- title فاضي
- title قصير جدًا
- orderIndex سالب
- course status لا يسمح بالتعديل
- section لا تتبع course المطلوب
- محاولة reorder غير صحيحة

---

## C) `UnauthorizedException`
أنت ذكرتها في الـ architecture، لكن الملف نفسه مش ظاهر عندي في `exceptions` list اللي فتحتها.  
لو موجود عندك في المشروع استخدمه هنا لحالات:
- المستخدم ليس owner للكورس
- أو ليس admin

### لو مش موجود:
- استخدم `BadRequestException` مؤقتًا، لكن الأفضل تضيفه لاحقًا

---

## D) `MethodArgumentNotValidException`
ده هيجيلك تلقائي من validation annotations على الـ DTOs.

### مثال:
- title فاضي
- title too short
- orderIndex أقل من 0

والـ `GlobalExceptionHandler` عندك أصلًا بيمسكه.

---

## E) `DataIntegrityViolationException`
مفيد لو:
- حصل duplicate title فعليًا
- أو constraint violation

لكن في `Section` أنت غالبًا مش هتحتاج unique title على مستوى database إلا لو أنت حابب تضيفه.

---

# 9) Validation Rules المقترحة للـ Section

## في `SectionRequest`
### title:
- required
- 2 إلى 100

### orderIndex:
- optional أو required حسب تصميمك
- لو required:
  - minimum 0

---

# 10) أفضل Flow للمستخدم عند إنشاء Section

## السيناريو الصح:
1. instructor يعمل course
2. course يكون `DRAFT`
3. instructor يضيف sections
4. instructor يضيف lessons داخل sections
5. بعد كده ينشر course

ده أفضل flow لأن:
- sections والlessons يبقوا متشافين قبل النشر
- course يطلع منظم

---

# 11) أنسب تصميم عملي جدًا لك في المشروع

## الملفات اللي هتعملها لمرحلة Section:

### في `application/dto/request`
- `SectionRequest`

### في `application/dto/response`
- `SectionResponse`

### في `application/mapper`
- `SectionMapper`

### في `application/services/content`
- `ContentService` أو `SectionService`  
  حسب naming اللي ناوي تمشي عليه

### في `application/services/content/impl`
- `ContentServiceImpl` أو `SectionServiceImpl`

### في `api/controller`
- `SectionController`  
  أو لو هتمشي naming موحد:
- `CourseContentController`

---

# 12) هل الأفضل `SectionService` ولا `ContentService`؟

بصراحة، بما إن structure عندك فيه:

- `content` service layer

فممكن تمشي بإحدى طريقتين:

## Option A — الأنظف:
- `ContentService`
- `ContentServiceImpl`

ويكون فيه methods تخص:
- section
- lesson

لكن ده هيتوسع شوية.

## Option B — الأبسط الآن:
- `SectionService`
- `SectionServiceImpl`

وبعدين `LessonService`
وده أوضح لو أنت لسه في البداية.

### أنصحك بـ:
**`SectionService` منفصل الآن**
عشان تدخل بسرعة وتبني صح.

---

# 13) الـ endpoints اللي أنصحك بها النهاردة بالضبط

## Create section
`POST /api/v1/courses/{courseId}/sections`

## Get all sections in course
`GET /api/v1/courses/{courseId}/sections`

## Get section by id
`GET /api/v1/courses/{courseId}/sections/{sectionId}`

## Update section
`PUT /api/v1/courses/{courseId}/sections/{sectionId}`

## Delete section
`DELETE /api/v1/courses/{courseId}/sections/{sectionId}`

## Optional reorder
`PATCH /api/v1/courses/{courseId}/sections/reorder`

---

# 14) ترتيب الشغل لو هتبدأ تنفذ

## اعملهم بالترتيب ده:
1. `SectionRequest`
2. `SectionResponse`
3. `SectionMapper`
4. `SectionService`
5. `SectionServiceImpl`
6. `SectionController`
7. مراجعة exceptions
8. اختبار authorization مع course ownership

---

# 15) ملحوظة مهمة جدًا على مشروعك الحالي

من اللي شفته:
- `Section` فيه `orderIndex`
- `SectionRepository` فيه query مرتبة

وده ممتاز جدًا.

لكن لازم تاخد بالك من حاجة:
### `orderIndex` لازم يبقى unique **داخل نفس course** لو عايز ترتيب نظيف
مش شرط على مستوى database، لكن على مستوى business logic.

يعني:
- course واحد: sections 0,1,2,3
- course تاني: يبدأ برضه من 0

وده طبيعي.

---

# 16) لو عايز recommendation نهائي مختصر جدًا

## `SectionService`:
- `createSection(courseId, request)`
- `updateSection(courseId, sectionId, request)`
- `getSectionById(courseId, sectionId)`
- `getSectionsByCourseId(courseId)`
- `deleteSection(courseId, sectionId)`
- optional `reorderSections(courseId, request)`

## `SectionController`:
- `POST /courses/{courseId}/sections`
- `GET /courses/{courseId}/sections`
- `GET /courses/{courseId}/sections/{sectionId}`
- `PUT /courses/{courseId}/sections/{sectionId}`
- `DELETE /courses/{courseId}/sections/{sectionId}`

## Exceptions:
- `ResourceNotFoundException`
- `BadRequestException`
- `UnauthorizedException` لو موجود/هتضيفه
- validation errors
- maybe `DataIntegrityViolationException`

---

لو تحب، الخطوة الجاية أقدر أعملهالك بنفس الأسلوب ده لكن على مستوى أدق جدًا:

### أكتب لك:
**“Section module blueprint”**
وفيه:
- اسم كل ملف
- محتوى كل ملف وظيفيًا
- كل method اسمها إيه
- كل endpoint مع access rules
- وكل exception فين تتستخدم

من غير كود برضه، لكن كأنه checklist تنفّذها مباشرة.

feat: implement sections & lessons module with pagination support                                                                                                                           
     - Add SectionRequest/Response, LessonRequest/Response DTOs                                          :                                                                            LSPs are disabled                       
     - Create SectionMapper and LessonMapper for entity-DTO mapping                                                                                                                                                           
     - Introduce PaginationResponse for paginated API results                                                                                                                         ▶ Modified Files                        
     - Add Lesson.orderIndex field and update Section relationships                                                                                                                                                           
     - Fix CourseMapper to properly inject UserMapper via constructor"                                                                                                                                                        