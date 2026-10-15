# Section 06: Exception Handling & Validation

> **الملف ده مبني على مذكراتك في `Courses_Pilot/NOTES/` (ملف 2: Auth exp مع GlobalExceptionHandler, ملف 6: @Validated, ملف 7: ApiResponse) ومجلد `Exception Handling/`.**  
> يغطي: المعالجة المركزية للاستثناءات (`@RestControllerAdvice`), هيكلة رسائل الأخطاء, التحقق من المدخلات (Bean Validation), والـ RFC 7807 Problem Details.

---

### Q1: How does Global Exception Handling work in Spring Boot? Explain `@RestControllerAdvice` and `@ExceptionHandler`.
**Level:** Junior / Mid  
**Related to:** Section 04 - Q3 (ApiResponse), Q2 (Custom Exceptions)

**Answer:**
في البرمجة البدائية، كان المطور يضع `try-catch` داخل كل Controller Method لتفادي انهيار التطبيق. هذا الأسلوب كارثي: يكرر الكود، يشوه الـ Business Logic، ويجعل شكل الـ Error Response غير متناسق.

Spring وفر حلاً معمارياً رائعاً مبنياً على **AOP (Aspect-Oriented Programming)** وهو: **`@RestControllerAdvice`**.

#### كيف يعمل؟
- الـ `@RestControllerAdvice` يمثل اعتراضاً مركزياً (Global Interceptor).
- عندما تفشل أي ميثود في أي Controller وترمي Exception، بدلاً من أن يصل الخطأ للمتصفح كـ 500 Internal Server Error قبيح، يقوم Spring بتوجيه الخطأ فوراً إلى كلاس الـ `@RestControllerAdvice`.
- داخل هذا الكلاس، نحدد ميثودز تحمل `@ExceptionHandler(TargetException.class)` لتفصيل الاستجابة.

```
       HTTP Request
             │
             ▼
      UserController ──► Service (يرمي UserNotFoundException!)
             │
      ┌──────┴──────────────────────────────────────────┐
      │ Exception لم يتم معالجته بـ try-catch           │
      └──────┬──────────────────────────────────────────┘
             │ Spring يعترض الخطأ ويوجهه
             ▼
┌────────────────────────────────────────────────────────┐
│             @RestControllerAdvice                      │
│                                                        │
│  @ExceptionHandler(UserNotFoundException.class)        │
│  public ResponseEntity<ApiResponse> handleNotFound()  │
│  ──► يحول الخطأ إلى JSON أنيق بـ 404 Not Found         │
└────────────────────────────────────────────────────────┘
```

**Short answer (30 seconds version):**
> "@RestControllerAdvice is a centralized exception interceptor applying AOP across all controllers. It allows writing modular @ExceptionHandler methods to catch specific exceptions, log them, and translate them into standardized, meaningful HTTP error responses."

---

### Q2: How do you build a Production-Ready Global Exception Handler with standardized error structure?
**Level:** Junior / Mid  
**Related to:** Q1, Q3 (Validation Errors)

**Answer:**
مأخوذة ومطورة من كود مذكراتك في `Courses_Pilot/NOTES/2-Auth exp.md`:

#### 1. بنية الـ Error Response القياسية:
```java
@Getter
@Builder
public class ErrorResponse {
    private boolean success;
    private int status;
    private String error;
    private String message;
    private String path;
    private LocalDateTime timestamp;
    private Map<String, String> validationErrors; // لتفاصيل أخطاء الـ Validation
}
```

#### 2. كلاس الـ GlobalExceptionHandler الكامل:
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // 1. التعامل مع خطأ عدم وجود المورد (404 Not Found)
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex, HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        ErrorResponse error = ErrorResponse.builder()
                .success(false)
                .status(HttpStatus.NOT_FOUND.value())
                .error(HttpStatus.NOT_FOUND.getReasonPhrase())
                .message(ex.getMessage())
                .path(request.getRequestURI())
                .timestamp(LocalDateTime.now())
                .build();
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    // 2. التعامل مع خطأ الصلاحيات والـ Security (403 Forbidden)
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex, HttpServletRequest request) {
        ErrorResponse error = ErrorResponse.builder()
                .success(false)
                .status(HttpStatus.FORBIDDEN.value())
                .error(HttpStatus.FORBIDDEN.getReasonPhrase())
                .message("You do not have permission to access this resource")
                .path(request.getRequestURI())
                .timestamp(LocalDateTime.now())
                .build();
        return new ResponseEntity<>(error, HttpStatus.FORBIDDEN);
    }

    // 3. صائد الأخطاء العام لغير المتوقع (500 Internal Server Error)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(Exception ex, HttpServletRequest request) {
        log.error("Unhandled internal server error occurred", ex); // نسجل الـ StackTrace كاملاً في الـ Log
        ErrorResponse error = ErrorResponse.builder()
                .success(false)
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase())
                .message("An unexpected error occurred. Please contact support.") // لا نسرب تفاصيل تقنية حساسة للعميل!
                .path(request.getRequestURI())
                .timestamp(LocalDateTime.now())
                .build();
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

