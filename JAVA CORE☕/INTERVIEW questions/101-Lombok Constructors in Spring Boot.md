# Lombok Constructors in Spring Boot: `@RequiredArgsConstructor` vs `@NoArgsConstructor` vs `@AllArgsConstructor`

بص يا علي، كلامك صح بنسبة كبيرة، بس محتاجين نفهم السبب مش نحفظ إن كل نوع Class بنستخدم معاه Annotation معينة.

## 1. الأول: كل Annotation بتعمل إيه؟

الـ Lombok مكتبة بتخلّصك من كتابة الـ Boilerplate Code، زي الـ Constructors والـ Getters والـ Setters.

|Annotation|بتعمل إيه؟|
|---|---|
|`@RequiredArgsConstructor`|بتعمل Constructor للـ `final` fields والـ `@NonNull` fields اللي محتاجة initialization.|
|`@NoArgsConstructor`|بتعمل Constructor من غير أي Parameters.|
|`@AllArgsConstructor`|بتعمل Constructor فيه Parameters لكل الـ fields.|

خلينا نشوف مثال بسيط.

## 2. ليه بنستخدم `@RequiredArgsConstructor` مع الـ Service والـ Controller؟

تخيّل عندنا `CourseService` محتاجة `CourseRepository` عشان تتعامل مع قاعدة البيانات.

```

@Service
@RequiredArgsConstructor
public class CourseService {

    private final CourseRepository courseRepository;

    public void getCourses() {
        courseRepository.findAll();
    }
}
```

إيه اللي بيحصل هنا؟

1. كتبنا `@Service` عشان Spring تتعرف على الكلاس باعتباره Service Bean.
    
2. كتبنا `private final CourseRepository`، يعني لازم الـ dependency دي تتحدد عند إنشاء الـ object ومينفعش نغيّر الـ reference بعدها.
    
3. كتبنا `@RequiredArgsConstructor`، فـ Lombok بتولّد الـ Constructor ده تلقائيًا:
    

```

public CourseService(CourseRepository courseRepository) {
    this.courseRepository = courseRepository;
}
```

كأننا كتبنا الـ Constructor بإيدينا بالضبط.

### طب Spring بتعمل إيه بالـ Constructor ده؟

Spring Container

بتدير الـ Beans والـ Dependencies

CourseRepository Bean

Spring بتجهزه وتحقنه في الـ Constructor

CourseService Bean

بتنشئ الـ object والـ dependency جاهزة جواه

دي اسمها Constructor Injection، وهي طريقة مفضلة لحقن الـ Dependencies في Spring.

ونفس الفكرة تنطبق على `@Component` و`@RestController`، لو عندهم dependencies محتاجينها عن طريق الـ Constructor.

ملحوظة: مش لازم تحط `@Autowired` على الـ Constructor لو عندك Constructor واحد بس؛ Spring بتستخدمه تلقائيًا.

## 3. طيب ليه الـ Entities بنستخدم معاها `@NoArgsConstructor`؟

بص على المثال ده:

```

@Entity
@NoArgsConstructor
@AllArgsConstructor
public class Course {

    @Id
    private Long id;

    private String title;
}
```

### `@NoArgsConstructor`

بتولّد:

```
public Course() {
}
```

ليه ده مهم؟

الـ JPA specification بتشترط إن الـ Entity يكون عندها `public` أو `protected` no-argument constructor، عشان الـ JPA provider، زي Hibernate، يقدر ينشئ الـ object عند قراءة البيانات من قاعدة البيانات.

### `@AllArgsConstructor`

بتولّد Constructor بالشكل ده:

```
public Course(Long id, String title) {
    this.id = id;
    this.title = title;
}
```

وده بيسمحلك تنشئ object وتمرر له كل الـ fields مرة واحدة:

```
Course course = new Course(1L, "Spring Boot");
```

لكن مش معنى كده إن كل Entity لازم يكون عندها `@AllArgsConstructor`؛ دي اختيارية، وممكن تكون غير مناسبة لو عندك `id` بيتولد تلقائيًا أو عايز تتحكم في إنشاء الـ Entity.

## 4. هل القاعدة دي ثابتة؟

لأ، خلينا نلخّص الصح:

|نوع الكلاس|الاختيار الشائع|السبب|
|---|---|---|
|`@Service`|`@RequiredArgsConstructor`|حقن الـ Dependencies المطلوبة.|
|`@Component`|`@RequiredArgsConstructor`|حقن الـ Dependencies المطلوبة.|
|`@RestController`|`@RequiredArgsConstructor`|حقن الـ Services والـ Dependencies المطلوبة.|
|`@Entity`|`@NoArgsConstructor`|متطلب للـ JPA؛ وغالبًا `protected` اختيار مناسب.|
|`@Entity`|`@AllArgsConstructor`|اختياري، لإنشاء object بكل الـ fields.|

افتكر إن الـ Annotations دي مش مرتبطة ببعض ارتباطًا إلزاميًا بنوع الكلاس؛ تقدر تستخدم `@RequiredArgsConstructor` في أي كلاس محتاج Constructor للـ required fields، مش بس Spring Beans.

## 5. أهم نقطة تحفظها للانترفيو

- `@RequiredArgsConstructor` → للـ `final` fields والـ `@NonNull` fields التي تحتاج initialization.
    
- `@NoArgsConstructor` → Constructor من غير Parameters، ومهم للـ JPA Entities.
    
- `@AllArgsConstructor` → Constructor لكل الـ fields، لكنه مش ضروري لكل Entity.
    
- `@Service` و`@RestController` → Spring بتدير الـ Beans، وLombok بتولّد الـ Constructor، وSpring بتستخدمه في الـ Dependency Injection.
    

الخلاصة: Lombok بتولّد الـ Constructor، وSpring بتستخدمه لحقن الـ Dependencies، أما Hibernate فبيحتاج الـ no-args constructor عشان يقدر يتعامل مع الـ Entities.