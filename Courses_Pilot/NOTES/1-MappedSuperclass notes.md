
---
@
# أولاً:

## @EntityListeners(AuditingEntityListener.class)

ده مرتبط بالـ **Auditing** في Spring Data JPA.

### يعني اي Auditing أصلاً؟

Auditing = النظام يملأ بيانات تلقائياً بدون ما تكتب كود كل مرة.

زي مثلاً:

- createdAt → يتكتب لوحده وقت الإنشاء
    
- updatedAt → يتحدث لوحده وقت التعديل
    
- createdBy → مين عمل الإنشاء
    
- lastModifiedBy → مين عدّل
    

بدل ما تعمل كده يدوي ❌  
Spring يعملها أوتوماتيك ✔️

---

## طيب الـ Auditing بيشتغل إزاي؟

Spring عنده Listener جاهز اسمه:

```java
AuditingEntityListener
```

ده عبارة عن كلاس **بيراقب lifecycle بتاع الـ entity**:

- قبل الحفظ
    
- قبل التحديث
    

ويملأ الحقول تلقائي.

### Lifecycle يعني اي؟

يعني لما يحصل:

```
persist -> insert
update  -> update
```

الـ listener يتدخل قبل العملية ويضيف القيم.

---

## ليه بنحطه في BaseEntity؟

عشان كل الـ Entities ترثه تلقائي 👇

لو حطيته في User بس:

- Course مش هيشتغل فيه auditing ❌
    

لكن لما نحطه في BaseEntity:

- كل Entity يرث → يبقى auditing شغال على كل الجداول ✔️
    

---

## مثال كامل

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### السيناريو اللي بيحصل فعلياً:

عند حفظ كورس جديد:

```java
courseRepository.save(course);
```

Spring يعمل داخلياً:

```
قبل الحفظ:
createdAt = now()
updatedAt = now()
```

بدون أي كود منك 😍

---

## شرط مهم جداً لتشغيل Auditing

لازم تفعل الـ Auditing في المشروع:

```java
@EnableJpaAuditing
@Configuration
public class JpaConfig {}
```

لو نسيت السطر ده → الـ annotations مش هتشتغل.

---

# ثانياً:

## @GeneratedValue(strategy = GenerationType.IDENTITY)

ده خاص بالـ **Primary Key generation**.

يعني مين هيولد الـ ID ؟

عندنا 4 استراتيجيات في JPA:

|Strategy|مين يولد الـ ID|
|---|---|
|AUTO|JPA يختار|
|IDENTITY|Database|
|SEQUENCE|Database Sequence|
|TABLE|جدول خاص|

---

## احنا ليه اخترنا IDENTITY؟

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

معناه:

> خلّي **قاعدة البيانات** هي اللي تولد الـ id تلقائي

زي auto increment.

---

## مثال SQL اللي بيحصل

في MySQL / Postgres:

```sql
id BIGINT AUTO_INCREMENT PRIMARY KEY
```

لما تعمل insert:

```sql
INSERT INTO users (name,email) VALUES (...)
```

قاعدة البيانات تعمل:

```
id = 1
id = 2
id = 3
```

بدون ما تبعته في الـ insert.

---

## ليه نحطه في BaseEntity؟

كل الجداول عندها id  
يبقى نحطه مرة واحدة بدل التكرار ✔️

---

## ليه IDENTITY مناسب لمشروعنا؟

ممتاز مع:

- MySQL
    
- PostgreSQL
    
- مشاريع REST API
    
- Microservices
    

وسهل ومباشر.

---

# خلاصة الصورة الكبيرة 🧠

## EntityListeners

يشغّل الـ auditing التلقائي لكل الـ entities.

## GeneratedValue IDENTITY

يخلي قاعدة البيانات تولد الـ id تلقائي.

---

# سيناريو كامل يحصل فعلياً

```java
User user = new User("Ali");
userRepository.save(user);
```

اللي يحصل خلف الكواليس:

1️⃣ DB تولد id  
2️⃣ Spring يملأ createdAt  
3️⃣ Spring يملأ updatedAt

وكل ده بدون كود إضافي منك 🔥

---

لو حابب الخطوة الجاية نشرح  
`@CreatedBy / @LastModifiedBy` وربطهم بالـ Security 👀 دي نقلة احترافية جداً.