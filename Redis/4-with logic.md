تمام يا علي. خليني أشرحلك Redis Cache في Spring Boot من أول ما تضيف الـ dependencies لحد ما تستخدمه صح في مشروع حقيقي زي Skill Hub.

---

# أولاً: هل Redis بيحتاج تعديل في Entity أو Repository؟

الإجابة: **لا.**

Redis Cache في Spring بيشتغل غالبًا على مستوى الـ Service.

يعني:

```text
Controller
    ↓
Service  ← هنا بنستخدم Cache
    ↓
Repository
    ↓
Database
```

الـ Entity والـ Repository بيكونوا زي ما هم.

---

# 1) فعل الـ Caching

في الـ Main Class: 

```java
@SpringBootApplication
@EnableCaching
public class SkillHubLearningPlatformApplication {
}
```

بدون:

```java
@EnableCaching
```

كل الـ annotations الخاصة بالكاش مش هتشتغل.

---

# 2) قراءة البيانات باستخدام @Cacheable

مثال:

```java
@Service
@RequiredArgsConstructor
public class CourseServiceImpl implements CourseService {

    private final CourseRepository courseRepository;
    private final CourseMapper courseMapper;

    @Override
    @Cacheable("courses")
    public List<CourseResponse> getAllCourses() {

        System.out.println("Fetching from Database...");

        return courseRepository.findAll()
                .stream()
                .map(courseMapper::toResponse)
                .toList();
    }
}
```

---

## ماذا يحدث؟

### أول Request

```text
GET /api/courses
```

Spring يعمل:

```text
هل يوجد Cache باسم courses؟
        ↓
هل يوجد Key لهذه الدالة؟
        ↓
لا
        ↓
نفذ الدالة
        ↓
Database
        ↓
خزن النتيجة في Redis
        ↓
ارجعها للعميل
```

هتشوف:

```text
Fetching from Database...
```

---

### ثاني Request

خلال الـ TTL:

```text
GET /api/courses
```

هيعمل:

```text
هل يوجد Cache؟
↓
نعم
↓
ارجع النتيجة من Redis
↓
لا تنفذ الدالة
```

مش هتشوف:

```text
Fetching from Database...
```

لأن الـ DB لم تُضرب أصلاً.

---

# 3) Cache Key

لو كتبت:

```java
@Cacheable("courses")
public List<CourseResponse> getAllCourses()
```

هيتولد Key مثل:

```text
courses::SimpleKey[]
```

---

لو:

```java
@Cacheable("courses")
public CourseResponse getCourseById(Long id)
```

واستدعيت:

```java
getCourseById(5L);
```

هيبقى:

```text
courses::5
```

---

# 4) تخصيص الـ Key

مثال:

```java
@Cacheable(
    value = "courses",
    key = "#id"
)
public CourseResponse getCourseById(Long id)
```

يبقى المفتاح:

```text
courses::5
```

بدل المفاتيح المعقدة.

---

# 5) تحديث البيانات

مثلاً:

```java
PUT /courses/5
```

لو حدثت الكورس والـ Cache فضلت زي ما هي، هيبقى عندك بيانات قديمة.

الحل:

```java
@CachePut(
    value = "courses",
    key = "#id"
)
public CourseResponse updateCourse(Long id, UpdateCourseRequest request) {

    Course course = repository.findById(id)
            .orElseThrow();

    course.setTitle(request.title());

    repository.save(course);

    return mapper.toResponse(course);
}
```

---

## ماذا يفعل @CachePut ؟

```text
نفذ الدالة دائماً
↓
حدث الـ Database
↓
حدث الـ Cache
↓
ارجع النتيجة
```

---

# 6) حذف البيانات

مثلاً:

```java
DELETE /courses/5
```

بعد الحذف لازم نمسح الكاش.

```java
@CacheEvict(
    value = "courses",
    key = "#id"
)
public void deleteCourse(Long id) {

    repository.deleteById(id);
}
```

