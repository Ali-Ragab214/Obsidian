```java

package com.notification.jpa.models;  

  
import jakarta.persistence.*;  
import lombok.AllArgsConstructor;  
import lombok.Builder;  
import lombok.Data;  
import lombok.NoArgsConstructor;  
import org.springframework.data.annotation.CreatedDate;  
import org.springframework.data.annotation.LastModifiedDate;  
import org.springframework.data.jpa.domain.support.AuditingEntityListener;  
import org.springframework.security.core.GrantedAuthority;  
import org.springframework.security.core.authority.SimpleGrantedAuthority;  
import org.springframework.security.core.userdetails.UserDetails;  
import org.springframework.util.CollectionUtils;  
  
import java.time.LocalDate;  
import java.time.LocalDateTime;  
import java.util.Collection;  
import java.util.List;  
  
@Entity  
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
@Table(name = "users")  
@EntityListeners(AuditingEntityListener.class)  
  
public class User  implements UserDetails {  


    @Id  
    @GeneratedValue(strategy = GenerationType.UUID) //universally unique identifier  
    private String id;  
  
    @Column(name = "First_Name" ,nullable = false)  
    private String firstName;  
    @Column(name = "Last_Name" ,nullable = false)  
    private String lastName;  
  
    @Column(name = "Email" , nullable = false,unique = true)  
    private String email;  
  
  
    @Column(name = "Phone_Number" ,nullable = false,unique = true)  
    private String  phoneNumber;  
  
    @Column(name = "Password" , nullable = false)  
    private String password;  
  
    @Column(name = "Date Of Birth" ,nullable = false)  
    private LocalDate dateOfBirth;  
  
    @CreatedDate  
    @Column(name = "Created_At" ,updatable = false,nullable = false)  
    private LocalDateTime createdAt;  
  
  
    @LastModifiedDate  
    @Column(name = "Last_Modified_Date",insertable = false)  
    private LocalDateTime lastModifiedDate ;  
  
    @Column(name = "Is_Enabled")  
    private  boolean enabled;  
  
  
    @Column(name = "Is_Account_LOCKED")  
    private boolean locked;  
  
    @Column(name = "Is_Expired")  
    private boolean expired;  
  
    @Column(name = "Is_Email_Verified")  
    private boolean emailVerified;  
  
    @Column(name = "Is_Phone_Verified")  
    private boolean  phoneVerified;  
  
  
@ManyToMany(cascade = {CascadeType.PERSIST,CascadeType.MERGE  
},fetch = FetchType.EAGER)  
@JoinTable(  
name = "users_roles",  
joinColumns = {  
@JoinColumn(  
name = "user_id"  
)  
}  
,  
inverseJoinColumns ={  
@JoinColumn(  
        name = "role_id"  
)  
  
}  
)  
    private List<Role> roles;  
  
  
  
  
//this method is used to return the authorities of the user which are the roles that the user has  
    //يعني بترجع ال roles اللي ال user عنده  
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        if(CollectionUtils.isEmpty(this.roles)){  
            return List.of();  
        }  
   return this.roles.stream().  
           map(role-> new SimpleGrantedAuthority(role.getName())).  
           toList();  
  
    }  
  
  
    @Override  
    public String getPassword() {  
        return this.password;  
    }  
  
    @Override  
    public String getUsername() {  
        return  this.email;  
    }  
  
    @Override  
    public boolean isAccountNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isAccountNonLocked() {  
        return !this.locked ;  
    }  
  
    @Override  
    public boolean isCredentialsNonExpired() {  
        return  !this.expired;  
    }  
}
```

ممتاز 👌 الكود اللي انت كاتبه قريب جدًا من الشكل الحقيقي اللي بيتستخدم في الشركات. خلينا نفهم الصورة الكاملة بشكل احترافي من أول UserDetails لحد الفلو كله.

سأفترض إننا بنبني **Spring Boot + Spring Security + JPA** مع إما **Session أو JWT**.

---

# 1) UserDetails + UserDetailsService (قلب Spring Security)

Spring Security لا يعرف كيان اسمه _User Entity_… هو يعرف فقط Interface اسمه:

- `UserDetails` → يمثل المستخدم داخل السيكيوريتي
    
- `UserDetailsService` → مسؤول عن جلب المستخدم من الداتابيز
    

يعني باختصار:

```
Database User  →  يتحول إلى UserDetails  →  Spring Security يستخدمه
```

### لماذا UserDetails؟

Spring يحتاج يعرف:

- username
    
- password
    
- roles/authorities
    
- account status
    

لذلك أنت تجعل الـ entity نفسها implement `UserDetails` ✔️  
وهذا **أكثر pattern شائع في المشاريع الحقيقية**.

---

# 2) شكل User Entity في المشاريع الحقيقية

الكود الذي كتبته ممتاز جدًا 👏 وهذه هي الحقول الشائعة فعلاً:

## الحقول الأساسية

تقريبًا كل سيستم حقيقي يحتوي:

