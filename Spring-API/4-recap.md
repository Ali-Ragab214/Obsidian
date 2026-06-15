

---

## Chapter 1 — ازاي Spring بيشتغل؟ (قبل أي كود)

### الفكرة الأساسية

```
بدون Spring:
─────────────
انت بتعمل كل حاجة بنفسك

UserService service = new UserService(
    new UserRepository(
        new DatabaseConnection(
            new Config()
        )
    )
);

مع Spring:
──────────
Spring بيعمل كل ده تلقائياً ✅

@Autowired
UserService service; // Spring جابه لك
```

### الـ IoC Container

```
┌─────────────────────────────────────┐
│         Spring IoC Container        │
│                                     │
│  بيعمل Objects وبيربطهم ببعض       │
│                                     │
│  UserController                     │
│       └── UserService               │
│               └── UserRepository    │
│                       └── DB        │
│                                     │
│  كل ده Spring بيعمله تلقائياً ✅    │
└─────────────────────────────────────┘
```

### الـ Bean

أي Object بتقوله Spring يتحكم فيه:

```java
// انت بتقوله بـ Annotation
@Service        // ← ده Bean
@Repository     // ← ده Bean
@Controller     // ← ده Bean
@Component      // ← ده Bean
```

---

## Chapter 2 — Annotations الأساسية

### @Component و أولاده

```java
// ─── الأب ───────────────────────────────────────
@Component
public class AnyClass { }
// بيقول لـ Spring: "اعمل Object من الـ class دي وتحكم فيه"

// ─── الأولاد (نفس @Component بس بمعنى أوضح) ────
@Repository       // للـ DB layer
public class UserRepository { }

@Service          // للـ Business Logic
public class UserService { }

@RestController   // للـ API endpoints
public class UserController { }
```

### @Autowired و Dependency Injection

الطريقة القديمة ❌:

```java
@Service
public class LessonService {

    @Autowired  // ← مش الأفضل
    private LessonRepository lessonRepository;
}
```

الطريقة الصح ✅ — Constructor Injection:

```java
@Service
@RequiredArgsConstructor  // ← Lombok بيعمل Constructor تلقائياً
public class LessonService {

    private final LessonRepository lessonRepository;
    private final LessonMapper lessonMapper;
    // Spring بيشوف الـ final fields ويحقنهم تلقائياً ✅
}
```

### ليه Constructor Injection أحسن؟

```
Field Injection (@Autowired):
─────────────────────────────
❌ مش ممكن تعمل Unit Test سهل
❌ الـ dependencies مش واضحة
❌ ممكن يبقى null

Constructor Injection:
──────────────────────
✅ سهل Testing
✅ الـ dependencies واضحة
✅ مستحيل يبقى null
✅ Best Practice
```

---

## Chapter 3 — Request Mapping

### الـ URLs

```java
@RestController
@RequestMapping("/api/v1/courses")  // Base URL
public class CourseController {

    // GET /api/v1/courses
    @GetMapping
    public ResponseEntity<?> getAllCourses() { }

    // GET /api/v1/courses/1
    @GetMapping("/{id}")
    public ResponseEntity<?> getCourseById(@PathVariable Long id) { }

    // POST /api/v1/courses
    @PostMapping
    public ResponseEntity<?> createCourse(@RequestBody CourseRequest request) { }

    // PUT /api/v1/courses/1
    @PutMapping("/{id}")
    public ResponseEntity<?> updateCourse(
            @PathVariable Long id,
            @RequestBody CourseRequest request) { }

    // DELETE /api/v1/courses/1
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteCourse(@PathVariable Long id) { }

    // PATCH /api/v1/courses/1/publish
    @PatchMapping("/{id}/publish")
    public ResponseEntity<?> publishCourse(@PathVariable Long id) { }
}
```

### شرح كل HTTP Method

|Method|بيعمل إيه|مثال|
|---|---|---|
|`GET`|جيب بيانات|جيب الـ courses|
|`POST`|عمل حاجة جديدة|عمل course جديد|
|`PUT`|عدّل كل حاجة|عدّل الـ course كله|
|`PATCH`|عدّل جزء بس|publish الـ course|
|`DELETE`|احذف|احذف الـ course|

---

## Chapter 4 — Parameters

### @PathVariable — من الـ URL

