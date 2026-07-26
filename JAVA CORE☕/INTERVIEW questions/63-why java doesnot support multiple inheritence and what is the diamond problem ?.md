السؤال ده من أشهر أسئلة Java Core، ومش بيتسأل علشان يعرف إنك حافظ الإجابة، لكن علشان يشوف **هل فاهم تصميم Java وقرارات الـ Language Designers ولا لأ.**

في الانترفيو، لو جاوبت بـ **"Because of Diamond Problem"** وسكت، فدي إجابة ناقصة.

لازم تعرف:

1. يعني إيه Multiple Inheritance.
    
2. إيه هي Diamond Problem.
    
3. ليه Java منعتها.
    
4. إزاي Java حلت المشكلة.
    
5. ليه Interfaces مختلفة.
    
6. إزاي تجاوب باحترافية.
    

---

# أولاً: يعني إيه Inheritance؟

الـ Inheritance هو إن Class يرث من Class تاني.

مثلاً

```java
class Animal {

    void eat() {
        System.out.println("Eating");
    }

}
```

```java
class Dog extends Animal {

}
```

يبقى

```
Animal
   ↑
 Dog
```

Dog ورث كل حاجة من Animal.

وده اسمه

**Single Inheritance**

---

# طيب يعني إيه Multiple Inheritance؟

يعني Class واحد يرث من أكتر من Class.

مثلاً

```java
class Father {

}
```

```java
class Mother {

}
```

لو Java كانت بتسمح

كان ممكن نكتب

```java
class Child extends Father, Mother {

}
```

يبقى

```
      Father
         ▲
         │
         │
      Child
         │
         │
         ▼
      Mother
```

Child بقى عنده Parentين.

وده اسمه

**Multiple Inheritance**

---

# المشكلة فين؟

تعالى ناخد مثال.

```java
class Father {

    void hello() {

        System.out.println("Father");

    }

}
```

---

```java
class Mother {

    void hello() {

        System.out.println("Mother");

    }

}
```

الاتنين عندهم

```java
hello()
```

---

دلوقتي

Child

```java
class Child extends Father, Mother {

}
```

ولما أكتب

```java
Child c = new Child();

c.hello();
```

السؤال

هيطبع إيه؟

```
Father
```

ولا

```
Mother
```

ولا

Error؟

Java مش هتعرف.

وده بالظبط

# Diamond Problem

---

# ليه اسمها Diamond؟

لأن شكل الوراثة بيكون كده

```
         A
       /   \
      B     C
       \   /
         D
```

A

↓

B

↓

D

وفي نفس الوقت

A

↓

C

↓

D

الرسم عامل زي

💎

Diamond

---

# مثال كامل

```java
class A {

    void print() {

        System.out.println("A");

    }

}
```

---

```java
class B extends A {

}
```

---

```java
class C extends A {

}
```

---

لو Java كانت بتسمح

```java
class D extends B, C {

}
```

يبقى

```
          A
         / \
        /   \
       B     C
        \   /
         \ /
          D
```

---

لما نكتب

```java
D obj = new D();

obj.print();
```

هيستخدم

```
B
```

ولا

```
C
```

ولا

```
A
```

؟

المترجم مش هيعرف.

وده اسمه

# Ambiguity

يعني

غموض.

---

# طيب المثال اللي فوق بسيط

المشكلة الأكبر

لو كل Parent غير الميثود.

```java
class Father {

    void drive() {

        System.out.println("BMW");

    }

}
```

---

```java
class Mother {

    void drive() {

        System.out.println("Mercedes");

    }

}
```

Child

```
يرث الاتنين
```

يبقى

```
drive()
```

أنهي واحدة؟

BMW

ولا

Mercedes؟

---

# إذن المشكلة الأساسية

ليست الوراثة نفسها.

المشكلة هي

**Method Ambiguity**

---

# هل دي المشكلة الوحيدة؟

لا.

في مشاكل تانية.

مثلاً

---

## State Conflict

```java
class Father {

    int age = 60;

}
```

---

```java
class Mother {

    int age = 55;

}
```

Child

هيبقى عنده

```
age
```

مين فيهم؟

---

## Constructor Conflict

كل Parent ليه Constructor.

أنهي Constructor يتنفذ الأول؟

---

## Method Resolution

لو فيه

