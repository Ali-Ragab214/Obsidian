

> **المحاضرة دي بتغطي:** تعريف الـ API، أنواعها، فوائدها، وأهم الـ Best Practices اللي لازم تطبقها كـ Software Architect.

---

## 📌 Part 1: Intro to APIs

### ما هو الـ API؟

بعد ما بنحدد كل الـ Functional Requirements للسيستم، بنبدأ نفكر في السيستم على إنه **Black Box** — يعني صندوق مجهول الداخل بس واضح من برّه.

الـ Black Box ده بيتميز بـ:

- **Behavior** → ماذا يفعل؟
- **Well-defined Interface** → إزاي بيتكلم مع العالم الخارجي؟

> 🔑 الـ **Interface** دي بتعتبر **contract** بين:
> 
> - **Engineers** اللي بيبنوا السيستم
> - **Client Applications** اللي بتستخدمه

---

### ليه بنسميه API؟

- الـ Interface ده بيتم استدعاؤه من **تطبيقات تانية (Applications)**
- فبسميه: **Application Programming Interface = API**
- في السيستمز الكبيرة، الـ API بيُستدعى remotely عبر الـ **Network**

---

### مين بيكلم الـ API؟

الـ Applications اللي بتكلم الـ API بتاعتنا ممكن تكون:

|النوع|المعنى|
|---|---|
|**Frontend Clients**|Mobile Apps / Web Browsers|
|**Backend Systems**|شركات تانية عندها Integration معانا|
|**Internal Systems**|أنظمة داخل نفس الـ Organization|

> ⚠️ كل **Component** في السيستم بتاعنا بيكون عنده **API** خاصة بيه، واللي بتُستدعى من Components تانية داخل السيستم.

---

## 📌 Part 2: API Categories

### الـ APIs متقسمة لـ 3 أنواع:

```
APIs
├── 1. Public APIs
├── 2. Private / Internal APIs
└── 3. Partner APIs
```

---

### 1️⃣ Public APIs

- متاحة للعموم (developers خارجيين)
- بتسمح لك بـ:
    - **التحكم في مَن** يستخدم السيستم من الخارج
    - **التحكم في كيف** يستخدمه
    - **أمان أفضل** (Better Security)
    - **حجب المستخدمين** المخالفين (Blacklisting)

> 💡 **مثال:** Twitter API، Google Maps API

---

### 2️⃣ Private / Internal APIs

- بتُستخدم داخل الـ Organization فقط
- الـ Components المختلفة في السيستم بتتكلم مع بعض عبرها
- مش مكشوفة للخارج خالص
- بتساعد في الـ **Internal Integration** بين الـ Microservices

> 💡 **مثال:** الـ Order Service بيكلم الـ Payment Service داخليًا

---

### 3️⃣ Partner APIs

- شبيهة بالـ Public APIs بس بـ **قيود**
- بتُكشف فقط لشركات/مستخدمين عندهم **علاقة تجارية** معانا في شكل:
    - **Customer Agreement** بعد شراء المنتج
    - **Subscription** للخدمة

> 💡 **مثال:** Stripe API — بس لو اشتريت خطة، Salesforce Partner APIs

---

## 📌 Part 3: Benefits of a Well-Designed API

فوايد الـ API المتصممة كويس:

1. **تسهيل الـ Integration:** Client يقدر يبدأ يبني فوق السيستم بتاعنا بسرعة وبسهولة
2. **Abstraction:** Client مش محتاج يعرف أي حاجة عن الـ Internal Design / Implementation
3. **Parallel Development:** بمجرد ما نعرّف ونكشف الـ API، الـ Clients يقدروا يعملوا Integration من غير ما يستنوا الـ Full Implementation
4. **Better Architecture:** الـ API بتساعدنا إحنا كمهندسين نصمم ونبني الـ Internal Structure بشكل أفضل — هي اللي بتحدد الـ Endpoints وأوجه الاستخدام

---

## 📌 Part 4: API Best Practices & Patterns

### 1️⃣ Complete Encapsulation (التغليف الكامل)

- الـ API لازم **تخبي** كل الـ Internal Details
- الـ Client ماشوفش:
    - نوع الـ Database
    - اللغة المستخدمة
    - الـ Internal Architecture
- لو غيّرت أي حاجة جوّا من غير ما تغير الـ API → الـ Client ماتأثرش

> ✅ **الهدف:** فصل كامل بين الـ Interface والـ Implementation

---

### 2️⃣ Easy to Use (سهولة الاستخدام)

الـ API الكويسة بتتميز بـ:

- **Naming واضح** للـ Endpoints والـ Parameters
- **Documentation** شاملة وسهلة
- **Consistent Design** — نفس الـ Pattern في كل الـ API
- **Meaningful Error Messages** — مش بس كود الـ Error، بل سبب الغلطة

> 💡 **Tip:** لو Developer جديد احتاج وقت طويل يفهم الـ API، فده مشكلة في التصميم مش في الـ Developer

---

### 3️⃣ Keeping Operations Idempotent (ثبات العمليات)

> 📖 **تعريف Idempotency:** "An operation doesn't have any additional effect on the result if it is performed more than once" → العملية لو تكررت أكتر من مرة، النتيجة بتفضل نفسها

#### ليه ده مهم جداً في الـ APIs؟

لأن الـ API بتشتغل عبر الـ **Network**، واللي بيبقى فيها:

- الرسالة ممكن تتضيع (Message Lost)
- الـ Response ممكن يتضيع (Response Lost)
- الرسالة ممكن ماتوصلش لأن Component مهم وقع

