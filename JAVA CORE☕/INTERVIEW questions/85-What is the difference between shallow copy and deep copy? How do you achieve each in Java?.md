طبعًا. دي من الحاجات اللي لازم **تفهمها مش تحفظها**، لأن موضوع الـ `shallow copy` و`deep copy` مرتبط جدًا بالـ **references والـ objects والـ memory** في Java.

# أولًا: يعني إيه Copy أصلًا؟

افترض عندنا:

```java
class Address {
    String city;

    Address(String city) {
        this.city = city;
    }
}

class Person {
    String name;
    Address address;

    Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }
}
```

وعندنا:

```java
Address address = new Address("Cairo");

Person p1 = new Person("Ali", address);
```

في الذاكرة بشكل مبسط:

```text
p1
 │
 ▼
┌─────────────────┐
│ Person          │
│ name = "Ali"    │
│ address ────────┼──────┐
└─────────────────┘      │
                         ▼
                  ┌──────────────┐
                  │ Address      │
                  │ city="Cairo" │
                  └──────────────┘
```

خد بالك:

`p1.address` مش بيخزن الـ `Address object` نفسه.

هو بيخزن **reference** يشاور على الـ object.

---

# 1. Shallow Copy

الـ **Shallow Copy** معناها:

> بنعمل object جديد، لكن الـ references الموجودة جواه بتفضل بتشير لنفس الـ objects الأصلية.

يعني بننسخ الـ **top-level object** فقط.

مثال:

```java
Person p2 = new Person(p1.name, p1.address);
```

دلوقتي:

```text
p1 ────────┐
           │
           ▼
      ┌─────────────┐
      │ Person      │
      │ name="Ali"  │
      │ address ────┼──────┐
      └─────────────┘      │
                           ▼
                      ┌──────────────┐
                      │ Address      │
                      │ city="Cairo" │
                      └──────────────┘
                           ▲
                      ┌────┘
                      │
      ┌─────────────┐ │
p2 ──►│ Person      │─┘
      │ name="Ali"  │
      │ address ────┘
      └─────────────┘
```

لاحظ:

```java
p1 != p2
```

لكن:

```java
p1.address == p2.address
```

دي **true**.

لأن الاتنين بيشاوروا على نفس `Address`.

---

# مثال يوضح المشكلة

```java
Address address = new Address("Cairo");

Person p1 = new Person("Ali", address);

Person p2 = new Person(p1.name, p1.address);

p2.address.city = "Alexandria";

System.out.println(p1.address.city);
```

النتيجة:

```text
Alexandria
```

ليه؟

لأن:

```text
p1.address ─────┐
                ▼
           Address
           city = Alexandria
                ▲
                │
p2.address ─────┘
```

إحنا غيرنا نفس الـ object.

---

# 2. Deep Copy

الـ **Deep Copy** مختلفة.

معناها:

> بنعمل object جديد، وكمان بنعمل copies جديدة لكل objects اللي جواه بدل ما نشارك نفس الـ references.

يعني:

```text
p1
 │
 ▼
Person #1
 │
 └──────► Address #1
           city = Cairo


p2
 │
 ▼
Person #2
 │
 └──────► Address #2
           city = Cairo
```

هنا:

```java
p1 != p2
```

وكمان:

```java
p1.address != p2.address
```

وده هو الفرق الأساسي.

---

# نعمل Deep Copy إزاي؟

مثلاً نكتب copy constructor:

```java
class Person {

    String name;
    Address address;

    Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    // Deep copy constructor
    Person(Person other) {
        this.name = other.name;
        this.address = new Address(other.address.city);
    }
}
```

وبعدين:

```java
Person p1 = new Person(
    "Ali",
    new Address("Cairo")
);

Person p2 = new Person(p1);
```

دلوقتي:

```java
p2.address.city = "Alexandria";
```

لو عملنا:

```java
System.out.println(p1.address.city);
```

هتطلع:

```text
Cairo
```

لأن:

```text
p1.address ───► Address #1
                 Cairo

p2.address ───► Address #2
                 Alexandria
```

كل واحد عنده object مستقل.

---

# طيب إيه علاقة الموضوع بالـ Primitive؟

دي نقطة مهمة جدًا.

لو عندك:

```java
class Person {
    String name;
    int age;
}
```

وعملت:

```java
Person p2 = new Person(p1.name, p1.age);
```

فالـ `int` بيتنسخ كـ value عادي.

يعني:

```text
p1.age = 25

        ↓ copy

p2.age = 25
```

تغيير:

```java
p2.age = 30;
```

مش هيغير:

```java
p1.age
```

لأن `int` primitive.

---

# لكن الـ String؟

الـ `String` Object، لكن الموضوع مختلف شوية.

```java
String name = "Ali";
```

الـ `String` **immutable**.

يعني لما تعمل:

```java
p2.name = "Ahmed";
```

إنت مش بتغير نفس String `"Ali"`.

