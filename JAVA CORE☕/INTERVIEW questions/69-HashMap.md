
---

# أول سؤال

> **What is HashMap?**

إجابة الانترفيو:

> **HashMap is a data structure that stores data as Key-Value pairs. It provides very fast insertion, deletion, and lookup operations with an average time complexity of O(1).**

يعني ببساطة...

بدل ما أخزن البيانات كـ List

```
Ali
Ahmed
Mohamed
```

بخزنها كـ

```
100 -> Ali
101 -> Ahmed
102 -> Mohamed
```

كل قيمة ليها مفتاح (Key).

---

# يعني اي HashMap أصلا؟

تخيل عندك مكتبة فيها مليون كتاب.

بدل ما كل مرة تمشي على الرفوف كلها...

بتدخل رقم الكتاب

```
Book ID = 202
```

يروح يقولك

```
Shelf 15
```

ثم تجيب الكتاب مباشرة.

دي فكرة الـ HashMap.

---

# بيستخدم في اي؟

تقريباً في كل مشروع Spring.

مثلاً

```
UserId -> User
```

```
15 -> Ali
```

أو

```
Email -> User
```

```
ali@gmail.com -> User Object
```

أو

```
Token -> User
```

أو

```
ProductId -> Product
```

---

# ليه اسمه HashMap؟

لأنه بيعتمد على حاجة اسمها

> Hash Function

يعني بياخد الـ Key

ويحولها إلى رقم.

مثلاً

```
Ali
```

يبقى

```
2067392
```

ثم يستخدم الرقم ده لتحديد مكان التخزين.

---

# أهم فكرة لازم تفهمها

الـ HashMap **مش بيدور على البيانات واحدة واحدة.**

هو بيحسب مكانها.

وده سر السرعة.

---

# البيانات بتتخزن فين؟

جوا الـ HashMap فيه Array كبيرة.

مثلاً

```
Index

0
1
2
3
4
5
6
7
```

كل Index اسمه

```
Bucket
```

يعني كل Bucket عبارة عن خانة.

---

مثلاً عندي

```
put("Ali",20)
```

الـ HashMap يعمل

```
hash("Ali")
```

طلع

```
5
```

يروح يخزن

```
Bucket 5

Ali ->20
```

---

بعدها

```
put("Ahmed",30)
```

طلع

```
2
```

يبقى

```
Bucket2

Ahmed->30
```

---

الشكل

```
0
1

2
Ahmed->30

3

4

5
Ali->20

6

7
```

---

# طيب ال Hash Function دي بتعمل اي؟

في Java كل Object عنده

```java
hashCode()
```

مثلاً

```java
String name="Ali";

name.hashCode();
```

هيرجع رقم.

الـ HashMap يستخدم الرقم ده.

---

# بس لحظة...

لو الرقم ضخم؟

مثلاً

```
524678567
```

والـ Array حجمها

```
16
```

هيعمل

```
524678567 % 16
```

فيطلع

```
7
```

يبقى يخزن في Bucket 7.

---

يعني المعادلة تقريباً

```
bucket = hash % arrayLength
```

(في التنفيذ الداخلي Java تستخدم عمليات Bitwise بدل `%` لأنها أسرع، لكن الفكرة المنطقية هي نفسها.)

---

# طيب لو اتنين Keys طلعوا نفس المكان؟

وده اسمه

# Collision

وده أشهر سؤال انترفيو.

مثلاً

```
Ali
```

طلع Bucket 5

وبرضو

```
Ahmed
```

طلع Bucket 5

يبقى

```
Bucket5

Ali

Ahmed
```

إزاي؟

---

في Java 7

كان بيستخدم

```
LinkedList
```

يبقى

```
Bucket5

Ali
 ↓

Ahmed
 ↓

Mohamed
```

---

لكن Java 8

لو عدد العناصر في Bucket زاد عن 8 (مع سعة جدول مناسبة)، يحول الـ LinkedList إلى

```
Red Black Tree
```

عشان البحث يبقى أسرع.

---

ليه؟

بدل

```
O(n)
```

تبقى

```
O(log n)
```

---

# يعني Java 8 أحسن؟

طبعاً.

لأنها قللت تأثير الـ Collision.

---

# شكل الـ HashMap

