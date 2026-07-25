جامد، ودي فعلاً تعتبر الخطوة الطبيعية بعد Derived Query Methods.

لو هرتبهم من حيث القوة في Spring Data JPA فهي كده:

> Derived Query → JPQL → Native SQL → Criteria API / Specifications → QueryDSL

وخليني أقولك حاجة مهمة قبل ما نبدأ:

> **لو فهمت JPQL كويس، هتفهم 80% من Spring Data JPA.**  
> لأن كل اللي بعده مبني عليه بشكل أو بآخر.

---

# أولاً: يعني إيه JPQL؟

JPQL = **Java Persistence Query Language**

هي لغة Query شبه SQL جدًا، لكن بتتعامل مع **الـ Entities** بدل الـ Tables.

يعني بدل ما تقول:

```sql
SELECT * FROM users;
```

بتقول

```jpql
SELECT u FROM User u
```

لاحظ الفرق

SQL

```sql
FROM users
```

JPQL

```jpql
FROM User
```

يعني:

SQL بيتعامل مع

- Tables
    
- Columns
    

أما JPQL بيتعامل مع

- Entity Classes
    
- Fields
    

وده أهم فرق.

---

# ليه أصلاً اتعمل JPQL؟

تخيل عندك

```java
@Entity
class User {

    private String firstName;

}
```

والـ Database فيها

```sql
users

first_name
```

لو استخدمت SQL

هتفضل حافظ أسماء الـ Tables والـ Columns.

لكن JPQL بيستخدم

```java
User
firstName
```

فلو غيرت اسم الجدول

```java
@Table(name="app_users")
```

ولا JPQL هيتأثر.

وده اسمه

Database Independence

---

# مقارنة سريعة

Derived Query

```java
findByNameAndAgeGreaterThan(...)
```

ممتازة

لكن أول ما الكويري تكبر

بتبقى:

```java
findByNameAndAgeGreaterThanAndDepartmentAndSalaryBetweenAndStatus...
```

😂😂

هتبقى كارثة.

ساعتها بنستخدم JPQL.

---

# شكل Repository

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

}
```

---

# كتابة JPQL

عن طريق

```java
@Query
```

مثال

```java
@Query("SELECT u FROM User u")
List<User> findAllUsers();
```

---

# معنى كل كلمة

```java
SELECT u
```

هات الـ Entity

---

```java
FROM User
```

User Entity

مش اسم الجدول

---

```java
u
```

Alias

زي

```sql
SELECT *
FROM users u
```

---

# WHERE

مثال

```java
@Query("""
SELECT u
FROM User u
WHERE u.name = :name
""")
List<User> findByName(String name);
```

أو

```java
List<User> findByName(@Param("name") String name);
```

---

# ليه استخدمنا :

```java
:name
```

دي اسمها

Named Parameter

أفضل من

```java
?
```

---

مثال

```java
@Query("""
SELECT u
FROM User u
WHERE u.age > :age
""")
```

---

تشغيلها

```java
repo.findAdults(18);
```

---

# أكتر من Parameter

```java
@Query("""
SELECT u
FROM User u
WHERE u.name=:name
AND u.age>=:age
""")
```

---

Repository

```java
List<User> search(

@Param("name") String name,

@Param("age") int age

);
```

---

# العمليات

Greater Than

```java
WHERE u.age > :age
```

---

Less Than

```java
WHERE u.salary < :salary
```

---

Between

```java
WHERE u.salary
BETWEEN :min
AND :max
```

---

LIKE

```java
WHERE u.name LIKE %:name%
```

مثلاً

```java
Ali
```

هيجيب

```
Ali Ragab

Ali Ahmed

