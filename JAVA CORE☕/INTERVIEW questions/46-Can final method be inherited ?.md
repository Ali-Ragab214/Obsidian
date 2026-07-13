ده سؤال مشهور جدًا في مقابلات Java، وفيه ناس بتتلخبط بين **Inheritance** و **Overriding**.

---

# Interview Question

> **Can a final method be inherited?**

## Short Answer

**Yes, a `final` method can be inherited, but it cannot be overridden.**

---

# Explanation

كلمة `final` على Method معناها:

> **This method cannot be overridden in the child class.**

لكنها **لا تمنع الـ Inheritance**.

يعني الابن هيرث الميثود ويقدر يستخدمها كما هي، لكنه لا يستطيع تغيير تنفيذها.

---

## Example 1

```java
class Parent {

    public final void display() {
        System.out.println("Hello");
    }
}

class Child extends Parent {

}
```

هنا

الـ Child ورث الـ `display()`.

يمكنك استدعاؤها:

```java
public class Test {

    public static void main(String[] args) {

        Child child = new Child();

        child.display();
    }
}
```

### Output

```text
Hello
```

إذن

✅ تم توريث الميثود.

---

# Example 2

لو حاولت تعمل Override

```java
class Parent {

    public final void display() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    @Override
    public void display() {
        System.out.println("Child");
    }
}
```

سيظهر Compile-time Error

```text
Cannot override the final method from Parent
```

لأن `final` تمنع الـ Overriding.

---

# What actually happens?

```text
Parent

display()

      ▲
      │
inherits

Child

display()   ← نفس الميثود الموروثة
```

الـ Child يحصل على نفس الميثود الموجودة في الـ Parent.

لكن لا يستطيع استبدالها.

---

# Why does Java allow inheritance?

لأن الهدف من `final` هو **حماية تنفيذ الميثود**، وليس منع استخدامها.

تخيل عندك

```java
class BankAccount {

    public final void withdraw() {
        authenticate();
        deductBalance();
        logTransaction();
    }
}
```

المبرمج يريد أن يضمن أن خطوات السحب ستظل كما هي دائمًا.

أي كلاس يرث `BankAccount` يستطيع استخدام `withdraw()`.

لكن لا يستطيع تغيير منطقها.

---

# Difference between final class and final method

|final method|final class|
|---|---|
|Can be inherited|Cannot be inherited|
|Cannot be overridden|Cannot be extended|

---

### Example

```java
final class Parent {

}
```

```java
class Child extends Parent {

}
```

Compile Error

لأن الـ Class نفسها `final`.

---

# Interview Answer (30 seconds)

> **Yes, a `final` method can be inherited because the `final` keyword does not prevent inheritance. It only prevents overriding. The child class inherits the method and can call it normally, but it cannot provide a new implementation for it. If the child tries to override a `final` method, the compiler throws a compile-time error.**

---

## قاعدة سهلة تحفظها

- ✅ `final method` → **Inherited ✔️, Overridden ❌**
    
- ❌ `final class` → **Inherited ❌ (cannot be extended at all)**