→ في الحالات دي، الـ Client بيعمل **Retry** تلقائياً → لازم العملية **تتحمل** الـ Retry من غير ما تعمل Side Effects

#### مثال:

|العملية|Idempotent؟|ليه؟|
|---|---|---|
|تحديث عنوان المستخدم لـ "Cairo"|✅ نعم|لو اتعملت 10 مرات، العنوان فاضل "Cairo"|
|إضافة $10 لرصيد المستخدم|❌ لأ|كل مرة بتضيف $10 جديدة|
|جلب بيانات المستخدم (GET)|✅ نعم|قراءة بس، مش بتغير حاجة|
|حذف Item بـ ID معين|✅ نعم|اتحذف أو مش موجود — نفس النتيجة|

> ⚡ **القاعدة:** في الـ Distributed Systems، صمم عملياتك دايماً تكون **Idempotent** قدر الإمكان

---

### 4️⃣ API Pagination (التصفيح)

#### المشكلة:

لما الـ Response يكون **كبير جداً** (Very Large Payload أو Dataset)، الـ Client مش هيقدر يتعامل معاه:

- المتصفح / الـ App ممكن يتعلق أو يقع
- الـ Network ممكن تتحمل أكتر من طاقتها

#### تخيل:

- بتفتح Gmail → بيظهرلك كل الإيميلات اللي بعتّهم من 10 سنين؟ 🤯
- بتدور على منتج في أمازون → بيظهرلك مليون نتيجة في صفحة واحدة؟ 🤯

#### الحل — Pagination:

بيسمح للـ Client بـ:

- طلب **Segment صغير** من الـ Response
- تحديد **الحجم الأقصى** لكل Response (Page Size)
- تحديد **Offset** — يعني من فين تبدأ في الـ Dataset

#### إزاي بيشتغل؟

```
GET /products?limit=20&offset=0   → أول 20 منتج
GET /products?limit=20&offset=20  → الـ 20 اللي بعدهم
GET /products?limit=20&offset=40  → الـ 20 اللي بعدهم
...وهكذا
```

> 🔁 عشان تاخد الـ Segment الجاي، بتزود الـ Offset بمقدار الـ Limit

#### أنواع الـ Pagination:

|النوع|إزاي بيشتغل|الاستخدام|
|---|---|---|
|**Offset Pagination**|`limit` + `offset`|بسيط، مناسب للبيانات الثابتة|
|**Cursor-based Pagination**|`cursor` → ID آخر عنصر|أحسن للبيانات اللي بتتغير كتير (Social Feeds)|
|**Page-based Pagination**|`page` + `per_page`|للـ UIs اللي فيها أرقام صفحات|

---

### 5️⃣ Asynchronous Operations (العمليات غير المتزامنة)

#### المشكلة:

بعض العمليات محتاجة **وقت طويل** جداً عشان تخلص — مش ممكن تسيب الـ Client يستنى:

- Running a big report بيكلم 50 database
- Big Data Analysis بيسكن ملايين records
- Compression of large video files

#### الحل — Asynchronous API Pattern:

بدل ما الـ Client يستنى، السيستم:

```
1. Client بيبعت Request
2. السيستم بيرد فوراً بـ Response فيه:
   - Job ID
   - Status: "Processing"
3. العملية بتشتغل في الـ Background
4. Client يقدر يتابع الـ Status بـ Job ID
   أو السيستم يعمل Callback/Webhook لما يخلص
```

#### إزاي بيعمل الـ Client Polling؟

```
POST /reports/generate
→ Response: { "job_id": "abc123", "status": "pending" }

GET /reports/abc123/status
→ Response: { "status": "processing", "progress": "45%" }

GET /reports/abc123/status
→ Response: { "status": "completed", "result_url": "/reports/abc123/result" }
```

#### طرق الـ Async Notification:

|الطريقة|الوصف|
|---|---|
|**Polling**|الـ Client بيسأل كل فترة عن الـ Status|
|**Webhooks**|السيستم بيكلم الـ Client لما يخلص|
|**WebSockets**|Connection مفتوح طول الوقت، السيستم بيبعت Updates|
|**Server-Sent Events (SSE)**|Server بيبعت Updates للـ Client automatically|

---

## 📌 Summary — Mental Model

```
API Design
│
├── Categories
│   ├── Public    → للعموم، مع Controls
│   ├── Private   → داخلي فقط
│   └── Partner   → Business Relationship فقط
│
├── Benefits
│   ├── Easy Integration
│   ├── Abstraction
│   ├── Parallel Development
│   └── Better Architecture
│
└── Best Practices
    ├── 1. Complete Encapsulation   → إخفاء الـ Internal Details
    ├── 2. Easy to Use              → Clear Naming + Good Docs
    ├── 3. Idempotent Operations    → تحمّل الـ Retries
    ├── 4. Pagination               → تقطيع الـ Large Responses
    └── 5. Async Operations         → مش بنستنى على العمليات الطويلة
```

---

## 🔗 Related Topics

- [[REST API Design]]
- [[GraphQL vs REST]]
- [[Microservices Communication]]
- [[System Design - Load Balancer]]
- [[CAP Theorem]]

---

> 📝 **ملاحظة المذاكرة:** أهم نقطة في المحاضرة دي هي فهم **ليه** الـ Best Practices دي موجودة — مش حفظها. كل Practice بتحل مشكلة حقيقية بتيجي من طبيعة الـ Distributed Systems.