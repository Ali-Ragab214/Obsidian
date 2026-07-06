# BaseEntity, Builder vs SuperBuilder, and JPA Auditing Notes

# 1. لماذا نستخدم BaseEntity؟

بدل ما نكرر نفس الحقول في كل Entity:

```java
private Long id;
private LocalDateTime createdAt;
private LocalDateTime updatedAt;
```

بنحطهم في Class مجردة (Abstract Class) اسمها BaseEntity.

```java
@MappedSuperclass
public abstract class BaseEntity {
    ...
}
```

وأي Entity ترث منها هتاخد الحقول دي تلقائيًا.

مثال:

```java
@Entity
public class Course extends BaseEntity {
    private String title;
}
```

هيتحول في قاعدة البيانات إلى:

```text
courses
--------
id
created_at
updated_at
title
```

بدون إنشاء جدول منفصل لـ BaseEntity.

---

# 2. ما معنى @MappedSuperclass ؟

```java
@MappedSuperclass
```

تعني:

> "هذه الفئة ليست Entity مستقلة، ولكن الحقول الموجودة بها يتم توريثها إلى الـ Entities الأبناء."

أي:

✓ لا يتم إنشاء جدول باسم BaseEntity. =>  مهمهههههههههه

✓ يتم نسخ الحقول إلى الجداول الأبناء.

مثال:

```java
Course extends BaseEntity
Lesson extends BaseEntity
Review extends BaseEntity
```

كل جدول منهم يحتوي على:

```text
id
created_at
updated_at
```

---

# 3. Builder vs SuperBuilder

## Builder

Lombok:

```java
@Builder
```

تقوم بإنشاء Builder للحقول الموجودة داخل الكلاس فقط.

مثال:

```java
@Builder
public class Course extends BaseEntity {

    private String title;

}
```

الاستخدام:

```java
Course.builder()
       .title("Spring")
       .build();
```

المشكلة:

Builder لا يرى حقول BaseEntity.

أي لا يمكنك كتابة:

```java
Course.builder()
       .id(1L)           // Error
       .createdAt(...)
       .title("Spring")
       .build();
```

لأن id و createdAt موجودين في BaseEntity.

---

## SuperBuilder

Lombok:

```java
@SuperBuilder
```

مصمم خصيصًا للـ Inheritance.

يستطيع الوصول إلى حقول الأب والابن.

مثال:

BaseEntity:

```java
@SuperBuilder
public abstract class BaseEntity {
    private Long id;
}
```

Course:

```java
@Entity
@SuperBuilder
public class Course extends BaseEntity {

    private String title;

}
```

الاستخدام:

```java
Course course = Course.builder()
        .id(1L)
        .title("Spring")
        .build();
```

هنا Builder يرى:

✓ حقول Course

✓ حقول BaseEntity

---

# 4. أيهما أفضل؟

## إذا لم تستخدم الوراثة

استخدم:

```java
@Builder
```

لأنه أبسط.

---

## إذا كان لديك Inheritance

مثل:

```text
BaseEntity
    ↓
Course
Lesson
Review
```

فالأفضل:

```java
@SuperBuilder
```

لأنه يدعم الوراثة بشكل صحيح.

---

# 5. هل وجود @Builder عندك يعتبر خطأ؟

ليس خطأ.

لأنك غالبًا لا تقوم ببناء:

```java
id
createdAt
updatedAt
```

يدويًا.

Hibernate هو من يتعامل معهم.

لذلك كثير من المشاريع تترك:

```java
@Builder
```

في الـ Entities.

---

# 6. متى أستخدم SuperBuilder فعلاً؟

إذا كنت تحتاج أثناء الاختبارات أو الـ Fixtures أو إنشاء Objects يدويًا إلى بناء خصائص BaseEntity.

مثال:

```java
Course.builder()
       .id(10L)
       .createdAt(...)
       ...
```

فوقتها استخدم:

```java
@SuperBuilder
```

---

# 7. ما هو JPA Auditing ؟

JPA Auditing هو نظام يجعل Spring يقوم بتعبئة حقول التتبع تلقائيًا.

مثل:

```java
createdAt
updatedAt
createdBy
updatedBy
```

