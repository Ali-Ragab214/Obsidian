


---

# 1️⃣ يعني ايه Filter أصلاً؟ (ببساطة جدًا)

تخيل إن أي Request بييجي للسيرفر لازم يعدي على **سلسلة حواجز (Checkpoints)** قبل ما يوصل للـ Controller.

الحواجز دي اسمها **Filters**.

كل Filter:

- يشوف الـ Request
    
- ممكن يعديه ✔️
    
- أو يوقفه ❌
    
- أو يعدل فيه ✏️
    

زي فلتر الميه 💧:  
المياه لازم تعدي عليه قبل ما توصل لك.

---

## ليه بنستخدم Filters؟

عشان نعمل حاجات **مش مرتبطة بالـ Business Logic** زي:

- Authentication (مين المستخدم؟)
    
- Authorization (مسموحله يعمل ايه؟)
    
- Logging
    
- CORS
    
- JWT validation
    

يعني الحاجات العامة اللي لازم تحصل قبل ما نوصل للـ Controller.

---

# 2️⃣ رحلة الـ Request في Spring Boot

الـ Request مش بيروح للـ Controller مباشرة ❌  
بيمشي في رحلة طويلة.

الرحلة ببساطة:

```
Client → Servlet Container → Filter Chain → DispatcherServlet → Controller
```

خلينا نبسطها:

1. الطلب ييجي من المتصفح أو الموبايل
    
2. يدخل على **Servlet Container** (Tomcat غالبًا)
    
3. يدخل على **Filter Chain** ← هنا Spring Security
    
4. لو عدى → يروح للـ Controller
    
5. يرجع Response لنفس الطريق بالعكس
    

---

# 3️⃣ فين Spring Security هنا؟

Spring Security عبارة عن **مجموعة Filters ضخمة جدًا**.

اسم السلسلة دي:

```
Spring Security Filter Chain
```

اسم الكلاس المسؤول عنها:

```
FilterChainProxy
```

يعني Spring Security مش شغال في Controllers…  
هو واقف **قبلها بالكامل**.

---

# 4️⃣ التشبيه العظيم 🔥 (بوابات المطار)

تخيل إن السيرفر = مطار ✈️

والـ Request = مسافر.

المسافر لازم يعدي على بوابات:

1️⃣ بوابة التذاكر → هل معاه Token ؟  
2️⃣ بوابة الجوازات → هل الهوية صحيحة؟  
3️⃣ بوابة الأمن → مسموح يدخل المكان ده؟  
4️⃣ لو عدى كله → يدخل الطيارة (Controller)

لو فشل في أي بوابة → يرجع البيت ❌

كل بوابة = Filter.

---

# 5️⃣ الفلو الحقيقي للـ Authentication و Authorization

خلينا نمشي خطوة خطوة مع Request فيه JWT:

```
Request → Security Filters → Controller
```

## خطوة 1: هل في Token؟

Filter: JWT Filter  
لو مفيش Token → غالبًا 401

## خطوة 2: هل Token صحيح؟

- يتحقق من signature
    
- يتحقق من expiration
    

لو غلط → 401 ❌

## خطوة 3: نحط المستخدم في SecurityContext

دلوقتي بقى عندنا:

```
Authenticated User (Principal)
```

## خطوة 4: Authorization

يوصل للـ Controller  
Spring Security يسأل:

```
هل المستخدم ده مسموح يدخل الـ endpoint؟
```

لو مش مسموح → 403 ❌

لو مسموح → يدخل ✔️

---

# 6️⃣ مثال بسيط جدًا على Request يمشي في الـ Chain

تخيل endpoint:

```
GET /api/profile
```

والـ endpoint محتاج login.

## السيناريو 1: بدون Token

1. يدخل الفلتر
    
2. يلاقي مفيش Token
    
3. يوقف السلسلة
    
4. يرجع:
    

```
401 Unauthorized
```

الـ Controller **لم يتم استدعاؤه أصلاً** 🚫

---

## السيناريو 2: Token صحيح

1. يدخل JWT filter
    
2. يتحقق من التوكن ✔️
    
3. يحط user في SecurityContext ✔️
    
4. يكمل باقي الفلاتر ✔️
    
5. يوصل للـ Controller ✔️
    

---

# 7️⃣ مثال كود بسيط لفلتر مخصص

فلتر يطبع كل Request (للفهم فقط):

```java
@Component
public class SimpleLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
                                    throws ServletException, IOException {

        System.out.println("Request URL: " + request.getRequestURI());

        // مهم جدًا: كمل السلسلة
        filterChain.doFilter(request, response);

        System.out.println("Response Status: " + response.getStatus());
    }
}
```

### أهم سطر هنا:

```java
filterChain.doFilter(request, response);
```

ده معناه:

> خلّي الطلب يكمل طريقه لباقي الفلاتر.

لو مسحت السطر ده →  
الطلب **يقف هنا للأبد** 😄

---

# 8️⃣ إضافة فلتر قبل فلتر التوثيق

في Spring Security:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

    http
        .addFilterBefore(new SimpleLoggingFilter(),
                UsernamePasswordAuthenticationFilter.class)
        .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
        );

    return http.build();
}
```

يعني:  
فلترنا هيشتغل **قبل فلتر تسجيل الدخول**.

---

# 9️⃣ أخطاء شائعة جدًا 😄

### ❌ نسيان doFilter

لو نسيت:

```java
filterChain.doFilter()
```

الـ request هيموت هنا 😅

---

### ❌ ترتيب الفلاتر غلط

لو حطيت JWT filter بعد authorization → مش هيشتغل.

الترتيب مهم جدًا.

---

### ❌ التفكير إن الفلتر زي Controller

الفلتر:

- قبل Controllers
    
- يشتغل على كل الطلبات
    

---

# 🔟 ازاي نعمل Debug للفلاتر؟

### 1) فعل Debug Logs 🔥

في application.properties:

```
logging.level.org.springframework.security=DEBUG
```

هتشوف:

- كل فلتر اشتغل
    
- ليه الطلب اترفض
    
- مين authenticated
    

ده كنز حرفيًا.

---

### 2) اطبع SecurityContext

داخل الفلتر:

```java
System.out.println(SecurityContextHolder.getContext().getAuthentication());
```

---

# 1️⃣1️⃣ Checklist سريعة للـ JWT Filter

لو هتبدأ تطبيق عملي:

✔️ اعمل JWT Filter extends OncePerRequestFilter  
✔️ استخرج Token من Header  
✔️ تحقق من التوكن  
✔️ اعمل Authentication object  
✔️ حطه في SecurityContext  
✔️ كمل chain

---

# 🎯 الخلاصة

- كل Request يعدي على سلسلة Filters
    
- Spring Security = مجموعة Filters
    
- Authentication يحصل داخل الفلاتر
    
- Authorization يحصل قبل دخول Controller
    
- لو Filter رفض الطلب → Controller لن يعمل
    

---

لو تحب نبدأ نكتب JWT Filter حقيقي خطوة خطوة ونفهم كل سطر فيه قولّي ونبدأ 🔥