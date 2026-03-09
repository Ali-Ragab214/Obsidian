

1. مفهوم Authentication
    
2. الفرق بين Authentication و Authorization
    
3. المكونات الأساسية
    
4. Flow التسجيل (Register)
    
5. Flow تسجيل الدخول (Login)
    
6. Flow استخدام الـ Token
    
7. أنواع Authentication
    
8. Best Practices
    

---

# أولاً: ما هو Authentication ؟

**Authentication** هو عملية **التحقق من هوية المستخدم**.

يعني النظام يتأكد أن الشخص الذي يحاول الدخول هو فعلاً صاحب الحساب.

مثال بسيط:

|المستخدم|ما يقدمه|النظام يتحقق|
|---|---|---|
|Ali|Email + Password|هل البيانات صحيحة؟|

لو صحيحة → يسمح له بالدخول.

---

## مثال من الحياة

عندما تدخل البنك:

1️⃣ تعطي البطاقة  
2️⃣ تدخل الرقم السري

النظام يتأكد أنك صاحب البطاقة.

هذا هو **Authentication**.

---

# الفرق بين Authentication و Authorization

|Authentication|Authorization|
|---|---|
|من أنت؟|ماذا يسمح لك أن تفعل؟|
|Login|Permissions|
|التحقق من الهوية|التحقق من الصلاحيات|

مثال:

|المستخدم|Authentication|Authorization|
|---|---|---|
|Ali|Logged in|Admin|
|Ahmed|Logged in|User|

---

# مكونات نظام Authentication

أي نظام Authentication يحتوي غالباً على:

```
Client (Frontend)
        |
        v
API (Backend)
        |
        v
Database
```

العناصر الأساسية:

### 1️⃣ User

المستخدم الذي يريد تسجيل الدخول.

---

### 2️⃣ Credentials

بيانات تسجيل الدخول مثل:

- Email
    
- Password
    
- Phone number
    

---

### 3️⃣ Hashing

تشفير كلمة السر قبل تخزينها.

مثال:

```
password: 123456
hash: $2a$10$ABCDXYZ...
```

لا يتم تخزين كلمة السر الأصلية.

---

### 4️⃣ Token

بعد تسجيل الدخول يحصل المستخدم على:

```
JWT Token
```

ويستخدمه للوصول للـ API.

---

# Flow كامل لنظام Authentication

## الصورة العامة

