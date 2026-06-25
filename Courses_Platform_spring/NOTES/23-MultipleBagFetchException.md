# 1. عنوان المشكلة: MultipleBagFetchException

الخطأ اللي ظهرلك هو:

```java
Hibernate MultipleBagFetchException
```

وده من أشهر مشاكل Hibernate.

---

## يعني اي Bag أصلاً؟

لما تعمل:

```java
@OneToMany
private List<Section> sections;
```

Hibernate بيعتبر الـ List دي:

```text
Bag
```

طالما مفيش:

```java
@OrderColumn
```

يعني Collection عادية غير مرتبة.

---

## عندك في المشروع

في Course:

```java
List<Section> sections;
```

وفي Section:

```java
List<Lesson> lessons;
```

الاتنين Bags.

---

# 2. هو كان بيحاول يعمل اي؟

الـ Agent عمل:

```java
@EntityGraph(
    attributePaths = {
        "instructor",
        "sections",
        "sections.lessons"
    }
)
```

يعني لما أجيب Course:

```text
هاتلي:
Course
Instructor
Sections
Lessons
```

كلهم مرة واحدة.

---

## Hibernate حولها تقريباً إلى

```sql
SELECT *
FROM courses c
LEFT JOIN sections s
    ON s.course_id = c.id
LEFT JOIN lessons l
    ON l.section_id = s.id
```

---

# 3. ليه Hibernate رفض؟

تخيل عندك:

```text
Course A
```

فيه:

```text
5 Sections
```

كل Section فيها:

```text
3 Lessons
```

---

النتيجة تصبح:

```text
Course A

Section 1 -> Lesson1
Section 1 -> Lesson2
Section 1 -> Lesson3

Section 2 -> Lesson1
Section 2 -> Lesson2
Section 2 -> Lesson3

Section 3 -> Lesson1
...
```

يعني:

```text
5 × 3 = 15 rows
```

---

Hibernate ساعتها يحتار:

```text
إزاي أركب
sections
و
lessons

من نفس النتيجة؟
```

لأن فيه تكرار ضخم.

فقال:

```text
أنا مش بدعم
Multiple Bags Fetch
```

ورمى:

```java
MultipleBagFetchException
```

---

# 4. كان بيحصل فين؟

في الصورة:

```java
@NamedEntityGraph(
    name = "Course.details",
    attributeNodes = {
        @NamedAttributeNode("instructor"),
        @NamedAttributeNode(
            value = "sections",
            subgraph = "sections.lessons"
        )
    },
    subgraphs = {
        @NamedSubgraph(
            name = "sections.lessons",
            attributeNodes = {
                @NamedAttributeNode("lessons")
            }
        )
    }
)
```

هنا كان بيقول:

```text
هات Sections
وهات Lessons جوة Sections
```

في نفس Query.

---

# 5. الحل اللي اتعمل

شال:

```java
sections.lessons
```

خالص.

وبقى:

```java
@NamedEntityGraph(
    name = "Course.details",
    attributeNodes = {
        @NamedAttributeNode("instructor"),
        @NamedAttributeNode("sections")
    }
)
```

يعني:

```text
Course
Instructor
Sections
```

بس.

---

# 6. طب والـ Lessons؟

هنا دخل دور:

```java
@BatchSize(size = 25)
```

---

مثال

عندك:

```text
10 Courses

كل Course فيه 5 Sections

يبقى 50 Sections
```

---

لما المابر يعمل:

```java
section.getLessons()
```

بدل:

```sql
SELECT * FROM lessons
WHERE section_id = 1

SELECT * FROM lessons
WHERE section_id = 2

SELECT * FROM lessons
WHERE section_id = 3

...
```

50 Query

---

Hibernate هيعمل:

```sql
SELECT *
FROM lessons
WHERE section_id IN
(
 1,2,3,4,5,
 ...
 25
)
```

وبعدين:

```sql
SELECT *
FROM lessons
WHERE section_id IN
(
26,27,...50
)
```

---

يعني بدل:

```text
50 Query
```

بقى:

```text
2 Query
```

---

# 7. عدد الـ Queries قبل وبعد

## قبل أي تحسين

لو عندك:

```text
10 Courses
5 Sections
3 Lessons
```

كان يحصل:

```text
1 Query Courses

10 Queries Sections

50 Queries Lessons

10 Queries Instructor
```

تقريباً:

```text
71 Query
```

وده الـ N+1.

---

## بعد الـ EntityGraph + BatchSize

أول Query:

```sql
Courses + Instructor + Sections
```

Query واحدة.

---

ثاني Query:

```sql
Lessons WHERE section_id IN (...)
```

Query واحدة.

---

الإجمالي:

```text
2 Queries فقط
```

---

# 8. ليه الحل ده ممتاز؟

لأنه جمع بين حاجتين:

## منع الـ Exception

لأنك لم تعد تعمل:

```java
sections.lessons
```

في نفس Fetch.

---

## منع الـ N+1

لأن:

```java
@BatchSize(25)
```

جاب الـ Lessons على دفعات.

---

# الخلاصة

المشكلة كانت:

```text
Hibernate لا يسمح بعمل JOIN FETCH
لـ List<Section>
و
List<Lesson>

في نفس الوقت
(MultipleBagFetchException)
```

الـ Agent كان بيعمل:

```java
sections + sections.lessons
```

في EntityGraph واحد.

فـ Hibernate رفض.

---

الحل اللي عمله:

✅ شال `sections.lessons` من الـ EntityGraph

✅ خلّى الـ EntityGraph يجيب:

```java
Course
Instructor
Sections
```

فقط.

✅ استخدم:

```java
@BatchSize(size = 25)
```

على Lessons.

فبقت الـ Lessons تتجاب بـ:

```sql
WHERE section_id IN (...)
```

بدل عشرات الاستعلامات.

---

بصراحة التعديل ده احترافي أكتر من النسخة الأولى، لأنه حل **مشكلتين مع بعض**:

1. `MultipleBagFetchException`
    
2. `N+1 Query Problem`
    

من غير ما يحمل كمية بيانات ضخمة في Query واحدة.