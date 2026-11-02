# Section 04: Spring MVC & REST APIs

> **الملف ده مبني على مذكراتك في `Spring-API/`, `Courses_Pilot/NOTES/` (ملف 7: Response Entity, ملف 8 و 17: Pagination, ملف 9: PathVariable & RequestParam), وسؤال الإنترفيو رقم 60 الخاص بالـ Pagination في المقابلات.**

---

### Q1: What is the complete Request Flow in Spring MVC? How does DispatcherServlet work?
**Level:** Junior / Mid  
**Related to:** Section 02 - Q8 (Servlet vs Spring), Q2 (HandlerMapping)

**Answer:**
بمجرد أن يرسل العميل (المتصفح أو الموبايل) HTTP Request، يمر الطلب برحلة منظمة جداً داخل Spring MVC:

```
  Client (Browser / Postman)
             │ HTTP Request (e.g. GET /api/v1/users/5)
             ▼
       Servlet Container (Tomcat)
             │
             ▼
  ┌──────────────────────────────────────────────────┐
  │         1. DispatcherServlet (Front Controller)  │
  └──────────────────────────┬───────────────────────┘
                             │
                             ├─► 2. HandlerMapping
                             │      "مين الـ Controller والـ Method المسؤولة عن الرابط ده؟"
                             │      ──► يرجع: UserController.getUserById()
                             │
                             ├─► 3. HandlerAdapter
                             │      "المسؤول عن تجهيز الـ Parameters واستدعاء الميثود فعلياً"
                             │
                             ├─► 4. UserController & Service & DB
                             │      "تنفيذ الـ Business logic وإرجاع UserDto"
                             │
                             ├─► 5. HttpMessageConverter (Jackson)
                             │      "يحول الـ Java Object إلى JSON String"
                             │
                             ▼
  ┌──────────────────────────────────────────────────┐
  │         6. DispatcherServlet يرسل الـ HTTP       │
  │            Response كـ 200 OK + JSON             │
  └──────────────────────────────────────────────────┘
```

#### المراحل الست بالتفصيل:
1. **DispatcherServlet:** يستقبل الـ HTTP Request كـ Front Controller موحد.
2. **HandlerMapping:** يفحص مسار الـ URL والـ HTTP Method ويبحث في الـ Controllers المسجلة ليجد الميثود المناسبة (`@GetMapping("/users/{id}")`).
3. **HandlerAdapter:** ينفذ عملية استدعاء الميثود؛ ويتولى قراءة الـ Parameters وتحويل الـ JSON في الـ Request Body إلى Java Object بواسطة `HttpMessageConverter`.
4. **Controller Execution:** يتم استدعاء الـ Controller method وتتواصل مع الـ Service والـ Repository لجلب البيانات.
5. **Serialization:** لو الكلاس عليه `@RestController`، يقوم `MappingJackson2HttpMessageConverter` بتحويل الـ return object إلى JSON ووضعه في الـ Response Body.
6. **Response Generation:** يرسل الـ `DispatcherServlet` الـ Response النهائي للعميل بالـ Status Code المناسب.

**Short answer (30 seconds version):**
> "The DispatcherServlet receives the HTTP request and queries the HandlerMapping to locate the matching controller method. The HandlerAdapter executes the method, using HttpMessageConverter to parse inputs. Finally, the returned object is serialized into JSON by Jackson and written directly to the HTTP response."

---

### Q2: What is the difference between `@PathVariable` and `@RequestParam`? When do you choose each?
**Level:** Junior  
**Related to:** Section 03 (Annotations), Q11 (REST Best Practices)

**Answer:**
مأخوذة من مذكراتك في `Courses_Pilot/NOTES/9-@Pathvariable and @RequestParam.md`:

#### 1. `@PathVariable` (مسار المورد داخل الـ URI):
- القيمة جزء أساسي لا يتجزأ من مسار الرابط (Path Segment).
- **الاستخدام الهندسي:** يستخدم للوصول إلى **مورد محدد بعينه (Resource Identity)**.
- مثال: جلب مستخدم محدد بالـ ID الخاص به:
```java
// URL: GET /api/v1/users/42
@GetMapping("/users/{id}")
public ResponseEntity<UserDto> getUser(@PathVariable("id") Long id) {
    return ResponseEntity.ok(userService.findById(id));
}
```

