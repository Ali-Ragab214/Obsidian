

---

# 🎙️ Interview Simulation — Java Core

**المستوى:** Intermediate | **الأسئلة:** 40

---

## السؤال 1 — Tracing 🔍

إيه الـ output من الكود ده؟

```java
class Animal {
    String name = "Animal";

    void speak() {
        System.out.println("I am " + name);
    }
}

class Dog extends Animal {
    String name = "Dog";

    @Override
    void speak() {
        System.out.println("I am " + name);
    }
}

public class Main {
    public static void main(String[] args) {
        Animal a = new Dog();
        a.speak();
        System.out.println(a.name);
    }
}
```

**إيه الـ output؟ وليه؟**




---

## الـ Output الصح

```
I am Dog
Animal
```

---

## الشرح بالتفصيل 🔍

### السطر الأول: `a.speak()` ✅ صح

```java
Animal a = new Dog();
a.speak();
```

- الـ `speak()` هي **instance method**
- Java بتستخدم **Dynamic Dispatch (Runtime Polymorphism)**
- يعني بتبص على الـ **object الفعلي** مش على الـ reference type
- الـ object فعلاً هو `Dog` ← فبتنفذ `Dog.speak()`
- ✅ طلعت `I am Dog` وده صح

---

### السطر التاني: `a.name` ❌ غلطت فيه

```java
System.out.println(a.name);
```

- الـ `name` هي **instance variable (field)**
- الـ Fields في Java مش بتتعمل لها **Dynamic Dispatch**
- بتتحدد بناءً على الـ **reference type** مش الـ object
- الـ reference type هو `Animal` ← فبياخد `Animal.name`
- ✅ طلعت `Animal`

---

## القاعدة الذهبية 🎯  الخلاصه هنا 

||بيتحدد بـ|
|---|---|
|**Methods**|الـ Object الفعلي (Runtime)|
|**Fields**|الـ Reference Type (Compile time)|

> ده اسمه **Field Hiding** وبيجي كتير في الانترفيو! 💪
> يعني ال field بيتحدد فالكومبايل علي عكس الميثود 

---

