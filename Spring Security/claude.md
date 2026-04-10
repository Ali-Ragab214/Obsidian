هشرحلك كل حاجة خطوة بخطوة بالعربي. المشروع ده كبير فهقسمه على أجزاء واضحة.

---

## أولاً — الـ Dependencies في pom.xml

```xml
<dependencies>
    <!-- Spring Boot Web — عشان نعمل REST APIs -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Security — ده اللي هيتكلم عنه الشرح كله -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JPA — للتعامل مع الـ Database -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- JWT — عشان نعمل Tokens -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
    </dependency>

    <!-- Validation — للـ @Valid و @NotBlank وغيرها -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Lombok — بيوفر كتابة getters/setters/constructors -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

---

## ثانياً — الـ Folder Structure الجديدة

```
com.notification.jpa/
├── controller/
│   ├── AuthController.java       ← جديد
│   ├── AuthorController.java
│   └── PublisherController.java
├── dto/
│   ├── AuthorDTO.java
│   ├── AuthorRequest.java
│   ├── PublisherDTO.java
│   ├── PublisherRequest.java
│   ├── RegisterRequest.java      ← جديد
│   ├── LoginRequest.java         ← جديد
│   └── AuthResponse.java         ← جديد
├── models/
│   ├── User.java                 ← جديد
│   └── Role.java                 ← جديد (Enum)
├── repositories/
│   ├── AuthorRepository.java
│   ├── PublisherRepository.java
│   └── UserRepository.java       ← جديد
├── service/
│   ├── AuthService.java          ← جديد
│   ├── AuthorService.java
│   ├── PublisherService.java
│   └── impl/
│       ├── AuthServiceImpl.java  ← جديد
│       ├── AuthorServiceImpl.java
│       └── PublisherServiceImpl.java
├── security/                     ← جديد (فولدر كامل)
│   ├── SecurityConfig.java
│   ├── JwtService.java
│   ├── JwtAuthFilter.java
│   └── CustomUserDetailsService.java
├── exception/
│   └── (موجود)
└── JpaApplication.java
```

---

## ثالثاً — الـ Database Schema

```sql
-- جدول الـ Users
CREATE TABLE users (
    id         BIGINT AUTO_INCREMENT PRIMARY KEY,
    username   VARCHAR(50)  UNIQUE NOT NULL,
    email      VARCHAR(100) UNIQUE NOT NULL,
    password   VARCHAR(255) NOT NULL,      -- هيتخزن مشفر دايماً
    role       VARCHAR(20)  NOT NULL       -- ADMIN أو USER
);
```

---

## رابعاً — الـ Models

### `User.java`

```java
package com.notification.jpa.models;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;
import java.util.List;

@Entity
// @Entity → بتقول لـ JPA إن الكلاس ده جدول في الـ Database

@Table(name = "users")
// @Table → بتحدد اسم الجدول في الـ DB

@Data
// @Data من Lombok → بتعمل getters وsetters وtoString تلقائياً

@Builder
// @Builder → بتخليك تعمل object بطريقة builder pattern
// مثال: User.builder().email("a@b.com").build()

@NoArgsConstructor
// بتعمل Constructor فاضي تلقائياً — JPA محتاجه

@AllArgsConstructor
// بتعمل Constructor بكل الـ fields تلقائياً

public class User implements UserDetails {
// implements UserDetails → ده Contract من Spring Security
// بيقوله "الكلاس ده هو اللي بيمثل المستخدم عندي"
// لازم تطبق مجموعة methods معينة

    @Id
    // @Id → ده الـ Primary Key في الجدول

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // @GeneratedValue → قيمته بتتعمل تلقائياً (AUTO INCREMENT)

    private Long id;

    @Column(unique = true, nullable = false)
    // unique = true → مينفعش يتكرر في الجدول
    // nullable = false → إجباري، مش ممكن يبقى فاضي

    private String username;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;
    // الـ password هيتخزن مشفر — مش plaintext أبداً

    @Enumerated(EnumType.STRING)
    // @Enumerated → بيخزن الـ Enum كـ String في الجدول
    // يعني هيتخزن "ADMIN" أو "USER" مش 0 أو 1