> ⚠️ **نصيحة أمنية هامة للإنترفيو (Gotcha):**  
> في بيئة الإنتاج، **إياك أن تسرب الـ Stack Trace أو رسائل الـ SQL Exceptions مباشرة للمستخدم** في استجابة الـ 500؛ لأن المهاجمين يستخدمون تفاصيل الكلاسات وأسماء الجداول في الـ SQL Injection واستغلال الثغرات! اطبع الـ StackTrace في الـ Logs الداخلية فقط، وأرجع رسالة عامة مؤدبة للعميل.

**Short answer (30 seconds version):**
> "A production-grade exception handler handles domain exceptions (404 Not Found), authentication/authorization errors (401/403), validation failures (400), and a fallback catch-all (500). Sensitive stack traces are logged internally for debugging but sanitized in the HTTP response returned to the client."

---

### Q3: How do you handle Bean Validation errors (`MethodArgumentNotValidException`)?
**Level:** Junior / Mid  
**Related to:** Section 03 (Validation Annotations)

**Answer:**
عندما ترسل بيانات غير صالحة في الـ Request Body لكلاس محمي بـ `@Valid`:
```java
public class RegisterRequest {
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}
```
إذا أرسل المستخدم إيميل فارغ أو باسورد قصير، Spring لن يستدعي الـ Controller method أصلاً، بل يرمي فوراً استثناء: **`MethodArgumentNotValidException`**.

#### معالجة الخطأ واستخراج الحقول الفاشلة بدقة:
```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidationErrors(
        MethodArgumentNotValidException ex, HttpServletRequest request) {

    Map<String, String> fieldErrors = new HashMap<>();
    
    // نستخرج كل حقل فاشل والرسالة الخاصة به
    ex.getBindingResult().getFieldErrors().forEach(error -> 
        fieldErrors.put(error.getField(), error.getDefaultMessage())
    );

    ErrorResponse response = ErrorResponse.builder()
            .success(false)
            .status(HttpStatus.BAD_REQUEST.value())
            .error("Validation Failed")
            .message("One or more fields failed validation checks")
            .path(request.getRequestURI())
            .timestamp(LocalDateTime.now())
            .validationErrors(fieldErrors) // إرجاع خريطة الأخطاء { "email": "...", "password": "..." }
            .build();

    return ResponseEntity.badRequest().body(response);
}
```

**شكل الـ JSON الناتج الرائع الذي يسعد مطور الـ Frontend:**
```json
{
  "success": false,
  "status": 400,
  "error": "Validation Failed",
  "message": "One or more fields failed validation checks",
  "path": "/api/v1/auth/register",
  "timestamp": "2026-10-15T10:30:00",
  "validationErrors": {
    "email": "Invalid email format",
    "password": "Password must be at least 8 characters"
  }
}
```

**Short answer (30 seconds version):**
> "When @Valid fails, Spring throws MethodArgumentNotValidException. We intercept it in our @RestControllerAdvice, iterate over the BindingResult's FieldErrors, extract the field names and validation messages into a map, and return an intuitive 400 Bad Request JSON payload."

---

### Q4: `@Valid` vs `@Validated` — What are Validation Groups?
**Level:** Mid  
**Related to:** Section 03 (Annotations)

**Answer:**
مأخوذة من مذكراتك بملف `Courses_Pilot/NOTES/6-@Validated.md`:

#### 1. الفرق التأسيسي:
- **`@Valid`:**
  - يأتي من مواصفة **Jakarta Bean Validation** القياسية.
  - يعمل فقط على الـ Method parameters (مثل `@Valid @RequestBody`).
  - **لا يدعم** الـ Validation Groups.
- **`@Validated`:**
  - Annotation خاص بـ **Spring Framework**.
  - يمكن وضعه على مستوى الـ Class لتفعيل التحقق على ميثودز الـ Service أو الـ Controller Parameters (مثل التحقق من `@PathVariable @Min(1) Long id`).
  - **يدعم بقوة الـ Validation Groups**.

#### 2. ما هي الـ Validation Groups ولماذا نحتاجها؟
تخيل كلاس `UserDto`:
- عند **الإنشاء (Create)**: حقل الـ `id` يجب أن يكون فارغاً، وحقل الـ `password` إجباري.
- عند **التعديل (Update)**: حقل الـ `id` إجباري، بينما حقل الـ `password` اختياري!

