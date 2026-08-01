من أكتر الأسئلة اللي بتيجي في انترفيوهات Java هو: **What is a Wrapper Class?** وغالبًا بيتبعه أسئلة عن **Autoboxing, Unboxing, Integer Cache**.

---

# أولًا: يعني إيه Wrapper Class؟
**Wrapper Class** is a class that represents a primitive data type as an Object. It allows you to use primitives in places that require Objects, such as Collections and Generics.

ببساطة جدًا...

في Java عندنا نوعين من البيانات:

1. **Primitive Data Types**
    
2. **Reference Types (Objects)**
    

الـ Primitive زي:

```java
int
double
char
boolean
long
short
float
byte
```

ودي **مش Objects**.

لكن أحيانًا Java بتحتاج Object بدل primitive، زي لما تستخدم Collections أو Generics.

عشان كده Java عملت لكل Primitive كلاس اسمه **Wrapper Class**.

|Primitive|Wrapper Class|
|---|---|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|
|boolean|Boolean|

مثال:

```java
int age = 22;
```

ده Primitive.

أما:

```java
Integer age = 22;
```

ده Object.

---

# ليه Java عملت Wrapper Classes؟

لأن الـ Primitive Types:

- مش Objects
    
- معندهاش Methods
    
- مينفعش تتحط في Collections
    

لكن الـ Wrapper Class:

- Object
    
- فيها Methods كتير
    
- تقدر تستخدمها مع أي API محتاج Object
    

---

# مثال

بدل

```java
int x = 10;
```

تقدر تعمل

```java
Integer x = Integer.valueOf(10);
```

أو

```java
Integer x = 10;
```

(Java هتعمل Autoboxing لوحدها.)

---

# أهم مميزات Wrapper Classes

## 1) استخدامها مع Collections

مينفعش:

```java
ArrayList<int> list = new ArrayList<>();
```

❌ Compile Error

لازم:

```java
ArrayList<Integer> list = new ArrayList<>();

list.add(10);
list.add(20);
```

ليه؟

لأن الـ Collections بتتعامل مع Objects فقط.

---

## 2) فيها Methods جاهزة

مثال:

```java
Integer x = 100;

System.out.println(x.compareTo(50));
System.out.println(x.toString());
System.out.println(x.doubleValue());
```

---

## 3) Parsing

مثلا عندك String:

```java
String age = "25";
```

تحوله لـ int:

```java
int x = Integer.parseInt(age);
```

أو

```java
Integer x = Integer.valueOf(age);
```

---

## 4) Constants

```java
System.out.println(Integer.MAX_VALUE);
System.out.println(Integer.MIN_VALUE);
```

النتيجة

```
2147483647

-2147483648
```

---

## 5) Utility Methods

مثال

```java
Integer.max(10,20);

Integer.min(10,20);

Integer.sum(5,10);
```

---

# أشهر Wrapper Classes

```java
Integer

Double

Boolean

Character

Long

Float

Byte

Short
```

---

# Autoboxing

قبل Java 5 كنت لازم تكتب

```java
Integer x = Integer.valueOf(10);
```

بعد Java 5:

```java
Integer x = 10;
```

Java بتحولها تلقائيًا إلى:

```java
Integer x = Integer.valueOf(10);
```

وده اسمه

> **Autoboxing**

---

مثال

```java
Integer age = 20;
```

الـ Compiler بيحولها إلى

```java
Integer age = Integer.valueOf(20);
```

---

# Unboxing

العكس.

لو عندك

```java
Integer age = 20;
```

وعملت

```java
int x = age;
```

Java بتحولها تلقائيًا إلى

```java
int x = age.intValue();
```

وده اسمه

> **Unboxing**

---

# مثال كامل

```java
Integer number = 10;//autoboxing

int x = number;//unboxing 

System.out.println(x);
```

---

# مثال على الاثنين

```java
Integer x = 5;     // Autoboxing

int y = x;         // Unboxing
```

---

# قيمة Wrapper ممكن تكون null

وده فرق مهم جدًا.

```java
Integer x = null;
```

صح.

لكن

```java
int x = null;
```

❌ Error

---

# NullPointerException

بسبب الـ Unboxing.

```java
Integer x = null;

int y = x;
```

النتيجة

```
NullPointerException
```

لأن Java بتحاول تعمل

```java
x.intValue();
```

لكن x = null.

---

# Wrapper Classes Immutable

يعني

```java
Integer x = 5;

x = 10;
```

مش بيغير الـ Object القديم.

بيعمل Object جديد.

---

# Integer Cache (سؤال انترفيو مشهور)

شوف الكود

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

النتيجة

```
true
```

لكن

