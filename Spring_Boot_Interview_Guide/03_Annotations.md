# Section 03: The Master Annotations Reference

> **مرجع شامل لكافة الـ Annotations المذكورة في مذكراتك عبر الأقسام المختلفة (`Spring Core`, `Spring Boot`, `Courses_Pilot/NOTES`, `Security`, `JPA`, `Exception Handling`).**  
> كل Annotation مشروح بسطر عربي + إنجليزي، مكان الاستخدام، مثال كود عملي، ومقارنات شهيرة للإنترفيو في النهاية.

---

## 1. Stereotype Annotations (الهوية والـ Beans)

### `@Component`
- **المعنى:** تسجيل الكلاس كـ Spring Bean عام يديره الـ IoC Container.
- **المكان:** Class-level.
```java
@Component
public class EmailValidator { ... }
```

### `@Service`
- **المعنى:** تصنيف الكلاس كجزء من طبقة الـ Business Logic (يرث `@Component`).
- **المكان:** Class-level.
```java
@Service
public class OrderService { ... }
```

### `@Repository`
- **المعنى:** تصنيف الكلاس كـ Data Access Layer مع تفعيل الـ Automatic Exception Translation للـ `DataAccessException`.
- **المكان:** Class-level أو Interface (مع Spring Data).
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> { ... }
```

### `@Controller`
- **المعنى:** كلاس مسؤول عن استقبال الـ HTTP Requests وإرجاع View (HTML/JSP/Thymeleaf).
- **المكان:** Class-level.
```java
@Controller
public class WebPageController { ... }
```

### `@RestController`
- **المعنى:** يجمع `@Controller` و `@ResponseBody` لإرجاع البيانات مباشرة بتنسيق JSON أو XML للـ REST APIs.
- **المكان:** Class-level.
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserRestController { ... }
```

---

## 2. Configuration & Core Annotations

### `@Configuration`
- **المعنى:** يعلن أن الكلاس يحتوي على ميثودز لتعريف وإنشاء الـ Beans يدوياً عبر `@Bean`.
- **المكان:** Class-level.
```java
@Configuration
public class SecurityConfig { ... }
```

### `@Bean`
- **المعنى:** تسجيل كائن كـ Bean في الـ Spring Context (يُستخدم عادة مع مكتبات الـ Third-party الخارجية).
- **المكان:** Method-level داخل `@Configuration`.
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### `@Value`
- **المعنى:** حقن قيم من ملف الـ `application.yml` أو متغيرات البيئة في الحقول مباشرة.
- **المكان:** Field-level أو Parameter-level.
```java
@Value("${jwt.secret.key}")
private String secretKey;
```

### `@Scope`
- **المعنى:** تحديد نطاق ودورة حياة الـ Bean (`singleton`, `prototype`, `request`, `session`).
- **المكان:** Class-level أو على `@Bean` method.
```java
@Component
@Scope("prototype")
public class ReportGenerator { ... }
```

### `@Primary`
- **المعنى:** إعطاء أولوية افتراضية لـ Bean معين عندما يتوفر أكثر من Implementation لنفس الـ Interface.
- **المكان:** Class-level أو على `@Bean` method.
```java
@Component
@Primary
public class ElectricEngine implements Engine { ... }
```

### `@Qualifier`
- **المعنى:** تحديد اسم الـ Bean المراد حقنه بدقة لفض التضارب عند وجود خيارات متعددة.
- **المكان:** Constructor param أو Field-level.
```java
public Car(@Qualifier("petrolEngine") Engine engine) { ... }
```

### `@Profile`
- **المعنى:** تفعيل Bean أو Configuration فقط في بيئة عمل محددة (مثل `dev`, `prod`, `test`).
- **المكان:** Class-level أو Method-level.
```java
@Component
@Profile("dev")
public class MockPaymentGateway implements PaymentGateway { ... }
```

