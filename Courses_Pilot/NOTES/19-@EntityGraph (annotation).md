بص يا علي، دي من أهم الحاجات في Spring Data JPA والإنترفيوهات بتحبها جدًا، لأنك هنا بتتكلم عن **Performance Optimization**.

خلينا نفهمها من الأول.

---

# ما هي مشكلة N+1 Problem؟

تخيل إن عندك الـ Entity دي:

```java
@Entity
public class Enrollment {

    @ManyToOne(fetch = FetchType.LAZY)
    private Student student;

    @ManyToOne(fetch = FetchType.LAZY)
    private Course course;
}
```

وعندك بيانات:

```text
Enrollment
------------
1 → Java Course
2 → Spring Course
3 → Docker Course
```

---

## لو عملت:

```java
List<Enrollment> enrollments =
        enrollmentRepository.findAll();
```

هيتنفذ Query واحدة:

```sql
SELECT * FROM enrollments;
```

ولنفترض رجع 3 Enrollments.

---

## المشكلة تظهر هنا

لو عملت:

```java
for (Enrollment enrollment : enrollments) {
    System.out.println(
        enrollment.getCourse().getTitle()
    );
}
```

إيه اللي هيحصل؟

بما إن:

```java
@ManyToOne(fetch = FetchType.LAZY)
```

فالـ Course لسه متجابش.

Hibernate هتقول:

> "أجيب الـ Course دلوقتي."

فتعمل Query:

```sql
SELECT * FROM courses WHERE id = 1;
```

ثم:

```sql
SELECT * FROM courses WHERE id = 2;
```

ثم:

```sql
SELECT * FROM courses WHERE id = 3;
```

---

# عدد الـ Queries

لو عندك 3 Enrollments:

```text
1 Query → enrollments
3 Queries → courses
```

المجموع:

```text
4 Queries
```

---

لو عندك 100 Enrollment:

```text
1 Query
+
100 Queries
=
101 Queries !!
```

وده اسمه:

# N+1 Problem

> **1 query to fetch parent entities + N queries to fetch related entities.**

---

# مثال من الحياة

تخيل إنك طلبت من الموظف:

> "هاتلي كل الطلاب المسجلين."

جابلك القائمة.

وبعدين كل مرة تقوله:

> "طيب هاتلي الكورس بتاع الطالب ده."

100 مرة.

بدل ما تقوله من الأول:

> "هاتلي الطلاب والكورسات بتاعتهم مرة واحدة."

---

# طيب ما هي @EntityGraph ؟

> **@EntityGraph is a JPA feature that tells Hibernate which associations should be fetched eagerly in a specific query.**

بالعربي:

> هي Annotation بتقول لـ Hibernate:
> 
> "أنا عارف إن العلاقة Lazy، لكن في الاستعلام ده تحديدًا هات العلاقة دي معايا."

---

# في الكود بتاعك

```java
@EntityGraph(attributePaths = "course")
Page<Enrollment> findByStudentId(
        Long studentId,
        Pageable pageable);
```

معناها:

> لما تجيب الـ Enrollments، هات الـ Course معاهم.

---

بدل ما Hibernate تعمل:

```sql
SELECT * FROM enrollments
WHERE student_id = ?;
```

ثم:

```sql
SELECT * FROM courses WHERE id=?;
SELECT * FROM courses WHERE id=?;
SELECT * FROM courses WHERE id=?;
```

---

هتعمل حاجة شبه:

```sql
SELECT e.*, c.*
FROM enrollments e
LEFT JOIN courses c
    ON e.course_id = c.id
WHERE e.student_id = ?;
```

---

# النتيجة

بدل:

```text
1 + N Queries
```

يبقى:

```text
1 Query only
```

وده بيحل مشكلة الـ N+1.

---

# مثال عملي

بدون EntityGraph:

```java
Page<Enrollment> enrollments =
    repository.findByStudentId(id, pageable);

for (Enrollment e : enrollments) {
    System.out.println(
        e.getCourse().getTitle()
    );
}
```

Hibernate Logs:

```sql
select * from enrollments;

select * from course where id=1;
select * from course where id=2;
select * from course where id=3;
```

---

مع EntityGraph:

```java
@EntityGraph(attributePaths = "course")
Page<Enrollment> findByStudentId(
        Long studentId,
        Pageable pageable);
```

Hibernate Logs:

```sql
select e.*, c.*
from enrollments e
left join courses c
on e.course_id = c.id
where e.student_id=?;
```

خلصنا.

---

# لماذا لا نستخدم EAGER دائمًا؟

ممكن حد يقول:

> طب ما أخليها:

```java
@ManyToOne(fetch = FetchType.EAGER)
```

وخلاص؟

المشكلة إن EAGER معناها:

> "هات العلاقة دي في كل استعلام."

حتى لو مش محتاجها.

مثلاً:

```java
repository.count();
```

ممكن Hibernate تجيب بيانات زيادة بدون داعي.

وده بيأثر على الأداء.

---

لذلك أفضل Practice هي:

```java
LAZY by default
```

ثم:

```java
@EntityGraph
```

في الأماكن التي تحتاج فيها البيانات المرتبطة.

---

# الفرق بين EntityGraph و JOIN FETCH

## JOIN FETCH

تكتبه بنفسك:

```java
@Query("""
    SELECT e
    FROM Enrollment e
    JOIN FETCH e.course
    WHERE e.student.id = :studentId
""")
List<Enrollment> findByStudentId(Long studentId);
```

---

## EntityGraph

تخلي Spring تعمله تلقائيًا:

```java
@EntityGraph(attributePaths = "course")
Page<Enrollment> findByStudentId(
        Long studentId,
        Pageable pageable);
```

---

# أيهما أفضل؟

### EntityGraph

✅ أسهل

✅ أنظف

✅ يعمل مع Derived Queries

✅ ممتاز مع Pagination

---

### JOIN FETCH

✅ مرونة أكبر

✅ مناسب للاستعلامات المعقدة

❌ قد يسبب مشاكل مع Pagination خاصة مع Collections.

---

# في حالتك أنت

```java
@EntityGraph(attributePaths = "course")
Page<Enrollment> findByStudentId(
        Long studentId,
        Pageable pageable);
```

أنت كنت تريد:

- جلب الـ Enrollment.
    
- جلب الـ Course المرتبط.
    
- تجنب N+1.
    
- دعم Pagination.
    

ولذلك كان `@EntityGraph` اختيارًا ممتازًا.

---

# سؤال إنترفيو مهم جدًا

### What is `@EntityGraph`?

> **`@EntityGraph` is a JPA feature used to specify which lazy-loaded associations should be fetched eagerly for a particular query to optimize performance and avoid the N+1 problem.**

---

### How does `@EntityGraph` solve the N+1 problem?

> **It fetches the required associations in the initial query instead of triggering additional queries for each entity, reducing 1 + N queries to a single optimized query.**

---

# ملخص سريع للحفظ

> مشكلة **N+1** تحدث عندما يتم تنفيذ Query واحدة لجلب البيانات الأساسية، ثم Query إضافية لكل عنصر لجلب العلاقات المرتبطة.

> `@EntityGraph` تسمح لك بتحديد العلاقات التي تريد تحميلها مع الاستعلام الأساسي، وبالتالي تمنع Hibernate من تنفيذ Queries إضافية وتحسن أداء التطبيق بشكل كبير.

---
---