    private Role role;

    // ===== Methods من UserDetails — Spring Security محتاجها =====

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // بترجع الـ Roles بتاعة المستخدم
        // SimpleGrantedAuthority → بتحول الـ Role لـ String Spring يفهمه
        return List.of(new SimpleGrantedAuthority(role.name()));
    }

    @Override
    public String getPassword() {
        return password;
        // Spring Security بيستخدم الـ method دي عشان يجيب الـ password المشفر
    }

    @Override
    public String getUsername() {
        return email;
        // بنرجع الـ email كـ username
        // يعني الـ login هيبقى بالـ email مش الـ username
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
        // الحساب مش منتهي؟ true = شغال
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
        // الحساب مش مقفول؟ true = مش مقفول
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
        // الـ password مش منتهي؟ true = لسه شغال
    }

    @Override
    public boolean isEnabled() {
        return true;
        // الحساب مفعّل؟ true = مفعّل
    }
}
```

### `Role.java`

```java
package com.notification.jpa.models;

public enum Role {
    USER,   // مستخدم عادي
    ADMIN   // مدير
    // Enum → يعني القيمة بتاخد واحدة بس من دول
}
```

---

## خامساً — الـ DTOs

### `RegisterRequest.java`

```java
package com.notification.jpa.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

@Data
public class RegisterRequest {

    @NotBlank(message = "Username is required")
    // @NotBlank → مش هيقبل null ولا string فاضية ""
    private String username;

    @Email(message = "Invalid email format")
    // @Email → بتتحقق إن الـ email بالصيغة الصح user@domain.com
    @NotBlank(message = "Email is required")
    private String email;

    @Size(min = 6, message = "Password must be at least 6 characters")
    // @Size → بتحدد الحد الأدنى والأقصى للحروف
    @NotBlank(message = "Password is required")
    private String password;
}
```

### `LoginRequest.java`

```java
package com.notification.jpa.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class LoginRequest {

    @Email
    @NotBlank
    private String email;

    @NotBlank
    private String password;
}
```

### `AuthResponse.java`

```java
package com.notification.jpa.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class AuthResponse {
    private String token;
    // ده الـ JWT Token اللي هيترجع للـ Client بعد الـ Login أو Register
    // الـ Client هيبعت الـ Token ده مع كل request تاني
}
```

---

## سادساً — الـ UserRepository

```java
package com.notification.jpa.repositories;

import com.notification.jpa.models.User;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
// JpaRepository → بيديك كل الـ CRUD operations جاهزة
// <User, Long> → User هو الـ Entity، Long هو نوع الـ ID

    Optional<User> findByEmail(String email);
    // Spring بيولد الـ Query تلقائياً من اسم الـ method
    // يعني هيعمل: SELECT * FROM users WHERE email = ?
    // Optional → يعني ممكن يرجع user أو null (بدون NullPointerException)
}
```

---

## سابعاً — الـ Security Classes

### `JwtService.java`

```java
package com.notification.jpa.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;
import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Service
// @Service → بيقول لـ Spring "الكلاس ده component، اعمله bean"
// Bean → يعني Spring هو اللي بيعمله object ويديه لأي حد محتاجه

public class JwtService {

    // الـ Secret Key → كلمة سرية بنستخدمها عشان نوقع الـ Token
    // مهم جداً تبقى طويلة وسرية ومش منحطهاش في الكود مباشرة
    private static final String SECRET_KEY =
        "404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970";
    // في الـ Production: ضيفها في application.properties

    // ===== Token Generation =====

    public String generateToken(UserDetails userDetails) {
        // UserDetails → الـ Interface اللي بيمثل المستخدم في Spring Security
        return generateToken(new HashMap<>(), userDetails);
        // بنوجه لـ method ثانية بـ claims فاضية
    }

