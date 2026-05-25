

> **المحاضرة دي بتغطي:** إيه هو الـ Load Balancer، ليه محتاجه، أنواعه، وإزاي بيشتغل في السيستمز الكبيرة.

---

## 📌 Part 1: ليه محتاج Load Balancer أصلاً؟

### المشكلة — بدون Load Balancer

تخيل عندك تطبيق زي SkillHub وعندك 3 servers عشان تتحمل الـ Load.

**السؤال:** الـ Client هيكلم مين منهم؟

```
Client → Server 1؟
Client → Server 2؟
Client → Server 3؟
```

لو كل Client اختار Server بنفسه → **Tight Coupling** يعني الـ Client عارف عناوين الـ Servers الداخلية — وده مشكلة كبيرة:

- لو Server 1 وقع → الـ Clients اللي بيكلموه هيتأثروا
- لو ضفت Server 4 جديد → محتاج تحدّث كل الـ Clients
- الـ Servers الداخلية مكشوفة للعالم الخارجي → أمان ضعيف

---

### الحل — Load Balancer = Single Server Abstraction

الـ Load Balancer بيوقف في النص ويقول للـ Client:

> "انت مش محتاج تعرف فيه كام server — كلمني أنا بس"

```
Client → Load Balancer → Server 1
                      → Server 2
                      → Server 3
```

الـ Client بيشوف حاجة واحدة بس — الـ Load Balancer. اللي جواه (كام server، فين، إيه نوعهم) ده مش شغلته.

> 💡 **ده اسمه Abstraction** — نفس المبدأ اللي اتكلمنا عنه في الـ API Design

---

## 📌 Part 2: Quality Attributes — فوائد الـ Load Balancer

### 1️⃣ Scalability (التوسع)

**Scaling Up — لما الـ Traffic يزيد:**

```
الـ Users زادوا → ضيف Server جديد → اللـ LB يبدأ يبعتله Requests تلقائياً
```

**Scaling Down — لما الـ Traffic يقل:**

```
الـ Users قلوا (مثلاً الساعة 3 الصبح) → شيل Server مش محتاجه → توفير فلوس ☑️
```

> 💡 **مثال عملي:** AWS Auto Scaling — بيضيف ويشيل servers تلقائياً حسب الـ Load

---

### 2️⃣ High Availability (التوافر العالي)

الـ Load Balancer بيعمل **Health Checks** دورية على كل server.

```
LB → Server 1: "هل أنت حي؟" → ✅ رد
LB → Server 2: "هل أنت حي؟" → ✅ رد
LB → Server 3: "هل أنت حي؟" → ❌ مش رادّ → خرج من الـ Pool
```

لو Server وقع → الـ LB بيوقف يبعتله Requests تلقائياً، والـ Users مش حاسين بحاجة.

> 💡 **مثال عملي:** Netflix — عندهم آلاف الـ Servers، لو 10 منهم وقعوا في نفس الوقت، الـ Users مش بيحسوا بأي حاجة

---

### 3️⃣ Performance / Throughput (الأداء)

**بدون LB:** Server واحد بياخد كل الـ Requests

```
100 req/sec → Server واحد → مش هيتحمل
```

**مع LB:** الـ Requests بتتوزع

```
100 req/sec → LB → Server 1: 33 req/sec
                 → Server 2: 33 req/sec
                 → Server 3: 34 req/sec
```

> ⚠️ **ملاحظة مهمة:** الـ LB بيضيف **Latency** صغيرة (وقت إضافي عشان يمر عليه)، بس ده مقبول جداً مقارنةً بالـ Performance الكلي اللي بيزيد

---

### 4️⃣ Maintainability (سهولة الصيانة)

لو محتاج تعمل Update لـ Server:

```
بدون LB: Server بيوقف → كل الـ Users بيتأثروا ❌

مع LB:
1. قول للـ LB "مش بتبعت لـ Server 1 دلوقتي"
2. Server 1 بيخلص الـ Requests الموجودة عنده
3. اعمل Update
4. رجّع Server 1 للـ Pool ✅
```

ده بيسمح بـ **Zero Downtime Deployment** — التطبيق شغال حتى وقت الـ Update

---

## 📌 Part 3: أنواع الـ Load Balancing

### النوع 1: DNS Load Balancing

