
---

# 🧠 أول سؤال: يعني اي Exception أصلاً؟

تخيل عندنا API فيه Author.

الـ user بعت request:

```
GET /authors/10
```

لكن في الحقيقة **مفيش Author رقم 10** في الداتابيز 😅

إيه اللي هيحصل؟

Spring هيكسر التطبيق ويرجع Error كبير فيه StackTrace طويل جدًا 🤯

وده شكل سيء جدًا:

- مش مفهوم للـ frontend
    
- فيه تفاصيل خطيرة عن السيرفر
    
- مش professional
    

إحنا عايزين بدل ده نرجع رد مرتب زي:

```json
{
  "success": false,
  "message": "Author not found",
  "status": 404
}
```

وهنا يظهر دور 💥

# Global Exception Handling

---

# 🎯 الفكرة ببساطة جدًا

بدل ما كل Controller يمسك الأخطاء لوحده  
نخلي **مكان واحد في المشروع كله** يمسك أي Error يحصل.

زي السنترال كدا ☎️

أي Error يحصل في المشروع → يروح للـ Global Handler.

---

# 👀 المشكلة لو معملناش Global Handling

هتضطر تعمل كدا في كل Controller 😵

```java
@GetMapping("/{id}")
public ResponseEntity<?> getAuthor(@PathVariable int id){
    try {
        Author author = service.getAuthor(id);
        return ResponseEntity.ok(author);
    } catch (Exception ex){
        return ResponseEntity.status(404).body("Author not found");
    }
}
```

تخيل تعمل try/catch في 50 API 😭  
كارثة.

الحل = Global Exception Handler 🔥

---

# 🏗️ هنحتاج 3 حاجات بس

ركز جدًا 👇

1️⃣ Exception خاص بينا  
2️⃣ شكل الرسالة اللي هنرجعها  
3️⃣ Global Handler يمسك الأخطاء

هنطبقهم على Author مثال حقيقي.

---

# ① نعمل Exception خاص بينا

بدل ما نستخدم Exception عام  
نقول: لو Author مش موجود → ارمي AuthorNotFoundException

## نعمل Class جديد

📁 package: exception

```java
public class AuthorNotFoundException extends RuntimeException {

    public AuthorNotFoundException(String message){
        super(message);
    }

}
```

ليه RuntimeException؟  
علشان نرميه من غير try/catch.

---

# ② نستخدمه في Service

هنا أهم نقطة في الشرح كله 👇

الـ Service هو اللي بيشوف Author موجود ولا لا.

```java
public Author getAuthorById(int id){
    return authorRepository.findById(id)
            .orElseThrow(() ->
                new AuthorNotFoundException("Author not found"));
}
```

### يعني اي اللي حصل هنا؟

لو الـ ID مش موجود:  
بدل ما يرجع null ❌  
هيعمل:

```
💥 throw new AuthorNotFoundException
```

طيب بعد ما يرمي Exception… يروح فين؟ 🤔

يروح للـ Global Handler 😎

---

# ③ نحدد شكل الرسالة اللي هترجع للـ Frontend

عايزين كل Errors تبقى بنفس الشكل.

نعمل Class اسمه ApiErrorResponse.

```java
import lombok.*;

@Data
@Builder
@AllArgsConstructor
public class ApiErrorResponse {

    private boolean success;
    private String message;
    private int status;
}
```

ده مجرد DTO هنرجعه في الـ response.

---

# ④ أهم خطوة 🔥 Global Exception Handler

نعمل Class جديد:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
}
```

اسمع الجملة دي واحفظها ❤️

> RestControllerAdvice = امسك أي Exception يحصل في أي Controller في المشروع

---

## نمسك AuthorNotFoundException

```java
@ExceptionHandler(AuthorNotFoundException.class)
public ResponseEntity<ApiErrorResponse> handleAuthorNotFound(
        AuthorNotFoundException ex){

    ApiErrorResponse error = ApiErrorResponse.builder()
            .success(false)
            .message(ex.getMessage())
            .status(404)
            .build();

    return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
}
```

---

# 🧠 السيناريو كامل يحصل ازاي؟

ركز معايا خطوة خطوة 👇

1️⃣ User يطلب Author مش موجود  
2️⃣ Controller ينادي Service  
3️⃣ Service يرمي AuthorNotFoundException  
4️⃣ Spring يوقف التنفيذ  
5️⃣ يلاقي GlobalExceptionHandler  
6️⃣ يشغّل method اللي فوق  
7️⃣ يرجع JSON مرتب

🔥 بدون أي try/catch في controller.

---

# 🧪 نجرب Request حقيقي

Request:

```
GET /authors/5
```

Response:

```json
{
  "success": false,
  "message": "Author not found",
  "status": 404
}
```

شايف الجمال؟ 😍

---

# ⚠️ طب لو حصل Error تاني مش متوقع؟

لازم Safety Net.

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ApiErrorResponse> handleAll(Exception ex){

    ApiErrorResponse error = ApiErrorResponse.builder()
            .success(false)
            .message("Something went wrong")
            .status(500)
            .build();

    return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
}
```