```
equals()

hashCode()

toString()
```

كل Parent غيرهم.

مين يكسب؟

---

علشان كده

مصممين Java قالوا

> الموضوع هيعمل مشاكل أكتر من فايدته.

---

# الحل في Java

Java قالت

هنسمح

بـ

```
Multiple Interface Inheritance
```

لكن

مش

```
Multiple Class Inheritance
```

يعني

ده ممنوع

```java
class C extends A, B {

}
```

لكن

ده مسموح

```java
class C implements A, B {

}
```

---

# ليه Interfaces مختلفة؟

قبل Java 8

Interfaces

مكانش فيها Implementation.

يعني

```java
interface Flyable {

    void fly();

}
```

مفيش Body.

إذن

لا يوجد Conflict.

---

Class

هو اللي هيكتب

```java
class Bird implements Flyable {

    @Override
    public void fly() {

    }

}
```

---

# لكن Java 8 جابت Default Methods

مثلاً

```java
interface A {

    default void print() {

        System.out.println("A");

    }

}
```

---

```java
interface B {

    default void print() {

        System.out.println("B");

    }

}
```

دلوقتي

رجعنا لنفس المشكلة!

---

لكن Java قالت

تمام.

أنت لازم تحلها بنفسك.

```java
class C implements A, B {

    @Override
    public void print() {

        A.super.print();

    }

}
```

أو

```java
@Override
public void print() {

    B.super.print();

}
```

أو تكتب Implementation جديد.

يعني

المبرمج هو اللي يحدد.

مش الـ JVM.

---

# سؤال انترفيو

## Why doesn't Java support multiple inheritance?

الإجابة الاحترافية:

> Java does not support multiple inheritance of classes to avoid ambiguity and complexity. If two parent classes contain methods or fields with the same name, the compiler cannot determine which implementation should be inherited. This issue is known as the Diamond Problem. Instead, Java supports multiple inheritance through interfaces, where conflicts are either avoided or explicitly resolved by the implementing class.

---

# سؤال تاني

## What is the Diamond Problem?

الإجابة:

> The Diamond Problem occurs when a class inherits from two parent classes that both inherit from the same superclass or define the same method. The child class cannot determine which implementation should be used, resulting in ambiguity.

---

# هل Java تمنع Multiple Inheritance بالكامل؟

الإجابة

❌ لا.

دي غلطة مشهورة.

الصحيح

```
Java supports

Multiple Interface Inheritance

✔

Java does NOT support

Multiple Class Inheritance

✘
```

---

# سؤال خبيث في الانترفيو

**Interviewer:**

> If Java doesn't support multiple inheritance, why can one class implement multiple interfaces?

الإجابة:

> Because interfaces represent contracts rather than state. Originally, interfaces contained only abstract methods, so there was no implementation conflict. Since Java 8, interfaces can have default methods, but any conflict must be explicitly resolved by the implementing class using method overriding.

---

# الخلاصة اللي تتحفظ

|السؤال|الإجابة|
|---|---|
|هل Java تدعم Multiple Inheritance؟|❌ للـ Classes، ✅ للـ Interfaces|
|ليه Java منعتها؟|لتجنب الـ Diamond Problem والـ Ambiguity|
|ما هي Diamond Problem؟|تعارض يحدث عندما يرث كلاس من مصدرين يقدمان نفس العضو (مثل نفس الميثود)، فلا يعرف أي تنفيذ يستخدم.|
|إزاي Java حلتها؟|باستخدام Interfaces، ومع `default` methods يجب على الكلاس حل التعارض بنفسه.|

---

## 🎯 إجابة 30 ثانية في الانترفيو

> **Java doesn't support multiple inheritance of classes because it can lead to ambiguity, known as the Diamond Problem. If two parent classes provide the same method, the compiler cannot determine which implementation the child class should inherit. To avoid this complexity, Java allows a class to extend only one class. However, Java supports multiple inheritance through interfaces because they define contracts, and if two interfaces contain conflicting default methods, the implementing class must explicitly override the method and resolve the conflict.**

دي تعتبر إجابة **Senior-level** لأنها مش بتقول "Java doesn't support multiple inheritance بسبب الـ Diamond Problem" وبس، لكنها بتشرح **المشكلة، سبب القرار، والاستثناء (Interfaces)**.