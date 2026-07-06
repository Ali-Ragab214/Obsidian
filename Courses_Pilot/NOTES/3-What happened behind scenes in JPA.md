
---

# أولًا: زمان قبل Spring Data JPA

قبل الـ JPA الناس كانت بتتعامل مع الداتابيز يدويًا.

كنت بتعمل:

```java
Connection connection = DriverManager.getConnection(...);

PreparedStatement ps =
    connection.prepareStatement(
        "SELECT * FROM users WHERE id = ?"
    );

ps.setLong(1, id);

ResultSet rs = ps.executeQuery();
```

يعني:

- تفتح Connection
    
- تكتب SQL
    
- تنفذ Query
    
- تقرأ ResultSet
    
- تحول البيانات لـ Objects
    
- تقفل Connection
    

وده كان:

- متعب
    
- مكرر
    
- مليان Bugs
    
- Boilerplate ضخم
    

---

# بعد ظهور Hibernate

الناس قالت:

> "ليه منحولش الـ Table لـ Object؟"

ومن هنا ظهر مفهوم:

# ORM

يعني:

## Object Relational Mapping

يعني:

|Database|Java|
|---|---|
|Table|Class|
|Row|Object|
|Column|Field|

مثال:

جدول:

```sql
users
```

يبقى:

```java
@Entity
public class User
```

---

# Hibernate بيعمل إيه؟

Hibernate هو الـ ORM Engine.

يعني هو اللي:

- يحول Object لـ SQL
    
- ويحّول SQL لـ Object
    

مثال:

```java
User user = new User();
user.setName("Ali");

entityManager.persist(user);
```

Hibernate يحولها:

```sql
INSERT INTO users(name) VALUES ('Ali');
```

---

# طب فين مشكلة Hibernate؟

رغم إنه ممتاز…

لكن لسه محتاج تكتب:

- DAO classes
    
- EntityManager
    
- Queries كتير
    

يعني كود كتير برضو.

---

# هنا ظهر Spring Data JPA

وده أهم جزء.

Spring قال:

> "إيه رأيكوا نولد الـ Repository تلقائي؟"

فبدل ما تعمل:

```java
public class UserRepositoryImpl {
    
    public User findById(Long id) {
        ...
    }

    public void save(User user) {
        ...
    }
}
```

تكتب بس:

```java
public interface UserRepository
       extends JpaRepository<User, Long> {
}
```

وسبرينج يعمل الباقي.

---

# يعني إيه JpaRepository<User, Long> ؟

دي Generic Interface.

```java
JpaRepository<EntityType, PrimaryKeyType>
```

يعني:

```java
JpaRepository<User, Long>
```

معناها:

- الـ Entity اسمها User
    
- الـ Primary Key نوعه Long
    

---

# أول ما التطبيق يشتغل

Spring يعمل الآتي:

## 1. يعمل Scan

يدور على:

```java
@Repository
```

أو أي Interface extends JpaRepository.

---

## 2. يلاقي UserRepository

```java
public interface UserRepository
       extends JpaRepository<User, Long>
```

---

## 3. يولد Class تلقائيًا

كأن Spring كتب:

```java
public class UserRepositoryImpl
       implements UserRepository
```

لكن أنت مش شايفه.

وده اسمه:

# Runtime Proxy

يعني Object متولد وقت التشغيل.

---

# الـ Proxy ده بيعمل إيه؟

فيه implementation لكل functions الجاهزة.

زي:

```java
save()
findById()
findAll()
delete()
count()
```

---

# مثال عملي

لما تكتب:

```java
userRepository.findById(1L);
```

اللي بيحصل داخليًا:

---

## Step 1

Spring يشوف:

```java
findById
```

---

## Step 2

يروح لـ Hibernate.

---

## Step 3

Hibernate يولد SQL:

```sql
SELECT *
FROM users
WHERE id = 1
```

---

## Step 4

JDBC Driver يبعت SQL للداتابيز.

---

## Step 5

الداتابيز ترجع البيانات.

---

## Step 6

Hibernate يحول الـ Row لـ Object.