بدون أن تكتب الكود بنفسك.

---

# 8. لماذا نستخدمه؟

بدل:

```java
course.setCreatedAt(LocalDateTime.now());

course.setUpdatedAt(LocalDateTime.now());
```

في كل Service.

Spring يقوم بذلك تلقائيًا.

---

# 9. كيف يعمل؟

أنت كتبت:

```java
@EntityListeners(AuditingEntityListener.class)
```

وهذا يعني:

> "استدعِ AuditingEntityListener قبل وبعد عمليات الحفظ والتحديث."

---

# 10. ما هو AuditingEntityListener ؟

هو Listener من Spring Data JPA.

يراقب أحداث الـ Entity Lifecycle.

مثل:

```text
قبل الحفظ (Persist)

قبل التعديل (Update)
```

ثم يملأ الحقول المناسبة.

---

# 11. ما هو دور @CreatedDate ؟

```java
@CreatedDate
private LocalDateTime createdAt;
```

تعني:

> "املأ هذا الحقل عند أول عملية Insert فقط."

مثال:

عند:

```java
courseRepository.save(course);
```

إذا كان الكورس جديدًا:

```text
createdAt = 2026-06-13 15:00
```

ولا يتغير بعد ذلك.

---

# 12. ما هو دور @LastModifiedDate ؟

```java
@LastModifiedDate
private LocalDateTime updatedAt;
```

تعني:

> "حدّث هذا الحقل في كل عملية Update."

مثال:

إنشاء:

```text
createdAt = 15:00
updatedAt = 15:00
```

بعد تعديل الكورس:

```text
createdAt = 15:00
updatedAt = 18:30
```

---

# 13. لماذا نضع:

```java
@Column(updatable = false)
```

على createdAt ؟

لأن تاريخ الإنشاء لا يجب أن يتغير.

أي:

```text
Insert → مسموح

Update → ممنوع
```

---

# 14. لماذا nullable = false ؟

لأن هذه الحقول يجب أن تحتوي دائمًا على قيمة.

```text
createdAt ≠ null

updatedAt ≠ null
```

---

# 15. هل يكفي وضع هذه Annotations فقط؟

لا.

يجب تفعيل Auditing في المشروع.

مثال:

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

بدونها:

```text
@CreatedDate
@LastModifiedDate
```

لن يعملوا.

---

# 16. @CreatedDate vs @CreationTimestamp

## @CreatedDate

من:

```text
Spring Data JPA
```

يعتمد على:

```text
JPA Auditing
```

مرن أكثر.

يمكن استخدام:

```text
createdBy
updatedBy
```

معه.

---

## @CreationTimestamp

من:

```text
Hibernate
```

يقوم Hibernate فقط بوضع وقت الإنشاء.

مثال:

```java
@CreationTimestamp
private LocalDateTime createdAt;
```

ولا يحتاج Auditing.

---

# 17. أيهما أفضل؟

في مشاريع Spring Boot الاحترافية:

يفضل:

```java
@CreatedDate
@LastModifiedDate
```

مع:

```java
@EnableJpaAuditing
```

لأنه يوفر نظام Auditing كامل.

أما:

```java
@CreationTimestamp
```

فهو مناسب للحالات البسيطة فقط.

---

# الخلاصة

- BaseEntity تمنع تكرار الحقول المشتركة.
    
- @MappedSuperclass تورث الحقول دون إنشاء جدول مستقل.
    
- @Builder لا يدعم الوراثة.
    
- @SuperBuilder يدعم الوراثة.
    
- في حالتك الحالية، استخدام Builder ليس خطأ لأن Hibernate يدير حقول BaseEntity.
    
- JPA Auditing يجعل Spring يملأ createdAt و updatedAt تلقائيًا.
    
- @EntityListeners(AuditingEntityListener.class) تربط الـ Entity بنظام الـ Auditing.
    
- @CreatedDate تُملأ عند الإنشاء فقط.
    
- @LastModifiedDate تتحدث مع كل تعديل.
    
- يجب تفعيل Auditing باستخدام @EnableJpaAuditing.
    
- في مشاريع Spring الاحترافية، يفضل استخدام Auditing على CreationTimestamp.