    public String generateToken(
        Map<String, Object> extraClaims,
        UserDetails userDetails
    ) {
        // extraClaims → معلومات إضافية بتتحط جوه الـ Token (زي الـ role مثلاً)
        return Jwts
            .builder()
            // builder → بنبني الـ Token خطوة خطوة

            .setClaims(extraClaims)
            // بنحط الـ extra claims

            .setSubject(userDetails.getUsername())
            // setSubject → بنحط الـ email (اللي بيمثل المستخدم) جوه الـ Token

            .setIssuedAt(new Date(System.currentTimeMillis()))
            // setIssuedAt → وقت عمل الـ Token

            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24))
            // setExpiration → وقت انتهاء الـ Token
            // 1000ms * 60s * 60m * 24h = يوم كامل

            .signWith(getSignInKey(), SignatureAlgorithm.HS256)
            // signWith → بنوقع الـ Token بالـ Secret Key
            // HS256 → نوع خوارزمية التشفير (HMAC SHA-256)

            .compact();
            // compact → بيحول كل ده لـ String جاهز للإرسال
    }

    // ===== Token Validation =====

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        // بنجيب الـ email من جوه الـ Token

        return (username.equals(userDetails.getUsername()))
            && !isTokenExpired(token);
        // بنتحقق إن الـ email في الـ Token = الـ user الحالي
        // وإن الـ Token مش منتهي
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
        // before(new Date()) → يعني وقت الانتهاء قبل دلوقتي = منتهي
    }

    // ===== Extract Information from Token =====

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
        // Claims::getSubject → بتجيب الـ Subject (الـ email) من الـ Token
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        // Function<Claims, T> → ده Generic — بيقبل أي function بتاخد Claims وترجع أي type
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts
            .parserBuilder()
            .setSigningKey(getSignInKey())
            // بنستخدم نفس الـ Secret Key عشان نفك الـ Token
            .build()
            .parseClaimsJws(token)
            // parseClaimsJws → بيفك الـ Token ويتحقق من صحته
            .getBody();
            // getBody → بيجيب محتوى الـ Token
    }

    private Key getSignInKey() {
        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);
        // BASE64.decode → بيحول الـ String ل bytes
        return Keys.hmacShaKeyFor(keyBytes);
        // hmacShaKeyFor → بيعمل Key من الـ bytes دي
    }
}
```

### `CustomUserDetailsService.java`

```java
package com.notification.jpa.security;

import com.notification.jpa.repositories.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
// @RequiredArgsConstructor من Lombok → بتعمل Constructor للـ final fields تلقائياً
// بدلاً من كتابة @Autowired أو Constructor يدوي

public class CustomUserDetailsService implements UserDetailsService {
// UserDetailsService → Interface من Spring Security
// بيقول لـ Spring "لما تحتاج تجيب المستخدم من الـ DB، استخدم الكلاس ده"

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        // loadUserByUsername → Spring Security بيستدعي الـ method دي تلقائياً
        // وقت الـ authentication عشان يجيب بيانات المستخدم من الـ DB

        return userRepository.findByEmail(email)
            .orElseThrow(() ->
                new UsernameNotFoundException("User not found with email: " + email)
                // UsernameNotFoundException → Exception جاهزة من Spring Security
                // لو المستخدم مش موجود
            );
    }
}
```

### `JwtAuthFilter.java`

```java
package com.notification.jpa.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.lang.NonNull;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import java.io.IOException;

@Component
// @Component → زي @Service بس أعم — بيقول لـ Spring اعمل Bean من الكلاس ده

public class JwtAuthFilter extends OncePerRequestFilter {
// OncePerRequestFilter → بيضمن إن الـ Filter ده بيشتغل مرة واحدة بس لكل Request
// Filter → حاجة بتشتغل قبل ما الـ Request توصل للـ Controller

    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;