ده يمسك أي Error في الدنيا.

---

# 📦 نحط الملفات فين في المشروع؟

structure بسيط جدًا:

```
api layer
   └── exception
         ├── AuthorNotFoundException
         ├── ApiErrorResponse
         └── GlobalExceptionHandler
```

---

# 💡 أهم 3 جمل تحفظهم

✔ Service يرمي Exception  
✔ Controller ميمسكش Exceptions  
✔ GlobalHandler هو اللي يرجع Response

---

لو وصلت لهنا وفهمت 💯  
نبدأ بعد كدا خطوة مهمة جدًا:  
**Validation Exceptions + DTO + @Valid** لأنها بتكمل الصورة كلها.




تمام يا علي، خلينا نفهم الموضوع خطوة خطوة ومرتب 👌

---

# 1️⃣ هل هنعمل Exception لكل Entity؟

**جواب:** **لا مش لازم كل Entity Exception**  
نعمل **Exceptions حسب الحاجة**، يعني لكل حالة ممكن تحصل فيها error مهم نحب نعالجها.

مثال على مشروعنا مع Author:

- عندنا Author
    
- لو ID مش موجود → نعمل `AuthorNotFoundException` ✅
    
- لو validation غلط → ممكن نعمل `InvalidAuthorDataException` (مش لازم لكل Entity إلا لو حابب)
    

أما لو عندك Entities تانية زي:

- `Customer` → لو مش موجود ممكن نعمل `CustomerNotFoundException`
    
- `Booking` → `BookingNotFoundException`
    
- إلخ…
    

يعني **كل Exception يمثل حالة خطأ واضحة**، مش لكل Entity كلاس واحد بالضرورة.

---

# 2️⃣ باقي الكلاسات اللي في فولدر exception

عادةً الفولدر فيه 3 أنواع من الكلاسات:

1️⃣ **Custom Exceptions**

- دي اللي اتكلمنا عنها، زي `AuthorNotFoundException`، `CustomerNotFoundException`
    
- مكانها في `exception` package.
    

2️⃣ **Global Exception Handler**

- كلاس واحد في الفولدر كله، مسؤول يمسك كل Exceptions.
    
- مثلا `GlobalExceptionHandler.java`
    

3️⃣ **Error Response DTO**

- كلاس واحد بس يحدد شكل الرسالة اللي هترجع للـ frontend
    
- مثلا `ApiErrorResponse.java`
    

> ملاحظ: الكلاس ده **مش Exception حقيقي**، لكنه يبقى في نفس الفولدر عشان جزء من الـ Exception Handling system.

---

# 3️⃣ ملخص التنظيم

```plaintext
exception
   ├── GlobalExceptionHandler.java   ← يمسك كل الاستثناءات
   ├── ApiErrorResponse.java         ← شكل الرسائل
   ├── AuthorNotFoundException.java  ← Exception خاص بالـ Author
   ├── CustomerNotFoundException.java
   ├── BookingNotFoundException.java
   └── InvalidAuthorDataException.java
```

**ملاحظات مهمة:**

- مش لازم تعمل Exception لكل Entity إلا لو فيه حالات فعلاً محتاجة handle خاصة
    
- لو نفس النوع من Error موجود في كذا Entity، ممكن تعمل Exception عام زي `ResourceNotFoundException` وتستخدمه لكل حاجة
    

مثال:

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message){
        super(message);
    }
}
```

وبعدين في كل Service:

```java
authorRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Author not found"));

customerRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
```

---

لو تحب، أقدر أرسملك **خريطة واضحة لكل الكلاسات في exception folder مع مثال على طريقة استخدامها لكل Entity** بحيث يكون كل شيء مرتب ومفهوم تمامًا 👌

هل تحب أعملها؟