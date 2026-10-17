
# Section 07: Spring Security & JWT Architecture

> **الملف ده مبني على مذكراتك الشاملة والمفصلة في `Courses_Pilot/NOTES/2-Auth exp.md`، ومجلدات `Spring Security/` و `Spring Security Last_Edition/` وسؤال الإنترفيو رقم 71 ورقم 11 و 12.**  
> يغطي: معمارية الـ Filter Chain، دورة حياة الـ Authentication، تفكيك الـ JWT، تشفير Passwords بـ BCrypt، الـ SecurityContext، والصلاحيات على مستوى الميثودز (`@PreAuthorize`).

---

### Q1: What is the difference between Authentication and Authorization?
**Level:** Junior  
**Related to:** Q2 (Architecture), Q7 (RBAC)

**Answer:**
مأخوذة من بداية مذكراتك بملف `2-Auth exp.md`:

#### 1. Authentication (المصادقة / التحقق من الهوية):
- **السؤال الأساسي:** "مين أنت؟" (**Who are you?**)
- هي عملية التحقق من أن المستخدم هو بالفعل الشخص الذي يدعيه (من خلال التحقق من الـ Credentials مثل الـ Email والباسورد، أو الـ OTP، أو البصمة).
- النتيجة: توليد **Identity مؤكدة** (تخزن في كائن `Authentication`).
- في حالة الفشل: يرجع السيرفر كود **`401 Unauthorized`**.

#### 2. Authorization (التفويض / فحص الصلاحيات):
- **السؤال الأساسي:** "مسموحلك تعمل إيه؟" (**What are you allowed to do?**)
- تحدث **بعد** اكتمال الـ Authentication بنجاح.
- هي عملية فحص هل هذا المستخدم المؤكد يمتلك الصلاحية (Role / Authority) لتنفيذ هذا الإجراء (مثل حذف مستخدم، تعديل منتج، أو قراءة تقرير مالي).
- النتيجة: السماح بتنفيذ العملية أو حجبها.
- في حالة الفشل: يرجع السيرفر كود **`403 Forbidden`**.

| وجه المقارنة | Authentication (AuthN) | Authorization (AuthZ) |
| :--- | :--- | :--- |
| **المعنى** | إثبات الهوية (Identity) | منح أو منع الصلاحية (Permissions) |
| **وقت الحدوث** | أولاً دائماً (عند الدخول) | ثانياً (عند محاولة تنفيذ أمر) |
| **HTTP Status Code عند الفشل** | `401 Unauthorized` | `403 Forbidden` |
| **المسؤول في Spring** | `AuthenticationManager` | `AccessDecisionManager` / `AuthorizationManager` |

**Short answer (30 seconds version):**
> "Authentication is verifying identity ('Who are you?'), failing with 401 Unauthorized. Authorization is checking permissions and roles ('What are you permitted to do?'), failing with 403 Forbidden. Authentication always precedes authorization."

---

### Q2: How does Spring Security work under the hood? Explain the Filter Chain.
**Level:** Mid  
**Related to:** Q1, Q5 (JwtAuthFilter), Q6 (SecurityFilterChain)

**Answer:**
مأخوذة من مذكراتك بملفات `Spring Security/0-FLOW.md` و `3.1-Security filter chain.md`:

Spring Security لا يبدأ عمله داخل الـ Controllers، بل يجلس كحارس عند بوابة السيرفر داخل الـ **Servlet Filters**.

#### معمارية الـ DelegatingFilterProxy و SecurityFilterChain:
1. يدخل الـ HTTP Request إلى الـ Servlet Container (Tomcat).
2. يقابل Tomcat فلتر واحد قياسي مسجل في السيرفلت اسمه: **`DelegatingFilterProxy`**.
3. هذا الفلتر لا يحتوي على منطق الحماية بنفسه؛ وظيفته الوحيدة هي **عمل جسر يربط عالم السيرفلت بعالم Spring IoC Container**.
4. يمرر الطلب لـ Spring Bean اسمه: **`FilterChainProxy`**.
5. يحتوي الـ `FilterChainProxy` على سلسلة فلاتر مرتبة بدقة تسمى **`SecurityFilterChain`**:

