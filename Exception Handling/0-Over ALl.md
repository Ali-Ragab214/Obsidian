# الدليل الشامل — Exception Handling في Java و Spring Boot

---

## المرحلة 1 — أساسيات Exceptions في Java

---

### الـ Hierarchy الكاملة

```
Throwable
├── Error                          ← مشاكل الـ JVM (OutOfMemory..) — منتلمسهاش
└── Exception
    ├── Checked Exceptions         ← لازم تعملها handle أو throws
    │   ├── IOException
    │   ├── SQLException
    │   └── ClassNotFoundException
    │
    └── RuntimeException (Unchecked)  ← مش لازم handle
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IllegalStateException
        └── Custom Exceptions ← هنعملها إحنا
```

---

### الفرق بين Checked و Unchecked

```java
// ✅ Checked — الـ compiler بيجبرك تتعامل معاها
public void readFile() throws IOException {
    FileReader file = new FileReader("test.txt");
}

// ✅ Unchecked — مش لازم تتعامل معاها
public void divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("Cannot divide by zero");
}
```

---

### القاعدة الذهبية في Spring Boot

```
99% من الوقت هتستخدم RuntimeException
لأن Checked Exceptions بتخلي الكود مليان try-catch في كل مكان
```

```java
// ❌ كل الـ Custom Exceptions بتاعتك تورث من دي
public class ResourceNotFoundException extends Exception { }

// ✅ الصح
public class ResourceNotFoundException extends RuntimeException { }
```

---

## المرحلة 2 — بناء Custom Exceptions صح

---

### الخطأ الشائع

```java
// ❌ Exception واحدة عامة لكل حاجة
public class AppException extends RuntimeException {
    public AppException(String message) {
        super(message);
    }
}

// استخدامها في كل مكان
throw new AppException("Course not found");
throw new AppException("Email already exists");
throw new AppException("Not authorized");
// المشكلة: كلهم هيترجعوا نفس الـ HTTP status ❌
```

---

### الصح — Exception لكل نوع مشكلة

```java
// 1️⃣ Base Exception — كل الـ custom exceptions بتورث منها
public abstract class BaseException extends RuntimeException {
    private final HttpStatus status;

    protected BaseException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}
```

```java
// 2️⃣ Specific Exceptions
public class ResourceNotFoundException extends BaseException {
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " not found with id: " + id, HttpStatus.NOT_FOUND);
    }
}

public class DuplicateResourceException extends BaseException {
    public DuplicateResourceException(String message) {
        super(message, HttpStatus.CONFLICT);
    }
}

public class UnauthorizedException extends BaseException {
    public UnauthorizedException(String message) {
        super(message, HttpStatus.FORBIDDEN);
    }
}

public class BadRequestException extends BaseException {
    public BadRequestException(String message) {
        super(message, HttpStatus.BAD_REQUEST);
    }
}
```

---

### ليه الطريقة دي أحسن؟

```
✅ كل Exception عندها الـ HTTP status بتاعها جواها
✅ الـ GlobalExceptionHandler بيقدر يتعامل معاهم كلهم بـ handler واحد
✅ سهل تضيف exception جديدة من غير ما تلمس الـ handler
```

---

## المرحلة 3 — `@ControllerAdvice` و `@ExceptionHandler`

---

### إيه هو `@ControllerAdvice`؟

```
Annotation بتقول لـ Spring:
"الـ class دي هتتعامل مع الـ exceptions
اللي بتحصل في أي Controller في المشروع كله"
```

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {
    // كل الـ handlers هنا
}
```

> `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody` تلقائياً — عشان الـ response يترجع JSON مباشرة.

---

### `@ExceptionHandler` — إزاي بيشتغل

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // بيمسك أي exception من النوع ده في أي controller
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ApiResponse.error(ex.getMessage()));
    }
}
```

```
الـ Flow:
Controller بيرمي Exception
        ↓
Spring بيدور على @ExceptionHandler
اللي بياخد نفس النوع
        ↓
بيشغله ويرجع الـ Response
```

---

### الطريقة الاحترافية — Handler واحد لكل الـ Custom Exceptions

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // ✅ handler واحد لكل الـ exceptions اللي بتورث من BaseException
    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ApiResponse<Void>> handleBaseException(BaseException ex) {
        return ResponseEntity
                .status(ex.getStatus())
                .body(ApiResponse.error(ex.getMessage()));
    }
}
```

بكده مش محتاج تعمل handler منفصل لكل exception — كلهم بيمروا من هنا. 🎯

---

## المرحلة 4 — Validation Exceptions

---

### `MethodArgumentNotValidException`

بتحصل لما `@Valid` يفشل على الـ DTO.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ApiResponse<Map<String, String>>> handleValidationErrors(
        MethodArgumentNotValidException ex) {

    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
    );

    return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ApiResponse.error("Validation failed", errors));
}
```

