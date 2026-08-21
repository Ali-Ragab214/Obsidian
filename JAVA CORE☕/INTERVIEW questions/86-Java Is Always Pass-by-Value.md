
ده من أهم المواضيع في Java، وسبب لخبطة كبيرة جدًا في الـ **primitives والـ objects**.

الفكرة الأساسية اللي لازم تثبت في دماغك:

> **Java always passes arguments by value. Always.**

لكن الفرق هو: **إيه القيمة اللي بتتنسخ؟**

- مع `primitive` → قيمة الـ primitive نفسها بتتنسخ.
    
- مع `object` → قيمة الـ **reference** بتتنسخ.
    

وده اللي بيخلي `foo(Dog x)` شكلها كأن Java بتبعت الـ Object نفسه، لكن الحقيقة مختلفة.

---

# 1. الأول نفهم يعني إيه Pass-by-Value

لما تعمل:

```java
go(x);
```

Java بتاخد **value** الموجودة في `x` وتحط نسخة منها في الـ parameter.

يعني:

```java
int x = 10;

go(x);
```

اللي بيحصل تقريبًا:

```text
x = 10
 │
 │ copy value
 ↓
z = 10
```

بقى عندنا متغيرين منفصلين:

```text
x ──> 10

z ──> 10
```

لو غيرت `z`:

```java
void go(int z) {
    z = 20;
}
```

يبقى:

```text
x ──> 10
z ──> 20
```

فالـ `x` ميتغيرش.

---

# 2. مثال الـ Primitive

عندنا:

```java
void go(int z) {
    z = 20;
}

public static void main(String[] args) {

    int x = 10;

    go(x); //pass by value 

    System.out.println(x);
}
```

الناتج:

```text
10
```

ليه؟

لأن:

```java
go(x);
```

بتبعت **نسخة من قيمة `x`**.

```text
Before:

x = 10


go(x)

        copy
x = 10 ───────> z = 10


Inside method:

z = 20


After:

x = 10
z = 20
```

فالـ `z` ملوش علاقة مباشرة بالـ `x`.

---

# 3. طيب إيه المختلف في الـ Object؟

هنا الجزء المهم جدًا.

عندنا:

```java
class Dog {
    int weight;
}
```

وعندنا:

```java
Dog d = new Dog();

d.weight = 10;
```

مهم تفهم إن:

```java
Dog d
```

مش معناه إن `d` هو الـ Object نفسه.

`d` عبارة عن **reference** يشاور على الـ Object.

تخيلها كده:

```text
d
│
│ reference
↓
┌─────────────┐
│ Dog Object  │
│ weight = 10 │
└─────────────┘
```

---

# 4. لما نعمل `foo(d)`

عندنا:

```java
void foo(Dog x) {
    x.weight = 20;
}
```

واستخدمنا:

```java
Dog d = new Dog();

d.weight = 10;

foo(d);

System.out.println(d.weight);
```

الناتج:

```text
20
```

ليه؟

هنا لازم نركز في الجملة دي:

> **Java passes the reference value by value.**

يعني Java **بتنسخ الـ reference**.

مش بتنسخ الـ Object.

قبل استدعاء `foo`:

```text
d
│
│
↓
┌─────────────┐
│ Dog Object  │
│ weight = 10 │
└─────────────┘
```

لما تعمل:

```java
foo(d);
```

بيتم نسخ الـ reference:

```text
        ┌─────────────────┐
        │                 ↓
d ──────┼──────────> Dog Object
        │                 ↑
        └───────────────── x
```

يعني:

```text
d ───────┐
         ↓
     Dog Object
         ↑
         │
x ───────┘
```

دلوقتي `d` و `x` **references مختلفين**، لكن الاتنين بيشاوروا على **نفس الـ Object**.

---

# 5. لما نعمل `x.weight = 20`

داخل الـ method:

```java
void foo(Dog x) {
    x.weight = 20;
}
```

`x` بيشاور على الـ Object نفسه.

فأنت بتعمل:

```text
x
│
↓
Dog Object
weight = 10
```

وبتقول:

```java
x.weight = 20;
```

فتصبح:

```text
x
│
↓
Dog Object
weight = 20
```

ولما الـ method تخلص:

