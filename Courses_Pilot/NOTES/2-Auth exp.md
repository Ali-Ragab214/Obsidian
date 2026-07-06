# دليل شامل: Spring Security في Skill Hub Learning Platform

سأشرح لك **كل جزء من نظام الأمان خطوة بخطوة** — وأوضح دوره، ليش موجود، ومترتب عليه إيه. في الآخر، أعطيك ملف منظّم تحطه في Obsidian.

---

## المحتويات
1. مقدمة عن Spring Security
2. الأجزاء الرئيسية في المشروع
3. تدفق Authentication (Login/Register)
4. تدفق Authorization (Roles & Permissions)
5. الأخطاء الشائعة والحلول
6. أمثلة عملية

---

## 1) مقدمة عن Spring Security

### ما هي Spring Security؟
- مكتبة Spring لـ **حماية التطبيق** من الوصول غير المصرح.
- تتعامل مع:
  - **Authentication (المصادقة)**: "من أنت؟" → هل البريد والكلمة المرور صحيحة؟
  - **Authorization (التفويض)**: "بتقدر تعمل إيه؟" → هل ليك صلاحية تدخل هنا؟
  - **CSRF Protection**: حماية من الهجمات
  - **Session Management**: إدارة الجلسات

---

## 2) الأجزاء الرئيسية في مشروعك

### 2.1) User Model (BaseEntity + UserDetails Interface)
**الملف**: `src/main/java/.../domain/models/User.java`

```
📄 User.java
├── يرث من BaseEntity (id, createdAt, updatedAt)
├── يُطبّق UserDetails Interface (من Spring Security)
├── الحقول:
│   ├── name: اسم المستخدم
│   ├── email: البريد
│   ├── password: الكلمة (مشفرة بـ BCrypt)
│   ├── enabled: هل الحساب مفعّل
│   └── role: الدور (Student/Instructor/Admin)
└── الـ Methods:
    ├── getAuthorities(): ترجع الأدوار
    ├── getUsername(): ترجع email
    ├── isEnabled(): هل الحساب فعّال
    └── ... (من UserDetails interface)
```

**ليش محتاجه؟**
- Spring Security بيستخدم UserDetails لـ:
  - تحميل بيانات المستخدم من DB
  - التحقق من الكلمة المرور
  - الحصول على الأدوار/الصلاحيات

---

### 2.2) DTO (Data Transfer Objects)
**الملفات**: 
- `RegisterRequest.java`
- `LoginRequest.java`
- `AuthResponse.java`
- `UserResponse.java`

```
📄 RegisterRequest (من الـ Client)
├── name: string
├── email: string
├── password: string
└── role: enum (Student/Instructor/Admin)

📄 LoginRequest (من الـ Client)
├── email: string
└── password: string

📄 AuthResponse (رد السيرفر)
├── token: JWT string
└── user: UserResponse {id, name, email, role}
```

**ليش محتاجهم؟**
- فصل الـ logic عن الـ API
- تحديد بيانات معينة ترسلها/تستقبلها
- @Valid للتحقق من البيانات

---

### 2.3) AuthService & AuthServiceImpl
**الملفات**: 
- `AuthService.java` (interface)
- `AuthServiceImpl.java` (implementation)

```java
// AuthService (Interface)
public interface AuthService {
    AuthResponse register(RegisterRequest request);
    AuthResponse login(LoginRequest request);
}

// AuthServiceImpl (Implementation)
@Service
@RequiredArgsConstructor
public class AuthServiceImpl implements AuthService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;
    private final UserMapper userMapper;
    
    @Override
    public AuthResponse register(RegisterRequest request) {
        // 1. التحقق: هل البريد موجود؟
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new IllegalStateException("Email already exists");
        }
        
        // 2. بناء كيان User جديد
        var user = User.builder()
                .name(request.getName())
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .role(request.getRole() != null ? request.getRole() : Role.Student)
                .enabled(true)
                .build();
        
        // 3. حفظ في DB
        userRepository.save(user);
        
        // 4. توليد JWT
        var token = jwtService.generateToken(user);
        
        // 5. تحويل User → UserResponse
        var userResponse = userMapper.toUserResponse(user);
        
        // 6. رد AuthResponse (token + user)
        return AuthResponse.builder()
                .token(token)
                .user(userResponse)
                .build();
    }
    
    @Override
    public AuthResponse login(LoginRequest request) {
        // 1. استخدام AuthenticationManager للتحقق
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getEmail(),
                        request.getPassword()
                )
        );
        
        // 2. إحضار User من DB
        var user = userRepository.findByEmail(request.getEmail())
                .orElseThrow(() -> new IllegalStateException("User not found"));
        
        // 3. توليد JWT
        var token = jwtService.generateToken(user);
        
        // 4. تحويل User → UserResponse
        var userResponse = userMapper.toUserResponse(user);
        
        // 5. رد AuthResponse
        return AuthResponse.builder()
                .token(token)
                .user(userResponse)
                .build();
    }
}
```