إنت بتخلي `p2.name` يشاور على String تاني.

عشان كده مشاركة reference للـ `String` عادةً مش بتعمل مشكلة زي `Address`.

---

# Shallow Copy باستخدام `clone()`

من الطرق التقليدية في Java استخدام:

```java
clone()
```

مثلاً:

```java
class Person implements Cloneable {

    String name;
    Address address;

    @Override
    public Person clone() {
        try {
            return (Person) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

لما تعمل:

```java
Person p2 = p1.clone();
```

`super.clone()` بيعمل **shallow copy**.

يعني:

```text
p1 ─────► Person #1
             │
             └────► Address #1
                      ▲
                      │
p2 ─────► Person #2 ──┘
```

فـ:

```java
p1 != p2
```

لكن:

```java
p1.address == p2.address
```

**true**.

---

# Deep Copy باستخدام `clone()`

ممكن تعمل Deep Copy يدويًا:

```java
@Override
public Person clone() {
    try {
        Person copy = (Person) super.clone();

        copy.address = new Address(this.address.city);

        return copy;

    } catch (CloneNotSupportedException e) {
        throw new RuntimeException(e);
    }
}
```

هنا:

```java
Person p2 = p1.clone();
```

هيعمل:

```text
p1 ─────► Person #1
             │
             └────► Address #1


p2 ─────► Person #2
             │
             └────► Address #2
```

فده Deep Copy.

لكن في modern Java، أنا عادةً أفضل **copy constructor أو factory method** بدل الاعتماد على `clone()`، لأنه أوضح وأسهل في التحكم.

---

# الفرق في مثال واحد

عندنا:

```java
Person p1 = new Person(
    "Ali",
    new Address("Cairo")
);
```

### Shallow Copy:

```java
Person p2 = new Person(p1.name, p1.address);
```

الـ diagram:

```text
       p1
        │
        ▼
    Person #1
        │
        ▼
   Address #1
     Cairo
        ▲
        │
    Person #2
        ▲
        │
       p2
```

نفس الـ `Address`.

---

### Deep Copy:

```java
Person p2 = new Person(p1);
```

والـ constructor:

```java
Person(Person other) {
    this.name = other.name;
    this.address = new Address(other.address.city);
}
```

الـ diagram:

```text
p1
 │
 ▼
Person #1
 │
 ▼
Address #1
Cairo


p2
 │
 ▼
Person #2
 │
 ▼
Address #2
Cairo
```

Objects مختلفة تمامًا.

---

# طيب لو عندي Objects متداخلة أكتر؟

هنا الموضوع بيبان أكتر.

مثلاً:

```java
class Person {
    String name;
    Address address;
}

class Address {
    String city;
    Country country;
}

class Country {
    String name;
}
```

عندك:

```text
Person
  │
  └── Address
         │
         └── Country
```

لو عايز **Deep Copy حقيقي**، مش كفاية تعمل:

```java
this.address = other.address;
```

ولا حتى:

```java
this.address = new Address(
    other.address.city,
    other.address.country
);
```

لأن `country` لسه shared.

لازم تعمل:

```text
Person copy
    │
    └── New Address
            │
            └── New Country
```

يعني الـ copy يمتد للـ **object graph** كله، حسب ما تحتاجه semantics بتاعة الـ class.

---

# مقارنة سريعة جدًا 🧠

||Shallow Copy|Deep Copy|
|---|---|---|
|Top-level object|جديد|جديد|
|Primitive fields|تتنسخ|تتنسخ|
|Object references|غالبًا نفس references|References تشير إلى objects جديدة|
|تعديل nested object|ممكن يؤثر على الأصل|لا يؤثر على الأصل|
|`p1.address == p2.address`|`true`|`false`|

---

# أهم نقطة تفهمها

خلي الجملة دي ثابتة في دماغك:

> **Shallow copy copies the object, but shares the referenced objects.**  
> **Deep copy copies the object and the objects it refers to.**

مثال:

```java
Person p2 = new Person(p1.name, p1.address);
```

ده:

```text
New Person
    +
Same Address
```

أما:

```java
Person p2 = new Person(p1);
```

لو الـ constructor بيعمل:

```java
this.address = new Address(other.address.city);
```

فده:

```text
New Person
    +
New Address
```

---

## 🎯 لو اتسألت في Interview

**Interviewer:** What is the difference between shallow copy and deep copy?

تقدر تقول:

> **A shallow copy creates a new top-level object, but its reference fields still point to the same objects as the original. A deep copy creates a new top-level object and also creates independent copies of the referenced mutable objects, so changes to the copied object's nested state don't affect the original.**

وبعدين لو سألك:

**How can you achieve them in Java?**

قول:

> **A shallow copy can be achieved using a copy constructor that copies references, or `clone()` with `super.clone()`. A deep copy can be implemented using a copy constructor or factory method that recursively creates new objects for the mutable referenced fields.**