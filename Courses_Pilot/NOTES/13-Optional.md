سؤال ممتاز، ودي من الحاجات اللي بتفرق بين مجرد كتابة كود وبين فهم Spring Data JPA فعلاً.

حالياً أنت عامل:

```java
Section findByTitleAndCourseId(String title, Long courseId);
```

فـ Spring Data هترجع:

- Entity لو لقتها
    
- `null` لو ملقتهاش
    

عشان كده كتبت:

```java
var section = sectionRepository.findByTitleAndCourseId(title, courseId);

if (section == null) {
    throw new ResourceNotFoundException(...);
}
```

وده صحيح 100%.

---

### إمتى أستخدم Optional؟

لما أكون متوقع إن النتيجة ممكن متبقاش موجودة.

بدل:

```java
Section findByTitleAndCourseId(String title, Long courseId);
```

اكتب:

```java
Optional<Section> findByTitleAndCourseId(String title, Long courseId);
```

وساعتها:

```java
return sectionRepository
        .findByTitleAndCourseId(title, courseId)
        .map(sectionMapper::toResponse)
        .orElseThrow(() ->
                new ResourceNotFoundException(
                        "Section not found with title: "
                                + title
                                + " and courseId: "
                                + courseId
                )
        );
```

---

### ليه Optional أحسن؟

لأن الـ Method Signature نفسها بتقول:

> "ممكن أرجع Section وممكن مرجعش حاجة"

بدل ما ترجع `null` وتسيب اللي هيستخدمها يخمن.

---

### إمتى أرجع Entity عادي؟

لو منطقياً المفروض القيمة موجودة دائماً.

مثال:

```java
boolean existsByEmail(String email);
```

أو

```java
long countByCourseId(Long courseId);
```

مفيش Optional هنا.

---

### في حالة findById؟

أكيد:

```java
Optional<Course> findById(Long id);
```

عشان Spring Data نفسها عاملاها كده.

لذلك بنكتب:

```java
Course course = courseRepository
        .findById(id)
        .orElseThrow(() ->
                new ResourceNotFoundException("Course not found"));
```

---

### في مشروعك الحالي أنصح بإيه؟

في Repository:

```java
Optional<Section> findByTitleAndCourseId(
        String title,
        Long courseId
);
```

وفي Service:

```java
@Transactional(readOnly = true)
@Override
public SectionResponse getSectionByTitleAndCourseId(
        String title,
        Long courseId
) {

    Section section = sectionRepository
            .findByTitleAndCourseId(title, courseId)
            .orElseThrow(() ->
                    new ResourceNotFoundException(
                            "Section not found with title: "
                                    + title
                                    + " and courseId: "
                                    + courseId
                    ));

    return sectionMapper.toResponse(section);
}
```

وده الشكل اللي هتلاقيه في أغلب مشاريع Spring Boot الاحترافية. استخدام `Optional` في الـ Repository methods اللي ممكن ترجع "لا شيء" بقى شبه Standard Practice.