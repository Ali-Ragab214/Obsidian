تمام دي خطوة مهمة جداً في كورسات الـ System Design 👌  
الجزء ده اسمه **System Constraints** وده بييجي دايماً بعد ما نخلص:

- Functional Requirements
    
- Non-Functional Requirements
    

خلينا نشرحه بأسلوب الكورسات + أمثلة واقعية.

---

# أولاً — يعني إيه System Constraints ؟

السلايد قالت:

> A system constraint is a decision already made for us restricting our freedom

ترجمتها ببساطة:

قيود النظام = حاجات **إنت مش حر تختارها**  
قرارات اتاخدت قبل ما تبدأ التصميم.

يعني المعماري مش بيبدأ من صفر…  
في حاجات **مفروضة عليه**.

---

## مثال بسيط يفهمك الفكرة فوراً

تخيل عميل قالك:

> عايزين السيستم يكون بـ .NET

خلاص ❌  
مينفعش تقول: خلينا نستخدم Node.js.

ده Constraint.

---

## ليه Constraints مهمة جداً ؟

السلايد قالت:

> System constraints are pillars for software architecture

يعني إيه pillars؟

يعني:  
النظام كله بيتبني فوق القيود دي.

زي أساسات المبنى.

لو تجاهلتها → التصميم كله غلط.

---

## أهم فكرة في السلايد

> بدل ما تشوفها قيود… شوفها نقطة بداية

ليه؟  
لأنها بتقلل عدد الاختيارات وبتخليك تبدأ أسرع.

بدل ما تحتار بين 100 تقنية → عندك 3 بس.

---

# أنواع System Constraints (مهم جداً في المقابلات)

الكورس دايماً يقسمهم 3 أنواع:

1. Technical
    
2. Business
    
3. Legal
    

نشرح كل واحد بمثال كورسي 🔥

---

# 1️⃣ Technical Constraints

دي قيود تقنية مفروضة عليك.

السلايد أمثلة:

- locked to hardware
    
- forced programming language
    
- forced database
    

---

## أمثلة من الكورسات

### مثال 1 — شركة كبيرة عندها Legacy System

شركة عندها سيستم قديم بـ Java + Oracle.

طلبوا منك تبني خدمة جديدة.

هل ينفع تستخدم MongoDB؟  
غالباً ❌

ليه؟  
عشان لازم يتكامل مع النظام القديم.

ده اسمه:  
**Legacy constraint**

---

### مثال 2 — Cloud Provider Constraint

عميل يقولك:

> لازم نستخدم AWS فقط.

خلاص:

- لازم تستخدم S3 بدل Google Cloud Storage
    
- لازم تستخدم DynamoDB أو RDS
    

اختياراتك اتقلصت.

---

### مثال 3 — Mobile App Constraint

لو السيستم لازم يشتغل Offline.

ده Constraint كبير جداً يأثر على:

- Database choice
    
- Sync strategy
    
- Architecture
    

---

# 2️⃣ Business Constraints

دي قيود جاية من البزنس مش التكنولوجيا.

أخطر نوع في الحقيقة 😄

---

## مثال كورسي مشهور جداً

> Budget محدود

Startup عندها 5000$ بس Cloud.

هل ينفع تعمل Multi-Region deployment زي Netflix؟  
طبعاً لأ.

يبقى لازم تختار:

- Monolith
    
- Cheap hosting
    
- Minimal infrastructure
    

ده Business Constraint.

---

### مثال تاني

> Deadline بعد 3 شهور

العميل عايز MVP بسرعة.

هل هتعمل Microservices ؟  
غالباً لأ.

ليه؟  
Microservices:

- محتاج وقت كبير
    
- DevOps
    
- Monitoring
    

يبقى الحل:  
Monolith أولاً → بعدين نفككه.

وده بيتقال في كل كورسات System Design.

---

### مثال تالت

> لازم يدعم 1 مليون مستخدم من أول يوم

ده Business Constraint بيأثر على:

- Scalability
    
- Architecture
    
- Infrastructure
    

---

# 3️⃣ Legal Constraints

دي قيود قانونية.

في الكورسات دي نقطة متقدمة جداً.

---

## أشهر مثال في العالم

### GDPR (أوروبا)

القانون بيقول:  
بيانات المستخدم الأوروبي لازم تتخزن داخل أوروبا.

يعني:  
لو بتستخدم AWS  
لازم تستخدم Region أوروبا.

ده Constraint قوي جداً.

---

### مثال تاني

Banking systems:

- لازم encryption قوي
    
- لازم Audit logs
    
- لازم data retention سنوات
    

القانون بيفرض Architecture.

---

# أهم فكرة بعد أنواع القيود 🔥

السلايد بعدها دخلت على:

## Loosely Coupled Architecture

وده أهم ربط في السلايد.

---

# ليه بنتكلم عن Loosely Coupled بعد Constraints ؟

السلايد قالت:

> If limited to database or third-party service, don’t tightly couple system to it

دي جملة ذهبية في الـ System Design.

---

## يعني إيه Tightly Coupled ؟

يعني السيستم مربوط بتكنولوجيا واحدة جامد جداً.

لو غيرتها → السيستم ينهار.

❌ مثال سيء:

الكود كله مكتوب مخصوص لـ MySQL queries.

لو غيرنا لـ PostgreSQL → كارثة.

---

## الحل: Loose Coupling

نحط **Layer abstraction** بين السيستم والتكنولوجيا.

---

## مثال كورسي واضح جداً

بدل ما الكود يكلم Stripe مباشرة:

❌ غلط

```
PaymentService → Stripe API مباشرة
```

لو غيرنا Stripe → نكسر السيستم.

---

### الصح

نعمل abstraction layer:

```
PaymentService → PaymentGateway Interface → Stripe
                                          → PayPal
```

لو غيرنا Stripe → نغير implementation فقط.

وده اسمه:  
**Loose Coupling**

---

# ليه ده مهم جداً في وجود Constraints ؟

لأن القيود ممكن تتغير مستقبلاً.

اليوم:  
AWS + MySQL + Stripe

بكرة:  
Azure + PostgreSQL + PayPal 😅

لو السيستم tightly coupled → كارثة.

لو loosely coupled → تغيير بسيط.

---

# الخلاصة النهائية (ملخص الكورس)

بعد ما نحدد:

1. Functional reqs
    
2. Non-Functional reqs
    

نحدد:  
3) Constraints

أنواعها:

- Technical
    
- Business
    
- Legal
    

ثم نبني Architecture:

- تتوافق مع القيود
    
- لكنها تظل مرنة (Loose coupling)
    

---

لو تحب نطبق الكلام ده على مشروع الـ Services بتاعك ونكتب Constraints حقيقية ليه زي المقابلات بالظبط 🔥