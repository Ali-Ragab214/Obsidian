# شرح ملفات PHP بالترتيب المنطقي 🐘

خليني أرتبلك الملفات الأول، وبعدين أشرح كل واحدة بالتفصيل.

---

## ترتيب الملفات المنطقي

```
١. connection.php     →  الاتصال بقاعدة البيانات
٢. login.php          →  صفحة تسجيل الدخول (HTML)
٣. register.php       →  صفحة التسجيل (HTML)
٤. server.php         →  المعالج (Register + Login Logic)
٥. profile.php        →  صفحة الملف الشخصي
٦. logout.php         →  تسجيل الخروج
٧. vulnerable.php     →  مثال على SQL Injection (للتعليم)
```

---

---

# ١. connection.php — الاتصال بقاعدة البيانات

```php
<?php
require "navbar.php";

$dbType   = "mysql";
$dbName   = "iti_os_46_2026";
$host     = "localhost";
$userName = "root";
$password = "";

try {
    $connection = new PDO("$dbType:host=$host;dbname=$dbName", $userName, $password);
    session_start();
} catch (PDOException $e) {
    echo $e->getMessage();
}
```

## إيه اللي بيحصل هنا؟

### أولاً: المتغيرات (Credentials)

```
$dbType   = "mysql"          →  نوع قاعدة البيانات
$dbName   = "iti_os_46_2026" →  اسم الـ database
$host     = "localhost"       →  عنوان السيرفر (جهازك)
$userName = "root"            →  اسم المستخدم
$password = ""                →  الباسورد (فاضي في الـ local)
```

### ثانياً: PDO إيه ده؟

```
PDO = PHP Data Objects

ده الطريقة الاحترافية للاتصال بقاعدة البيانات في PHP
بدل الـ mysqli القديمة

مميزاتها:
✅ بتدعم أنواع databases مختلفة (MySQL, PostgreSQL, SQLite)
✅ بتحمي من SQL Injection
✅ كودها أنظف وأحسن
```

### ثالثاً: Connection String

```php
new PDO("mysql:host=localhost;dbname=iti_os_46_2026", "root", "")
         ▲              ▲              ▲                  ▲      ▲
         نوع الـ db    عنوان السيرفر  اسم الـ database  user  password
```

### رابعاً: try/catch

```
try {
    جرب تعمل الاتصال
} catch (PDOException $e) {
    لو فشل، اطبع رسالة الخطأ
}

لو ما فيش catch والاتصال فشل → الموقع كله هيقع!
```

### خامساً: session_start()

```php
session_start();
```

```
ده بيقول لـ PHP:
"ابدأ حفظ بيانات المستخدم في الـ Session"

Session = بيانات مؤقتة بتتحفظ على السيرفر
لحد ما المستخدم يغلق المتصفح أو يعمل logout

لازم تتكتب في أول الكود قبل أي output
```

---

---

# ٢. login.php — صفحة تسجيل الدخول

```html
<?php
require "navbar.php";
if (isset($_GET["message"])) {
    echo "<p class='alert alert-danger'>" . $_GET["message"] . "</p>";
}
?>
<h1 class="text-center text-success">Login Page</h1>
<form action="server.php" method="post">
    <input type="text" name="email">
    <input type="password" name="password">
    <button name="btn-login" type="submit">Login</button>
</form>
```

## إيه اللي بيحصل هنا؟

### الـ Form

```
action="server.php"  →  البيانات هتتبعت فين؟ → server.php
method="post"        →  الطريقة (POST أحسن من GET للبيانات الحساسة)
name="btn-login"     →  ده مهم جداً! server.php بيتعرف على الزرار ده بالاسم ده
```

### رسالة الخطأ

```php
if (isset($_GET["message"])) {
    echo $_GET["message"];
}
```

```
لو الـ URL فيه ?message=something
→ اعرضه للمستخدم

مثال:
login.php?message=check your email or password
                  ▲
              الرسالة دي بتيجي من server.php
              لما الـ login يفشل
```

---

---

# ٣. register.php — صفحة التسجيل

```html
<form action="server.php" method="post">
    <input type="text"     name="name">
    <input type="email"    name="email">
    <input type="password" name="password">
    <button name="btn-register" type="submit">Register</button>
</form>
```

## نفس فكرة login.php بس بزيادة حقل الاسم

```
name="btn-register"  →  server.php بيتعرف إن الطلب ده تسجيل جديد
name="name"          →  اسم المستخدم
name="email"         →  الإيميل
name="password"      →  الباسورد
```

