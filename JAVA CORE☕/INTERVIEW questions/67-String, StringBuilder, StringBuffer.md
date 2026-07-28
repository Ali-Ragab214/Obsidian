تمام، دلوقتي إحنا في **Java Backend Interview**، وأنا الـ Interviewer.

---

# السؤال الأول

> **Can you explain the difference between String, StringBuilder, and StringBuffer in Java?**

الإجابة المثالية في الانترفيو تكون بالشكل ده:

> **All three classes are used to represent and manipulate sequences of characters. The main differences are mutability and thread safety.**
> 
> - **String** is **immutable**, meaning once it's created, it cannot be changed.
>     
> - **StringBuilder** is **mutable** and **not thread-safe**, so it's faster and preferred in single-threaded environments.
>     
> - **StringBuffer** is **mutable** and **thread-safe** because its methods are synchronized, making it suitable for multi-threaded environments, although it is slower than StringBuilder.
>     

دي إجابة أول دقيقة في الانترفيو، وبعدها تدخل في التفاصيل.

---

# أولاً : String

## يعني إيه String؟

الـ String عبارة عن **Object** بيخزن مجموعة من الـ Characters.

```java
String name = "Ali";
```

لكن أهم حاجة لازم تعرفها...

## String Immutable

يعني **لا يمكن تغيير محتوى الـ String بعد إنشائه.**

وده من أشهر أسئلة الانترفيو.

مثال

```java
String s = "Hello";

s.concat(" World");

System.out.println(s);
```

النتيجة

```text
Hello
```

ليه؟

لأن

```java
concat()
```

ما عدلتش الـ String.

هي عملت Object جديد.

---

لو عملت

```java
s = s.concat(" World");
```

النتيجة

```text
Hello World
```

لأنك خليت الـ Reference يشاور على الـ Object الجديد.

---

# ماذا يحدث في الذاكرة؟

نفترض

```java
String s = "Hello";
```

الذاكرة

```text
Heap

String Pool

+---------+
| Hello   |
+---------+

s
 │
 ▼
Hello
```

بعدها

```java
s = s.concat(" World");
```

اللي يحصل

```text
Heap

String Pool

+--------------+
| Hello        |
+--------------+

+------------------+
| Hello World      |
+------------------+

s
 │
 ▼
Hello World
```

الـ Object القديم ما اتغيرش.

اتعمل Object جديد.

---

# لماذا String Immutable؟

ده سؤال انترفيو مشهور جدًا.

الإجابة:

## 1- Security

مثلاً

```java
String username = "admin";
```

لو الـ String كانت Mutable

أي Method ممكن تغيرها.

وده خطر جدًا.

---

## 2- String Pool

علشان JVM تعمل Reuse للـ Strings.

مثلاً

```java
String a = "Java";

String b = "Java";
```

الاتنين بيشاوروا على نفس الـ Object.

```text
a ─────┐
       │
       ▼
    "Java"
       ▲
       │
b ─────┘
```

لو كانت Mutable

لما تغير

```java
a
```

يبقى

```java
b
```

هيتغير كمان.

وده كارثة.

---

## 3- Thread Safety

لأنها Immutable.

مش محتاج Synchronization.

أي Thread يقرأها بأمان.

---

## 4- HashCode Caching

الـ String بتستخدم كثير في

```java
HashMap
HashSet
```

الـ hashCode بيتحسب مرة واحدة.

لو الـ String بتتغير

يبقى الـ HashMap كله يبوظ.

---

# String Pool

سؤال مشهور جدًا.

```java
String s1 = "Java";

String s2 = "Java";
```

هل هما Object واحد؟

الإجابة

Yes.

```java
System.out.println(s1 == s2);
```

true

---

لكن

```java
String s3 = new String("Java");

System.out.println(s1 == s3);
```

false

لأن

```java
new
```

بتعمل Object جديد.

---

# StringBuilder

دلوقتي

نفترض عندك

```java
String s = "";
```

وعملت

```java
for(int i=0;i<10000;i++)
{
    s += i;
}
```

هيحصل

10000 Object.

وده بطيء جدًا.

---

هنا ظهر

StringBuilder.

---

## StringBuilder Mutable

يعني

الـ Object نفسه بيتغير.

مثال

```java
StringBuilder sb = new StringBuilder("Hello");

sb.append(" World");

System.out.println(sb);
```

الناتج

```text
Hello World
```

مفيش Object جديد.

نفس الـ Object اتعدل.

---

الذاكرة

قبل

```text
Hello
```

بعد

```text
Hello World
```

نفس الـ Object.

---

# لماذا أسرع؟

لأنه

لا يعمل

```text
New Object

New Allocation

Garbage Collection
```

كل مرة.

---

# أهم Methods

```java
append()

insert()

delete()

replace()

reverse()

length()

capacity()
```

---

مثال

```java
StringBuilder sb = new StringBuilder();

sb.append("Ali");

sb.append(" ");

sb.append("Ragab");

System.out.println(sb);
```

---

# Capacity

سؤال انترفيو.

