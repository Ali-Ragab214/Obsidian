# Spring Security — الشرح الكامل من الأول للآخر

---

## أولاً — ايه اللي بيحصل هنا؟ الصورة الكبيرة

قبل ما ندخل في أي كود، لازم تفهم الفكرة الأساسية:

**المشكلة:** عندك API، ومش عايز أي حد يدخل عليه من غير إذن. **الحل:** Spring Security بيقف قدام كل Request وبيسأل: _"مين انت؟ وعندك إذن؟"_

الفلو العام:

```
Request جاي → Spring Security يمسكه → يتحقق من الـ User → يسمح أو يرفض
```

---

## ثانياً — شرح كل كلاس بالتفصيل

---

### 🔷 1. `SecurityConfig.java` — قلب الموضوع كله

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
```

**`@Configuration`** → بتقول لـ Spring إن الكلاس ده فيه إعدادات Beans. **`@EnableWebSecurity`** → بتفعّل Spring Security في التطبيق كله.

---

#### الـ Bean الأول: `SecurityFilterChain`

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .csrf(customizer -> customizer.disable())
        .authorizeHttpRequests(request -> request.anyRequest().authenticated())
        .httpBasic(Customizer.withDefaults())
        .sessionManagement(session ->
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .build();
}
```

ده بيحدد **قواعد الأمان** — خط بخط:

|السطر|بيعمل إيه|
|---|---|
|`.csrf().disable()`|بيوقف حماية CSRF لأننا بنشتغل مع Postman/API مش Browser|
|`.anyRequest().authenticated()`|كل endpoint محتاج user يكون logged in|
|`.httpBasic()`|طريقة الـ authentication هي Basic Auth (username:password في الـ Header)|
|`.sessionCreationPolicy(STATELESS)`|السيرفر مش هيحفظ أي Session — كل Request لازم يجيب بياناته معاه|

> **STATELESS ده المفتاح** — ومنه هتفهم ليه JWT موجود، لكن ده بعدين.

---

#### الـ Bean التاني: `AuthenticationProvider`

```java
@Bean
public AuthenticationProvider authenticationProvider() {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider(userDetailsService());
    provider.setPasswordEncoder(NoOpPasswordEncoder.getInstance());
    return provider;
}
```

**`DaoAuthenticationProvider`** → النوع ده من الـ Provider بيجيب الـ User من Database (عن طريق `UserDetailsService`) ويقارن الباسورد.

**`NoOpPasswordEncoder`** → مش بيعمل أي encryption للباسورد — Plain Text. ده مش للـ Production أبداً، بس للتعلم.

> **في الـ Production:** هتستخدم `BCryptPasswordEncoder` بدله.

---

