أيوه، ده بيندرج تحت:

# **Method Overloading + Autoboxing + Method Resolution**

خلينا نفهمه بالمفيد 👇

```java
void test(int x) {
    System.out.println("int");
}

void test(Integer x) {
    System.out.println("Integer");
}
```

عندنا هنا **Method Overloading** لأن عندك نفس اسم الميثود `test` لكن الـ parameters مختلفة:

- واحدة تستقبل `int`
    
- واحدة تستقبل `Integer`
    

---

## 1. لو بعت `int`

```java
test(10);
```

الناتج:

```text
int
```

### ليه؟

لأن `10` أصلًا نوعها `int`، فـ Java هتختار:

```java
void test(int x)
```

وده اسمه **Exact Match**.

---

## 2. طب ليه مش اختارت `Integer`؟

عشان Java ممكن تعمل **Autoboxing**:

```java
int → Integer
```

يعني كان ممكن تحول:

```java
10
```

إلى:

```java
Integer.valueOf(10)
```

وبعدين تستدعي:

```java
test(Integer x)
```

**لكن Java تفضل الـ Exact Match على الـ Autoboxing.**

يعني الترتيب هنا:

```text
Exact Match
    ↓
Autoboxing / Widening
```

فبما إن `int` موجود بالفعل، مش محتاجة تعمل Boxing.

---

# مثال 3: لو بعت Integer

```java
Integer num = 10;

test(num);
```

الناتج:

```text
Integer
```

لأن هنا عندنا **Exact Match** مع:

```java
void test(Integer x)
```

---

# الخلاصة المهمة جدًا 🔥

الكود:

```java
void test(int x)
void test(Integer x)
```

مع:

```java
test(10);
```

Java بتعمل الآتي:

```text
10 is int
   ↓
هل يوجد test(int)؟
   ↓
Yes
   ↓
Call test(int)
```

ولا تحتاج تعمل:

```text
int → Integer
```

### الزتونة للانترفيو:

> **When overloaded methods differ by primitive and wrapper types, Java prefers an exact type match over autoboxing.**

مثال:

```java
test(10);        // int
test(Integer.valueOf(10)); // Integer
```

وده جزء مهم من فهم **كيف Java تختار أي overloaded method يتم استدعاؤها (Method Resolution)**.

==java prefer exact type match ,when overloaded methods differ at the parameter between primitive and Wrapper java prefer the exact match and will take the Primitive instead of Wrapper that will lead to AutoBoxing processes .==

