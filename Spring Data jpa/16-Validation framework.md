

---

````markdown
# Jakarta Bean Validation — Spring Boot

## ما هو الـ Validation Framework؟

**Validation Framework** = نظام بيتحقق إن الداتا الجاية من المستخدم **صالحة** قبل ما تدخل السيستم.

بيحصل قبل:
- الـ insert أو الـ update في الـ database
- تنفيذ أي business logic

---

## المشكلة من غير Validation

لو المستخدم بعت:

```json
{
  "fname": "",
  "email": "abc",
  "age": -5
}
````

النتيجة:

- بيانات غلط بتتخزن في الـ DB
- السيستم ممكن يـ crash
- الـ DB بتتملي بـ garbage data

---

## الحل في Spring Boot

Spring بيستخدم **Jakarta Bean Validation** عن طريق:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## آلية العمل

```
HTTP Request
     ↓
Controller (@Valid)
     ↓
Validation يشتغل أوتوماتيك
     ↓
فيه error? → يمنع التنفيذ + يرجع error response
مفيش error? → يكمل تنفيذ الـ logic
```

---

## Annotations — شرح كامل

### String Validation

|Annotation|الهدف|مثال|
|---|---|---|
|`@NotNull`|مش null|أي field لازم يتبعت|
|`@NotEmpty`|مش null ومش فاضي `""`|String أو Collection|
|`@NotBlank`|مش null، مش فاضي، ومش spaces بس|اسم المستخدم|
|`@Size`|طول محدد|password بين 8 و 20 حرف|
|`@Pattern`|regex معين|رقم موبايل بصيغة معينة|

```java
public class UserRequest {

    @NotNull
    private String nationalId;       // مش null بس، ممكن يكون ""

    @NotEmpty
    private String firstName;        // مش null ومش ""، بس ممكن "   "

    @NotBlank
    private String username;         // مش null، مش ""، ومش "   "

    @Size(min = 8, max = 20)
    private String password;         // بين 8 و 20 حرف

    @Pattern(regexp = "^01[0125][0-9]{8}$")
    private String phone;            // رقم موبايل مصري
}
```

> [!tip] الفرق بين الـ 3
> 
> - `@NotNull` → `" "` ✅ مقبول
> - `@NotEmpty` → `" "` ✅ مقبول
> - `@NotBlank` → `" "` ❌ مرفوض

---

### Email Validation

|Annotation|الهدف|
|---|---|
|`@Email`|لازم يكون بصيغة email صحيحة|

```java
@Email
private String email;   // "test@mail.com" ✅ | "abc" ❌
```

---

### Number Validation

|Annotation|الهدف|مثال|
|---|---|---|
|`@Min(n)`|القيمة لازم تكون `>= n`|العمر >= 18|
|`@Max(n)`|القيمة لازم تكون `<= n`|العمر <= 120|
|`@Positive`|أكبر من صفر بالظبط `> 0`|السعر|
|`@PositiveOrZero`|أكبر من أو يساوي صفر `>= 0`|الكمية|
|`@Negative`|أقل من صفر `< 0`|—|
|`@NegativeOrZero`|أقل من أو يساوي صفر `<= 0`|—|

```java
@Min(18)
@Max(120)
private int age;

@Positive
private double price;        // 0.0 ❌ | 0.01 ✅

@PositiveOrZero
private int quantity;        // -1 ❌ | 0 ✅ | 5 ✅
```

---

### Date Validation

|Annotation|الهدف|
|---|---|
|`@Past`|لازم يكون تاريخ في الماضي|
|`@PastOrPresent`|الماضي أو النهارده|
|`@Future`|لازم يكون تاريخ في المستقبل|
|`@FutureOrPresent`|المستقبل أو النهارده|

```java
@Past
private LocalDate birthDate;         // تاريخ الميلاد لازم في الماضي

@Future
private LocalDateTime appointmentDate;   // موعد لازم في المستقبل
```

---

### Nested Object Validation

|Annotation|الهدف|
|---|---|
|`@Valid`|يشغّل الـ validation جوه الـ nested object|

```java
public class CourseRequest {

    @NotBlank
    private String title;

    @Valid              // لازم تحطها عشان يتحقق من جوه الـ Address
    @NotNull
    private Address address;
}

public class Address {

    @NotBlank
    private String street;

    @NotBlank
    private String city;
}
```

> [!warning] من غير `@Valid` على الـ nested object الـ validation هيتجاهل الـ fields الجوه Address تماماً

---

## الـ Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<String> create(
            @Valid @RequestBody UserRequest request) {

        // لو الـ validation فشل مش هيوصل هنا أصلاً
        userService.save(request);
        return ResponseEntity.ok("User created successfully");
    }
}
```

---

## مثال كامل

### الـ DTO

```java
public class AuthorRequest {

    @NotBlank(message = "الاسم الأول مطلوب")
    private String firstName;

    @NotBlank(message = "الاسم الأخير مطلوب")
    private String lastName;

    @Email(message = "صيغة الإيميل غلط")
    @NotBlank(message = "الإيميل مطلوب")
    private String email;

    @Min(value = 18, message = "العمر لازم يكون 18 سنة على الأقل")
    @Max(value = 100, message = "العمر لازم يكون أقل من 100")
    private int age;

    @Past(message = "تاريخ الميلاد لازم يكون في الماضي")
    private LocalDate birthDate;
}
```

### الـ Request الغلط

```json
{
  "firstName": "",
  "lastName": "Doe",
  "email": "wrong-email",
  "age": 10,
  "birthDate": "2099-01-01"
}
```

### الـ Response

```json
{
  "firstName": "الاسم الأول مطلوب",
  "email": "صيغة الإيميل غلط",
  "age": "العمر لازم يكون 18 سنة على الأقل",
  "birthDate": "تاريخ الميلاد لازم يكون في الماضي"
}
```

---

## Validation Levels

```
┌─────────────────────────────────────┐
│         Request Level               │
│   @NotBlank / @Email / @Min ...     │
│   على الـ DTO في الـ Controller     │
├─────────────────────────────────────┤
│         Database Level              │
│   NOT NULL / UNIQUE / CHECK ...     │
│   على الـ Table في الـ DB           │
└─────────────────────────────────────┘
```

> [!important] الاتنين مكملين لبعض مش بديل
> 
> - الـ **Request Level** بيحمي الـ API ويرجع رسالة واضحة للـ user
> - الـ **DB Level** بيحمي البيانات من أي حاجة وصلت بشكل مباشر

---

## الخلاصة

|العنصر|القيمة|
|---|---|
|الـ Dependency|`spring-boot-starter-validation`|
|تفعيل الـ Validation|`@Valid` على الـ parameter في الـ Controller|
|أهم Annotations|`@NotBlank`, `@Email`, `@Min`, `@Max`, `@Past`, `@Future`|
|الـ Validation بيحصل|قبل دخول الـ method تماماً|
|لو فيه error|بيرجع `400 Bad Request` أوتوماتيك|

---

## الخطوات الجاية

- [ ] Custom Validation — تعمل annotation بنفسك
- [ ] Global Exception Handler — ترجع errors بشكل احترافي زي الشركات

```

---

ده الـ note كامل جاهز للـ Obsidian. الـ callouts زي `> [!tip]` و `> [!warning]` و `> [!important]` بتتعرض بشكل جميل في Obsidian أوتوماتيك. 

قولي لو تكمل بالـ **Global Exception Handler** أو الـ **Custom Validation**.
```