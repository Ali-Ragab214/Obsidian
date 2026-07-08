ممتاز يا علي. هشرحلك الكود كأننا قاعدين في Technical Interview، وهقولك ليه كل سطر اتكتب، وإيه اللي بيحصل داخليًا في Spring، وكمان هقولك الأخطاء الموجودة في الكود لأنه فيه كام غلطة لو سبته كده.

---

# أولاً: الـ Flow كامل

قبل ما نشرح الكود لازم تفهم الـ Flow.

لما تعمل

```http
POST /api/v1/payements
```

The flow will be: 

```
Client
   │
   ▼
DispatcherServlet
   │
   ▼
IdempotencyInterceptor.preHandle()
   │
   ├── لو UUID موجود
   │      return false
   │      Stop
   │
   └── لو UUID مش موجود
          cache.add(uuid)
          return true
               │
               ▼
PaymentController
               │
               ▼
Service
               │
               ▼
Repository
               │
               ▼
Database
```

الـ Interceptor هو أول واحد بيمسك الـ Request قبل الـ Controller.

---

# PaymentController

```java
@RestController
```

يعني الكلاس ده هيستقبل HTTP Requests.

بدل ما يعمل

```
return View
```

هيعمل

```
return JSON
```

---

```java
@RequestMapping("/api/v1/payements")
```

ده Base URL.

يعني أي Method جواه هتبدأ بـ

```
/api/v1/payements
```

---

```java
@Slf4j
```

دى Annotation من Lombok.

بدل ما تكتب

```java
private static final Logger logger =
LoggerFactory.getLogger(PaymentController.class);
```

Lombok بيولدها أوتوماتيك.

يعني داخليًا بيحولها تقريبًا لـ

```java
private static final Logger log =
LoggerFactory.getLogger(PaymentController.class);
```

---

## @PostMapping

يعني

```
POST /api/v1/payements
```

---

## Method

```java
public void pay(...)
```

دي هتتنفذ فقط لو الـ Interceptor سمح.

لو الـ Interceptor رجع false

الميثود دي مش هتشتغل أصلاً.

---

## RequestHeader

```java
@RequestHeader String uuid
```

يعني

Spring هيدور على Header اسمه

```
uuid
```

مثلاً

```
POST /payments

uuid:1234
```

ويحطه في المتغير

```
uuid
```

---

## log.info()

```java
log.info(testMessage);
```

يعني

اطبع رسالة

بـ مستوى

```
INFO
```

---

## الـ Logging Levels

Spring بيستخدم Logback.

الـ Levels هم

```
TRACE
DEBUG
INFO
WARN
ERROR
```

---

مثلاً

```java
log.info("Payment Started");
```

هيطلع

```
INFO
Payment Started
```

---

```java
log.warn("Duplicate UUID");
```

هيطلع

```
WARN
Duplicate UUID
```

---

```java
log.error("Database Down");
```

هيطلع

```
ERROR
Database Down
```

---

# إزاي تشوف الـ Logs؟

بعد ما تعمل Run.

افتح

```
Run
```

أو

```
Services
```

هتلاقى تحت

```
Console
```

زي

```
2026-07-07
INFO
Payment Started
```

---

لو مش ظاهر

```
View

Tool Windows

Run
```

أو

```
Alt + 4
```

---

لو عايز كل INFO تظهر

فى

application.properties

```
logging.level.root=INFO
```

أو

```
logging.level.com.example=DEBUG
```

---

# ندخل على الـ Interceptor

```java
implements HandlerInterceptor
```

ليه؟

لأن Spring عنده Interface اسمها

```
HandlerInterceptor
```

فيها

```
preHandle()

postHandle()

afterCompletion()
```

أنت مهتم بـ

```
preHandle()
```

---

## ليه preHandle؟

لأنها بتتنفذ

قبل Controller.

وده المطلوب.

---

```java
private static Set<String> cache =
new HashSet<>();
```

دى أهم سطر.

---

## ليه Set؟

لأن

Set

لا تسمح بتكرار العناصر.

```
Ali

Ali

Ali
```

هيبقى

```
Ali
```

مرة واحدة.

---

## ليه HashSet؟

لأن

```
contains()
```

سريعة جدًا.