```java
Integer a = 200;
Integer b = 200;

System.out.println(a == b);
```

النتيجة

```
false
```

ليه؟

لأن Java بتعمل Cache للأرقام من:

```
-128

إلى

127
```

أي Integer في الرينج ده بيستخدم نفس الـ Object.

أما خارجه فبيعمل Object جديد.

---

# الفرق بين == و equals()

```java
Integer a = 200;
Integer b = 200;

System.out.println(a == b);
```

```
false
```

لكن

```java
System.out.println(a.equals(b));
```

```
true
```

لأن

`==`

بيقارن الـ Reference.

أما

`equals()`

بيقارن القيمة.

---

# Primitive vs Wrapper

|Primitive|Wrapper|
|---|---|
|سريع جدًا|أبطأ قليلًا|
|لا يمكن أن يكون null|يمكن أن يكون null|
|ليس Object|Object|
|لا يحتوي على Methods|يحتوي على Methods|
|لا يعمل مع Collections|يعمل مع Collections|
|أقل استهلاكًا للذاكرة|يستهلك ذاكرة أكبر|

---

# إمتى أستخدم Primitive؟

لما:

- بعمل حسابات كتير.
    
- مهتم بالأداء.
    
- مش محتاج Object.
    

مثال

```java
int sum = 0;
```

---

# إمتى أستخدم Wrapper؟

لما:

- أستخدم Collection.
    
- أستخدم Generic.
    
- محتاج null.
    
- محتاج Methods الخاصة بالكلاس.
    
- أتعامل مع APIs تتطلب Objects.
    

---

# مثال عملي

```java
List<Integer> numbers = new ArrayList<>();

numbers.add(10); // autoboxing
numbers.add(20);

int sum = 0;

for (Integer num : numbers) {
    sum += num; //unboxing
}

System.out.println(sum);
```

الـ `add(10)` حصل فيها **Autoboxing**، وداخل الحلقة حصل **Unboxing**.

---

# أسئلة انترفيو

### 1. What is a Wrapper Class?

**الإجابة:** هي Class بيمثل Primitive Type كـ Object، وبيسمح باستخدام الـ Primitive في الأماكن التي تتطلب Objects مثل Collections وGenerics.

---

### 2. Why do we need Wrapper Classes?

- لاستخدام الـ Collections.
    
- لاستخدام الـ Generics.
    
- للاستفادة من الـ Methods الجاهزة.
    
- للسماح بقيمة `null`.
    

---

### 3. What is Autoboxing?

التحويل التلقائي من Primitive إلى Wrapper.

```java
Integer x = 5;
```

---

### 4. What is Unboxing?

التحويل التلقائي من Wrapper إلى Primitive.

```java
Integer x = 5;
int y = x;
```

---

### 5. What is the difference between `parseInt()` and `valueOf()`؟

```java
int x = Integer.parseInt("10");
```

يرجع **primitive int**.

أما:

```java
Integer x = Integer.valueOf("10");
```

يرجع **Integer Object**.

---

### 6. Can Wrapper Classes be null?

نعم.

```java
Integer x = null;
```

أما:

```java
int x = null;
```

فهذا غير مسموح.

---

### 7. Why does this code throw `NullPointerException`?

```java
Integer x = null;
int y = x;
```

لأن Java بتحاول تعمل `x.intValue()` أثناء الـ Unboxing، و`x` تساوي `null`.

---

### 8. Why is `Integer` slower than `int`?

لأن `Integer` عبارة عن Object، وإنشاؤه وإدارته يستهلكان ذاكرة ووقتًا أكثر من الـ Primitive.

---

### 9. Why does this print `true`?

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

بسبب **Integer Cache** الذي يخزن القيم من **-128 إلى 127** ويعيد استخدام نفس الـ Object.

---

### 10. Why does this print `false`?

```java
Integer a = 200;
Integer b = 200;

System.out.println(a == b);
```

لأن 200 خارج نطاق الـ Cache، فيتم إنشاء Object جديد لكل متغير، وبالتالي يختلف الـ Reference.

---

## الخلاصة التي تحفظها للانترفيو

- **Wrapper Class = Object يمثل Primitive Type.**
    
- **السبب الأساسي لوجودها:** التعامل مع Collections وGenerics وAPIs التي تحتاج Objects.
    
- **Autoboxing:** `int → Integer`.
    
- **Unboxing:** `Integer → int`.
    
- **يمكن أن تكون `null`** بعكس الـ Primitive.
    
- **استخدم `equals()` للمقارنة بين القيم** وليس `==` مع الـ Wrapper Classes.
    
- **يوجد Integer Cache للقيم من -128 إلى 127**، وهو سبب اختلاف نتيجة `==` في بعض الحالات.