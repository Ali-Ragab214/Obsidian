بص يا صاحبي، السؤال ده مهم جدًا في Java Interviews، لأنه بيختبر فهمك للفرق بين الـ `final fields` والـ `default values`.

## 1. نفهم الكود الأول

```
class User {
    final int id;

    User() {
    }
}
```

عندنا `instance field` اسمه `id`، وعليه كلمة `final`.

معنى `final` هنا إنك لازم تدي المتغير قيمة مرة واحدة، ومينفعش تغيّرها بعد كده.

المشكلة إن الـ constructor خلص من غير ما يدي `id` أي قيمة، وبالتالي الكود ده بيعمل Compilation Error.

```
final int id; // لم تتم تهيئته

User() {
    // id لم يأخذ قيمة
}
```

## 2. طيب، إيه حوار الـ Default Values؟

في Java، الـ default value بتختلف حسب نوع المتغير، وكمان حسب مكان تعريفه.

### أولًا: Instance Variables

دي المتغيرات اللي بتكون جوه الـ class وبره الـ methods والـ constructors.

|Data type|Default value|
|---|---|
|`byte`, `short`, `int`, `long`|`0`|
|`float`, `double`|`0.0`|
|`char`|`'\u0000'`|
|`boolean`|`false`|
|Reference types زي `String` و`User`|`null`|

مثال:

```
class User {
    int age;
    boolean active;
    String name;
}

User user = new User();

System.out.println(user.age);    // 0
System.out.println(user.active); // false
System.out.println(user.name);   // null
```

ليه ده اشتغل؟ لأن Java بتدي الـ instance fields قيمًا افتراضية تلقائيًا.

لكن خد بالك: وجود default value مش معناه إنك تقدر تسيب كل field من غير initialization.

### ثانيًا: `final` Instance Variables

هنا القاعدة مختلفة.

```
class User {
    final int id;
}
```

ده Compilation Error، لأن الـ `final` instance field لازم يكون اتعيّن بشكل مؤكد قبل انتهاء كل constructor.

يعني لازم تعمل واحد من دول:

الطريقة الأولى: عند تعريف المتغير (Field Initialization)

```
class User {
    final int id = 10;
}
```

الطريقة الثانية: داخل الـ constructor

```
class User {
    final int id;

    User() {
        id = 10;
    }
}
```

الطريقة الثالثة: باستخدام Initializer Block

```
class User {
    final int id;

    {
        id = 10;
    }
}
```

التلاتة صح.

## 3. طب لو عندي أكتر من Constructor؟

```
class User {
    final int id;

    User(int id) {
        this.id = id;
    }

    User() {
        // Compilation Error
    }
}
```

ليه؟

لأن كل constructor لازم يضمن إن `id` اتعيّن. الـ constructor الأول عيّنه، لكن التاني لأ.

تقدر تصلح ده مثلًا باستخدام constructor chaining:

```
class User {
    final int id;

    User(int id) {
        this.id = id;
    }

    User() {
        this(0);
    }
}
```

كده الاتنين صح؛ لأن الـ constructor التاني بيستدعي الأول، والأول بيعيّن `id`.

## 4. طب هل الـ Local Variables ليها Default Values؟

لأ! ودي نقطة انترفيو مهمة جدًا.

```
class Test {
    void show() {
        int x;
        System.out.println(x); // Compilation Error
    }
}
```

مع إن `x` نوعه `int`، فهو هنا Local Variable، مش `Instance Variable`.

الـ local variables مش بتاخد default values تلقائيًا، ولازم تديها قيمة قبل ما تقرأها.

## الخلاصة اللي تحفظها للانترفيو

Normal instance field

بياخد default value تلقائيًا.

`final` instance field

لازم يتعيّن بشكل مؤكد عند التعريف، أو في initializer، أو في كل constructor.

Local variable

مفيش default value تلقائي؛ لازم تهيّئه قبل استخدامه.

سؤال سريع ليك: إيه ناتج الكود ده؟

```
class User {
    final int id = 10;

    User() {
        id = 20;
    }
}
```

يطبع `20`

يطبع `10`

Compilation Error

تأكيد الإجابة