---

## ماذا يحدث؟

```text
احذف من Database
↓
احذف من Redis
```

---

# 7) حذف كل الكاش

مثلاً بعد إنشاء Course جديد:

```java
@CacheEvict(
    value = "courses",
    allEntries = true
)
public CourseResponse createCourse(CreateCourseRequest request) {

    Course course = mapper.toEntity(request);

    repository.save(course);

    return mapper.toResponse(course);
}
```

---

## لماذا allEntries؟

لأن:

```java
getAllCourses()
```

بيخزن قائمة كاملة.

بعد إضافة Course جديد:

الكاش القديمة أصبحت غير صحيحة.

فنمسحها كلها.

---

# 8) أهم Annotations

## @Cacheable

```text
لو البيانات موجودة
↓
ارجعها من Redis
↓
لا تنفذ الدالة
```

---

## @CachePut

```text
نفذ الدالة دائماً
↓
حدث Redis
```

---

## @CacheEvict

```text
امسح Redis
```

---

# 9) أين أستخدم الكاش؟

في Skill Hub مثلاً:

## مناسب جداً:

### Categories

```java
getAllCategories()
```

لأنها لا تتغير كثيرًا.

---

### Services

```java
getAllServices()

getServiceById()
```

---

### Course Details

لو مشروع تعليمي.

---

### بيانات عامة

مثل:

```java
Countries
Cities
Roles
Settings
```

---

# 10) أين لا أستخدمه؟

لا تعمل Cache لـ:

### Login

```java
login()
```

---

### JWT

---

### Payments

---

### Booking Requests الحية

إذا كانت تتغير باستمرار.

---

### Notifications الفورية

---

# 11) كيف أعرف أن Redis تعمل؟

شغل Redis.

ثم:

```bash
redis-cli
```

ثم:

```bash
KEYS *
```

مثلاً:

```text
courses::SimpleKey[]
courses::5
categories::SimpleKey[]
```

---

عرض القيمة:

```bash
GET "courses::5"
```

---

معرفة الوقت المتبقي:

```bash
TTL "courses::5"
```

مثلاً:

```text
540
```

يعني متبقي 540 ثانية.

---

# 12) ماذا يحدث فعليًا داخل Spring؟

لو عندك:

```java
@Cacheable("courses")
public CourseResponse getCourseById(Long id)
```

Spring يعمل Proxy.

بدلاً من:

```text
Controller
↓
Service
↓
Repository
```

تصبح:

```text
Controller
↓
Spring Cache Proxy
↓
Redis
↓      ↓
Hit    Miss
↓       ↓
Return  Service
        ↓
    Repository
        ↓
    Database
        ↓
      Redis
        ↓
      Return
```

---

# 13) نقطة مهمة جداً

لو كتبت:

```java
public void methodA() {
    methodB();
}

@Cacheable("courses")
public CourseResponse methodB() {
}
```

الكاش **لن تعمل**.

لأن:

```text
Self Invocation
```

يتجاوز الـ Proxy.

لازم يكون الاستدعاء من Bean أخرى أو من خارج الكلاس.

---

# خلاصة استخدام Redis Cache في مشروعك

لو كنت مكانك في Skill Hub، سأفعل التالي:

### أعمل Cache لـ:

```text
✓ Categories
✓ Services
✓ Service Details
✓ Courses
✓ Public Settings
✓ Static Reference Data
```

### لا أعمل Cache لـ:

```text
✗ Login
✗ JWT
✗ Payments
✗ Real-time Booking Status
✗ Notifications
```

### وسأستخدم:

```java
@Cacheable
```

للقراءة،

```java
@CachePut
```

للتحديث،

```java
@CacheEvict
```

للحذف أو بعد عمليات الـ Create.

لو طبقت الحاجات دي، يبقى أنت فعليًا استخدمت Redis Cache في مشروع Spring Boot بطريقة Production-style صحيحة.