---

---

# ٤. server.php — قلب التطبيق (الأهم)

الملف ده فيه جزئين كبار:

```
الجزء الأول  →  Register Logic
الجزء الثاني →  Login Logic
```

---

## الجزء الأول: Register

```php
if (isset($_POST["btn-register"])) {

    $userName     = $_POST['name'];
    $userEmail    = $_POST['email'];
    $userPassword = $_POST['password'];
```

### أولاً: التحقق من الاسم (Validation)

```php
$namePattern = "/^[a-zA-Z]{3,}$/";
if (!preg_match($namePattern, $userName)) {
    header("location:register.php?message=enter valid name");
}
```

```
"/^[a-zA-Z]{3,}$/"  →  ده Regex (Regular Expression)

^         →  ابدأ من الأول
[a-zA-Z]  →  حروف إنجليزية بس (كبيرة أو صغيرة)
{3,}      →  3 حروف على الأقل
$         →  لحد الآخر

يعني: الاسم لازم حروف إنجليزية وأكتر من 3 حروف
```

### ثانياً: التحقق من الباسورد

```php
$passwordPattern = "/^[0-9]{6,15}$/";
if (!preg_match($passwordPattern, $userPassword)) {
    header("location:register.php?message=invalid password");
}
```

```
[0-9]    →  أرقام بس
{6,15}   →  من 6 لـ 15 رقم
```

### ثالثاً: تشفير الباسورد (مهم جداً!)

```php
$encreptedPassword = password_hash($userPassword, PASSWORD_DEFAULT);
```

```
مش هنحفظ الباسورد كده:  "123456"
هنحفظه كده:             "$2y$10$kCsZoi1kmZMLqV0MdQ0Q7..."

ليه؟
لو قاعدة البيانات اتسرقت →
  المهاجم مش هيعرف الباسورد الحقيقي أبداً!

password_hash()   →  بتشفر الباسورد
PASSWORD_DEFAULT  →  بتستخدم أحسن خوارزمية تشفير (bcrypt)
```

### رابعاً: التحقق من الإيميل

```php
if (!filter_var($userEmail, FILTER_VALIDATE_EMAIL)) {
    header("location:register.php?message=enter valid Email");
}
```

```
filter_var() →  دالة PHP جاهزة بتتحقق من صحة الإيميل
               بتتأكد إن فيه @ ونقطة وشكله صح
```

### خامساً: التحقق إن الإيميل مش موجود قبل كده

```php
$query  = "select email from users where email='$userEmail'";
$data   = $connection->query($query);
$result = $data->fetchAll(PDO::FETCH_ASSOC);

if ($result) {
    header("location:register.php?message=this email already exist");
    exit;
}
```

```
بنسأل قاعدة البيانات:
"في أي يوزر عنده نفس الإيميل ده؟"

لو $result مش فاضية  →  الإيميل موجود قبل كده → redirect
لو $result فاضية     →  الإيميل جديد → كمّل
```

### سادساً: Insert في قاعدة البيانات

```php
$query = "insert into users (name,email,password) values(:name,:email,:password)";
$sqlQuery = $connection->prepare($query);
$sqlQuery->execute([
    ':name'     => $userName,
    ':email'    => $userEmail,
    ':password' => $encreptedPassword,
]);
```

```
لاحظ  :name  :email  :password
دي Prepared Statements (بتحمي من SQL Injection!)

بدل ما تكتب:
"insert into users values('$userName')"  ← خطر!

بتكتب:
"insert into users values(:name)"   ← آمن ✅
وبعدين:
execute([':name' => $userName])
```

#### الفرق بصورة واضحة:

```
❌ طريقة خطرة (SQL Injection ممكن):
$query = "select * from users where email='$email'";

✅ طريقة آمنة (Prepared Statement):
$query = "select * from users where email=:email";
$sql->execute([':email' => $email]);
```

---

## الجزء الثاني: Login

```php
if (isset($_POST["btn-login"])) {

    $userEmail    = $_POST['email'];
    $userPassword = $_POST['password'];

    $query = "select * from users where email=:email";
    $sqlQuery = $connection->prepare($query);
    $sqlQuery->execute([':email' => $userEmail]);
    $user = $sqlQuery->fetch(PDO::FETCH_ASSOC);
```

### إيه الفرق بين fetch و fetchAll؟