تقريبًا

```
O(1)
```

---

## ليه static؟

علشان

يبقى فيه نسخة واحدة فقط.

مش كل Request يعمل

HashSet جديدة.

---

## cache

هنا هو مجرد

Memory Cache.

مش Redis.

مش Database.

مجرد RAM.

---

# preHandle()

```java
public boolean preHandle(...)
```

ليه بترجع boolean؟

لو

```
true
```

كمّل.

لو

```
false
```

وقف.

---

## request

```java
HttpServletRequest request
```

ده الـ Request كله.

منه تقدر تجيب

Headers

Cookies

Body

Parameters

URI

Method

كل حاجة.

---

## response

```
HttpServletResponse
```

ده اللى هترجعه.

تقدر تغير Status Code.

تضيف Headers.

---

## handler

ده الـ Controller Method

اللى هيتم استدعاؤها.

---

## أول سطر

```java
String uuid =
request.getHeader("universal unique identifier");
```

يعني

هات Header

اسمه

```
universal unique identifier
```

بس هنا عندك مشكلة.

أنت فى الكنترولر

عامل

```
@RequestHeader String uuid
```

يعني الهيدر اسمه

```
uuid
```

لكن هنا بتدور على

```
universal unique identifier
```

يبقى الاتنين لازم يكونوا نفس الاسم.

مثلاً

```
Idempotency-Key
```

وده الأشهر.

---

بعدها

```java
if(cache.contains(uuid))
```

لو موجود

يبقى العملية اتنفذت.

---

```java
return false;
```

يعني

وقف الـ Request.

ومتدخلش Controller.

---

لو مش موجود

```java
cache.add(uuid);
```

ضيفه.

---

بعدها

```java
return HandlerInterceptor.super.preHandle(...)
```

الأفضل تكتب

```java
return true;
```

لأن الـ Default Implementation أصلًا بترجع true، وده أوضح.

---

# WebFilterConfigurer

```java
implements WebMvcConfigurer
```

Spring بيسمحلك تعدل

MVC Configuration

عن طريقه.

---

```java
addInterceptors()
```

بتقول لـ Spring

سجل الـ Interceptor ده.

---

```java
registry.addInterceptor(
new IdempotencyInterceptor())
```

يعني

كل Request

مر عليه الأول.

---

```java
.addPathPatterns("/api/v1/payments")
```

يعني

الـ Interceptor

هيشتغل

على الـ URL ده فقط.

---

# عندك غلطة كبيرة جدًا

فى Controller

```
/api/v1/payements
```

بزيادة حرف e.

وفى Config

```
/api/v1/payments
```

بدون e.

يبقى الـ Interceptor مش هيشتغل أصلاً.

---

# فيه مشكلة أكبر

أنت بتحفظ الـ UUID

قبل تنفيذ الدفع.

وده غلط.

ليه؟

تخيل

```
cache.add(uuid)
```

وبعدين

Database وقعت.

يبقى العملية فشلت.

لكن الـ UUID اتحفظ.

لما اليوزر يعمل Retry

هترفضه.

مع إنه مفروض يتنفذ.

---

فى الـ Production

بيكون الترتيب

```
Start Transaction

↓

نفذ Payment

↓

لو نجحت

احفظ Idempotency Key

↓

Commit
```

---

# مشاكل الكود

1. ❌ HashSet مش Thread Safe.
    
2. ❌ بيضيع بعد Restart.
    
3. ❌ مينفعش مع أكتر من Server.
    
4. ❌ مفيش Expiration.
    
5. ❌ بيضيف الـ UUID قبل نجاح العملية.
    
6. ❌ اسم الـ Header مختلف.
    
7. ❌ Path مختلف (`payements` vs `payments`).
    
8. ❌ `WebFilterConfigurer` لازم يكون عليه `@Configuration` (أو يتسجل كـ Bean)، وإلا Spring مش هيستخدمه.
    

---

أنا أرشح إننا في الخطوة الجاية نعمل **نسخة Production Ready** لنفس المثال باستخدام **Spring Boot + Redis**، لأن ده هو اللي هتلاقيه في الشركات، وساعتها هتشوف الـ Idempotency بتتعمل بشكل احترافي.