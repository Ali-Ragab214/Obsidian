تمام 🔥 انت كده وصلت للمرحلة المهمة جدًا:  
**نضيف Authentication + Registration حقيقيين على مشروعك الحالي.**

أنا هابني الخطة على نفس Structure اللي ظاهر في الصورة عندك 👌  
(Controller – DTO – Mapper – Models – Repositories – Service – Exception)

وده هيبقى **Blueprint احترافي كامل** تقدر تطبقه خطوة خطوة.

---

# 🧠 أولاً: هنبني ايه بالظبط؟

هنعمل نظام كامل فيه:

- Register
    
- Login
    
- JWT Authentication (Stateless)
    
- Roles (USER / ADMIN)
    
- حماية الـ APIs
    
- Logout
    
- Refresh Token (اختياري MVP+)
    

---

# 1️⃣ هيكلة الفولدر الجديدة داخل مشروعك

هنضيف Packages جديدة فقط بدون كسر مشروعك.

داخل  
`com.notification.jpa`

أضف الفولدرات دي:

```
security
   ├── config
   │      SecurityConfig
   │      JwtAuthenticationFilter
   │      JwtService
   │
   ├── service
   │      CustomUserDetailsService
   │
   ├── auth
   │      AuthController
   │      AuthService
   │      AuthenticationRequest
   │      AuthenticationResponse
   │      RegisterRequest
```

وهنعدل داخل:

```
models
repositories
```

---

# 2️⃣ Dependencies مهمة في pom.xml

أضف:

```xml
<dependencies>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>

</dependencies>
```

---

# 3️⃣ Database Design (مهم جدًا)

هنضيف 3 جداول:

## User Entity

```java
@Entity
@Table(name = "users")
@Getter @Setter
public class User {

    @Id @GeneratedValue
    private Long id;

    @Column(unique = true)
    private String email;

    private String password;
    private String name;

    @Enumerated(EnumType.STRING)
    private Role role;
}
```

## Role Enum

```java
public enum Role {
    USER,
    ADMIN
}
```

---

## UserRepository

```java
public interface UserRepository extends JpaRepository<User,Long> {
    Optional<User> findByEmail(String email);
}
```

---

# 4️⃣ أهم جزء 🔐 SecurityConfig

ده قلب النظام كله.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtFilter;
    private final CustomUserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
          .csrf(csrf -> csrf.disable())
          .sessionManagement(session ->
              session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
          )
          .authorizeHttpRequests(auth -> auth
              .requestMatchers("/api/auth/**").permitAll()
              .requestMatchers("/api/admin/**").hasRole("ADMIN")
              .anyRequest().authenticated()
          )
          .userDetailsService(userDetailsService)
          .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### شرح سريع:

- Disable CSRF → لأننا REST API
    
- STATELESS → JWT بدون sessions
    
- permitAll → auth endpoints فقط
    
- addFilterBefore → تشغيل JWT filter قبل Spring Security
    

---

# 5️⃣ CustomUserDetailsService

Spring لازم يعرف يجيب المستخدم من DB.

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository repo;

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        User user = repo.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getEmail())
                .password(user.getPassword())
                .roles(user.getRole().name())
                .build();
    }
}
```

---

# 6️⃣ JWT Service

توليد التوكن.

```java
@Service
public class JwtService {

    private final String SECRET = "very-secret-key";

    public String generateToken(UserDetails user){
        return Jwts.builder()
                .setSubject(user.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis()+86400000))
                .signWith(SignatureAlgorithm.HS256, SECRET)
                .compact();
    }
}
```

---

# 7️⃣ JWT Filter

يقرأ التوكن من كل Request.

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final CustomUserDetailsService userService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {

        String header = request.getHeader("Authorization");

        if(header == null || !header.startsWith("Bearer ")){
            chain.doFilter(request,response);
            return;
        }

        String token = header.substring(7);
        String email = Jwts.parser()
                .setSigningKey("very-secret-key")
                .parseClaimsJws(token)
                .getBody().getSubject();

        UserDetails user = userService.loadUserByUsername(email);

        UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(
                        user,null,user.getAuthorities());

        SecurityContextHolder.getContext().setAuthentication(auth);

        chain.doFilter(request,response);
    }
}
```

---

# 8️⃣ Flow كامل: Registration 🔥

## RegisterRequest DTO

```java
@Data
public class RegisterRequest {
    private String email;
    private String password;
    private String name;
}
```

## AuthService (Register + Login)

```java
@Service
@RequiredArgsConstructor
public class AuthService {

    private final UserRepository repo;
    private final PasswordEncoder encoder;
    private final JwtService jwtService;
    private final AuthenticationManager authManager;

    public String register(RegisterRequest request){
        User user = new User();
        user.setEmail(request.getEmail());
        user.setPassword(encoder.encode(request.getPassword()));
        user.setName(request.getName());
        user.setRole(Role.USER);

        repo.save(user);
        return "User registered";
    }

    public String login(AuthenticationRequest request){

        authManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getEmail(),
                request.getPassword()
            )
        );

        User user = repo.findByEmail(request.getEmail()).orElseThrow();
        return jwtService.generateToken(
                new org.springframework.security.core.userdetails.User(
                        user.getEmail(),user.getPassword(),
                        List.of(new SimpleGrantedAuthority("ROLE_"+user.getRole()))
                )
        );
    }
}
```

---

## AuthController

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthService service;

    @PostMapping("/register")
    public String register(@RequestBody RegisterRequest request){
        return service.register(request);
    }

    @PostMapping("/login")
    public String login(@RequestBody AuthenticationRequest request){
        return service.login(request);
    }
}
```

---

# 9️⃣ Flow تسجيل الدخول

1️⃣ User يرسل email + password  
2️⃣ AuthenticationManager يتحقق  
3️⃣ لو صح → يتولد JWT  
4️⃣ يرجع التوكن للفرونت  
5️⃣ الفرونت يرسله في كل Request:

```
Authorization: Bearer TOKEN
```

---

# 🔟 Logout

في JWT مفيش logout حقيقي (Stateless)  
الفرونت يحذف التوكن فقط.

---

# 1️⃣1️⃣ تشغيل المشروع

1️⃣ شغل المشروع  
2️⃣ Postman:

### Register

```
POST /api/auth/register
```

### Login

```
POST /api/auth/login
```

هيجيلك JWT 🔥

### جرب endpoint محمي

```
GET /private
Authorization: Bearer TOKEN
```

---

# 🎯 MVP جاهز الآن

أنت كده عندك:

- Register
    
- Login
    
- JWT
    
- Roles
    
- Protected APIs
    

الخطوة الجاية الطبيعية:

- Refresh Token
    
- Email verification
    
- Forgot password
    

---

لو تحب نطبق الخطوة الجاية (Refresh Token + Role ADMIN + حماية AuthorController عندك) نكمل فورًا 👌