#### إيه هو الـ DNS أصلاً؟

الـ DNS = **Domain Name System** = "دليل التليفون بتاع الإنترنت"

لما بتكتب `www.google.com` في المتصفح:

```
1. المتصفح بيسأل الـ DNS: "إيه عنوان www.google.com؟"
2. الـ DNS بيرد: "العنوان هو 142.250.185.46"
3. المتصفح بيكلم الـ IP ده مباشرة
```

> بدون DNS كنت لازم تحفظ IP لكل موقع بتدخله!

#### إزاي بيشتغل DNS Load Balancing؟

بدل ما الـ DNS يرجع IP واحد، بيرجع **قائمة من الـ IPs**:

```
"www.skillhub.com" → 
    123.53.0.1 (Server 1)
    123.53.0.2 (Server 2)
    123.53.0.3 (Server 3)
```

وبيوزعهم بطريقة **Round-Robin** — يعني:

- Client 1 → يجيله `123.53.0.1`
- Client 2 → يجيله `123.53.0.2`
- Client 3 → يجيله `123.53.0.3`
- Client 4 → يجيله `123.53.0.1` (من الأول تاني)

#### الـ TTL (Time To Live)

الـ DNS Response بيتـ Cache في أكتر من مكان:

- الـ Browser
- الـ OS
- الـ Router

الـ TTL بيقول "الـ Cache ده صالح لمدة كذا ثانية/دقيقة"

```
TTL = 300 ثانية (5 دقايق)
يعني كل 5 دقايق، الـ Client بيسأل الـ DNS تاني
```

#### ✅ مميزات DNS Load Balancing:

- **بسيط جداً** — مش محتاج hardware أو software إضافي
- **مجاني** — بييجي مع الـ Domain Name تلقائياً

#### ❌ عيوب DNS Load Balancing:

**عيب 1: مش بيعمل Health Check**

لو Server 2 وقع — الـ DNS مش عارف ومش هيوقف يبعت Clients ليه:

```
Client → DNS → "روح Server 2"
Client → Server 2 → ❌ Server وقع → الـ Client هيشوف Error
```

**عيب 2: Round-Robin بس — مش ذكي**

مش بيفرق بين:

- Server قوي (32 Core) وserver ضعيف (4 Core)
- Server مشغول 90% وserver فاضي 10%

كل الـ Requests بتتوزع بالتساوي حتى لو مش منطقي.

**عيب 3: الـ Client بيشوف IPs الـ Servers مباشرة**

```
DNS يرجع: [123.53.0.1, 123.53.0.2, 123.53.0.3]
```

Client خبيث يقدر يقول: "أنا هكلم 123.53.0.1 بس بكل الـ Requests" → يعمل **DDoS** على Server واحد بدل ما يتوزع

**عيب 4: الـ Caching بيسبب مشاكل**

لو شيلت Server وعوضته بحاجة تانية — الـ Clients اللي عندهم الـ Cache القديم هيفضلوا بيكلموا الـ IP القديم لحد ما الـ TTL ينتهي.

---

### النوع 2 و 3: Hardware & Software Load Balancers

دول بيحلوا المشاكل اللي الـ DNS ما حلهاش.

#### Hardware Load Balancer

جهاز مادي متخصص 100% في الـ Load Balancing:

- سريع جداً (Dedicated Hardware)
- غالي جداً
- بتلاقيه في الـ Banks والـ Enterprises الكبيرة

#### Software Load Balancer

برنامج بيشتغل على أي Server عادي:

- أرخص بكتير
- مرن أكتر
- أمثلة: **Nginx، HAProxy، Traefik**

#### الفرق عن الـ DNS:

**1. بيعمل Health Checks:**

```
كل 10 ثواني مثلاً:
LB → Server 1: ping → ✅
LB → Server 2: ping → ✅
LB → Server 3: ping → ❌ → شيله من الـ Pool فوراً
```

**2. Load Balancing ذكي:**

بيشوف:

- Current Load على كل server
- عدد الـ Open Connections
- نوع الـ Hardware

وبيبعت الـ Request للـ server الأنسب — مش بالدور بس.

**3. الـ Client بيشوف IP واحد بس (IP الـ LB):**

```
Client → 10.0.0.1 (LB IP فقط)
LB → يقرر يبعت لـ Server 2 مثلاً
```

