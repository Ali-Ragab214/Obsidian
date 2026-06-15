بص يا علي، الكلاس ده فيه كذا نقطة مهمة جدًا، ودي من الحاجات اللي بتفرق بين واحد بيعرف JPA "تمشي معاه" وواحد فاهم JPA فعلًا.
```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "courses", uniqueConstraints = @UniqueConstraint(columnNames = "title", name = "uk_course_title"))
public class Course extends BaseEntity {

    @Column(nullable = false, unique = true)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Column(nullable = false)
    @DecimalMin(value = "0.0", inclusive = true, message = "Price cannot be negative")
    private BigDecimal price;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private CourseLevel level;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private CourseStatus status = CourseStatus.DRAFT;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "instructor_id", nullable = false)
    private User instructor;

    //avoid cascade.all {make it specific}
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL , orphanRemoval = true)
//    @Builder.Default
    private List<Section> sections = new ArrayList<>();

    //why default for one to many is lazy
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL)
    private List<Enrollment> enrollments = new ArrayList<>();

    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL)
    private List<Review> reviews = new ArrayList<>();
}
```

---

# أولًا: ليه الـ OneToMany default بتاعها LAZY ؟

عندك:

```java
@OneToMany(mappedBy = "course")
private List<Section> sections;
```

أنت سألت:

```java
// why default for one to many is lazy
```

الإجابة:

> لأن تحميل الـ Collections بيكون مكلف جدًا.

يعني تخيل عندك:

```text
Course
 └─ 20 Sections
      └─ كل Section فيها 50 Lessons
```

وعندك 100 كورس.

لو الـ OneToMany كانت EAGER افتراضيًا، مجرد ما تعمل:

```java
courseRepository.findAll();
```

هيحصل:

```text
100 Course
↓
2000 Sections
↓
100000 Lessons
```

كلهم يتحملوا في الذاكرة حتى لو أنت عايز عنوان الكورس بس.

وده كارثة.

عشان كده JPA قالت:

```text
ManyToOne   → EAGER (افتراضيًا)
OneToMany   → LAZY (افتراضيًا)
```

يعني:

```java
Course course = repo.findById(1L);
```

هيجيب:

```text
Course فقط
```

ولما تعمل:

```java
course.getSections();
```

ساعتها Hibernate يعمل Query تانية.

---

# طيب هل EAGER وحشة؟

غالبًا آه.

أنا شخصيًا في Spring Boot:

> 99% من الوقت بسيب الـ OneToMany LAZY.

بل وحتى الـ ManyToOne بحولها لـ LAZY يدويًا.

مثلاً:

```java
@ManyToOne(fetch = FetchType.LAZY)
private User instructor;
```

وده اللي أنت عامله فعلًا 👏.

---

# ثانيًا: المشكلة في الـ List دي

عندك:

```java
@OneToMany(mappedBy = "course",
           cascade = CascadeType.ALL,
           orphanRemoval = true)
private List<Section> sections = new ArrayList<>();
```

---

## هل دي غلط؟

لا.

بالعكس:

> دي أفضل من إنك تسيبها null.

---

## لو عملت كده:

```java
private List<Section> sections;
```

هيبقى:

```java
Course course = new Course();

course.getSections().add(section);
```

↓

```text
NullPointerException
```

لأن:

```java
sections == null
```

---

## لكن لما تعمل:

```java
private List<Section> sections = new ArrayList<>();
```

هيبقى:

```java
Course course = new Course();

course.getSections().add(section);
```

↓

شغال عادي.

---

# طب ليه كنت كاتب:

```java
@Builder.Default
```

ومتشال؟

هنا النقطة المهمة جدًا.

عندك:

```java
@Builder
public class Course {
    private List<Section> sections = new ArrayList<>();
}
```

---

# المشكلة مع Lombok Builder

لو عملت:

```java
Course course = Course.builder()
        .title("Spring")
        .build();
```

أنت ممكن تتوقع إن:

```java
sections = new ArrayList<>();
```

صح؟

للأسف...

> غلط.

---

Lombok Builder بيتجاهل الـ Initial Values.

يعني الناتج هيبقى:

```java
course.getSections()
```

↓

```text
null
```

مش ArrayList.

---

# الحل

تكتب:

```java
@Builder.Default
private List<Section> sections = new ArrayList<>();
```

ساعتها Lombok هيقول:

