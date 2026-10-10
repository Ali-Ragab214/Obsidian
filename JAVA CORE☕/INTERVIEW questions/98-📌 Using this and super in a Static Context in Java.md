
---


### 1. يعني إيه `static` و `this`؟

- `static` method: بتتبع الـ Class نفسه، ومش محتاجة تعمل Object عشان تستدعيها.
    
- `this`: بتشير للـ Current Object، يعني الـ Object اللي شغّال عليه الكود حاليًا.
    
- `super`: بتستخدم للوصول لأعضاء الـ Parent Class من خلال الـ Object الحالي.
    

### 2. ليه مينفعش نستخدم `this` جوه `static` method؟

بص على المثال ده:

```
class Test {
    int v = 5;

    static void print() {
        System.out.println(this.v);
    }
}
```

الكود ده فيه Compilation Error؛ لأن `print()` هي `static` method، وبالتالي مش مرتبطة بـ Object معيّن.

تخيّل إن عندك:

```
Test obj1 = new Test();
Test obj2 = new Test();
```

كل Object عنده نسخة خاصة من `v`.

- `obj1.v` ممكن تكون `5`.
    
- `obj2.v` ممكن تكون `10` لو غيّرنا قيمتها.
    

لما تستدعي `Test.print()`، الميثود مش عارفة أنت تقصد أنهي Object، وبالتالي `this` ملهاش معنى في السياق ده.

القاعدة الذهبية للإنترفيو

`this` و `super` لا يمكن استخدامهما مباشرةً داخل `static context`؛ لأنهما مرتبطان بالـ Object، بينما `static` مرتبطة بالـ Class.

### 3. إزاي نصلّح الكود؟

الحل الأول: نشيل `static` من الميثود.

```
class Test {
    int v = 5;

    void print() {
        System.out.println(this.v);
    }
}

Test obj = new Test();
obj.print(); // 5
```

هنا عندنا Object محدد، فـ `this.v` بتشير إلى المتغير `v` الخاص بيه.

الحل الثاني: نخلي المتغير `static` ونستخدمه مباشرةً.

```
class Test {
    static int v = 5;

    static void print() {
        System.out.println(v);
    }
}

Test.print(); // 5
```

هنا `v` نفسها تابعة للـ Class، فمش محتاجين `this`.

### 4. هل نفس القاعدة تنطبق على `super`؟

أيوه، بص على المثال:

```
class Parent {
    void show() {}
}

class Child extends Parent {
    static void print() {
        super.show(); // Compilation Error
    }
}
```

السبب إن `super` بتتعامل مع الـ Parent Class من خلال الـ Object الحالي، والـ `static` method معندهاش `this` أو `super` بشكل مباشر.

### 5. سؤال إنترفيو سريع

إيه ناتج الكود ده


class Test {
    int v = 5;

    static void print() {
        System.out.println(v);
    }
}
```

A. Prints 5

B. Prints 0

C. Compilation Error

D. NullPointerException

شوف الإجابة واشرح السببو