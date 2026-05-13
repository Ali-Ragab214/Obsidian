
---

# Software Architecture & System Design — Lecture 1 Notes

---

# 1️⃣ Why Structure Matters

**Everything we build has a structure.**

أي System كبير لازم يكون له:

- Components (أجزاء)
    
- Communication between components
    
- Constraints & Requirements
    

بدون Structure → النظام يبقى عشوائي وغير قابل للتوسع أو الصيانة.

---

# 2️⃣ What is Software Architecture?

**Definition (Interview Friendly)**  
Software Architecture is:

> A high-level description of the system structure, its components, and how they communicate to satisfy requirements and constraints.

### الكلمات المهمة في التعريف 👇

- **High-level** → مش تفاصيل كود
    
- **Structure** → تقسيم النظام
    
- **Components** → Services / Modules / Layers
    
- **Communication** → APIs / Events / Messages
    
- **Requirements & Constraints** → سبب التصميم
    

💡 يعني بنصمم _شكل النظام قبل كتابة الكود_.

---

# 3️⃣ Software Development Lifecycle (High Level)

مراحل بناء أي System:

```
Design → Implementation → Testing → Deployment
```

### 1) Design

- فهم المتطلبات
    
- اختيار Architecture
    
- اختيار التكنولوجيا
    

### 2) Implementation

- كتابة الكود
    
- تقسيم modules
    
- تطبيق design patterns
    

### 3) Testing

- Unit testing
    
- Integration testing
    
- Performance testing
    

### 4) Deployment

- نشر النظام على السيرفر
    
- Monitoring
    
- Scaling
    

---

# 4️⃣ System Requirements (أهم جزء في البداية)

## What are System Requirements?

> Formal description of what we need to build.

لكن في **Large Scale Systems** المتطلبات بتكون مختلفة عن متطلبات كود عادي.

### الفرق بين Small vs Large Systems

|Small Task|Large Scale System|
|---|---|
|Method / Algorithm|Full distributed system|
|Clear requirements|Ambiguous requirements|
|Small scope|Huge scope|
|Low risk|High risk|

### خصائص متطلبات الأنظمة الكبيرة

- Big scope
    
- High level of abstraction
    
- High ambiguity (غموض)
    

وده سبب إن **Requirement Gathering** مهم جدًا.

---

# 5️⃣ Importance of Requirement Gathering ⚠️

ليه لازم نجمع المتطلبات كويس؟

لأن أي خطأ هنا = كارثة بعدين 😅

أهم المخاطر:

- بناء system غلط
    
- تكلفة تعديل عالية جدًا
    
- مشاكل scalability
    
- مشاكل performance
    
- تأخير المشروع
    

💡 Rule:

> Fixing a bug in requirements is 100x cheaper than fixing it after deployment.

---

# 6️⃣ Types of Requirements (مهم جدًا للانترفيو 🔥)

أي System Requirements بتتقسم لـ 3 أنواع:

```
1) Functional Requirements
2) Non-Functional Requirements
3) System Constraints
```

---

## 6.1 Functional Requirements (FR)

> What the system DOES.

يعني الـ Features.

### أمثلة:

- User can register
    
- User can login
    
- User can book a service
    
- User can pay online
    
- Admin can approve jobs
    

💡 في الانترفيو:

> Functional = Features.

---

## 6.2 Non-Functional Requirements (NFR)

اسمها كمان **Quality Attributes**

> How WELL the system performs.

دي أهم جزء في System Design interviews 🔥

### أشهر Non-Functional Requirements:

|Attribute|المعنى|
|---|---|
|Scalability|يتحمل ملايين المستخدمين|
|Availability|يشتغل 24/7|
|Performance|سرعة الاستجابة|
|Security|حماية البيانات|
|Reliability|يقلل الفشل|
|Maintainability|سهل التعديل|
|Usability|سهل الاستخدام|

💡 شركات FAANG بتركز جدًا على الجزء ده.

---

## 6.3 System Constraints

> Limitations or boundaries on the system.

يعني قيود لازم نمشي عليها.

### أمثلة:

- لازم نستخدم AWS
    
- لازم نستخدم MySQL
    
- Budget محدود
    
- Deadline 3 شهور
    
- لازم يشتغل على Mobile فقط
    

الـ Constraints بتأثر مباشرة على الـ Architecture.

---

# 7️⃣ العلاقة بين الأنواع الثلاثة

مثال سريع:

### Build Ride Sharing App 🚗

**Functional Requirements**

- User requests ride
    
- Driver accepts ride
    

**Non-Functional Requirements**

- Handle 1M users
    
- Response < 2 sec
    
- 99.99% availability
    

**Constraints**

- Must use Google Maps API
    
- Must launch in 3 months
    

---

# 🎯 Interview Summary

لو سألوك:  
**What are the types of system requirements?**

قول:

> Functional requirements describe what the system does.  
> Non-functional requirements describe how well the system performs.  
> Constraints define the limitations and boundaries of the system.

---

لو حابب المحاضرة الجاية نكمل **Requirement Gathering + Tradeoffs** لأنهم مهمين جدًا في الانترفيوهات.