الكلاس ده من أهم الحاجات في JPA Auditing   
== a class in infra layer 
وهو مسؤول عن إن Hibernate / Spring يملوا بيانات الـ auditing تلقائيًا.

يعني إيه Auditing أصلًا؟

يعني:

# تتبع البيانات تلقائيًا

زي:

- إمتى الـ record اتعمل
    
- إمتى اتعدل
    
- مين اللي عدله (ممكن بعدين)
    

---

# الكلاس بتاعك

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

خليني أفصلهالك سطر سطر 👇

---

# أول Annotation

```java
@Configuration
```

دي بتقول لـ Spring:

# "الكلاس ده Config Class"

يعني:

- فيه إعدادات
    
- فيه Beans
    
- فيه Configurations
    

زي:

- SecurityConfig
    
- SwaggerConfig
    
- JpaConfig
    

كلهم Config classes.

---

# أهم Annotation هنا 🔥

```java
@EnableJpaAuditing
```

دي اللي بتشغل الـ Auditing System كله.

يعني Spring يقول:

# "ابدأ راقب الـ Entities"

---

# طب بيعمل إيه فعليًا؟

لما تحط:

```java
@EnableJpaAuditing
```

Spring يبدأ يشتغل مع annotations زي:

```java
@CreatedDate
@LastModifiedDate
```

---

# مثال عملي

## BaseEntity

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

---

# اللي بيحصل وقت التشغيل

لما تعمل:

```java
userRepository.save(user);
```

Spring تلقائيًا يعمل:

```java
createdAt = now()
updatedAt = now()
```

من غير ما تكتبهم بنفسك 🔥

---

# ولما تعمل Update

```java
user.setName("Ali");
userRepository.save(user);
```

Spring يحدث:

```java
updatedAt = now()
```

تلقائي.

---

# من غير EnableJpaAuditing ؟

الحقول دي:

```java
@CreatedDate
@LastModifiedDate
```

مش هتشتغل.

وهتفضل:

- null
    
- أو لازم تملأها يدويًا
    

---

# العلاقة بين الحاجات دي

```text
@EnableJpaAuditing
        ↓
يفعل Auditing System
        ↓
@EntityListeners(AuditingEntityListener.class)
        ↓
يسمع لأي Save / Update
        ↓
@CreatedDate / @LastModifiedDate
        ↓
يملأ القيم تلقائيًا
```

---

# ليه دي Best Practice؟

بدل ما تعمل:

```java
user.setCreatedAt(LocalDateTime.now());
```

في كل Service…

Spring يعملها لوحده.

---

# كمان مهم جدًا

أنت عامل:

```java
@MappedSuperclass
```

في BaseEntity غالبًا.

وده معناه:

أي Entity يعمل inherit من BaseEntity:

```java
public class User extends BaseEntity
```

ياخد:

- id
    
- createdAt
    
- updatedAt
    

تلقائي.

---

# النتيجة

بدل ما تكرر:

```java
private Long id;
private LocalDateTime createdAt;
private LocalDateTime updatedAt;
```

في كل Entity…

تكتبهم مرة واحدة فقط 🔥

---

# هل JpaConfig لازم يبقى موجود؟

مش شرط.

تقدر تحط:

```java
@EnableJpaAuditing
```

في Main Class.

لكن:

# الأفضل تحطه في Config class منفصلة

عشان:

- التنظيم
    
- scalability
    
- clean architecture
    

---

# بعدين ممكن تضيف حاجات أقوى جدًا هنا

زي:

## مين عدل الـ record

```java
@CreatedBy
@LastModifiedBy
```

مثلاً:

- Admin
    
- Instructor
    
- User ID
    

وده بيتستخدم في الأنظمة الكبيرة جدًا.