    public JwtAuthFilter(JwtService jwtService,
                         CustomUserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(
        @NonNull HttpServletRequest request,
        // HttpServletRequest → الـ Request اللي جاية من الـ Client

        @NonNull HttpServletResponse response,
        // HttpServletResponse → الـ Response اللي هترجع للـ Client

        @NonNull FilterChain filterChain
        // FilterChain → سلسلة الـ Filters، بنستدعيها عشان نكمل للـ Filter التاني
    ) throws ServletException, IOException {

        // 1) جيب الـ Authorization Header من الـ Request
        final String authHeader = request.getHeader("Authorization");
        // Authorization Header → الـ Client بيبعت فيه: "Bearer eyJhbGc..."
        // Bearer → كلمة ثابتة قبل الـ Token

        final String jwt;
        final String userEmail;

        // 2) لو مفيش Header أو مش بيبدأ بـ "Bearer "
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            // نعدي الـ Request للـ Filter التاني بدون ما نعمل حاجة
            return;
        }

        // 3) جيب الـ Token بعد كلمة "Bearer "
        jwt = authHeader.substring(7);
        // substring(7) → بتشيل أول 7 حروف ("Bearer ")
        // الباقي هو الـ Token الحقيقي

        // 4) جيب الـ Email من الـ Token
        userEmail = jwtService.extractUsername(jwt);

        // 5) لو الـ Email موجود والمستخدم مش authenticated لحد دلوقتي
        if (userEmail != null &&
            SecurityContextHolder.getContext().getAuthentication() == null) {
            // SecurityContextHolder → زي صندوق بيحتفظ بمعلومات المستخدم الحالي
            // getAuthentication() == null → يعني المستخدم مش authenticated لسه

            // 6) جيب بيانات المستخدم من الـ DB
            UserDetails userDetails =
                userDetailsService.loadUserByUsername(userEmail);

            // 7) تحقق إن الـ Token صحيح
            if (jwtService.isTokenValid(jwt, userDetails)) {

                // 8) عمل Authentication Token
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        // userDetails → بيانات المستخدم
                        null,
                        // credentials → null لأننا مش محتاجين الـ password هنا
                        userDetails.getAuthorities()
                        // authorities → الـ Roles بتاعة المستخدم
                    );

                authToken.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                    // بنضيف تفاصيل الـ Request زي الـ IP Address
                );

                // 9) حط الـ Authentication في الـ SecurityContext
                SecurityContextHolder.getContext()
                    .setAuthentication(authToken);
                // دلوقتي Spring Security عارف إن المستخدم ده authenticated
            }
        }

        // 10) كمل للـ Filter التاني في السلسلة
        filterChain.doFilter(request, response);
    }
}
```

### `SecurityConfig.java`

```java
package com.notification.jpa.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
// @Configuration → بيقول لـ Spring إن الكلاس ده فيه Beans جاهزة للاستخدام

@EnableWebSecurity
// @EnableWebSecurity → بتفعّل Spring Security في المشروع
// بدونها Spring Security مش هيشتغل

@EnableMethodSecurity
// @EnableMethodSecurity → بتخلي تقدر تستخدم @PreAuthorize على الـ methods
// مثال: @PreAuthorize("hasRole('ADMIN')")

