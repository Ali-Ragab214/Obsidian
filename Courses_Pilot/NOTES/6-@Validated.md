ممتاز جدًا يا علي، لأنك وصلت لنقطة معظم الناس بتستخدم فيها `@Validated` بدون ما تفهم هي بتعمل إيه فعلًا. خلينا نفهمها من الصفر.

---

to achieve validation annotation on the methd parameter



احفظ الجملة دي:

> `@Valid` is used for validating request body objects, while `@Validated` enables method-level validation for parameters such as `@RequestParam` and `@PathVariable`.

# أولًا: ما هي `@Validated`؟

هي Annotation من Spring:

```java
import org.springframework.validation.annotation.Validated;
```

وظيفتها:

> **تفعيل الـ Method Validation على الـ Controller أو الـ Service.**

يعني تخلي Spring يراجع الـ Validation Annotations الموجودة على الـ Parameters بتاعة الـ Method.

---

# طب يعني إيه Method Validation؟

يعني تتحقق من صحة:

- `@PathVariable`
    
- `@RequestParam`
    
- `@RequestHeader`
    
- أي Parameter عادي في الـ Method
    

مثال:

```java
@GetMapping("/{id}")
public LessonResponse getLesson(
        @Min(1) @PathVariable Long id
){
    ...
}
```

أنت هنا بتقول:

> id لازم يكون أكبر من أو يساوي 1.

---

# هل ده هيشتغل لوحده؟

لا.

لو كتبت:

```java
@RestController
public class LessonController {

    @GetMapping("/{id}")
    public LessonResponse getLesson(
            @Min(1) @PathVariable Long id
    ){
        ...
    }
}
```

واليوزر عمل:

```http
GET /lessons/0
```

غالبًا الـ Validation مش هتشتغل.

---

# هنا يأتي دور `@Validated`

لما تكتب:

```java
@RestController
@Validated
public class LessonController {
}
```

Spring بيعمل Proxy حوالين الـ Controller.

والـ Proxy ده قبل ما ينفذ أي Method بيقول:

> "استنى... هل في Validation Annotations على Parameters؟"

لو لقى:

```java
@Min
@Max
@NotBlank
@NotNull
...
```

هيتحقق منها الأول.

لو فشلت:

مش هيدخل الـ Method أصلًا.

وهيرمي Exception:

```java
ConstraintViolationException
```

---

# طب `@Valid` بتعمل إيه؟

دي حاجة مختلفة.

مثال:

```java
@PostMapping
public void create(
        @Valid @RequestBody LessonRequest request
){
}
```

لو:

```java
public record LessonRequest(

        @NotBlank
        String title,

        @NotNull
        Integer duration
){}
```

Spring بيعمل Validation على الـ DTO نفسه.

---

# الفرق الحقيقي

## @Valid

بتتحقق من:

```java
@RequestBody
```

والـ Objects.

---

## @Validated

بتتحقق من:

```java
@PathVariable
@RequestParam
@RequestHeader
Method Parameters
```

---

# تعال نشوف Controller بتاعك

أنت عامل:

```java
@RestController
@Validated
@RequestMapping(...)
public class LessonController {
```

السؤال:

> هل استخدمتها فعلًا؟

الإجابة:

## نعم، استخدمتها هنا:

```java
@GetMapping
public ResponseEntity<ApiResponse<PaginationResponse<LessonResponse>>> getLessonsInSection(

        @PathVariable Long courseId,

        @PathVariable Long sectionId,

        @RequestParam(defaultValue = "0")
        @Min(0)
        int page,

        @RequestParam(defaultValue = "10")
        @Min(1)
        @Max(100)
        int size
)
```

---

# لو شلت @Validated

وخليت:

```java
GET /lessons?page=-1&size=500
```

قد يحصل:

```text
page = -1
size = 500
```

وتدخل Service عادي.

---

# لكن مع @Validated

لو حد عمل:

```http
GET ?page=-1
```

Spring هيقول:

```text
ConstraintViolationException
```

لأن:

```java
@Min(0)
```

اتفشلت.

---

ولو عمل:

```http
GET ?size=1000
```

هيقول:

```text
ConstraintViolationException
```

لأن:

```java
@Max(100)
```

اتفشلت.

---

# طب هل @Validated مستخدمة في createLesson؟

أنت عندك:

```java
@PostMapping
public ResponseEntity<?> createLesson(

        @Valid
        @RequestBody LessonRequest request
)
```

هنا:

> `@Validated` ملهاش أي دور.

اللي شغال هنا هو:

```java
@Valid
```

---

# طب هل محتاج @Validated في Controller بتاعك؟

الإجابة:

## نعم.

لأنك عندك:

```java
@Min(0)
int page

@Min(1)
@Max(100)
int size
```

وده بالضبط السيناريو اللي اتعملت عشانه.

---

# طب لو شلت الـ Min والـ Max؟

هيبقى عندك:

```java
@Valid @RequestBody
```

بس.

وساعتها:

```java
@Validated
```

تبقى ملهاش لازمة وممكن تشيلها.

---

# ملخص الإنترفيو

احفظ الجملة دي:

> `@Valid` is used for validating request body objects, while `@Validated` enables method-level validation for parameters such as `@RequestParam` and `@PathVariable`.

يعني:

```text
@Valid      → DTO Validation

@Validated  → Method Parameter Validation
```

وبالتالي في الـ `LessonController` بتاعك:

- `@Valid` مستخدمة في:
    
    - `createLesson()`
        
    - `updateLesson()`
        
- `@Validated` مستخدمة في:
    
    - `getLessonsInSection()` بسبب:
        

```java
@Min(0) int page

@Min(1)
@Max(100) int size
```

وده استخدام صحيح واحترافي جدًا.