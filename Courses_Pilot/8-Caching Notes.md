.

---

# أول حاجة: ليه عملنا CacheConstants أصلاً؟

```java
public final class CacheConstants {

    public static final String COURSE = "courses";
    public static final String ALL = "courses:all";
    public static final String PUBLISHED = "courses:published";
}
```

بدل ما تكتب:

```java
@Cacheable(cacheNames = "courses")
```

في 20 مكان.

لو بكرة غيرت الاسم:

```java
"courses"
```

إلى

```java
"course-cache"
```

هتعدل هنا فقط.

---

# تاني حاجة: الكاشات الموجودة

عندك 3 Cache Regions

## 1

```java
COURSE = "courses"
```

دي للكورس الواحد.

مثال:

```java
getCourseById(5)
```

هيتخزن:

```text
courses
 └── 5 => CourseResponse
```

---

## 2

```java
ALL = "courses:all"
```

دي للـ

```java
getAllCourses()
```

مثال:

```text
courses:all
 └── SimpleKey.EMPTY => List<CourseResponse>
```

---

## 3

```java
PUBLISHED = "courses:published"
```

دي للـ

```java
getAllPublishedCourses()
```

مثال:

```text
courses:published
 └── SimpleKey.EMPTY => List<CourseResponse>
```

---

# نفهم getCourseById

```java
@Cacheable(
    cacheNames = CacheConstants.COURSE,
    key = "#id",
    unless = "#result.status.name() != 'PUBLISHED'"
)
```

---

أول مرة:

```java
GET /courses/5
```

يدخل DB

---

يرجع:

```java
CourseResponse
```

---

لو الكورس:

```java
PUBLISHED
```

يتخزن.

---

في Redis:

```text
courses
 └── 5 => CourseResponse
```

---

المرة اللي بعدها:

```java
GET /courses/5
```

مش هيروح DB أصلاً.

---

# ليه unless؟

لو الكورس Draft

```java
DRAFT
```

صاحبه فقط يشوفه.

لو خزناه في Cache

هيبقى Shared.

---

مثال:

Ahmed فتح الكورس

الكورس Draft

اتخزن.

---

بعدها Mohamed طلب نفس ID

Spring هيجيب من Cache مباشرة.

وهيعدي Security Check.

مصيبة.

---

عشان كده:

```java
unless = "#result.status.name() != 'PUBLISHED'"
```

يعني:

> لو مش Published متخزنش.

ودي حركة ممتازة.

---

# getAllCourses

```java
@Cacheable(cacheNames = CacheConstants.ALL)
```

---

أول Request:

```java
GET /courses
```

DB

---

يتخزن:

```text
courses:all
```

---

أي Request بعد كده:

```java
GET /courses
```

Cache فقط.

---

# getAllPublishedCourses

نفس الكلام.

---

# createCourse

```java
@CacheEvict(
    cacheNames = {
        CacheConstants.ALL,
        CacheConstants.PUBLISHED
    },
    allEntries = true
)
```

---

قبل الإنشاء:

```text
courses:all
```

فيه 20 Course

---

أنشأت Course جديد.

---

لو سيبت الكاش:

```java
getAllCourses()
```

هيرجع 20

مش 21.

---

عشان كده:

```java
allEntries = true
```

تمسح القائمة كلها.

---

# ليه ممسحش COURSE؟

سؤال مهم.

---

لأنك بتنشئ Course جديد.

مفيش Cache Entry للكورس ده أصلاً.

فمفيش حاجة تتمسح.

---

# updateCourse

```java
@Caching(
   evict = {
      @CacheEvict(cacheNames="courses", key="#id"),
      @CacheEvict(cacheNames={"courses:all","courses:published"}, allEntries=true)
   }
)
```

---

نفترض:

```java
Course 5
```

متخزن.

---

و

```java
getAllCourses()
```

متخزن.

---

عدلت عنوان الكورس.

---

لازم تمسح:

### الكورس نفسه

```text
courses
 └── 5
```

---

### القوائم

```text
courses:all
courses:published
```

---

عشان تتبني من جديد.

---

وده بالظبط اللي بيحصل.

---

# publishCourse

دي أكتر واحدة ذكية.

```java
@CachePut(
    cacheNames = CacheConstants.COURSE,
    key = "#id"
)
```

---

نفترض:

```java
Course 5
```

كان Draft.

---

عملت:

```java
publishCourse(5)
```

---

بدل:

```java
CacheEvict
```

عمل:

```java
CachePut
```

يعني:

> حدث الكاش بالقيمة الجديدة مباشرة.

---

في Redis:

قبل:

```text
courses
 └── 5 => Draft
```

بعد:

```text
courses
 └── 5 => Published
```

---

من غير Request إضافي.

---

وده استخدام صحيح جداً لـ CachePut.

---

# deleteCourse

```java
@CacheEvict(cacheNames = COURSE, key="#id")
```

يمسح:

```text
courses
 └── 5
```

---

وكمان يمسح:

```text
courses:all
courses:published
```

---

وده صح.

---

# ليه LessonService و SectionService بيعملوا Evict؟

دي نقطة ناس كتير بتغلط فيها.

---

نفترض:

```java
GET /course/5
```

رجع:

```json
{
  "title":"Spring",
  "sections":[...]
}
```

واتخزن.

---

بعدها عملت:

```java
createLesson(...)
```

---

الكورس نفسه اتغير.

لأن:

```text
sections
lessons
duration
count
```

اتغيروا.

---

فلازم:

```java
@CacheEvict(
    cacheNames = CacheConstants.COURSE,
    key = "#courseId"
)
```

---

عشان أول Request بعد كده يجيب نسخة جديدة.

---

وده صح 100%.

---

# هل في حاجة غلط؟

في رأيي:

❌ لا.

---

# هل في حاجة ممكن تتشال؟

أيوة.

الدوال دي:

```java
private boolean isAdmin(...)
```

و

```java
private boolean isOwner(...)
```

مش مستخدمين.

امسحهم.

---

كمان:

```java
@CachePut
```

في

```java
publishCourse
```

ممكن تخليه:

```java
@CacheEvict(cacheNames = COURSE, key = "#id")
```

برضه هيشتغل.

---

لكن:

```java
@CachePut
```

أسرع وأحسن شوية.

فسيبه.

---

# التقييم النهائي

### Design

9/10

### Simplicity

9.5/10

### Over Engineering

1/10

### مناسبة لمشروع Skill Hub

10/10

### لو أنا Senior في Review

كنت هوافق على الـ PR بالشكل الحالي مع ملاحظة واحدة:

```java
remove unused methods:
isAdmin()
isOwner()
```

غير كده الكاش عندك دلوقتي نظيف، بسيط، وسهل تشرحه في الانترفيو بدون وجع دماغ.