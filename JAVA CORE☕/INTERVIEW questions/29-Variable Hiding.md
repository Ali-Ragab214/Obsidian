الصورة دي من أشهر الأمثلة اللي بتلخبط الناس في Java، لأنها بتبين الفرق بين **Variable Hiding** و **Method Overriding**.

الكود:

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

# أولًا: إيه اللي بيحصل هنا؟

عندنا:

```java
class Bike {
    int speedLimit = 90;
}
```

ودي الـ Parent class.

وعندها Instance Variable اسمها:

```java
speedLimit = 90
```

---

بعد كده:

```java
class Honda3 extends Bike {
    int speedLimit = 150;
}
```

هنا الابن عمل Variable بنفس الاسم.

هل ده Override؟

**لا.**

ده اسمه **Variable Hiding**.

---

# يعني إيه Variable Hiding؟

يعني الـ Child يعرف Variable بنفس اسم Variable موجودة في الـ Parent.

مثال:

```java
class Parent {
    int x = 10;
}

class Child extends Parent {
    int x = 20;
}
```

هنا فيه متغيرين مختلفين:

```
Parent.x = 10

Child.x = 20
```

الـ Child مخفاش القديم.

هو فقط أنشأ Variable جديدة.

---

# هل Java بتدعم Variable Overriding؟

**الإجابة: لا.**

Java لا تدعم Variable Overriding.

هي تدعم فقط:

- Method Overriding ✅
    
- Variable Hiding ✅
    

---

# ليه Java مبتدعمش Variable Overriding؟

لأن الـ Variables بيتم تحديدها أثناء الـ **Compile Time**. =>مهممممممم

أما الـ Methods بيتم تحديدها أثناء الـ **Run Time**.=>مهمممممممم

وده الفرق الأساسي.

---

# نرجع للكود

```java
Bike obj = new Honda3();
```

لاحظ:

```
Reference Type = Bike

Object Type = Honda3
```

ودي نقطة مهمة جدًا.

يعني:

```
Reference -----> Bike

Object --------> Honda3
```

---

لما تكتب:

```java
obj.speedLimit
```

Java هتبص على إيه؟

هتبص على:

**Reference Type**

مش الـ Object.

يعني هتروح لـ:

```
Bike
```

وتجيب:

```java
90
```

---

# إذًا الـ Output

```text
90
```

---

# ليه؟

لأن الـ Variables لا تستخدم Runtime Polymorphism.

لكن تستخدم Compile Time Binding.

---

# طب لو كانت Method؟

لو كتبنا:

```java
class Bike {

    void print() {
        System.out.println("Bike");
    }
}

class Honda3 extends Bike {

    @Override
    void print() {
        System.out.println("Honda");
    }

    public static void main(String[] args) {

        Bike obj = new Honda3();

        obj.print();
    }
}
```

الـ Output هيكون:

```
Honda
```

ليه؟

لأن الـ Methods تستخدم:

**Dynamic Dispatch**

أو

**Runtime Polymorphism**

---

# مقارنة مهمة جدًا

## Variables

```java
Bike obj = new Honda3();

System.out.println(obj.speedLimit);
```

Java تنظر إلى:

```
Reference Type

Bike
```

Output

```
90
```

---

## Methods

```java
Bike obj = new Honda3();

obj.print();
```

Java تنظر إلى:

```
Actual Object

Honda3
```

Output

```
Honda
```

---

# الفرق بين Variable Hiding و Method Overriding

|Variable Hiding|Method Overriding|
|---|---|
|يحدث مع Variables|يحدث مع Methods|
|Compile Time|Runtime|
|يعتمد على Reference Type|يعتمد على Object Type|
|لا يوجد Polymorphism|يوجد Runtime Polymorphism|
|لا يمكن استخدام `@Override`|يجب استخدام `@Override` (مستحب)|

---

# شكل الذاكرة

بعد تنفيذ:

```java
Bike obj = new Honda3();
```

هيكون عندك:

```
Reference

obj
 │
 │
 ▼

Honda3 Object
------------------------
Bike.speedLimit = 90
Honda3.speedLimit = 150
------------------------
```

الاتنين موجودين داخل الـ Object.

لكن لما تكتب:

```java
obj.speedLimit
```

Java تقول:

```
Reference Type = Bike

يبقى هات Bike.speedLimit
```

فتطبع:

```
90
```

ولو عملت:

```java
Honda3 obj = new Honda3();

System.out.println(obj.speedLimit);
```

هيكون الناتج:

```
150
```

لأن الـ Reference Type هنا أصبح `Honda3`.

---

## قاعدة ذهبية في مقابلات Java

احفظ الجملة دي لأنها من أكثر الأسئلة شيوعًا:

- **Fields (Variables) are hidden, not overridden.**
    
- **Methods are overridden and resolved at runtime using Dynamic Dispatch.**
    

دي من أساسيات الـ OOP في Java، وغالبًا هتتقابل مع السؤال ده في أي مقابلة Junior أو Mid-level.