ده سؤال مشهور جدًا في مقابلات Java، وبيستخدموه عشان يعرفوا إذا كنت فاهم **Method Overloading** و**طريقة تشغيل JVM**.

---

# Interview Question

> **Can you overload the `main()` method in Java?**

## Short Answer

**Yes, you can overload the `main()` method.**

However,

**The JVM always starts execution from only one specific signature:**

```java
public static void main(String[] args)
```

أي نسخة تانية من `main()` تعتبر مجرد Method عادية، ولن تستدعيها الـ JVM تلقائيًا.

---

# Why?

لما تشغل البرنامج

```bash
java Test
```

الـ JVM تبدأ تبحث داخل الكلاس عن Method بالمواصفات الآتية:

```java
public
static
void
main(String[] args)
```

ولازم تحقق الأربع شروط:

- `public`
    
- `static`
    
- `void`
    
- تستقبل `String[] args`
    

إذا وجدتها يبدأ التنفيذ منها.

إذا لم يجدها سيظهر Runtime Error.

---

# Example 1

```java
public class Test {

    public static void main(String[] args) {
        System.out.println("Original Main");
    }

    public static void main() {
        System.out.println("Overloaded Main");
    }
}
```

### Output

```text
Original Main
```

لماذا؟

لأن الـ JVM وجدت الـ Main القياسية.

ولم تهتم بالنسخة الأخرى.

---

# Example 2

يمكنك استدعاء النسخة الأخرى بنفسك.

```java
public class Test {

    public static void main(String[] args) {

        System.out.println("Main 1");

        main();
    }

    public static void main() {

        System.out.println("Main 2");
    }
}
```

Output

```text
Main 1
Main 2
```

هنا

- الـ JVM بدأت من  
    `main(String[] args)`
    
- ثم أنت استدعيت النسخة الثانية يدويًا.
    

---

# Example 3

يمكنك عمل أكثر من Overload.

```java
public class Test {

    public static void main(String[] args) {
        main(10);
        main("Ali");
    }

    public static void main(int x) {
        System.out.println(x);
    }

    public static void main(String name) {
        System.out.println(name);
    }
}
```

Output

```text
10
Ali
```

---

# What if the original main doesn't exist?

مثلاً

```java
public class Test {

    public static void main() {

    }
}
```

ثم

```bash
java Test
```

ستظهر رسالة مشابهة لـ

```text
Error: Main method not found in class Test
```

لأن الـ JVM لم تجد

```java
public static void main(String[] args)
```

---

# كيف تبحث الـ JVM عن main؟

```text
java Test

        │
        ▼
Load Test.class

        │
        ▼
Search for

public static void main(String[] args)

        │
   ┌────┴─────┐
   │          │
Found      Not Found
   │          │
   ▼          ▼
Execute    Runtime Error
```

---

# Why must it be static?

لأن الـ JVM لا تنشئ Object من الكلاس عند بداية التشغيل.

فهي تحتاج Method يمكن استدعاؤها مباشرة.

```text
JVM

↓

Test.main(args)

✓ static
```

أما لو كانت

```java
public void main(String[] args)
```

فكانت ستحتاج

```java
new Test().main(args);
```

لكن من سيقوم بإنشاء الـ Object؟

لهذا يجب أن تكون `static`.

---

# Why must it be public?

لأن الـ JVM خارج الكلاس.

فتحتاج الوصول إليها.

لو كانت

```java
private static void main(String[] args)
```

لن تستطيع الـ JVM الوصول إليها.

وسيظهر Runtime Error.

---

# Why void?

لأن الـ JVM لا تتوقع أي قيمة راجعة من نقطة بداية البرنامج.

---

# Why String[] args?

لأن أي Arguments تكتبها أثناء تشغيل البرنامج

```bash
java Test Ali 22 Egypt
```

ستصل إلى البرنامج في صورة Array.

```java
public static void main(String[] args) {

    System.out.println(args[0]); // Ali
    System.out.println(args[1]); // 22
    System.out.println(args[2]); // Egypt
}
```

---

# Overloading vs Overriding

|Overloading|Overriding|
|---|---|
|Same class|Parent & Child|
|Different parameters|Same parameters|
|Compile-time polymorphism|Runtime polymorphism|
|Allowed with `main()`|Not applicable because `main()` is `static`|

---

# Can we override main()?

تقنيًا يمكنك كتابة `main()` بنفس التوقيع في الكلاس الابن، لكن هذا **ليس Overriding** لأن `main()` هي `static`.

في Java، **الـ static methods لا تُعمل Override، وإنما تُعمل Method Hiding**.

مثال:

```java
class Parent {

    public static void main(String[] args) {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    public static void main(String[] args) {
        System.out.println("Child");
    }
}
```

إذا شغلت:

```bash
java Child
```

النتيجة

```text
Child
```

وإذا شغلت:

```bash
java Parent
```

النتيجة

```text
Parent
```

كل Class له نقطة بداية خاصة به.

---

# Interview Answer (30 seconds)

> **Yes, the `main()` method can be overloaded because it's just like any other method. However, the JVM starts execution only from the standard entry point `public static void main(String[] args)`. Any overloaded versions of `main()` are treated as normal methods and will only execute if they are explicitly called from the standard `main()` method. If the standard signature is missing, the JVM throws a runtime error saying that the main method was not found.**


# jvm looks for standard main method only 
