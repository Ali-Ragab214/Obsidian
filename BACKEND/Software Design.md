

---

# 🎓 كورس Software Design من الصفر للاحتراف

## 🗺️ الخطة العامة (8 أسابيع)

|الأسبوع|الموضوع|
|---|---|
|1|مقدمة في Software Design + المبادئ الأساسية|
|2|Architectural Styles (أنماط المعمارية)|
|3|كيفية التفكير في التصميم (Workflow)|
|4|تفكيك الأنظمة + Coupling & Cohesion|
|5|Interface + API + Data Modeling|
|6|Persistence + Security Basics|
|7|الرسومات والـ UML والـ Documentation|
|8|مشروع متكامل + التوسع لأنظمة أكبر|

كل أسبوع:

- شرح مبسط
    
- مثال عملي
    
- تمرين
    
- أخطاء شائعة
    

---

# 🟢 الأسبوع 1 — مقدمة ومبادئ التصميم

## ما هو Software Design ؟

هو مرحلة التفكير قبل كتابة الكود.  
نحدد:

- النظام هيعمل إيه؟
    
- هيبقى شكله إيه؟
    
- الأجزاء هتتكلم مع بعض إزاي؟
    

💡 مثال بسيط:  
قبل ما تبني عمارة → المهندس بيعمل Blueprint.

الـ Software Design = Blueprint للبرنامج.

---

## لماذا مهم؟

بدونه هتحصل مشاكل:

- الكود يبقى صعب التعديل 😵
    
- Bugs كتير
    
- المشروع ينهار مع الوقت
    
- صعوبة إضافة Features
    

الهدف:

- كود سهل التعديل
    
- قابل للتوسع
    
- مفهوم
    

---

# ⭐ المبادئ الأساسية

## 1️⃣ SOLID Principles

أهم 5 قواعد في التصميم.

### S — Single Responsibility

الكلاس يكون ليه وظيفة واحدة فقط.

❌ غلط:

```csharp
class UserService {
  registerUser()
  sendEmail()
  saveToDatabase()
}
```

✅ صح:

- UserService → تسجيل المستخدم
    
- EmailService → إرسال الإيميل
    
- UserRepository → حفظ الداتا
    

---

### O — Open/Closed

الكود يكون:

- مفتوح للإضافة
    
- مغلق للتعديل
    

يعني نضيف features بدون ما نكسر القديم.

---

### L — Liskov Substitution

أي Class ابن ينفع يستخدم مكان الأب بدون مشاكل.

---

### I — Interface Segregation

ما تجبرش كلاس يطبق Methods مش محتاجها.

---

### D — Dependency Injection

اعتمد على Interfaces مش Classes مباشرة.

---

## 2️⃣ DRY — Don't Repeat Yourself

ما تكررش نفس الكود.

❌ غلط:  
نسخ نفس الكود في 5 أماكن.

---

## 3️⃣ KISS — Keep It Simple

أبسط حل غالباً هو الأفضل.

---

## 4️⃣ YAGNI — You Aren't Gonna Need It

ما تبنيش Features مش محتاجها حالياً.

---

## 🧠 مثال بسيط (To-Do App)

الـ App فيه:

- إضافة مهمة
    
- حذف مهمة
    
- عرض المهام
    

تصميم مبدئي:

```
[Controller] → [Service] → [Repository] → [Database]
```

---

## 📝 تمرين الأسبوع 1

صمم مبدئياً Note App يحتوي:

- إضافة Note
    
- تعديل Note
    
- حذف Note
    
- عرض Notes
    

اكتب:

- أسماء Classes المطلوبة فقط (بدون كود)
    

---

## ⚠️ أخطاء شائعة

❌ كتابة كود بدون تفكير  
❌ Class واحد يعمل كل حاجة  
❌ تعقيد الحل بدون داعي

---

## ملخص الأسبوع 1

- التصميم = تخطيط قبل الكود
    
- تعلمنا SOLID + DRY + KISS + YAGNI
    
- بدأنا التفكير في تقسيم النظام
    

---

# 🟡 الأسبوع 2 — Architectural Styles

## ما هي Architecture؟

شكل النظام الكبير وكيف الأجزاء تتواصل.

---

## 1️⃣ Monolithic Architecture

برنامج واحد كبير.

```
[UI + Logic + Database] → App واحد
```

✅ سهل البداية  
❌ صعب التوسع لاحقاً

---

## 2️⃣ Microservices

تقسيم النظام لخدمات صغيرة.