يعني:

|SQL Row|Java Object|
|---|---|
|id=1,name=Ali|User object|

---

## Step 7

يرجعلك:

```java
Optional<User>
```

---

# طب save() بتشتغل إزاي؟

دي عبقرية Hibernate.

لما تعمل:

```java
userRepository.save(user);
```

Hibernate يسأل:

# هل الـ object جديد ولا موجود؟

---

## لو الـ ID = null

يبقى:

```sql
INSERT
```

---

## لو الـ ID موجود

يبقى:

```sql
UPDATE
```

---

# طب Spring عرف إزاي يكتب Queries من اسم الميثود؟

دي Feature اسمها:

# Query Method Derivation

يعني:  
Spring بيفهم الـ query من اسم الميثود.

---

# مثال

```java
findByEmail(String email)
```

Spring يحللها:

|جزء|معنى|
|---|---|
|find|SELECT|
|By|شرط|
|Email|column|

فيولد:

```sql
SELECT *
FROM users
WHERE email = ?
```

---

# مثال أصعب

```java
findByRoleAndActive(Role role, boolean active)
```

تتحول:

```sql
SELECT *
FROM users
WHERE role = ?
AND active = ?
```

---

# existsByEmail

```java
boolean existsByEmail(String email);
```

تتحول:

```sql
SELECT EXISTS(...)
```

---

# طب @Query دي ليه؟

لما الـ query تبقى معقدة.

مثال:

```java
@Query("""
SELECT u
FROM User u
WHERE u.createdAt > :date
""")
List<User> findNewUsers(LocalDateTime date);
```

هنا:  
أنت كتبت JPQL بنفسك.

---

# JPQL مش SQL

دي نقطة مهمة جدًا.

في SQL:

```sql
SELECT *
FROM users
```

لكن في JPQL:

```java
SELECT u
FROM User u
```

ليه؟

لأن JPQL بيتعامل مع:

- Classes
    
- Objects
    

مش Tables مباشرة.

---

# طب Service Layer دورها إيه؟

الـ Service:

- فيها Business Logic
    
- مش بتتعامل مع SQL
    

مثال:

```java
@Service
public class AuthService {

    private final UserRepository userRepository;

    public void register(RegisterRequest request) {

        if(userRepository.existsByEmail(request.getEmail())) {
            throw new BadRequestException("Email already exists");
        }

        User user = new User();

        userRepository.save(user);
    }
}
```

---

# ليه ممنوع الـ Controller يكلم Repository مباشرة؟

عشان نفصل المسؤوليات.

---

# Architecture الصح

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

---

# ليه التقسيم ده مهم؟

## Controller

مسؤول عن:

- Request
    
- Response
    

---

## Service

مسؤول عن:

- Business Logic
    

---

## Repository

مسؤول عن:

- Data Access
    

---

# Repository Pattern

الـ Repository Pattern فكرته:

> "خلّي الداتابيز مستخبية"

يعني الـ Service ميعرفش:

- PostgreSQL
    
- MySQL
    
- MongoDB
    

هو يعرف بس:

```java
userRepository.save(user)
```

---

# طب لو غيرت الداتابيز؟

الـ Service مش هيتأثر.

وده من أهم فوائد الـ abstraction.

---

# ملخص مهم جدًا

الـ Flow الحقيقي:

```text
Controller
    ↓
Service
    ↓
Repository Interface
    ↓
Spring Data JPA
    ↓
Hibernate
    ↓
JDBC
    ↓
Database
```

---

# الفرق بين الأجزاء

|Layer|مسؤوليته|
|---|---|
|Repository|Data Access|
|JPA|Abstraction|
|Hibernate|ORM Engine|
|JDBC|يكلم الداتابيز|
|Database|تخزين البيانات|

---

# أهم حاجة لازم تستوعبها

أنت لما تكتب:

```java
UserRepository extends JpaRepository
```

أنت مش بتكتب implementation…

أنت بتوصف لسبرينج:

> "أنا محتاج Repository للـ User"

وسبرينج يولدها كاملة لوحده وقت التشغيل.