```java
// URL: /courses/5/sections/2/lessons/1
@GetMapping("/courses/{courseId}/sections/{sectionId}/lessons/{id}")
public ResponseEntity<?> getLesson(
        @PathVariable Long courseId,   // = 5
        @PathVariable Long sectionId,  // = 2
        @PathVariable Long id          // = 1
) { }
```

### @RequestParam — من الـ Query String

```java
// URL: /courses?page=0&size=10&sort=title
@GetMapping("/courses")
public ResponseEntity<?> getCourses(
        @RequestParam(defaultValue = "0")  int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(required = false)    String sort
        // required = false يعني اختياري
) { }
```

### @RequestBody — من الـ Body

```java
// Body: { "title": "Spring Boot", "duration": 30 }
@PostMapping
public ResponseEntity<?> createLesson(
        @Valid @RequestBody LessonRequest request
        // @Valid ← بيفعّل الـ Validation
) { }
```

### @AuthenticationPrincipal — الـ User اللي عمل Login

```java
@PostMapping
public ResponseEntity<?> createCourse(
        @RequestBody CourseRequest request,
        @AuthenticationPrincipal UserDetails userDetails
) {
    String email = userDetails.getUsername(); // email الـ instructor
    return courseService.createCourse(request, email);
}
```

---

## Chapter 5 — Request & Response DTOs

### Request DTO

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class LessonRequest {

    @NotBlank(message = "Title is required")
    @Size(min = 2, max = 100, message = "Title must be between 2 and 100 characters")
    private String title;

    @NotBlank(message = "Video URL is required")
    @Pattern(
            regexp = "^(https?://).*",
            message = "Video URL must start with http:// or https://"
    )
    private String videoUrl;

    @NotNull(message = "Duration is required")
    @Min(value = 1, message = "Duration must be at least 1 minute")
    @Max(value = 300, message = "Duration cannot exceed 300 minutes")
    private Integer duration;

    private Boolean isPreview = false;
    private Integer orderIndex = 0;
}
```

### Response DTO

```java
// Record — أبسط وأنظف من Class للـ Response
public record LessonResponse(
        Long id,
        String title,
        String videoUrl,
        Integer duration,
        Boolean isPreview,
        Integer orderIndex,
        Long sectionId
) {}
```

### ليه بنفصل Request عن Response؟

```
Entity (DB)
    ↑ مش بنبعته للـ Client أبداً
    │  (فيه passwords, internal fields)
    │
    ├── Request DTO  ← Client بيبعته لينا
    │      (بس الـ fields اللي محتاجينها)
    │
    └── Response DTO ← بنبعته للـ Client
           (بس الـ fields اللي يشوفها)
```

---

## Chapter 6 — ApiResponse Wrapper

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;

    public static <T> ApiResponse<T> success(String message, T data) {
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

---

## Chapter 7 — ResponseEntity

```java
// ─── 200 OK ───────────────────────────────────────
return ResponseEntity.ok(
        ApiResponse.success("Done", data)
);

// ─── 201 Created ──────────────────────────────────
return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(ApiResponse.success("Created", data));

// ─── 204 No Content ───────────────────────────────
return ResponseEntity.noContent().build();
```

---

## Chapter 8 — Validation

### Annotations

```java
@NotNull      // مش null
@NotBlank     // مش null ومش فاضية (للـ String)
@NotEmpty     // مش null ومش empty
@Size(min=2, max=100)
@Min(1)
@Max(300)
@Email
@Pattern(regexp="...")
@Positive     // رقم موجب
@Negative     // رقم سالب
@Future       // تاريخ في المستقبل
@Past         // تاريخ في الماضي
```

### ازاي بتشتغل؟

```
Client بيبعت Request
        │
        ▼
@Valid بيفحص كل الـ Annotations
        │
        ├── كل حاجة تمام → كمّل ✅
        │
        └── في error → MethodArgumentNotValidException ❌
                              │
                              ▼
                     GlobalExceptionHandler
                              │
                              ▼
                     Response بـ 400 Bad Request
```

---

## Chapter 9 — Exception Handling

### Custom Exceptions

```java
// 404
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// 409
public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}

