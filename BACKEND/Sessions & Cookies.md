
    

---

# Sessions & Cookies — 

---

# 1️⃣ المشكلة الأساسية في HTTP

## HTTP is Stateless

بروتوكول **HTTP** بطبيعته **Stateless**.

يعني:

> كل Request يُعامل كأنه أول مرة يحدث الاتصال.

السيرفر **لا يتذكر أي شيء** عن الطلبات السابقة.

### مثال

```
Request 1 → Login
Request 2 → Open Home Page
```

السيرفر **لا يعرف أن المستخدم سجل الدخول**.

لذلك نحتاج إلى آلية لحفظ حالة المستخدم.

وهنا يظهر دور:

```
Cookies
Sessions
```

---

# 2️⃣ كيف يتم حفظ حالة المستخدم

الفكرة الأساسية:

```
User logs in
        │
        ▼
Server creates Session
        │
        ▼
Server sends SessionID inside Cookie
        │
        ▼
Browser sends Cookie with every request
```

---

## Diagram — Authentication Flow

```
Browser                    Server
   │                         │
   │   POST /login           │
   ├────────────────────────►│
   │                         │
   │      Validate user      │
   │                         │
   │  Create Session         │
   │                         │
   │ Set-Cookie: session_id  │
   │◄────────────────────────┤
   │                         │
   │   GET /home             │
   │ Cookie: session_id      │
   ├────────────────────────►│
   │                         │
   │   Find Session          │
   │                         │
   │     Allow Access        │
   │◄────────────────────────┤
```

---

# 3️⃣ Cookies

## تعريف

الـ **Cookie** هي:

> قطعة بيانات صغيرة يتم تخزينها داخل المتصفح (Client Side).

يتم إرسالها تلقائيًا مع كل request إلى السيرفر.

### Definition (English)

A **Cookie** is a small piece of data stored in the **client browser** and automatically sent with every HTTP request to the server.

---

## أين يتم تخزينها

```
Browser Storage
   │
   ├── Cookies
   ├── LocalStorage
   └── SessionStorage
```

---

## خصائص Cookies

|الخاصية|الشرح|
|---|---|
|Storage Location|Client (Browser)|
|Size Limit|تقريبًا 4KB|
|Visibility|المستخدم يمكنه رؤيتها|
|Expiration|لها تاريخ انتهاء|
|Automatic Sending|تُرسل مع كل Request|

---

## مثال PHP — إنشاء Cookie

```php
setcookie(
    "username",
    "Ali",
    time() + (86400 * 30) // 30 days
);
```

### المعاملات

```
name
value
expiration time
path
domain
secure
httponly
```

---

## قراءة Cookie

```php
if (isset($_COOKIE['username'])) {
    echo "Welcome " . $_COOKIE['username'];
}
```

---

## حذف Cookie

```php
setcookie("username", "", time() - 3600);
```

الفكرة:

```
set expiry in the past
```

---

# 4️⃣ Sessions

## تعريف

الـ **Session** هي:

> بيانات يتم تخزينها على السيرفر بينما يحتفظ المتصفح فقط بـ Session ID.

### Definition (English)

A **Session** stores data on the **server**, while the browser only stores a small identifier called **Session ID**.


1️⃣ السيرفر ينشئ **Session**  
2️⃣ يعطي المتصفح **Session ID** في Cookie  
3️⃣ المتصفح يرسل هذا الـ ID مع كل Request   لان السيرفر عنده الداتا بتاع السيشن فيعرف اليوزر من ال id 
4️⃣ السيرفر يستخدمه ليعرف المستخدم

---
login =>create session =>session stored in server memory =>session id stored in browser
## كيف تعمل

```
User logs in
     │
     ▼
Server creates Session
     │
     ▼
Session stored in server memory
     │
     ▼
Session ID sent to browser
```

---

## Diagram — Session Storage

```
Browser
   │
   │  Cookie: PHPSESSID=abc123
   ▼
Server
   │
   ├── Session Store
   │      │
   │      ├── abc123
   │      │      user_id = 5
   │      │      role = admin
   │
   │
```

