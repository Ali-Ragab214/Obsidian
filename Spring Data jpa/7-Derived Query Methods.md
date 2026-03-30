حلو 🔥 دي من أهم الحاجات في Spring Data JPA  
**Derived Query Methods** تعتبر سوبر باور حرفياً 😄

خلينا نشرحها من الصفر وبشكل عملي.

---

# يعني ايه Derived Query Methods ؟

هي طريقة تخلي **Spring يكتبلك SQL لوحده**  
بمجرد اسم الميثود بس 😎

بدل ما تكتب:

```sql
select * from author where name = ?
```

انت تكتب بس:

```java
findByName(String name)
```

و Spring يعمل الباقي 🤯

---

# ليه موجودة أصلاً ؟ (لزمـتها)

قبل Spring Data JPA كنا بنكتب:

- JPQL
    
- Native SQL
    
- Criteria API
    

وده كان:

- طويل
    
- متعب
    
- مكرر كتير
    

فجت فكرة:

> "طب ما نخلّي اسم الميثود نفسه يكون Query"

وده اسمه:  
**Query Method DSL**

---

# نبدأ على الريبو بتاعك 👇

انت عامل:

```java
public interface AuthorRepository extends JpaRepository<Author,Integer> {
}
```

خلينا نفترض إن الـ Entity:

```java
@Entity
public class Author {

    @Id
    private Integer id;

    private String name;
    private String email;
    private int age;
}
```

---

# 1️⃣ أبسط Derived Query

## البحث باسم المؤلف

```java
List<Author> findByName(String name);
```

Spring يترجمها:

```sql
select * from author where name = ?
```

وتستخدمها:

```java
authorRepository.findByName("Ali");
```

---

# 2️⃣ البحث بأكتر من شرط

```java
List<Author> findByNameAndAge(String name, int age);
```

SQL:

```sql
select * from author 
where name = ? and age = ?
```

---

# 3️⃣ OR بدل AND

```java
List<Author> findByNameOrEmail(String name, String email);
```

---

# 4️⃣ Comparison Operators 🔥

## أكبر من

```java
List<Author> findByAgeGreaterThan(int age);
```

SQL:

```sql
where age > ?
```

## أصغر من

```java
List<Author> findByAgeLessThan(int age);
```

## بين قيمتين

```java
List<Author> findByAgeBetween(int start, int end);
```

---

# 5️⃣ Like / Contains (بحث نصي مهم جداً)

## contains

```java
List<Author> findByNameContaining(String keyword);
```

لو كتبت:

```java
findByNameContaining("li")
```

SQL:

```sql
where name like %li%
```

---

## يبدأ بـ

```java
List<Author> findByNameStartingWith(String prefix);
```

## ينتهي بـ

```java
List<Author> findByNameEndingWith(String suffix);
```

---

# 6️⃣ Ignore Case 😍

```java
List<Author> findByNameIgnoreCase(String name);
```

---

# 7️⃣ Order By Sorting

```java
List<Author> findByAgeGreaterThanOrderByAgeDesc(int age);
```

SQL:

```sql
where age > ? order by age desc
```

---

# 8️⃣ Top / First (Pagination بسيطة)

```java
Author findTopByOrderByAgeDesc();
```

يرجع أكبر Author في العمر 👴

```java
List<Author> findTop3ByOrderByAgeDesc();
```

---

# 9️⃣ Exists Query (مهم جداً للـ validation)

```java
boolean existsByEmail(String email);
```

تستخدمها في register مثلاً:

```java
if(authorRepository.existsByEmail(email)){
   throw new RuntimeException("Email already exists");
}
```

---

# 🔥 اهم نقطة لازم تحفظها

اسم الميثود بيتكوّن من 3 أجزاء:

```
find + By + Properties + Conditions
```

مثال:

```
findByNameAndAgeGreaterThanOrderByAgeDesc
```

Spring بيعمل Parser للاسم ويحوّله SQL.

---

# امتي استخدم Derived Queries ؟

استخدمها لما:

- Query بسيط
    
- Query متوسط
    
- Query متكرر
    

❌ متستخدمهاش لما:

- Query معقد جداً
    
- Joins كتير
    
- Subqueries
    

ساعتها نستخدم:

- @Query (JPQL)
    
- Native Query
    

ودي الخطوة اللي بعدها 👀

---

تحب نخش على **@Query annotation** ؟ 😏🔥