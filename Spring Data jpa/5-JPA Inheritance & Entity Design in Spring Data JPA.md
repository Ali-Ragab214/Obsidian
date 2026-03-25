
important !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
تمام يا علي 🔥 الجزء ده مهم جدًا وبيظهر في المشاريع الكبيرة لما يكون عندك **Entities ليها صفات مشتركة**

هديك شرح منظم + احترافي 👇

---

# 🧩 العنوان الشامل

## 🚀 **JPA Inheritance & Entity Design in Spring Data JPA**

---

# 🧠 أولًا: JPA Inheritance Overview

## ✅ يعني إيه Inheritance في JPA؟

> إنك تعمل كلاس أب (Parent) فيه خصائص مشتركة  
> وبعدين كلاسز تانية ترث منه

---

## 🔥 مثال بسيط

```java
public class Person {
    private String name;
    private String email;
}
```

```java
public class Customer extends Person {
    private String address;
}
```

```java
public class Worker extends Person {
    private String job;
}
```

💥 بدل ما تكرر `name + email` في كل كلاس

---

## 🎯 في JPA الموضوع بيتربط بالداتابيز

Hibernate (Hibernate) لازم يعرف:

> "أحول الوراثة دي لجدول إزاي؟"

---

# ⚖️ ثانيًا: Inheritance vs Composition

## 🟢 Inheritance (وراثة)

✔ تستخدمها لما:

- الكلاسات ليها علاقة "is-a"
    

👉 مثال:

- Worker **is a** Person
    

---

## 🔵 Composition (تركيب)

✔ تستخدمها لما:

- الكلاس "has-a"
    

👉 مثال:

- Order **has a** User
    

```java
class Order {
    private User user;
}
```

---

## 🧠 القاعدة الذهبية

> استخدم Composition أكتر من Inheritance  
> إلا لو العلاقة منطقية جدًا

---

# 🗄️ ثالثًا: Database Class Diagram (الفكرة)

لما تعمل Inheritance عندك 3 طرق رئيسية:

---

## 1️⃣ SINGLE_TABLE

كل الكلاسات في جدول واحد

|id|name|type|address|job|
|---|---|---|---|---|

✔ سريع  
❌ فيه null كتير

---

## 2️⃣ JOINED

كل كلاس ليه جدول

Person  
Customer  
Worker

✔ منظم  
❌ فيه joins

---

## 3️⃣ TABLE_PER_CLASS

كل كلاس جدول مستقل

✔ بسيط  
❌ performance وحش

---

# 🟡 رابعًا: MappedSuperclass (مهم جدًا 🔥)

## ✅ يعني إيه؟

> كلاس فيه خصائص مشتركة  
> ❌ مش Entity  
> ❌ ملوش جدول

---

## 🔥 مثال

```java
import jakarta.persistence.*;
import lombok.*;

@MappedSuperclass
@Getter
@Setter
public abstract class BaseEntity {

    @Id
    @GeneratedValue
    private Integer id;

    private String createdAt;
}
```

---

## 🎯 لما نستخدمه؟

لما عايز:

- تشارك fields بين entities
    
- من غير ما تعمل جدول ليه
    

---

## ❗ مهم جدًا

❌ مينفعش تعمل query عليه  
❌ ملوش repository

---

# 🟠 خامسًا: Create Child Classes

## 🔥 مثال عملي

```java
@Entity
public class Customer extends BaseEntity {

    private String address;
}
```

---

```java
@Entity
public class Worker extends BaseEntity {

    private String job;
}
```

---

## 🎯 النتيجة في الداتابيز

Customer table:

- id
    
- createdAt
    
- address
    

Worker table:

- id
    
- createdAt
    
- job
    

💥 كلهم ورثوا من BaseEntity

---

# 🔥 إمتى أستخدم MappedSuperclass؟

✔ لما عندك:

- id
    
- timestamps
    
- audit fields
    

---

# 🧠 الفرق بين MappedSuperclass و Entity Inheritance

|Feature|MappedSuperclass|Entity Inheritance|
|---|---|---|
|جدول|❌ لا|✔ نعم|
|Query عليه|❌ لا|✔ نعم|
|الهدف|مشاركة properties|تمثيل hierarchy|

---

# 🚀 سيناريو من مشروعك (مهم ليك 🔥)

في مشروع الخدمات بتاعك:

## اعمل:

```java
@MappedSuperclass
class BaseEntity {
    id
    createdAt
}
```

وبعدين:

- Customer
    
- Worker
    
- Service
    
- Booking
    

كلهم يرثوا منه 💥

---

# 💡 الخلاصة

- Inheritance → لتقليل التكرار
    
- Composition → الأفضل غالبًا
    
- عندك 3 استراتيجيات للداتابيز
    
- MappedSuperclass → مشاركة fields بدون جدول
    
- Child classes ترث عادي جدًا
    

---

# 🔥 الخطوة الجاية (مهم جدًا)

لو عايز تبقى تقيل بجد 👇

👉 أشرحلك:

- `@Inheritance(strategy = …)` بالتفصيل
    
- الفرق بين `@MappedSuperclass` و `@Entity` عملي
    
- أو نطبق كل ده على مشروعك الحقيقي
    

قولّي نكمل على إيه 💪