**ليش محتاجه؟**
- **منطق التسجيل و الدخول**
- فصل الـ business logic عن الـ controller
- تطبيق مسؤولية واحدة (Single Responsibility)

---

### 2.4) JWT Service (توليد & التحقق من التوكنات)
**الملفات**: 
- `JwtService.java` (interface)
- `JwtServiceImpl.java` (implementation)

```
JWT (JSON Web Token) = توكن رقمي يُثبت أن المستخدم مُصرّح

هيكل JWT:
Header.Payload.Signature

Header: {"alg":"HS256","typ":"JWT"}
Payload: {"sub":"ali@example.com","exp":1234567890}
Signature: HMAC256(header.payload, secret-key)
```

```java
@Service
public class JwtServiceImpl implements JwtService {
    
    @Value("${application.security.jwt.secret-key}")
    private String secretKey;  // مفتاح سري من application.properties
    
    @Value("${application.security.jwt.expiration}")
    private long expirationMs;  // مدة صلاحية التوكن (86400000 ms = 1 يوم)
    
    @Override
    public String generateToken(UserDetails userDetails) {
        // توليد JWT جديد
        return Jwts.builder()
                .subject(userDetails.getUsername())  // subject = email
                .issuedAt(new Date())  // متى تم الإنشاء
                .expiration(new Date(System.currentTimeMillis() + expirationMs))
                .signWith(getSigningKey())  // التوقيع بـ secret key
                .compact();  // تجميع الأجزاء الثلاث
    }
    
    @Override
    public String extractUsername(String token) {
        // استخراج email من التوكن
        return extractClaim(token, Claims::getSubject);
    }
    
    @Override
    public boolean isTokenValid(String token, UserDetails userDetails) {
        // التحقق من:
        // 1. subject (email) يطابق
        // 2. التوكن لم ينقضِ
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    // helpers
    private SecretKey getSigningKey() {
        // فك تشفير الـ secret key من base64
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

**ليش محتاجه؟**
- **توليد التوكنات** عند registration/login
- **التحقق من التوكنات** في الطلبات القادمة
- مدة صلاحية محددة تجعل التطبيق آمن أكتر

---

### 2.5) JWT Authentication Filter
**الملف**: `JwtAuthFilter.java`

```
هذا الـ Filter بيجري على **كل طلب** قبل ما يوصل للـ Controller
```

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {
    
    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {
        
        // 1. استخراج Authorization header
        final String authHeader = request.getHeader("Authorization");
        
        // 2. التحقق من أن الـ header يبدأ بـ "Bearer "
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);  // تخطى للـ filter التالي
            return;
        }
        
        // 3. استخراج التوكن (بدون "Bearer " prefix)
        final String jwt = authHeader.substring(7);  // skip "Bearer "
        
        // 4. استخراج email من التوكن
        final String userEmail = jwtService.extractUsername(jwt);
        
        // 5. إذا كان هناك email والمستخدم ما سجّل دخول بعد
        if (userEmail != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            
            // 6. تحميل بيانات المستخدم من DB
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(userEmail);
            
            // 7. التحقق من صحة التوكن
            if (jwtService.isTokenValid(jwt, userDetails)) {
                
                // 8. بناء authentication token
                var authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()  // الأدوار/الصلاحيات
                );
                
                // 9. ربط بيانات الطلب
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                // 10. حفظ في SecurityContext (مهم جداً)
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        // 11. الاستمرار للـ filter التالي
        filterChain.doFilter(request, response);
    }
}
```

**ليش محتاجه؟** 
- بيتحقق من **كل طلب** إن كان فيه توكن صحيح
- لو الـ request في Authorization header → يحمّل المستخدم في SecurityContext
- البقية من Spring Security بتفحص هل المستخدم عنده الصلاحيات المطلوبة

---

