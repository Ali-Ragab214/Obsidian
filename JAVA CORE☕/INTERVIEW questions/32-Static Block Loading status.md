

الناس بتفتكر إن الـ **Static Block** بيتنفذ أول ما البرنامج يبدأ، وده **مش صحيح**.

الصح هو:

> **الـ Static Block بيتنفذ أول مرة يحصل فيها Class Loading للكلاس.**

يعني لازم الـ JVM تحمل الكلاس الأول.

---

## الحالة الأولى: مفيش أي استخدام للكلاس

```java
class Employee {

    static {
        System.out.println("Static Block");
    }
}

public class Main {

    public static void main(String[] args) {

        System.out.println("Hello");

    }
}
```

### الـ Output

```text
Hello
```

❌ لن ترى:

```text
Static Block
```

ليه؟

لأن الـ JVM عمرها ما احتاجت تحمل كلاس `Employee`.

---

## الحالة الثانية: عملنا Object

```java
Employee emp = new Employee();
```

الترتيب:

```text
Static Block
Constructor
```

---

## الحالة الثالثة: استدعينا Static Method فقط

```java
Employee.print();
```

```java
class Employee {

    static {
        System.out.println("Static Block");
    }

    static void print() {
        System.out.println("Print");
    }
}
```

### الـ Output

```text
Static Block
Print
```

مع إننا **معملناش Object**.

ليه؟

لأن استدعاء أي عضو `static` يتطلب تحميل الكلاس.

---

## الحالة الرابعة: استخدمنا Static Variable

```java
System.out.println(Employee.companyName);
```

هيطبع

```text
Static Block
Google
```

---

## الحالة الخامسة: `Class.forName()`

```java
Class.forName("Employee");
```

حتى بدون إنشاء Object.

النتيجة:

```text
Static Block
```

لأن `Class.forName()` يجبر الـ JVM على تحميل الكلاس وتشغيل الـ Static Initializer.

---

# إذن السؤال:

> **هل لازم أعمل Object علشان الـ Static Block يشتغل؟**

**الإجابة: لا.**

يكفي أي شيء يجعل الـ JVM تحمل الكلاس، مثل:

- ✅ إنشاء Object.
    
- ✅ استدعاء Static Method.
    
- ✅ الوصول إلى Static Variable (في أغلب الحالات).
    
- ✅ `Class.forName()`.
    

أما إذا لم يُستخدم الكلاس إطلاقًا، فلن يتم تحميله، وبالتالي **لن يُنفذ الـ Static Block**.

---

### قاعدة تحفظها للإنترفيو

> **Static Block executes once, when the class is loaded into the JVM—not when an object is created.**

أي:

> **الـ Static Block يُنفذ مرة واحدة عند تحميل الكلاس في الـ JVM، وليس عند إنشاء الـ Object.**