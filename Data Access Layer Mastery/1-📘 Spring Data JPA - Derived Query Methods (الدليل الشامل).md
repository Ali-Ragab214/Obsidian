# 📘 Spring Data JPA - Derived Query Methods (الدليل الشامل)

> **المستوى:** من الصفر حتى مستوى Senior + Interview  
> **الهدف:** بعد الانتهاء من هذا الشرح ستكون قادرًا على كتابة معظم استعلامات Spring Data JPA بدون كتابة SQL أو JPQL.

---

# أولًا: ما هي الـ Data Access Layer؟

قبل أن نتكلم عن Derived Query Methods، لازم نفهم إحنا فين.

لو عندك Application Spring Boot.

```
Controller
      │
      ▼
Service
      │
      ▼
Repository (Data Access Layer)
      │
      ▼
Database
```

الـ Repository هو المسؤول عن:

- قراءة البيانات
    
- حفظ البيانات
    
- تعديل البيانات
    
- حذف البيانات
    

يعني أي تعامل مع Database يمر من هنا.

مثلاً:

```
UserController
        │
        ▼
UserService
        │
        ▼
UserRepository
        │
        ▼
PostgreSQL
```

---

# ما هي Derived Query Methods؟

هي ميزة موجودة في Spring Data JPA تسمح لك بكتابة Query **من اسم الميثود فقط**.

بدل أن تكتب:

```sql
SELECT *
FROM users
WHERE email = ?
```

تكتب فقط

```java
findByEmail(String email)
```

وSpring يفهم لوحده ويولد الـ SQL.

يعني هو يقوم بعمل:

```
Method Name
      │
      ▼
Parser
      │
      ▼
JPQL
      │
      ▼
SQL
```

أنت لا تكتب SQL إطلاقًا.

---

# مثال

لدينا Entity

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    private String email;

    private Integer age;

    private Boolean active;
}
```

Repository

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

}
```

---

## نريد البحث بالإيميل

بدلاً من

```sql
SELECT *
FROM users
WHERE email='ali@gmail.com';
```

نكتب

```java
Optional<User> findByEmail(String email);
```

فقط.

Spring يولد

```sql
SELECT *
FROM users
WHERE email=?
```

---

# لماذا اسمها Derived؟

لأن الـ Query تم اشتقاقها من اسم الميثود.

```
findByEmail
```

↓

يفهم Spring

```
SELECT *
WHERE email=?
```

---

# كيف يقرأ Spring اسم الميثود؟

يقرأها هكذا:

```
find
By
Email
```

كل كلمة لها معنى.

```
find
```

يعني SELECT

```
By
```

يعني WHERE

```
Email
```

اسم العمود

---

# Anatomy

```
findByName
│    │
│    └──── field
│
└──────── operation
```

---

# أهم Operations

---

## find

يرجع البيانات.

```java
findByEmail()
```

---

## exists

يرجع boolean

```java
existsByEmail(String email);
```

SQL

```sql
SELECT EXISTS(...)
```

Use Case

التأكد أن الإيميل موجود.

---

## count

يرجع عدد.

```java
countByActiveTrue();
```

SQL

```sql
SELECT COUNT(*)
```

---

## delete

يحذف.

```java
deleteByEmail(String email);
```

---

## remove

مثل delete.

```java
removeByAgeLessThan(18);
```

---

# أشهر Keywords

---

## And

```java
findByNameAndAge(String name,Integer age)
```

SQL

```sql
WHERE name=?
AND age=?
```

---

## Or

```java
findByNameOrEmail(...)
```

---

## Between

```java
findByAgeBetween(20,30)
```

SQL

```sql
age BETWEEN 20 AND 30
```

---

## LessThan

```java
findByAgeLessThan(30)
```

---

## GreaterThan

```java
findByAgeGreaterThan(20)
```

---

## LessThanEqual

```java
findByAgeLessThanEqual(30)
```

---

## GreaterThanEqual

```java
findByAgeGreaterThanEqual(30)
```

---

## IsNull

```java
findByEmailIsNull()
```

---

## IsNotNull

```java
findByEmailIsNotNull()
```

---

## True

```java
findByActiveTrue()
```

---

## False

```java
findByActiveFalse()
```

---

## Like

```java
findByNameLike("%Ali%")
```

---

## Containing ⭐

الأكثر استخدامًا.

```java
findByNameContaining("Ali")
```

Spring يضيف `%`.

---

## StartingWith

```java
findByNameStartingWith("Al")
```

```
LIKE 'Al%'
```

---

## EndingWith

```java
findByNameEndingWith("li")
```

```
LIKE '%li'
```

---

## IgnoreCase

```java
findByNameIgnoreCase(String name)
```

---

## In

```java
findByAgeIn(List<Integer> ages)
```

SQL

```sql
WHERE age IN (...)
```

---

## NotIn

```java
findByAgeNotIn(...)
```

---

## OrderBy

