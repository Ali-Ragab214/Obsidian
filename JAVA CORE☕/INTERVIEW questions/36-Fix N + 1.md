# N+1 Problem in Spring Boot / Hibernate

## واحدة من أشهر مشاكل الأداء (Performance Problems) في الـ JPA

هذا السؤال يُسأل كثيرًا في مقابلات Spring Boot لأنه يقيس فهمك لكيفية عمل Hibernate داخليًا.

---

# أولاً: ما هي الـ N+1 Problem؟

هي مشكلة تحدث عندما يقوم Hibernate بتنفيذ:

- **Query واحدة** لجلب البيانات الأساسية (1)
    
- ثم **N Queries إضافية** لجلب البيانات المرتبطة (Relationships)
    

لذلك سميت

```text
1 + N Queries
```

وهذا يؤدي إلى بطء شديد في التطبيق.

---

# مثال عملي

لدينا كيانين:

## User

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user")
    private List<Order> orders;

}
```

---

## Order

```java
@Entity
public class Order {

    @Id
    private Long id;

    private String product;

    @ManyToOne
    private User user;

}
```

---

لدينا في قاعدة البيانات

```text
Users

1  Ali
2  Ahmed
3  Omar
```

وكل مستخدم لديه Orders.

---

# Repository

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

---

# Service

```java
List<User> users = repository.findAll();
```

---

حتى الآن لا توجد مشكلة.

لكن بعد ذلك

```java
for(User user : users){

    System.out.println(user.getOrders());

}
```

هنا تبدأ المشكلة.

---

# ماذا يحدث داخليًا؟

## أول Query

```sql
SELECT *
FROM users;
```

يعيد

```text
Ali

Ahmed

Omar
```

---

عندما يصل Hibernate إلى

```java
user.getOrders()
```

لـ Ali

ينفذ

```sql
SELECT *
FROM orders
WHERE user_id = 1;
```

---

ثم Ahmed

```sql
SELECT *
FROM orders
WHERE user_id = 2;
```

---

ثم Omar

```sql
SELECT *
FROM orders
WHERE user_id = 3;
```

---

إجمالي الـ Queries

```text
SELECT users

+

SELECT orders for Ali

+

SELECT orders for Ahmed

+

SELECT orders for Omar
```

=

```text
4 Queries
```

---

لو عندك

```text
1000 User
```

سيصبح

```text
1 + 1000

=

1001 Query
```

وهذه هي الـ N+1 Problem.

---

# الرسم

```text
findAll()

↓

SELECT users

↓

Ali

↓

SELECT Orders

↓

Ahmed

↓

SELECT Orders

↓

Omar

↓

SELECT Orders
```

---

# لماذا يحدث هذا؟

بسبب

```java
@OneToMany
```

الذي يكون افتراضياً

```java
LAZY
```

أي

```text
لا تحمل Orders

إلا عند الحاجة
```

ولذلك كل مرة تستدعي

```java
user.getOrders()
```

Hibernate ينفذ Query جديدة.

---

# هل جعلها EAGER يحل المشكلة؟

لو كتبنا

```java
@OneToMany(fetch = FetchType.EAGER)
```

سيقوم Hibernate بتحميل Orders مباشرة.

لكن هذا **ليس Best Practice**.

---

## لماذا؟

لأن كل مرة تجلب User

سيجلب Orders.

حتى لو لم تكن تحتاجها.

---

مثال

```java
User user = repository.findById(1L);
```

قد تحتاج فقط اسم المستخدم.

لكن Hibernate سيجلب أيضًا جميع Orders.

---

إذا كان المستخدم لديه

```text
5000 Orders
```

ستُحمَّل كلها دون داعٍ.

---

لذلك

```text
EAGER ≠ Solution
```

---

# الحل الأول: JOIN FETCH

مثال

```java
@Query("""
select u
from User u
join fetch u.orders
""")
```

Hibernate سينفذ

```sql
SELECT
u.*,
o.*
FROM users u
JOIN orders o
```

Query واحدة فقط.

---

# لكن هناك حل أفضل.

---

# @EntityGraph

وهو من أفضل حلول الـ N+1.

---

## ما هو؟

هو Annotation تخبر Hibernate

```text
عندما تجلب User

