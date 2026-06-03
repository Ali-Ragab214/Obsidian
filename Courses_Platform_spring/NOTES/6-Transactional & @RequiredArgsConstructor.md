تمام، دول من أهم الحاجات في Spring، وخليني أبسطهم لك جدًا:

---

# 1. `@RequiredArgsConstructor`

## يعني إيه؟

دي من Lombok.

بتعمل constructor تلقائي لكل الـ fields اللي `final` أو `@NonNull`.

---

## مثال بدونها:

```java
@Service
public class CourseService {

    private final CourseRepository courseRepository;
    private final CourseMapper courseMapper;

    public CourseService(
            CourseRepository courseRepository,
            CourseMapper courseMapper
    ) {
        this.courseRepository = courseRepository;
        this.courseMapper = courseMapper;
    }
}
```

---

## مع `@RequiredArgsConstructor`:

```java
@Service
@RequiredArgsConstructor
public class CourseService {

    private final CourseRepository courseRepository;
    private final CourseMapper courseMapper;
}
```

✔ نفس النتيجة  
✔ بس كود أقل  
✔ Spring بيعمل DI تلقائي

---

## الخلاصة:

> بتوفر عليك كتابة constructor

---

# 2. `@Transactional`

## يعني إيه؟

دي بتقول:

> "أي عمليات داخل الميثود دي لازم تتنفذ كـ transaction واحدة"

---

## مثال بسيط

```java
@Transactional
public void enrollStudent() {

    enrollmentRepository.save(enrollment);

    courseRepository.updateCourseStats();

    paymentRepository.save(payment);
}
```

---

## يعني إيه "Transaction واحدة"؟

يا إما:

✔ كل العمليات تنجح  
أو  
❌ كلهم يفشلوا ويرجعوا زي ما كانوا

---

## مثال مهم جدًا:

لو عندك:

```java
save(enrollment);
update(course);
sendEmail();
```

وحصل error في آخر خطوة:

```java
sendEmail() failed
```

---

## بدون @Transactional:

```text
save تم
update تم
email فشل ❌
→ البيانات اتسجلت نص نص (bug)
```

---

## مع @Transactional:

```text
كل العمليات تتلغى
database rollback
```

---

# 🧠 فين نستخدمها؟

## في Service layer غالبًا:

```java
@Service
@Transactional
public class CourseServiceImpl {
}
```

أو على method معينة:

```java
@Transactional
public CourseResponse createCourse() {
}
```

---

# ⚠️ مهم جدًا

## تستخدمها في:

✔ Create  
✔ Update  
✔ Delete  
✔ عمليات فيها أكتر من DB call

---

## متستخدمهاش في:

❌ read-only methods (زي getAllCourses)

---

# 🎯 الخلاصة

|Annotation|معناها|
|---|---|
|`@RequiredArgsConstructor`|بيعمل constructor تلقائي للـ dependencies|
|`@Transactional`|بيخلي العمليات كلها تتم كـ unit واحدة (نجاح أو فشل مع بعض)|

---

# 💡 تشبيه بسيط

- `@RequiredArgsConstructor` = بيجهز لك الأدوات (wiring)
    
- `@Transactional` = بيقولك "يا تم كل الشغل يا مفيش حاجة تحصل"
    

---

لو عايز، أقدر أشرح لك **إمتى يحصل rollback بالظبط وإزاي Spring بيعمله جوه Hibernate** ده مهم جدًا في المشاريع الكبيرة 👌

---
لما احط @Transactional  فوق ال class معني كدا ان كل الميثودز اللي في الكلاس هتبقا كدا يعني ياما كل العمليات كلها تتم يااما لا 
لكن لما بعمل كدا @Transactional )readonly = true(  دا بيبقا اي 
أيوه فهمك في الأول صحيح 👌  
لو حطّيت `@Transactional` فوق الـ **class** يبقى كل الميثودز داخله هتشتغل داخل Transaction (إلا لو ميثود معينة غيّرت الإعدادات).

---

## 🧠 طيب يعني إيه `@Transactional(readOnly = true)`؟

دي معناها إن الـ Transaction دي **مخصصة للقراءة فقط**.

يعني:

- الهدف الأساسي: **SELECT / قراءة بيانات**
    
- ومفروض **مفيش أي تعديل على الداتابيز**
    

---

## ⚙️ بيحصل إيه فعليًا لما تحطها؟

Spring (مع JPA/Hibernate) بيعمل Optimizations زي:

### 1. تحسين الأداء

- Hibernate ممكن يوقف tracking للتغييرات (dirty checking)
    
- يعني مش هيقعد يراقب الـ entities لو اتغيرت ولا لا
    

---

### 2. منع عمليات الكتابة (سلوكيًا)

- مش بيمنعك 100% تقنيًا في كل الحالات
    
- لكن بيخلي الـ persistence context “read-only”
    

---

### 3. تحسين في الـ DB layer (حسب الـ DB)

بعض قواعد البيانات ممكن تعمل optimizations زي:

- استخدام read-only connections
    
- تحسين query execution
    

---

## 🚨 مهم جدًا تفهم النقطة دي

`readOnly = true` مش "حماية أمنية"

يعني:  
لو كتبت:

```java
@Transactional(readOnly = true)
public void updateUser() {
    user.setName("Ali");
}
```

Hibernate ممكن:

- **يتجاهل التغيير**  
    أو
    
- التغيير مايتحفظش في الـ DB عند flush
    

لكن مش شرط يرمي error.

---

## 🆚 الفرق السريع

|الحالة|المعنى|
|---|---|
|`@Transactional`|Read + Write (طبيعي)|
|`@Transactional(readOnly = true)`|Read فقط + تحسين أداء|

---

## 🧠 تستخدمه إمتى؟

✔ في:

- getAll
    
- getById
    
- search queries
    
- reports
    

❌ متستخدموش في:

- create
    
- update
    
- delete
    

---

## 🔥 الخلاصة

`readOnly = true` يعني:

> “أنا داخل Transaction بس هدفي قراءة البيانات فقط، فساعدني أكون أسرع وما تتابعش تغييرات ولا تحضرني للكتابة”

---

لو حابب أشرح لك:

- الفرق بين Transaction في Spring و DB transaction فعليًا
    
- أو إزاي Hibernate بيتعامل مع flush و dirty checking تحت الغطاء
    

قول 👍