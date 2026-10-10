أيوه يا صاحبي، كلامك صح بنسبة كبيرة جدًا، بس خلينا نضبط نقطة مهمة في Java.

لو عندك `sealed class`، فأي direct subclass ليها لازم يحقق شرطين:

1. يكون محدد بكلمة `final` أو `sealed` أو `non-sealed`.
    
2. يكون مسموح له بالوراثة منها، يعني يكون موجود في `permits`، أو يكون ضمنيًا مسموحًا به لو الـ compiler قدر يحدد الـ subclasses من نفس ملف الـ source.
    

## مثال

```
sealed class Animal permits Dog, Cat, Bird {
}

final class Dog extends Animal {
}

sealed class Cat extends Animal permits Lion {
}

non-sealed class Bird extends Animal {
}

final class Lion extends Cat {
}
```

لاحظ إن:

- `Dog`: استخدمنا `final`، وبالتالي مفيش class تقدر تعمل منه inheritance.
    
- `Cat`: استخدمنا `sealed`، وبالتالي هو كمان بيحدد مين يقدر يرث منه.
    
- `Bird`: استخدمنا `non-sealed`، وبالتالي أي class تقدر تعمل منه inheritance عادي.
    
- `Lion`: لازم هو كمان يحدد وضعه باستخدام `final` أو `sealed` أو `non-sealed`، واستخدمنا `final`.
    

## طب إيه اللي يحصل لو نسيت الكلمات دي؟

```
sealed class Animal permits Dog {
}

class Dog extends Animal { // Compilation Error
}
```

ده خطأ لأن `Dog` لم يحدد وضعه باستخدام واحدة من الكلمات الثلاث.

خد بالك من نقطة الـ permits: لو كتبت `permits` صراحةً، لازم الـ direct subclasses المسموح بها تكون مذكورة فيه. لكن `permits` مش لازم يتكتب دائمًا؛ ممكن الـ compiler يستنتج القائمة ضمنيًا في حالات معينة، مثل وجود الـ subclasses في نفس ملف الـ source.

### إجابة انترفيو جاهزة

> Every direct subclass of a sealed class must explicitly declare itself as `final`, `sealed`, or `non-sealed`. It must also be permitted by the sealed superclass, either explicitly through the `permits` clause or implicitly when the permitted subclasses can be inferred by the compiler.

دي إجابة دقيقة ومناسبة للانترفيو.