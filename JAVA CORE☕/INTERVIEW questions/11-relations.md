# علاقات الـ JPA/Hibernate — الدليل الشامل

---

## أولاً: أنواع العلاقات

---

### 1. One-to-One

يعني كل record في جدول مرتبط بـ record واحد بالظبط في جدول تاني.

**مثال حياتي:** User ↔ Profile

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private Profile profile;
}

@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String bio;
    private String avatarUrl;

    @OneToOne(mappedBy = "profile")
    private User user;
}
```

**في الـ Database:**

```
users table:  id | email | profile_id (FK)
profiles table: id | bio | avatar_url
```

---

### 2. One-to-Many / Many-to-One

الأكتر استخداماً. Entity واحدة مرتبطة بـ entities كتير.

**مثال حياتي:** Instructor → Courses (instructor عنده كورسات كتير)

```java
@Entity
public class Instructor {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "instructor", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Course> courses = new ArrayList<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "instructor_id")
    private Instructor instructor;
}
```

**في الـ Database:**

```
instructors table: id | name
courses table: id | title | instructor_id (FK)
```

> الـ FK دايماً بتبقى على جانب الـ Many — يعني في جدول الـ Course.

---

### 3. Many-to-Many

كل entity ممكن تكون مرتبطة بأكتر من entity في الجانب التاني.

**مثال حياتي:** Students ↔ Courses (student ممكن يتسجل في كورسات كتير، والكورس فيه students كتير)

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses = new ArrayList<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students = new ArrayList<>();
}
```

**في الـ Database:**

```
students table: id | name
courses table: id | title
student_courses table: student_id (FK) | course_id (FK)
```

---

## ثانياً: المفاهيم المهمة

---

### `mappedBy`

بتقول لـ Hibernate "مش أنا اللي شايل الـ FK، الطرف التاني هو المسؤول."

```java
// Course هو الـ owner لأنه شايل الـ FK (instructor_id)
// Instructor بيقول: "التعريف موجود عند instructor في الـ Course"
@OneToMany(mappedBy = "instructor")
private List<Course> courses;
```

> **القاعدة:** الـ `mappedBy` دايماً على الـ side اللي **مش** شايل الـ FK.

---

### `@JoinColumn`

بتحدد اسم الـ FK column في الـ database.

```java
@ManyToOne
@JoinColumn(name = "instructor_id") // اسم الـ column في جدول courses
private Instructor instructor;
```

---

### `CascadeType`

بتحدد لما تعمل عملية على entity، تتعمل تلقائياً على الـ related entities.

```java
@OneToMany(cascade = CascadeType.ALL)
// لما تحذف Instructor، هيتحذف كل الـ courses بتاعته تلقائياً
```

|النوع|المعنى|
|---|---|
|`ALL`|كل العمليات|
|`PERSIST`|لما تعمل save|
|`MERGE`|لما تعمل update|
|`REMOVE`|لما تعمل delete|
|`REFRESH`|لما تعمل refresh|

> **تحذير:** متستخدمش `CASCADE.ALL` مع `@ManyToMany` — هتتحذف data مش المفروض.

---

### `FetchType`

بيحدد امتى Hibernate يجيب الـ related data من الـ database.

```java
// LAZY - بيجيب الـ courses بس لما تطلبهم فعلاً
@OneToMany(fetch = FetchType.LAZY)
private List<Course> courses;

// EAGER - بيجيب الـ instructor مع الـ course في نفس الـ query
@ManyToOne(fetch = FetchType.EAGER)
private Instructor instructor;
```

||LAZY|EAGER|
|---|---|---|
|متى يجيب الداتا|لما تطلبها|فوراً مع الـ parent|
|الـ Default في|`@OneToMany`, `@ManyToMany`|`@ManyToOne`, `@OneToOne`|
|الأداء|أحسن في معظم الحالات|ممكن يعمل مشاكل|

> **Best Practice:** استخدم `LAZY` دايماً إلا لو عندك سبب قوي لـ `EAGER`.

---

## ثالثاً: مشكلة N+1 وازاي تتجنبها

دي أشهر مشكلة في الـ JPA.

```java
// ❌ غلط - هيعمل query لكل instructor على حدة
List<Instructor> instructors = instructorRepository.findAll();
instructors.forEach(i -> System.out.println(i.getCourses().size()));
// لو عندك 10 instructors = 11 queries !

// ✅ صح - جيبهم كلهم في query واحدة
@Query("SELECT i FROM Instructor i JOIN FETCH i.courses")
List<Instructor> findAllWithCourses();
```

---

## رابعاً: Many-to-Many مع Extra Columns

لو محتاج تحط data زيادة في الـ join table (زي تاريخ التسجيل):

```java
// مش هتستخدم @ManyToMany، هتعمل entity وسطية
@Entity
public class Enrollment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne
    @JoinColumn(name = "course_id")
    private Course course;

    private LocalDateTime enrolledAt; // extra data
    private String grade;             // extra data
}
```

> **القاعدة:** لو الـ join table فيها columns زيادة غير الـ FKs، اعملها entity منفصلة.

---

## ملخص سريع

```
One-to-One   → @JoinColumn على أي جانب، mappedBy على التاني
One-to-Many  → FK في جدول الـ Many، mappedBy على الـ One
Many-to-One  → @JoinColumn هنا دايماً
Many-to-Many → @JoinTable + جدول وسطي تلقائي
```