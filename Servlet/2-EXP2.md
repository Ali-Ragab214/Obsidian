ممتاز. بما إنك بتذاكر للـ **Java Backend Interviews**، فأنا هشرحلك الموضوع **من الصفر** بطريقة منظمة، مش مجرد تعريفات. اعتبر دي Chapter كاملة تقدر تحطها في Obsidian.

---

# Servlets, Tomcat & DispatcherServlet - Complete Guide

---

# قبل ما نفهم Servlet لازم نفهم المشكلة

تخيل إنك كتبت برنامج Java عادي.

```java
public class Main {

    public static void main(String[] args) {

        System.out.println("Hello");

    }

}
```

البرنامج ده بيشتغل لما أنت تشغله بنفسك.

لكن السؤال هو...

**لو واحد فتح موقعك من الموبايل، إزاي برنامج Java هيعرف إن فيه حد بعت Request؟**

Java أصلاً مش بتعرف تتعامل مع HTTP.

هي مجرد لغة.

مش بتعرف يعني إيه:

```
GET
POST
PUT
DELETE
```

ولا تعرف يعني URL.

ولا Headers.

ولا Cookies.

ولا Sessions.

يبقى محتاجين حد يقعد يسمع على Port معين ويستقبل الـ HTTP Requests.

وهنا ظهر دور الـ Web Server.

---

# ما هو Web Server؟

الـ Web Server هو برنامج بيستقبل طلبات HTTP من الإنترنت.

أشهرهم:

- Apache HTTP Server
    
- Nginx
    
- Caddy
    

لكن...

في Java فيه نوع خاص اسمه **Servlet Container**.

---

# ما هو Servlet Container؟

هو Web Server متخصص في تشغيل Java Servlets.

أشهر واحد في العالم هو:

**Apache Tomcat**.

Tomcat وظيفته:

- يفتح Port (زي 8080)
    
- يسمع لأي HTTP Request
    
- يفهم HTTP Protocol
    
- يعمل Parsing للـ Request
    
- ينشئ Objects تمثل الـ Request والـ Response
    
- يستدعي الـ Servlet المناسبة
    
- يرجع Response للعميل
    

يعني Tomcat هو الوسيط بين الإنترنت وبين كود Java.

يمكن تخيله كالتالي:

```
Internet
      │
      ▼
  Apache Tomcat
      │
      ▼
 Java Code
```

بدون Tomcat، كود Java بتاعك مش هيستقبل أي Request.

---

# ما هي الـ Servlet؟

الـ Servlet عبارة عن Java Class مسؤولة عن معالجة HTTP Requests.

يعني أي Request يدخل، لازم Servlet تستقبله.

مثال بسيط جدًا:

```java
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
            throws IOException {

        response.getWriter().println("Hello");

    }

}
```

لو المتصفح طلب:

```
GET /hello
```

هيتنفذ:

```
doGet()
```

ولو بعت:

```
POST /hello
```

هيتنفذ:

```
doPost()
```

---

# يعني إيه HttpServlet؟

الـ HttpServlet هو Class جاهز موجود في Servlet API.

هو اللي بيوفر Functions زي:

```
doGet()

doPost()

doPut()

doDelete()

doPatch()
```

أنت بتعمل Override بس.

---

# مين بينشئ Servlet؟

أنت؟

لا.

Tomcat هو اللي بينشئها.

أول ما السيرفر يشتغل:

```
Tomcat

↓

يقرأ كل Servlets

↓

ينشئ Objects

↓

يحطها في الذاكرة
```

إذن:

```
Tomcat

↓

new HelloServlet()
```

مش أنت.

---

# Lifecycle الخاصة بالServlet

كل Servlet بتمر بأربع مراحل.

## 1- Loading

Tomcat يقرأ الـ Servlet Class.

---

## 2- Instantiation

```
new HelloServlet()
```

مرة واحدة فقط.

---

## 3- init()

تتنادى مرة واحدة.

```java
public void init() {

}
```

تستخدمها لو عايز تعمل Initialization.

---

## 4- service()

كل Request يدخل.

يعني:

```
Request 1

↓

service()
```

ثم

```
Request 2

↓

service()
```

وهكذا.

---

## 5- destroy()

عند غلق السيرفر.

```
destroy()
```

---

الـ Lifecycle بالكامل:

```
Loading

↓

Instantiation

↓

init()

↓

service()

↓

service()

↓

service()

↓

destroy()
```

---

# كيف تعمل service()؟

