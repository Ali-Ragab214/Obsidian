# 🎯 Interview Headline

# **Array vs List in Java – What's the Difference? When Should You Use Each?**

دا من أشهر أسئلة الانترفيو، وغالبًا بيبدأ بالشكل ده:

> **"What's the difference between an Array and a List in Java? Which one would you use and why?"**

الإجابة المثالية مش إنك تعدد الفروق وبس، لكن تشرح **الفكرة الأساسية** الأول.

---

# أولًا: يعني إيه Array؟

الـ **Array** هو هيكل بيانات (Data Structure) بيخزن مجموعة عناصر **من نفس النوع** داخل مساحة متجاورة في الذاكرة.

```java
int[] numbers = {10, 20, 30, 40};
```

أو

```java
String[] names = new String[5];
```

### أهم خصائص الـ Array

✅ الحجم ثابت (Fixed Size)

بمجرد ما تعمل Array بحجم معين...

```java
new int[5]
```

مش هتقدر تخليها

```
6
10
100
```

إلا لو أنشأت Array جديدة.

---

✅ الوصول لأي عنصر سريع جدًا

```java
numbers[2]
```

لأن العناصر جنب بعض في الذاكرة.

الوقت:

```
O(1)
```

---

✅ بتخزن Primitive Types

زي

```java
int
double
char
boolean
```

وكمان Objects

```java
Student[]
```

---

# مثال

```java
int[] arr = {5, 10, 15};

System.out.println(arr[1]);
```

Output

```
10
```

---

# مميزات Array

- أسرع في الوصول للعناصر
    
- أقل استهلاكًا للذاكرة
    
- مناسبة للأحجام الثابتة
    
- تدعم Primitive Types مباشرة
    

---

# عيوب Array

❌ الحجم ثابت

❌ مفيش

```java
add()
remove()
contains()
sort()
```

كل ده لازم تعمله بنفسك.

---

# طيب إيه هي List؟

الـ List هي Interface موجودة في Java Collections Framework.

يعني هي Collection بتسمح بتخزين عناصر بترتيب معين.

أشهر Implementation ليها:

```java
ArrayList
LinkedList
Vector
```

أغلب الوقت لما حد يقول List يقصد

```java
ArrayList
```

---

مثال

```java
List<String> names = new ArrayList<>();

names.add("Ali");
names.add("Ahmed");
names.add("Sara");
```

لاحظ

مش محتاج أحدد الحجم.

---

# أهم خصائص List

✅ Dynamic Size

كل ما تعمل

```java
add()
```

الحجم بيكبر لوحده.

---

✅ فيها Methods جاهزة

```java
add()

remove()

contains()

set()

clear()

sort()

isEmpty()

size()
```

---

مثال

```java
names.remove("Ahmed");

names.add("Omar");

System.out.println(names.contains("Ali"));
```

---

# List لا تخزن Primitive Types

مينفعش

```java
List<int>
```

لازم

```java
List<Integer>
```

وده بسبب

**Wrapper Classes**

---

# مثال

```java
List<Integer> numbers = new ArrayList<>();

numbers.add(10);
numbers.add(20);
numbers.add(30);
```

---

# الفرق الأساسي

## Array

```
ثابتة
```

## List

```
ديناميكية
```

وده أهم فرق.

---

# مقارنة Interview

|Feature|Array|List (ArrayList)|
|---|---|---|
|Size|Fixed|Dynamic|
|Primitive Types|✅ نعم|❌ لا (Wrapper فقط)|
|Add|❌|✅|
|Remove|❌|✅|
|Contains|❌|✅|
|Sorting|يدوي|Collections.sort() أو sort()|
|Performance|أسرع قليلًا|أبطأ قليلًا|
|Memory|أقل|أكثر|
|موجودة في|Java Language|Java Collections Framework|

---

# طيب ArrayList جواها عاملة إزاي؟

ودي نقطة انترفيو مهمة جدًا.

المفاجأة...

> **ArrayList مبنية أصلًا باستخدام Array.**

يعني لما تعمل

```java
ArrayList<String> list = new ArrayList<>();
```

في الخلفية بيكون فيه Array.

مثلاً

```
Capacity = 10
```

لما تمتلئ

```
[ ]
```

