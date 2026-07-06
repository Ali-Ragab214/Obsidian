

---

## أولاً — الـ Derived Queries (من غير @Query)

```java
List<Course> findByInstructorId(Long instructorId);
List<Course> findByStatus(CourseStatus status);
List<Course> findByLevel(CourseLevel level);
```

دول Spring بيولدهم تلقائياً من اسم الـ method — مش محتاج تكتب SQL.

**بتحطها امتى؟**

- لما الـ condition بسيطة (field واحد أو اتنين)
- مثال: `findByStatus` → Spring بيحولها لـ:

```sql
SELECT * FROM courses WHERE status = ?
```

---

## تانياً — الـ @Query (JPQL)
**JPQL** اختصار لـ:

**Java Persistence Query Language**

---

يعني بالعربي: لغة query خاصة بـ Java بتتكلم مع الـ **Objects** مش مع الـ **Database** مباشرة.

```java
// SQL — بتتكلم مع الـ Database
SELECT * FROM courses WHERE status = 'PUBLISHED'

// JPQL — بتتكلم مع الـ Java Objects
SELECT c FROM Course c WHERE c.status = 'PUBLISHED'
//           ↑ اسم الـ Class          ↑ اسم الـ field في الـ Class
```

فايدتها إنها **مش مرتبطة بـ Database معينة** — سواء PostgreSQL أو MySQL أو Oracle، الـ JPQL هتشتغل من غير ما تغير أي حاجة.

```java
@Query("SELECT c FROM Course c WHERE c.status = 'PUBLISHED'")
List<Course> findAllPublished();
```

**JPQL مش SQL** — بتكتب باسم الـ Entity مش اسم الـ Table:

```java
// SQL
SELECT * FROM courses WHERE status = 'PUBLISHED'

// JPQL
SELECT c FROM Course c WHERE c.status = 'PUBLISHED'
//           ↑ اسم الـ Class مش الـ Table
```

---

## تالثاً — الـ @Param

```java
@Query("SELECT c FROM Course c WHERE c.level = :level")
List<Course> findPublishedByLevel(@Param("level") CourseLevel level);
```

الـ `:level` في الـ Query بتاخد قيمتها من الـ `@Param("level")` — لازم الاسمين يكونوا متطابقين.

---

## بتستخدم @Query امتى؟

|الحالة|الحل|
|---|---|
|condition بسيطة|Derived Query|
|أكتر من condition|`@Query`|
|بتعمل JOIN بين entities|`@Query`|
|الاسم هيبقى طويل جداً|`@Query`|

مثال اسم طويل:

```java
// ❌ اسم مش عملي
findByStatusAndLevelAndInstructorId(...)

// ✅ أوضح
@Query("SELECT c FROM Course c WHERE c.status = :status AND c.level = :level AND c.instructor.id = :id")
```

---

## في الـ Queries بتاعتك — ملاحظة مهمة

```java
// ❌ بتكتب String بدل Enum
@Query("SELECT c FROM Course c WHERE c.status = 'PUBLISHED'")

// ✅ الأصح تستخدم الـ Enum
@Query("SELECT c FROM Course c WHERE c.status = :status")
List<Course> findAllPublished(@Param("status") CourseStatus status);

// واستخدامها في الـ Service
courseRepository.findAllPublished(CourseStatus.PUBLISHED);
```

لأن لو غيرت اسم الـ Enum value مستقبلاً الـ String هتبقى غلط من غير ما تعرف.

---

## الـ Queries دي هتستخدمها فين؟

```
findAllPublished         → getAllCourses() للـ Students (يشوفوا المنشور بس)
findPublishedByLevel     → getCoursesByLevel() للـ Students
findPublishedByInstructor → getCoursesByInstructor() للـ Students
```

يعني الـ Student دايماً يشوف PUBLISHED بس، أما الـ Instructor يشوف كورساته كلها.