```
 HTTP Request (from Browser / App)
        │
        ▼
 Tomcat Filter Chain
        │
        ▼
 DelegatingFilterProxy (Bridge to Spring Context)
        │
        ▼
 FilterChainProxy (Manages Security Filters)
        │
        ▼
┌────────────────────────────────────────────────────────┐
│             SecurityFilterChain                        │
│                                                        │
│  1. CorsFilter (معالجة Cross-Origin)                   │
│  2. CsrfFilter (حماية CSRF - معطلة مع JWT)            │
│  3. JwtAuthenticationFilter (قراءة Bearer Token مخصص)   │
│  4. UsernamePasswordAuthenticationFilter               │
│  5. ExceptionTranslationFilter (ترجمة الأخطاء لـ 401/403)│
│  6. AuthorizationFilter (فحص هل الرابط مفتوح أم محمي)  │
└───────────────────────┬────────────────────────────────┘
                        │ لو نجحت جميع الفلاتر
                        ▼
                DispatcherServlet ──► Controller
```

**Short answer (30 seconds version):**
> "Spring Security operates as a series of servlet filters managed by the DelegatingFilterProxy and FilterChainProxy. Incoming requests traverse ordered filters (CORS, CSRF, JWT authentication, and authorization) before reaching the DispatcherServlet. If any filter rejects the credentials, the chain halts immediately with an error."

---

### Q3: What are `UserDetails`, `UserDetailsService`, and `AuthenticationManager`?
**Level:** Mid  
**Related to:** Q4 (Login Flow)

**Answer:**
هذه الكلاسات الثلاثة تمثل مثلث الـ Authentication في Spring Security، كما في كود مذكراتك:

#### 1. `UserDetails` Interface:
- هو الـ Adapter الذي يترجم كلاس الـ `User` الخاص بتطبيقك إلى النموذج الذي يفهمه Spring Security.
- يلزمك عمل `implements UserDetails` على كائن المستخدم، ويوفر ميثودز هامة:
  - `getAuthorities()`: يرجع قائمة الـ Roles/Permissions (مثل `ROLE_STUDENT`, `ROLE_ADMIN`).
  - `getPassword()`: الباسورد المشفر المخزن في الداتا بيز.
  - `getUsername()`: المعرف الفريد (الإيميل أو اسم المستخدم).
  - ميثودز فحص الحساب (`isAccountNonLocked`, `isEnabled`).

#### 2. `UserDetailsService` Interface:
- يحتوي على ميثود وحيدة:
  ```java
  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
  ```
- وظيفتها: الذهاب لقاعدة البيانات والبحث عن المستخدم بواسطة الإيميل وتحويله لـ `UserDetails`.

#### 3. `AuthenticationManager` & `DaoAuthenticationProvider`:
- الـ `AuthenticationManager` هو المنسق العام الذي ينفذ الـ Authentication.
- يفوض المهمة لـ `AuthenticationProvider` (عادة `DaoAuthenticationProvider`):
  1. يستدعي `userDetailsService.loadUserByUsername()`.
  2. يقارن الباسورد الخام القادم من الـ Login Request مع الباسورد المشفر القادم من الداتا بيز باستخدام `PasswordEncoder.matches()`.
  3. لو تطابقا: يرجع كائن `Authentication` كامل وموثوق!

**Short answer (30 seconds version):**
> "UserDetails is the contract representing the authenticated principal and its granted authorities. UserDetailsService fetches this user data by username from the database. AuthenticationManager (via DaoAuthenticationProvider) orchestrates the authentication process by matching submitted credentials against the stored password using a PasswordEncoder."

---

### Q4: Explain the complete JWT Authentication Flow (Register, Login, and Protected Requests).
**Level:** Mid / Senior  
**Related to:** Q3, Q5