```java
findByActiveTrueOrderByAgeAsc()
```

---

Descending

```java
findByActiveTrueOrderByAgeDesc()
```

---

# استخدام أكتر من Keyword

```java
findByAgeGreaterThanAndActiveTrueOrderByNameAsc()
```

Spring يفهمها كلها.

---

# Nested Property

لدينا

```java
User

↓

Department

↓

name
```

يمكن كتابة

```java
findByDepartmentName(String departmentName)
```

بدون Query.

---

# Top

```java
findTopByOrderBySalaryDesc()
```

أعلى موظف راتبًا.

---

Top3

```java
findTop3ByOrderBySalaryDesc()
```

---

First

```java
findFirstByOrderByIdDesc()
```

---

Distinct

```java
findDistinctByDepartmentName(...)
```

---

# Return Types

يمكن أن ترجع

Entity

```java
User
```

Optional

```java
Optional<User>
```

List

```java
List<User>
```

Page

```java
Page<User>
```

Slice

```java
Slice<User>
```

Stream

```java
Stream<User>
```

Boolean

```java
boolean
```

Long

```java
long
```

---

# Use Cases

## Login

```java
findByEmail()
```

---

## Register

```java
existsByEmail()
```

---

## Dashboard

```java
countByActiveTrue()
```

---

## Search

```java
findByNameContaining()
```

---

## Pagination

```java
findByActiveTrue(Pageable pageable)
```

---

## Latest Users

```java
findTop10ByOrderByCreatedAtDesc()
```

---

## Employees

```java
findByDepartmentName()
```

---

## Products

```java
findByPriceBetween()
```

---

## Orders

```java
findByStatus()
```

---

## Inventory

```java
findByQuantityLessThan()
```

---

# متى لا نستخدم Derived Queries؟

إذا أصبح اسم الميثود طويلًا جدًا.

مثلاً

```java
findByDepartmentNameAndSalaryGreaterThanAndAgeBetweenAndActiveTrueOrderByCreatedAtDesc()
```

😅

هنا الأفضل استخدام:

- JPQL
    
- @Query
    
- Specification
    
- QueryDSL
    

---

# أشهر أخطاء المبتدئين

❌

```java
findByEmails()
```

بينما اسم الحقل

```
email
```

يجب أن يكون مطابقًا لاسم الـ Property في الـ Entity.

---

❌

```java
findByemail()
```

الحرف الأول بعد By يجب أن يكون Capital.

```java
findByEmail()
```

---

❌

نسيان أن

```java
findByEmail()
```

قد لا يجد بيانات.

الأفضل

```java
Optional<User>
```

---

# أسئلة انترفيو

## 1

**What are Derived Query Methods?**

---

## 2

How does Spring Data JPA generate SQL from method names?

---

## 3

Difference between

```
findBy
```

and

```
existsBy
```

---

## 4

Difference between

```
Containing
```

and

```
Like
```

---

## 5

Difference between

```
Top
```

and

```
First
```

---

## 6

When should you stop using Derived Query Methods?

---

## 7

Can Derived Queries navigate relationships?

مثال

```java
findByDepartmentName()
```

---

## 8

How does Spring know the column name?

من خلال اسم الـ **Property** في الـ Entity، وليس اسم العمود في قاعدة البيانات. فإذا كان لديك:

```java
@Column(name = "user_email")
private String email;
```

فالميثود تكون:

```java
findByEmail(...)
```

وليس في الداتا بيز:

```java
findByUserEmail(...)
```

---

## 9

Can Derived Query Methods support pagination and sorting?

نعم، بإضافة `Pageable` أو `Sort` كـ Parameter:

```java
Page<User> findByActiveTrue(Pageable pageable);

List<User> findByRole(String role, Sort sort);
```

---

## 10

What are the limitations of Derived Query Methods?

- أسماء الميثود قد تصبح طويلة جدًا وصعبة القراءة.
    
- لا تناسب الـ Queries المعقدة (Subqueries, Aggregations, Joins المعقدة).
    
- في هذه الحالة نستخدم:
    
    - `@Query`
        
    - JPQL
        
    - Native Query
        
    - Specifications
        
    - QueryDSL
        

---

# 💡 الخلاصة

احفظ المعادلة التالية لأنها تلخص الفكرة كلها:

```
Repository Method Name
          │
          ▼
Spring Data Parser
          │
          ▼
JPQL
          │
          ▼
Hibernate
          │
          ▼
SQL
          │
          ▼
Database
```

**Derived Query Methods** هي واحدة من أقوى ميزات Spring Data JPA لأنها تقلل كتابة الأكواد، تجعل الـ Repository أكثر وضوحًا، وتغطي معظم عمليات البحث اليومية. لكن عندما تصبح شروط البحث معقدة أو يطول اسم الميثود بشكل مبالغ فيه، فمن الأفضل الانتقال إلى `@Query` أو `Specifications` أو `QueryDSL`.