#### 2. `@RequestParam` (استعلام إضافي - Query Parameter):
- القيمة تأتي بعد علامة الاستفهام `?` في نهاية الرابط على هيئة `key=value`.
- **الاستخدام الهندسي:** يستخدم للـ **Filtering, Searching, Sorting, and Pagination** (بيانات إضافية لتخصيص شكل أو حجم النتيجة).
- مثال: جلب المستخدمين مع الترقيم والفلترة:
```java
// URL: GET /api/v1/users?status=ACTIVE&page=0&size=10
@GetMapping("/users")
public ResponseEntity<Page<UserDto>> getUsers(
        @RequestParam(defaultValue = "ACTIVE") String status,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    return ResponseEntity.ok(userService.findAllByStatus(status, PageRequest.of(page, size)));
}
```

| المعيار | `@PathVariable` | `@RequestParam` |
| :--- | :--- | :--- |
| **شكل الرابط** | `/products/15` | `/products?category=electronics&page=2` |
| **الغرض الرئيسي** | الإشارة لمورد فريد (Identification) | فلترة وترتيب وتحديد كمية البيانات (Filtering & Sorting) |
| **الإلزامية الافتراضية** | إجباري دائماً (بدونه يتغير المسار 404) | اختياري (ويمكن تعيين `defaultValue` أو `required = false`) |

**Short answer (30 seconds version):**
> "@PathVariable extracts values directly from the URI path to identify a specific resource (/users/{id}). @RequestParam extracts query parameters from the query string (/users?role=admin) used primarily for filtering, pagination, and sorting."

---

### Q3: Why should we use `ResponseEntity<T>`? What is the standard ApiResponse wrapper pattern?
**Level:** Junior / Mid  
**Related to:** Section 06 (Exception Handling)

**Answer:**
مأخوذة من مذكراتك بملف `7-Response Entity and API Response.md`:

#### 1. لماذا نستخدم `ResponseEntity<T>`؟
لو قمت بإرجاع الـ Data مباشرة من الـ Controller method:
```java
@PostMapping
public UserDto createUser(@RequestBody UserDto dto) {
    return userService.save(dto); // ⚠️ هيرجع دايماً 200 OK حتى لو كان Create!
}
```
المشكلة هنا: ليس لديك تحكم مرن في **HTTP Status Code** ولا في **HTTP Headers**.

`ResponseEntity` يمثل الـ **كامل استجابة الـ HTTP** (Status Code + Headers + Body):
```java
@PostMapping
public ResponseEntity<UserDto> createUser(@RequestBody UserDto dto) {
    UserDto created = userService.save(dto);
    return ResponseEntity
            .status(HttpStatus.CREATED) // 201 Created القياسي لعمليات الـ POST
            .header("Custom-Header", "Value")
            .body(created);
}
```

#### 2. نمط الغلاف الموحد للـ API (ApiResponse Pattern):
في المشاريع الاحترافية، لا نرجع الـ JSON مبعثراً في كل endpoint بشكل مختلف، بل نوحد الـ Response Format حتى يسهل على مطوري الـ Frontend معالجته، كما في مذكراتك:

```java
@Getter
@Builder
@AllArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;

    public static <T> ApiResponse<T> success(T data, String message) {
        return ApiResponse.<T>builder()
                .success(true)
                .message(message)
                .data(data)
                .timestamp(LocalDateTime.now())
                .build();
    }

    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .data(null)
                .timestamp(LocalDateTime.now())
                .build();
    }
}
```

**الاستخدام في الـ Controller:**
```java
@GetMapping("/{id}")
public ResponseEntity<ApiResponse<UserDto>> getUser(@PathVariable Long id) {
    UserDto user = userService.findById(id);
    return ResponseEntity.ok(ApiResponse.success(user, "User fetched successfully"));
}
```

**Short answer (30 seconds version):**
> "ResponseEntity represents the entire HTTP response, giving full control over the status code, custom headers, and payload. Wrapping responses in a generic ApiResponse<T> structure ensures consistent API contracts across the entire application for both success and error states."

---


### Q4: Pagination in Interviews: Offset-based vs Cursor-based Pagination
**Level:** Mid / Senior  
**Related to:** Section 05 (Spring Data JPA), System Design

