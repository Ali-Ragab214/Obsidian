تمام، هعملك شرح شامل ومرتب زي ما بتحب، جاهز تحطه على Obsidian مباشرة.

# Servlets, Tomcat & DispatcherServlet — الدليل الكامل

---

# 1) إيه هو الـ Servlet أصلاً؟

**Servlet** = Java class معياري (من Jakarta EE spec) مصمم خصيصاً للتعامل مع HTTP requests وresponses.

```text
Servlet = Class
implements
javax.servlet.Servlet (أو jakarta.servlet.Servlet)
```

بيوفر methods زي:

```java
init()
service()
doGet()
doPost()
doPut()
doDelete()
destroy()
```

**الفكرة الأساسية:** الـ Servlet هي الطريقة اللي بيها كود الـ Java بيعرف يتكلم مع بروتوكول HTTP.
جافا دي مجرد لغه مش هتعرف تتعامل مع الريكوستات 

---

# 2) إيه هو Servlet Container؟

الـ Servlet نفسها **مش قادرة تشتغل لوحدها**. هي محتاجة برنامج يديرها، اسمه:

```text
Servlet Container
```

وظيفته:

- يفتح Socket ويستني عليه HTTP requests
- يحول الـ raw bytes الجايين من الشبكة لـ `HttpServletRequest` object
- ينادي على الـ Servlet الصح حسب الـ URL
- ياخد الـ `HttpServletResponse` ويحولها لـ HTTP response فعلي يرجع للـ Client
- بيدير الـ **lifecycle** بتاعت الـ Servlet (متى تتعمل، متى تتدمر)

---

# 3) طيب إيه هو Tomcat بالظبط؟

```text
Apache Tomcat = Servlet Container | Web Server
```

هو **implementation** حقيقي لمواصفة الـ Servlet. يعني:

```text
Servlet Specification (المعيار/الورقة)
        |
        v
Tomcat (التنفيذ الفعلي/البرنامج)
```

## هل Tomcat هو Web Server ولا Application Server؟

Tomcat فعلياً:

```text
Web Server (HTTP handling)
   +
Servlet Container (Servlet lifecycle management)
```

لكنه **مش** Full Application Server زي WildFly أو GlassFish، لأنه مش بيدعم كل مواصفات Jakarta EE (زي EJB مثلاً) — هو بس بيدعم Servlets وJSP.

## Tomcat في Spring Boot تحديداً

عادي بتفتكر إنك محتاج تنزل Tomcat وتحطه على السيرفر وتـ deploy عليه ملف `.war`. لكن Spring Boot بيعمل حاجة مختلفة:

```text
Spring Boot Application
       |
       v
Embedded Tomcat (جوا الـ JAR نفسه)
```

يعني Tomcat **بقى مكتبة (dependency) جوا مشروعك**، مش سيرفر منفصل. لما بتعمل:

```bash
./mvnw spring-boot:run
```

Spring بيشغل نسخة Tomcat جوانية تلقائياً على البورت اللي إنت محدده (افتراضياً 8080).

```xml
<!-- ده موجود ضمنياً جوا spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
</dependency>
```

---

# 4) دورة حياة الـ Servlet (Servlet Lifecycle)

```text
1) Loading
   Container يحمل الـ Servlet class

2) Instantiation
   Container يعمل object واحد منها (Singleton غالباً)

3) init()
   بيتنفذ مرة واحدة بس، أول ما الـ Servlet تتحمل

4) service()
   بيتنفذ مع كل Request جديد
   وهو اللي بيوجه لـ doGet/doPost/doPut/doDelete
   حسب نوع الـ HTTP Method

5) destroy()
   بيتنفذ مرة واحدة، لما السيرفر يقفل أو الـ Servlet تتشال
```

**نقطة مهمة:** الـ Servlet Instance واحدة بتخدم **كل** الـ requests (مش بتتعمل object جديد لكل request) — ده سبب مهم ليه لازم تكون الـ Servlets **Stateless** (متخزنش state في instance variables، لأنها بتتشارك بين كل الـ threads).

---

