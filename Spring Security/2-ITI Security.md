حلو جدًا 👌 انت كده دخلت على أهم مفاهيم Spring Security فعلاً. خلينا نشرحهم بطريقة مرتبة وبسيطة وبأمثلة.

---

# 1️⃣ Principal (مين المستخدم أصلاً؟)

كلمة **Principal** معناها:

> الهوية الحالية اللي بتتعامل مع السيستم

يعني ببساطة:  
الشخص اللي عامل login دلوقتي.

في 90% من التطبيقات → الـ Principal = **User**

### في Spring Security

بعد ما اليوزر يعمل login، السيستم بيحط الـ Principal داخل:

```
SecurityContext
```

وتقدر تجيبه في أي مكان:

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
Object principal = auth.getPrincipal();
```

غالبًا هيكون:

```java
UserDetails
```

يعني الـ Principal هو:

- user authenticated ✔️
    
- معروف هويته ✔️
    
- مسموح نقرر يعمل ايه ✔️
    

👉 أهم نقطة:  
**Authentication = مين انت**  
**Authorization = مسموحلك تعمل ايه**

---

# 2️⃣ Role vs Privilege

دي من أكتر الحاجات اللي بتتلخبط.

## Privilege (Permission)

دي أصغر وحدة صلاحيات.

مثال:

- CREATE_ORDER
    
- DELETE_USER
    
- VIEW_REPORTS
    
- EDIT_PROFILE
    

دي بنسميها **Authorities / Permissions**

---

## Role

الـ Role هو **مجموعة من الصلاحيات**.

يعني Role = container للـ Privileges.

### مثال واقعي

|Role|Privileges|
|---|---|
|ADMIN|CREATE_USER, DELETE_USER, VIEW_REPORTS|
|CUSTOMER|CREATE_ORDER, VIEW_ORDER|
|WORKER|ACCEPT_BOOKING, REJECT_BOOKING|

يعني لما تقول:

```
hasRole("ADMIN")
```

انت ضمنيًا بتقول:

```
عنده كل الصلاحيات المرتبطة بالـ ADMIN
```

في Spring Security:

```
ROLE_ADMIN
ROLE_USER
```

دا convention معروف.

---

# 3️⃣ RBAC (Role Based Access Control)

ده النظام التقليدي اللي معظم التطبيقات بتستخدمه.

الفكرة:

> الصلاحيات مبنية على الـ Roles فقط.

### شكل العلاقة

```
User → Role → Privileges
```

### مثال

لو عندك endpoint:

```
/admin/deleteUser
```

تحميه كده:

```java
@PreAuthorize("hasRole('ADMIN')")
```

لو المستخدم مش Admin → ❌ Access Denied

### مميزاته

✔️ بسيط  
✔️ سهل التنفيذ  
✔️ مناسب لـ 80% من التطبيقات

---

# 4️⃣ ABAC (Attribute Based Access Control)

ده نظام أذكى وأقوى بكتير.

بدل ما تعتمد على Role بس،  
تعتمد على **Attributes (خواص)**.

يعني القرار بيتاخد بناء على:

- صفات المستخدم
    
- صفات الريكوست
    
- صفات الريسورس
    
- الوقت / المكان / IP / location
    

### مثال قوي 🔥

تخيل endpoint:

```
GET /orders/{id}
```

في RBAC:

```
ROLE_CUSTOMER → يشوف الاوردرات
```

لكن المشكلة 🤔  
هل ينفع يشوف **كل الاوردرات؟** طبعًا لا.

في ABAC:  
نقول:

```
يسمح له يشوف الاوردر لو:
order.userId == currentUser.id
```

دي مش Role…  
دي Condition.

في Spring Security:

```java
@PreAuthorize("#userId == authentication.principal.id")
```

### مثال تاني

موظف HR يقدر يشوف بيانات الموظفين  
لكن فقط في ساعات العمل!

```
time between 9 and 5
```

ده ABAC.

### الخلاصة

|RBAC|ABAC|
|---|---|
|based on role|based on conditions|
|static|dynamic|
|simple|powerful|

👉 المشاريع الكبيرة بتستخدم الاتنين مع بعض.

---

# 5️⃣ Stateful vs Stateless Authentication

دي مهمة جدًا في Spring Security و JWT 🔥

---

## Stateful Authentication (Session)

ده النظام القديم (Classic login).

### الفلو

1. اليوزر يعمل login
    
2. السيرفر ينشئ **Session**
    
3. يخزنها في السيرفر
    
4. يدي للمتصفح Cookie فيها SessionId
    
5. كل Request → السيرفر يروح يجيب Session من الذاكرة
    

```
Browser → Cookie(SessionId) → Server → Session Store
```

### المشكلة

- السيرفر بيخزن sessions 😵‍💫
    
- لو السيرفر زاد → لازم Session shared
    
- scaling صعب
    

ده اللي Spring Security بيعمله افتراضيًا.

---

## Stateless Authentication (JWT)

ده النظام الحديث (REST APIs).

السيرفر **لا يخزن أي جلسات**.

### الفلو

1. اليوزر يعمل login
    
2. السيرفر يولد JWT Token
    
3. يبعته للمستخدم
    
4. كل Request يبعته في Header:
    

```
Authorization: Bearer token
```

5. السيرفر يتحقق من التوكن فقط ✔️
    

لا يوجد Sessions نهائيًا.

```
Client → JWT → Server verifies signature
```

### مميزاته

✔️ scalable جدًا  
✔️ مناسب للـ mobile و Angular  
✔️ Microservices friendly  
✔️ لا يحتاج Session store

👉 لذلك 99% من APIs الحديثة = Stateless + JWT

---

# 🎯 الخلاصة السريعة

- Principal → المستخدم الحالي اللي عامل login
    
- Privilege → صلاحية واحدة
    
- Role → مجموعة صلاحيات
    
- RBAC → Authorization بالRoles
    
- ABAC → Authorization بالConditions
    
- Stateful → Session في السيرفر
    
- Stateless → JWT بدون Sessions
    

---

لو حابب نبدأ نطبق الكلام ده عملي في Spring Security + JWT خطوة خطوة قولّي ونبدأ Implementation 🔥



---
`
Spring security Archeticture =>  
![[Pasted image 20260406211409.png]]


 اي فلتر من دول يقدر يوقف الريكوست ويقوله مش هتكمل 
 -Delegating filter Proxy =>
 
 