```
Array

↓

Bucket

↓

Node

↓

Key

↓

Value

↓

next
```

كل عنصر عبارة عن Node.

```java
class Node{

    int hash;

    K key;

    V value;

    Node next;

}
```

---

# put() بيشتغل ازاي؟

مثال

```java
map.put("Ali",20);
```

الخطوات

١

يحسب

```
hashCode()
```

↓

٢

يحدد الـ Bucket

↓

٣

يشوف المكان فاضي؟

لو آه

يحط العنصر.

---

لو لا

يشوف

هل الـ Key موجود؟

لو موجود

يعمل Update.

---

لو مش موجود

يحطه آخر الـ Chain (أو في الـ Tree إذا تحولت).

---

# مثال

```java
map.put("Ali",20);

map.put("Ali",40);
```

النتيجة

```
Ali=40
```

مش عنصرين.

لأن الـ Key لازم يكون Unique.

---

# get()

```java
map.get("Ali");
```

يحسب

```
hashCode
```

↓

يروح Bucket

↓

يدور داخلها

↓

يرجع Value.

---

# remove()

نفس الفكرة.

يحساب الـ Hash

↓

يروح للـ Bucket

↓

يشيل العنصر.

---

# هل Value لازم تكون Unique؟

لا.

```
1->Ali

2->Ali
```

عادي جداً.

---

لكن

```
1->Ali

1->Ahmed
```

لا.

هيعمل

```
Update
```

---

# هل يقبل null؟

نعم.

```java
map.put(null,"Ali");
```

مسموح.

لكن

Null Key واحد فقط.

---

أما

```
null values
```

عدد غير محدود.
يعني ممكن اعمل null key    only one 
but inifinite null values 

---

# ليه Null Key واحد؟

لأن مفيش `hashCode()` لـ `null`، فـ `HashMap` بتتعامل معاه بحالة خاصة وتخزنه في Bucket مخصص (عادةً أول Bucket).

---

# ترتيب العناصر؟

لا.

الـ HashMap

**غير مرتب.**

كل مرة ممكن يطلع ترتيب مختلف.

---

لو عايز ترتيب؟

```
LinkedHashMap
```

يحافظ على ترتيب الإدخال.

---

لو عايز ترتيب أبجدي؟

```
TreeMap
```

---

# الـ Initial Capacity

افتراضياً

```
16
```

---

يعني

```
Array size =16
```

---

# Load Factor

افتراضياً

```
0.75
```

يعني

```
16 ×0.75

=12
```

أول ما توصل لـ 12 عنصر

يحصل

```
Resize
```

---

# Resize يعني اي؟

يكبر الـ Array.

من

```
16
```

تبقى

```
32
```

ثم يعيد توزيع العناصر على الـ Buckets الجديدة (Rehashing).

---

# ليه بيعمل Resize؟

عشان يقلل الـ Collision.

---

# التعقيد complexity 

|العملية|Average|Worst Case|
|---|---|---|
|put|O(1)|O(n)|
|get|O(1)|O(n) _(أو O(log n) داخل Tree Bucket)_|
|remove|O(1)|O(n)|

---

# ليه O(1)؟

لأنه عارف مكان العنصر مباشرة.

---

# ليه O(n)؟

لو كل العناصر راحت Bucket واحدة.

---

# هل HashMap Thread Safe؟

لا.

دي أشهر سؤال.

لازم تقول

> HashMap is not thread-safe.

---

لو أكتر من Thread؟

استخدم

```java
ConcurrentHashMap
```

---

# الفرق بين HashMap و Hashtable

|HashMap|Hashtable|
|---|---|
|Not Thread Safe|Thread Safe|
|Faster|Slower|
|يسمح بـ null key|لا يسمح|
|يسمح بـ null value|لا يسمح|

---

# ليه لازم الـ Key يكون Immutable؟

مثال

```java
class User{

String name;

}
```

لو استخدمته Key

ثم غيرت

```
name
```

هيتغير

```
hashCode
```

فالـ HashMap مش هتعرف توصله تاني.

عشان كده أفضل مفاتيح هي الأنواع غير القابلة للتغيير مثل `String`، أو كائنات مخصصة لا تتغير بعد إنشائها.

---

