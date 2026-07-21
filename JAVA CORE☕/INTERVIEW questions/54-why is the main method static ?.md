ده من أشهر أسئلة الـ Java Interview، والإجابة اللي بيسمعها الإنترفيوور غالبًا هي:

> **The `main()` method is static because the JVM needs to call it without creating an object of the class.**

لكن تعال نفهم **ليه** بالتفصيل.

---

# أولًا: ماذا يحدث عند تشغيل برنامج Java؟

لما تكتب:

```bash
java Main
```

الـ JVM تعمل الآتي:

1. تحمل الـ `Main.class` إلى الذاكرة.
    
2. تبحث عن:
    

```java
public static void main(String[] args)
```

3. تستدعيها مباشرة.
    

السؤال هنا:

**هل الـ JVM عندها Object من `Main`؟**

الإجابة:

**لا.**

---

# ماذا لو كانت `main()` ليست static؟

افترض أنها كده:

```java
class Main {

    public void main(String[] args) {

        System.out.println("Hello");

    }

}
```

لكي تستدعيها الـ JVM، لازم تعمل:

```java
Main obj = new Main();

obj.main(args);
```

لكن هنا تظهر مشكلة.

---

# المشكلة الأساسية (Chicken and Egg Problem)

لكي تنشئ Object:

```java
new Main();
```

قد يحتاج الـ Constructor إلى بيانات أو قد ينفذ منطقًا معينًا.

السؤال:

**من الذي سينشئ هذا الـ Object؟**

لو احتاجت الـ JVM إلى إنشاء Object قبل تشغيل `main()`، ستكون مضطرة لاتخاذ قرارات تخص برنامجك (أي Constructor تستخدم؟ ماذا لو كان خاصًا؟ ماذا لو احتاج Arguments؟).

بدلًا من ذلك، Java جعلت نقطة البداية (`main`) مستقلة عن أي Object.

---

# ماذا تعني static؟

أي عضو `static` ينتمي إلى **الـ Class نفسه** وليس إلى أي Object.

يعني يمكن استدعاؤه هكذا:

```java
Main.main(args);
```

بدون:

```java
Main obj = new Main();
```

---

# مقارنة

### Static Method

```java
class Test {

    static void hello() {

        System.out.println("Hello");

    }

}
```

تستدعيها:

```java
Test.hello();
```

لا تحتاج Object.

---

### Non-static Method

```java
class Test {

    void hello() {

        System.out.println("Hello");

    }

}
```

لازم:

```java
Test obj = new Test();

obj.hello();
```

---

# ماذا تفعل الـ JVM؟

هي تعمل تقريبًا:

```java
Main.main(args);
```

وليس:

```java
Main obj = new Main();

obj.main(args);
```

ولهذا يجب أن تكون `main()` **static**.

---

# سؤال إنترفيو مشهور

**هل يمكن للـ `main()` أن تنشئ Objects؟**

طبعًا.

```java
class Main {

    public static void main(String[] args) {

        Main obj = new Main();

        obj.print();

    }

    void print() {
        System.out.println("Hello");
    }
}
```

هنا الـ JVM استدعت `main()` لأنها `static`، وبعد ذلك `main()` نفسها أنشأت Object واستدعت الـ Methods غير الـ `static`.

---

# لماذا لا تكون الـ Constructor هي نقطة البداية؟

لأن الـ Constructor لا يمكن استدعاؤه إلا بعد إنشاء Object:

```java
new Main();
```

لكن لإنشاء الـ Object، لازم فيه كود يبدأ التنفيذ أولًا.

لذلك Java احتاجت نقطة بداية يمكن استدعاؤها **بدون أي Object**، فكانت `static main()`.

---

# إجابة إنترفيو احترافية

> The `main()` method is `static` because it is the entry point of a Java application. When the JVM starts a program, no object of the class exists yet. Since static methods belong to the class rather than an instance, the JVM can invoke `main()` directly without creating an object. If `main()` were non-static, the JVM would first need to create an object just to call it, which would complicate the startup process.

---

## سؤال متابعة بيجي بعدها كثير

إذا كانت `main()` **static**، **لماذا لا يمكنها استدعاء non-static methods مباشرة؟**

وده سؤال مهم جدًا في مقابلات Java لأنه بيختبر فهمك للعلاقة بين `static` والـ Objects.