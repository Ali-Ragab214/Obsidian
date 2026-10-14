بص يا علي، دول 4 أسئلة مهمين جدًا في Java OOP وبييجوا كتير في الـ Interviews. هنشرحهم ببساطة مع أمثلة Java والـ Rules اللي تحفظها. 👌

# 1. Diamond Problem 💎

يعني إيه Diamond Problem؟

مشكلة بتحصل لما Class يرث من أكتر من Parent Class، والـ Parents عندهم نفس الـ Method، فالـ Compiler مش بيبقى عارف يختار أنهي Implementation.

تخيل الشكل ده:

A

show()

B

C

D

لو `B` و`C` عندهم نفس الـ Method الموروثة أو الـ Implementation بتاعها، فـ `D` ممكن تواجه تعارض في الاختيار.

### هل Java بتسمح بالـ Multiple Inheritance للـ Classes؟

لأ، Java بتمنع إن Class يعمل `extends` لأكتر من Class، وده بيمنع الـ Diamond Problem بالشكل ده.

```
class A {
    void show() {}
}

class B extends A {}

class C extends A {}

// Compilation error!
class D extends B, C {}
```

لكن Java بتسمح للـ Class إنه يعمل `implements` لأكتر من Interface.

ولو عندك اتنين Interfaces فيهم نفس الـ `default method`، لازم تحل التعارض بنفسك:

```
interface A {
    default void show() {
        System.out.println("A");
    }
}

interface B {
    default void show() {
        System.out.println("B");
    }
}

class C implements A, B {

    @Override
    public void show() {
        A.super.show();
    }
}
```

هنا `C` اختارت تنفّذ نسخة `A` باستخدام `A.super.show()`.

Interview takeaway: Java doesn't support multiple inheritance of classes, but it supports multiple interfaces. Conflicting inherited default methods must be resolved.

# 2. Covariant Return Types

يعني إيه Covariant Return Type؟

ببساطة: لما تعمل `Override` لـ Method، تقدر ترجع نوع أكثر تحديدًا من الـ Return Type الأصلي، بشرط إن النوع الجديد يكون Subtype منه.

مثال:

```
class Animal {}

class Dog extends Animal {}

class Parent {
    Animal getAnimal() {
        return new Animal();
    }
}

class Child extends Parent {

    @Override
    Dog getAnimal() {
        return new Dog();
    }
}
```

ركز في الحتة دي:

- الـ Parent بيرجع `Animal`.
    
- الـ Child بيرجع `Dog`.
    
- و`Dog extends Animal`، بالتالي `Dog` نوع أكثر تحديدًا من `Animal`.
    
- إذن الـ Override صحيح.
    

لكن لو عملنا العكس:

```
class Parent {
    Dog getAnimal() {
        return new Dog();
    }
}

class Child extends Parent {

    @Override
    Animal getAnimal() { // Compilation error
        return new Animal();
    }
}
```

ده خطأ؛ لأن `Animal` مش Subtype من `Dog`.

خد بالك: الـ Covariant Return Types بتشتغل مع الـ Reference Types، مش الـ Primitive Types.

مثلاً، مينفعش تعمل Override لـ Method بترجع `int` وتخليها ترجع `long`.

# 3. Can you override private / static / final methods?

دي أهم مقارنة تحفظها:

|نوع الـ Method|هل ينفع Override؟|السبب|
|---|---|---|
|`private`|❌ لا|مش متاحة للـ Subclass عشان يعملها Override|
|`static`|❌ لا|بتعمل Method Hiding مش Overriding|
|`final`|❌ لا|Java بتمنع تغيير الـ Implementation بالـ Override|
|`public`|✅ نعم|بشرط الالتزام بقواعد الـ Overriding|
|`protected`|✅ نعم|بشرط عدم تقليل الـ Access Level|
|Package-private|✅ حسب الـ Package|لازم تكون الـ Method موروثة ومتاحة للـ Subclass|

## A. Private Method

```
class Parent {
    private void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    private void show() {
        System.out.println("Child");
    }
}
```

هل ده Override؟

لأ. الاتنين Methods منفصلتين؛ لأن `private` methods مش موروثة بالمعنى اللي يسمح بالـ Overriding.

حتى لو الاسم والـ Parameters متطابقين، دي مش Overriding.

## B. Static Method — Method Hiding

```
class Parent {
    static void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void show() {
        System.out.println("Child");
    }
}

Parent obj = new Child();
obj.show();
```

الـ Output:

```
Parent
```

ليه؟

لأن `static` methods بتتحدد حسب نوع الـ Reference، مش نوع الـ Object.

```
Parent obj = new Child();
obj.show();        // Parent

Child child = new Child();
child.show();      // Child
```

وده اسمه Method Hiding مش Method Overriding.

## C. Final Method

```
class Parent {
    final void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    void show() { // Compilation error
        System.out.println("Child");
    }
}
```

السبب إن `final` بتمنع الـ Subclass من عمل Override للـ Method.

الهدف إنك تضمن إن الـ Implementation دي متتغيرش عن طريق الـ Inheritance.

# 4. ملخص سريع للـ Interview

1. Can Java override a private method?

2. Can Java override a static method?

3. Can Java override a final method?

4. What is a covariant return type?

5. Does Java support the Diamond Problem?

نصيحتي للانترفيو: متكتفيش بحفظ إن `static` مش بتتعملها Override؛ لازم تعرف تشرح الفرق بين الـ Compile-time reference type والـ Runtime object type، لأن ده اللي بيفرق بين إجابة حافظها وإجابة فاهمها.