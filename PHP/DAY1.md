
---

# 🐘 PHP – Full Fundamentals Notes (Structured)

---

# 📝 1) Comments in PHP

```php
// comment
# comment
/*
   Multi-line comment
*/
```

✔ `//` و `#` → تعليق سطر واحد  
✔ `/* */` → تعليق متعدد الأسطر

---

# ⚡ 2) Dynamic Typing (Loosely Typed Language)

```php
// loosely , weekly typed language  (dynamic typing)
// variable can take any datatype and can change its datatype during execution
```

📌 معنى الكلام ده:

- المتغير مش مربوط بنوع معين.
    
- ممكن يبدأ Integer ويتحول String أو Float أثناء التنفيذ.
    

مثال (من كودك):

```php
$n1=8;
$n1="open source";
$n1=8.8;
```

---

# 📂 3) include vs require

```php
// include "navbar1.php"; 
require "navbar.php";
```

🔹 `include`

- لو الملف مش موجود → Warning
    
- البرنامج يكمل عادي
    

🔹 `require`

- لو الملف مش موجود → Fatal Error
    
- التنفيذ يقف فورًا
    

📌 استخدم `require` للملفات الأساسية زي config أو database.

---

# 🖨 4) Output Functions

```php
// echo $track;
// print $track;
// print_r($track);
// var_dump($track);
```

|Function|الاستخدام|
|---|---|
|echo|يطبع string – يقبل أكثر من parameter|
|print|يطبع string – parameter واحد – بيرجع 1|
|print_r|يطبع array أو string بشكل مقروء|
|var_dump|يطبع النوع + القيمة بالتفصيل|

---

# 🔍 5) gettype()

```php
// echo gettype($track);
```

✔ ترجع نوع المتغير  
مثال: integer, string, array

---

# 📊 6) Data Types in PHP

1. String
    
2. Integer
    
3. Float
    
4. Boolean
    
5. Array
    
6. Object
    
7. NULL
    
8. Resource
    

---

# 🧪 7) Testing Data Types

```php
is_int($age);
is_string("iti");
is_bool($Is_Student);
is_double($price);
is_array($students);
is_numeric("5d");
```

📌 ملاحظة:  
`is_numeric("5d")` → false  
لأنها مش رقم صريح.

---

# ✅ 8) Truthy & Falsy Values

## ✔ Truthy Values

- 1
    
- "text"
    
- true
    
- أي رقم ≠ 0
    
- أي string ≠ "0"
    
- أي array ≠ []
    

## ❌ Falsy Values

- 0
    
- ""
    
- "0"
    
- null
    
- false
    
- []
    

📌 مثال مهم من كودك:

```php
if("0"){
    echo "true";
}else{
    echo "false";
}
```

🔴 الناتج → false  
لأن `"0"` تعتبر falsy في PHP.

---

# ⚖ 9) Comparison Operators

```php
$n1="5";
$n2=5;

var_dump($n1==$n2);
var_dump($n1===$n2);
```

✔ `==` → مقارنة قيمة فقط  
✔ `===` → مقارنة قيمة + نوع

📌 دايمًا يفضل تستخدم `===`

---

# 🔄 10) Type Casting

```php
$n1="5";

$n2=(int)$n1;  
$n1=(int)$n1;
```

✔ `(int)` → تحويل  
✔ `settype()` → يغير نوع المتغير نفسه

---

# 🔢 11) Operators

## 1️⃣ Arithmetic Operators

```php
+  -  *  /  %  **
pow($n1,$n2);
```

📌 `%` → باقي القسمة  
📌 `**` → أس

---

## 2️⃣ Assignment Operators

```php
+=
-=
*=
/=
%=
**=
```

مثال:

```php
$n1 += $n2;  // $n1 = $n1 + $n2
```

---

## 3️⃣ Comparison Operators

```php
==  ===  !=  !==  <  >  <=  >=
```

---

## 4️⃣ Logical Operators

```php
&&
||
!
```

✔ `&&` → كل الشروط لازم true  
✔ `||` → شرط واحد true يكفي  
✔ `!` → عكس القيمة

📌 الفرق بين `&&` و `AND`

- `&&` أولوية أعلى من `AND`
    

---

# 🔀 12) Conditional Statements

## If / Else

```php
if($grade>=90){
    echo "Grade is A";
}else if($grade>=80){
    echo "Grade is B";
}else{
    echo "$n1 is zero";
}
```

---

## Switch

```php
switch ($grade) {
    case 100:
        echo "Grade is A+";
        break;
    default:
        echo "Grade is C";
}
```

---

## Ternary Operator

```php
(condition)?true:false;
```

مثال:

```php
($n1%2==0)?print"Even number":print"Odd number";
```

---

# 🔼 13) Increment / Decrement

```php
echo $n1++;  
echo $n1--;  
echo --$n1;  
echo ++$n1;
```

📌 Post → يرجع القيمة ثم يزيد  
📌 Pre → يزيد ثم يرجع القيمة

---

# 🔁 14) Loops

## While

```php
while($n1<7){
    $n1++;
}
```

## Do While

```php
do{
    $n1++;
}while($n1<7);
```

## For

```php
for($i=0;$i<7;$i++){
    echo $i;
}
```

## Foreach
# 🧠 شكل الـ foreach الرسمي

foreach ($array as $key => $value) {  
    // code  
}

```php
foreach ($arr as $index => $value) {
    echo $value;
}
```

---

# 📦 15) Arrays

## Indexed Array

```php
$arr = ["mahmoud", "adel", 1, true, 3.3, ["iti", "open source"]];
```

✔ index يبدأ من 0  
✔ `count($arr)` → طول المصفوفة

---

## Multidimensional Array

```php
$arr = [
    ["mahmoud", "adel", "mohammed"],
    [1, 2, 3],
];
```

الوصول:

```php
$arr[0][1];
```

---

## Associative Array

```php
$student = [
    "name" => "mahmoud",
    "age" => 26,
];
```

Loop:

```php
foreach ($student as $key => $value) {
    echo "Key: $key , Value: $value";
}
```

---

# 🧩 16) Array Functions

```php
array_keys($arr);
array_values($arr);
count($arr);
```

---

# 🔗 17) Array Concatenation

```php
$arr1=["mahmoud","adel"];
$arr2=["mohammed","sara","aya"];

$arr3=array_merge($arr1,$arr2);
$array3=[...$arr1,...$arr2];
```

📌 `array_merge()` يعيد ترتيب المفاتيح الرقمية  
📌 Spread operator (PHP 7.4+)

---

# 🎯 Final Important Notes

- PHP Dynamic Typed
    
- استخدم `===` بدل `==`
    
- `require` أفضل للملفات الأساسية
    
- `var_dump()` أقوى أداة Debug
    

---
