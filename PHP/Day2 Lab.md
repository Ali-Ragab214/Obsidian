جامد جدًا يا علي 🔥  
ده شغل مذاكرة محترم 👌  
هرتبهولك بصيغة **Markdown نظيفة** تنفع تحطها في **Obsidian** كـ note منظمة 👇

---

#  PHP Practice – Strings & Arrays

---

# 🔹 String Operations

## 1️⃣ Reverse Words in a String

```php
function reverse($str){
    $arr = explode(" ", $str);      // Convert string to array
    $reversedArr = array_reverse($arr);
    return implode(" ", $reversedArr); // Convert array back to string
}
```

✔️ الفكرة:

- `explode()` → يحول string لـ array
    
- `array_reverse()` → يعكس الترتيب
    
- `implode()` → يرجعهم string تاني
    

---

## 2️⃣ Reverse Characters

```php
function reverseChars($str){
    $reversedStr = "";
    for($i = strlen($str)-1; $i >= 0; $i--){
        $reversedStr .= $str[$i];
    }
    return $reversedStr;
}
```

✔️ بنلف من آخر حرف لأول حرف

---

## 3️⃣ Count Word "track"

```php
function count_track($str)
{
    $counter = 0;
    $arr = explode(" ", $str);

    for($i = 0; $i < count($arr); $i++)
    {
        if($arr[$i] == "track")
        {
            $counter++;
        }
    }

    return $counter;
}
```

---

## 4️⃣ Count Character 'a'

```php
function count_a($str)
{
    $counter = 0;

    for($i = 0; $i < strlen($str); $i++)
    {
        if($str[$i] == "a")
        {
            $counter++;
        }
    }

    return $counter;
}
```

---

## 5️⃣ Find Position of Character

```php
$pos = strpos($str,'a');
```

✔️ `strpos()` → بترجع أول مكان للحرف

---

# 🔹 Numeric Array Operations

```php
$arr = array(22,25,28,15,27,19,13,10,0,2,1);
```

---

## 🔸 Sort Ascending

```php
sort($arr);
```

---

## 🔸 Reverse Array

```php
$arr = array_reverse($arr);
```

---

## 🔸 Max & Min

```php
$max = max($arr);
$min = min($arr);
```

---

## 🔸 Sum

```php
function sum($arr)
{
    $sum = 0;

    for($i = 0; $i < count($arr); $i++)
    {
        $sum += $arr[$i];
    }

    return $sum;
}
```

---

## 🔸 Average

```php
function average($arr)
{
    return sum($arr) / count($arr);
}
```

---

## 🔸 Even Numbers

```php
function evens($arr)
{
    $newarr = [];

    for ($i = 0; $i < count($arr); $i++)
    {
        if ($arr[$i] % 2 == 0)
        {
            $newarr[] = $arr[$i];
        }
    }

    return $newarr;
}
```

---

## 🔸 Max Even Number

```php
$max_even = max(evens($arr));
```

---

## 🔸 Max Odd Number

```php
$max_odd = max(array_filter($arr, function($num) {
    return $num % 2 != 0;
}));
```

---

# 🔹 String Array Operations

```php
$names = ["mohammed","ahmed","mahmoud","yousef","omar"];
```

### 🔸 Sort

```php
sort($names);
```

### 🔸 Reverse

```php
$names = array_reverse($names);
```

---

# 🔹 Associative Array (Multi-Dimensional)

```php
$data = [
    ["name"=>"hazem","age"=>24,"address"=>"cairo"],
    ["name"=>"david","age"=>22,"address"=>"menia"],
    ["name"=>"mahmoud","age"=>25,"address"=>"shebin"],
];
```

---

## 🔸 Sort By Age (Ascending)

```php
function sortByAgeAsc($array) {
    usort($array, function($a, $b) {
        return $a['age'] <=> $b['age'];
    });
    
    return $array;
}
```

✔️ `<=>` اسمها Spaceship Operator

---

# 🔹 Build HTML Table

```php
echo "<table border='1'>";
echo "<tr>
<th>id</th>
<th>name</th>
<th>age</th>
<th>address</th>
</tr>";

for($i=0;$i<count($data);$i++)  
{
    echo "<tr>";
    echo "<td>".$i."</td>";
    echo "<td>".$data[$i]['name']."</td>";
    echo "<td>".$data[$i]['age']."</td>";
    echo "<td>".$data[$i]['address']."</td>";
    echo "</tr>";
}   

echo "</table>";
```

---

# 🧾 Functions Used Summary

|Function|Purpose|
|---|---|
|explode()|String → Array|
|implode()|Array → String|
|array_reverse()|Reverse array|
|sort()|Sort ascending|
|max()|Maximum value|
|min()|Minimum value|
|usort()|Custom sorting|
|strpos()|Find position|
|array_filter()|Filter array|

---

# 💡 Notes

- استخدم `count($arr)` عشان تجيب طول array
    
- استخدم `strlen($str)` عشان تجيب طول string
    
- الـ associative array بيتوصل للقيم بالمفتاح
    

---

لو عايز أظبطهولك كـ:

- 📘 Version مذاكرة نظري + ملخص
    
- 📗 Cheat Sheet سريع
    
- 📙 أو أرتبه كـ Interview Revision Note
    

قولي ونظبطه احترافي أكتر 🔥