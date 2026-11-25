بص، دي من الحاجات المهمة في Java خصوصًا لما تذاكر **Inheritance + Interfaces + Polymorphism**، وبتظهر في أسئلة الـ OOP والـ Diamond Problem.

هنمشي على الأربع Rules، وكل واحدة هنشوف فيها الكود، مين اللي هيتنفذ، وليه.

# **1. Class wins — الـ Class تكسب**

لو الـ Class عندها Method موجودة بنفس الـ Signature بتاعة الـ `default method` الموجودة في الـ Interface، فالـ Class implementation هي اللي بتكسب.

### **مثال:**

```java
interface Animal {
    default void sound() {
        System.out.println("Animal sound");
    }
}

class Dog {
    public void sound() {
        System.out.println("Dog sound");
    }
}

class Labrador extends Dog implements Animal {
}
```

لو نفذنا:

```
Labrador dog = new Labrador();
dog.sound();
```

الـ Output:

```
Dog sound
```

### **ليه؟**

عشان `Labrador` ورثت الـ `sound()` من الـ `Dog`، وفي نفس الوقت عندها الـ `default sound()` من الـ `Animal`.

Java بتدي الأولوية للـ Class inheritance على الـ Interface default methods.

يعني:

```
Dog.sound()          ← inherited from superclass
Animal.sound()       ← default method

Winner: Dog.sound()
```

حتى لو الـ `Dog` مش عامل `implements Animal` أصلاً.

### **طب لو الـ superclass abstract؟**

نفس القاعدة، طالما عنده implementation فعلية للـ method.

```java
abstract class Animal {
    public void sound() {
        System.out.println("Animal sound");
    }
}

interface Pet {
    default void sound() {
        System.out.println("Pet sound");
    }
}

class Dog extends Animal implements Pet {
}
```

`Dog.sound()` هتنفذ `Animal.sound()`.

**مهم:** لو الـ superclass عنده abstract method بس، ومفيش implementation فعلية، مفيش implementation تكسب من ناحية Class wins؛ والـ concrete subclass لازم توفر implementation مناسبة لو مفيش واحدة متاحة.

# **2. Most Specific Interface Wins — الـ Interface الأقرب تكسب**

لو عندك Interface بتورث من Interface تانية، والاتنين عندهم `default method` بنفس الاسم والـ signature، فالـ Child Interface هي اللي تكسب.

### **مثال:**

```java
interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

interface B extends A {
    @Override
    default void hello() {
        System.out.println("Hello from B");
    }
}

class C implements B {
}
```

لما نعمل:

```
C obj = new C();
obj.hello();
```

الـ Output:

```
Hello from B
```

### **ليه؟**

عندنا الـ inheritance hierarchy دي:

**Interface A**

`hello() → Hello from A`

**Interface B extends A**

`hello() → Hello from B`

**Class C implements B**

الـ `B` أكثر Specific من `A` لأنها وارثة منها، وبالتالي الـ `B` هي الأقرب للـ Class `C`.

Java بتختار الـ implementation الموجودة في `B`.

### **طيب لو B مش عاملة Override؟**

```java
interface A {
    default void hello() {
        System.out.println("A");
    }
}

interface B extends A {
}

class C implements B {
}
```

هنا الـ Output:

```
A
```

لأن `B` ورثت الـ default method من `A`، ومفيش implementation أحدث تستبدلها.

# **3. Conflict → Compile-time Error — مفيش Interface أقرب من التانية**

لو الـ Class بتعمل `implements` لأكتر من Interface، والـ Interfaces دول مش بينهم علاقة Inheritance، وكل واحدة عندها `default method` بنفس الـ Signature، Java مش هتعرف تختار أنهي واحدة.

### **مثال:**

```java
interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

class C implements A, B {
}
```

ده هيعمل Compile-time Error.

لأن `C` عندها اختيارين:

```java
A.hello()
B.hello()
```

ومفيش واحدة أكثر Specific من التانية، ولا فيه Superclass عندها implementation تحسم الموضوع.

### **الحل: تعمل Override بنفسك**

```java
class C implements A, B {

    @Override
    public void hello() {
        System.out.println("Hello from C");
    }
}
```

دلوقتي:

```
C obj = new C();
obj.hello();
```

الـ Output:

```
Hello from C
```

إنت كده حسمت الـ Conflict، والـ implementation بتاعت الـ Class هي اللي هتتنفذ.

### **مثال واقعي على Conflict**

تخيل:

```java
interface Camera {
    default void start() {
        System.out.println("Camera started");
    }
}

interface MusicPlayer {
    default void start() {
        System.out.println("Music started");
    }
}

class SmartPhone implements Camera, MusicPlayer {
}
```

