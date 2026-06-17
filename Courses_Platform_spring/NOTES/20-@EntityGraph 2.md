أيوة بالضبط يا علي 👌، ودي نقطة مهمة جدًا لازم تفهمها.

لما تكتب:

```java
@EntityGraph(attributePaths = "course")
```

فـ `"course"` هنا لازم تكون:

> **اسم الـ Field الموجود داخل الـ Entity الأساسية التي ترجعها الـ Repository.**

---

## في حالتك

عندك Repository:

```java
public interface EnrollmentRepository extends JpaRepository<Enrollment, Long>
```

يعني الـ Entity الأساسية هي:

```java
Enrollment
```

ولنفترض إن الـ Entity شكلها كده:

```java
@Entity
public class Enrollment {

    @Id
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Student student;

    @ManyToOne(fetch = FetchType.LAZY)
    private Course course;
}
```

فهنا الـ Fields الموجودة هي:

```text
id
student
course
```

لذلك تقدر تكتب:

```java
@EntityGraph(attributePaths = "course")
```

أو:

```java
@EntityGraph(attributePaths = "student")
```

أو حتى:

```java
@EntityGraph(attributePaths = {"student", "course"})
```

---

## مثال

لو كتبت:

```java
@EntityGraph(attributePaths = {"student", "course"})
List<Enrollment> findAll();
```

Hibernate هتجيب:

```text
Enrollment
    +
Student
    +
Course
```

في نفس الاستعلام.

---

## طيب لو كتبت حاجة مش موجودة؟

مثلاً:

```java
@EntityGraph(attributePaths = "teacher")
```

بينما `Enrollment` لا تحتوي على:

```java
private Teacher teacher;
```

هيحصل Exception وقت تشغيل التطبيق، زي:

```text
Unable to locate Attribute with the given name [teacher]
```

لأن JPA هتقول:

> "أنا مش لاقية Field اسمها teacher جوه Enrollment."

---

# طيب لو العلاقة متداخلة؟

نفترض:

```java
@Entity
public class Enrollment {

    @ManyToOne(fetch = FetchType.LAZY)
    private Course course;
}
```

و:

```java
@Entity
public class Course {

    @ManyToOne(fetch = FetchType.LAZY)
    private Instructor instructor;
}
```

تقدر تعمل:

```java
@EntityGraph(attributePaths = {"course.instructor"})
```

يعني:

> هات الـ Enrollment، ومعاه الـ Course، ومع الـ Course هات الـ Instructor.

Hibernate هتعمل Join مناسب لكل العلاقات دي.

---

## سؤال إنترفيو ممكن يجي

### Q: What does `attributePaths` refer to in `@EntityGraph`?

> **It refers to the names of the association fields in the root entity returned by the repository method.**

بالعربي:

> هي أسماء الـ Associations الموجودة داخل الـ Entity الأساسية التي يعيدها الـ Repository.

---

## ملخص سريع

لو عندك:

```java
JpaRepository<Enrollment, Long>
```

فإن:

```java
@EntityGraph(attributePaths = "course")
```

تعني:

> "لما تجيب Enrollment، هات الـ course المرتبط بيه أيضًا."

والـ `"course"` هنا ليس اسم جدول في قاعدة البيانات، وليس اسم Entity عشوائي، وإنما هو **اسم الـ Field داخل كلاس Enrollment نفسه**.