لو الـ Request:

```
GET /users
```

الـ service تعمل:

```
↓

doGet()
```

ولو:

```
POST
```

تعمل:

```
↓

doPost()
```

يعني service هي Dispatcher صغير داخل HttpServlet.

---

# هل كل Request يعمل Servlet جديدة؟

لا.

وده من أهم الأسئلة في الانترفيو.

الغلط الشائع:

```
Request

↓

new Servlet()
```

وده خطأ.

الصحيح:

```
Servlet واحدة

↓

Request 1 → Thread 1

↓

Request 2 → Thread 2

↓

Request 3 → Thread 3
```

كل المستخدمين بيستخدموا نفس Object.

---

# لماذا هذا مهم؟

لأن لو عملت:

```java
public class UserServlet extends HttpServlet {

    private String username;

}
```

فالمتغير ده هيبقى مشترك بين كل المستخدمين.

وده يعمل مشاكل اسمها:

```
Race Conditions
```

لذلك Servlets لازم تكون Stateless.

---

# ما هو Thread؟

كل Request يدخل Tomcat، Tomcat يخصص له Thread من Thread Pool.

يعني:

```
Client A

↓

Thread 1

↓

Servlet
```

وفي نفس الوقت:

```
Client B

↓

Thread 2

↓

Servlet
```

---

# من أين تأتي HttpServletRequest؟

لما يجي Request:

```
GET /users?id=5
```

Tomcat يعمل Parsing.

ثم ينشئ Object:

```
HttpServletRequest
```

ويضع داخله:

- URL
    
- Query Parameters
    
- Headers
    
- Cookies
    
- Body
    
- Session
    

ثم يرسله للـ Servlet.

---

# HttpServletResponse

كذلك Tomcat ينشئ Response Object.

أنت فقط تكتب فيه.

مثلاً:

```java
response.setStatus(200);

response.getWriter().println("Hello");
```

ثم Tomcat يحوله إلى HTTP Response حقيقي ويرسله للمتصفح.

---

# ما هو Servlet API؟

هو Specification فيها Interfaces وClasses مثل:

```
Servlet

GenericServlet

HttpServlet

HttpServletRequest

HttpServletResponse

Filter

ServletContext

ServletConfig
```

Tomcat هو اللي ينفذ هذه الـ Specification.

---

# المشكلة في الـ Servlets

تخيل عندك API فيها:

```
GET /users

GET /courses

POST /courses

PUT /users

DELETE /reviews

...
```

هتحتاج عشرات أو مئات الـ Servlets.

وده كان صعب جدًا.

---

# هنا جاء Spring MVC

Spring قال:

بدل:

```
100 Servlet
```

نعمل:

```
Servlet واحدة فقط
```

واسمها:

**DispatcherServlet**

---

# ما هي DispatcherServlet؟

هي Servlet عادية جدًا.

لكنها تعتبر Front Controller.

يعني كل Requests تدخل عليها أولًا.

```
Browser

↓

Tomcat

↓

DispatcherServlet
```

مش على Controller.

---

# لماذا اسمها Dispatcher؟

لأنها تعمل Dispatch.

أي توزع الطلبات.

مثلًا:

```
GET /users
```

ترسل إلى:

```
UserController
```

أما:

```
GET /courses
```

ترسل إلى:

```
CourseController
```

---

# كيف تعرف الـ Controller الصحيح؟

Spring أثناء بدء التشغيل يعمل Scan.

ويقرأ:

```
@RestController

@RequestMapping

@GetMapping

@PostMapping
```

ثم يبني جدولًا داخليًا.

مثلاً:

|URL|Controller Method|
|---|---|
|/users|UserController.getUsers()|
|/courses|CourseController.getCourses()|

هذا الجدول يسمى:

```
Handler Mapping
```

---

# ماذا يحدث عند تشغيل Spring Boot؟

بترتيب مبسط:

1. تبدأ الـ JVM.
    
2. يبدأ Spring Boot.
    
3. ينشئ `ApplicationContext`.
    
4. يعمل Component Scan.
    
5. ينشئ Beans.
    
6. يشغل Embedded Tomcat.
    
7. Tomcat ينشئ `DispatcherServlet`.
    
8. `DispatcherServlet` تجمع جميع الـ Mappings.
    
9. التطبيق يصبح جاهزًا لاستقبال الطلبات.
    

---

# ماذا يحدث عند وصول Request؟

لنفترض:

```
GET /api/users/10
```