Mohamed Ali
```

---

Starts With

```java
LIKE :name%
```

---

Ends With

```java
LIKE %:name
```

---

# AND

```java
WHERE u.age>20
AND u.active=true
```

---

# OR

```java
WHERE u.role='ADMIN'
OR u.role='MANAGER'
```

---

# IN

بدل

```java
OR
OR
OR
OR
```

نستخدم

```java
WHERE u.role IN :roles
```

Repository

```java
List<User> find(List<String> roles);
```

---

# NOT IN

```java
WHERE u.role NOT IN :roles
```

---

# ORDER BY

```java
SELECT u
FROM User u
ORDER BY u.salary DESC
```

---

ASC

```java
ORDER BY u.name ASC
```

---

# DISTINCT

```java
SELECT DISTINCT u.department
FROM User u
```

---

# COUNT

```java
SELECT COUNT(u)
FROM User u
```

ترجع

```java
Long
```

---

# SUM

```java
SELECT SUM(u.salary)
FROM User u
```

---

# AVG

```java
SELECT AVG(u.salary)
FROM User u
```

---

# MAX

```java
SELECT MAX(u.salary)
```

---

# MIN

```java
SELECT MIN(u.salary)
```

---

# EXISTS ؟

مفيش EXISTS بنفس أسلوب SQL في أغلب استخدامات JPQL، لكن غالبًا بتستخدم:

```java
SELECT COUNT(u) > 0
```

أو

```java
existsBy...
```

في Spring Data.

---

# Pagination

```java
Page<User> findAll(Pageable pageable);
```

أو

```java
@Query("""
SELECT u
FROM User u
WHERE u.active=true
""")
Page<User> find(Pageable pageable);
```

---

# JOIN

دي من أهم الحاجات.

لو عندي

```java
User
```

و

```java
Department
```

```java
@ManyToOne
Department department;
```

أجيب اسم القسم

```java
SELECT u
FROM User u
JOIN u.department d
WHERE d.name='IT'
```

لاحظ

مش كتبنا

```
department_id
```

لأننا بنتعامل مع Object.

---

# LEFT JOIN

```java
LEFT JOIN u.department d
```

حتى لو مفيش Department.

---

# FETCH JOIN

من أهم الحاجات في الانترفيو.

```java
SELECT u
FROM User u
JOIN FETCH u.department
```

وده بيحل

N+1 Problem

لأنه بيجيب البيانات مرة واحدة.

---

# Projection

بدل ما أجيب Entity كاملة

```java
SELECT u.name
```

---

أو

```java
SELECT u.name,u.salary
```

---

DTO Projection

```java
public record UserDto(

String name,

double salary

){}
```

JPQL

```java
SELECT new com.demo.UserDto(
u.name,
u.salary
)
FROM User u
```

ودي من أفضل الممارسات لما تكون عايز تعرض بيانات فقط بدل تحميل الـ Entity بالكامل، لأنها تقلل استهلاك الذاكرة وتحسن الأداء.

---

# Update Query

JPQL

```java
@Modifying
@Transactional

@Query("""
UPDATE User u
SET u.salary=:salary
WHERE u.id=:id
""")
```

لازم

```java
@Modifying
```

علشان دي مش Select.

---

# Delete

```java
DELETE
FROM User u
WHERE u.id=:id
```

برضه

```java
@Modifying
```

---

# Subquery

```java
SELECT u
FROM User u
WHERE u.salary >

(
SELECT AVG(x.salary)

FROM User x

)
```

هيجيب الموظفين اللي مرتبهم أعلى من المتوسط.

---

# CASE WHEN

```java
SELECT
CASE

WHEN u.salary>10000

THEN 'HIGH'

ELSE 'LOW'

END

FROM User u
```

---

# Functions

Upper

```java
UPPER(u.name)
```

Lower

```java
LOWER(u.name)
```

Length

```java
LENGTH(u.name)
```

Concat

```java
CONCAT(u.firstName,' ',u.lastName)
```

Trim

```java
TRIM(u.name)
```

Current Date

```java
CURRENT_DATE
```

Current Time

```java
CURRENT_TIME
```

Current Timestamp

```java
CURRENT_TIMESTAMP
```

---

# Derived ولا JPQL؟

استخدم Derived Query لما:

- الكويري بسيطة.
    
- اسم الميثود ما زال مقروءًا.
    
- مفيش Joins معقدة أو Aggregation.
    

استخدم JPQL لما:

- الكويري طويلة أو اسم الميثود بقى غير عملي.
    
- محتاج `JOIN` أو `FETCH JOIN`.
    
- محتاج `GROUP BY` أو `HAVING`.
    
- محتاج `DTO Projection`.
    
- محتاج `UPDATE` أو `DELETE`.
    
- محتاج دوال مثل `COUNT` أو `SUM` أو `AVG`.
    
- محتاج Subqueries أو CASE expressions.
    

---

# Use Cases حقيقية

### 1. Login

```java
SELECT u
FROM User u
WHERE u.email=:email
```

---

### 2. Search

```java
WHERE LOWER(u.name)
LIKE LOWER(CONCAT('%',:keyword,'%'))
```

---

### 3. Dashboard

```java
SELECT COUNT(u)
```

---

### 4. Highest Salary

```java
SELECT MAX(u.salary)
```

---

### 5. Department Employees

```java
JOIN u.department d
```

---

### 6. Employees Above Average

```java
WHERE salary>

