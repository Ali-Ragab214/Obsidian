ممتاز، السلايدز دي فيها أغلب أساسيات Spring Security والـ Enterprise Security. هشرحها كأنك داخل Interview في شركة، وبالتفصيل، مع أمثلة وDiagrams بسيطة.

---

# أولاً: Enterprise Request Lifecycle

السلايد الأولى بتشرح رحلة أي Request من أول ما يخرج من المتصفح لحد ما يوصل للـ Spring Boot Application.

```
Client
   |
HTTPS Request
   |
DNS/CDN
   |
WAF
   |
Reverse Proxy
   |
Load Balancer
   |
API Gateway (اختياري)
   |
Spring Security
   |
Business Logic
   |
Database
```

---

## 1) Client Layer

هو أي Client بيستخدم السيستم.

- Browser
    
- Mobile App
    
- React
    
- Angular
    
- External API
    
- IoT
    

مثال

```
Chrome

GET /products
```

---

## 2) Edge Layer

دي أول حاجة تستقبل الريكوست.

وظيفتها

- DNS
    
- CDN
    
- DDOS Protection
    
- TLS Termination
    
- Bot Detection
    

---

### CDN

يحفظ الصور والـ CSS والـ JS قريب من المستخدم.

بدل

```
Egypt ---> USA Server
```

يبقى

```
Egypt ---> Egypt Edge Server
```

أسرع بكتير.

---

### DDOS Protection

لو مليون Request جه مرة واحدة

Cloudflare

هيوقفها قبل ما توصل للسيرفر.

---

### TLS Termination

بدل ما Spring Boot يفك HTTPS

Cloudflare أو Nginx هما اللي يفكوه.

وده بيقلل الحمل.

---

## 3) Security Layer

دي أهم Layer.

فيها

- WAF
    
- SQL Injection Protection
    
- XSS
    
- XXE
    
- SSRF
    
- IP Blocking
    

---

### WAF

Web Application Firewall

بيفحص الريكوست.

مثال

واحد بعت

```
?id=1 OR 1=1
```

هيمنعه.

---

## 4) Web Proxy

زي

- Apache
    
- Nginx
    

وظيفته

- Reverse Proxy
    
- Compression
    
- Static Files
    
- Logging
    
- Keep Alive
    

---

مثال

بدل كل Request يروح لـ Spring

```
Browser

↓

Nginx

↓

Spring Boot
```

---

## 5) Load Balancer

لو عندك

```
Server1

Server2

Server3
```

الـ Load Balancer يوزع الريكوست عليهم.

مثلاً

```
Request1 -> Server1

Request2 -> Server2

Request3 -> Server3
```

---

الـ Algorithms

### Round Robin

```
1
2
3
1
2
3
```

---

### Least Connections

يبعت للسيرفر الأقل ضغط.

---

### Sticky Session

يرجع نفس العميل لنفس السيرفر.

---

---

# API Gateway

في الصورة مكتوب Kong.

وده أشهر API Gateway.

---

هو بيكون قبل الـ Microservices.

```
Client

↓

Gateway

↓

User Service

↓

Order Service

↓

Product Service
```

---

الـ Gateway يقدر يعمل

- Authentication
    
- Authorization
    
- JWT Validation
    
- Rate Limiting
    
- Logging
    
- CORS
    
- Routing
    
- Monitoring
    

---

## Rate Limiting

يعني تمنع شخص يعمل مليون Request.

مثلاً

```
100 Requests / minute
```

لو بعت

```
101
```

يرجع

```
429 Too Many Requests
```

وده يمنع

- Spam
    
- Brute Force
    
- DDOS
    

---

# Authentication vs Authorization

أشهر سؤال Interview.

Authentication

يعني

**Who are you?**

---

Authorization

يعني

**What can you do?**

---

مثال

بتدخل شركة.

الأمن

يسألك البطاقة.

دي

Authentication