الرحلة تكون:

```
Browser

↓

TCP Connection

↓

HTTP Request

↓

Tomcat

↓

DispatcherServlet

↓

Handler Mapping

↓

UserController

↓

Service

↓

Repository

↓

Hibernate

↓

Database

↓

Hibernate

↓

Repository

↓

Service

↓

Controller

↓

DispatcherServlet

↓

Tomcat

↓

Browser
```

---

# أين يعمل Spring Security؟

قبل أن يصل الطلب إلى `DispatcherServlet` يمر غالبًا على سلسلة من الـ Filters.

```
Browser

↓

Tomcat

↓

Servlet Filters

↓

Spring Security Filter Chain

↓

DispatcherServlet

↓

Controller
```

لذلك لو الـ JWT غير صحيح، الـ Controller لن يُستدعى أصلًا.

---

# أين تعمل Interceptors؟

بعد اختيار الـ Controller وقبل تنفيذه.

```
DispatcherServlet

↓

HandlerInterceptor

↓

Controller
```

تُستخدم لأشياء مثل Logging أو التحقق من صلاحيات إضافية أو قياس زمن التنفيذ.

---

# كيف يبني Spring الـ Response؟

إذا رجع الـ Controller:

```java
return user;
```

فـ Spring يستخدم `HttpMessageConverter` لتحويل كائن Java إلى JSON (عادةً باستخدام مكتبة Jackson)، ثم يكتب الـ JSON داخل `HttpServletResponse`، وبعدها يقوم Tomcat بإرساله للعميل.

---

# الفرق بين Tomcat و Servlet و DispatcherServlet

|العنصر|وظيفته|
|---|---|
|**Apache Tomcat**|Servlet Container يشغل التطبيق، يستقبل HTTP Requests، ويدير دورة حياة الـ Servlets.|
|**Servlet**|Java Class تستقبل الطلب وتعالجه وتكتب الاستجابة.|
|**DispatcherServlet**|Servlet خاصة بـ Spring MVC تستقبل جميع الطلبات وتوجهها إلى الـ Controller المناسب.|
|**Controller**|يحتوي منطق التعامل مع الـ HTTP Endpoint ويستدعي الـ Service.|
|**Service**|يحتوي الـ Business Logic.|
|**Repository**|يتعامل مع قاعدة البيانات.|

---

# الرسم الكامل

```text
                    Internet
                        │
                        ▼
                HTTP Request
                        │
                        ▼
               Apache Tomcat
                        │
                        ▼
        Servlet Filters (مثل Spring Security)
                        │
                        ▼
              DispatcherServlet
                        │
                        ▼
               Handler Mapping
                        │
                        ▼
                  Controller
                        │
                        ▼
                    Service
                        │
                        ▼
                  Repository
                        │
                        ▼
                Hibernate / JPA
                        │
                        ▼
                   Database
                        │
                        ▲
                (تعود النتيجة)
                        │
                        ▼
              DispatcherServlet
                        │
                        ▼
              HttpServletResponse
                        │
                        ▼
                 Apache Tomcat
                        │
                        ▼
                 HTTP Response
                        │
                        ▼
                     Browser
```

---

# أهم أسئلة الانترفيو المتعلقة بالموضوع

1. **What is a Servlet?**
    
    - Java class تعمل داخل Servlet Container لاستقبال ومعالجة HTTP Requests.
        
2. **What is Tomcat?**
    
    - Servlet Container (وWeb Server) مسؤول عن تشغيل تطبيقات Java Web وإدارة الـ Servlets.
        
3. **Does Spring Boot use Servlets?**
    
    - نعم، Spring MVC يعتمد على `DispatcherServlet` وهي Servlet تعمل فوق Tomcat.
        
4. **Is a new Servlet object created for every request?**
    
    - لا، يتم إنشاء Servlet واحدة غالبًا، بينما يُخصَّص Thread مختلف لكل Request.
        
5. **What is the role of DispatcherServlet?**
    
    - استقبال جميع الطلبات، اختيار الـ Controller المناسب، ثم تنسيق إنشاء الـ Response وإرجاعه للعميل.
        

---

إذا فهمت هذا الفصل بالكامل، فستكون فاهم الأساس الذي بُني عليه Spring MVC، وستجد أن كثيرًا من المفاهيم مثل **Filters، Interceptors، Spring Security، Request Mapping، وREST Controllers** أصبحت منطقية لأنها كلها تعمل فوق نفس البنية الأساسية.