#### الـ Bean التالت: `UserDetailsService`

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user1 = User.withUsername("ali").password("{noop}1234").roles("USER").build();
    UserDetails user2 = User.withUsername("Sara").password("{noop}1234").roles("ADMIN").build();
    UserDetails user3 = User.withUsername("malek").password("{noop}1234").roles("ADMIN").build();
    return new InMemoryUserDetailsManager(user1, user2, user3);
}
```

ده بيخزن الـ Users **في الـ RAM** مش في Database. كل ما تقفل التطبيق، الـ Users بتتمسح.

`{noop}` → بتقول لـ Spring إن الباسورد Plain Text ومش محتاج Decode.

> **ملاحظة مهمة:** الكلاس ده بيتعارض مع `MyUserDetailsService` — بعدين هنشرح ده.

---

### 🔷 2. `User.java` — الـ Entity

```java
@Entity
@Builder
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    @Id
    private int id;
    private String username;
    private String password;
}
```

ده **موديل الـ Database** — جدول اسمه `user` فيه 3 columns.

|Annotation|معناها|
|---|---|
|`@Entity`|ده جدول في الـ Database|
|`@Id`|الـ Primary Key|
|`@Data`|Lombok بتولد getters/setters/toString تلقائي|
|`@Builder`|بتخلي تعمل Object بأسلوب `User.builder().username("ali").build()`|

---

### 🔷 3. `UserPrinciple.java` — الجسر بين User وSecurity

```java
public class UserPrinciple implements UserDetails {
    private User user;
```

**المشكلة اللي بيحلها:** Spring Security مش بيعرف يشتغل مع `User` بتاعك مباشرة. هو عايز Object من نوع `UserDetails`.

**الحل:** بتعمل كلاس `UserPrinciple` بيـ Wrap الـ `User` بتاعك وبيـ Implement `UserDetails`.

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    return Collections.singleton(new SimpleGrantedAuthority("USER"));
}
```

بترجع الـ Roles — هنا كل الـ Users ليهم دور "USER" بس (Hard-coded، مش مثالي).

```java
@Override
public String getPassword() { return user.getPassword(); }

@Override
public String getUsername() { return user.getUsername(); }
```

بيجيب الـ Password والـ Username من الـ User object اللي اتـ Wrap.

```java
@Override
public boolean isAccountNonExpired() { return true; }
// وكمان isAccountNonLocked و isCredentialsNonExpired و isEnabled
```

كلهم return true = الـ Account سليم ومفيش مشاكل.

---

### 🔷 4. `UserRepo.java` — البوابة للـ Database

```java
public interface UserRepo extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

`JpaRepository` → بتدي Automatic operations زي `save()`, `findAll()`, `deleteById()`.

`findByUsername(String username)` → Spring Data بتفهم من الاسم وبتعمل:

```sql
SELECT * FROM user WHERE username = ?
```

من غير ما تكتب SQL بنفسك.

> **ملاحظة:** الـ `@Id` في الـ User عنده type `int` لكن الـ Repo بيقول `Long` — ده Bug صغير في الكود.

---

### 🔷 5. `MyUserDetailsService.java` — بيجيب الـ User من الـ DB

```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UserRepo repo;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = repo.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }
        return new UserPrinciple(user);
    }
}
```

ده **قلب الـ Authentication من الـ Database**.

الفلو بتاعه:

```
1. Spring Security بتسأل: "فين User اسمه ali؟"
2. loadUserByUsername("ali") بتتشغل
3. repo.findByUsername("ali") بتجيبه من الـ DB
4. بتـ Wrap فيه UserPrinciple وترجعه لـ Spring
5. Spring تقارن الباسورد وتقرر يدخل أو لا
```

---

## ثالثاً — الفلو الكامل خطوة بخطوة

```
┌─────────────────────────────────────────────────────────────┐
│  1. Request جاي: GET /api/hello                             │
│     مع Header: Authorization: Basic YWxpOjEyMzQ=            │
│     (ده Base64 لـ "ali:1234")                               │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  2. SecurityFilterChain بتمسك الـ Request                   │
│     - بتشوف إن anyRequest().authenticated() → لازم تـ Login │
│     - بتشوف httpBasic() → تتوقع Basic Auth في الـ Header    │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  3. DaoAuthenticationProvider بيشتغل                        │
│     - بيستدعي MyUserDetailsService.loadUserByUsername("ali") │
│     - MyUserDetailsService بيروح للـ DB عن طريق UserRepo    │
│     - UserRepo بيرجع User object                            │
│     - بيتحول لـ UserPrinciple                               │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  4. مقارنة الباسورد                                         │
│     - NoOpPasswordEncoder بيقارن "1234" == "1234" ✓         │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│  5. السماح بالدخول → الـ Request يوصل لـ Controller        │
└─────────────────────────────────────────────────────────────┘
```

---

## رابعاً — التناقض في الكود ده

في مشكلة موجودة في الكود بتاعك:

في `SecurityConfig` عندك Bean بـ `UserDetailsService` بيرجع `InMemoryUserDetailsManager` (Users في الـ RAM).

وفي نفس الوقت عندك `MyUserDetailsService` بـ `@Service` بيجيب من الـ Database.

**Spring هتختار واحد بس.** والـ Bean الـ Explicit في `SecurityConfig` هو اللي بياخد أولوية — يعني `MyUserDetailsService` مش بيشتغل!

**الحل الصح:** تمسح الـ Bean من `SecurityConfig` وتسيب `MyUserDetailsService` يشتغل لوحده.

---

## خامساً — هل ده بديل JWT؟ وإيهم أحسن؟

### الفرق الجوهري:

||Basic Auth (ده اللي شغال دلوقتي)|JWT|
|---|---|---|
|**بيحصل إيه في كل Request**|السيرفر بيروح الـ DB ويتحقق|السيرفر بيتحقق من الـ Token من غير DB|
|**الـ User بيبعت إيه**|username:password في كل Request|Token واحد (String طويل)|
|**الأمان**|الباسورد بيتبعت دايماً → خطر|الباسورد بيتبعت مرة واحدة بس|
|**الـ Performance**|DB Query في كل Request|لا DB في التحقق|
|**الـ Expiry**|مفيش|عنده وقت انتهاء|
|**الـ Scalability**|صعب مع أكتر من سيرفر|سهل جداً|

---

### فهم JWT بالتوازي مع اللي اتعلمته:

**Basic Auth (دلوقتي):**

```
كل Request → "انا ali وباسووردي 1234"
             → DB Query
             → تحقق
             → رد
```

**JWT:**

```
أول مرة بس → "انا ali وباسووردي 1234"
              → السيرفر يتحقق من الـ DB
              → السيرفر يعمل Token موقّع: "eyJhbGci..."
              → يبعته للـ Client

كل Request جديد → "معايا التوكن ده"
                  → السيرفر يتحقق من التوقيع بس (مفيش DB)
                  → رد
```

**الـ UserDetailsService** — نفسه في الاتنين، بس في JWT بيتشغل مرة واحدة بس (وقت الـ Login).

---

### امتى تستخدم إيه؟

**استخدم Basic Auth لما:**

- بتعمل Project صغير للتعلم
- Internal APIs بين سيرفرات موثوقة
- مش محتاج Scalability كبيرة

**استخدم JWT لما:**

- بتعمل Production App
- عندك Mobile App أو Frontend منفصل
- عندك أكتر من سيرفر (Microservices)
- محتاج الـ Token يـ Expire بعد وقت معين
- عايز تحمي الباسورد أكتر

---

### الخلاصة الأهم:

اللي اتعلمته ده **أساس JWT مش بديله**. في JWT هتلاقي:

- نفس `UserDetailsService` ✓
- نفس `UserPrinciple` ✓
- نفس `SecurityFilterChain` بس مع إضافات ✓
- بس هيضيف: **Filter جديد** بيقرأ الـ Token، وـ **Util class** يعمل ويتحقق من الـ Token

يعني اللي اتعلمته ده **مش هيتعوض — هو الجزء الأول من JWT**.