@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final CustomUserDetailsService userDetailsService;

    @Bean
    // @Bean → بيقول لـ Spring "الـ method دي بترجع Object، احتفظ بيه واديه لأي حد محتاجه"

    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // SecurityFilterChain → بيحدد قواعد الأمان للـ HTTP requests
        // HttpSecurity → بنستخدمه عشان نكوّن الـ Security

        http
            .csrf(csrf -> csrf.disable())
            // csrf → Cross-Site Request Forgery
            // disable() → بنوقفه لأننا بنستخدم JWT (stateless)
            // CSRF مهم في الـ session-based apps بس مش محتاجينه هنا

            .authorizeHttpRequests(auth -> auth

                .requestMatchers("/api/auth/**").permitAll()
                // requestMatchers → بيحدد الـ URLs
                // "/api/auth/**" → كل URLs اللي بتبدأ بـ /api/auth
                // permitAll() → متاحة للجميع بدون login (register و login)

                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                // hasRole("ADMIN") → بس الـ ADMIN يقدر يدخل

                .anyRequest().authenticated()
                // anyRequest() → أي request تانية
                // authenticated() → لازم المستخدم يكون logged in
            )

            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                // STATELESS → مش هنحتفظ بـ Session في الـ Server
                // كل request لازم تجيب الـ JWT Token معاها
                // ده أساس الـ JWT Authentication
            )

            .authenticationProvider(authenticationProvider())
            // بنقوله "استخدم الـ Provider ده للـ Authentication"

            .addFilterBefore(jwtAuthFilter,
                UsernamePasswordAuthenticationFilter.class);
            // addFilterBefore → بنضيف الـ JWT Filter قبل الـ Default Filter
            // يعني الـ JWT Filter بيشتغل الأول ويتحقق من الـ Token

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        // AuthenticationProvider → بيتحقق من الـ credentials (email + password)

        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        // DaoAuthenticationProvider → بيجيب المستخدم من الـ DB ويتحقق من الـ Password

        authProvider.setUserDetailsService(userDetailsService);
        // قوله "اجيب المستخدم من هنا"

        authProvider.setPasswordEncoder(passwordEncoder());
        // قوله "استخدم الـ Encoder ده عشان تقارن الـ Passwords"

        return authProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config
    ) throws Exception {
        // AuthenticationManager → المسؤول عن عملية الـ Authentication كلها
        // بنستخدمه في الـ AuthService وقت الـ Login
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
        // BCryptPasswordEncoder → خوارزمية تشفير قوية جداً
        // بتشفر الـ password قبل ما تتحط في الـ DB
        // مش ممكن حد يفك التشفير — حتى لو اخترق الـ DB
        // عند الـ Login بتقارن الـ password المدخلة بالمشفرة
    }
}
```

---

## ثامناً — الـ Service

### `AuthService.java` (Interface)

```java
package com.notification.jpa.service;

import com.notification.jpa.dto.AuthResponse;
import com.notification.jpa.dto.LoginRequest;
import com.notification.jpa.dto.RegisterRequest;

public interface AuthService {
    AuthResponse register(RegisterRequest request);
    AuthResponse login(LoginRequest request);
}
```

### `AuthServiceImpl.java`

```java
package com.notification.jpa.service.impl;

import com.notification.jpa.dto.AuthResponse;
import com.notification.jpa.dto.LoginRequest;
import com.notification.jpa.dto.RegisterRequest;
import com.notification.jpa.models.Role;
import com.notification.jpa.models.User;
import com.notification.jpa.repositories.UserRepository;
import com.notification.jpa.security.JwtService;
import com.notification.jpa.service.AuthService;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class AuthServiceImpl implements AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    @Override
    public AuthResponse register(RegisterRequest request) {

        // 1) لو الـ Email موجود قبل كده → ارمي Exception
        if (userRepository.findByEmail(request.getEmail()).isPresent()) {
            throw new RuntimeException("Email already registered");
        }

        // 2) ابني الـ User object
        var user = User.builder()
            .username(request.getUsername())
            .email(request.getEmail())
            .password(passwordEncoder.encode(request.getPassword()))
            // passwordEncoder.encode() → بتشفر الـ Password قبل الحفظ
            // مثال: "123456" → "$2a$10$..."
            .role(Role.USER)
            // كل مستخدم جديد بيبقى USER تلقائياً
            .build();

        // 3) احفظ في الـ DB
        userRepository.save(user);

        // 4) اعمل JWT Token
        var jwtToken = jwtService.generateToken(user);

        // 5) ارجع الـ Token للـ Client
        return AuthResponse.builder()
            .token(jwtToken)
            .build();
    }

    @Override
    public AuthResponse login(LoginRequest request) {

        // 1) Spring Security بيتحقق من الـ Email والـ Password
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getEmail(),
                request.getPassword()
                // Spring هيستدعي loadUserByUsername تلقائياً
                // وهيقارن الـ Password بالمشفر في الـ DB
                // لو غلط → بيرمي AuthenticationException تلقائياً
            )
        );

        // 2) لو وصلنا هنا يبقى الـ credentials صح
        // جيب المستخدم من الـ DB
        var user = userRepository.findByEmail(request.getEmail())
            .orElseThrow();

        // 3) اعمل JWT Token
        var jwtToken = jwtService.generateToken(user);

        // 4) ارجع الـ Token
        return AuthResponse.builder()
            .token(jwtToken)
            .build();
    }
}
```

---

## تاسعاً — الـ Controller

### `AuthController.java`

```java
package com.notification.jpa.controller;