(
SELECT AVG(...)
)
```

---

### 7. Bulk Update

```java
UPDATE User
SET active=false
```

---

### 8. Reports

```java
GROUP BY
```

مع

```java
COUNT()
```

---

# أشهر أخطاء المبتدئين

❌ كتابة اسم الجدول بدل الـ Entity.

```sql
FROM users
```

الصحيح:

```jpql
FROM User
```

---

❌ كتابة اسم العمود.

```sql
first_name
```

الصحيح:

```java
firstName
```

---

❌ نسيان `@Param`.

---

❌ نسيان `@Modifying` في Update/Delete.

---

❌ نسيان `@Transactional` مع عمليات التعديل.

---

❌ استخدام `JOIN` عادي ثم الوصول للعلاقات خارج الـ Transaction، مما قد يسبب `LazyInitializationException` إذا كانت العلاقة Lazy ولم يتم تحميلها.

---

# أسئلة انترفيو وإجاباتها

### 1. ما هو JPQL؟

لغة استعلامات خاصة بـ JPA تتعامل مع الـ **Entities وحقولها** بدل الـ **Tables وColumns**، مما يجعل الكود مستقلًا عن تفاصيل قاعدة البيانات.

---

### 2. ما الفرق بين JPQL و SQL؟

- SQL يتعامل مع الجداول والأعمدة.
    
- JPQL يتعامل مع الـ Entities والـ Fields.
    
- JPQL Portable بين قواعد البيانات المختلفة لأنه يعتمد على الـ Mapping الخاص بـ JPA.
    

---

### 3. ما الفرق بين Derived Query و JPQL؟

- Derived Query مناسب للاستعلامات البسيطة ويُنشأ تلقائيًا من اسم الميثود.
    
- JPQL مناسب للاستعلامات المعقدة التي تتضمن Joins وAggregations وDTO Projections وBulk Updates.
    

---

### 4. لماذا نستخدم `@Query`؟

لأن بعض الاستعلامات لا يمكن التعبير عنها بسهولة باستخدام أسماء الميثود، أو لأن اسم الميثود سيصبح طويلًا وغير قابل للقراءة.

---

### 5. ما هو `JOIN FETCH`؟

هو Join يقوم بجلب الـ Association في نفس الاستعلام، ويستخدم غالبًا لتجنب مشكلة **N+1 Query Problem** وتحسين الأداء.

---

### 6. لماذا نستخدم `@Modifying`؟

لإخبار Spring Data أن الاستعلام يقوم بعملية **UPDATE** أو **DELETE** وليس **SELECT**.

---

### 7. هل يمكن لـ JPQL إنشاء DTO مباشرة؟

نعم، باستخدام Constructor Expression:

```java
SELECT new com.example.dto.UserDto(u.name, u.email)
FROM User u
```

---

### 8. هل JPQL أسرع من Derived Query؟

لا، في النهاية Spring يحول Derived Query إلى JPQL (أو ما يعادلها) ثم يقوم مزود JPA مثل Hibernate بترجمتها إلى SQL. الفرق الأساسي في سهولة كتابة وصيانة الكود، وليس في الأداء.

---

### 9. متى أستخدم Native Query بدل JPQL؟

عندما تحتاج ميزات خاصة بقاعدة البيانات (مثل Window Functions أو CTEs أو JSON Functions)، أو تريد استخدام SQL لا يدعمه JPQL.

---

### 10. هل JPQL يتعامل مع Lazy Loading؟

بشكل غير مباشر. إذا احتجت تحميل العلاقات مع الـ Entity في نفس الاستعلام، فاستخدم `JOIN FETCH` لتجنب تحميل كل علاقة في Query منفصلة أو الوقوع في `LazyInitializationException`.

---

## نصيحتي للمذاكرة

احفظ التسلسل ده لأنه بيمثل رحلة أغلب مشاريع Spring Boot:

1. **Derived Query Methods** → للاستعلامات البسيطة.
    
2. **JPQL** → للاستعلامات المتوسطة والمتقدمة.
    
3. **Native SQL** → عند الحاجة لإمكانيات خاصة بقاعدة البيانات.
    
4. **Specifications (Criteria API)** → عندما تكون شروط البحث Dynamic.
    
5. **QueryDSL** → لمشاريع Enterprise الكبيرة التي تحتاج Type Safety واستعلامات ديناميكية معقدة.
    

ولو أنت داخل على انترفيو Spring Backend، فأكثر ثلاث نقاط بيتسأل عنهم في JPQL هم:

- الفرق بين **JPQL و SQL**.
    
- الفرق بين **JOIN و JOIN FETCH** وعلاقتهما بمشكلة **N+1**.
    
- **DTO Projection** وسبب استخدامه لتحسين الأداء بدل إرجاع الـ Entity كاملة.