ده من **أشهر أسئلة الـ Java Interview** لأنه بيختبر فهمك لـ:

- Inheritance
    
- Polymorphism
    
- Variable Hiding
    
- Method Overriding
    
- Compile-time vs Runtime Polymorphism
    

خلينا نشرحه من الصفر.

---

# أولاً: الكود

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {
    int speedLimit = 150;

    public static void main(String[] args) {

        Bike obj = new Honda3();

        System.out.println(obj.speedLimit);

    }
}
```

---

# السؤال الأول: ما هو الناتج؟

الإجابة هي:

```text
90
```

وليس

```text
150
```

وده اللي بيخلي ناس كتير تغلط.

---

# خلينا نمشي سطر سطر

## الخطوة الأولى

```java
class Bike {
    int speedLimit = 90;
}
```

عملنا Class اسمه Bike.

عنده Variable اسمه

```java
speedLimit
```

قيمته

```text
90
```

---

## الخطوة الثانية

```java
class Honda3 extends Bike {
    int speedLimit = 150;
}
```

هنا Honda3 ورثت من Bike.

لكن عملت Variable جديد بنفس الاسم.

لاحظ:

هي **لم تعدل القديم**

هي عملت واحد جديد.

يعني أصبح عندنا:

```
Bike
-------
speedLimit = 90
```

و

```
Honda3
-------
speedLimit =150
```

كل Class عنده Variable باسمه.

---

# طيب لما عملنا

```java
Bike obj = new Honda3();
```

الجملة دي محتاجة تتفهم كويس.

تنقسم لحاجتين.

## Reference

```java
Bike obj
```

يعني المتغير نوعه

```
Bike
```

---

## Object

```java
new Honda3()
```

يعني الـ Object الحقيقي

```
Honda3
```

إذن

```
Reference
    Bike

       │

       ▼

Honda3 Object
```

وده اسمه

# Upcasting

يعني

Subclass

↓

Superclass

وده أشهر استخدام للـ Polymorphism.

---

# السؤال

لما أكتب

```java
obj.speedLimit
```

Java هتبص على مين؟

الجواب:

على نوع الـ Reference

وليس الـ Object.

لأن

**Variables لا تعمل Runtime Polymorphism**

هي تعمل

## Compile Time Binding

يعني أثناء الـ Compile.

الـ Compiler شايف إن

```java
obj
```

نوعها

```java
Bike
```

فيقول

```
هات speedLimit الموجودة داخل Bike
```

إذن

```
90
```

---

# لماذا لا أخذ 150؟

لأن

الـ Variable

لا يتم Override.

هي فقط

Hidden.

وده الفرق الكبير.

---

# Variable Hiding

لو الأب عنده

```java
int speed = 90;
```

والابن كتب

```java
int speed = 150;
```

فهو لم يعمل Override.

هو أخفى Variable الأب.

وده اسمه

## Variable Hiding

وليس

Method Overriding.

---

# ماذا يحدث داخل الذاكرة؟

الـ Object الحقيقي

```
Honda3
```

بيحتوي على

```
Bike Part

speedLimit =90
```

وأيضاً

```
Honda Part

speedLimit =150
```

يعني الـ Object نفسه فيه الاثنين.

```
Honda3 Object

-------------------

Bike.speedLimit =90

Honda.speedLimit=150
```

لكن

الـ Reference تحدد أي واحد يتم الوصول إليه.

---

# لو غيرنا الـ Reference

```java
Honda3 obj = new Honda3();

System.out.println(obj.speedLimit);
```

النتيجة

```
150
```

لأن الـ Reference أصبحت

Honda3.

---

# أهم نقطة

## Variables لا تعمل Overriding

لكن

## Methods تعمل Overriding.

وده فرق مهم جداً.

---

# مثال على الـ Methods

الأب

```java
class Bike {

    void speed() {

        System.out.println("90");

    }

}
```

الابن

```java
class Honda3 extends Bike {

    void speed() {

        System.out.println("150");

    }

}
```

ثم

```java
Bike obj = new Honda3();

obj.speed();
```

الناتج

```
150
```

رغم إن الـ Reference

Bike.

---

# لماذا هنا طبع 150؟

لأن

Methods

تعمل

## Runtime Polymorphism

يعني JVM تبص على الـ Object الحقيقي.

```
Reference

Bike

      │

      ▼

Honda3 Object
```

فتشوف

```
Honda3
```

وتنفذ

```
Honda3.speed()
```

---

# إذن عندنا قانونين

## بالنسبة للـ Variables

Java تنظر إلى

```
Reference Type
```

وليس الـ Object.

يعني

```java
Bike obj = new Honda3();

System.out.println(obj.speedLimit);
```

↓

```
90
```

---

## بالنسبة للـ Methods

Java تنظر إلى

```
Actual Object
```

يعني

```java
Bike obj = new Honda3();

obj.speed();
```

↓

```
150
```

---

# لماذا فعلت Java ذلك؟

لأن الـ Variable ليس لها سلوك (Behavior)، هي مجرد بيانات، لذلك يمكن للـ Compiler تحديدها أثناء الترجمة (**Compile Time Binding**).

أما الـ Method فهي تمثل سلوك الكائن (Behavior)، وجوهر الـ Polymorphism هو أن السلوك يختلف حسب نوع الكائن الحقيقي، لذلك يتم اختيارها أثناء التشغيل (**Runtime Binding**).

---

# مقارنة تحفظها للإنترفيو

|المقارنة|Variables|Methods|
|---|---|---|
|هل يمكن Override؟|❌ لا|✅ نعم|
|اسم المفهوم|Variable Hiding|Method Overriding|
|يعتمد على|Reference Type|Actual Object|
|وقت الاختيار|Compile Time|Runtime|
|نوع الـ Polymorphism|Static Binding|Dynamic Binding|

---

# سؤال إنترفيو مشهور جدًا

ما ناتج الكود التالي؟

```java
class A {
    int x = 10;

    void show() {
        System.out.println("A");
    }
}

class B extends A {
    int x = 20;

    @Override
    void show() {
        System.out.println("B");
    }
}

public class Test {
    public static void main(String[] args) {

        A obj = new B();

        System.out.println(obj.x);

        obj.show();

    }
}
```

الإجابة:

```
10
B
```

**السبب:**

- `obj.x` → يعتمد على **نوع الـ Reference (`A`)**، فيطبع `10`.
    
- `obj.show()` → يعتمد على **نوع الـ Object الحقيقي (`B`)**، فيطبع `B`.
    

دي واحدة من أهم أسئلة الـ OOP في Java، ولو فهمت الفرق بين **Variable Hiding** و **Method Overriding** هتكون فهمت جزءًا كبيرًا من الـ Polymorphism في اللغة.