### `@Lazy` [ADDED - not in my notes]
- **المعنى:** تأخير إنشاء الـ Bean حتى يتم استدعاؤه لأول مرة بدلاً من إنشائه وقت الـ Startup.
- **المكان:** Class أو Field أو Method.
```java
@Component
@Lazy
public class HeavyReportService { ... }
```

---

## 3. Web & REST Annotations

### `@RequestMapping`
- **المعنى:** تحديد المسار العام (Base Path) ونوع الميثود للـ Controller أو الـ Endpoint.
- **المكان:** Class-level أو Method-level.
```java
@RequestMapping("/api/v1/products")
```

### `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
- **المعنى:** اختصارات مخصصة لتحديد HTTP Methods مباشرة.
- **المكان:** Method-level.
```java
@GetMapping("/{id}")
public ResponseEntity<ProductDto> getById(@PathVariable Long id) { ... }
```

### `@PathVariable`
- **المعنى:** استخراج قيمة متغيرة ممررة داخل الـ URL Path (`/users/{id}`).
- **المكان:** Parameter-level.
```java
@GetMapping("/{id}")
public UserDto getUser(@PathVariable("id") Long userId) { ... }
```

### `@RequestParam`
- **المعنى:** استخراج Query Parameters من الـ URL (`/users?page=1&size=10`).
- **المكان:** Parameter-level.
```java
@GetMapping
public List<UserDto> getUsers(@RequestParam(defaultValue = "0") int page) { ... }
```

### `@RequestBody`
- **المعنى:** قراءة الـ HTTP Request Body (JSON) وتحويله تلقائياً لـ Java Object عبر Jackson.
- **المكان:** Parameter-level.
```java
@PostMapping
public ResponseEntity<UserDto> create(@RequestBody CreateUserDto dto) { ... }
```

### `@ResponseBody`
- **المعنى:** توجيه القيمة المعادة من الميثود لتوضع مباشرة في الـ HTTP Response Body (كـ JSON/XML).
- **المكان:** Method-level أو Class-level.

### `@ResponseStatus`
- **المعنى:** تحديد HTTP Status Code ثابت يرجع عند اكتمال الميثود أو رمي Exception معين.
- **المكان:** Method-level أو Exception Class.
```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping
public UserDto create(...) { ... }
```

---

## 4. Validation Annotations (Jakarta / Hibernate Validator)

### `@Valid`
- **المعنى:** أمر القياسي (Jakarta Bean Validation) لتفعيل فحص الحقول على الـ Object الممرر.
- **المكان:** Parameter-level (غالباً بجانب `@RequestBody`).
```java
@PostMapping
public ResponseEntity<Void> register(@Valid @RequestBody RegisterRequest req) { ... }
```

### `@Validated`
- **المعنى:** نسخة متقدمة خاصة بـ Spring تدعم الـ Validation Groups والتحقق على مستوى كلاسات الـ Service أو الـ Controller.
- **المكان:** Class-level أو Parameter-level.
```java
@RestController
@Validated
public class UserController { ... }
```

### قيود التحقق الشائعة:
- **`@NotNull`:** الحقل لا يقبل `null` (يقبل النص الفارغ `""`).
- **`@NotEmpty`:** الحقل لا يقبل `null` ولا يقبل أن يكون طوله صفر `""` أو Collection فارغة.
- **`@NotBlank`:** الأقوى للنصوص: لا يقبل `null`، ولا `""`، ولا مسافات بيضاء فقط `"   "`.
- **`@Size(min = 3, max = 50)`:** تحديد طول النص أو حجم الـ Collection.
- **`@Min(value)` / `@Max(value)`:** تحديد الحد الأدنى والأقصى للأرقام.
- **`@Email`:** التأكد من صحة تنسيق البريد الإلكتروني.
- **`@Pattern(regexp = "...")`:** التحقق باستخدام Regular Expression مخصص.

---

## 5. Spring Data JPA & Hibernate Annotations