**Answer:**
مأخوذة نصاً من كود مذكراتك بملف `Courses_Pilot/NOTES/2-Auth exp.md`:

```
               ┌────────────────────────────────────────────────────────┐
               │             1. رحلة الـ Login (توليد الـ Token)        │
               └────────────────────────────────────────────────────────┘
  Client ──► POST /api/v1/auth/login { email, password }
                  │
                  ▼
         AuthService & AuthenticationManager
                  │ 1. فحص صحة الإيميل والباسورد المشفر بـ BCrypt
                  │ 2. لو صحيح ──► JwtService.generateToken(user)
                  ▼
  Client ◄── 200 OK { token: "eyJhbGciOiJIUzI1NiIsInR5c..." }


               ┌────────────────────────────────────────────────────────┐
               │             2. رحلة طلب محمي (Protected Resource)      │
               └────────────────────────────────────────────────────────┘
  Client ──► GET /api/v1/orders
             Header: Authorization: Bearer eyJhbGciOi...
                  │
                  ▼
        JwtAuthenticationFilter (OncePerRequestFilter)
                  │ 1. استخراج الـ Token من هيدر Authorization
                  │ 2. فك التشفير واستخراج الـ username (Email)
                  │ 3. فحص هل الـ Token منتهي الصلاحية؟ (Expiration check)
                  │ 4. تحميل بيانات المستخدم وتعيينها في الـ SecurityContext
                  ▼
        SecurityContextHolder.getContext().setAuthentication(auth)
                  │
                  ▼
        OrderController (يتم معالجة الطلب بنجاح لأن المستخدم موثوق)
```

**Short answer (30 seconds version):**
> "During login, credentials are verified by the AuthenticationManager; upon success, the server generates a signed stateless JWT returned to the client. For subsequent requests, the client attaches the JWT in the Authorization Bearer header, where a custom JwtAuthenticationFilter validates the signature, extracts the user details, and populates the SecurityContextHolder."

---

### Q5: How do you implement `JwtAuthenticationFilter`? Explain `OncePerRequestFilter`.
**Level:** Mid  
**Related to:** Q2, Q4

**Answer:**
من كود مذكراتك:

#### لماذا نرث `OncePerRequestFilter`؟
في بيئة الـ Servlets، قد يعاد توجيه الـ Request داخلياً (Forward or Include). الوراثة من `OncePerRequestFilter` تضمن بشكل قطعي أن هذا الفلتر سيعمل **مرة واحدة فقط لكل طلب وارد من العميل**؛ مما يمنع فحص الـ JWT مرتين هباءً.

#### الكود الكامل للفلتر:
```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            @NonNull HttpServletRequest request,
            @NonNull HttpServletResponse response,
            @NonNull FilterChain filterChain
    ) throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");
        final String jwt;
        final String userEmail;

        // 1. التحقق من وجود Bearer Token في الهيدر
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response); // مرر للفلتر التالي بدون مصادقة
            return;
        }

        jwt = authHeader.substring(7); // قص كلمة "Bearer "
        userEmail = jwtService.extractUsername(jwt);

        // 2. لو استخرجنا الإيميل والـ SecurityContext غير مسجل فيه مستخدم حالياً
        if (userEmail != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(userEmail);

            // 3. التحقق من صحة وصلاحية التوكن
            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                );
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                // 4. تسجيل المستخدم في الـ SecurityContext (هنا أصبح موثوقاً!)
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        // 5. متابعة بقية الفلاتر في السلسلة
        filterChain.doFilter(request, response);
    }
}
```

**Short answer (30 seconds version):**
> "JwtAuthenticationFilter extends OncePerRequestFilter to ensure single execution per request. It extracts the 'Bearer ' token from the Authorization header, validates its expiration and signature via JwtService, retrieves user authorities, and sets an authenticated UsernamePasswordAuthenticationToken into the SecurityContextHolder."

---

### Q6: SecurityConfig Configuration (Stateless Sessions & CSRF Disabling)
**Level:** Mid  
**Related to:** Q2, Q4