import com.notification.jpa.dto.AuthResponse;
import com.notification.jpa.dto.LoginRequest;
import com.notification.jpa.dto.RegisterRequest;
import com.notification.jpa.service.AuthService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
// @RestController → بيقول لـ Spring إن الكلاس ده Controller
// وكل الـ responses بترجع JSON تلقائياً

@RequestMapping("/api/auth")
// @RequestMapping → بيحدد الـ Base URL لكل الـ endpoints في الكلاس

@RequiredArgsConstructor
public class AuthController {

    private final AuthService authService;

    @PostMapping("/register")
    // @PostMapping → بيحدد إن الـ endpoint ده بيستقبل POST requests
    // URL الكامل: POST /api/auth/register

    public ResponseEntity<AuthResponse> register(
        @RequestBody @Valid RegisterRequest request
        // @RequestBody → بيحول الـ JSON اللي جاي لـ Object
        // @Valid → بيشغّل الـ Validations اللي حطيناها في الـ DTO
    ) {
        return ResponseEntity.ok(authService.register(request));
        // ResponseEntity.ok() → بيرجع Response بـ Status 200
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(
        @RequestBody @Valid LoginRequest request
    ) {
        return ResponseEntity.ok(authService.login(request));
    }
}
```

---

## عاشراً — `application.properties`

```properties
# Database
spring.datasource.url=jdbc:mysql://localhost:3306/jpa_db
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
# update → بيعمل الجداول تلقائياً لو مش موجودة، وبيعدل لو في تغييرات
# في الـ Production استخدم "validate" أو flyway/liquibase

spring.jpa.show-sql=true
# show-sql → بيعرض الـ SQL Queries في الـ Console (للـ Development بس)

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# JWT Secret (ضيفها هنا بدل ما تحطها في الكود)
jwt.secret=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
jwt.expiration=86400000
# 86400000 ms = 24 ساعة
```

---

## حادي عشر — Flow الـ Registration والـ Login

```
=== Registration ===
Client → POST /api/auth/register
  → JwtAuthFilter (مفيش Token → عدّى)
  → AuthController.register()
  → @Valid (تحقق من الـ Input)
  → AuthServiceImpl.register()
    → تحقق إن الـ Email مش موجود
    → شفّر الـ Password بـ BCrypt
    → احفظ في الـ DB
    → اعمل JWT Token
  → ارجع { "token": "eyJ..." }

=== Login ===
Client → POST /api/auth/login
  → JwtAuthFilter (مفيش Token → عدّى)
  → AuthController.login()
  → AuthServiceImpl.login()
    → authenticationManager.authenticate()
      → loadUserByUsername(email) من الـ DB
      → قارن الـ Password بـ BCrypt
      → لو غلط → 401 Unauthorized
    → اعمل JWT Token
  → ارجع { "token": "eyJ..." }

=== Request محمية ===
Client → GET /api/publishers (مع Header: Authorization: Bearer eyJ...)
  → JwtAuthFilter
    → جيب الـ Token من الـ Header
    → فك الـ Token واجيب الـ Email
    → جيب المستخدم من الـ DB
    → تحقق إن الـ Token صح ومش منتهي
    → حط المستخدم في الـ SecurityContext
  → PublisherController.getAllPublishers()
  → ارجع البيانات
```

---

## هل ده كافي كـ Junior؟

**نعم، وزيادة.** اللي عملناه هنا بيغطي:

- JWT Authentication كامل
- Registration + Login
- Password Encoding
- Role-based Authorization
- Filter Chain
- Stateless Security

اللي ممكن تضيفه بعدين لما تكبر:

- Refresh Tokens
- Email Verification
- Rate Limiting
- Exception Handling موحدة لـ Security errors

ابدأ بيه كده وهيبهر أي حد يشوف الكود.