# Global Exception Handling in Spring Boot

## لماذا نستخدمه؟ وكيف يعمل؟ وما هي أفضل الممارسات؟

هذا من أكثر الأسئلة شيوعًا في مقابلات Spring Boot.

---

# السؤال

> **Why do we use Global Exception Handling in Spring Boot? How does it work internally?**

---

# أولاً: ما هي المشكلة؟

لنفترض أن عندك Service

```java
@Service
public class UserService {

    public User getUser(Long id) {

        return repository.findById(id)
                .orElseThrow(() -> new UserNotFoundException());
    }

}
```

وفي Controller

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id){

    return userService.getUser(id);
}
```

لو الـ User غير موجود

سيحدث

```java
UserNotFoundException
```

---

بدون Global Exception Handler

الـ Exception ستصعد بهذا الشكل

```text
Repository

↓

Service

↓

Controller

↓

DispatcherServlet

↓

Tomcat

↓

HTTP 500
```

وسيعود للمستخدم

```json
{
   "timestamp":"...",
   "status":500,
   "error":"Internal Server Error"
}
```

وهذا سيء جداً.

لماذا؟

لأن العميل (Client) لا يعرف ما المشكلة.

---

# الحل الخاطئ

أن تعمل

```java
try{
   ...
}
catch(...)
```

في كل Controller.

مثال

```java
@GetMapping("/{id}")
public ResponseEntity<?> getUser(Long id){

    try{

        return ResponseEntity.ok(service.getUser(id));

    }catch(UserNotFoundException e){

        return ResponseEntity.status(404).body("User Not Found");
    }

}
```

لو عندك

100 Endpoint

ستكرر نفس الكود

100 مرة.

وهذا مخالف لمبدأ

```text
DRY
Don't Repeat Yourself
```

---

# الحل الصحيح

نعمل

```text
Global Exception Handler
```

مرة واحدة.

وأي Exception يحدث

Spring يرسله له تلقائياً.

---

# كيف يعمل داخلياً؟

عندما يحدث Exception

الفلو يكون كالتالي

```text
Browser

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository

↓

Exception
```

بدلاً من أن يرسلها مباشرة إلى Tomcat

Spring يسأل

```text
هل يوجد Exception Handler؟
```

إذا وجد

```java
@ControllerAdvice
```

↓

ينادي عليه.

---

الـ Flow يصبح

```text
Controller

↓

Exception

↓

@ControllerAdvice

↓

JSON Response

↓

Client
```

---

# كيف ننشئه؟

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

أو

```java
@ControllerAdvice
```

لكن الأفضل مع REST API هو

```java
@RestControllerAdvice
```

لأنه يعيد JSON مباشرة.

---

# معالجة Exception معين

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(){

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body("User Not Found");

    }

}
```

لو حدث

```text
UserNotFoundException
```

سيعود

```http
404
```

مع

```json
User Not Found
```

---

# ماذا يحدث داخلياً؟

Spring يبحث داخل

```java
@ExceptionHandler
```

عن النوع

```java
UserNotFoundException.class
```

إذا وجد تطابقاً

ينفذ هذه الـ Method.

---

# يمكن استقبال الـ Exception

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<?> handle(UserNotFoundException ex){

}
```

فنستطيع استخدام

```java
ex.getMessage()
```

مثال

```java
throw new UserNotFoundException("User 15 Not Found");
```

↓

الرد

```json
{
   "message":"User 15 Not Found"
}
```

---

# لماذا لا نرجع String؟

الأفضل

إنشاء

DTO

---

مثال

```java
public class ErrorResponse {

    private String message;

    private int status;

    private LocalDateTime timestamp;

}
```

ثم

```java
return new ErrorResponse(...)
```

النتيجة

```json
{
    "message":"User Not Found",
    "status":404,
    "timestamp":"2026-07-05T12:00"
}
```

وهذا هو الـ Best Practice.

---

# هل نعمل Exception لكل شيء؟

نعم.

مثلاً

```text
UserNotFoundException

EmailAlreadyExistsException

ProductNotFoundException

OrderNotFoundException

InvalidPasswordException

