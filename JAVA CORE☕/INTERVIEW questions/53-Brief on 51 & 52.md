هنا بقى فيه نقطة دقيقة جدًا، والإجابة هي: **لا، مش دايمًا.**

اللي بيحدد النتيجة هو **نوع الـ Reference**، مش مجرد وجود Variable Hiding.

## الحالة الأولى: Reference من نوع الـ Parent

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {
    int speedLimit = 150;
}

public class Main {
    public static void main(String[] args) {
        Bike obj = new Honda3();

        System.out.println(obj.speedLimit);
    }
}
```

الناتج:

```text
90
```

ليه؟

لأن الـ Reference نوعه `Bike`.

---

## الحالة الثانية: Reference من نوع الـ Child

```java
Honda3 obj = new Honda3();

System.out.println(obj.speedLimit);
```

الناتج:

```text
150
```

ليه؟

لأن الـ Reference نوعه `Honda3`.

---

## الحالة الثالثة: الوصول للـ Parent من داخل الـ Child

```java
class Bike {
    int speedLimit = 90;
}

class Honda3 extends Bike {

    int speedLimit = 150;

    void print() {
        System.out.println(speedLimit);
        System.out.println(super.speedLimit);
    }
}
```

الناتج:

```text
150
90
```

هنا استخدمنا `super` للوصول للمتغير الموجود في الـ Parent.

---

# القاعدة الذهبية

بالنسبة للـ **Variables**:

> **Java تنظر إلى نوع الـ Reference، وليس إلى نوع الـ Object.**

يعني:

```java
Bike obj = new Honda3();
```

أي وصول لمتغير هيكون كأنك بتقول:

```java
Bike.speedLimit
```

لكن:

```java
Honda3 obj = new Honda3();
```

أي وصول لمتغير هيكون:

```java
Honda3.speedLimit
```

---

# احفظ الجدول ده للإنترفيو

|الكود|الناتج|السبب|
|---|---|---|
|`Bike obj = new Honda3(); obj.speedLimit;`|`90`|Reference = `Bike`|
|`Honda3 obj = new Honda3(); obj.speedLimit;`|`150`|Reference = `Honda3`|
|`super.speedLimit` داخل `Honda3`|`90`|الوصول مباشرة لمتغير الـ Parent|

---

### سؤال إنترفيو ممكن ييجي بعده مباشرة

لو غيرنا `speedLimit` من **Variable** إلى **Method**:

```java
class Bike {
    void print() {
        System.out.println("90");
    }
}

class Honda3 extends Bike {
    @Override
    void print() {
        System.out.println("150");
    }
}

Bike obj = new Honda3();
obj.print();
```

**هل هتطبع 90 ولا 150؟ ولماذا؟**

لو قدرت تجاوب على السؤال ده، يبقى فهمت الفرق الحقيقي بين **Variable Hiding** و **Method Overriding**.