بدلاً من عمل اثنين DTOs مختلفين لكل عملية صغيرة، نستخدم **Validation Groups**:
```java
// 1. Interfaces لتمييز المجموعات
public interface OnCreate {}
public interface OnUpdate {}

// 2. تطبيق المجموعات على الـ DTO
public class UserDto {
    @Null(groups = OnCreate.class, message = "ID must be null on creation")
    @NotNull(groups = OnUpdate.class, message = "ID is required for update")
    private Long id;

    @NotBlank(groups = OnCreate.class, message = "Password required on creation")
    private String password;
}

// 3. في الـ Controller نحدد المجموعة المطلوبة عبر @Validated
@PostMapping
public ResponseEntity<Void> create(@Validated(OnCreate.class) @RequestBody UserDto dto) { ... }

@PutMapping
public ResponseEntity<Void> update(@Validated(OnUpdate.class) @RequestBody UserDto dto) { ... }
```

**Short answer (30 seconds version):**
> "@Valid is the standard Jakarta validation annotation, while @Validated is a Spring-specific variant. The primary advantage of @Validated is its support for Validation Groups, allowing developers to enforce different validation rules on the same DTO for distinct operations (e.g., Create vs Update)."

---

### Q5: What is RFC 7807 (Problem Details for HTTP APIs) in Spring Boot 3? [ADDED - not in my notes]
**Level:** Mid / Senior  
**Related to:** Section 04 (REST Standards)

**Answer:**
في الماضي، كانت كل شركة وكل مشروع يخترع شكلاً مخصصاً (Schema) لرسائل الأخطاء (مثل كتابة `error_msg` أو `errorMessage` أو `description`).

لحل هذه العشوائية في صناعة الويب، وضعت الـ IETF معياراً رسمياً موحداً في وثيقة **RFC 7807** باسم: **Problem Details**.
بدءاً من **Spring Boot 3 (Spring 6)**، أصبح هذا المعيار مدعوماً ومدمجاً في صلب الفريم وورك عبر كلاس: `org.springframework.http.ProblemDetail`.

#### تفعيل المعيار بسطر واحد في `application.yml`:
```yaml
spring:
  mvc:
    problemdetails:
      enabled: true
```

#### الاستخدام في كود Spring:
```java
@ExceptionHandler(ResourceNotFoundException.class)
public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
    ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    problem.setTitle("Resource Missing");
    problem.setType(URI.create("https://api.mycompany.com/errors/not-found"));
    problem.setProperty("timestamp", Instant.now()); // حقول إضافية مخصصة
    return problem;
}
```

**الـ JSON القياسي العالمي الناتج:**
```json
{
  "type": "https://api.mycompany.com/errors/not-found",
  "title": "Resource Missing",
  "status": 404,
  "detail": "User with id 42 does not exist",
  "instance": "/api/v1/users/42",
  "timestamp": "2026-10-15T12:00:00Z"
}
```

**Short answer (30 seconds version):**
> "RFC 7807 is an industry-standard specification for reporting machine-readable HTTP error details. Spring Boot 3 natively supports it via the ProblemDetail class, providing uniform fields (type, title, status, detail, instance) to eliminate arbitrary custom error schemas."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What is the difference between `@ControllerAdvice` and `@RestControllerAdvice`? (`@RestControllerAdvice` combines `@ControllerAdvice` and `@ResponseBody`).*
2. *How do you create a Custom Validator annotation in Spring? (Create an `@interface` and implement `ConstraintValidator<Annotation, Type>`).*
3. *What happens if an exception is thrown inside a `@ExceptionHandler` method? (It bubbles up to the Servlet container, resulting in standard 500 error page).*
4. *Can you order multiple `@ControllerAdvice` classes? (Yes, using `@Order` or implementing `Ordered`).*

---

## 🔗 System Design Connection: Observability, Error Budgets & Dead Letter Queues

### Real-world System Design Application:
في الأنظمة الموزعة (Microservices)، معالجة الأخطاء ليست مجرد كود جافا بل ركيزة أساسية في **System Reliability**:
1. **Error Tracking & Sentry:**
   - داخل `GlobalExceptionHandler`، يتم إرسال أخطاء الـ 500 فوراً لمنصات المراقبة مثل **Sentry** أو **Datadog** مع رقم `traceId` فريد لتتبع رحلة الطلب عبر الخدمات.
2. **Circuit Breakers (Resilience4j):**
   - عندما تبدأ إحدى الخدمات الخارجية في الفشل المستمر وإرجاع استثناءات، يقوم الـ Circuit Breaker بقطع الاتصال مؤقتاً لتفادي انهيار السيرفر التتابعي (**Cascading Failure**).
3. **Dead Letter Queue (DLQ):**
   - في معالجة الرسائل غير المتزامنة (Kafka / RabbitMQ)، عندما يفشل استهلاك رسالة بعد عدة محاولات (Retries)، يتم نقلها للـ DLQ لفحصها يدوياً بدون تعطيل الـ Pipeline.
