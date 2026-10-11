# Inner Classes في Java

الـ nested class هو كلاس معرّف جوه كلاس تاني. وبينقسم لقسمين كبار: **static nested** و **non-static (inner)**.

## الأنواع الأربعة

**1. Static Nested Class**  
كلاس جوه كلاس تاني بـ `static`. مش محتاج object من الـ outer عشان تعمل منه instance.

```java
class Outer {
    private static int x = 10;
    static class Nested {
        void show() { System.out.println(x); } // يشوف الـ static بس
    }
}
Outer.Nested n = new Outer.Nested();
```

**2. Member Inner Class (non-static)**  
كلاس جوه كلاس من غير `static`. لازم يكون فيه object من الـ outer الأول، وبيشوف كل حاجة في الـ outer حتى الـ private والـ instance fields.

```java
class Outer {
    private int x = 10;
    class Inner {
        void show() { System.out.println(x); }
    }
}
Outer o = new Outer();
Outer.Inner i = o.new Inner();
```

**3. Local Inner Class**  
كلاس معرّف جوه method. مرئي جوه الـ method دي بس، وبيقدر يستخدم المتغيرات المحلية لو كانت `final` أو effectively final.

**4. Anonymous Inner Class**  
كلاس من غير اسم، بتعرّفه وتعمل منه object في نفس اللحظة. بيتستخدم مع interface أو abstract class.

```java
Runnable r = new Runnable() {
    public void run() { System.out.println("Hi"); }
};
```

(في Java 8+ غالبًا بنستخدم lambda بدله لو الـ interface فيه method واحدة.)

## Static vs Non-static Nested

||Static Nested|Non-static (Inner)|
|---|---|---|
|محتاج object من الـ outer؟|لأ|أيوه|
|بيشوف إيه من الـ outer؟|الـ static members بس|كل حاجة، حتى الـ private والـ instance|
|الـ reference للـ outer|مفيش|فيه reference ضمني (`Outer.this`)|
|ينفع يكون فيه static members؟|أيوه|من Java 16+ أيوه، قبلها لأ|
|استخدامه|كلاس مساعد مرتبط بالـ outer منطقيًا (زي Builder)|لما الـ inner محتاج state الـ outer|

## المختصر المفيد (للإنترفيو)

- **Static nested** = كلاس عادي بس متغلف جوه كلاس تاني لسبب تنظيمي. مفيش ارتباط بـ instance معين.
- **Inner (non-static)** = مربوط بـ instance من الـ outer، وبيشيل reference ليه.
- **نقطة مهمة:** الـ inner class الـ non-static ممكن يسبب **memory leak** لأنه ماسك reference للـ outer، فلو الـ inner عاش أكتر من الـ outer، الـ outer مش هيتمسح من الـ GC.
- **القاعدة العملية:** لو الكلاس مش محتاج يوصل لـ instance fields بتاعة الـ outer، خليه `static` دايمًا.

## أمثلة حقيقية هتقابلها

- `Builder` pattern: غالبًا `static class Builder` جوه الكلاس.
- `Map.Entry`: static nested interface جوه `Map`.
- الـ Event listeners والـ callbacks: anonymous classes (أو lambdas).
- `Iterator` implementation في الـ collections: غالبًا inner class عشان يوصل لبيانات الـ collection.

لو حابب، أعملك كويز على الموضوع ده زي اللي قبله.