> "لو المستخدم مبعتش قيمة للـ sections، استخدم الـ ArrayList دي."

---

يعني:

## بدون Builder.Default

```java
Course course = Course.builder()
        .title("Spring")
        .build();

course.getSections(); // null
```

---

## مع Builder.Default

```java
Course course = Course.builder()
        .title("Spring")
        .build();

course.getSections(); // []
```

---

# يبقى عندك مشكلة فعلًا؟

عندك:

```java
@Builder
```

ومعندكش:

```java
@Builder.Default
```

على:

```java
sections
enrollments
reviews
```

فآه...

> دي مشكلة حقيقية.

---

# هل هتظهر دايمًا؟

لا.

لو كنت بتعمل:

```java
new Course();
```

مش هتظهر.

لأن Java Constructor هيشغل:

```java
= new ArrayList<>();
```

---

لكن لو بتعمل:

```java
Course.builder()
       .title(...)
       .build();
```

هتظهر.

---

# ثالثًا: هل Cascade.ALL مناسب؟

عندك:

```java
cascade = CascadeType.ALL
```

---

## Sections

أنا شايف إنه مناسب جدًا.

ليه؟

لأن:

```text
Course
 ↓
Sections
```

الـ Section ملهاش معنى من غير Course.

لو مسحت الكورس:

↓

تمسح السكاشن.

---

## orphanRemoval = true

دي ممتازة هنا.

يعني:

```java
course.getSections().remove(section);
```

↓

Hibernate يعمل:

```sql
DELETE FROM sections
WHERE id = ?
```

مش بس يفك العلاقة.

---

# لكن Enrollment ؟

عندك:

```java
@OneToMany(mappedBy = "course",
           cascade = CascadeType.ALL)
private List<Enrollment> enrollments;
```

هنا أنا مش مقتنع بـ ALL.

ليه؟

---

Enrollment يعتبر:

```text
Transaction History
```

أو سجل تسجيل الطلاب.

---

لو حد مسح الكورس:

هل عايز تمسح كل الـ Enrollments تلقائيًا؟

ممكن.

لكن في أنظمة كتير:

> لأ.

بيحتفظوا بالسجل.

---

فأنا غالبًا كنت هستخدم:

```java
cascade = {PERSIST, MERGE}
```

أو حتى بدون Cascade.

---

# Review ؟

عندك:

```java
@OneToMany(mappedBy = "course",
           cascade = CascadeType.ALL)
private List<Review> reviews;
```

ودي أكثر حاجة عليها جدل.

أنا شخصيًا:

> مش بحب Cascade.ALL هنا.

ليه؟

لأن الـ Review يعتبر User Generated Content.

وفي أنظمة كتير:

لو الكورس اتمسح:

↓

التقييمات تتحفظ للأرشفة.

---

# لو كنت Team Lead للمشروع ده كنت هعمل:

## Section

```java
@OneToMany(
    mappedBy = "course",
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
@Builder.Default
private List<Section> sections = new ArrayList<>();
```

لأنها Child حقيقي للكورس.

---

## Enrollment

```java
@OneToMany(mappedBy = "course")
@Builder.Default
private List<Enrollment> enrollments = new ArrayList<>();
```

بدون Cascade.ALL.

---

## Review

```java
@OneToMany(mappedBy = "course")
@Builder.Default
private List<Review> reviews = new ArrayList<>();
```

بدون Cascade.ALL.

---

# الخلاصة

- ✅ `OneToMany` افتراضيًا `LAZY` لأن تحميل الـ Collections مكلف جدًا.
    
- ✅ تهيئة الـ Lists بـ `new ArrayList<>()` فكرة صحيحة.
    
- ⚠️ مع `@Builder` لازم تضيف `@Builder.Default` وإلا الـ Lists هتبقى `null` عند استخدام الـ Builder.
    
- ✅ `Cascade.ALL + orphanRemoval=true` ممتازين مع `Section`.
    
- ⚠️ `Cascade.ALL` مع `Enrollment` و `Review` مش غلط، لكنه غالبًا مش أفضل اختيار من منظور تصميم الـ Domain.
    
- ولو سألك حد في إنترفيو: **"متى تستخدم orphanRemoval؟"** جاوب: _لما يكون الـ Child مملوك بالكامل للأب، وإزالته من الـ Collection تعني حذفه فعليًا من قاعدة البيانات._