هل لما أعمل:

```
phone.start();
```

أبدأ الكاميرا ولا أشغل الموسيقى؟

Java مش هتقرر نيابة عنك؛ لازم إنت تحدد السلوك.

```java
class SmartPhone implements Camera, MusicPlayer {

    @Override
    public void start() {
        System.out.println("SmartPhone started");
    }
}
```

# **4. Explicit Call — تختار بنفسك أنهي Default Method تنفذ**

هنا بنكون عندنا Conflict بين الـ Interfaces، لكن بدل ما نستبدل السلوك بالكامل، ممكن نختار نستدعي الـ implementation بتاعة Interface معينة، أو الاتنين مع بعض.

### **مثال:**

```java
interface A {
    default String hello() {
        return "A";
    }
}

interface B {
    default String hello() {
        return "B";
    }
}

class C implements A, B {

    @Override
    public String hello() {
        return A.super.hello() + " + " + B.super.hello();
    }
}
```

لو عملنا:

```java
C obj = new C();

System.out.println(obj.hello());
```

الـ Output:

```
A + B
```

### **نفهم السطر ده:**

```
A.super.hello()
```

معناه:

> استدعي الـ default implementation الموجودة في الـ Interface A مباشرة.

والسطر ده:

```
B.super.hello()
```

معناه:

> استدعي الـ default implementation الموجودة في الـ Interface B مباشرة.

فإنت جمعت الاتنين في Implementation واحدة بتاعت `C`.

### **طب لو عايز A بس؟**

```java
class C implements A, B {

    @Override
    public String hello() {
        return A.super.hello();
    }
}
```

Output:

```
A
```

ولو عايز `B` بس:

```
@Override
public String hello() {
    return B.super.hello();
}
```

Output:

```
B
```

ولو عايز تعمل تعديل على الـ Behavior الأصلي:

```
@Override
public String hello() {
    return "Class C: " + A.super.hello();
}
```

Output:

```
Class C: A
```

# **ملخص الأربع Rules للحفظ في الإنترفيو**

|Rule|الحالة|النتيجة|
|---|---|---|
|Class wins|الـ Class أو الـ Superclass عندها Implementation|الـ Class Implementation تكسب|
|Most specific|Interface وارثة من Interface تانية وعندهم نفس الـ Default Method|الـ Child Interface تكسب|
|Conflict|اتنين Interfaces غير مرتبطين وعندهم نفس الـ Default Method|Compile-time Error لو Class لم تعمل Override|
|Explicit call|الـ Class عايزة تختار Implementation معينة|`A.super.method()`|

## **خريطة سريعة تفتكر بيها مين بيكسب**

**هل الـ Superclass عندها Method Implementation؟**

Yes → Class wins

No ↓

**هل فيه Interface أكثر Specific؟**

Yes → Most specific wins

No ↓

**هل فيه Conflict بين Interfaces؟**

**Yes → Override required**

اختار implementation يدويًا باستخدام `A.super.method()`

## **أسئلة ممكن تتسأل بعدهم**

1. **Why did Java introduce default methods in interfaces?**  
عشان تقدر تضيف methods جديدة للـ interfaces مع توفير implementation، من غير ما تكسر كل الـ existing classes اللي بتعمل implement للـ interface، بشرط ألا ينتج conflict غير محلول.

2. **Can a default method be overridden?**  
Yes، عادي جدًا.

3. **Can a class override a default method and call the original implementation?**  
Yes، باستخدام `InterfaceName.super.method()` لو الـ Interface direct superinterface ومسموح بالـ syntax.

4. **Can an interface override a default method inherited from another interface?**  
Yes، وتقدر تستدعي implementation الأصلية باستخدام `A.super.method()` داخل الـ Interface الـ Child.

5. **Does the Diamond Problem exist in Java?**  
Yes, with interface default methods, Java may encounter ambiguity, but it resolves it through precedence rules or requires an explicit override.

6. **Why doesn't Java have the same ambiguity with multiple class inheritance?**  
لأن Java بتسمح بالـ Multiple Inheritance of Type عن طريق Interfaces، لكن مش Multiple Inheritance of Classes، فبتقلل مشاكل وراثة الـ State والـ Implementation من أكتر من Parent Class.

وأهم حاجة تربط الموضوع باللي بتذاكره حاليًا: **Interfaces + Inheritance + Polymorphism + Composition**. الـ Default Methods بتدي الـ Interface إمكانية توفير Behavior، لكن وجود Behavior مش معناه إن الـ Interface اتحولت لـ Abstract Class؛ لسه لكل واحدة دورها في التصميم.