
---

# أولًا: فكرة المحاضرة الأساسية

الموضوع كله عن **إزاي نجمع Functional Requirements صح**  
يعني: إزاي نفهم النظام المطلوب قبل ما نبدأ نبرمج.

---

# الجزء الأول: المشكلة في الطريقة التقليدية (Formal Method القديمة)

المحاضرة كاتبة:

> we ask the client to describe everything they need  
> for complex systems not a good approach

يعني إيه؟

زمان كان تحليل النظام يتم كده:

- نروح للعميل
    
- نقوله: "قولنا كل اللي انت عايزه"
    
- نكتب الكلام ونمشي نبرمج
    

### المشكلة؟

دي طريقة سيئة جدًا للأنظمة الكبيرة ❌

ليه؟  
لأن العميل:

- مش فاهم تقنيًا
    
- مش عارف كل السيناريوهات
    
- مش عارف الاستثناءات
    
- بيفتكر حاجات وينسى حاجات
    

📌 النتيجة:  
نظام مليان مشاكل بعد التسليم.

عشان كده ظهرت طريقة أقوى 👇

---

# الجزء الثاني: الطريقة الأقوى لجمع المتطلبات

المحاضرة بتقول:

### الطريقة الأقوى تعتمد على حاجتين:

1. Use Cases
    
2. User Flows
    

خلينا نفهمهم صح.
;l
---

# Use Case يعني إيه؟

المحاضرة بتقول:

> Situation / Scenario in which our system is used

يعني:  
سيناريو حقيقي بيستخدم فيه المستخدم النظام.

📌 يعني نحول "الطلبات" إلى "مواقف حقيقية".

---

### مثال بسيط (تطبيق خدمات زي مشروعك)

بدل ما العميل يقول:

> عايزين Booking system

نحولها لـ Use Cases:

- Customer books service
    
- Worker accepts booking
    
- Customer cancels booking
    
- Customer pays
    

شايف الفرق؟  
بقينا بنفكر في **مواقف حقيقية** مش Features عامة.

---

# User Flow يعني إيه؟

المحاضرة بتقول:

> Step-by-Step graphical representation of each use case

يعني:  
نشرح كل Use Case **خطوة بخطوة**.

📌 Use Case = السيناريو  
📌 User Flow = خطوات السيناريو

---

# خطوات عملية جمع المتطلبات (أهم جزء)

المحاضرة قالت 3 خطوات أساسية 👇

---

## الخطوة 1️⃣: تحديد كل الـ Actors (المستخدمين)

> first we need to identify all actors/users in the system

### Actor = أي شخص بيتعامل مع النظام

مثال تطبيق خدمات:

- Customer 👤
    
- Worker 👷‍♂️
    
- Admin 👨‍💼
    
- Payment Gateway 💳
    

⚠️ مش شرط يكون إنسان فقط  
أي System خارجي يعتبر Actor.

---

## الخطوة 2️⃣: تحديد كل Use Cases

> capture and describe all possible use cases

بعد ما عرفنا المستخدمين → نسأل:

كل Actor ممكن يعمل إيه في النظام؟

### مثال:

### Customer Use Cases

- Register
    
- Login
    
- Choose service
    
- Send booking request
    
- Cancel request
    
- Pay
    

### Worker Use Cases

- Register
    
- Accept booking
    
- Reject booking
    
- Complete job
    

### Admin Use Cases

- Approve workers
    
- Manage categories
    
- Manage users
    

📌 كده بقينا فاهمين "كل اللي ممكن يحصل في النظام".

---

## الخطوة 3️⃣: عمل User Flow لكل Use Case

> expand each use case through flow of events

دي أهم خطوة 🔥

نبدأ ناخد كل Use Case ونفصّله خطوة بخطوة.

---

### مثال مهم: Book Service

نكتب السيناريو كـ Flow:

1. Customer opens app
    
2. Select category
    
3. Select service
    
4. System fetches nearby workers
    
5. Customer sends booking request
    
6. Worker receives notification
    
7. Worker accepts / rejects
    

📌 كل خطوة اسمها **Event**.

---

## كل Event يحتوي على حاجتين

المحاضرة قالت:

> each event contain : action and data

يعني كل خطوة فيها:

### 1) Action (فعل)

ايه اللي حصل؟

### 2) Data (بيانات)

ايه الداتا اللي اتحركت؟

---

### مثال Event:

"Customer sends booking request"

- Action: Send request
    
- Data:
    
    - customerId
        
    - serviceId
        
    - location
        
    - time
        

وهنا بدأنا ندخل في تصميم الـ Backend 🔥

---

# الجزء الأخير: استخدام UML و Sequence Diagram

المحاضرة قالت:

> UML used mostly for software design  
> used to represent interactions between entities

يعني بعد ما نكتب الـ User Flow  
نحوّله إلى **Sequence Diagram**

---

## يعني إيه Sequence Diagram؟

هو رسم يوضح:  
مين كلم مين؟ وبأي ترتيب؟

### مثال Ride Completion (زي Uber)

Actors:

- Customer
    
- App
    
- Driver
    
- Payment Service
    

Flow:

Customer → App → Driver → Payment → App → Customer

الرسم بيبين:

- ترتيب الرسائل
    
- التفاعل بين الأنظمة
    
- من يبدأ الحدث
    
- ردود الفعل
    

وده مهم جدًا للـ Backend Design.

---

# الخلاصة (Summary المحاضرة)

المحاضرة بتلخص العملية كلها في 3 خطوات:

### 1️⃣ Identify Actors

مين هيستخدم النظام؟

### 2️⃣ Gather Use Cases

النظام هيُستخدم في إيه؟

### 3️⃣ Expand Use Cases → User Flows

السيناريو ده بيحصل إزاي خطوة بخطوة؟

---

# ليه الكلام ده مهم جدًا ليك كـ Backend Developer؟

لأن ده بيحوّل:

- الأفكار → Endpoints
    
- الـ Flows → Database design
    
- الـ Events → APIs
    

📌 مثال:  
Use Case: Book Service  
→ يتحول إلى:

- POST /bookings
    
- GET /workers/nearby
    
- PATCH /booking/accept
    

شايف ازاي System Analysis بيتحول مباشرة لـ Backend؟ 😎

---

لو حابب في الخطوة الجاية نطبق الكلام ده عملي على مشروع الخدمات بتاعك ونعمل Use Cases و User Flows حقيقية — دي أهم خطوة في الـ System Design.