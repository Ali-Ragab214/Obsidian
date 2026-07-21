سؤال ممتاز جدًا، وده من الأسئلة الكلاسيكية في انترفيوهات Java Backend. فيه ناس كتير بتتلخبط بين **Java**, **Servlet**, و **Spring**. خليني أبني الصورة من الصفر.

---

# الصورة الكبيرة

تخيل إن عندك مطعم 🍽️.

- العميل = **Browser / Client**
    
- الطلب = **HTTP Request**
    
- الشيف = **Java Code**
    
- المدير اللي بيوزع الشغل = **Servlet**
    
- شركة كاملة بتدير المطعم (حجوزات، مخزن، موظفين...) = **Spring**
    

---

# أولًا: ما هو Servlet؟

**Servlet ليس Framework.**

هو عبارة عن **Java API / Specification** لتعريف إزاي Java تتعامل مع HTTP Requests و HTTP Responses.

يعني هو Interface تقريبًا بالشكل ده:

```java
public interface Servlet {

    void init();

    void service(ServletRequest req,
                 ServletResponse res);

    void destroy();

}
```

أنت غالبًا مش بتطبق `Servlet` مباشرة.

بتورث من

```java
HttpServlet
```

مثلاً

```java
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req,
                         HttpServletResponse res) {

        res.getWriter().println("Hello");
    }

}
```

هنا

```java
doGet()
```

هتتنفذ لما يجيلك

```http
GET /users
```

---

# من الذي يستدعي Servlet؟

أنت مش بتعمل

```java
new UserServlet()
```

فيه حاجة اسمها

## Servlet Container

زى

- Tomcat
    
- Jetty
    
- Undertow
    

وده مسئول عن

```
HTTP Request

↓

Tomcat

↓

Servlet

↓

HTTP Response
```

يعني Tomcat يقول

> "جالي Request"

↓

ينادي

```java
doGet()
```

---

# هل Servlet Framework؟

❌ لا.

هو مجرد Specification + API.

يعني بيقول:

> "أي Container لازم يطبق القواعد دي."

---

# طيب Spring إيه؟

Spring Framework عبارة عن Framework ضخم جدًا.

بيعمل

- Dependency Injection
    
- IoC
    
- AOP
    
- Security
    
- Transactions
    
- Validation
    
- REST APIs
    
- وغيرها.
    

---

# هل Spring يستخدم Servlet؟

الإجابة:

**نعم.**

ودي أهم نقطة.

---

Spring MVC مبني فوق Servlet.

يعني

```
Browser

↓

Tomcat

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository
```

شايف

```
DispatcherServlet
```

؟

ده نفسه

```java
HttpServlet
```

---

بص عليه

```java
DispatcherServlet

↓

extends FrameworkServlet

↓

extends HttpServletBean

↓

extends HttpServlet
```

يعني

DispatcherServlet

هو Servlet.

---

# إذًا عندما أكتب

```java
@RestController

class UserController {

    @GetMapping("/users")
    public String hello() {
        return "Hello";
    }

}
```

هل Spring هو اللي استقبل الـ Request؟

لا.

اللي حصل

```
Browser

↓

Tomcat

↓

DispatcherServlet

↓

UserController
```

يعني Tomcat نادى Servlet.

والـ Servlet ده هو DispatcherServlet.

بعدها DispatcherServlet لقى

```
@GetMapping("/users")
```

وراح نادى الـ Controller.

---

# بدون Spring

أنت كنت هتكتب

```java
public class UserServlet extends HttpServlet {

    protected void doGet(...) {

    }

}
```

لكل Endpoint.

---

لو عندك

100 Endpoint

↓

100 Servlet.

---

أما Spring

بيخليك تعمل

```java
@GetMapping()

@PostMapping()

@DeleteMapping()

```

أسهل بكتير.

---

# مقارنة

## Servlet

```java
@WebServlet("/users")
public class UserServlet extends HttpServlet {

    protected void doGet(...) {

    }

}
```

---

## Spring

```java
@RestController

class UserController {

    @GetMapping("/users")

    public String getUsers(){

    }

}
```

أبسط بكتير.

---

# هل Spring بديل لـ Servlet؟

الإجابة الدقيقة:

**لا.**

Spring **ليس بديلًا** لـ Servlet.

هو Framework **مبني فوق Servlet** (في تطبيقات Spring MVC وSpring Boot التقليدية).

يعني Spring يستخدم Servlet بدل ما يلغيها.

---

# هل أقدر أستخدم Servlet بدون Spring؟

أكيد.

زمان قبل Spring كان الناس بتعمل مواقع كاملة بالـ Servlet وJSP.

---

# هل أقدر أستخدم Spring بدون Servlet؟

في Spring MVC التقليدي:

❌ لا.

لأنه مبني على Servlet API.

لكن فيه مشروع تاني اسمه

**Spring WebFlux**

وده مبني على Reactive APIs ومش بيعتمد على Servlet بالطريقة التقليدية.

---

# ليه ظهر Spring؟

لأن كتابة تطبيق كامل بالـ Servlet كانت متعبة جدًا.

تخيل مشروع فيه

- Authentication
    
- JWT
    
- Transactions
    
- Validation
    
- Dependency Injection
    
- Database
    
- Exception Handling
    
- Security
    

لو هتعمل ده كله بالـ Servlet هتكتب آلاف الأسطر بنفسك.

Spring وفر كل ده جاهز.

---

# لو سألك الـ Interviewer

**What is the difference between Servlet and Spring?**

إجابة احترافية:

> A Servlet is a Java API and specification used to handle HTTP requests and responses. It is not a framework. It runs inside a Servlet Container such as Tomcat. Spring, on the other hand, is a full-featured framework that provides dependency injection, transaction management, security, data access, and web development support. Spring MVC is built on top of the Servlet API, using `DispatcherServlet` as its front controller to route requests to application controllers.

---

# احفظ الرسمة دي لأنها هتربط كل المفاهيم مع بعض

```text
                Java
                  │
                  ▼
        Servlet API (Specification)
                  │
                  ▼
      Servlet Container (Tomcat)
                  │
                  ▼
        DispatcherServlet (Spring MVC)
                  │
                  ▼
     @Controller / @RestController
                  │
                  ▼
             Service Layer
                  │
                  ▼
          Repository / Database
```

**الخلاصة في سطر واحد:**

- **Servlet** = التكنولوجيا الأساسية في Java لمعالجة HTTP Requests.
    
- **Spring MVC** = Framework يستخدم Servlet (وبالتحديد `DispatcherServlet`) ويضيف فوقها طبقات وميزات تجعل تطوير تطبيقات الويب أسهل وأسرع وأكثر تنظيمًا.