# 5) رحلة الـ Request الكاملة في Spring Boot (خطوة خطوة)

```text
[1] Client (Browser/Postman)
        |
        | HTTP Request
        v
[2] Network / Socket
        |
        v
[3] Tomcat (Embedded Servlet Container)
        |
        | يحول الـ raw request لـ
        | HttpServletRequest object
        v
[4] DispatcherServlet
   (دي Servlet واحدة بس، مسجلة على "/"
    وبتستقبل كل الـ requests)
        |
        v
[5] Spring بيبدأ يحدد "مين المفروض يرد على الـ request ده؟"
        |
        v
[6] HandlerMapping
   (بيدور جوا كل الـ @Controllers
    عن أنهي method مطابقة لـ
    URL + HTTP Method)
        |
        v
[7] HandlerAdapter
   (بيجهز استدعاء الـ method الصح
    ويعمل bind للـ @RequestParam,
    @PathVariable, @RequestBody...)
        |
        v
[8] Interceptors (لو موجودين)
   (زي Middleware، بيشتغلوا
    قبل وبعد الـ Controller)
        |
        v
[9] Controller Method
   (مثال: @GetMapping("/users"))
        |
        v
[10] Service Layer
   (Business logic)
        |
        v
[11] Repository / Database
        |
        v
[12] الرد بيرجع تصاعدياً:
   Repository -> Service -> Controller
        |
        v
[13] Controller بيرجع Object
   (مثلاً List<User>)
        |
        v
[14] HttpMessageConverter
   (بيحول الـ Object لـ JSON
    عن طريق Jackson تلقائياً،
    وده سبب شغل @RestController)
        |
        v
[15] DispatcherServlet
   يبني الـ HttpServletResponse
        |
        v
[16] Tomcat
   يحول الـ Response لـ raw HTTP bytes
        |
        v
[17] Client
   بيستقبل الـ JSON Response
```

---

# 6) إيه هي DispatcherServlet بالظبط؟

```text
DispatcherServlet = Servlet واحدة كبيرة
كاتبها فريق Spring نفسه
```

بدل ما إنت تكتب Servlet منفصلة لكل endpoint (زي الطريقة القديمة قبل Spring)، Spring بيعمل **Servlet واحدة بس**، مسجلة على كل الـ URLs:

```java
// ده بيحصل تلقائياً جوا Spring Boot
@WebServlet("/")
public class DispatcherServlet extends HttpServlet {
    // Front Controller Pattern
}
```

## ليه اسمها "Dispatcher"؟

لأن وظيفتها إنها **توزّع (dispatch)** كل request جاي لها للمكان الصح، مش إنها ترد بنفسها. الاسم التقني للـ pattern ده:

```text
Front Controller Pattern
```

يعني: نقطة دخول واحدة (single entry point) لكل الـ requests، وهي المسؤولة عن التوجيه.

## مكونات DispatcherServlet الداخلية

```text
DispatcherServlet
    │
    ├── HandlerMapping
    │      "أنهي Controller/Method يرد على الـ URL ده؟"
    │
    ├── HandlerAdapter
    │      "إزاي أنادي على الـ method ده بالـ parameters الصح؟"
    │
    ├── HandlerExceptionResolver
    │      "لو حصل Exception، مين هيتعامل معاه؟"
    │      (ده اللي بيخلي @ExceptionHandler يشتغل)
    │
    ├── ViewResolver
    │      (مهم في MVC التقليدي مع Views زي Thymeleaf،
    │       مش مستخدم في REST APIs لأن @RestController
    │       بيرجع JSON مباشرة مش View)
    │
    └── HttpMessageConverter
           "إزاي أحول الـ Java Object لـ JSON/XML؟"
           (Jackson هو الافتراضي)
```

---

# 7) الفرق بين @Controller و @RestController هنا

```java
@Controller
public class PageController {
    @GetMapping("/home")
    public String home() {
        return "home"; // اسم View (زي home.html)
                        // هيمر على ViewResolver
    }
}
```

