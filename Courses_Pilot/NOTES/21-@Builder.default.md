ممتاز إنك سألت، لأن `@Builder.Default` من الحاجات اللي ناس كتير بتحطها من غير ما تفهم هي بتحل إيه.

---

## الأول: مشكلة Lombok Builder

افترض عندك Entity:

```java
@Builder
public class User {

    private String name;

    private boolean active = true;
}
```

أنت متوقع إن:

```java
User user = User.builder()
        .name("Ali")
        .build();
```

يبقى:

```java
active = true
```

صح؟

❌ غلط.

لأن لما تستخدم `@Builder`، Lombok بيتجاهل القيمة الافتراضية اللي في الـ field.

فيبقى:

```java
active = false
```

لأن الـ boolean default بتاع Java هو false.

---

## الحل

```java
@Builder.Default
private boolean active = true;
```

دلوقتي:

```java
User user = User.builder()
        .name("Ali")
        .build();
```

هيبقى:

```java
active = true
```

---

## مثال أوضح

```java
@Builder
public class Course {

    private String title;

    @Builder.Default
    private Integer studentsCount = 0;
}
```

لما تعمل:

```java
Course course = Course.builder()
        .title("Java")
        .build();
```

النتيجة:

```java
title = Java
studentsCount = 0
```

بدون `@Builder.Default` كانت هتبقى:

```java
studentsCount = null
```

---

# ليه الـ Agent حطها على معظم الـ Attributes؟

غالبًا كان عندك حاجة زي:

```java
private Boolean isActive = true;

private Integer progress = 0;

private Integer totalLessons = 0;

private String role = "STUDENT";

private Set<Skill> skills = new HashSet<>();
```

ولقي إنك بتستخدم:

```java
@Builder
```

فقال:

> "لو حد أنشأ Object بالـ Builder القيم الافتراضية دي ممكن تضيع."

فحوّلها إلى:

```java
@Builder.Default
private Boolean isActive = true;

@Builder.Default
private Integer progress = 0;

@Builder.Default
private String role = "STUDENT";

@Builder.Default
private Set<Skill> skills = new HashSet<>();
```

---

# هل أحطها على كل Field؟

❌ لا.

خطأ شائع جدًا.

مثال:

```java
private String name;

private String email;
```

مالهاش معنى:

```java
@Builder.Default
private String name;
```

لأن مفيش Default Value أصلاً.

---

# إمتى أستخدمها؟

لما يكون عندك قيمة افتراضية حقيقية:

✅

```java
@Builder.Default
private boolean enabled = true;

@Builder.Default
private int progress = 0;

@Builder.Default
private Role role = Role.STUDENT;

@Builder.Default
private List<String> tags = new ArrayList<>();
```

❌

```java
@Builder.Default
private String name;

@Builder.Default
private String email;
```

---

# في الـ Request DTO

مثال:

```java
@Builder
public record CreateCourseRequest(
    String title,
    Integer duration
) {}
```

هنا أصلًا مفيش داعي لـ `@Builder.Default`.

لأن الـ Client هو اللي هيبعت القيم.

---

# بالنسبة للـ Entity في مشاريع Spring

أكثر الأماكن اللي أستخدم فيها `@Builder.Default` هي:

```java
@Builder.Default
private Boolean active = true;

@Builder.Default
private Integer progress = 0;

@Builder.Default
private LocalDateTime createdAt = LocalDateTime.now();

@Builder.Default
private Set<Role> roles = new HashSet<>();

@Builder.Default
private List<Lesson> lessons = new ArrayList<>();
```

---

لو تبعتلي Entity من مشروع الـ Job Board أو Skill Hub اللي الـ Agent عدلها، أقولك **كل `@Builder.Default` منها صح ولا زيادة ومش محتاجها**.