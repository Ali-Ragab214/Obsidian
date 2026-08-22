

بص يا علي، خلينا نفهم **Autoboxing** من الصفر، وبعدها نشوف أهم الـ **performance issues** والـ **bugs** اللي ممكن تحصل في الـ Java interviews.

---

# 1. الأول: يعني إيه Boxing؟

عندنا في Java نوعين:

### Primitive

```java
int x = 10;
double y = 5.5;
boolean active = true;
```

### Wrapper Classes

```java
Integer x = 10;
Double y = 5.5;
Boolean active = true;
```

كل primitive له Wrapper:

|Primitive|Wrapper|
|---|---|
|`int`|`Integer`|
|`long`|`Long`|
|`double`|`Double`|
|`float`|`Float`|
|`char`|`Character`|
|`boolean`|`Boolean`|
|`byte`|`Byte`|
|`short`|`Short`|

---

# 2. What is Autoboxing?

**Autoboxing** هو إن Java تعمل تحويل تلقائي من:

```text
primitive → wrapper object
```

مثلاً:

```java
int x = 10;

Integer y = x;
```

أنت كتبت:

```java
Integer y = x;
```

لكن Java تقريبًا بتحولها إلى:

```java
Integer y = Integer.valueOf(x);
```

يعني:

```text
int
 ↓
Autoboxing
 ↓
Integer
```

---

# 3. والعكس اسمه Unboxing

لو عندك:

```java
Integer x = 10;

int y = x;
```

Java تعمل تلقائيًا:

```java
int y = x.intValue();
```

وده اسمه:

**Unboxing**

يعني:

```text
Integer
   ↓
Unboxing
   ↓
int
```

---

# 4. مثال مهم جدًا

```java
Integer x = 10;
```

هنا حصل:

```text
10
↓
autoboxing
↓
Integer object
```

أما:

```java
int x = 10;
```

مفيش object للـ `Integer`.

---

# 5. ليه Autoboxing موجود أصلاً؟

لأن بعض الـ Java APIs محتاجة **Objects** وليس primitives.

أشهر مثال:

```java
List<Integer> numbers = new ArrayList<>();

numbers.add(10);
numbers.add(20);
numbers.add(30);
```

`List` لا تستطيع أن تكون:

```java
List<int>
```

لأن Generics في Java بتتعامل مع reference types.

فـ Java تعمل:

```java
numbers.add(Integer.valueOf(10));
```

تلقائيًا.

وده أحد أهم استخدامات Autoboxing.

---

# 6. أهم مشكلة: Performance

دي نقطة مهمة جدًا في الـ interview.

شوف:

```java
Integer x = 10;

x++;
```

إنت ممكن تتخيل إن:

```java
x++;
```

بسيطة.

لكن `Integer` **immutable**، فـ Java مش بتغير نفس الـ object.

تقريبًا العملية بتكون:

```java
x = Integer.valueOf(x.intValue() + 1);
```

يعني:

```text
Integer
   ↓
unboxing
   ↓
int
   ↓
+ 1
   ↓
autoboxing
   ↓
Integer
```

فلو عملت ده ملايين المرات:

```java
Integer x = 0;

for (int i = 0; i < 1_000_000; i++) {
    x++;
}
```

أنت بتعمل conversions كتير، وممكن يحصل allocation لـ objects حسب القيم والـ JVM optimizations.

بينما:

```java
int x = 0;

for (int i = 0; i < 1_000_000; i++) {
    x++;
}
```

أبسط وأسرع عادةً.

### الزتونة:

لو مش محتاج Object، استخدم:

```java
int
```

بدل:

```java
Integer
```

خصوصًا في **performance-critical code** أو loops كبيرة.

---

# 7. Bug خطير جدًا: NullPointerException

دي من أهم مشاكل Autoboxing.

عندك:

```java
Integer x = null;

int y = x;
```

Java محتاجة تعمل:

```java
int y = x.intValue();
```

لكن:

```java
x == null
```

فمفيش object تستدعي عليه `intValue()`.

النتيجة:

```text
NullPointerException
```

---

# 8. مثال أخطر

```java
Integer a = null;

if (a > 10) {
    System.out.println("Greater");
}
```

هيحصل:

```text
Integer
 ↓
unboxing
 ↓
int
```

يعني تقريبًا:

```java
if (a.intValue() > 10)
```

وبالتالي:

```text
NullPointerException
```

### عشان كده

خلي بالك جدًا من:

```java
Integer
Long
Double
Boolean
```

لأنهم ممكن يكونوا:

```java
null
```

لكن:

```java
int
long
double
boolean
```

مش ممكن يكونوا `null`.

---

# 9. مشكلة مشهورة جدًا: `==`

دي من أشهر أسئلة الـ interview.

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

الناتج غالبًا:

```text
true
```

ليه؟

Java عندها **Integer cache** لبعض القيم، ومنها عادةً:

```text
-128 → 127
```

فممكن `a` و `b` يشاوروا على نفس الـ cached object.

