## هل Static Methods تتعمل Override؟

**الإجابة القصيرة:** لأ — بس فيه حاجة اسمها **Method Hiding**.

---

### الأول — فاهم ليه مينفعش Override؟

الـ `@Override` بيشتغل على مبدأ **Runtime Polymorphism** — يعني Java بتقرر في وقت التشغيل تنادي إيه.

الـ `static` methods مش مرتبطة بـ **Object** — مرتبطة بالـ **Class** نفسه. فالقرار بيتاخد وقت الـ Compile مش Runtime.

```java
class Animal {
    static void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {
    static void sound() {
        System.out.println("Dog sound");
    }
}

// لما تقول كده:
Animal a = new Dog();
a.sound();

// Output:
// Animal sound  ← مش Dog!
```

> لو كانت Override حقيقية المفروض تطبع "Dog sound" — بس دي مش Override، دي **Method Hiding**.

---

### الفرق بين Override و Method Hiding

```java
class Animal {
    // Instance Method
    void walk() {
        System.out.println("Animal walking");
    }

    // Static Method
    static void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {
    
    @Override  // ✅ Override حقيقية
    void walk() {
        System.out.println("Dog walking");
    }

    // ⚠️ Method Hiding مش Override
    // لو حطيت @Override هنا → Compile Error
    static void sound() {
        System.out.println("Dog sound");
    }
}

public class Main {
    public static void main(String[] args) {
        
        Animal a = new Dog();
        
        a.walk();   // Dog walking  ← Override: بيتبع الـ Object
        a.sound();  // Animal sound ← Hiding: بيتبع الـ Reference Type
    }
}
```

---

### جدول الفرق

||Override|Method Hiding|
|---|---|---|
|**نوع الـ Method**|Instance|Static|
|**القرار بيتاخد امتى؟**|Runtime|Compile time|
|**بيتبع إيه؟**|الـ Object الفعلي|نوع الـ Reference|
|**`@Override` شغالة؟**|✅|❌ Compile Error|

---

### مثال توضيحي كامل

```java
class Parent {
    static void staticMethod() {
        System.out.println("Parent - Static");
    }

    void instanceMethod() {
        System.out.println("Parent - Instance");
    }
}

class Child extends Parent {
    static void staticMethod() {
        System.out.println("Child - Static");
    }

    @Override
    void instanceMethod() {
        System.out.println("Child - Instance");
    }
}

public class Main {
    public static void main(String[] args) {

        Parent p = new Child();

        p.staticMethod();   // Parent - Static ← الـ Reference نوعه Parent
        p.instanceMethod(); // Child - Instance ← الـ Object فعلاً Child

        Child c = new Child();

        c.staticMethod();   // Child - Static ← الـ Reference نوعه Child
        c.instanceMethod(); // Child - Instance
    }
}
```

---

### الخلاصة

```
Static Method  → Method Hiding
               → القرار وقت الـ Compile
               → بيتبع نوع الـ Reference

Instance Method → Override
                → القرار وقت الـ Runtime
                → بيتبع الـ Object الفعلي
```

> 💡 **في الـ Interviews:** لو سألوك "هل Static methods بتتعمل Override؟" — الإجابة الصح: "لأ، بس فيه Method Hiding — والفرق الجوهري إن Override بيتبع الـ Object، والـ Hiding بيتبع الـ Reference Type."

override follow object unlike hiding follow refrence 