**Answer:**
هذا الموضوع من أهم وأقوى المواضيع المشروحة في مذكراتك بملف `60-Pagination in interviews.md` و `8-Pagination.md`:

في أي نظام يحتوي على ملايين السجلات، مستحيل أن ترجع كل البيانات دفعة واحدة (`SELECT * FROM orders`) وإلا ستسقط الذاكرة (OutOfMemoryError) ويموت السيرفر. الحل هو الـ **Pagination**.

#### 1. Offset-based Pagination (الترقيم المعتمد على الصفحات):
- الفكرة: قسّم البيانات لصفحات، وحدد رقم الصفحة وحجمها (`page=5&size=20`).
- الاستعلام الفعلي في SQL:
  ```sql
  SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 100;
  ```
- في Spring Data JPA:
  ```java
  Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
  Page<Product> productPage = productRepository.findAll(pageable);
  ```

**مشاكل الـ Offset-based (عيوب قاتلة عند كبر البيانات):**
1. **انهيار الأداء مع الـ Large Offsets (Performance Degradation):**
   - عندما تقول لقاعدة البيانات: `OFFSET 1000000 LIMIT 20`.
   - قاعدة البيانات مضطرة لقراءة ومسح **مليون وعشرين صف** من الديسك، ثم رمي المليون صف في القمامة وإعطائك آخر 20 صفاً فقط!
2. **مشكلة الصفوف الشبحية (Phantom Rows / Missing items):**
   - لو المستخدم يتصفح الصفحة الأولى، وخلال ذلك قام مستخدم آخر بإضافة منتج جديد في البداية؛ عندما يضغط العميل على الصفحة رقم 2، سيرى آخر عنصر من الصفحة الأولى مكرراً أمامه!

#### 2. Cursor-based Pagination (Keyset Pagination):
- الفكرة: بدلاً من القفز برقم الصفحة، نطلب البيانات بعد **آخر عنصر رآه المستخدم (Pointer / Cursor)**، عادة الـ Primary Key أو التوقيت:
  ```sql
  SELECT * FROM products WHERE id > 1050 ORDER BY id ASC LIMIT 20;
  ```
- قاعدة البيانات تستخدم الـ **B-Tree Index** الخاص بالـ `id` مباشرة للوصول للصف رقم 1050 في زمن ثابت $O(\log N)$ أو $O(1)$ ولا تقرأ الصفوف السابقة أبداً!

| المقارنة | Offset-based Pagination | Cursor-based (Keyset) Pagination |
| :--- | :--- | :--- |
| **استعلام الـ SQL** | `LIMIT 20 OFFSET 50000` | `WHERE id > 50000 ORDER BY id LIMIT 20` |
| **الأداء عند ملايين السجلات** | بطيء جداً $O(N)$ (مسح كامل) | سريع وثابت دائماً $O(\log N)$ بفضل الـ Index |
| **القفز لصفحة محددة** | ✅ يدعم القفز لصفحة 10 مباشرة | ❌ لا يدعم القفز العشوائي (Next / Prev فقط) |
| **التأثر بالإضافة والحذف المستمر** | يعاني من تكرار أو سقوط عناصر | متسق تماماً ولا يتأثر بالصفوف الجديدة |
| **أين يُستخدم؟** | لوحات التحكم (Admin Dashboards), صفحات الجداول | تطبيقات الـ Social Feeds (Twitter, Instagram, Infinite Scroll) |

**Short answer (30 seconds version):**
> "Offset pagination skips records using SQL OFFSET; it's simple and allows jumping to arbitrary pages, but performs poorly with large datasets (O(N)) and suffers from data shifting. Cursor pagination queries records relative to the last seen record ID (WHERE id > last_seen_id); it offers constant O(log N) indexed performance, making it the industry choice for large-scale infinite scrolling."

---

### Q5: What are REST API Best Practices and API Versioning Strategies?
**Level:** Junior / Mid  
**Related to:** System Design

**Answer:**
مستوحاة من مذكراتك في مجلد `System Design/12-Api Design.md` و `30-Api Versioning.md`:

#### 1. Best Practices لتصميم RESTful APIs:
- **استخدم الأسماء للـ Resources وليس الأفعال:**
  - ❌ خطأ: `/api/getUsers`, `/api/deleteUser/5`, `/api/createNewProduct`
  - ✅ صح: `/api/v1/users` (مع استخدام الـ HTTP Methods للتعبير عن الفعل: GET, POST, DELETE, PUT).
