# 🔹 What is an API?

![Image](https://substackcdn.com/image/fetch/%24s_%21g3db%21%2Cf_auto%2Cq_auto%3Agood%2Cfl_progressive%3Asteep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4a38175b-11e8-40ae-879c-ab3ce2027089_2008x1252.png)

![Image](https://www.researchgate.net/publication/369336081/figure/fig3/AS%3A11431281129405338%401679524682508/REST-API-request-response-flow.jpg)

![Image](https://eu-central-1-shared-euc1-02.graphassets.com/AvHQ3RDvFSousA8iwElOKz/resize%3Dwidth%3A2048/bixkNpDoQMepBnMH4hv6)

![Image](https://eu-central-1-shared-euc1-02.graphassets.com/AvHQ3RDvFSousA8iwElOKz/mV5ebqbwTFeFHUjv9bP7)

## 📌 Definition

**API (Application Programming Interface)**  
هو مجموعة من القواعد (rules) والبروتوكولات اللي بتسمح لتطبيقين يتكلموا مع بعض.

ببساطة:

> API هو “contract” بين **Client** و **Server**

### مثال عملي:

لما الموبايل بتاعك يطلب بيانات الطقس:

- App (Client)
    
- يبعث Request
    
- للـ Server
    
- يرجعله Response في شكل JSON
    

---

# 🔹 API Types

## 1️⃣ SOAP API

![Image](https://imagedelivery.net/PVooPtpJE-25QaNkbEuXvw/images/soapui/docs/wsdl-request-raw-editor_new.png/public)

![Image](https://www.researchgate.net/publication/220144364/figure/fig3/AS%3A667632667729923%401536187440188/SOAP-based-Web-services-architecture.png)

![Image](https://www.researchgate.net/publication/282330205/figure/fig1/AS%3A614156868788230%401523437816369/An-example-WSDL-file.png)

![Image](https://i.sstatic.net/7cSQn.png)

### 📌 SOAP = Simple Object Access Protocol

- بيستخدم XML فقط
    
- Strict Protocol
    
- بيعتمد على WSDL
    
- أقدم من REST
    

### Example Request:

```xml
<Envelope>
   <Body>
      <GetUser>
         <Id>1</Id>
      </GetUser>
   </Body>
</Envelope>
```

### ✅ Advantages

- Security عالي (WS-Security)
    
- مناسب للـ Enterprise systems
    
- Transactions قوية
    

### ❌ Disadvantages

- Heavy
    
- XML بس
    
- معقد شوية
    

---

## 2️⃣ REST API

![Image](https://miro.medium.com/1%2Am3jEkdc9SKTK6vNPhRHCqg.jpeg)

![Image](https://media.licdn.com/dms/image/v2/D4D12AQHxG4Prn4ZrBQ/article-cover_image-shrink_600_2000/article-cover_image-shrink_600_2000/0/1718998421506?e=2147483647&t=-2LQBR1-lDFdKqTy66DNfGm4cTz7xy27k8-QYYsYaio&v=beta)

![Image](https://www.researchgate.net/publication/342575529/figure/fig5/AS%3A908293190991879%401593565380171/API-Response-in-JSON-Format.png)

![Image](https://www.researchgate.net/publication/351288013/figure/fig1/AS%3A1019536090357760%401620087754018/Example-HWBI-REST-API-GET-response-body-A-successful-content-negotiation-will-return-a.png)

### 📌 REST = Representational State Transfer

- بيعتمد على HTTP
    
- Stateless
    
- بيدعم JSON (الأشهر)
    

### Example:

```
GET /users/1
```

Response:

```json
{
  "id": 1,
  "name": "Ali"
}
```

### HTTP Methods:

|Method|Purpose|
|---|---|
|GET|Read|
|POST|Create|
|PUT|Update|
|DELETE|Delete|

---

## 3️⃣ GraphQL (مهم تعرفه)

![Image](https://images-www.contentful.com/fo9twyrwpveg/3HXxHkavTUnHUXviiVxF7N/72a7593a0443ceb83489e88d2eb5dd00/image2.png)

![Image](https://cdn.cosmicjs.com/7d9a6f90-7080-11eb-87a2-9be5e90cdf74-GraphQLvsRest.png)

![Image](https://docs.dgraph.io/assets/images/schema-sketch-d2fd2389debb46a39bff09551dc92edf.png)

![Image](https://cdn.sanity.io/images/p2zxqf70/production/4b7f2edb1e23c9afd77908b058e6904ada648cc2-900x519.png)

- Client بيطلب exactly اللي هو عايزه
    
- Single endpoint
    
- Flexible جدا
    

---

# 🔹 What is Web API؟

Web API = API بيشتغل over HTTP.

يعني:

> أي API شغال على الويب اسمه Web API

مثال:  
ASP.NET Web API

---

# 🔹 What is REST API؟

REST API هو Web API لكن:

- بيتبع REST principles
    
- Stateless
    
- Resources based
    

---

# 🔹 Advantages of Using API

### ✅ 1. Separation of Concerns

Frontend ≠ Backend

### ✅ 2. Reusability

نفس API يستخدمه:

- Web
    
- Mobile
    
- Desktop
    

### ✅ 3. Scalability

### ✅ 4. Security (لو معمولة صح)

---

# ❌ Disadvantages

- Network latency
    
- Security risks
    
- Versioning complexity
    
- Overhead لو مش محتاجها
    

---

# 🔹 API Economy

## 📌 يعني ايه API Economy؟

الشركات بتبني APIs عشان:

- تبيع خدماتها
    
- تفتح Platform ecosystem
    
- تعمل integrations
    

### Examples:

- Stripe API
    
- Twilio API
    
- Google Maps API
    

### ليه مهم؟

- Monetization
    
- Business scaling
    
- Partnerships
    

---

# 🔹 API Routing

![Image](https://www.ifourtechnolab.com/pics/routin.webp)

![Image](https://miro.medium.com/1%2AE4WwbdA-NSJJimYZA9r6Cg.png)

![Image](https://assets.apidog.com/blog/2024/03/path-parameter-example.png)

![Image](https://d3rtegloudc34g.cloudfront.net/_next/image/?q=75&url=https%3A%2F%2Fik.imagekit.io%2Fd9mvewbju%2FCourse%2Fpath-segments-example_aHA2yEdGQ&w=3840)

Routing = ازاي Request توصل للـ Controller المناسب.

Example:

```
/api/users/1
```

ASP.NET Example:

```csharp
[Route("api/[controller]")]
```

Types:

- Convention based
    
- Attribute routing
    

---

# 🔹 Pagination

![Image](https://assets.bytebytego.com/diagrams/0076-api-pagination-101.png)

![Image](https://miro.medium.com/1%2As_7rrm3GS7fd6xUCDflo_g.png)

![Image](https://images-www.contentful.com/fo9twyrwpveg/ekZ79QV8z7bD4t4B1o7dq/5e5845fbbe09da690ba43214aac653d3/image4.png)

![Image](https://miro.medium.com/0%2A8G6mdYzH1a8D5LrX)

لما يكون عندك 1 مليون record.

بدل ما ترجّعهم كلهم:

```
GET /users?page=1&pageSize=10
```

### Types:

1. Offset Pagination
    
2. Cursor Pagination (أفضل performance)
    

Example Response:

```json
{
  "data": [],
  "page": 1,
  "totalPages": 50
}
```

---

# 🔹 API Versioning

مهم جدا في production systems.

### ليه؟

عشان لو غيرت API متكسرش clients.

### طرق:

1. URL versioning
    

```
/api/v1/users
```

2. Query versioning
    

```
/api/users?version=1
```

3. Header versioning
    

### Best Practice:

URL versioning هو الأشهر.

---

# 🔹 Swagger & OpenAPI

![Image](https://static1.smartbear.co/swagger/media/images/tools/opensource/swaggerhub-swaggerui.png)

![Image](https://s3.us-west-1.wasabisys.com/idbwmedia.com/images/api/yamlscreenshot.png)

![Image](https://static1.smartbear.co/swagger/media/images/tools/opensource/swagger_ui.png)

![Image](https://swagger.io/docs/_astro/hero-img.CQIKAqF0_1585RE.webp)

## 📌 OpenAPI

Specification بيحدد شكل API:

- Endpoints
    
- Request
    
- Response
    

## 📌 Swagger

Tool بيستخدم OpenAPI عشان:

- يولد Documentation
    
- يعمل UI تجرب منه ال endpoints
    

Example:  
في ASP.NET:

```csharp
builder.Services.AddSwaggerGen();
```

---

# 🔥 حاجات مهمة جدا عن API (ناس كتير بتنسى)

## 1️⃣ Authentication & Authorization

- JWT
    
- OAuth2
    

## 2️⃣ Status Codes

|Code|Meaning|
|---|---|
|200|OK|
|201|Created|
|400|Bad Request|
|401|Unauthorized|
|404|Not Found|
|500|Server Error|

## 3️⃣ Rate Limiting

عشان تمنع abuse

## 4️⃣ CORS

## 5️⃣ Idempotency

## 6️⃣ Caching

---

# 🎯 كـ Backend Developer (زيك يا علي)

بما إنك شغال ASP.NET و Clean Architecture  
لازم تبقى فاهم:

- REST principles
    
- HTTP lifecycle
    
- Middleware
    
- Filters
    
- Model Binding
    
- Validation
    
- Global Exception Handling
    
- Logging
    
- API Security
    

---

# 🔥 خلاصة كبيرة تربطلك الدنيا

|Concept|ببساطة|
|---|---|
|API|طريقة تواصل بين systems|
|Web API|API شغال over HTTP|
|REST API|Web API ملتزم REST|
|SOAP|Protocol قديم بيعتمد XML|
|Swagger|Documentation Tool|
|OpenAPI|Specification|
|Pagination|تقسيم البيانات|
|Versioning|إدارة التغييرات|
|API Economy|APIs = Business asset|

---
         ┌──────────────┐
         │   Frontend   │
         └───────┬──────┘
                 │ HTTP
                 ▼
         ┌──────────────┐
         │   API Layer  │
         │  (REST API)  │
         └───────┬──────┘
                 │
                 ▼
         ┌──────────────┐
         │   Database   │
         └──────────────┘





---

# 🔥 أولاً: يعني إيه REST API بجد؟

## 📌 REST مش Protocol

REST هو **Architectural Style** اتعمل بواسطة  
Roy Fielding في رسالة الدكتوراه بتاعته سنة 2000.

REST بيقولك:

> ازاي تصمم Web API بشكل scalable, simple, performant

---

# 🧠 REST API = API بيحقق 6 Constraints

## 1️⃣ Client-Server

![Image](https://substackcdn.com/image/fetch/%24s_%21g3db%21%2Cf_auto%2Cq_auto%3Agood%2Cfl_progressive%3Asteep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4a38175b-11e8-40ae-879c-ab3ce2027089_2008x1252.png)

![Image](https://www.researchgate.net/publication/364044816/figure/fig3/AS%3A11431281087355504%401664551895898/Front-end-and-back-end-separation-framework.jpg)

![Image](https://madooei.github.io/cs421_sp20_homepage/assets/client-server-1.png)

![Image](https://www.altexsoft.com/media/2021/03/layered-architecture.png)

- Frontend مسؤول عن UI
    
- Backend مسؤول عن Business Logic
    
- بينهم API
    

---

## 2️⃣ Stateless (أهم نقطة) 

# 🔹 1️⃣ Stateless API

## 📌 التعريف:

الـ **Stateless** معناها إن **كل Request مستقل بذاته**  
السيرفر _مش بيخزن أي معلومات_ عن العميل بين الطلبات.

كل Request لازم يحتوي على كل المعلومات اللي السيرفر محتاجها عشان يرد.

---

## 🎯 مثال عملي

تخيل API تسجيل دخول:

Client  --->  Server  
POST /login  
{  
  "email": "ali@gmail.com",  
  "password": "123"  
}

السيرفر بيرجع Token:

{  
  "token": "abc123"  
}

بعد كده أي Request لازم تبعت الـ Token:

GET /profile  
Authorization: Bearer abc123

السيرفر مش فاكر علي عمل login قبل كده  
هو بس بيتأكد من الـ Token في كل مرة.

---

## 📊 Diagram

Request 1  ----> Server  
Request 2  ----> Server  
Request 3  ----> Server  
  
(Each request independent)

---

## ✅ مميزات Stateless

- scalable جدًا 🚀
    
- أسهل في التوزيع على عدة سيرفرات
    
- أقل استهلاك memory
    
- مناسب للـ REST APIs
    

---

## ❌ عيوب

- لازم تبعت بيانات كتير مع كل request
    
- ممكن يزيد حجم البيانات



![Image](https://substack-post-media.s3.amazonaws.com/public/images/7e4801c3-e3aa-4ab6-8fe6-759af4a1f91a_1684x1196.png)

![Image](https://miro.medium.com/1%2A_njX4rWrMoYhgIbF6Piprg.jpeg)

![Image](https://assets.bytebytego.com/diagrams/0244-jwt-101-key-to-stateless-authentication.png)

![Image](https://images.ctfassets.net/23aumh6u8s0i/RqMA41JEHFthu4sEZ5zd4/8f6438a08464257c4787a9a221d8d4b2/4)

كل Request لازم يحتوي على كل المعلومات.

يعني:

```http
GET /orders
Authorization: Bearer eyJhbGci...
```

السيرفر مش بيخزن session.

### ليه ده مهم؟

- Scalability
    
- Load Balancing سهل
    
- Microservices friendly
    

---

## 3️⃣ Resource-Based

REST بيفكر في Resources مش Actions.

❌ غلط:

```
/getUsers
/createUser
```

✅ صح:

```
GET    /users
POST   /users
GET    /users/5
DELETE /users/5
```

---

## 4️⃣ Uniform Interface

![Image](https://www.altexsoft.com/media/2021/03/uniform_interface.png)

![Image](https://images.ctfassets.net/vwq10xzbe6iz/5sBH4Agl614xM7exeLsTo7/9e84dce01735f155911e611c42c9793f/rest-api.png)

![Image](https://miro.medium.com/1%2AXZ_C8CgA3v9HCZFLsQiyMQ.png)

![Image](https://community.ptc.com/legacyfs/online/thingworx/2959_rest_syntax.png)

كل حاجة تمشي بنفس النظام:

- URL يمثل Resource
    
- HTTP Method يمثل Action
    

---

## 5️⃣ Cacheable

Response ممكن يتحط في Cache:

```http
Cache-Control: max-age=3600
```

---

## 6️⃣ Layered System

![Image](https://svbtleusercontent.com/u3vxovtCYz6R4TaScpv78L0xspap.png)

![Image](https://learn.microsoft.com/en-us/azure/architecture/microservices/images/gateway.png)

![Image](https://ngrok.com/blog-assets/images/2024-04-17-reverse-proxy-vs-api-gateway/blog-reverse-proxy-vs_diagram-reverse-proxy-%281%29.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2A-vJjmtQPPNNk8F9Ln2sETw.png)

Client ممكن يكلم:

- API Gateway
    
- Proxy
    
- Load Balancer
    
- Server
    

من غير ما يعرف.

---

# 🔥 أهم Diagrams لازم أي Backend يفهمها

---

# 1️⃣ HTTP Lifecycle Diagram

![Image](https://user-images.githubusercontent.com/773481/181182150-8cc2b6c4-2b50-4e85-afd7-e5c2d1c98b2c.png)

![Image](https://miro.medium.com/1%2Au7vAE2TovG8M1sTK5cWmKw.png)

![Image](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index/_static/middleware-pipeline.svg?view=aspnetcore-10.0)

![Image](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index/_static/request-delegate-pipeline.png?view=aspnetcore-10.0)

### Flow:

1. Client يبعث Request
    
2. Middleware Pipeline
    
3. Routing
    
4. Controller
    
5. Service Layer
    
6. Database
    
7. Response يرجع JSON
    

---

# 2️⃣ REST CRUD Mapping Diagram

![Image](https://parallelagile.github.io/CodeBot/images/domain-model/generated-api-layers.jpg)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AzfteFouDU_fDepZf.jpg)

![Image](https://www.altexsoft.com/media/2021/03/api_types.png)

![Image](https://restful-api-design.readthedocs.io/en/latest/_images/concepts.png)

|Operation|HTTP|Example|
|---|---|---|
|Create|POST|/orders|
|Read|GET|/orders/5|
|Update|PUT|/orders/5|
|Delete|DELETE|/orders/5|

---

# 3️⃣ REST vs SOAP Diagram

![Image](https://media.licdn.com/dms/image/v2/D4D12AQG_0xpiYc80bA/article-cover_image-shrink_600_2000/article-cover_image-shrink_600_2000/0/1692707797311?e=2147483647&t=r6FZk9UmTGurG2pMfd3ikOc29qhPyEEbv9PKrBXQ4dg&v=beta)

![Image](https://images.ctfassets.net/lzny33ho1g45/JNLA2tnVWEbD7dfJuLLw4/ed19f6c3593fd106ba95538e0fbdc924/soap-vs-rest.webp)

![Image](https://www.researchgate.net/publication/275461741/figure/fig10/AS%3A294464443699214%401447217202901/Performance-Analysis-of-REST-VS-SOAP.png)

![Image](https://www.researchgate.net/publication/312566917/figure/fig2/AS%3A452632142716929%401484927320140/REST-and-SOAP-response-time-measurements-10.png)

|REST|SOAP|
|---|---|
|JSON|XML فقط|
|Lightweight|Heavy|
|Simple|Complex|
|Stateless|ممكن Stateful|

---

# 4️⃣ REST API Clean Architecture Flow (مهم ليك 👇)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ADCXtTdOyywZc-iNfcaDEuQ.png)

![Image](https://miro.medium.com/1%2Asura91gPMoCjPNvZWsAO_g.png)

![Image](https://static.apiseven.com/apisix-webp/2022/09/20/6328a7ca42d26.webp)

![Image](https://static.apiseven.com/apisix-webp/2022/09/20/63289f6ad9d61.webp)

### Flow:

Controller  
⬇  
Application Layer (CQRS / MediatR)  
⬇  
Domain  
⬇  
Infrastructure  
⬇  
Database

ده نفس اللي بتطبقه في مشروعك Services.

---

# 🔥 فهم أعمق لـ REST

## REST = Resource Oriented Design

مثال من مشروعك:

❌ غلط:

```
/bookService
/cancelBooking
```

✅ صح:

```
POST   /bookings
DELETE /bookings/{id}
```

---

# 🔥 Best Practices في REST API

### 1️⃣ Use Nouns not Verbs

```
/users ✔
/getUsers ✖
```

### 2️⃣ Use Plural

```
/orders
```

### 3️⃣ Proper Status Codes

- 200 OK
    
- 201 Created
    
- 204 No Content
    
- 400 Bad Request
    
- 404 Not Found
    

---

# 🔥 Example Full REST API Design

## Users Resource

```
GET    /users
GET    /users/1
POST   /users
PUT    /users/1
DELETE /users/1
```

Response:

```json
{
  "id": 1,
  "name": "Ali",
  "email": "ali@mail.com"
}
```

---

# 🧠 الفرق الحقيقي بين Web API و REST API

|Web API|REST API|
|---|---|
|أي API over HTTP|Web API + REST constraints|

يعني:

> كل REST API هو Web API  
> مش كل Web API هو REST

---

# 🔥 أخطاء ناس كتير بتقع فيها

- بيعمل endpoint لكل Action
    
- بيستخدم POST لكل حاجة
    
- مش بيستخدم status codes صح
    
- مش بيحترم idempotency
    
- بيكسر versioning
    

---

# 🎯 خلاصة احترافية ليك يا علي

بما إنك Backend Developer:

لازم تبقى قادر:

- تصمم Resource Model صح
    
- تطبق Stateless JWT
    
- تستخدم proper status codes
    
- تعمل pagination
    
- تطبق versioning
    
- تبني Clean Architecture
    
- توثق باستخدام Swagger / OpenAPI
    

---