```text
d
│
↓
Dog Object
weight = 20
```

لذلك:

```java
System.out.println(d.weight);
```

تطبع:

```text
20
```

---

# 6. خد بالك من نقطة مهمة جدًا ⚠️

ممكن تسأل:

> "طب ما Java كده Pass-by-Reference؟"

**لا.**

Java **مش Pass-by-Reference**.

Java:

> **Always Pass-by-Value**

لكن في حالة الـ Object:

> الـ value اللي بتتعمل لها copy هي **reference**.

يعني:

```java
Dog d = new Dog();

foo(d);
```

مش معناها:

```text
Pass Object
```

ولكن:

```text
Pass a copy of the reference
```

---

# 7. مثال يثبت إن Java فعلًا Pass-by-Value

بص على المثال ده:

```java
class Dog {
    int weight;
}

void foo(Dog x) {

    x = new Dog();

    x.weight = 50;
}
```

وعندنا:

```java
Dog d = new Dog();

d.weight = 10;

foo(d);

System.out.println(d.weight);
```

الناتج:

```text
10
```

وهنا السؤال:

**ليه مش 50؟**

دي أهم نقطة في الموضوع.

---

## قبل `foo(d)`

```text
d ───────────────┐
                 ↓
             ┌─────────┐
             │ Dog     │
             │ weight 10
             └─────────┘
```

لما نعمل:

```java
foo(d);
```

الـ reference بيتنسخ:

```text
d ───────────────┐
                 ↓
             ┌─────────┐
             │ Dog     │
             │ weight 10
             └─────────┘
                 ↑
                 │
x ───────────────┘
```

دلوقتي:

```java
x = new Dog();
```

أنت غيرت **الـ reference الموجود في `x`**.

فيبقى:

```text
d ───────────────> Dog A
                    weight = 10


x ───────────────> Dog B
                    weight = 0
```

وبعدين:

```java
x.weight = 50;
```

فتصبح:

```text
d ───────────────> Dog A
                    weight = 10


x ───────────────> Dog B
                    weight = 50
```

لما الـ method تخلص، `x` يختفي.

لكن `d` لسه بيشاور على:

```text
Dog A
weight = 10
```

لذلك:

```java
System.out.println(d.weight);
```

الناتج:

```text
10
```

---

# 8. الفرق بين الحالتين

### الحالة الأولى:

```java
void foo(Dog x) {
    x.weight = 20;
}
```

أنت بتعدل **الـ Object**.

```text
d ───────┐
         ↓
      Object
      weight=20
         ↑
         │
x ───────┘
```

إذن التعديل ظاهر بعد الـ method.

---

### الحالة الثانية:

```java
void foo(Dog x) {
    x = new Dog();
}
```

أنت بتغير **الـ reference نفسه**.

```text
d ──────> Object A

x ──────> Object B
```

الـ `d` نفسه لم يتغير.

---

# 9. احفظ القاعدة دي 🔥

لما تبعت `primitive`:

```java
int x = 10;

foo(x);
```

بيحصل:

```text
x = 10
 ↓ copy
z = 10
```

**تغيير `z` لا يغير `x`.**

---

لما تبعت Object:

```java
Dog d = new Dog();

foo(d);
```

بيحصل:

```text
d ───────┐
         ↓
       Object
         ↑
         │
x ───────┘
```

**الـ reference بيتعمله copy، والـ copy والـ original بيشاوروا على نفس الـ Object.**

لذلك:

```java
x.weight = 20;
```

يغير الـ Object، وبالتالي:

```java
d.weight
```

يشوف التغيير.

---

# 🎯 Interview Answer

لو اتسألت:

**Is Java pass-by-value or pass-by-reference?**

جاوب:

> **Java is always pass-by-value. For primitive types, the actual value is copied. For objects, the value of the reference is copied, so both references can point to the same object.**

ولو سألك:

**Why can a method modify an object's fields but cannot change the caller's reference?**

تقول:

> Because the method receives a copy of the reference. Both references point to the same object, so modifying the object's fields is visible. But if the method assigns the parameter to a new object, only the local copy of the reference changes.

دي من أهم نقاط Java اللي لازم تكون فاهمها **مش حافظها**.