PaymentFailedException
```

كل Exception يمثل Business Rule.

---

# أين نضعهم؟

```text
exception/

    UserNotFoundException

    ProductNotFoundException

    OrderNotFoundException

    EmailAlreadyExistsException

    GlobalExceptionHandler
```

---

# ماذا لو عندنا Validation؟

مثلاً

```java
@NotBlank

@Email

@Size
```

وأرسل المستخدم

```json
{
   "email":"abc"
}
```

سيحدث

```text
MethodArgumentNotValidException
```

نلتقطه

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
```

---

ثم

```java
List<String> errors =
ex.getBindingResult()
  .getFieldErrors()
```

ونعيد

```json
{
   "errors":[
      "Name is required",
      "Email is invalid"
   ]
}
```

---

# ماذا عن Exception غير المتوقع؟

نعمل

```java
@ExceptionHandler(Exception.class)
```

وهذا آخر Handler.

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handle(Exception ex){

}
```

أي Exception لم يجد له Spring Handler

سيصل هنا.

---

# ترتيب البحث

لو حدث

```text
UserNotFoundException
```

Spring يبحث

```text
هل يوجد

@ExceptionHandler(UserNotFoundException.class)

↓

نعم

↓

ينفذه
```

إن لم يجد

↓

```text
هل يوجد RuntimeException؟
```

↓

إن لم يجد

↓

```text
Exception.class
```

---

# أفضل شكل للـ Error Response

بدلاً من

```json
{
   "message":"User Not Found"
}
```

الأفضل

```json
{
   "timestamp":"2026-07-05T20:10",

   "status":404,

   "error":"NOT_FOUND",

   "message":"User Not Found",

   "path":"/users/15"
}
```

وهذا قريب من RFC 7807 (Problem Details) أو يمكنك استخدام `ProblemDetail` في Spring Boot 3.

---

# ماذا نفعل عندما يزيد عدد الـ Entities؟

لنفترض أصبح لدينا

```text
User

Product

Order

Category

Customer

Payment

Review
```

سنحتاج Exceptions كثيرة.

أفضل تنظيم

```text
exception
│
├── user
│      UserNotFoundException
│      UserAlreadyExistsException
│
├── product
│      ProductNotFoundException
│      ProductOutOfStockException
│
├── order
│      OrderNotFoundException
│
├── common
│      GlobalExceptionHandler
│      ErrorResponse
```

---

# هل نكرر Handler لكل Exception؟

لا.

بدلاً من كتابة

```java
@ExceptionHandler(UserNotFoundException.class)
```

و

```java
@ExceptionHandler(ProductNotFoundException.class)
```

و

```java
@ExceptionHandler(CategoryNotFoundException.class)
```

يمكننا إنشاء Exception أساسي.

---

## Base Exception

```java
public abstract class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }

}
```

---

ثم

```java
public class UserNotFoundException
        extends ResourceNotFoundException {

    public UserNotFoundException(Long id){

        super("User not found with id " + id);

    }

}
```

و

```java
public class ProductNotFoundException
        extends ResourceNotFoundException {

    public ProductNotFoundException(Long id){

        super("Product not found with id " + id);

    }

}
```

---

الآن Handler واحد يكفي

```java
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(
        ResourceNotFoundException ex,
        HttpServletRequest request) {

    ErrorResponse response = new ErrorResponse(
            LocalDateTime.now(),
            404,
            "NOT_FOUND",
            ex.getMessage(),
            request.getRequestURI()
    );

    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
}
```

لأن جميع Exceptions ترث من `ResourceNotFoundException`، فإن أي استثناء من هذا النوع سيُعالج بنفس الطريقة.

---

# أفضل هيكل في المشاريع الكبيرة

```text
exception
│
├── base
│     ResourceNotFoundException
│     BusinessException
│
├── user
│     UserNotFoundException
│
├── product
│     ProductNotFoundException
│
├── order
│     OrderNotFoundException
│
├── ErrorResponse
│
└── GlobalExceptionHandler
```

بهذا الشكل، عندما تضيف Entity جديدة (مثل `Invoice` أو `Category`) لن تحتاج إلى تعديل الـ `GlobalExceptionHandler` إذا كانت ترث من نفس الـ Base Exception، مما يجعل المشروع أسهل في التوسع والصيانة.