الـ Servers الداخلية مخفية تماماً ← أمان أفضل

#### الاستخدام الداخلي (Internal Use):

الـ SW/HW Load Balancer مش بس للـ External Traffic — بيستخدم داخل السيستم بين الـ Services:

```
User → LB (خارجي) → Frontend Service
                   → Frontend Service
                   → Frontend Service
                         ↓
                   LB (داخلي) → Fulfillment Service
                              → Fulfillment Service
```

كل Service ليها LB خاص بيها جوّا السيستم.

---

### النوع 4: GSLB — Global Server Load Balancer

#### المشكلة اللي بيحلها:

لو عندك Users في أوروبا وأمريكا، والـ Servers بتاعتك في أمريكا بس:

```
User في ألمانيا → Request بيعدي المحيط الأطلنطي → Servers في أمريكا
                → وقت الاستجابة: 200ms+ ❌
```

#### الحل — GSLB:

الـ GSLB = **هجين بين DNS و Hardware/Software LB**

عنده:

- قدرة الـ DNS (بيرجع IP حسب اللي بيسأل)
- ذكاء الـ HW/SW LB (Health Checks + Smart Routing)

```
User في ألمانيا → GSLB → يشوف: أقرب Data Center = أوروبا
                       → يرجعله IP الـ LB في أوروبا
                       → وقت الاستجابة: 20ms ✅

User في كندا → GSLB → يشوف: أقرب Data Center = US East
                    → يرجعله IP الـ LB في US East
                    → وقت الاستجابة: 30ms ✅
```

#### الـ GSLB بيـ Route على أساس:

- **Location** — أقرب Data Center جغرافياً
- **Health** — لو Data Center في أوروبا وقع، يحوّل كل الـ Traffic لـ US East
- **Current Traffic** — لو Data Center مشغول، يبعت لتاني
- **CPU Load** — يبعت للـ Data Center الأقل تحميلاً
- **Bandwidth** — يبعت للـ Data Center الأسرع اتصالاً بالـ User

#### مثال عملي — Disaster Recovery:

```
Data Center في أوروبا اشتعلت فيه نار 🔥

بدون GSLB: كل الـ Users في أوروبا مش هيوصلوا للسيستم ❌

مع GSLB: اكتشف إن الـ Data Center وقع
         → حوّل كل الـ Traffic لـ US East تلقائياً
         → Users في أوروبا ممكن يحسوا ببطء شوية
         → بس السيستم فاضل شغال ✅
```

---

## 📌 Part 4: المقارنة الكاملة

||DNS LB|HW/SW LB|GSLB|
|---|---|---|---|
|**Health Check**|❌|✅|✅|
|**Smart Routing**|❌ Round-Robin بس|✅|✅|
|**Security**|❌ IPs مكشوفة|✅|✅|
|**Geographic**|❌|❌|✅|
|**التكلفة**|مجاني|متوسطة-عالية|عالية|
|**التعقيد**|بسيط جداً|متوسط|معقد|
|**الاستخدام**|مشاريع صغيرة|معظم المشاريع|Enterprise/Global|

---

## 📌 Summary — Mental Model

```
Load Balancer
│
├── ليه؟
│   ├── Scalability     → زيادة وتقليل الـ Servers حسب الحاجة
│   ├── High Availability → السيستم فاضل شغال لو Server وقع
│   ├── Performance     → توزيع الـ Load على Servers كتير
│   └── Maintainability → Update بدون Downtime
│
└── أنواعه
    ├── DNS LB          → بسيط ومجاني، بس مش ذكي
    ├── Hardware LB     → سريع وغالي، للـ Enterprises
    ├── Software LB     → مرن ورخيص (Nginx, HAProxy)
    └── GSLB            → للسيستمز العالمية، هجين DNS + HW/SW
```

---

## 🔗 Related Topics

- [[12 - API Design]]
- [[Microservices Architecture]]
- [[High Availability Patterns]]
- [[CDN - Content Delivery Network]]
- [[System Design - Scalability]]

---

> 📝 **ملاحظة المذاكرة:** في الـ Interviews بيسألوا كتير "لو عندك ملايين Users إزاي تصمم السيستم؟" — الـ Load Balancer دايماً جزء من الإجابة. افهم الفرق بين الأنواع الأربعة كويس.