// 403
public class UnauthorizedException extends RuntimeException {
    public UnauthorizedException(String message) {
        super(message);
    }
}
```

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleNotFound(
            ResourceNotFoundException ex
    ) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ApiResponse<Void>> handleDuplicate(
            DuplicateResourceException ex
    ) {
        return ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(UnauthorizedException.class)
    public ResponseEntity<ApiResponse<Void>> handleUnauthorized(
            UnauthorizedException ex
    ) {
        return ResponseEntity
                .status(HttpStatus.FORBIDDEN)
                .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Map<String, String>>> handleValidation(
            MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult()
          .getFieldErrors()
          .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));

        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(ApiResponse.<Map<String, String>>builder()
                        .success(false)
                        .message("Validation failed")
                        .data(errors)
                        .timestamp(LocalDateTime.now())
                        .build());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Void>> handleGeneric(Exception ex) {
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("Something went wrong"));
    }
}
```

---

## Chapter 10 — Controller كامل

```java
@RestController
@RequestMapping("/api/v1/courses/{courseId}/sections/{sectionId}/lessons")
@RequiredArgsConstructor
public class LessonController {

    private final LessonService lessonService;

    @PostMapping
    public ResponseEntity<ApiResponse<LessonResponse>> create(
            @PathVariable Long courseId,
            @PathVariable Long sectionId,
            @Valid @RequestBody LessonRequest request,
            @AuthenticationPrincipal UserDetails userDetails
    ) {
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(ApiResponse.success(
                        "Lesson created",
                        lessonService.createLesson(
                                courseId, sectionId,
                                request, userDetails.getUsername()
                        )
                ));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<LessonResponse>> getById(
            @PathVariable Long courseId,
            @PathVariable Long sectionId,
            @PathVariable Long id
    ) {
        return ResponseEntity.ok(ApiResponse.success(
                "Lesson retrieved",
                lessonService.getLessonById(courseId, sectionId, id)
        ));
    }

    @GetMapping
    public ResponseEntity<ApiResponse<PaginationResponse<LessonResponse>>> getAll(
            @PathVariable Long courseId,
            @PathVariable Long sectionId,
            @RequestParam(defaultValue = "0")  int page,
            @RequestParam(defaultValue = "10") int size
    ) {
        return ResponseEntity.ok(ApiResponse.success(
                "Lessons retrieved",
                lessonService.getLessonsBySection(courseId, sectionId, page, size)
        ));
    }

    @GetMapping("/preview")
    public ResponseEntity<ApiResponse<List<LessonResponse>>> getPreview(
            @PathVariable Long courseId,
            @PathVariable Long sectionId
    ) {
        return ResponseEntity.ok(ApiResponse.success(
                "Preview lessons retrieved",
                lessonService.getPreviewLessonsBySection(courseId, sectionId)
        ));
    }

    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<LessonResponse>> update(
            @PathVariable Long courseId,
            @PathVariable Long sectionId,
            @PathVariable Long id,
            @Valid @RequestBody LessonRequest request,
            @AuthenticationPrincipal UserDetails userDetails
    ) {
        return ResponseEntity.ok(ApiResponse.success(
                "Lesson updated",
                lessonService.updateLesson(
                        courseId, sectionId, id,
                        request, userDetails.getUsername()
                )
        ));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> delete(
            @PathVariable Long courseId,
            @PathVariable Long sectionId,
            @PathVariable Long id,
            @AuthenticationPrincipal UserDetails userDetails
    ) {
        lessonService.deleteLesson(
                courseId, sectionId, id, userDetails.getUsername()
        );
        return ResponseEntity.ok(ApiResponse.success("Lesson deleted", null));
    }
}
```

---

## الـ Flow كامل

```
HTTP Request
     │
     ▼
Security Filter (JWT)
     │
     ▼
Controller
  ├── @PathVariable
  ├── @RequestBody + @Valid
  └── @RequestParam
     │
     ▼
Validation
  ├── ✅ Pass → Service
  └── ❌ Fail → 400 Bad Request
     │
     ▼
Service (Business Logic)
  ├── ResourceNotFoundException  → 404
  ├── DuplicateResourceException → 409
  └── UnauthorizedException      → 403
     │
     ▼
Repository → DB
     │
     ▼
Mapper → Response DTO
     │
     ▼
ResponseEntity<ApiResponse<T>>
     │
     ▼
GlobalExceptionHandler (لو في error)
     │
     ▼
JSON Response
```

---

دلوقتي عندك الصورة كاملة ✅ عايز تطبق على مشروعك وتكتب الـ `LessonController`؟