بعدها

يسمحلك تدخل الدور الخامس.

دي

Authorization

---

في Spring Security

```
Login

↓

Authentication

↓

JWT Generated

↓

Authorization
```

---

مثال

```
Ali
```

Role

```
ADMIN
```

يقدر

```
DELETE USER
```

لكن

```
Ahmed
```

Role

```
USER
```

ميقدرش.

---

# Principal

وده مهم جداً.

Principal

هو

**أي Entity داخل النظام تم إثبات هويته.**

غالباً

User

لكن ممكن يكون

- Robot
    
- Service
    
- API Client
    

---

يعني

```
Principal

↓

Ali
```

أو

```
Principal

↓

Payment Service
```

---

في Spring

```
Authentication extends Principal
```

يعني Authentication فيه Principal.

---

داخل Authentication

عندنا

```
Principal

Credentials

Authorities

Details

Authenticated
```

---

## 1) Principal

هو المستخدم نفسه.

مثلاً

```
Ali
```

أو

```
User Object
```

---

## 2) Credentials

بيانات تسجيل الدخول.

مثلاً

```
Password

OTP

JWT أثناء التحقق
```

بعد الـ Login غالبًا Spring يمسح الـ Password من الذاكرة لأسباب أمنية.

---

## 3) Details

بيانات إضافية.

مثلاً

```
IP

Browser

Session ID

Location
```

---

## 4) Authenticated

Boolean

```
true

false
```

هل الشخص تم تسجيل دخوله؟

---

## 5) Authorities

الـ Permissions.

مثلاً

```
READ

WRITE

DELETE
```

---

Diagram

```
Authentication
      |
-----------------------------------
| Principal       -> Ali
| Credentials     -> password
| Details         -> IP
| Authorities     -> READ,WRITE
| Authenticated   -> true
-----------------------------------
```

---

# Sessions

زمان كان Login بيشتغل كده.

```
Client

↓

Username

Password

↓

Server

↓

Create Session

↓

Store Session In Memory

↓

Return Cookie
```

---

بعدها

كل Request

يبعت

```
Cookie

SESSIONID
```

---

السيرفر

يجيب الـ Session.

---

# What is Cookie؟

الـ Cookie

هي ملف صغير جداً يخزنه الـ Browser.

بيستخدمه في

- Session
    
- Language
    
- Theme
    
- Remember Me
    

---

مثال

```
Set-Cookie:

SESSIONID=ABC123
```

المتصفح يحفظها.

بعدها

```
Cookie:

SESSIONID=ABC123
```

يبعتها تلقائياً.

---

## Types of Cookies

### Session Cookie

تعيش لحد ما تقفل المتصفح.

مثال

```
Login Session
```

---

### Persistent Cookie

بتعيش أيام أو شهور.

مثال

```
Remember Me

Dark Mode

Language
```

---

# Stateful Application

يعني

السيرفر بيحتفظ بحالة المستخدم.

مثلاً

```
User

↓

Server

↓

Session Stored
```

---

كل Request

السيرفر

يروح يدور

على الـ Session.

---

Diagram

```
Client

↓

Cookie

↓

Server

↓

Session

↓

Memory
```

---

### Advantages

- أسهل للمبتدئين.
    
- يمكن إلغاء الـ Session بسهولة.
    
- مناسب للتطبيقات التقليدية.
    

---

### Disadvantages

- يستهلك Memory.
    
- صعب في التوسع (Scalability).
    
- يحتاج Sticky Sessions أو Session Replication.
    
- أداء يقل مع كثرة المستخدمين لأن السيرفر يحتفظ ببيانات جلسات كثيرة ويبحث عنها باستمرار.
    

---

# Stateless Application

السيرفر

لا يحتفظ بأي Session.

كل Request يحتوي كل المعلومات المطلوبة.

---

مثال

```
Authorization

Bearer JWT
```

كل Request