اجلب Orders معها
```

في Query واحدة.

---

# مثال

Repository

```java
public interface UserRepository
extends JpaRepository<User,Long>{

    @EntityGraph(attributePaths = "orders")
    List<User> findAll();

}
```

---

عندما تستدعي

```java
repository.findAll();
```

Hibernate يولد Query مشابهة لـ

```sql
SELECT
u.*,
o.*
FROM users u
LEFT JOIN orders o
```

بدلاً من

```text
1001 Query
```

سيصبح

```text
1 Query
```

---

# ماذا تعني attributePaths؟

مثلاً

```java
@EntityGraph(attributePaths = "orders")
```

أي

```text
Load Orders
```

---

لو

```java
@EntityGraph(attributePaths = {
    "orders",
    "address"
})
```

يعني

```text
Load Orders

+

Load Address
```

في نفس الـ Query.

---

# مثال أكبر

```java
@Entity
public class User{

    @OneToMany
    List<Order> orders;

    @OneToOne
    Address address;

    @ManyToMany
    List<Role> roles;

}
```

يمكن كتابة

```java
@EntityGraph(attributePaths = {
        "orders",
        "address",
        "roles"
})
```

---

سيولد Hibernate

```text
User

↓

Orders

↓

Address

↓

Roles
```

كلها في Query واحدة أو باستخدام الـ JOIN المناسب.

---

# كيف يعمل داخليًا؟

بدون EntityGraph

```text
findAll()

↓

SELECT users

↓

Proxy Orders

↓

getOrders()

↓

SELECT Orders
```

---

مع EntityGraph

```text
findAll()

↓

Hibernate يرى EntityGraph

↓

يبني Fetch Plan

↓

يجلب Orders مع User

↓

ينشئ Objects

↓

انتهى
```

---

# ما هو Fetch Plan؟

هو خطة يخبر بها Hibernate

```text
ما الذي سيتم تحميله؟

وما الذي لن يتم تحميله؟
```

---

# مثال

بدون EntityGraph

```text
User

↓

Orders Lazy

↓

Address Lazy

↓

Role Lazy
```

---

مع EntityGraph

```text
User

↓

Orders Loaded

↓

Address Loaded

↓

Role Lazy
```

---

# هل EntityGraph أفضل من EAGER؟

نعم.

لأنه

```text
EAGER

↓

دائمًا
```

أما

```text
EntityGraph

↓

فقط في هذا الـ Query
```

---

مثلاً

```java
findAll()
```

يمكن أن يحمل Orders.

بينما

```java
findById()
```

لا يحملها.

وهذا يعطيك مرونة كبيرة.

---

# مثال عملي

```java
@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

بينما

```java
Optional<User> findById(Long id);
```

لن يحمل Orders.

---

# هل يعمل مع Nested Relationships؟

نعم.

مثلاً

```text
User

↓

Orders

↓

Items
```

يمكن كتابة

```java
@EntityGraph(attributePaths = {
        "orders",
        "orders.items"
})
```

---

Hibernate سيجلب

```text
User

↓

Orders

↓

Items
```

دفعة واحدة.

---

# متى أستخدم @EntityGraph؟

استخدمه عندما:

- لديك علاقات `LAZY`.
    
- تعرف مسبقًا أنك ستحتاج العلاقات في هذا الاستعلام.
    
- تريد تجنب مشكلة N+1 بدون تغيير إعدادات الـ Entity نفسها.
    

---

# مقارنة بين الحلول

|الطريقة|تحل N+1؟|Best Practice؟|ملاحظات|
|---|---|---|---|
|`FetchType.LAZY`|❌|✅|الافتراضي الموصى به، لكن قد يسبب N+1 إذا لم تنتبه.|
|`FetchType.EAGER`|أحيانًا|❌|قد يحمل بيانات كثيرة بلا حاجة ويؤثر على الأداء.|
|`JOIN FETCH`|✅|✅|ممتاز لاستعلامات مخصصة ومعقدة.|
|`@EntityGraph`|✅|✅✅|أنظف وأكثر مرونة مع Repository methods.|

---

# إجابة مختصرة في المقابلة

> **The N+1 problem occurs when Hibernate executes one query to fetch the parent entities and then executes an additional query for each parent entity to load its lazy relationships. For example, fetching 100 users and then accessing each user's orders can result in 101 SQL queries. This degrades performance significantly. One of the best solutions is using `@EntityGraph`, which tells Hibernate to fetch the required associations eagerly for a specific query only, generating a single optimized SQL query with the necessary joins while keeping the entity mapping itself lazy.**

هذه الإجابة توضح أنك لا تعرف المشكلة فقط، بل تفهم **سببها الداخلي وكيفية معالجتها بطريقة احترافية**.