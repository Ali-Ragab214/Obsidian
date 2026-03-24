تمام يا علي 👌  
خلينا نغطي **Topic مهم جدًا في الـ Backend** وهو **ORM** خصوصًا في بيئة **Java Persistence API** و **Hibernate ORM** لأنهم الأكثر استخدامًا مع **Spring Boot**.

هشرح لك الموضوع **من الصفر لحد مستوى الانترفيو**.

---

# أولاً: ما هو ORM ؟

**ORM = Object Relational Mapping**

يعني:

```
Mapping
Between
Objects (Java Classes)
and
Database Tables
```

---

# المشكلة التي يحلها ORM

لو اشتغلت بدون ORM ستكتب:

```sql
SELECT * FROM users
```

ثم تحول النتيجة إلى Object في Java.

لكن ORM يعمل ذلك تلقائيًا.

---

# الفكرة

```
Database Table
      │
      ▼
Java Entity
```

---

# مثال

## جدول في قاعدة البيانات

```
users
-----
id
name
email
```

---

## Java Entity

```java
@Entity
public class User {

 @Id
 private Long id;

 private String name;

 private String email;

}
```

---

الآن يمكننا حفظ المستخدم هكذا:

```java
userRepository.save(user);
```

بدل كتابة SQL.

---

# Diagram

```
Java Object
   │
   ▼
Hibernate
   │
   ▼
SQL Query
   │
   ▼
Database
```

---

# ثانياً: JPA vs Hibernate

## ما هو JPA ؟

**Java Persistence API**

هو **Specification (معيار)** وليس Framework.

يعني يحدد:

```
rules
interfaces
annotations
```

---

## ما هو Hibernate ؟

**Hibernate ORM**

هو **Implementation لـ JPA**.

يعني:

```
JPA = Interface
Hibernate = Implementation
```

---

# مثال

نكتب الكود باستخدام JPA:

```java
@Entity
```

لكن التنفيذ الفعلي يتم بواسطة Hibernate.

---

# ثالثاً: Entity Mapping

Entity Mapping يعني **ربط Class بجدول Database**.

---

## مثال

```java
@Entity
@Table(name = "products")
public class Product {

 @Id
 @GeneratedValue
 private Long id;

 private String name;

 private double price;

}
```

---

# ماذا يحدث؟

```
Product Class
      │
      ▼
products table
```

---

# Mapping Columns

يمكن تحديد اسم العمود.

```java
@Column(name="product_name")
private String name;
```

---

# رابعاً: Relationships

العلاقات بين الجداول.

---

# 1️⃣ One To Many

مثال:

```
Customer
   │
   ├── Order
   ├── Order
   └── Order
```

---

## Database

```
customers
---------
id
name

orders
---------
id
customer_id
```

---

## Entity

```java
@Entity
public class Customer {

 @Id
 private Long id;

 @OneToMany(mappedBy="customer")
 private List<Order> orders;

}
```

---

```java
@Entity
public class Order {

 @Id
 private Long id;

 @ManyToOne
 private Customer customer;

}
```

---

# Diagram

```
Customer
   │
   ▼
List<Order>
```

---

# 2️⃣ Many To Many

مثال:

```
Student
   │
   │
   ▼
Course
```

الطالب يمكنه أخذ عدة كورسات.

---

## Database

```
students
courses
students_courses
```

---

## Entity

```java
@ManyToMany
private List<Course> courses;
```

---

# Diagram

```
Student
   │
   ├── Course
   ├── Course
   └── Course
```

---

# خامساً: Lazy vs Eager

من أهم الأسئلة في الانترفيو.

---

# Lazy Loading

تحميل البيانات **عند الحاجة فقط**.

---

## مثال

عند جلب Customer:

```java
Customer customer = repo.findById(1);
```

لن يتم تحميل:

```
orders
```

إلا إذا طلبناها.

```java
customer.getOrders();
```

---

# Diagram

```
Load Customer
     │
     ▼
Orders NOT loaded
     │
     ▼
customer.getOrders()
     │
     ▼
Orders loaded
```

---

# Eager Loading

تحميل البيانات **مباشرة مع الكائن**.

---

## مثال

```java
@OneToMany(fetch = FetchType.EAGER)
```

عند جلب Customer:

```
Customer
Orders
```

يتم تحميلهم فورًا.

---

# المشكلة

Eager قد يسبب:

```
performance problems
```

لو البيانات كثيرة.

---

# القاعدة

غالبًا نستخدم:

```
Lazy Loading
```

---

# مقارنة

|Type|التحميل|
|---|---|
|Lazy|عند الطلب|
|Eager|فورًا|

---

# مثال كامل

```java
@Entity
public class Customer {

 @Id
 private Long id;

 private String name;

 @OneToMany(fetch = FetchType.LAZY)
 private List<Order> orders;

}
```

---

# الزتونة (ملخص الانترفيو)

ORM:

```
convert objects
to
database tables
```

---

JPA:

```
Specification
```

---

Hibernate:

```
JPA Implementation
```

---

Relationships:

```
OneToMany
ManyToOne
ManyToMany
OneToOne
```

---

Loading:

```
Lazy → default
Eager → careful
```

---

لو حابب يا علي، أشرح لك بعده **أخطر موضوع في ORM بيوقع 90٪ من المطورين** وهو:

```
N+1 Query Problem
```

وده سؤال مشهور جدًا في الانترفيوهات.