```
fetch()    →  بترجع صف واحد بس (Array)
fetchAll() →  بترجع كل الصفوف (Array of Arrays)

في الـ Login: بنبحث بالإيميل → مفروض نلاقي مستخدم واحد بس
              إذاً → fetch()
```

### التحقق من الباسورد

```php
if ($user && password_verify($userPassword, $user["password"])) {
    $_SESSION['login_ID'] = $user['id'];
    header("location:profile.php");
    exit;
} else {
    header("location:login.php?message=check your email or password");
    exit;
}
```

```
password_verify(الباسورد الـ plain, الباسورد المشفر)
→ بترجع true أو false

يعني:
password_verify("123456", "$2y$10$kCsZoi1...") → true ✅

مش محتاج تشفر الباسورد بنفسك وتقارنه
password_verify بتعمل ده تلقائي!
```

### الـ Session

```php
$_SESSION['login_ID'] = $user['id'];
```

```
بنحفظ ID المستخدم في الـ Session
عشان في أي صفحة تانية نعرف مين ده

$_SESSION  →  زي dictionary بيتحفظ على السيرفر
              ومربوط بالمستخدم ده بس
```

---

---

# ٥. profile.php — صفحة الملف الشخصي

```php
require "connection.php";

if (isset($_SESSION['login_ID'])) {
    $userId = $_SESSION['login_ID'];
    $query  = "select * from users where id=:id";
    $sqlQuery = $connection->prepare($query);
    $sqlQuery->execute([':id' => $userId]);
    $user = $sqlQuery->fetch(PDO::FETCH_ASSOC);
    var_dump($user);
} else {
    header("location:login.php?message=you must login first");
}
```

## إيه اللي بيحصل؟

```
١. بتشيك: في session موجودة؟ (يعني المستخدم logged in؟)

٢. لو أيوه:
   → جيب الـ ID من الـ Session
   → ابحث في قاعدة البيانات عن المستخدم ده
   → اعرض بياناته

٣. لو لأ:
   → redirect لصفحة الـ login مع رسالة
```

### ده اللي بيتسمى Protected Page

```
أي صفحة محتاجة login →
   أول حاجة اتعملها:
   if (!isset($_SESSION['login_ID'])) {
       header("location:login.php");
       exit;
   }
```

---

---

# ٦. logout.php — تسجيل الخروج

```php
require "connection.php";
session_destroy();
header("location:login.php");
```

## بساطة تامة!

```
session_destroy()  →  بتمسح كل بيانات الـ Session
                       يعني المستخدم مش logged in تاني

header("location:login.php")  →  redirect لصفحة الـ login
```

---

---

# ٧. vulnerable.php — مثال SQL Injection (تعليمي!)

```php
$email    = $_POST['email'];
$password = $_POST['password'];

$query = "select * from user 
          WHERE email = '$email' 
          AND password = '$password'";

$result = $connection->query($query);
$user   = $result->fetch(PDO::FETCH_ASSOC);

if ($user) {
    echo "Login Success (Site Hacked)";
}
```

## المشكلة هنا إيه؟

المستخدم ممكن يكتب في الـ email:

```
' OR '1'='1
```

فالـ Query هيبقى:

```sql
SELECT * FROM user
WHERE email = '' OR '1'='1'
AND password = ''
```

```
'1'='1'  →  دايماً true!
يعني الـ query هيرجع كل الـ users
وهيعمل login بدون باسورد! 😱
```

## الحل هو Prepared Statements اللي شفناها في server.php

```
❌ $query = "select * from users where email='$email'";

✅ $query = "select * from users where email=:email";
   $sql->execute([':email' => $email]);
```

---

---

# ملخص الـ Flow كامل

```
المستخدم يفتح register.php
         ↓
يملا الـ form ويضغط Register
         ↓
البيانات تتبعت لـ server.php (POST)
         ↓
server.php:
  ✅ Validate الاسم
  ✅ Validate الباسورد
  ✅ Validate الإيميل
  ✅ تشفير الباسورد
  ✅ تأكد إن الإيميل مش موجود
  ✅ Insert في قاعدة البيانات
         ↓
المستخدم يروح login.php
         ↓
server.php:
  ✅ ابحث عن الإيميل
  ✅ password_verify
  ✅ حفظ الـ ID في Session
         ↓
Redirect لـ profile.php
         ↓
profile.php تشيك الـ Session
  ✅ اعرض بيانات المستخدم
         ↓
logout.php → session_destroy()
```

---

عايز أشرح أي جزء بتفاصيل أكتر؟ 🎯