---

## بدء Session

يجب تشغيل Session في بداية كل صفحة.

```php
session_start();
```

---

## تخزين بيانات

```php
$_SESSION['user_id'] = 1;
$_SESSION['user_name'] = "Ali";
$_SESSION['role'] = "admin";
```

---

## قراءة البيانات

```php
echo $_SESSION['user_name'];
```

---

## حذف متغير

```php
unset($_SESSION['user_name']);
```

---

## حذف كل Session

```php
session_destroy();
```

يستخدم غالبًا في **Logout**.

---

# 5️⃣ Login Flow — System Design

## Diagram

```
User
 │
 │ login
 ▼
Server
 │
 │ verify password
 ▼
Create Session
 │
 │ session_id
 ▼
Browser Cookie
 │
 │ future requests
 ▼
Server validates session
```

---

# 6️⃣ مثال Login System احترافي

### login.php

```php
session_start();

if($_SERVER['REQUEST_METHOD'] === 'POST'){

    $email = $_POST['email'];
    $password = $_POST['password'];

    $user = getUserFromDB($email);

    if($user && password_verify($password,$user['password'])){

        session_regenerate_id(true);

        $_SESSION['user_id'] = $user['id'];
        $_SESSION['role'] = $user['role'];

        header("Location: home.php");
        exit;
    }
}
```

---

# 7️⃣ حماية الصفحات

## Middleware Pattern

```
Request
   │
   ▼
Auth Middleware
   │
   ├── Session exists → continue
   │
   └── No session → redirect
```

---

### Example

```php
session_start();

if(!isset($_SESSION['user_id'])){
    header("Location: login.php");
    exit;
}
```

---

# 8️⃣ الفرق بين Sessions و Cookies

|Feature|Session|Cookie|
|---|---|---|
|Storage|Server|Browser|
|Security|High|Medium|
|Size|Large|4KB|
|Visibility|Hidden|Visible|
|Lifetime|Session based|Configurable|

---

# 9️⃣ متى تستخدم كل واحد

## استخدم Sessions

```
Authentication
User roles
Shopping carts
Sensitive data
Flash messages
```

---

## استخدم Cookies

```
Remember Me
Language preference
Theme settings
Tracking
Analytics
```

---

# 🔟 Best Security Practices

## 1️⃣ HttpOnly Cookie

```php
setcookie(
"user",
"Ali",
time()+86400,
"/",
"",
false,
true
);
```

هذا يمنع:

```
JavaScript access
```

ويحمي من:

```
XSS attacks
```

---

## 2️⃣ Session Regeneration

```php
session_regenerate_id(true);
```

يحمي من:

```
Session Fixation Attack
```

---

## 3️⃣ Secure Cookies

```php
setcookie(
"name",
"value",
time()+86400,
"/",
"",
true,
true
);
```

```
secure = true
```

يعني:

```
HTTPS only
```

---

# 1️⃣1️⃣ أخطاء شائعة

❌ تخزين كلمة المرور في Cookie

```php
setcookie("password","1234");
```

❌ تخزين بيانات حساسة في Cookie

---

✅ الصحيح

```
Sensitive Data → Session
Preferences → Cookies
```

---

# 1️⃣2️⃣ Architecture Insight (مهم للـ Backend)

في الأنظمة الكبيرة يتم تخزين Sessions في:

```
Redis
Memcached
Database
```

بدل:

```
Server memory
```

السبب:

```
Scalability
Load balancing
Distributed systems
```

---

# خلاصة الفكرة

```
HTTP → Stateless

Cookies → client storage

Sessions → server storage

Session ID → bridge between client and server
```

---

لو أردت يا علي، أستطيع أيضًا أن أجهز لك **نسخة أقوى بكثير لمهندسي Backend** تشمل:

- **Session Fixation**
    
- **Session Hijacking**
    
- **JWT vs Session**
    
- **كيف تعمل Authentication في الأنظمة الكبيرة**
    

وهي أشياء مهمة جدًا لأي **Backend Engineer**.