### `@Entity`
- **المعنى:** تعريف الكلاس كـ JPA Entity يقابله جدول في قاعدة البيانات.
- **المكان:** Class-level.
```java
@Entity
@Table(name = "users")
public class User { ... }
```

### `@Table`
- **المعنى:** تخصيص اسم الجدول والـ Unique Constraints أو الـ Indexes الخاصة به.
- **المكان:** Class-level.
```java
@Table(name = "users", uniqueConstraints = @UniqueConstraint(columnNames = "email"))
```

### `@Id` & `@GeneratedValue`
- **المعنى:** `@Id` يحدد الـ Primary Key، و `@GeneratedValue` يحدد استراتيجية التوليد التلقائي (مثل `IDENTITY`).
- **المكان:** Field-level.
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

### `@Column`
- **المعنى:** تخصيص اسم العمود، هل يقبل `null`، الطول، والـ Unique.
- **المكان:** Field-level.
```java
@Column(name = "user_email", nullable = false, unique = true, length = 150)
private String email;
```

### `@MappedSuperclass` (من مذكراتك - ملف 1):
- **المعنى:** كلاس أب مشترك يحمل حقولاً مكررة (مثل `id`, `createdAt`, `updatedAt`) ترثها الـ Entities بدون إنشاء جدول مستقل للأب.
- **المكان:** Class-level.
```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity { ... }
```

### `@EntityListeners` & `@CreatedDate` / `@LastModifiedDate`
- **المعنى:** تفعيل مراقبة وتحديث تواريخ الإنشاء والتعديل تلقائياً بواسطة JPA Auditing.
- **المكان:** Class-level و Field-level.
```java
@CreatedDate
@Column(updatable = false, nullable = false)
private LocalDateTime createdAt;
```

### علاقات الجداول:
- **`@OneToMany`:** علاقة واحد لمتعدد (افتراضياً `FetchType.LAZY` كما في مذكراتك).
- **`@ManyToOne`:** علاقة متعدد لواحد (افتراضياً `FetchType.EAGER`، والـ Best Practice تحويلها لـ `LAZY`).
- **`@ManyToMany`:** علاقة متعدد لمتعدد باستخدام جدول وسيط `@JoinTable`.
- **`@JoinColumn`:** تحديد اسم عمود الـ Foreign Key في الجدول.

### حلول مشاكل الأداء في JPA:
- **`@EntityGraph` (من مذكراتك - ملف 19 و 20):** يحدد العلاقات التي سيتم عمل Eager Fetch لها في كويري معين كـ `LEFT JOIN` لمنع مشكلة N+1.
- **`@BatchSize` (من مذكراتك - ملف 22):** يحدد حجم الدفعة لجلب العلاقات الكسولة بـ `IN (...)` بدلاً من جلبها واحدة تلو الأخرى.

### `@Transactional`
- **المعنى:** إدارة معاملات قاعدة البيانات (Transactions) آلياً (Commit عند النجاح، و Rollback عند حدوث Runtime Exception).
- **المكان:** Method-level أو Class-level.
```java
@Transactional(readOnly = true)
public UserDto getUserById(Long id) { ... }
```

---

## 6. Exception Handling Annotations

### `@ControllerAdvice` / `@RestControllerAdvice`
- **المعنى:** معالج أخطاء مركزي (Global Exception Handler) يعترض الأخطاء الصادرة من جميع الـ Controllers في التطبيق.
- **المكان:** Class-level.
```java
@RestControllerAdvice
public class GlobalExceptionHandler { ... }
```

### `@ExceptionHandler`
- **المعنى:** ربط ميثود معينة لمعالجة نوع استثناء محدد وتشكيل الـ Response المناسب.
- **المكان:** Method-level داخل الـ Advice.
```java
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<ApiResponse<Void>> handleNotFound(ResourceNotFoundException ex) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ApiResponse.error(ex.getMessage()));
}
```

---

## 7. Security Annotations

### `@EnableWebSecurity`
- **المعنى:** تفعيل إعدادات الحماية وتخصيص الـ `SecurityFilterChain`.
- **المكان:** Configuration Class.

