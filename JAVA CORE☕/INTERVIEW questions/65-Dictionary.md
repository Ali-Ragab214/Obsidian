ممتاز. لكن فيه نقطة مهمة جدًا قبل ما نبدأ.

في **Java** مفيش حاجة اسمها **Dictionary** بالمعنى اللي في Python أو C#. في الانترفيو لو قلت "Dictionary" غالبًا المقصود هو **Map**، وبالتحديد **HashMap**.

> لو سألك:
> 
> **"Do you know Dictionary in Java?"**
> 
> الإجابة الاحترافية:
> 
> **"Java has an old `Dictionary` abstract class, but it has been obsolete for many years. In modern Java we use the `Map` interface, most commonly `HashMap`."**

---

# 🎯 Interview Headline

# **Map (HashMap) in Java – Complete Interview Guide**

---

# أولاً: ما هي الـ Map؟

الـ **Map** هي Data Structure بتخزن البيانات على هيئة

```text
Key  ->  Value
```

مش Elements بس.

يعني كل قيمة مرتبطة بمفتاح (Key).

مثال

```text
ID      Name

101  -> Ali
102  -> Ahmed
103  -> Sara
```

---

## الحياة الواقعية

تخيل Dictionary اللغة الإنجليزية.

```text
Apple
      ↓
تفاحة

Car
      ↓
سيارة

Book
      ↓
كتاب
```

الكلمة هي

```text
Key
```

والترجمة هي

```text
Value
```

وده بالضبط الـ Map.

---

# إنشاء HashMap

```java
Map<Integer, String> students = new HashMap<>();
```

هنا

```text
Integer
```

نوع الـ Key

و

```text
String
```

نوع الـ Value

---

# إضافة بيانات

```java
students.put(101, "Ali");
students.put(102, "Ahmed");
students.put(103, "Sara");
```

أصبحت

|Key|Value|
|---|---|
|101|Ali|
|102|Ahmed|
|103|Sara|

---

# الوصول لقيمة

```java
System.out.println(students.get(102));
```

Output

```text
Ahmed
```

---

# كيف يتم البحث؟

الـ Array

```text
0
1
2
3
```

بتبحث بالـ Index.

أما الـ Map

بتبحث بالـ Key.

---

# هل الـ Key يتكرر؟

❌ لا

```java
map.put(1,"Ali");
map.put(1,"Ahmed");
```

النتيجة

```text
1 -> Ahmed
```

القيمة القديمة تم استبدالها.

---

# هل الـ Value تتكرر؟

نعم.

```java
map.put(1,"Ali");
map.put(2,"Ali");
```

مفيش مشكلة.

---

# أشهر العمليات

## put

```java
map.put(1,"Ali");
```

إضافة أو تحديث.

---

## get

```java
map.get(1);
```

ترجع القيمة.

---

## remove

```java
map.remove(1);
```

يحذف العنصر.

---

## containsKey

```java
map.containsKey(100);
```

هل الـ Key موجود؟

---

## containsValue

```java
map.containsValue("Ali");
```

هل القيمة موجودة؟

---

## size

```java
map.size();
```

عدد العناصر.

---

## clear

```java
map.clear();
```

يمسح كل شيء.

---

## isEmpty

```java
map.isEmpty();
```

---

# Trace بسيط

```java
Map<Integer,String> map=new HashMap<>();

map.put(1,"Ali");
map.put(2,"Ahmed");
map.put(3,"Sara");

map.put(2,"Omar");
```

النتيجة

|Key|Value|
|---|---|
|1|Ali|
|2|Omar|
|3|Sara|

لأن

```text
Key = 2
```

كان موجودًا بالفعل.

---

# كيف تعمل HashMap داخليًا؟

دي أهم نقطة في أي انترفيو Java.

المبرمج يكتب

```java
map.put(101,"Ali");
```

لكن داخليًا يحدث:

```
Key

↓

hashCode()

↓

Hash

↓

Bucket

↓

تخزين العنصر
```

يعني لا تبحث عنصرًا عنصرًا.

بل تحسب

```text
Hash
```

ثم تذهب مباشرة إلى المكان المتوقع.

---

# لماذا HashMap سريعة؟

لأنها تعتمد على

```text
Hashing
```

بدلاً من

```text
Linear Search
```

---

مثال

لو عندك مليون Student.

في List

قد تضطر للمرور على آلاف العناصر حتى تجد المطلوب.

أما HashMap

تحسب

```text
hashCode
```

ثم تصل مباشرة إلى الـ Bucket المناسب.

---

# الـ Time Complexity

|العملية|Average|
|---|---|
|put|O(1)|
|get|O(1)|
|remove|O(1)|
|containsKey|O(1)|

---

لكن في أسوأ حالة

```text
O(n)
```

وده سؤال انترفيو مشهور جدًا.

---

# يعني إيه Worst Case؟

لو أكثر من Key أعطوا نفس

```text
hashCode()
```

كلهم يذهبون إلى نفس الـ Bucket.

وده يسمى