### 2.6) CustomUserDetailsService
**الملف**: `CustomUserDetailsService.java`

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        // البحث عن المستخدم بـ email في DB
        return userRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + email));
    }
}
```

**ليش محتاجه؟**
- Spring Security تحتاج طريقة لتحميل UserDetails من مصدر البيانات
- هنا بنتحمّل من Database عن طريق email

---

### 2.7) SecurityConfig
**الملف**: `SecurityConfig.java`

```
هذا الـ Configuration بيعرّف:
- أي مسارات محمية/عامة
- أي roles مطلوبة
- كيف بننتقل البيانات (JWT stateless)
- أي filters نستخدم
```

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final JwtAuthFilter jwtAuthFilter;
    private final CustomUserDetailsService customUserDetailsService;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                // 1. عطّل CSRF (لأننا stateless + JWT)
                .csrf(AbstractHttpConfigurer::disable)
                
                // 2. استخدم STATELESS sessions (ما في session cookies)
                .sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                
                // 3. قيّد الوصول للمسارات
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/v1/auth/**").permitAll()  // عام (register/login)
                        .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")  // إداري فقط
                        .requestMatchers("/api/v1/instructor/**").hasAnyRole("INSTRUCTOR", "ADMIN")
                        .anyRequest().authenticated()  // بقية المسارات محمية
                )
                
                // 4. أضف Authentication Provider
                .authenticationProvider(authenticationProvider())
                
                // 5. أضف JWT Filter قبل البيانات الأساسية
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public AuthenticationProvider authenticationProvider() {
        // بناء DaoAuthenticationProvider (يستخدم DB للتحقق)
        var provider = new DaoAuthenticationProvider(customUserDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        // مسؤول عن التحقق من البيانات
        return config.getAuthenticationManager();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        // تشفير الكلمات المرور بـ BCrypt (قوي جداً)
        return new BCryptPasswordEncoder();
    }
}
```

**ليش محتاجه؟**
- **التحكم الكامل** في أي مسار محمي وأي دور محتاج
- **Stateless**: لا توجد cookies، فقط JWT في headers
- **Filter Chain**: ترتيب الفحوصات

---

### 2.8) AuthController
**الملف**: `AuthController.java`

```java
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {
    
    private final AuthService authService;
    
    @PostMapping("/register")
    public ResponseEntity<ApiResponse<?>> register(@Valid @RequestBody RegisterRequest request) {
        var response = authService.register(request);
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(ApiResponse.success(response, "Registration successful", HttpStatus.CREATED));
    }
    
    @PostMapping("/login")
    public ResponseEntity<ApiResponse<?>> login(@Valid @RequestBody LoginRequest request) {
        var response = authService.login(request);
        return ResponseEntity.ok(ApiResponse.success(response, "Login successful"));
    }
}
```

**ليش محتاجه؟**
- **نقطة دخول الـ API** لـ register و login
- تستقبل الطلبات وتُرسلها لـ AuthService

---

### 2.9) GlobalExceptionHandler
**الملف**: `GlobalExceptionHandler.java`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(IllegalStateException.class)
    public ResponseEntity<ApiResponse<?>> handleIllegalState(IllegalStateException ex) {
        // عند "email exists" مثلاً
        return ResponseEntity.badRequest()
                .body(ApiResponse.error(ex.getMessage()));
    }
    
    @ExceptionHandler(BadCredentialsException.class)
    public ResponseEntity<ApiResponse<?>> handleBadCredentials(BadCredentialsException ex) {
        // كلمة مرور خاطئة
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(ApiResponse.error("Invalid email or password"));
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<?>> handleValidation(MethodArgumentNotValidException ex) {
        // بيانات غير صحيحة (email غير valid مثلاً)
        Map<String, String> errors = ex.getBindingResult().getFieldErrors().stream()
                .collect(Collectors.toMap(
                        FieldError::getField,
                        FieldError::getDefaultMessage,
                        (a, b) -> a
                ));
        return ResponseEntity.badRequest()
                .body(ApiResponse.error("Validation failed", errors));
    }
}
```

**ليش محتاجه؟**
- **معالجة الأخطاء موحدة** برد JSON منسّق
- رسائل خطأ واضحة للـ Client

---

## 3) تدفق Authentication (Login/Register)

### تدفق التسجيل (Register):
```
1. Client POST /api/v1/auth/register
   {
     "name": "Ali",
     "email": "ali@example.com",
     "password": "secret123",
     "role": "Student"
   }
   