الـ Response هيبقى:

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": {
    "title": "Title is required",
    "price": "Price must be positive"
  }
}
```

---

## المرحلة 5 — Database Exceptions

---

### `DataIntegrityViolationException`

بتحصل لو حاولت تكسر unique constraint.

```java
@ExceptionHandler(DataIntegrityViolationException.class)
public ResponseEntity<ApiResponse<Void>> handleDataIntegrityViolation(
        DataIntegrityViolationException ex) {
    return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(ApiResponse.error("Duplicate entry or constraint violation"));
}
```

---

### `MethodArgumentTypeMismatchException`

بتحصل لو path variable من نوع غلط.

```java
// GET /api/courses/abc  ← "abc" مش Long
@ExceptionHandler(MethodArgumentTypeMismatchException.class)
public ResponseEntity<ApiResponse<Void>> handleTypeMismatch(
        MethodArgumentTypeMismatchException ex) {
    return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ApiResponse.error("Invalid parameter type: " + ex.getName()));
}
```

---

## المرحلة 6 — الـ Fallback Handler

---

### دايماً اعمل catch-all في الآخر

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ApiResponse<Void>> handleGenericException(Exception ex) {
    // Log الـ error عشان تقدر تتابعه
    log.error("Unexpected error occurred", ex);

    return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ApiResponse.error("An unexpected error occurred"));
}
```

```
ليه مهم؟
لو حصل أي exception مش متوقع
مش هيطلع الـ stack trace للـ client
ده security risk كبير لو ظهر
```

---

## الـ GlobalExceptionHandler النهائي الكامل

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ApiResponse<Void>> handleBaseException(BaseException ex) {
        log.warn("Business exception: {}", ex.getMessage());
        return ResponseEntity
                .status(ex.getStatus())
                .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Map<String, String>>> handleValidation(
            MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(ApiResponse.error("Validation failed", errors));
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ApiResponse<Void>> handleDataIntegrity(
            DataIntegrityViolationException ex) {
        return ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(ApiResponse.error("Duplicate entry or constraint violation"));
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ApiResponse<Void>> handleTypeMismatch(
            MethodArgumentTypeMismatchException ex) {
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(ApiResponse.error("Invalid parameter: " + ex.getName()));
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ApiResponse<Void>> handleAccessDenied(
            AccessDeniedException ex) {
        return ResponseEntity
                .status(HttpStatus.FORBIDDEN)
                .body(ApiResponse.error("You don't have permission to perform this action"));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Void>> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("An unexpected error occurred"));
    }
}
```

---

## المرحلة 7 — Best Practices للـ Production

---

### 1. اعمل Error Code مش بس Message

```java
public record ErrorResponse(
    boolean success,
    String errorCode,     // "COURSE_NOT_FOUND"
    String message,
    LocalDateTime timestamp,
    String path
) {}
```

```
ليه؟ لأن الـ Frontend بيقدر يتعامل مع errorCode
بدل ما يعمل string matching على الـ message
```

---

### 2. اعمل Logging مستويات مختلفة

```java
log.warn("Business logic error: {}", ex.getMessage());   // متوقع
log.error("System error", ex);                            // مش متوقع
```

---

### 3. متكشفش تفاصيل حساسة

```java
// ❌ خطر
catch (SQLException ex) {
    return ApiResponse.error(ex.getMessage()); // ممكن يكشف DB structure
}

// ✅ صح
catch (SQLException ex) {
    log.error("Database error", ex); // اللوج بس
    return ApiResponse.error("A database error occurred");
}
```

---

## ملخص الـ Roadmap اللي تتعلمه

```
1. Java Exception Hierarchy (Checked vs Unchecked)
2. Custom Exceptions مع BaseException pattern
3. @RestControllerAdvice + @ExceptionHandler
4. Validation Exceptions (MethodArgumentNotValidException)
5. Database Exceptions (DataIntegrityViolationException)
6. Security Exceptions (AccessDeniedException)
7. Generic Fallback Handler
8. Logging strategy
9. Error codes للـ Frontend
```

---

## Interview Questions

```
1. إيه الفرق بين Checked و Unchecked Exceptions؟
2. ليه بنستخدم RuntimeException في الـ Custom Exceptions؟
3. إيه دور @ControllerAdvice؟
4. إزاي بتتعامل مع Validation Errors في Spring؟
5. ليه مهم عندك Generic Exception Handler في الآخر؟
6. إزاي بتمنع الـ sensitive data من الظهور في الـ error response؟
```

---

> **الخطوة الجاية:** طبق الـ `BaseException` pattern ده في الـ SkillHub بتاعك بدل الـ Exceptions المنفصلة اللي عندك دلوقتي — هيوحد الكود ويخليه أنضف بكتير. عايز تبدأ فيها؟