- **الجمع بدلاً من المفرد:** استخدم `/users` بدلاً من `/user`.
- **التسلسل الهرمي للعلاقات (Nested Resources):**
  - لجلب منشورات مستخدم معين: `GET /api/v1/users/{userId}/posts`
- **استخدم الـ HTTP Status Codes الدقيقة:**
  - `200 OK`: للعمليات الناجحة العادية.
  - `201 Created`: بعد نجاح الـ POST وإنشاء السجل.
  - `204 No Content`: بعد نجاح الحذف DELETE.
  - `400 Bad Request`: خطأ في المدخلات (Validation Failure).
  - `401 Unauthorized`: المستخدم غير مسجل دخول أصلاً (Missing/Invalid Token).
  - `403 Forbidden`: المستخدم مسجل دخول لكن لا يمتلك الصلاحية الكافية (Role mismatch).
  - `404 Not Found`: المورد غير موجود.
  - `409 Conflict`: تعارض (مثل محاولة تسجيل إيميل مسجل مسبقاً).

#### 2. استراتيجيات الـ API Versioning:
عند تعديل الـ API بطريقة غير متوافقة مع الإصدارات السابقة (**Breaking Changes**)، يجب عمل Versioning:
1. **URI Versioning (الأكثر شيوعاً واستقراراً):**
   - المسار: `/api/v1/orders` ثم `/api/v2/orders`.
   - الميزة: سهلة الفهم للجميع وتعمل مع الـ Caching بسلاسة.
2. **Query Parameter Versioning:**
   - المسار: `/api/orders?version=1`.
3. **Custom Headers Versioning:**
   - إرسال هيدر في الطلب: `X-API-VERSION: 2`.
4. **Content Negotiation / Accept Header:**
   - إرسال: `Accept: application/vnd.company.app-v2+json`.

**Short answer (30 seconds version):**
> "REST best practices require using nouns for resources (/users), appropriate HTTP methods (GET, POST, PUT, DELETE), standard HTTP status codes (201 Created, 404 Not Found, 409 Conflict), and predictable URL nesting. API versioning is most cleanly handled via URI path versioning (/api/v1/resource) to prevent breaking existing mobile/frontend clients."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What is Idempotency? Which HTTP methods are idempotent? (GET, PUT, DELETE are idempotent; POST is not).*
2. *What is Content Negotiation and how do you support both XML and JSON in Spring MVC? (`@GetMapping(produces = { "application/json", "application/xml" })`).*
3. *What is the difference between PUT and PATCH? (PUT updates the entire resource, PATCH updates specific fields partially).*
4. *How does Spring MVC handle file uploads? (Via `MultipartFile` and standard multipart resolver).*

---

## 🔗 System Design Connection: API Gateway, Rate Limiting & Pagination

### Real-world System Design Application:
في الأنظمة الموزعة والـ Microservices، واجهات الـ REST لا تفتح مباشرة للعامة، بل تجلس خلف **API Gateway** (مثل Spring Cloud Gateway أو Kong):

```
 Clients (Mobile / Web)
          │
          ▼
┌──────────────────────────────────────┐
│          API Gateway                 │
│  - SSL Termination                   │
│  - Rate Limiting (Token Bucket)      │
│  - Authentication (JWT Validation)   │
│  - Routing (/api/v1/users ──► Serv)  │
└──────────────────┬───────────────────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
┌──────────────────┐ ┌──────────────────┐
│   User Service   │ │  Order Service   │
│ (Spring Boot)    │ │ (Spring Boot)    │
└──────────────────┘ └──────────────────┘
```

**System Design Patterns:**
1. **Rate Limiting:** حماية الـ Endpoints من هجمات الـ DoS أو سوء استخدام السيرفر عن طريق تحديد عدد الطلبات المسموحة لكل مستخدم في الدقيقة (مثلاً 100 req/min باستخدام خوارزمية Token Bucket في Redis).
2. **Pagination at Scale:** حماية شبكة الاتصال وقواعد البيانات من الاختناق بمنع جلب جداول عملاقة بدون `limit` و `offset/cursor`.