```text
Collision
```

---

مثال

```
Bucket 5

↓

Ali

↓

Ahmed

↓

Sara

↓

Omar
```

بدلاً من الوصول مباشرة

يضطر إلى المرور عليهم.

---

# كيف حلت Java مشكلة Collision؟

في Java 8 وما بعدها:

إذا زاد عدد العناصر في الـ Bucket عن حد معين (مع شروط أخرى مثل حجم الجدول)، تتحول من

```text
LinkedList
```

إلى

```text
Red-Black Tree
```

فتصبح عمليات البحث في هذه الـ Bucket تقريبًا

```text
O(log n)
```

بدلاً من

```text
O(n)
```

---

# لماذا لا تستخدم List دائمًا؟

لو عندك

```text
Employee ID

↓

Employee
```

الـ List ستجبرك على البحث عنصرًا عنصرًا.

```java
for(...)
```

أما الـ HashMap

```java
employees.get(1500);
```

مباشرة.

---

# أشهر أنواع Map

## HashMap

- الأسرع عادةً
    
- لا يحافظ على ترتيب الإدخال
    
- يسمح بـ **null Key واحد** و**عدة null Values**
    

---

## LinkedHashMap

يحافظ على ترتيب الإدخال.

```text
Ali

Ahmed

Sara
```

يبقوا بنفس الترتيب.

---

## TreeMap

يرتب العناصر حسب الـ Key.

```text
5

2

9

1
```

تصبح

```text
1

2

5

9
```

لكن عملياته تكون

```text
O(log n)
```

---

# HashMap vs TreeMap

|Feature|HashMap|TreeMap|
|---|---|---|
|Search|O(1) Average|O(log n)|
|Order|لا|نعم (Sorted by Key)|
|Uses Hashing|نعم|لا|
|Uses Tree|لا|نعم (Red-Black Tree)|
|يسمح بـ null Key|نعم (واحد)|لا|

---

# HashMap vs List

|List|HashMap|
|---|---|
|تخزن Elements|تخزن Key-Value|
|البحث غالبًا O(n)|البحث O(1) Average|
|الوصول بالـ Index|الوصول بالـ Key|
|تسمح بالتكرار|الـ Key لا يتكرر|

---

# أشهر استخدامات HashMap

✅ Cache

```text
UserId

↓

User
```

---

✅ Session Storage

---

✅ Employee Lookup

---

✅ Product Lookup

---

✅ Configuration

---

✅ API Response Mapping

---

# أسئلة انترفيو مشهورة

### 1. Why is HashMap faster than List?

**الإجابة:**  
لأنها تعتمد على **Hashing** للوصول المباشر إلى الـ Bucket المناسب، بينما الـ List غالبًا تحتاج إلى البحث الخطي (`O(n)`).

---

### 2. Can HashMap have duplicate keys?

**لا.**

إذا أضفت نفس الـ Key مرة أخرى، سيتم استبدال القيمة القديمة.

---

### 3. Can HashMap have duplicate values?

**نعم.**

---

### 4. Can HashMap store null?

- **Key:** نعم، مفتاح واحد فقط يمكن أن يكون `null`.
    
- **Value:** نعم، يمكن تخزين عدة قيم `null`.
    

---

### 5. Why is TreeMap slower?

لأنه يعتمد على **Red-Black Tree** للحفاظ على ترتيب المفاتيح، لذلك عملياته تكون `O(log n)` بدلًا من `O(1)` في المتوسط.

---

### 6. What is Collision?

هي حالة ينتج فيها أكثر من Key نفس قيمة الـ Hash، فيتم وضعهم في نفس الـ Bucket، ويجب على `HashMap` التعامل مع ذلك.

---

## 🎯 الإجابة الذهبية في الانترفيو (40 ثانية)

> **"A Map stores data as key-value pairs, unlike a List which stores only elements. The most common implementation is HashMap. It uses hashing to calculate the bucket where a key should be stored, giving average O(1) time complexity for put(), get(), and remove(). Keys must be unique, while values can be duplicated. I typically use a HashMap when I need fast lookups by an identifier such as user ID, product ID, or email. If I need insertion order, I use LinkedHashMap, and if I need sorted keys, I use TreeMap."**

---

📌 **ملحوظة أخيرة مهمة للانترفيو:** لو سألك الـ Interviewer عن **Map**، فغالبًا هيكمل بأسئلة متقدمة مثل:

- ما هو `hashCode()` و`equals()`؟ ولماذا يجب أن يكونا متوافقين؟
    
- كيف تعمل `HashMap` داخليًا؟
    
- ما هو الـ **Load Factor** والـ **Capacity**؟
    
- ما هو **Rehashing** ومتى يحدث؟
    
- كيف تتعامل `HashMap` مع الـ **Collisions**؟
    
- ما الفرق بين `HashMap` و`ConcurrentHashMap`؟
    

دي تعتبر مرحلة الـ **Advanced Java Collections**، وهي من أكثر المواضيع شيوعًا في مقابلات Java Backend.