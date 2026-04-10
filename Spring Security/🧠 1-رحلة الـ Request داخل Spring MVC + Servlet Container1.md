
![[Pasted image 20260406005919.png]]


# 🧠 رحلة الـ Request داخل **Spring MVC + Servlet Container**

الصورة دي بتشرح رحلة أي Request من المستخدم لحد ما يرجع Response باستخدام **Spring MVC** جوه **Servlet Container** زي **Apache Tomcat** (اللي بيشرحها كورس Telusko).

خلينا نمشي في الفلو خطوة خطوة بالتفصيل 👇

---

# 1️⃣ المستخدم يرسل Request

على الشمال في الصورة فيه المستخدم (Browser / Mobile).

لما تكتب URL زي:

```
http://localhost:8080/home
```

المتصفح يعمل:

```
HTTP Request → السيرفر
```

الـ Request يوصل للسيرفر (Tomcat).

---

# 2️⃣ دخول الـ Request إلى Servlet Container (Tomcat)

Tomcat هو **البوابة** اللي تستقبل كل الـ HTTP requests.

أول ما الطلب يوصل:

- Tomcat يمسك الطلب
    
- يقرر مين في التطبيق هيعالجه
    

لكن Tomcat **مش بيكلم Controllers مباشرة** ❌  
لازم يعدي على حاجة اسمها **Filters**.

---

# 3️⃣ المرور على Filters (f1, f2, f3, f4)

العمود اللي في الصورة فيه:

```
f1
f2
f3
f4
```

دول اسمهم **Servlet Filters**.

## دور الـ Filters

دي طبقة قبل التطبيق بتعمل مهام عامة زي:

- Logging (تسجيل الطلبات)
    
- Security (التحقق من التوكن)
    
- CORS
    
- Compression
    
- Rate limiting
    

مثال عملي:

```
Request فيه JWT ؟
✔️ نعم → كمل
❌ لا → رجّع 401 Unauthorized
```

يعني:

```
Request → Filter1 → Filter2 → Filter3 → Filter4
```

لو أي Filter رفض الطلب → الرحلة تنتهي هنا.

لو كله تمام → نروح للمرحلة المهمة 👇

---

# 4️⃣ الوصول إلى قلب Spring MVC ❤️

## 👉 Front Controller (DispatcherServlet)

في الصورة مكتوب:

```
front controller
```

ده أهم جزء في Spring MVC واسمه الحقيقي:

```
DispatcherServlet
```

💡 ده **العقل المدبر للتطبيق**.

وظيفته:

- يستقبل كل الطلبات
    
- يعرف أي Controller مسؤول عن الطلب
    
- يبعته للمكان الصح
    

يعني هو "الـ Traffic Police" 🚦

---

# 5️⃣ تحديد الـ Controller المناسب (Routing)

DispatcherServlet يسأل:

```
الـ URL ده يروح لمين؟
```

Spring يشوف الـ mappings:

مثال:

```java
@GetMapping("/home")
public String home()
```

فيقرر:

```
/home → HomeController
/add → ProductController
/balance → AccountController
```

وده ظاهر في الصورة:

```
home
add
balance
```

---

# 6️⃣ تنفيذ الـ Controller Logic

الـ Controller يبدأ الشغل الحقيقي:

مثال:

```java
@GetMapping("/balance")
public double getBalance() {
    return accountService.getBalance();
}
```

غالبًا يحصل:

```
Controller → Service → Repository → Database
```

يتم تجهيز **Response**.

---

# 7️⃣ رجوع الـ Response إلى DispatcherServlet

بعد ما الـ Controller يخلص:

- يرجّع النتيجة لـ DispatcherServlet
    

النتيجة ممكن تكون:

- JSON
    
- HTML page
    
- String
    
- Object
    

Spring يحولها تلقائيًا لـ HTTP Response.

---

# 8️⃣ الرجوع عبر نفس Filters مرة أخرى

الـ Response يرجع بالعكس:

```
Controller → DispatcherServlet → Filters → Tomcat → Browser
```

الـ Filters ممكن تعمل:

- إضافة Headers
    
- Logging للـ Response
    
- Compression
    

---

# 9️⃣ المتصفح يستلم النتيجة 🎉

أخيرًا:

```
HTTP Response → Browser
```

المستخدم يشوف:

- صفحة
    
- JSON
    
- بيانات
    

انتهت الرحلة 👏

---

# 🔁 الفلو الكامل بشكل مختصر

```
User
  ↓
Tomcat (Servlet Container)
  ↓
Filters (Security / Logging / CORS)
  ↓
DispatcherServlet (Front Controller)
  ↓
Controller
  ↓
Service → Repository → DB
  ↓
Controller Response
  ↓
DispatcherServlet
  ↓
Filters
  ↓
Tomcat
  ↓
Browser
```

---

# 🎯 أهم فكرة لازم تحفظها

## Spring MVC كله مبني على فكرة:

```
Front Controller Pattern
```

يعني:

> كل الطلبات تدخل من باب واحد (DispatcherServlet).

---

لو حابب بعد كده نشرح دور DispatcherServlet بالتفصيل الداخلي خطوة بخطوة جوه Spring قولّي 🔥