2. AuthController يستقبل و يُرسل لـ AuthServiceImpl
   
3. AuthServiceImpl:
   ✓ التحقق: البريد موجود؟ → نعم = IllegalStateException
   ✓ إنشاء User object جديد
   ✓ تشفير الكلمة بـ BCryptPasswordEncoder
   ✓ حفظ في DB
   
4. JwtServiceImpl:
   ✓ توليد JWT token (مدة 1 يوم)
   
5. UserMapper:
   ✓ تحويل User → UserResponse (بدون password)
   
6. ترجع AuthResponse:
   {
     "token": "eyJhbGc...",
     "user": { "id": 1, "name": "Ali", "email": "...", "role": "Student" }
   }
   
7. Client يحفظ التوكن في localStorage/sessionStorage
```

### تدفق الدخول (Login):
```
1. Client POST /api/v1/auth/login
   {
     "email": "ali@example.com",
     "password": "secret123"
   }
   
2. AuthController → AuthServiceImpl
   
3. AuthServiceImpl:
   ✓ استخدام AuthenticationManager لـ التحقق:
     - البريد موجود؟
     - الكلمة تطابق بعد التشفير؟
   ✓ لو غلط → BadCredentialsException
   
4. إذا صحيح:
   ✓ إحضار User من DB
   ✓ توليد JWT token
   ✓ ترجع AuthResponse (نفس التسجيل)
   