# equals() و hashCode()

ده أهم سؤال في الانترفيو.

القاعدة الذهبية:

إذا كان

```java
a.equals(b)
```

يساوي

```
true
```

يبقى لازم

```
a.hashCode()==b.hashCode()
```

لكن العكس ليس صحيحًا؛ ممكن كائنان لهما نفس `hashCode()` لكن `equals()` بينهما تكون `false`، وده يؤدي إلى Collision فقط.

---

مثال

```java
Map<Employee,String> map=new HashMap<>();
```

لو عملت

```java
Employee e1=new Employee(1);

Employee e2=new Employee(1);
```

من غير Override

```
equals

hashCode
```

الـ HashMap هيعتبرهم شخصين مختلفين.

---

# أشهر أسئلة الانترفيو

### 1- Why HashMap is fast?

لأنه يستخدم Hash Function للوصول مباشرة إلى الـ Bucket بدل البحث الخطي.

---

### 2- What is Collision?

وجود أكثر من Key في نفس الـ Bucket.

---

### 3- How Java 8 solved Collision?

تحويل الـ LinkedList إلى Red-Black Tree عندما يزيد عدد العناصر في الـ Bucket عن حد معين.

---

### 4- Difference between HashMap and TreeMap?

HashMap أسرع ومتوسطه O(1) لكنه غير مرتب.

TreeMap أبطأ (O(log n)) لكنه يحتفظ بالعناصر مرتبة حسب المفتاح.

---

### 5- Difference between HashMap and LinkedHashMap?

LinkedHashMap يحافظ على ترتيب الإدخال.

---

### 6- Can HashMap store duplicate keys?

لا.

---

### 7- Can HashMap store duplicate values?

نعم.

---

### 8- Why override hashCode()?

لكي تعمل الكائنات المخصصة كمفاتيح بشكل صحيح، مع الحفاظ على العقد بين `equals()` و`hashCode()`.

---

### 9- Why is HashMap not thread-safe?

لأن عمليات القراءة والكتابة المتزامنة قد تؤدي إلى نتائج غير متوقعة أو فقدان بيانات إذا لم تتم مزامنتها.

---

### 10- متى تستخدم HashMap؟

- البحث السريع بواسطة مفتاح.
    
- تخزين بيانات على شكل Key → Value.
    
- بناء Cache.
    
- تجميع البيانات (Grouping).
    
- عدّ التكرارات (Frequency Count).
    
- فهرسة الكائنات حسب معرفها (ID).
    

# أمثلة عملية

### 1) عدّ تكرار الكلمات

```java
Map<String, Integer> count = new HashMap<>();

for (String word : words) {
    count.put(word, count.getOrDefault(word, 0) + 1);
}
```

---

### 2) البحث عن مستخدم بالـ ID

```java
Map<Long, User> users = new HashMap<>();

users.put(1L, new User("Ali"));

User user = users.get(1L);
```

---

### 3) تجميع الموظفين حسب القسم

```java
Map<String, List<Employee>> employeesByDept = new HashMap<>();
```

---

## إجابة نموذجية في الانترفيو (مختصرة)

> **HashMap is a key-value based data structure that uses hashing to store and retrieve data efficiently. Internally, it uses an array of buckets. The key's `hashCode()` determines the target bucket, and `equals()` is used to find the exact key within that bucket. On average, `put()`, `get()`, and `remove()` are O(1). In case of collisions, Java stores multiple entries in the same bucket using a linked list, and since Java 8, it converts long chains into a Red-Black Tree for better performance. HashMap is not thread-safe, allows one null key and multiple null values, and does not guarantee insertion order.**

إذا حفظت وفهمت النقاط السابقة، فستكون مستعدًا للإجابة على حوالي **90% من أسئلة HashMap** في مقابلات Java Backend. أما الـ **10%** المتبقية فهي تتعلق بالتفاصيل الداخلية جدًا مثل خوارزمية إعادة توزيع العناصر (rehashing)، طريقة حساب الـ hash داخل `HashMap`، وأسباب اختيار قيم مثل `16` و`0.75`، وهي أسئلة تُطرح غالبًا في الشركات التي تركز على Java Core بشكل عميق مثل Oracle أو Amazon أو بعض فرق الـ JVM.