```java
StringBuilder sb = new StringBuilder();
```

الـ Capacity الافتراضية

```text
16
```

---

لو عملت

```java
new StringBuilder("Hello");
```

الـ Capacity

```text
16 + 5 = 21
```

---

لو امتلأت؟

الـ JVM تكبرها بالمعادلة

```text
(newCapacity = oldCapacity * 2 + 2)
```

---

# StringBuffer

هو تقريبًا

نفس StringBuilder.

لكن

Thread Safe.

---

يعني

كل Method تقريبًا

مكتوب عليها

```java
synchronized
```

مثلاً

```java
append()
```

تبقى

```java
public synchronized StringBuffer append(...)
```

---

وده يمنع

Race Condition.

---

# Race Condition

نفترض Threadين

```text
Thread 1

append("A")
```

و

```text
Thread 2

append("B")
```

في نفس الوقت.

---

StringBuilder

ممكن يحصل

Data Corruption.

---

لكن

StringBuffer

بيعمل Lock.

Thread واحد يدخل.

---

# لماذا StringBuffer أبطأ؟

لأن

كل Method

بتعمل

```text
Acquire Lock

Execute

Release Lock
```

وده له Cost.

---

# مقارنة شاملة

|Feature|String|StringBuilder|StringBuffer|
|---|---|---|---|
|Mutable|❌ No|✅ Yes|✅ Yes|
|Thread Safe|✅ بسبب الـ Immutability|❌ No|✅ Yes|
|Performance|بطيء عند التعديل المتكرر|الأسرع|أبطأ من Builder|
|Synchronization|لا يحتاج|لا يوجد|يوجد|
|يستخدم مع|النصوص الثابتة|Single Thread|Multi Thread|

---

# إمتى أستخدم كل واحد؟

## استخدم String

لو النص

مش بيتغير.

```java
String country = "Egypt";
```

---

## استخدم StringBuilder

لو

هتعدل كتير.

```java
StringBuilder sql = new StringBuilder();

sql.append("SELECT * ");

sql.append("FROM USERS ");

sql.append("WHERE ID=1");
```

وده الاستخدام الأشهر في المشاريع.

---

## استخدم StringBuffer

لو

أكثر من Thread

بيعدلوا نفس النص.

وده استخدامه قليل جدًا في المشاريع الحديثة.

---

# StringBuilder ولا StringBuffer؟

لو سألك

> Which one should you prefer?

الإجابة

> **Prefer StringBuilder unless you specifically need thread safety. If multiple threads modify the same character sequence, then use StringBuffer.**

---

# أسئلة انترفيو مشهورة

## 1)

**Why is String immutable?**

الإجابة

- Security.
    
- String Pool.
    
- Thread Safety.
    
- HashCode Caching.
    

---

## 2)

**Where are String literals stored?**

الإجابة

داخل

String Pool.

---

## 3)

```java
String a = "Java";

String b = "Java";

a == b
```

الإجابة

```text
true
```

---

## 4)

```java
String a = new String("Java");

String b = new String("Java");

a == b
```

الإجابة

```text
false
```

---

## 5)

```java
a.equals(b)
```

الإجابة

```text
true
```

لأن `equals()` في `String` بيقارن المحتوى، بينما `==` بيقارن الـ References.

---

## 6)

**Which one is mutable?**

الإجابة

```text
StringBuilder

StringBuffer
```

---

## 7)

**Which one is thread-safe?**

الإجابة

```text
StringBuffer
```

---

## 8)

**Which one is faster?**

الإجابة

```text
StringBuilder
```

---

## 9)

**Why is StringBuilder faster?**

الإجابة

لأنه

- لا يستخدم `synchronized`.
    
- لا ينشئ Object جديدًا مع كل تعديل.
    
- يعدل نفس الـ Character Array داخليًا.
    

---

## 10)

**Can StringBuilder be converted to String?**

نعم.

```java
StringBuilder sb = new StringBuilder("Java");

String s = sb.toString();
```

---

# سؤال متقدم جدًا (Senior Interview)

**هل StringBuilder بيعدل الـ String؟**

الإجابة: **لا**.

هو بيحتفظ داخليًا بـ **`char[]` (أو في الإصدارات الحديثة تمثيل داخلي مكافئ)** قابل للتعديل. وعندما تستدعي:

```java
String s = sb.toString();
```

يتم إنشاء **String جديدة** من محتوى الـ `StringBuilder`. بعد ذلك، أي تعديل على `sb` **لن يؤثر** على `s` لأن الـ `String` كائن مستقل وغير قابل للتغيير.

---

# الإجابة الذهبية في الانترفيو (30 ثانية)

> **String is immutable and ideal for constant text because it is secure, thread-safe by design, and supports String Pool optimization. StringBuilder is mutable and not thread-safe, making it the fastest choice for frequent string modifications in single-threaded applications. StringBuffer is also mutable but thread-safe because its methods are synchronized, so it's appropriate when multiple threads need to modify the same character sequence, although it has a performance cost compared to StringBuilder.**