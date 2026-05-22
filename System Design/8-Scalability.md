# 08 - القابلية للتوسع (Scalability)

> **التعريف:**  
> القابلية للتوسع هي قدرة النظام على التعامل مع زيادة الحمل أو عدد المستخدمين بشكل سهل وبتكلفة مناسبة، عن طريق إضافة موارد للنظام.

---

# 📌 لماذا الـ Scalability مهمة ؟

- أي System ال load/traffic عليه **لا يظل ثابتًا**
    
- عدد المستخدمين والطلبات يزيد مع الوقت
    
- لذلك يجب تصميم النظام بحيث يتحمل النمو بدون انهيار أو تدهور كبير في الأداء
    

---

# ⚠️ تدهور الأداء (Performance Degradation)

عندما يزيد الـ Load عن حد معين، يبدأ أداء النظام في التدهور.

---

## Design 1 ❌ —  

- زمن الاستجابة (Response Time) يزيد بسرعة جدًا مع زيادة عدد الـ Requests
    
- نقطة التدهور تحصل مبكرًا
    
- بعد نقطة التدهور:
    
    - النظام يصبح بطيئًا جدًا
        
    - تجربة المستخدم تسوء
        

---

## Design 2 ✅ — تصميم جيد

- زمن الاستجابة يظل ثابتًا ومستقرًا لفترة أطول
    
- نقطة التدهور تأتي متأخرًا
    
- النظام يتحمل ضغطًا أكبر قبل أن يتأثر
    

---

> 💡 **الهدف الأساسي:**  
> تحريك نقطة التدهور لأبعد نقطة ممكنة ناحية اليمين على محور الـ Load.

---

# 📊 أنواع الـ Scalability بالنسبة للـ Resources

|نوع الـ Scalability|السلوك|
|---|---|
|**Linear**|مثالي — مضاعفة الموارد = مضاعفة الأداء|
|**Sub-linear**|جيد — يوجد بعض الـ Overhead لكن مازال يتوسع بشكل جيد|
|**Near-flat**|مقبول|
|**Negative (Degradation)**|❌ سيء — إضافة موارد تجعل الأداء أسوأ|

---

> إضافة الموارد يجب أن تحسن الأداء دائمًا أو على الأقل تحافظ عليه.  
> إذا كانت إضافة الموارد تقلل الأداء → فهناك مشكلة جوهرية في الـ Architecture.

---

# 🔁 أنواع الـ Scalability

```txt
        ↑ Scale Up (Vertical)
        |
        |————————————→ Scale Out (Horizontal)
       /
      ↙ Team / Organizational Scalability
```

---

# 1. التوسع الرأسي (Vertical Scalability / Scale Up)

> يعني زيادة إمكانيات نفس الجهاز.

مثل:

- CPU أقوى
    
- RAM أكبر
    
- SSD أسرع
    

---

## ✅ المميزات

- أي Application تقريبًا يستفيد منها
    
- غالبًا لا تحتاج لتعديل الكود
    
- نقل التطبيق لجهاز أقوى سهل نسبيًا
    

---

## ❌ العيوب

- يوجد حد أقصى لحجم أي Machine
    
- النظام يظل مركزيًا (Centralized)
    

وبالتالي يصعب تحقيق:

- High Availability
    
- Fault Tolerance
    

---

# مثال

بدل ما السيرفر عنده:

```txt
8 GB RAM
```

نزوده لـ:

```txt
64 GB RAM
```

---

# 2. التوسع الأفقي (Horizontal Scalability / Scale Out)

> يعني تشغيل أكثر من نسخة من التطبيق على أكثر من Machine. 
> add more resources

---

# مثال

بدل:

```txt
Server واحد
```

يبقى عندك:

```txt
10 Servers
```

ويتم توزيع الـ Requests عليهم.

---

## ✅ المميزات

- لا يوجد حد نظري للتوسع
    
- يمكن إضافة أو إزالة Servers بسهولة
    
- إذا صُمم بشكل صحيح يمكن تحقيق:
    
    - High Availability
        
    - Fault Tolerance
        

---

## ❌ العيوب

- يحتاج تعديلات معمارية في البداية
    
- يزيد الـ Complexity
    
- يحتاج تنسيق بين الـ Servers
    

---

# أمثلة على المشاكل في Horizontal Scaling

- Session Management
    
- Distributed Cache
    
- Database Replication
    
- Synchronization
    

---

# 3. Team / Organizational Scalability