![Image](https://assets-developers.ringcentral.com/dpw/guide/images/oauth-password-flow.png?v=2025-11-03)

![Image](https://media2.dev.to/dynamic/image/width%3D1280%2Cheight%3D720%2Cfit%3Dcover%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F8wiw2dbjerzq6br66qv8.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AARZjUHyHZaakU0xuXwrrtg.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ACQJbBjLgF_XBt39HJt04_Q.jpeg)

---

# Flow رقم 1: عملية التسجيل (Register)

## الخطوات

### 1️⃣ المستخدم يرسل البيانات

```
POST /register
```

Body:

```json
{
 "name": "Ali",
 "email": "ali@gmail.com",
 "password": "123456"
}
```

---

### 2️⃣ Backend يستقبل الطلب

الـ Controller يستقبل الطلب.

```
AuthController
```

---

### 3️⃣ Validation

يتأكد من:

- email صحيح
    
- password قوية
    
- المستخدم غير موجود
    

مثال:

```
Email already exists?
```

---

### 4️⃣ Password Hashing

يتم تشفير كلمة السر.

مثال:

```
123456
```

تصبح:

```
$2a$10$GHSDHSDHGSDH...
```

باستخدام:

- BCrypt
    
- Argon2
    
- PBKDF2
    

---

### 5️⃣ حفظ المستخدم في Database

```
Users Table
```

|id|email|password_hash|
|---|---|---|
|1|[ali@gmail.com](mailto:ali@gmail.com)|hashed_password|

---

### 6️⃣ إنشاء Token (اختياري)

بعض الأنظمة تعطي Token مباشرة بعد التسجيل.

---

### Diagram لعملية التسجيل

```
User
 |
 |  Register Request
 v
Controller
 |
 | Validation
 v
Service
 |
 | Hash Password
 v
Database
 |
 | Save User
 v
Response
```

---

# Flow رقم 2: تسجيل الدخول (Login)

## الخطوات

### 1️⃣ المستخدم يرسل Login

```
POST /login
```

Body:

```json
{
 "email": "ali@gmail.com",
 "password": "123456"
}
```

---

### 2️⃣ Backend يبحث عن المستخدم

```
SELECT * FROM Users WHERE email='ali@gmail.com'
```

---

### 3️⃣ التحقق من Password

النظام يقارن:

```
entered password
```

مع

```
stored hashed password
```

باستخدام:

```
bcrypt.compare()
```

---

### 4️⃣ إنشاء JWT Token

إذا كانت صحيحة يتم إنشاء:

```
JWT Token
```

---

### شكل الـ JWT

يتكون من 3 أجزاء:

```
HEADER.PAYLOAD.SIGNATURE
```

مثال:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VySWQiOjF9
.
abcXYZsignature
```

---

### 5️⃣ إرسال Token للمستخدم

Response:

```json
{
 "token": "jwt_token_here"
}
```

---

# Diagram لعملية Login

```
User
 |
 | Login Request
 v
Controller
 |
 v
Service
 |
 | Check Email
 v
Database
 |
 | Verify Password
 v
JWT Generator
 |
 v
Response (Token)
```

---

# Flow رقم 3: استخدام الـ Token

بعد تسجيل الدخول يستخدم المستخدم الـ Token في كل Request.

---

## مثال

```
GET /orders
```

Header:

```
Authorization: Bearer JWT_TOKEN
```

---

## Backend يتحقق من Token

الـ Middleware أو Filter يقوم بـ:

1️⃣ قراءة الـ Token  
2️⃣ التحقق من الـ Signature  
3️⃣ استخراج userId

---

### Diagram

```
Client
 |
 | Request + Token
 v
Middleware
 |
 | Validate Token
 v
Controller
 |
 v
Response
```

---

# أنواع Authentication

## 1️⃣ Session Based Authentication

قديم.

```
Client
   |
   v
Session ID
   |
   v
Server Memory
```

---

## 2️⃣ Token Based Authentication

الأكثر استخداماً.

مثل:

```
JWT
```

---

## 3️⃣ OAuth

مثل تسجيل الدخول عبر:

- Google
    
- Facebook
    

---

# مكونات Authentication في Backend Architecture

عادةً تكون:

```
AuthController
     |
     v
AuthService
     |
     v
UserRepository
     |
     v
Database
```

---

# مثال في Clean Architecture

بما أنك تستخدم **Clean Architecture** يا علي.

الطبقات تكون:

```
API
 └── AuthController

Application
 └── LoginCommand
 └── RegisterCommand

Domain
 └── User Entity

Infrastructure
 └── UserRepository
 └── JWT Service
```

---

# Best Practices

### 1️⃣ لا تخزن Password كـ Plain Text

❌ خطأ

```
password: 123456
```

✔ الصحيح

```
hashed password
```

---

### 2️⃣ استخدم HTTPS

لمنع سرقة البيانات.

---

### 3️⃣ استخدم Expiration للـ Token

مثال:

```
token expires in 1 hour
```

---

### 4️⃣ استخدم Refresh Token

حتى لا يطلب Login كل مرة.

---

# الخلاصة

Authentication Flow بالكامل:

```
User Register
     |
     v
Password Hash
     |
     v
Save User
     |
     v
Login
     |
     v
Verify Password
     |
     v
Generate JWT
     |
     v
Client uses Token
     |
     v
Backend verifies Token
```

---

