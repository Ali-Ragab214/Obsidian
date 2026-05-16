

---

# أولاً: يعني ايه Quality Attributes ؟

العنوان في السلايد:

## Quality attributes (Non-Functional requirements)

يعني:  
مش النظام بيعمل ايه ❌  
لكن **النظام بيعمله ازاي وبجودة قد ايه** ✅

---

## الفرق بين Functional و Non-Functional

### Functional Requirement

بتوصف سلوك السيستم.

مثال:

- المستخدم يعمل Login
    
- المستخدم يبحث عن منتج
    
- المستخدم يحجز خدمة
    

دي اسمها **Features**.

---

### Non-Functional Requirement (Quality Attributes)

بتوصف **جودة تنفيذ الفيتشر**.

يعني:

- السرعة
    
- الأمان
    
- الاستقرار
    
- القابلية للتطوير
    

📌 وده أهم جزء في السلايد:

> systems are redesigned not because of functional requirements

الجملة دي مهمة جدًا ومعناها:

💥 أغلب الأنظمة بيتعملها Re-design مش لأن الفيتشر ناقصة  
لكن لأن **الجودة سيئة**.

وده حقيقي 100%.

---

# ليه الأنظمة بتتفشل؟

السلايد كتب الأسباب 👇

- isn’t fast enough → بطيء
    
- doesn’t scale → مش بيستحمل ضغط
    
- slow to develop → التطوير بقى صعب
    
- hard to maintain → صعب يتصان
    
- not secure enough → مش آمن
    

دي أسباب انهيار شركات فعليًا.

أمثلة واقعية:

- موقع ينهار وقت التخفيضات
    
- أبلكيشن بيفتح ببطء
    
- كل تعديل يبوظ السيستم
    

📌 ده كله Quality Attributes مش Features.

---

# أهم جملة في السلايد 💎

> they have direct correlation with the architecture of our system

يعني ايه؟

يعني:

## الArchitecture بيتبني علشان Quality Attributes مش Features.

الـ Login ممكن يتعمل بـ 100 طريقة  
لكن:

- لو عايزه سريع
    
- وآمن
    
- وScalable
    

هتختار Architecture مختلفة تمامًا.

---

# مثال السلايد (مهم جدًا)

الجملة:

> when a user clicks search … results within 100 ms

خلينا نفكها:

### الجزء الأول:

"when user clicks search"  
ده Functional ✔️

### الجزء التاني:

"within 100 ms"  
ده Non-Functional ✔️

📌 إذن الجملة فيها الاتنين مع بعض.

والسلايد كتب:

> within at most 100 milliseconds → performance

صح 100%.

ده اسمه **Performance Requirement**.

---

# مثال تاني: Availability

السلايد:

> online store must be available 00.0% of times

المقصود هنا:

- 99%
    
- 99.9%
    
- 99.99%
    

وده اسمه:

## Availability

يعني السيستم يفضل شغال.

---

# مثال تالت: Deployability

> team can deploy twice a week

دي نقطة قوية جدًا ومش الناس بتاخد بالها منها.

دي معناها:

## سرعة نشر التحديثات

لو السيستم صعب يتنشر → يبقى Architecture سيئة.

---

# أهم قاعدة في Quality Attributes

السلايد كتب:

> must be measurable & testable

دي جملة ذهبية 🏆

لازم تكون بالأرقام.

❌ سيستم سريع  
✅ الاستجابة أقل من 200ms

❌ سيستم آمن  
✅ 2FA + encryption + OWASP compliance

❌ سيستم مستقر  
✅ uptime 99.9%

---

# أهم جملة في المحاضرة كلها 💥

> no single software architecture can provide all quality attributes

يعني ايه؟

مستحيل تحقق كل الصفات بأقصى درجة.

لازم تعمل Trade-offs.

---

## مثال واضح جدًا

لو عملت النظام:

- Secure جدًا 🔒
    
- Encryption في كل حاجة
    

هيبقى:

- أبطأ 🐢
    

لو عملته:

- Fast جدًا ⚡
    

ممكن:

- تقلل security
    

📌 اسمها:

## Trade-offs in Architecture

وده قلب الـ System Design.

---

# ندخل على الجزء الأخير

## Feasibility (إمكانية التنفيذ)

السلايد بدأ:

> make sure system capable of delivering what client asks

يعني قبل ما تقول "هنعمل السيستم"  
لازم تسأل:

هل ينفع أصلاً نعمله؟

---

# أنواع Feasibility (مهم جدًا)

أي System Analyst لازم يدرس 4 أنواع:

---

## 1️⃣ Technical Feasibility

هل التكنولوجيا تسمح؟

مثال:  
العميل عايز:

- تطبيق يخدم 10 مليون مستخدم من أول يوم
    

هل عندك:

- Cloud؟
    
- Load balancing؟
    
- Budget للسيرفرات؟
    

لو لا → غير قابل تقنيًا.

---

## 2️⃣ Financial Feasibility

هل فيه فلوس تكفي؟

ممكن النظام يكلف:

- 1 مليون دولار
    

والعميل معاه:

- 50 ألف فقط 😅
    

يبقى غير قابل ماليًا.

---

## 3️⃣ Time Feasibility

هل الوقت يكفي؟

العميل:

> عايز نسخة زي Uber خلال شهر

🚨 مستحيل.

---

## 4️⃣ Operational Feasibility

هل الناس هتعرف تستخدمه؟

مثال:  
نظام معقد جدًا لموظفين غير تقنيين.

هيفشل حتى لو ممتاز تقنيًا.

---

# الخلاصة النهائية 🔥

## Functional Requirements

ماذا يفعل النظام؟

## Non-Functional (Quality Attributes)

كيف يفعل النظام وبأي جودة؟

أهم Quality Attributes:

- Performance
    
- Scalability
    
- Availability
    
- Security
    
- Maintainability
    
- Deployability
    

ولازم تكون:  
✔️ measurable  
✔️ testable

ولا يوجد Architecture مثالي.

---

لو تحب نطبق الكلام ده عملي على مشروع التخرج بتاعك ونسحب Quality Attributes حقيقية ليه 🔥 ده تدريب مهم جدًا.