```java
@RestController // = @Controller + @ResponseBody
public class ApiController {
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
        // هيمر مباشرة على HttpMessageConverter
        // وير جع JSON، من غير ViewResolver خالص
    }
}
```

---

# 8) Filters vs Interceptors (نقطة كتير بتتلخبط فيها)

## Servlet Filter

```text
جزء من الـ Servlet Specification نفسها (مش Spring)
بيشتغل قبل ما الـ Request يوصل حتى لـ DispatcherServlet
```

```text
Request
   |
   v
Filter 1 (مثلاً: CORS)
   |
   v
Filter 2 (مثلاً: Logging)
   |
   v
DispatcherServlet
```

مثال:

```java
@Component
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, 
                          FilterChain chain) throws IOException, ServletException {
        System.out.println("Request received: " + req);
        chain.doFilter(req, res); // يكمل السلسلة
    }
}
```

## Spring Interceptor

```text
جزء من Spring نفسه (مش من الـ Servlet spec)
بيشتغل بعد ما DispatcherServlet يستقبل الـ request،
لكن قبل ما يوصل للـ Controller
```

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, 
                              HttpServletResponse response, Object handler) {
        // قبل الـ Controller
        return true;
    }

    @Override
    public void postHandle(...) {
        // بعد الـ Controller، قبل الـ Response يترجع
    }
}
```

## الترتيب الكامل مع بعض

```text
Client Request
     |
     v
[Servlet Filters]  <- من الـ Servlet Spec، أعم وأشمل
     |
     v
DispatcherServlet
     |
     v
[Spring Interceptors] <- من Spring، أقرب للـ business logic
     |
     v
Controller
```

**قاعدة بسيطة:** استخدم **Filter** لحاجات عامة جداً (CORS, logging خام, compression). استخدم **Interceptor** لحاجات مرتبطة بـ Spring نفسه (auth logic مرتبطة بـ Handler، أو محتاج تعرف مين الـ Controller/Method المستهدف).

---

# 9) مقارنة سريعة: Servlet الخام مقابل Spring MVC

|العملية|Servlet خام|Spring MVC|
|---|---|---|
|تسجيل الـ URL|`@WebServlet("/users")` لكل endpoint|`@GetMapping`/`@PostMapping` جوا Controller واحد|
|قراءة JSON من الـ Body|يدوي (`request.getReader()` + parsing)|تلقائي (`@RequestBody` + Jackson)|
|كتابة JSON للـ Response|يدوي (`response.getWriter().write(...)`)|تلقائي (`return` + `HttpMessageConverter`)|
|التعامل مع HTTP Methods|method منفصلة لكل verb|annotation واحدة بتحدد الـ method|
|Dependency Injection|مش موجودة أصلاً|`@Autowired` / Constructor Injection|
|Validation|يدوي بالكامل|`@Valid` + `ValidationPipe`-equivalent|

---

# 10) خلاصة شجرة العلاقات (Big Picture)

```text
Jakarta EE Specification
        |
        | (يحدد الشكل النظري)
        v
Servlet Interface
        |
        | (implementation حقيقي)
        v
Apache Tomcat (Servlet Container)
        |
        | (بيشغّل Servlet واحدة بس)
        v
DispatcherServlet (كتبها Spring)
        |
        | (بتوزع لـ)
        v
HandlerMapping -> HandlerAdapter -> Interceptors -> Controller
        |
        v
Service Layer -> Repository -> Database
```

---

# 11) جملة تلخيصية لو اتسألت في مقابلة

> "الـ Servlet هي المعيار الأساسي في Java للتعامل مع HTTP. Tomcat هو الـ Servlet Container اللي بيدير الـ Servlets دي، ومدمج جوا Spring Boot نفسه (Embedded Tomcat). Spring بيستخدم Servlet واحدة بس اسمها DispatcherServlet، شغالة بـ Front Controller Pattern — كل الـ requests بتعدي عليها الأول، وهي اللي بتوزعهم على الـ Controllers الصح عن طريق HandlerMapping وHandlerAdapter، وده اللي بيخلينا نكتب Controllers بسيطة من غير ما نتعامل مع تفاصيل الـ Servlet API يدوياً."