```
GET /products

Authorization:

Bearer eyJ...
```

السيرفر يتحقق من الـ JWT ثم ينفذ الطلب دون الرجوع لذاكرة جلسات.

---

Diagram

```
Client

↓

JWT

↓

Server

↓

Verify JWT

↓

Response
```

---

### Advantages

- أسرع في الـ Scaling.
    
- لا يحتاج Memory للـ Sessions.
    
- مناسب جداً للـ REST APIs وMicroservices.
    
- أي Server يقدر يخدم أي Request.
    

---

### Disadvantages

- لا يمكن إلغاء JWT بسهولة قبل انتهاء صلاحيته (إلا باستخدام Blacklist أو حلول مشابهة).
    
- حجم كل Request أكبر قليلاً بسبب إرسال الـ Token.
    
- يجب حماية الـ Token جيداً.
    

---

# Stateful vs Stateless

|Feature|Stateful|Stateless|
|---|---|---|
|Server stores session|✅|❌|
|Memory usage|High|Very Low|
|Scalability|Lower|Excellent|
|REST APIs|❌ غالبًا|✅ الأفضل|
|JWT|❌|✅|
|Performance|يقل مع زيادة الجلسات|أفضل غالبًا لأنه لا يدير Sessions|
|Security|يعتمد على Session وCookies، معرض لـ CSRF إذا لم يُحمَ|يعتمد على حماية الـ Token، وأقل تعرضًا لـ CSRF عند استخدام Bearer Token|

---

# Why REST APIs use Stateless؟

لأن

كل Request مستقل.

```
GET /users
```

لا يعتمد على أي Request قبله.

وده يخلي

- أسهل في الـ Scaling.
    
- مناسب للـ Cloud.
    
- مناسب للـ Load Balancer.
    
- مناسب للـ Microservices.
    

---

# Basic Auth → Session → JWT → OAuth → OIDC

تطور الـ Authentication.

## 1) Basic Authentication

كل Request

يبعت

```
Username

Password
```

وده غير آمن نسبيًا إذا لم يُستخدم مع HTTPS، كما أنه يرسل بيانات الاعتماد في كل طلب.

---

## 2) Session Authentication

Login مرة.

بعدها

Cookie.

---

## 3) JWT

Login مرة.

بعدها

JWT.

---

## 4) OAuth2

ليس Login.

هو

تفويض.

يعني

تسمح لتطبيق يستخدم بياناتك بدون إعطائه Password.

مثال

```
Spotify

↓

Google
```

---

## 5) OIDC

هو

OAuth2

Identity.

يعني

يعرف

مين المستخدم.

مثال

```
Login with Google
```

---

# CSRF (Brief)

Cross-Site Request Forgery

هو هجوم يخدع المتصفح لإرسال Request للموقع وأنت مسجل دخول بالفعل.

مثال:

1. أنت مسجل دخول في البنك.
    
2. فتحت موقعًا خبيثًا.
    
3. الموقع الخبيث يرسل طلب تحويل أموال إلى البنك.
    
4. المتصفح يرسل الـ Session Cookie تلقائيًا، فيظن البنك أن الطلب منك.
    

```
Victim Browser
      |
      | (Cookie automatically sent)
      v
Bank Server
```

### الحماية

- استخدام CSRF Token.
    
- استخدام `SameSite` للـ Cookies.
    
- التحقق من `Origin` أو `Referer`.
    
- في REST APIs التي تستخدم JWT في `Authorization` Header، يكون خطر CSRF أقل بكثير لأن المتصفح لا يرسل الـ Bearer Token تلقائيًا مثل الـ Cookie.
    

---

دي تعتبر الأساسيات اللي لازم تكون فاهمها قبل الدخول في تفاصيل Spring Security Filters و`SecurityContext` و`AuthenticationProvider` و`UserDetailsService` وازاي الـ JWT بيتحقق داخل الـ Filter Chain.