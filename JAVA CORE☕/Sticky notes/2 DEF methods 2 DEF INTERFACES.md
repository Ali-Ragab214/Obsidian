 لو عندي كلاس بي implement two different interfaces and the two diff interfaces has the same name for default methods 
 الإجابة الصحيحة هي:

**B) Override the method ✅**

---

## ليه؟

في Java 8 لما ظهر مفهوم **default methods في interfaces**، حصلت مشكلة محتملة:

لو كلاس implements اتنين interfaces  
والاتنين فيهم **نفس الـ default method بنفس التوقيع**  
يبقى يحصل **conflict** (تعارض).

في الحالة دي Java بتجبرك إنك تحل التعارض بنفسك عن طريق **override** للميثود داخل الكلاس.

---

## مثال عملي

```java
interface A {
    default void print() {
        System.out.println("A");
    }
}

interface B {
    default void print() {
        System.out.println("B");
    }
}
```

لو عملت:

```java
class Test implements A, B {
}
```

❌ هيطلع Compile-time error  
لأن Java مش عارفة تختار أنهي print() تستخدم.

---

## الحل الإجباري

لازم تعمل Override:

```java
class Test implements A, B {

    @Override
    public void print() {
        System.out.println("Resolved");
    }
}
```

✔️ كده التعارض اتحل.

---

## طب هل نقدر نستخدم super ؟

أيوه — لكن **داخل الـ override نفسه فقط**:

```java
class Test implements A, B {

    @Override
    public void print() {
        A.super.print();  // اختيار نسخة A
        B.super.print();  // اختيار نسخة B
    }
}
```

لكن المهم إنك **لازم تعمل override الأول**  
عشان كده الإجابة الصح هي **B**.

---

## ليه باقي الاختيارات غلط؟

❌ A) Nothing → غلط، لازم تحل التعارض  
❌ C) Use super keyword → مش كفاية لوحدها، لازم override  
❌ D) Causes runtime error → غلط، ده Compile-time error

---

لو عايز أختبرك بسؤال أصعب شوية في موضوع multiple inheritance في Java؟ 😈