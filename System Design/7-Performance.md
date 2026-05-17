
---

# 🚀 أولاً: يعني ايه Performance أصلاً ؟

لما نقول السيستم سريع ولا بطيء… بنقيس حاجتين أساسيتين: 

1️⃣ **Response Time (Latency)** → الوقت اللي المستخدم بيستناه  
2️⃣ **Throughput** → السيستم يقدر يخدم كام Request في الثانية

الاتنين دول أهم Metric في أي System Design. 

---

# ⏱️ 1) Response Time (Latency)

تعريفه في النوتس صح:

> الوقت بين إرسال الطلب واستلام الرد

يعني المستخدم ضغط Login → امتى استلم النتيجة؟

لو 200ms → ممتاز  
لو 5 ثواني → المستخدم هيقفل الأبلكيشن 😅

---

## ✨ Response Time بيتقسم لحاجتين

### 1️⃣ Processing Time (وقت المعالجة)

ده الوقت اللي السيستم **بيشتغل فعلاً فيه**.

مثال:

- Validate request
    
- Query database
    
- Process logic
    
- Build response
    

ده الوقت "المفيد".

---

### 2️⃣ Waiting Time (وقت الانتظار)

ده أخطر جزء 😈

ده الوقت اللي الطلب:

- مستني DB
    
- مستني thread فاضي
    
- مستني network
    
- مستني queue
    

وده غالباً بيكون **أكبر سبب البطء**.

💡 أهم حقيقة:

> أغلب latency في الأنظمة مش Processing… بل Waiting.

---

## 🎯 مثال عملي

تخيل endpoint:

GET /doctors

السيناريو:

- Processing = 50ms
    
- Database waiting = 300ms
    
- Network = 100ms
    

يبقى:

```
Response time = 450ms
```

شايف؟  
المعالجة 50ms بس 😅 والباقي انتظار.

---

# 🔥 أهم قاعدة في الأداء

> لو عايز تسرّع السيستم → قلل WAITING مش PROCESSING

وده سبب وجود:

- Cache
    
- Queues
    
- Async jobs
    
- Indexing
    

---

# ⚡ 2) Throughput

تعريف النوتس:

> عدد الـ requests اللي السيستم يقدر يخدمها في الثانية

مثال:

- Server يخدم 100 request/sec
    
- يبقى Throughput = 100 RPS
    

---

## العلاقة بين Latency و Throughput

دي علاقة مهمة جداً في الانترفيو 👇

لو Response time بقى أطول → Throughput يقل  
ليه؟

لأن السيرفر بيقعد وقت أطول مع كل request.

مثال:

لو كل request بياخد 1 ثانية  
Server عنده 10 threads  
يبقى أقصى throughput = 10 req/sec

لو خفضنا latency لنص ثانية  
يبقى throughput = 20 req/sec 🔥

---

# 📊 Response Time Distribution (أهم جزء)

السلايد التاني بيسأل سؤال خطير جداً:

> نقيس الأداء بالـ Average ولا Median ولا Max ؟

ودي نقطة interview مشهورة جدًا 👌

---

## ❌ Average (المتوسط) خداع كبير

مثال:

```
100ms
120ms
110ms
90ms
5 seconds 😱
```

المتوسط ≈ 1.1 ثانية  
لكن أغلب المستخدمين بياخدوا 100ms بس!

يبقى المتوسط **كذاب** 😅

---

## ❌ Maximum (أسوأ حالة)

لو اعتمدنا على Max  
أي request بطيء يخرب الأرقام.

مش عملي.

---

## ✅ الحل الصح: Percentiles 🎯

أهم مفهوم في Performance كله.

بنقول:

- P50 → 50% من الطلبات أسرع من كده (median)
    
- P95 → 95% من الطلبات أسرع من كده
    
- P99 → 99% من الطلبات أسرع من كده
    

وده اللي الشركات الكبيرة بتستخدمه.

---

### 🎯 مثال

لو قلنا:

```
P95 = 300ms
```

يعني:

> 95% من المستخدمين بياخدوا response أقل من 300ms

وده مهم جداً لتجربة المستخدم.

---

## 📈 الصورة بتاعة Histogram معناها ايه؟

الرسم ده بيوري توزيع أوقات الاستجابة.

مثلاً:

- معظم الطلبات 100–200ms
    
- شوية قليلين 500ms
    
- قليل جداً 2 ثانية
    

الطلبات البطيئة دي اسمها:

> Long Tail 🐍

ودي أخطر حاجة في الأداء.

لأن:

- المستخدم اللي بياخد 2 ثانية = هيشتكي 😅
    

---

# 🧠 ليه بنركز على P95 و P99 ؟

لأن المستخدمين مش Average.

لو عندك 1 مليون مستخدم:

- 1% = 10,000 مستخدم 😳
    

يعني لو P99 بطيء → آلاف المستخدمين متضايقين.

---

# 🛠️ نطبق الكلام ده إزاي كمطور Backend ؟

ده أهم جزء ليك 🔥

---

## 1️⃣ Measure First (لازم تقيس)

قبل ما تحسن الأداء لازم تقيسه.

أدوات:

- Logging response time
    
- Monitoring (Prometheus / Grafana)
    
- APM tools
    

في Nest مثلاً:

- Interceptor يحسب زمن كل request
    

---

## 2️⃣ قلل Waiting Time

أهم حلول الأداء:

### 🟢 Database

- Indexes
    
- Pagination
    
- Avoid N+1 queries
    

### 🟢 Caching

- Redis
    
- Cache frequently requested data
    

مثال:

```
GET /categories
```

مش لازم DB كل مرة.

---

### 🟢 Async Jobs

حاجات مش لازم تحصل فوراً:

- Send email
    
- Notifications
    
- Image processing
    

نستخدم:  
Queue (BullMQ)

بدل ما المستخدم يستنى.

---

### 🟢 Connection Pooling

بدل فتح اتصال DB لكل request.

---

## 3️⃣ Scalability

لو الضغط زاد:

- Horizontal scaling
    
- Load balancer
    

وده بيرفع Throughput.

---

# 🎯 الخلاصة اللي تحفظها للانترفيو

لو اتسألت عن Performance قول:

- أهم metric: Latency + Throughput
    
- Latency = processing + waiting
    
- أكبر مشكلة = waiting time
    
- لا نعتمد على average → نستخدم P95/P99
    
- نحل الأداء بـ:
    
    - caching
        
    - indexing
        
    - async jobs
        
    - scaling
        

---
# Degradation Point
باختصار:

**Degradation point** = النقطة اللي لما النظام يوصلها، يبدأ **الأداء يتراجع بشكل ملحوظ**.

---

مثلاً عندك API بيشتغل تمام مع 1000 request/sec، لما يوصل 5000 يبدأ الـ response time يتأخر، والـ errors تزيد — دي هي الـ degradation point.

---

**بيحصل عشان:**

- الـ CPU أو Memory وصل للـ limit
- الـ Database connections اتخنقت
- الـ Thread pool اتملى
- الـ Cache miss rate زادت

---

**في Spring Boot مثلاً** — لو الـ HikariCP connection pool عنده 10 connections وجالك 50 request في نفس الوقت، هتلاقي الـ requests الباقية بتستنى → ده degradation.

---

ببساطة: هو الـ **breaking point** بس قبل ما النظام يـ crash كامل — بيكون لسه شغال بسببطء أو بـ errors جزئية.