```
User Service
Order Service
Payment Service
```

✅ قابل للتوسع  
❌ أعقد في الإدارة

---

## 3️⃣ Layered Architecture (الأشهر)

```
Presentation → Application → Domain → Infrastructure
```

أنت تستخدمها في .NET Clean Architecture 👍

---

## 4️⃣ Client–Server

```
Frontend (Angular)
Backend (API)
Database
```

---

## 📝 تمرين الأسبوع 2

هل To-Do App:

- Monolith ولا Microservices ؟ ولماذا؟
    

---

## ⚠️ أخطاء شائعة

❌ استخدام Microservices لمشروع صغير  
❌ اختيار Architecture بدون سبب

---

## ملخص الأسبوع 2

- تعلمنا أشهر Architectures
    
- عرفنا امتى نستخدم كل واحد
    

---

# 🔵 الأسبوع 3 — كيف نفكر في التصميم (Workflow)

## خطوات تصميم أي نظام

### 1️⃣ Requirements Gathering

نفهم المطلوب.

مثال To-Do:

- المستخدم يسجل
    
- يضيف مهمة
    
- يعدل مهمة
    

---

### 2️⃣ Use Cases (سيناريوهات)

مثال:

- User adds task
    
- User marks task done
    

---

### 3️⃣ System Sketch

رسم مبدئي:

```
User → API → Service → DB
```

---

### 4️⃣ اختيار Architecture

نختار الأنسب للمشروع.

---

### 5️⃣ توثيق القرار

نكتب:

- لماذا اخترنا هذا التصميم؟
    

---

## 📝 تمرين الأسبوع 3

اكتب Use Cases لتطبيق حجز خدمات (زي مشروعك).

---

## ⚠️ أخطاء شائعة

❌ القفز للكود مباشرة  
❌ تجاهل المتطلبات

---

# 🟣 الأسبوع 4 — Decomposition + Coupling/Cohesion

## تقسيم النظام

نقسمه Modules:

```
Auth Module
Task Module
Notification Module
```

---

## Coupling (الاعتماد بين الأجزاء)

كل ما يقل → أفضل.

---

## Cohesion (ترابط الكلاس)

كل ما يزيد → أفضل.

---

## 📝 تمرين

قسم Note App إلى Modules.

---

# 🟠 الأسبوع 5 — API + Interface + Data Modeling

## API Design Basics

مثال REST:

```
GET /tasks
POST /tasks
PUT /tasks/1
DELETE /tasks/1
```

---

## Data Modeling

مثال Task:

```
Task
- Id
- Title
- IsDone
- CreatedAt
```

---

## 📝 تمرين

صمم Endpoints لتطبيق الحجز.

---

# 🔴 الأسبوع 6 — Database + Security Basics

## Persistence

اختيار DB:

- SQL → بيانات مترابطة
    
- NoSQL → بيانات ضخمة
    

---

## Security Basics

- Authentication (تسجيل دخول)
    
- Authorization (صلاحيات)
    
- Password Hashing
    
- JWT
    

---

## 📝 تمرين

ما البيانات الحساسة في تطبيق الخدمات؟

---

# 🟤 الأسبوع 7 — UML & Diagrams

## أهم الرسومات

### Class Diagram

```
User ────< Task
```

### Sequence Diagram

```
User → API → Service → DB
```

---

## 📝 تمرين

ارسم Class Diagram لتطبيق Notes (ASCII).

---

# ⚫ الأسبوع 8 — مشروع متكامل

تصميم System لحجز خدمات:

```
Customer App
Worker App
Booking Service
Payment Service
Notification Service
```

---

# 📚 Glossary (مصطلحات مهمة)

|المصطلح|المعنى|
|---|---|
|Architecture|الهيكل العام للنظام|
|Module|جزء من النظام|
|API|واجهة التواصل|
|Coupling|الاعتماد بين الأجزاء|
|Cohesion|ترابط الكلاس|
|SOLID|مبادئ تصميم|

---

# 📖 مصادر للتعلم

## كتب

- Clean Architecture — Robert C. Martin
    
- Designing Data-Intensive Applications
    
- Head First Design Patterns
    

## كورسات

- Software Design & Architecture (Coursera)
    
- System Design Basics (YouTube)
    

---

# ❤️ ماذا نفعل الآن؟

لو حاب نبدأ التطبيق العملي الحقيقي 👇  
نبدأ بالأسبوع 1 ونحل التمرين مع بعض خطوة بخطوة.