5. Client يحفظ التوكن
```

---

## 4) تدفق Authorization (Roles & Permissions)

### ال Roles في المشروع:
```java
public enum Role {
    Student,      // الطلاب
    Instructor,   // المعلمون
    Admin         // الإداريون
}
```

### كيف تُستخدم الأدوار:

#### في SecurityConfig:
```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/v1/auth/**").permitAll()  // للجميع
    .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")  // إداري فقط
    .requestMatchers("/api/v1/instructor/**").hasAnyRole("INSTRUCTOR", "ADMIN")
    .anyRequest().authenticated()  // محتاج تسجيل دخول
)
```

#### في User.java (getAuthorities):
```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    // تحويل Role → GrantedAuthority (Spring Security format)
    return List.of(new SimpleGrantedAuthority("ROLE_" + role.name().toUpperCase()));
    // مثال: User.role = Student → "ROLE_STUDENT"
}
```

### لو بتعمل controller محمي:
```java
@RestController
@RequestMapping("/api/v1/instructor")
public class CourseController {
    
    @PostMapping("/courses")
    public ResponseEntity<?> createCourse(@Valid @RequestBody CourseRequest request) {
        // فقط INSTRUCTOR و ADMIN يقدرو يدخلو
        // Spring Security تفحص تلقائياً
        // ...
    }
    
    @GetMapping("/admin/stats")
    @PreAuthorize("hasRole('ADMIN')")  // اختياري: فحص صريح في الـ method
    public ResponseEntity<?> getStats() {
        // فقط ADMIN
        // ...
    }
}
```

### تدفق فحص الدور:
```
1. Client يرسل طلب مع Authorization header:
   GET /api/v1/instructor/courses
   Authorization: Bearer eyJhbGc...
   
2. JwtAuthFilter:
   ✓ استخراج التوكن
   ✓ استخراج email من التوكن
   ✓ تحميل User من DB
   ✓ بناء UsernamePasswordAuthenticationToken و فيه:
     - principal = UserDetails (فيه email و role)
     - authorities = [ROLE_INSTRUCTOR]  (من getAuthorities())
   ✓ حفظ في SecurityContextHolder
   
3. DispatcherServlet (Spring):
   ✓ فحص SecurityConfig:
     - المسار = /api/v1/instructor/**
     - المطلوب = hasAnyRole("INSTRUCTOR", "ADMIN")
     - المستخدم عنده "ROLE_INSTRUCTOR"؟ ✓ نعم
   
4. مسموح الدخول للـ Controller
```

---

## 5) الأخطاء الشائعة والحلول

| المشكلة | السبب | الحل |
|--------|------|-----|
| 401 Unauthorized بعد Login | التوكن مش محطوط في Authorization header صحيح | استخدم `Authorization: Bearer <token>` |
| 400 Email already exists | البريد موجود | سجّل بريد جديد |
| 401 Invalid credentials | الكلمة غلط | تأكد من الكلمة |
| 403 Forbidden | المستخدم ما عنده الدور المطلوب | روح لـ /api/v1/auth/login بحساب الدور الصح |
| Token expired | التوكن صلاحيته 1 يوم | اعمل login مرة تانية للحصول على توكن جديد |
| Validation error | البيانات غير صحيحة (email مش valid مثلاً) | تأكد من format البيانات قبل الإرسال |

---

## 6) أمثلة عملية

### مثال 1: Register
```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ahmed",
    "email": "ahmed@example.com",
    "password": "pass123456",
    "role": "Student"
  }'
```

**الرد (201):**
```json
{
  "success": true,
  "message": "Registration successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhaG1lZEBleGFtcGxlLmNvbSIsImlhdCI6MTY4NDAwMDAwMCwiZXhwIjoxNjg0MDg2NDAwfQ.xxx",
    "user": {
      "id": 1,
      "name": "Ahmed",
      "email": "ahmed@example.com",
      "role": "Student"
    }
  },
  "timestamp": "2026-05-24T10:00:00",
  "statusCode": 201
}
```

### مثال 2: Login
```bash
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ahmed@example.com",
    "password": "pass123456"
  }'
```

**الرد (200):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGc...",
    "user": {...}
  },
  "timestamp": "2026-05-24T10:05:00",
  "statusCode": 200
}
```

### مثال 3: استخدام التوكن في طلب محمي
```bash
TOKEN="eyJhbGc..."

curl -X GET http://localhost:8080/api/v1/courses/1 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"
```

---

## ملف منظّم للـ Obsidian

```markdown
# Spring Security في Skill Hub Learning Platform

## مقدمة سريعة
- **Authentication**: التحقق من هوية المستخدم (login/register)
- **Authorization**: التحقق من صلاحيات المستخدم (Roles)
- **JWT**: توكن رقمي بدلاً من Session cookies

---

## المكونات الرئيسية

### 1️⃣ User Model (التخزين)
- **الملف**: `domain/models/User.java`
- **يُطبّق**: `UserDetails` interface (Spring Security)
- **الحقول الأساسية**:
  - `email`: المعرّف الفريد
  - `password`: مشفرة بـ BCrypt
  - `role`: Student/Instructor/Admin
  - `enabled`: حالة الحساب
- **المسؤوليات**:
  - تخزين بيانات المستخدم
  - توفير الأدوار والصلاحيات

### 2️⃣ DTOs (النقل)
- **RegisterRequest**: البريد + الكلمة + الاسم + الدور
- **LoginRequest**: البريد + الكلمة
- **AuthResponse**: التوكن + بيانات المستخدم
- **UserResponse**: بدون حقل الكلمة

### 3️⃣ AuthService (المنطق)
**register()**:
  1. تحقق: هل البريد موجود؟
  2. شفّر الكلمة بـ BCrypt
  3. أنشئ User
  4. لو موجود → IllegalStateException
  5. ولّد JWT token
  6. ترجع AuthResponse

**login()**:
  1. تحقق: البريد + الكلمة صحيحة؟
  2. استخدم AuthenticationManager
  3. لو غلط → BadCredentialsException
  4. ولّد JWT token
  5. ترجع AuthResponse

### 4️⃣ JwtService (التوكنات)
**generateToken()**: 
  - Subject = email
  - Expiration = 1 يوم
  - توقيع بـ Secret Key

**isTokenValid()**: 
  - هل الـ Subject يطابق؟
  - هل التوكن منقضي؟

### 5️⃣ JwtAuthFilter (الفحص)
يجري على **كل طلب**:
  1. استخرج Authorization header
  2. تحقق من تنسيق "Bearer <token>"
  3. استخرج email من التوكن
  4. تحمّل UserDetails من DB
  5. تحقق من صحة التوكن
  6. حفظ في SecurityContext → بقية الطلب يعرف هوية المستخدم

### 6️⃣ CustomUserDetailsService (تحميل البيانات)
- تطبيق `UserDetailsService` interface
- دالة واحدة: `loadUserByUsername(email)`
- مسؤولة عن تحميل User من DB عند الحاجة

### 7️⃣ SecurityConfig (القوانين)
**Authorize**:
  - `/api/v1/auth/**`: للجميع (register/login)
  - `/api/v1/admin/**`: ADMIN فقط
  - `/api/v1/instructor/**`: INSTRUCTOR + ADMIN
  - الباقي: authenticated

**الميزات**:
  - CSRF disabled (stateless API)
  - STATELESS: لا sessions
  - JWT Filter added

### 8️⃣ AuthController (الواجهة)
- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`

### 9️⃣ GlobalExceptionHandler (الأخطاء)
- معالجة موحدة للأخطاء
- رد JSON منسّق

---

## الأدوار (Roles)

### الأدوار المتاحة:
```
Student    → طالب
Instructor → معلم
Admin      → إداري
```

### كيف تُستخدم؟
```java
// في SecurityConfig
.hasRole("ADMIN")  // يسمح فقط ADMIN
.hasAnyRole("INSTRUCTOR", "ADMIN")  // يسمح بـ واحد منهم

// في الـ Controller
@PreAuthorize("hasRole('ADMIN')")  // فحص في الـ method
public ResponseEntity<?> deleteUser() { ... }
```

### في getAuthorities():
```java
// User.role = Student
"ROLE_STUDENT"  // Spring Security format
```

---

## تدفق الفحص (Request Flow)

### عند Startup:
```
1. Spring تحمّل SecurityConfig
2. تبني SecurityFilterChain
3. تسجّل JwtAuthFilter قبل UsernamePasswordAuthenticationFilter
```

### عند طلب جديد:
```
1. JwtAuthFilter.doFilterInternal()
   ├─ استخرج Authorization header
   ├─ تحقق من "Bearer " prefix
   ├─ استخرج التوكن
   ├─ استخرج email من التوكن
   ├─ تحمّل UserDetails من DB
   └─ حفظ في SecurityContext

2. DispatcherServlet
   ├─ تحقق من SecurityConfig rules
   ├─ فحص الدور إن لزم
   └─ سمح/ارفض الطلب

3. Controller يستلم الطلب
   └─ يقدر يستخدم SecurityContextHolder.getContext()
      للحصول على بيانات المستخدم
```

---

## استخدام في الـ Code

### للحصول على المستخدم الحالي:
```java
@GetMapping("/profile")
public ResponseEntity<?> getProfile() {
    var authentication = SecurityContextHolder.getContext().getAuthentication();
    var userDetails = (UserDetails) authentication.getPrincipal();
    String email = userDetails.getUsername();
    // ...
}
```

### للتحقق من دور معين:
```java
@PostMapping("/admin-only")
@PreAuthorize("hasRole('ADMIN')")  // أو @Secured("ROLE_ADMIN")
public ResponseEntity<?> adminAction() {
    //...
}
```

### في أي service:
```java
@Service
public class CourseService {
    public void createCourse(CourseRequest request) {
        var authentication = SecurityContextHolder.getContext().getAuthentication();
        var user = (User) authentication.getPrincipal();
        // تحقق: هل INSTRUCTOR؟
        if (!user.getRole().equals(Role.Instructor)) {
            throw new RuntimeException("Not authorized");
        }
    }
}
```

---

## configurations في application.properties

```properties
# JWT Configuration
application.security.jwt.secret-key=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
application.security.jwt.expiration=86400000  # 1 يوم (ms)
```

---

## الأخطاء الشائعة

| رمز | المعنى | الحل |
|-----|--------|-----|
| 401 | Unauthorized (بدون توكن أو توكن خاطئ) | أرسل Authorization: Bearer <token> |
| 403 | Forbidden (عندك توكن لكن ما عندك الدور) | استخدم حساب فيه الدور المطلوب |
| 400 | Bad Request (بيانات غلط) | تأكد من format البيانات |

---

## الخلاصة

```
JWT Flow:
Register → توكن → حفظ محليًا
   ↓
كل طلب → Authorization: Bearer <token>
   ↓
JwtAuthFilter → فحص صحة التوكن
   ↓
SecurityContext → بيانات المستخدم + أدواره
   ↓
Controller → معرفة هوية المستخدم و دوره
```

---

## مصادر مهمة

- `SecurityConfig.java`: القوانين
- `JwtAuthFilter.java`: فحص كل طلب
- `AuthService.java`: منطق register/login
- `User.java`: تطبيق UserDetails
```

---

## الخلاصة النهائية

أنت الآن عارف:
✅ كل مكون موجود ليش  
✅ كل واحد بيعمل إيه  
✅ الـ flow بين المكونات  
✅ كيف تستخدم الأدوار  
✅ الأخطاء الشائعة  

هل تحتاج توضيح أي حاجة أخرى أو عايز أضيف حاجة على الملف؟