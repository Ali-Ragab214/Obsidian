
تمام يا علي 🔥  
هشرحلك أهم **PHP Superglobals** اللي طلبتهم:

- `$_GET`
    
- `$_POST`
    
- `$_SERVER`
    
- `header()`
    

وهقولك كل واحدة بتعمل ايه + تستخدمها ازاي + إزاي تتنقل بين الصفحات + مثال شامل كامل 💪

---

# أولًا: يعني ايه Superglobals في PHP؟

دي متغيرات جاهزة في PHP  
بتبقى متاحة في أي مكان في السكريبت  
ومش محتاج تعملها global

---

# 1️⃣ $_GET

## 📌 بتعمل ايه؟

بتستقبل البيانات اللي بتيجي من الـ URL

يعني البيانات اللي بتتبعت بالطريقة دي:

```
page.php?name=ali&age=25
```

---

## 📌 مثال عملي

### page1.php

```php
<a href="page2.php?name=Ali&age=25">Go To Page 2</a>
```

---

### page2.php

```php
<?php
echo "Name: " . $_GET['name'];
echo "<br>";
echo "Age: " . $_GET['age'];
?>
```

---

## 📌 شكل الرابط هيبقى كده:

```
page2.php?name=Ali&age=25
```

---

## 📌 إمتى أستخدم GET؟

✔️ لما البيانات مش حساسة  
✔️ لما تحب البيانات تظهر في الـ URL  
✔️ في البحث – الفلترة – pagination

---

# 2️⃣ $_POST

## 📌 بتعمل ايه؟

بتستقبل البيانات من الفورم  
والبيانات **مش بتظهر في الـ URL**

---

## 📌 مثال

### form.php

```php
<form method="POST" action="result.php">
    <input type="text" name="username">
    <input type="password" name="password">
    <button type="submit">Send</button>
</form>
```

---

### result.php

```php
<?php
echo "Username: " . $_POST['username'];
echo "<br>";
echo "Password: " . $_POST['password'];
?>
```

---

## 📌 الفرق بين GET و POST

|GET|POST|
|---|---|
|البيانات بتظهر في URL|مخفية|
|حجم محدود|حجم أكبر|
|أقل أمان|أفضل للأمان|
|تستخدم للقراءة|تستخدم للإرسال|

---

# 3️⃣ $_SERVER

## 📌 بتعمل ايه؟

بتجيب معلومات عن السيرفر والطلب

---

## 📌 أهم استخداماته

```php
<?php
echo $_SERVER['PHP_SELF'];        // اسم الملف الحالي
echo "<br>";
echo $_SERVER['SERVER_NAME'];     // اسم السيرفر
echo "<br>";
echo $_SERVER['REQUEST_METHOD'];  // GET ولا POST
echo "<br>";
echo $_SERVER['REMOTE_ADDR'];     // IP المستخدم
?>
```

---

## 📌 مثال مهم عملي

نعرف الفورم اتبعت بـ POST ولا GET

```php
<?php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    echo "Form Sent Using POST";
} else {
    echo "Not Sent Yet";
}
?>
```

دي طريقة احترافية تمنع إن الصفحة تنفذ الكود غير لما الفورم يتبعت 👌

---

# 4️⃣ header()

## 📌 بتعمل ايه؟

بتتحكم في HTTP Header  
وأهم استخدام ليها هو **التحويل بين الصفحات**

---

## 📌 مثال تحويل

```php
header("Location: page2.php");
exit();
```

⚠️ مهم جدًا:  
لازم تتحط قبل أي echo  
ولا هيطلع error

---

# 🔥 إزاي أتنقل بين الصفحات؟

عندك 3 طرق:

### 1️⃣ عن طريق `<a>`

```html
<a href="page2.php">Go</a>
```

---

### 2️⃣ عن طريق form

```html
<form action="page2.php" method="POST">
```

---

### 3️⃣ عن طريق header()

```php
header("Location: page2.php");
exit();
```

---

# 🔥🔥 مثال كامل شامل عملي

## 🎯 نعمل تسجيل دخول بسيط

---

## index.php

```php
<form method="POST" action="login.php">
    <input type="text" name="username" placeholder="Enter username">
    <input type="password" name="password" placeholder="Enter password">
    <button type="submit">Login</button>
</form>
```

---

## login.php

```php
<?php

if ($_SERVER['REQUEST_METHOD'] == "POST") {

    $username = $_POST['username'];
    $password = $_POST['password'];

    if ($username == "ali" && $password == "1234") {

        header("Location: home.php?user=" . $username);
        exit();

    } else {

        echo "Wrong Data";
    }

}
?>
```

---

## home.php

```php
<?php
echo "Welcome " . $_GET['user'];
?>
```

---

# 🔥 اللي حصل هنا

1. المستخدم كتب البيانات
    
2. البيانات راحت بـ POST
    
3. اتأكدنا منها
    
4. عملنا redirect بـ header
    
5. بعتنا الاسم بـ GET
    
6. عرضناه في الصفحة الجديدة
    

---

# 💎 ملخص سريع

|المتغير|وظيفته|
|---|---|
|$_GET|يستقبل بيانات من URL|
|$_POST|يستقبل بيانات من form|
|$_SERVER|معلومات عن السيرفر والطلب|
|header()|تحويل صفحات / تحكم في الهيدر|

---

لو عايز أعملك  
🔴 مشروع مصغر فيه validation  
🔴 أو أشرحلك session & cookies  
🔴 أو أعملك CRUD كامل

قولي ونعمله عملي خطوة خطوة 🔥