```
id
email (username)
password (hashed)
enabled
locked
accountExpired
credentialsExpired
roles
createdAt
```

وأنت أضفت:

- phone
    
- emailVerified
    
- phoneVerified  
    وهذا واقعي جدًا لأن معظم الأنظمة فيها verification.
    

## العلاقة مع Role

العلاقة ManyToMany شائعة جدًا:

```
User  --- many-to-many --- Role
```

وroles تتحول إلى Authorities هنا 👇

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    return roles.stream()
        .map(role -> new SimpleGrantedAuthority(role.getName()))
        .toList();
}
```

🔴 ملاحظة مهمة جدًا:  
يجب أن تكون أسماء roles بالشكل:

```
ROLE_USER
ROLE_ADMIN
```

لأن Spring يتوقع prefix = `ROLE_`

---

# 3) Authentication Flow (كيف يتم تسجيل الدخول فعليًا)

هذه أهم جزئية لفهم الصورة الكبيرة.

## خطوة 1 — Request login

اليوزر يرسل:

```
POST /login
email + password
```

## خطوة 2 — Security Filters

Spring Security يمرر الطلب عبر سلسلة Filters.

أهمهم:

- UsernamePasswordAuthenticationFilter (في session)
    
- JwtAuthenticationFilter (في JWT)
    

## خطوة 3 — AuthenticationManager

الفيلتر ينادي:

```
AuthenticationManager.authenticate()
```

## خطوة 4 — UserDetailsService

Spring ينادي:

```java
loadUserByUsername(email)
```

هنا أنت تجلب المستخدم من DB.

## خطوة 5 — Password check

Spring يقارن:

```
raw password VS encoded password
```

باستخدام PasswordEncoder.

لو صح → يتم إنشاء Authentication Object.

## خطوة 6 — SecurityContext

يتم حفظ المستخدم داخل:

```
SecurityContextHolder
```

ومن هنا أي endpoint يقدر يعرف المستخدم الحالي.

---

# 4) الكود الأساسي المطلوب في أي مشروع

## UserRepository

```java
@Repository
public interface UserRepository extends JpaRepository<User, String> {
    Optional<User> findByEmail(String email);
}
```

---

## UserDetailsService implementation

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository repo;

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        return repo.findByEmail(email)
                .orElseThrow(() ->
                        new UsernameNotFoundException("User not found"));
    }
}
```

لاحظ أنه يرجع User مباشرة لأنه implement UserDetails ✔️

---

## PasswordEncoder Bean

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

⚠️ لا تستخدم أبداً:

- NoOpPasswordEncoder ❌
    
- MD5 / SHA1 ❌
    

BCrypt هو المعيار.

---

## SecurityConfig (أبسط إعداد)

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final CustomUserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .userDetailsService(userDetailsService)
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

---

# 5) Pitfalls & Best Practices (مهم جداً في المشاريع)

## ❌ تخزين الباسورد plain text

أكبر كارثة أمنية.

التسجيل يجب أن يكون:

```java
user.setPassword(passwordEncoder.encode(request.getPassword()));
```

---

## Account flags (أنت عاملها ✔️)

Spring يدعم:

|flag|الهدف|
|---|---|
|enabled|تفعيل بعد verification|
|locked|حظر الحساب|
|accountExpired|صلاحية الحساب|
|credentialsExpired|تغيير الباسورد|

لكن يجب تعديل الدوال:

```java
@Override
public boolean isEnabled() {
    return this.enabled;
}

@Override
public boolean isAccountNonExpired() {
    return !this.expired;
}
```

أنت حالياً ترجع true دائماً → هذا خطأ شائع.

---

## Password Reset

لا تحفظ reset token داخل User.  
الأفضل جدول منفصل:

```
password_reset_tokens
```

---

## CSRF

- Session apps → فعّل CSRF
    
- JWT APIs → عطّله
    

---

## JWT vs Session

|Session|JWT|
|---|---|
|web apps|mobile + SPA|
|stateful|stateless|
|cookie|Authorization header|

في 2025+ معظم الأنظمة تستخدم **JWT**.

---

# 6) هل هذا Structure مستخدم فعلاً في الشركات؟

نعم ✔️ بنسبة كبيرة جدًا.

هذا يسمى:

```
Custom UserDetails + JPA Entity
```

وهو الـ **standard industry approach**.

لكن في المشاريع الكبيرة يتم إضافة:

- audit logs
    
- login attempts
    
- refresh tokens
    
- MFA
    
- OAuth2 / Google login
    

---

# هل تقدر تستخدم هذا الكود في مشاريعك القادمة؟

الإجابة المختصرة: **نعم 100%**  
بل أنت بالفعل على الطريق الصحيح.

التعديلات الصغيرة التي أنصح بها:

1. ربط flags بالدوال (isEnabled وغيرها)
    
2. إضافة enum للـ roles بدلاً من string (اختياري)
    
3. استخدام JWT لاحقاً
    

---

لو تحب الخطوة التالية نعمل flow كامل:

- register endpoint
    
- login endpoint
    
- JWT generation 🔥