**Answer:**
مأخوذة من مذكراتك بملف `2-Auth exp.md`:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // تفعيل @PreAuthorize على الميثودز
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // 1. تعطيل حماية CSRF (لأننا نستخدم Stateless JWT ولَسنا معتمدين على Cookies)
            .csrf(AbstractHttpConfigurer::disable)
            
            // 2. إدارة الجلسة كـ Stateless (لا تحتفظ بـ HttpSession في الرام)
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // 3. تخصيص الـ Endpoints المفتوحة والمحمية
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            
            // 4. تفعيل الـ Authentication Provider وإضافة فلتر الـ JWT قبل فلاتر Spring الافتراضية
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

#### لماذا نعطل CSRF مع JWT؟ (سؤال إنترفيو شهير!):
- **CSRF (Cross-Site Request Forgery)** يحدث عندما يعتمد الموقع على **Browser Cookies** لحفظ الجلسة، فيستغل موقع خبيث إرسال المتصفح للـ Cookie تلقائياً.
- في حالة **JWT المعتمدة على الـ Headers (`Authorization: Bearer ...`)**، المتصفح **لا يرسل الهيدر تلقائياً أبداً**؛ بل يجب أن يكتبه كود الـ JavaScript عمداً في الـ Request. وبالتالي فالهجوم مستحيل، وتعطيل CSRF آمن تماماً وموصى به في الـ REST APIs عديمة الحالة (Stateless).

**Short answer (30 seconds version):**
> "In stateless JWT APIs, we configure SessionCreationPolicy.STATELESS to prevent server-side session memory consumption. We safely disable CSRF protection because JWT tokens sent via the Authorization header cannot be automatically forged by cross-site browser requests like cookies."

---

### Q7: Role-Based Access Control (RBAC): `@PreAuthorize` vs URL-based Authorization
**Level:** Mid  
**Related to:** Section 03 (Security Annotations)

**Answer:**
مأخوذة من مذكراتك في ملف `Courses_Pilot/NOTES/11-@PreAuthorized.md` و `12-# Authentication Parameter في Spring Security.md`:

#### 1. URL-based Security (في الـ `SecurityConfig`):
- ممتازة لحماية مسارات كاملة دفعة واحدة:
  ```java
  .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
  ```

#### 2. Method-level Security (`@PreAuthorize`):
- توفر حماية دقيقة جداً (**Fine-grained Access Control**) مع دعم **SpEL (Spring Expression Language)**.
- تسمح بفحص قيم الـ Parameters وربطها بالمستخدم الحالي!

```java
@RestController
@RequestMapping("/api/v1/profiles")
public class ProfileController {

    // مسموح فقط لـ ADMIN أو للمستخدم صاحب الحساب نفسه!
    @PreAuthorize("hasRole('ADMIN') or #username == authentication.principal.username")
    @PutMapping("/{username}")
    public ResponseEntity<Void> updateProfile(
            @PathVariable String username,
            @RequestBody UpdateProfileDto dto) {
        profileService.update(username, dto);
        return ResponseEntity.ok().build();
    }
}
```

#### كيف تستقبل المستخدم الحالي في الـ Controller Parameter؟ (من مذكراتك - ملف 12):
بدلاً من الاستخراج اليدوي للـ Token:
```java
@GetMapping("/me")
public ResponseEntity<UserDto> getCurrentUser(@AuthenticationPrincipal User currentUser) {
    return ResponseEntity.ok(userMapper.toDto(currentUser));
}
```

**Short answer (30 seconds version):**
> "@PreAuthorize enables fine-grained, declarative method-level authorization using SpEL. It supports role checks (hasRole) and dynamic parameter checks (comparing resource IDs with authentication.principal), while @AuthenticationPrincipal conveniently injects the authenticated user directly into controller methods."

---

### Q8: What is Password Hashing? Why BCrypt and not MD5/SHA-256?
**Level:** Junior / Mid  
**Related to:** Q3 (DaoAuthenticationProvider)