لكن:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

غالبًا:

```text
false
```

لأنهم ممكن يكونوا object مختلفين.

---

# 10. عشان كده استخدم `equals()`

بدل:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

استخدم:

```java
System.out.println(a.equals(b));
```

فتكون:

```text
true
```

لأن `equals()` بتقارن **القيمة**.

بينما `==` مع objects بتقارن:

```text
هل الاتنين نفس الـ object؟
```

---

# 11. Autoboxing ممكن يعمل Bug بطريقة غريبة

شوف:

```java
Integer a = 10;
Integer b = 20;

Integer c = a + b;
```

إيه اللي حصل؟

مش إن Java جمعت Objects.

هي عملت:

```text
a
↓
unboxing
↓
10

b
↓
unboxing
↓
20

10 + 20
↓
30

30
↓
autoboxing
↓
Integer
```

يعني تقريبًا:

```java
Integer c =
    Integer.valueOf(a.intValue() + b.intValue());
```

---

# 12. Autoboxing مع Method Overloading

دي نقطة interview حلوة.

```java
void test(int x) {
    System.out.println("int");
}

void test(Integer x) {
    System.out.println("Integer");
}
```

لو عملت:

```java
test(10);
```

الناتج:

```text
int
```

لأن Java تفضل **exact match** على إنها تعمل boxing.

---

أما:

```java
Integer x = 10;

test(x);
```

الناتج:

```text
Integer
```

لأن عندنا exact match.

---

# 13. ترتيب اختيار الـ overload مهم

Java عمومًا تفضل:

```text
Exact match
↓
Widening
↓
Boxing
↓
Varargs
```

مثلاً:

```java
void test(long x) {
    System.out.println("long");
}

void test(Integer x) {
    System.out.println("Integer");
}

test(10);
```

الناتج:

```text
long
```

ليه؟

`int → long`

ده **widening**.

أما `int → Integer` فهو **boxing**.

والـ widening له أولوية على boxing.

---

# 14. مشكلة Performance في Collections

شوف:

```java
List<Integer> numbers = new ArrayList<>();

for (int i = 0; i < 1_000_000; i++) {
    numbers.add(i);
}
```

هنا كل:

```java
i
```

هو:

```java
int
```

لكن `List<Integer>` محتاجة:

```java
Integer
```

فبيحصل:

```text
int
 ↓
Autoboxing
 ↓
Integer
 ↓
List
```

فلو عندك كمية ضخمة جدًا من البيانات، ممكن الـ boxing/unboxing يسبب:

- زيادة في memory usage
    
- زيادة في object creation
    
- ضغط أكبر على Garbage Collector
    
- performance أقل
    

عشان كده في بعض الـ high-performance applications ممكن تستخدم primitive-specialized collections أو arrays بدل `List<Integer>` حسب الحالة.

---

# 15. مقارنة مهمة جدًا

### Primitive

```java
int x = 10;
```

مناسب لما:

- مش محتاج `null`
    
- عايز performance أفضل
    
- بتعمل حسابات كتير
    

### Wrapper

```java
Integer x = 10;
```

مناسب لما:

- محتاج Object
    
- محتاج تستخدمه مع Generics
    
- محتاج `null`
    
- API معينة بتطلب reference type
    

---

# 16. مثال يجمع كل المشاكل

```java
Integer x = null;

int y = x;
```

هنا:

```text
Integer
   ↓
Unboxing
   ↓
int
```

لكن `x == null`.

إذن:

```text
NullPointerException
```

---

ومثال تاني:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

ممكن:

```text
false
```

لأن `==` بتقارن references.

لكن:

```java
System.out.println(a.equals(b));
```

تطلع:

```text
true
```

---

# 🎯 الزتونة بتاعة الـ Interview

لو سألوك:

> **What is autoboxing?**

قول:

> **Autoboxing is the automatic conversion of a primitive type to its corresponding wrapper class, for example converting `int` to `Integer`.**

مثال:

```java
int x = 10;
Integer y = x;
```

Java تقريبًا تعمل:

```java
Integer y = Integer.valueOf(x);
```

ولو سألوك:

> **What are the potential problems?**

قول 3 حاجات أساسية:

### 1. Performance

```java
Integer x = 0;

for (...) {
    x++;
}
```

ممكن يعمل boxing/unboxing بشكل متكرر.

### 2. NullPointerException

```java
Integer x = null;
int y = x;
```

الـ unboxing هنا يسبب:

```text
NullPointerException
```

### 3. `==` مع Wrapper Objects

```java
Integer a = 1000;
Integer b = 1000;

a == b
```

ممكن تكون:

```text
false
```

لأنك بتقارن references، لذلك للمقارنة بالقيمة استخدم:

```java
a.equals(b)
```

**احفظ الجملة دي:**

> **Autoboxing = primitive → wrapper. Unboxing = wrapper → primitive. Boxing can introduce performance overhead, and unboxing a null wrapper can cause NullPointerException.**