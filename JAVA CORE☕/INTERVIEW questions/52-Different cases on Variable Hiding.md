سؤال ممتاز، والإجابة هتوضحلك الفرق بين **Variable Hiding** و**Inheritance**.

لو الاسم **مختلف حتى بحرف واحد**، فده **مش Variable Hiding أصلاً**.

مثلاً:

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {
    int speedLimits = 150; // لاحظ الـ s
}
```

هنا عندك متغيرين مختلفين تمامًا:

- `speedLimit`
    
- `speedLimits`
    

Java تعتبرهم متغيرين مستقلين.

---

# مثال 1

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {
    int speedLimits = 150;

    public static void main(String[] args) {
        Honda3 obj = new Honda3();

        System.out.println(obj.speedLimit);
        System.out.println(obj.speedLimits);
    }
}
```

الناتج:

```text
90
150
```

ليه؟

لأن `Honda3` ورثت `speedLimit` من الأب، وأضافت متغيرًا جديدًا اسمه `speedLimits`.

---

# مثال 2: Upcasting

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {
    int speedLimits = 150;

    public static void main(String[] args) {

        Bike obj = new Honda3();

        System.out.println(obj.speedLimit);

        // System.out.println(obj.speedLimits); ❌ Compile Error
    }
}
```

الناتج:

```text
90
```

والسطر الثاني هيطلع **Compile Error**.

---

# ليه Error؟

لأن نوع الـ Reference هو:

```java
Bike obj
```

فالـ Compiler بيبص على Class `Bike` فقط، ويدور على:

```java
speedLimits
```

مش هيلاقيها.

يبقى يقول:

> **Cannot resolve symbol 'speedLimits'**

رغم إن الـ Object الحقيقي `Honda3` فيه المتغير ده.

---

# لو عملنا Downcasting؟

```java
Bike obj = new Honda3();

Honda3 h = (Honda3) obj;

System.out.println(h.speedLimit);
System.out.println(h.speedLimits);
```

الناتج:

```text
90
150
```

لأن بعد الـ Downcasting، الـ Reference بقت من نوع `Honda3`، فأصبح يقدر يشوف:

- المتغير الموروث `speedLimit`.
    
- والمتغير الجديد `speedLimits`.
    

---

# القاعدة المهمة

لو الاسم **نفسه بالضبط**:

```java
class A {
    int x = 10;
}

class B extends A {
    int x = 20;
}
```

ده اسمه **Variable Hiding**.

أما لو الاسم مختلف:

```java
class A {
    int x = 10;
}

class B extends A {
    int y = 20;
}
```

فده **مش Hiding**، ده مجرد **متغير جديد** في الـ Child Class.

---

## قاعدة تحفظها للإنترفيو

- **نفس الاسم** → Variable Hiding.
    
- **اسم مختلف** → New Variable.
    
- الـ Child دائمًا يرث متغيرات الـ Parent (طالما ليست `private`)، ويمكنه أيضًا إضافة متغيرات جديدة خاصة به.