**Answer:**
تخزين الباسوردات كنص صريح (Plaintext) جريمة أمنية تعرض النظام للدمار لو تم اختراق قاعدة البيانات.

#### لماذا نستخدم BCrypt وليس خوارزميات مثل MD5 أو SHA-256؟
1. **MD5 و SHA-256 خوارزميات سريعة جداً (Fast Hashing):**
   - صُممت للتحقق من سلامة الملفات وليس للباسوردات.
   - المهاجم يستطيع تجربة **مليارات الباسوردات في الثانية** باستخدام كروت الشاشة (GPUs) وتفكيك الباسورد عبر الـ Rainbow Tables وهجمات الـ Brute Force.
2. **BCrypt خوارزمية بطيئة عمداً (Slow / Adaptive Hash):**
   - تحتوي على **Work Factor (Cost Factor)**: يمكنك تحديد درجة الصعوبة؛ مثلاً جعل التشفير يستغرق 100ms للعملية الواحدة.
   - هذا التأخير غير محسوس للمستخدم العادي عند تسجيل الدخول، ولكنه يجعل من المستحيل رياضياً على المخترق تجربة ملايين الاحتمالات.
3. **توليد Salt تلقائي (Automatic Salting):**
   - يولد BCrypt قيمة عشوائية (Salt) فريدة لكل مستخدم ويدمجها داخل الـ Hash الناتج (`$2a$10$...`).
   - النتيجة: لو اشترك مستخدمان بنفس الباسورد تماماً `"password123"`، سيكون الـ Hash الخاص بكل واحد منهما في قاعدة البيانات **مختلفاً تماماً**؛ مما يدمر الـ Rainbow Tables بالكامل!

**Short answer (30 seconds version):**
> "Fast hashes like MD5 and SHA-256 are susceptible to GPU-accelerated brute-force and rainbow table attacks. BCrypt is an adaptive, slow hashing algorithm with built-in unique salting and a configurable work factor, making brute-force computation mathematically unfeasible."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *How do you handle JWT Token Revocation / Blacklisting on Logout? (Redis blacklist with TTL matching token expiration).*
2. *What is the difference between Access Token and Refresh Token? (Access Token has short lifespan e.g. 15 mins; Refresh Token is stored securely to rotate access tokens without forcing re-login).*
3. *What is `SecurityContextHolder.MODE_INHERITABLETHREADLOCAL`? (Allows child threads in async operations to inherit the security context).*
4. *How does OAuth2 / OpenID Connect (OIDC) integrate with Spring Security?*

---

## 🔗 System Design Connection: Stateless Auth, Redis Blacklists & Token Rotation

### Real-world System Design Application:
في الأنظمة الموزعة الضخمة (High-Scale Microservices)، إدارة الـ Auth تتطلب حل مشكلتين:
1. **Stateless Scalability:**
   - بفضل الـ JWT، أي سيرفر في الـ Cluster يستطيع فحص صحة التوكن محلياً باستخدام الـ Secret Key بدون الحاجة لعمل Database Query في كل طلب (Zero DB hits for authentication!).
2. **The Invalidation Problem & Redis Blacklist:**
   - مشكلة الـ JWT الأزلية: التوكن سيبقى صالحاً حتى تاريخ انتهائه حتى لو تم اختراق المستخدم أو قام بتسجيل الخروج!
   - **الحل:** عند تسجيل الخروج (Logout)، نقوم بحفظ الـ `jti` (Token ID) في **Redis In-Memory Cache** مع تعيين TTL يطابق وقت انتهاء التوكن. يفحص الفلتر الـ Redis سريعاً (في أقل من 1ms) للتأكد أن التوكن لم يتم إبطاله.

```
 Client (Protected Request)
          │
          ▼
┌──────────────────────────────────────┐
│       Spring Boot Security Filter    │
│  1. Cryptographic Signature Check    │ ──► Local CPU (Fast!)
│  2. Check Redis Blacklist            │ ──► Redis (<1ms)
└──────────────────┬───────────────────┘
                   │ If Valid
                   ▼
       Controller / Business Logic
```
