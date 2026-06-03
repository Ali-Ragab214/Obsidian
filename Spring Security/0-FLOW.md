![[Pasted image 20260531202208.png]]الخريطة اتقسمت لمرحلتين واضحين:

**مرحلة ١ — Login:** المستخدم بيبعت البيانات، الـ AuthenticationManager يوجهها للـ AuthenticationProvider اللي بيتحقق منها عن طريق الـ UserDetailsService والـ PasswordEncoder، وبعدين بيحفظها في الـ SecurityContext ويرجع JWT.

**مرحلة ٢ — كل Request:** المستخدم بيبعت الـ JWT في الـ Header، الـ JwtAuthFilter بيتحقق منه، يحمّل الـ SecurityContext، والـ Request يوصل للـ Controller.

كل box قابل للضغط لو عايز تعمق في أي جزء منهم.

لو هتعمل **Login + Register + Roles** بطريقة احترافية في Spring Boot 3 / Spring Security 6، ومشروعك مش Clean Architecture معقد جدًا، فده Structure ممتاز ومستخدم في شركات كتير:

```text
src/main/java
│
├── config
│   └── SecurityConfig.java
│
├── auth
│   ├── AuthenticationController.java
│   ├── AuthenticationService.java
│   ├── AuthenticationRequest.java
│   ├── AuthenticationResponse.java
│   ├── RegisterRequest.java
│   └── JwtService.java
│
├── user
│   ├── User.java
│   ├── UserRepository.java
│   ├── Role.java
│   └── UserService.java
│
├── security
│   ├── JwtAuthenticationFilter.java
│   └── CustomUserDetailsService.java
│
└── AmigosSecurityApplication.java
```

---

# 1. User Entity

```java
@Entity
public class User {

    @Id
    @GeneratedValue
    private Integer id;

    private String firstName;

    private String lastName;

    private String email;

    private String password;

    @Enumerated(EnumType.STRING)
    private Role role;
}
```

---

# 2. Role Enum

```java
public enum Role {
    USER,
    ADMIN
}
```

بعد كده في الـ Security:

```java
.hasRole("ADMIN")
.hasRole("USER")
```

---

# 3. UserRepository

```java
@Repository
public interface UserRepository
        extends JpaRepository<User,Integer> {

    Optional<User> findByEmail(String email);
}
```

---

# 4. RegisterRequest DTO

```java
public class RegisterRequest {

    private String firstName;

    private String lastName;

    private String email;

    private String password;
}
```

---

# 5. AuthenticationRequest DTO

```java
public class AuthenticationRequest {

    private String email;

    private String password;
}
```

---

# 6. AuthenticationResponse DTO

```java
public class AuthenticationResponse {

    private String token;
}
```

---

# 7. AuthenticationService

هنا بيحصل:

### register()

```java
public AuthenticationResponse register(
        RegisterRequest request)
```

المهام:

- التحقق من عدم وجود المستخدم
    
- تشفير الباسورد
    
- حفظ المستخدم
    
- إنشاء JWT
    

---

### authenticate()

```java
public AuthenticationResponse authenticate(
        AuthenticationRequest request)
```

المهام:

- التحقق من الإيميل والباسورد
    
- إنشاء JWT
    

---

# 8. AuthenticationController

```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthenticationController {
}
```

Endpoints:

```java
POST /register

POST /authenticate
```

---

# 9. JwtService

مسؤول عن:

```java
generateToken()

extractUsername()

isTokenValid()
```

---

# 10. JwtAuthenticationFilter

هذا أهم Filter في المشروع.

في كل Request:

```text
Authorization: Bearer xxxxx
```

يقوم بـ:

1. قراءة التوكن
    
2. استخراج الإيميل
    
3. تحميل المستخدم
    
4. التحقق من التوكن
    
5. وضع المستخدم داخل SecurityContext
    

---

# 11. CustomUserDetailsService

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {
}
```

```java
loadUserByUsername(String email)
```

تجيب المستخدم من قاعدة البيانات.

---

# 12. SecurityConfig

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
}
```

يحتوي على:

### SecurityFilterChain

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http)
```

### PasswordEncoder

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### AuthenticationProvider

```java
@Bean
AuthenticationProvider authenticationProvider()
```

### AuthenticationManager

```java
@Bean
AuthenticationManager authenticationManager(
        AuthenticationConfiguration config)
```

---

# شكل الفلو النهائي

```text
Register
   |
   v
Controller
   |
   v
AuthenticationService
   |
   v
UserRepository
   |
   v
Database
   |
   v
JWT Generated
```

---

```text
Login
   |
   v
Controller
   |
   v
AuthenticationService
   |
   v
AuthenticationManager
   |
   v
UserDetailsService
   |
   v
JWT Generated
```

---

لو هدفك تتعلم Spring Security بشكل احترافي في 2026، فأنا أنصحك تبني المشروع بهذا الترتيب:

1. User Entity
    
2. Role Enum
    
3. UserRepository
    
4. SecurityConfig
    
5. PasswordEncoder
    
6. CustomUserDetailsService
    
7. Register Endpoint
    
8. Login Endpoint
    
9. JWT Service
    
10. JWT Filter
    
11. Role-Based Authorization (`ADMIN`, `USER`)
    

وده تقريبًا نفس الهيكل اللي هتلاقيه في أغلب كورسات Spring Security الحديثة بعد تحديثها لـ Spring Security 6.
