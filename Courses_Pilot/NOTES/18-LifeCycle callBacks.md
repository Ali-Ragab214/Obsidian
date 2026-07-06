بص يا علي، اللي في الصورة ده من الحاجات المهمة جدًا في JPA وHibernate، وكثير من الناس بتستخدمها من غير ما تفهم هي بتحصل إمتى بالضبط.

أنت كنت عامل الأول كده:

```java
@Column(nullable = false)
@Min(value = 0, message = "Progress percentage cannot be negative")
@Max(value = 100, message = "Progress percentage cannot exceed 100")
private Integer progressPercentage = 0;
```

وبعدين غيرتها لـ:

```java
@Column(nullable = false)
private Integer progressPercentage = 0;

@PrePersist
@PreUpdate
private void validateProgress() {
    if (progressPercentage == null ||
        progressPercentage < 0 ||
        progressPercentage > 100) {

        throw new IllegalArgumentException(
            "Progress percentage must be between 0 and 100");
    }
}
```

خلينا نفهم يعني إيه `@PrePersist` و `@PreUpdate`.

---

# ما هي Lifecycle Callbacks؟

الـ Entity في JPA ليها دورة حياة (Lifecycle).

يعني الكائن بيمر بمراحل مختلفة أثناء التعامل مع قاعدة البيانات.

مثلاً:

```text
إنشاء Entity
    ↓
Persist (Insert)
    ↓
Update
    ↓
Delete
```

JPA بتسمح لك تشغل ميثود معينة في لحظات معينة من دورة الحياة دي.

---

# أشهر Lifecycle Callbacks 

```java
@PrePersist
@PostPersist

@PreUpdate
@PostUpdate

@PreRemove
@PostRemove

@PostLoad
```

---

# ما معنى @PrePersist ؟

> **Method that executes before the entity is inserted into the database.**

بالعربي:

> ميثود يتم تنفيذها قبل عمل Insert للـ Entity في قاعدة البيانات.

---

## مثال

```java
@Entity
public class User {

    private LocalDateTime createdAt;

    @PrePersist
    public void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
```

لما تعمل:

```java
userRepository.save(user);
```

أول مرة فقط يحدث:

```text
@PrePersist
    ↓
INSERT INTO users ...
```

---

## النتيجة

بدل ما تعمل:

```java
user.setCreatedAt(LocalDateTime.now());
```

كل مرة،

JPA هتعملها تلقائيًا.

---

# متى تعمل @PrePersist ؟

لو عندك:

```java
User user = new User();

repository.save(user);
```

وكان الـ User جديدًا (id = null غالبًا):

```text
PrePersist
↓
Insert
↓
PostPersist
```

---

# ما معنى @PreUpdate ؟

> **Method that executes before the entity is updated in the database.**

بالعربي:

> ميثود يتم تنفيذها قبل تحديث البيانات في قاعدة البيانات.

---

## مثال

```java
@Entity
public class User {

    private LocalDateTime updatedAt;

    @PreUpdate
    public void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

لو عملت:

```java
user.setName("Ali");

repository.save(user);
```

وكان الـ User موجود بالفعل:

```text
PreUpdate
↓
UPDATE users ...
↓
PostUpdate
```

---

# في المثال الموجود عندك

```java
@PrePersist
@PreUpdate
private void validateProgress() {

    if (progressPercentage == null ||
        progressPercentage < 0 ||
        progressPercentage > 100) {

        throw new IllegalArgumentException(
            "Progress percentage must be between 0 and 100");
    }
}
```

يعني:

> قبل ما الـ Enrollment يتعمله Insert أو Update، اتأكد إن الـ progress بين 0 و100.

---

## لو حصل:

```java
enrollment.setProgressPercentage(120);

repository.save(enrollment);
```

هيحدث:

```text
save()
↓
PreUpdate أو PrePersist
↓
validateProgress()
↓
IllegalArgumentException
↓
لن يتم تنفيذ SQL
```

---

# الفرق بينها وبين @Min و @Max

## @Min و @Max

دي Bean Validation.

```java
@Min(0)
@Max(100)
private Integer progressPercentage;
```

بتحتاج Validation يحصل.

غالبًا عن طريق:

```java
@Valid
```

مثلاً:

```java
public ResponseEntity<?> create(
        @Valid @RequestBody Request request)
```

---

## @PrePersist و @PreUpdate

دي JPA Lifecycle Callbacks.

تشتغل تلقائيًا عند الحفظ أو التحديث.

حتى لو مفيش:

```java
@Valid
```

---

# مثال يوضح الفرق

لو عملت:

```java
Enrollment enrollment = new Enrollment();
enrollment.setProgressPercentage(150);

repository.save(enrollment);
```

### باستخدام @Min/@Max فقط

قد يتم الحفظ إذا لم يتم تشغيل Bean Validation.

---

### باستخدام @PrePersist/@PreUpdate

```text
save()
↓
validateProgress()
↓
Exception
↓
لن يتم الحفظ
```

---

# استخدامات شائعة جدًا

## 1) إنشاء تاريخ الإنشاء

```java
@PrePersist
void prePersist() {
    createdAt = LocalDateTime.now();
}
```

---

## 2) تحديث تاريخ آخر تعديل

```java
@PreUpdate
void preUpdate() {
    updatedAt = LocalDateTime.now();
}
```

---

## 3) تنظيف البيانات

مثلاً:

```java
@PrePersist
@PreUpdate
void normalizeEmail() {
    email = email.trim().toLowerCase();
}
```

---

## 4) التحقق من قواعد الـ Domain

مثل حالتك:

```java
if (progress < 0 || progress > 100)
    throw new IllegalArgumentException();
```

---

# هل من الجيد استخدامها للـ Validation؟

الإجابة:

### نعم، ولكن بحذر.

- لو كانت قواعد تخص الـ Domain نفسه (Business Invariants) → ممتاز.
    
- لو كانت Validation خاصة بطلبات الـ API → الأفضل Bean Validation (`@Valid`).
    

مثلاً:

```text
Email format
Password length
Request validation
```

يفضل لها:

```java
@NotBlank
@Email
@Size
```

أما:

```text
رصيد الحساب لا يكون سالبًا
نسبة التقدم بين 0 و100
سعر المنتج لا يكون أقل من صفر
```

فمناسب جدًا وضعها داخل الـ Entity باستخدام Lifecycle Callbacks.

---

# سؤال إنترفيو

### What is `@PrePersist`?

> It is a JPA lifecycle callback method that is executed before an entity is inserted into the database.

---

### What is `@PreUpdate`?

> It is a JPA lifecycle callback method that is executed before an existing entity is updated in the database.

---

### Why use them?

> To perform automatic operations such as validation, setting timestamps, or preparing entity data before persistence operations.

---

# ملخص سريع للحفظ

> `@PrePersist` تعمل قبل تنفيذ الـ `INSERT`.

> `@PreUpdate` تعمل قبل تنفيذ الـ `UPDATE`.

> الاثنان جزء من **JPA Entity Lifecycle Callbacks**، وتُستخدم لتنفيذ منطق معين تلقائيًا مثل الـ validation، ضبط التواريخ، أو تجهيز البيانات قبل حفظها في قاعدة البيانات.