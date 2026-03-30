حلو 🔥 دلوقتي دخلنا على المرحلة اللي بعدها في JPA  
**Custom Queries باستخدام @Query**

دي بنستخدمها لما Derived Queries تبقى مش كفاية.

---

# امتى احتاج Custom Query أصلاً؟

استخدم Derived Methods لما الكويري بسيط.

لكن استخدم **@Query** لما:

- الكويري طويل
    
- فيه joins
    
- فيه aggregation (count / sum)
    
- فيه subquery
    
- فيه update/delete
    
- محتاج تتحكم في SQL بنفسك
    

يعني ببساطة:

> لما اسم الميثود يبقى رواية 😂

---

# أنواع Custom Queries في Spring Data JPA

عندنا نوعين:

1️⃣ JPQL Query  
2️⃣ Native SQL Query

خلينا نمشي واحدة واحدة.

---

# أولاً: JPQL Query

JPQL = Java Persistence Query Language

⚠️ أهم نقطة  
JPQL بيتعامل مع **Entity مش Table**

يعني تكتب:

```
Author
```

مش:

```
author_table
```

---

## مثال 1 — البحث بالاسم

بدل Derived Method:

```java
List<Author> findByName(String name);
```

نكتبها Custom:

```java
@Query("SELECT a FROM Author a WHERE a.name = :name")
List<Author> findAuthorsByName(@Param("name") String name);
```

لاحظ:

- a ده alias زي SQL
    
- :name ده parameter
    

---

## مثال 2 — شرطين

```java
@Query("SELECT a FROM Author a WHERE a.age > :age AND a.email = :email")
List<Author> findAuthors(@Param("age") int age,
                         @Param("email") String email);
```

---

## Example 3 — LIKE search 🔥

```java
@Query("SELECT a FROM Author a WHERE a.name LIKE %:keyword%")
List<Author> searchByName(@Param("keyword") String keyword);
```

دي مهمة جداً للـ search.

---

# Projection (نرجع جزء من البيانات فقط) 😍

بدل ما نرجع Author كامل نرجع الاسم بس:

```java
@Query("SELECT a.name FROM Author a")
List<String> findAllNames();
```

مهم جداً لتحسين الأداء.

---

# Aggregation Queries 🔥🔥

## Count

```java
@Query("SELECT COUNT(a) FROM Author a")
long countAuthors();
```

## Average Age

```java
@Query("SELECT AVG(a.age) FROM Author a")
double averageAge();
```

---

# Pagination مع @Query

```java
@Query("SELECT a FROM Author a WHERE a.age > :age")
Page<Author> findAuthors(@Param("age") int age, Pageable pageable);
```

وتستخدمها:

```java
PageRequest page = PageRequest.of(0,5);
authorRepository.findAuthors(20,page);
```

---

# Update & Delete Queries ⚠️ مهم جداً

لازم نستخدم:

```
@Modifying
@Transactional
```

## Update

```java
@Modifying
@Transactional
@Query("UPDATE Author a SET a.age = :age WHERE a.id = :id")
int updateAuthorAge(@Param("id") int id,
                    @Param("age") int age);
```

---

## Delete

```java
@Modifying
@Transactional
@Query("DELETE FROM Author a WHERE a.age < :age")
int deleteYoungAuthors(@Param("age") int age);
```

---

# ثانياً: Native SQL Query

لما تحتاج SQL حقيقي 100%

مثلاً فيه table اسمها:

```
authors
```

## مثال Native Query

```java
@Query(value = "SELECT * FROM authors WHERE age > :age",
       nativeQuery = true)
List<Author> findAuthorsNative(@Param("age") int age);
```

---

# امتى استخدم Native ؟

لما تحتاج:

- Database specific features
    
- Stored procedures
    
- Complex joins
    
- Performance tuning
    

لكن الأفضل دايماً:

> استخدم JPQL الأول

---

# الفرق بين Derived vs @Query vs Native

|النوع|امتى استخدمه|
|---|---|
|Derived|Queries بسيطة|
|@Query JPQL|Queries متوسطة|
|Native SQL|Queries معقدة جداً|

---

# الخلاصة 🏆

Custom Queries = تحكم كامل في الاستعلامات

أهم annotations تحفظهم:

```java
@Query
@Param
@Modifying
@Transactional
```

---

الخطوة اللي بعدها الطبيعية جداً:  
**Pagination + Sorting بشكل احترافي** 😏🔥