الـ ArrayList بتعمل

1. إنشاء Array جديدة أكبر.
    
2. نسخ كل العناصر.
    
3. حذف القديمة.
    

وده سبب إن

```
add()
```

مش دايمًا

```
O(1)
```

لكن غالبًا

```
Amortized O(1)
```

أما وقت إعادة التوسعة فيكون:

```
O(n)
```

---

# طيب إمتى أستخدم Array؟

لو

✅ الحجم معروف

مثال

```java
12 months

7 days

RGB Colors

Matrix

Chess Board

Coordinates
```

كلهم أحجام ثابتة.

---

مثال

```java
String[] months = {
"Jan",
"Feb",
"Mar"
...
};
```

---

# إمتى أستخدم List؟

لو البيانات بتزيد وتقل.

مثال

```
Users

Orders

Products

Students

Employees

Messages
```

كلها Dynamic.

---

مثال

```java
List<User> users = new ArrayList<>();
```

كل شوية

```
add

remove
```

---

# في Spring Boot بنستخدم إيه؟

تقريبًا 95% من الوقت

```java
List
```

مثال

```java
List<User> findAll();
```

أو

```java
List<OrderDto>
```

ليه؟

لأن عدد البيانات في قاعدة البيانات غير معروف مسبقًا، وبالتالي نحتاج Collection ديناميكية.

---

# هل Array أسرع؟

نعم.

لأنها:

- لا تحتاج لإعادة تخصيص غالبًا.
    
- لا يوجد Overhead خاص بالـ Collections.
    
- الوصول بالـ Index مباشر.
    

لكن الفرق في أغلب تطبيقات الـ Backend اليومية غير مؤثر مقارنة بمرونة `List`.

---

# سؤال انترفيو مشهور

### هل ArrayList هي List؟

لا.

**List** عبارة عن **Interface**.

أما **ArrayList** فهي **Class** بتنفذ (Implements) الـ List.

```java
List<String> list = new ArrayList<>();
```

هنا:

- `List` → Interface
    
- `ArrayList` → Implementation
    

وده تطبيق لمبدأ **Programming to an Interface**، لأنه بيسمح لك تغير التنفيذ لاحقًا (مثل `LinkedList`) بدون تغيير الكود الذي يتعامل مع الـ `List`.

---

# سؤال انترفيو

### هل Array هي Collection؟

الإجابة:

**لا.**

الـ Array جزء من لغة Java نفسها (Language Feature)، أما الـ Collection فهي جزء من **Java Collections Framework**، لذلك الـ Array لا تطبق واجهات مثل `Collection` أو `List`.

---

# سؤال انترفيو

### لماذا لا نستعمل Array دائمًا؟

لأنها:

- حجمها ثابت.
    
- لا توفر عمليات مثل `add()` و`remove()`.
    
- إعادة تكبيرها تتطلب إنشاء Array جديدة ونسخ العناصر يدويًا.
    

أما `List` فتتكفل بكل ذلك تلقائيًا.

---

# سؤال انترفيو

### لماذا لا نستعمل List دائمًا؟

لأن:

- تستهلك ذاكرة أكثر.
    
- لا تدعم Primitive Types مباشرة.
    
- يوجد Overhead إضافي بسبب المرونة التي تقدمها.
    
- إذا كان الحجم ثابتًا ومعروفًا، فالـ Array تكون أبسط وأسرع.
    

---

# الإجابة المثالية في الانترفيو (30 ثانية)

> **"An Array has a fixed size, stores elements of the same type, supports primitive types, and provides very fast indexed access with low memory overhead. A List, usually an ArrayList, is part of the Java Collections Framework. It has a dynamic size and provides built-in methods like add(), remove(), and contains(), making it much easier to work with changing data. I use an Array when the size is fixed and known in advance, and I use a List when the number of elements can grow or shrink, which is the case in most Spring Boot applications."**

**3. إيه هو Time Complexity للعمليات الأساسية في الـ Array؟ (Access, Search, Insert, Delete)**

> الإجابة: Access = O(1)، Search = O(n) (أو O(log n) لو Sorted وعملنا Binary Search)، Insert/Delete في الآخر = O(1)، لكن في النص أو الأول = O(n) لأنك محتاج تزحزح العناصر.