### `@PreAuthorize` (من مذكراتك - ملف 11):
- **المعنى:** فحص صلاحيات المستخدم قبل تنفيذ الميثود مباشرة باستخدام تعبيرات SpEL.
- **المكان:** Method-level.
```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
@DeleteMapping("/{userId}")
public void deleteUser(@PathVariable Long userId) { ... }
```

### `@Secured` [ADDED - not in my notes]
- **المعنى:** طريقة أقدم لفحص الأدوار لا تدعم SpEL المعقد (مثل `@Secured("ROLE_ADMIN")`).

---

## 8. Lombok Annotations (الأكثر استخداماً في مذكراتك)

### `@RequiredArgsConstructor` (من مذكراتك - ملف 6):
- **المعنى:** يولد Constructor يحتوي فقط على الحقول المعرفة كـ `final`، وهو السلاح الأمثل للـ Constructor Injection مع Spring.

### `@Builder` & `@Builder.Default` (من مذكراتك - ملف 21 و 21.1 و 66):
- **المعنى:** توليد نمط الـ Builder. تنبيه خطير: لو عندك حقل له قيمة ابتدائية (Default value) واستخدمت `@Builder`، القيمة ستتحول لـ `null` ما لم تضع فوق الحقل `@Builder.Default`!
```java
@Builder
public class Order {
    @Builder.Default
    private OrderStatus status = OrderStatus.PENDING; // بدون Annotation دي هتبقى null!
}
```

### `@SuperBuilder` (من مذكراتك - ملف 15):
- **المعنى:** يدعم نمط الـ Builder مع الوراثة بين الـ Parent Class والـ Child Entity.

---

## 📊 أهم المقارنات السريعة للانترفيو

### 1. `@Controller` vs `@RestController`
| المعيار | `@Controller` | `@RestController` |
| :--- | :--- | :--- |
| **نوع الـ Response** | صفحة HTML أو View Template | بيانات خام JSON / XML في الـ Body |
| **المكونات** | كلاس `@Component` مستقل | `@Controller` + `@ResponseBody` مدمجان |
| **الاستخدام** | Monolithic SSR (Thymeleaf/JSP) | RESTful Web APIs |

### 2. `@RequestParam` vs `@PathVariable`
| المعيار | `@PathVariable` | `@RequestParam` |
| :--- | :--- | :--- |
| **شكل الـ URL** | `/api/users/42` | `/api/users?id=42&sort=asc` |
| **الاستخدام المثالي** | تحديد مورد رئيسي محدد (Resource Identifier) | الفلترة، الترتيب، والـ Pagination |
| **إلزامية القيمة** | إجباري ضمن مسار الرابط | اختياري ويمكن وضع `defaultValue` |

### 3. `@NotNull` vs `@NotEmpty` vs `@NotBlank`
| القيمة | `@NotNull` | `@NotEmpty` | `@NotBlank` |
| :--- | :---: | :---: | :---: |
| `null` | ❌ يرفض | ❌ يرفض | ❌ يرفض |
| `""` (نص فارغ) | ✅ يقبل | ❌ يرفض | ❌ يرفض |
| `"   "` (مسافات بيضاء) | ✅ يقبل | ✅ يقبل | ❌ يرفض |
| `"Ali"` | ✅ يقبل | ✅ يقبل | ✅ يقبل |

### 4. `@Valid` vs `@Validated`
| المعيار | `@Valid` | `@Validated` |
| :--- | :--- | :--- |
| **المصدر** | القياسي القياسي من Jakarta EE | خاص بـ Spring Framework |
| **دعم الـ Validation Groups** | لا يدعم المجموعات | ✅ يدعم تقسيم القواعد لمجموعات مختلفة |
| **مكان الاستخدام** | ميثود باراميترز وخصائص الكائنات | على مستوى الكلاس أو الميثودز لتفعيل AOP validation |
