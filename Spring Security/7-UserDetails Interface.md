


---

# 1️⃣ يعني ايه UserDetails ؟

`UserDetails` هو **Interface** في Spring Security.  
وظيفته الأساسية:

> يمثل الـ User داخل Spring Security

Spring Security يعتمد عليه عشان يعرف:

- اسم المستخدم (username)
    
- كلمة المرور (password)
    
- الصلاحيات (roles/authorities)
    
- حالة الحساب (active, locked, expired …)
    

**بمعنى آخر:**   
Spring Security مش هيعرف أي شيء عن الـ User إلا لو أنت عرّفته من خلال UserDetails.

---

# 2️⃣ أهم الميثودات في UserDetails

```java
public interface UserDetails {
    String getUsername();
    String getPassword();
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

**شرح سريع لكل واحدة:**

|Method|معنى|
|---|---|
|getUsername()|اسم المستخدم لتسجيل الدخول|
|getPassword()|كلمة المرور|
|getAuthorities()|الصلاحيات (roles/privileges)|
|isAccountNonExpired()|الحساب لم ينتهي صلاحيته|
|isAccountNonLocked()|الحساب غير مقفل|
|isCredentialsNonExpired()|كلمة المرور لم تنته صلاحيته|
|isEnabled()|الحساب مفعل أم لا|

> Spring Security يستخدم هذه القيم أثناء **Authentication و Authorization**.

---

# 3️⃣ ليه نحتاج نعمل Custom UserDetails ؟

لأن غالبًا عندك **User entity خاص بك** في قاعدة البيانات، وممكن يكون فيه حقول إضافية زي:

- email
    
- phone
    
- address
    
- profile info
    

لو استخدمنا `User` الافتراضي في Spring، مش هيعرف الـ fields دي.  
علشان كده نعمل **Custom UserDetails**.

---

# 4️⃣ مثال عملي: Custom UserDetails

نفترض عندك entity اسمها `User`:

```java
public class User {
    private Long id;
    private String username;
    private String password;
    private String role;
    private boolean active;
}
```

نعمل Custom UserDetails class:

```java
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;
import java.util.List;

public class MyUserDetails implements UserDetails {

    private User user;

    public MyUserDetails(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(user.getRole()));
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isActive();
    }
}
```

✅ كده Spring Security يقدر يستخدم الـ User اللي عندك مباشرة.

---

# 5️⃣ استخدامه مع UserDetailsService

Spring Security يحتاج **Service** تجيب الـ UserDetails من قاعدة البيانات.

```java
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class MyUserDetailsService implements UserDetailsService {

    // ممكن تستخدم JPA Repository هنا
    private final UserRepository userRepository;

    public MyUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                        .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        return new MyUserDetails(user);
    }
}
```

> Spring Security هيستدعي `loadUserByUsername` أثناء عملية الـ Authentication.

---

# 6️⃣ فين UserDetails و MyUserDetailsService بيتستخدموا؟

- **UserDetails** → كل ما Spring Security يحتاج معلومات عن الـ User
    
- **UserDetailsService** → يستخدمه Spring Security عشان **يحضر UserDetails** من الـ DB
    

الفلو:

```
Login Request → AuthenticationManager
    → UserDetailsService.loadUserByUsername()
        → MyUserDetails (UserDetails)
            → Password + Authorities
                → Spring Security يقرر لو يسمح ولا لأ
```

---

# 7️⃣ خلاصة عملية

1️⃣ عندك entity User  
2️⃣ تعمل Class يطبق UserDetails  
3️⃣ تعمل Service يجيب UserDetails من DB  
4️⃣ Spring Security يستخدمهم للـ Authentication و Authorization

> بعد كده تقدر بسهولة تضيف Roles، Permissions، Account Status.

---

لو تحب، أقدر أرسم لك **فلو كامل للـ Login في Spring Security باستخدام UserDetails** خطوة خطوة مع كل classes اللي بيتعاملوا مع بعض.

تحب أعمله لك؟