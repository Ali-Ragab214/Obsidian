
تمام يا علي 👨‍💻🔥  
الكود ده اسمه:

# ✅ **PDO CRUD Script**

أو بشكل أدق:

> **Database Connection + Query Execution باستخدام PDO**

وده الشكل الكلاسيك اللي أي Backend Developer بيبدأ بيه التعامل مع الداتا بيز.

دلوقتي هشرحلك الكود سطر سطر كأننا بنفك موتور 👇

---

# 🔹 الجزء الأول: بيانات الاتصال

```php
$dbType = "mysql";
$dbName = "iti_os_46_2026";
$host = "localhost";
$userName = "root";
$password = "";
```

ده اسمه:

## ✅ Database Credentials

|المتغير|معناه|
|---|---|
|dbType|نوع الداتا بيز|
|dbName|اسم الداتا بيز|
|host|السيرفر|
|userName|يوزر الداتا بيز|
|password|باسورد الداتا بيز|

---

# 🔹 إنشاء الاتصال (Connection)

```php
$connection = new PDO("$dbType:host=$host;dbname=$dbName", $userName, $password);
```

ده اسمه:

# ✅ PDO Instantiation

يعني إنشاء Object من Class اسمه PDO

هو بيعمل اي هنا؟

1. بيروح يتصل بالـ MySQL
    
2. يفتح Connection Socket
    
3. يرجعلك Object اسمه `$connection`
    

---

# 🔹 try / catch

```php
try {
   ...
} catch (PDOException $e) {
   echo $e->getMessage();
}
```

ده اسمه:

# ✅ Exception Handling

لو حصل خطأ:

- database مش موجودة
    
- الباسورد غلط
    
- السيرفر مش شغال
    

هيمسك الخطأ بدل ما يكسر الموقع.

---

# 🔹 session_start()

```php
session_start();
```

ده لبدء السيشن  
ممكن تستخدمه في login بعد كده.

---

# =============================

# 🔥 الجزء التاني: SELECT

```php
$query = "select * from users";
```

ده اسمه:

# ✅ SQL Query String

لسه String عادي.

---

```php
$sqlQuery = $connection->prepare($query);
```

ده اسمه:

# ✅ Prepared Statement

يعني:

- بيحول النص SQL لكائن Statement
    
- يجهزه للتنفيذ
    
- يحمي من SQL Injection
    

---

```php
$sqlQuery->execute();
```

ده اسمه:

# ✅ Execute Statement

بيتنفذ فعليًا على السيرفر.

---

```php
$data = $sqlQuery->fetchAll(PDO::FETCH_ASSOC);
```

ده اسمه:

# ✅ Fetching Data

بيجيب كل الصفوف ويرجع Array بالشكل ده:

```php
[
  ["id"=>1,"name"=>"Ali"],
  ["id"=>2,"name"=>"Ahmed"]
]
```

---

# =============================

# 🔥 الجزء التالت: INSERT

```php
$query ="insert into users (name,email,password) values('php','php@gmail.com','1234')";
```

ده اسمه:

# ✅ Insert Statement (Static)

بس ده غلط أمنيًا ❌

---

```php
$sqlQuery = $connection->prepare($query);
$sqlQuery->execute();
```

ده اسمه:

# ✅ Execute Insert Query

---

# 🎯 الكود ده بيعمل اي بشكل عام؟

هو عبارة عن:

# 🔥 Manual CRUD Script باستخدام PDO

CRUD معناها:

|الحرف|المعنى|
|---|---|
|C|Create|
|R|Read|
|U|Update|
|D|Delete|

---

# 🧠 المصطلحات اللي لازم تحفظها

- PDO Object
    
- DSN
    
- Prepared Statement
    
- Execute
    
- Fetch
    
- Exception Handling
    
- CRUD Operations
    

---

# ⚠️ بس عندك مشكلة في الكود

انت بتعمل:

```php
$connection = new PDO(...)
```

أكتر من مرة.

المفروض تعمله مرة واحدة بس وتستخدمه.

---

# ✅ الشكل الاحترافي الصح

## 📂 config/db.php

```php
function connect() {
    return new PDO("mysql:host=localhost;dbname=iti_os_46_2026;charset=utf8mb4",
        "root",
        "",
        [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
    );
}
```

وبعدين:

```php
$connection = connect();
```

---

# 💪 المستوى الأعلى شوية

لو سألتك في انترفيو:

الكود ده اسمه اي؟

تقول:

> This is a PDO-based database interaction script implementing CRUD operations with exception handling and prepared statements.

🔥 كده تبان فاهم مش حافظ.

---

لو عايز بقا:

- أشرحلك دورة تنفيذ الـ Query جوه MySQL Engine
    
- أعملك CRUD كامل عملي
    
- أوريك إزاي تعمل Repository Pattern احترافي
    

قولّي ونطلع Level أعلى 👨‍💻🔥