ممتاز، لأن لو فهمت `@BatchSize` هتفهم نص مشاكل الـ N+1 في Hibernate.

---

# الأول: المشكلة بدون BatchSize

عندك:

```java
Course
 └── List<Section> sections

Section
 └── List<Lesson> lessons
```

وكلهم:

```java
fetch = FetchType.LAZY
```

يعني Hibernate مش هيجيب الـ Lessons إلا لما تطلبها.

---

نفترض عندك:

```text
Course 1
  ├─ Section 1
  ├─ Section 2
  ├─ Section 3

Course 2
  ├─ Section 4
  ├─ Section 5
```

---

لما تعمل:

```java
List<Course> courses = courseRepository.findAll();
```

هيجيب:

```sql
SELECT * FROM courses
```

بس.

---

بعدها المابر يعمل:

```java
course.getSections()
```

تمام.

وبعدين:

```java
section.getLessons()
```

---

Hibernate هيقول:

```text
أوكي
عايز Lessons بتاعة Section 1
```

فيعمل:

```sql
SELECT * FROM lessons
WHERE section_id = 1
```

---

بعدها:

```text
عايز Lessons بتاعة Section 2
```

```sql
SELECT * FROM lessons
WHERE section_id = 2
```

---

بعدها:

```sql
SELECT * FROM lessons
WHERE section_id = 3
```

---

وهكذا...

---

لو عندك:

```text
50 Sections
```

هيعمل:

```text
50 Query
```

عشان الـ Lessons فقط.

وده الـ N+1.

---

# هنا BatchSize بيعمل اي؟

لما تكتب:

```java
@OneToMany(...)
@BatchSize(size = 25)
private List<Lesson> lessons;
```

أنت بتقول لـ Hibernate:

> لو احتجت Lazy Collections كتير من نفس النوع، متجبهمش واحدة واحدة.

---
# الخلاصه هنا 
بدل:

```sql
SELECT * FROM lessons
WHERE section_id = 1

SELECT * FROM lessons
WHERE section_id = 2

SELECT * FROM lessons
WHERE section_id = 3
```

هيجمعهم.

---

فيعمل:

```sql
SELECT *
FROM lessons
WHERE section_id IN
(
1,2,3,4,5,
6,7,8,9,10,
...
25
)
```

مرة واحدة.

---

لو عندك 50 Section

وحاطط:

```java
@BatchSize(25)
```

هيبقى:

```sql
Query 1 -> sections 1..25

Query 2 -> sections 26..50
```

بس.

---

# ليه 25 بالذات؟

مش رقم سحري.

الـ Agent اختاره كحل متوسط.

ممكن:

```java
@BatchSize(10)
```

أو

```java
@BatchSize(50)
```

أو

```java
@BatchSize(100)
```

---

غالباً الناس بتحط:

```java
10
16
25
50
```

حسب حجم البيانات.

---

# طب ليه استخدمناه عندك؟

بسبب المشكلة دي:

```java
@EntityGraph(
    attributePaths = {
        "sections",
        "sections.lessons"
    }
)
```

دي عملت:

```text
MultipleBagFetchException
```

---

فبقينا قدام اختيارين:

### اختيار 1

نجيب:

```java
sections
```

فقط.

ونسيب:

```java
lessons
```

LAZY

---

### اختيار 2

لما نحتاج الـ lessons

نخلي Hibernate يجيبهم:

```java
@BatchSize(25)
```

على دفعات.

---

فأصبح:

```text
Query 1
Courses + Sections

Query 2
Lessons Batch
```

بدل:

```text
70 Query
```

---

# تخيلها بالمصري

بدون BatchSize:

```text
روح هات دفتر أحمد

روح هات دفتر علي

روح هات دفتر محمد

روح هات دفتر كريم
```

50 مشوار.

---

مع BatchSize:

```text
هاتلي دفاتر
أحمد
وعلي
ومحمد
وكريم
و...
25 واحد مرة واحدة
```

مشوار واحد.

---

# إمتى أستخدم BatchSize؟

غالبًا مع:

```java
@OneToMany
@ManyToMany
```

اللي:

```java
LAZY
```

ومش عايز تعمل:

```java
JOIN FETCH
```

عليها.

---

في مشروع SkillHub بتاعك تحديدًا:

```java
Course
  -> Sections

Section
  -> Lessons
```

استخدام:

```java
@BatchSize(size = 25)
```

على `lessons`

قرار ممتاز لأنه:

✅ حل N+1

✅ منع MultipleBagFetchException

✅ خفف حجم الـ JOIN الضخم

✅ مناسب جدًا لحجم مشروعك الحالي.