> الـ Architecture تؤثر أيضًا على سرعة الشغل بين الفرق.

---

# مشكلة الـ Monolith الكبير

لو عندك:

- 20 Team
    
- وكلهم يعملون على نفس المشروع
    

ستظهر مشاكل مثل:

- اجتماعات كثيرة
    
- Merge Conflicts
    
- صعوبة فهم الـ Business Logic
    
- Testing أبطأ وأصعب
    
- Onboarding صعب للمطورين الجدد
    

---

# الحل

تقسيم النظام إلى:

- Modules
    
- Services
    
- Microservices
    

---

## ✅ الفوائد

كل Team:

- تمتلك Service خاصة بها
    
- تطور بشكل مستقل
    
- تختبر بشكل مستقل
    
- تعمل Deploy بدون التأثير على باقي الفرق
    

---

# لماذا الـ Microservices ظهرت ؟

أحد أهم الأسباب هو:

```txt
Team Scalability
```

وليس فقط الـ Performance.

---

# 🧠 أهم المفاهيم

|المفهوم|الشرح|
|---|---|
|**Degradation Point**|النقطة التي يبدأ عندها الأداء في التدهور|
|**Vertical Scaling**|أسهل لكن محدود|
|**Horizontal Scaling**|أصعب لكنه شبه غير محدود|
|**Team Scalability**|الـ Architecture تؤثر على سرعة الفريق|
|**Linear Scalability**|أفضل نوع — الموارد تزيد → الأداء يزيد بنفس النسبة|

---

# الفرق الحقيقي بين Vertical و Horizontal

|Vertical|Horizontal|
|---|---|
|جهاز واحد أقوى|عدة أجهزة|
|أسهل|أعقد|
|محدود|شبه غير محدود|
|لا يوفر Fault Tolerance|يوفر Fault Tolerance|
|لا يوفر High Availability|يوفر High Availability|

---

# مثال مهم جدًا

## Vertical Scaling

عندك:

```txt
1 Server
```

والـ CPU وصل 100%.

الحل:

```txt
تزود قوة الـ Server
```

---

## Horizontal Scaling

بدل ما تستخدم Server أقوى:

```txt
تزود عدد الـ Servers
```

ويتم توزيع الحمل عليهم باستخدام:

```txt
Load Balancer
```

---

# العلاقة مع Node.js و NestJS

Node.js عبارة عن:

```txt
Single Threaded Runtime
```

يعني كل Instance لديها Thread واحد فقط.

لذلك غالبًا في الإنتاج الحقيقي:

- نشغل أكثر من Instance
    
- أو أكثر من Container
    
- أو أكثر من Pod في Kubernetes
    

وده شكل من أشكال:

```txt
Horizontal Scaling
```

---

# مثال واقعي

لو عندك NestJS API:

والـ Traffic زاد جدًا…

بدل ما ترفع إمكانيات Machine فقط:

✅ الأفضل غالبًا:

- تشغيل أكثر من NestJS Instance
    
- وضع Load Balancer أمامهم
    

---

# الشكل الشائع في Production

```txt
Users
   ↓
Load Balancer
   ↓
-------------------
|    |    |    |
API  API  API  API
```

---

# نقطة مهمة جدًا

الـ Scalability ليست فقط:

```txt
زيادة عدد الـ Servers
```

لكن أيضًا:

- تحسين الـ Architecture
    
- تقليل الـ Bottlenecks
    
- تحسين الـ Database Design
    
- استخدام Caching
    
- تقليل Blocking Operations
    

---

# 🚀 الخلاصة

## الهدف من الـ Scalability

هو أن النظام يتحمل النمو بدون انهيار.

---

# أنواع الـ Scalability

## Vertical Scaling

- أسهل
    
- لكن محدود
    

---

## Horizontal Scaling

- أعقد
    
- لكنه الأفضل للأنظمة الكبيرة
    

---

## Team Scalability

- الـ Architecture تؤثر على إنتاجية الفريق
    

---

# أهم فكرة

> النظام الجيد ليس الذي يعمل فقط…  
> بل الذي يستمر في العمل حتى مع زيادة الضغط.

---

# Topics مرتبطة

- [[Load Balancing]]
    
- [[High Availability]]
    
- [[Fault Tolerance]]
    
- [[Microservices]]
    
- [[CAP Theorem]]
    

---

_المصدر: ITI Open Source Track — Software Architecture & System Design, Lecture 8_