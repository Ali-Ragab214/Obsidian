ولا يهمك، تعال نفهمها بالرسمة، لأنها بتتلخبط على ناس كتير.

## أول حاجة: إيه هو `Integer`؟

لما تكتب:

```java
Integer x = 128;
```

فأنت **مش بتخزن الرقم نفسه**، أنت بتخزن **Object** جواه الرقم 128.

تخيلها كده:

```text
x
 |
 ▼
+---------+
| value=128 |
+---------+
```

---

## لما تعمل:

```java
Integer x = 128;
Integer y = 128;
```

هل `x` و `y` بيشاوروا على نفس الـ Object؟

الإجابة: **لأ**.

بيبقى كده:

```text
x ─────► +---------+
         | value=128 |
         +---------+

y ─────► +---------+
         | value=128 |
         +---------+
```

فيه **Objectين مختلفين**، رغم إن القيمة اللي جواهم واحدة.

---

## طيب `==` بيقارن إيه؟

`==` مع الـ Objects بيقارن **العنوان (Reference)**، مش القيمة.

يعني بيسأل:

> هل `x` و `y` بيشاوروا على نفس الـ Object؟

في المثال السابق:

```text
x ----► Object رقم 1

y ----► Object رقم 2
```

إذن:

```java
System.out.println(x == y);
```

الناتج:

```text
false
```

---

## طب ليه مع 127 بتطلع `true`؟

جرب:

```java
Integer x = 127;
Integer y = 127;

System.out.println(x == y);
```

الناتج:

```text
true
```

ليه؟

لأن الـ JVM عندها حاجة اسمها **Integer Cache**.

للأرقام من:

```text
-128 إلى 127
```

بتعمل **Object واحد فقط** وتخليه في الذاكرة.

يبقى:

```text
        +---------+
        | value=127 |
        +---------+
         ▲       ▲
         |       |
         x       y
```

الاتنين بيشاوروا على **نفس الـ Object**.

لذلك:

```java
x == y
```

ترجع:

```text
true
```

---

## أما 128؟

الـ Cache خلص عند 127.

فلما تكتب:

```java
Integer x = 128;
Integer y = 128;
```

الـ JVM بتقول:

> 128 مش موجودة في الـ Cache، هعمل Object جديد.

فيبقى:

```text
Object 1 (128)  ◄── x

Object 2 (128)  ◄── y
```

إذن:

```java
x == y
```

يساوي:

```text
false
```

---

## طيب لو استخدمت `equals()`؟

```java
Integer x = 128;
Integer y = 128;

System.out.println(x.equals(y));
```

دي مش بتبص على العنوان، دي بتبص على **القيمة**.

يعني:

```text
128 == 128
```

فالنتيجة:

```text
true
```

---

## سؤال إنترفيو مشهور

ماذا سيطبع الكود التالي؟

```java
Integer a = 127;
Integer b = 127;

Integer c = 128;
Integer d = 128;

System.out.println(a == b);
System.out.println(c == d);

System.out.println(a.equals(b));
System.out.println(c.equals(d));
```

الإجابة:

```text
true
false
true
true
```

---

### سؤال ليك بقى (ممكن ييجي في الإنترفيو):

تتوقع ناتج الكود ده إيه؟

```java
Integer x = 128;
int y = 128;

System.out.println(x == y);
```

فكر فيه وقولي قبل ما أشرح، لأنه بيدخل مفهوم جديد اسمه **Auto-unboxing**.