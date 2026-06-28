# Java 8 Interface: Default Methods vs Static Methods

## Before Java 8

قبل Java 8 كان الـ Interface يحتوي فقط على:

- Abstract Methods
    
- Constants (`public static final`)
    

مثال:

```java
public interface Animal {

    void makeSound();

}
```

وكان أي Class يعمل `implements` للـ Interface **لازم** يعمل Override لكل الـ Abstract Methods.

```java
public class Dog implements Animal {

    @Override
    public void makeSound() {
        System.out.println("Woof");
    }
}
```

---

## المشكلة التي كانت موجودة قبل Java 8

افترض أن لديك Interface مستخدم في آلاف الكلاسات:

```java
public interface Animal {

    void makeSound();

}
```

وبعد سنة أردت إضافة Method جديدة:

```java
void sleep();
```

سيصبح:

```java
public interface Animal {

    void makeSound();

    void sleep();

}
```

النتيجة:

```text
Compilation Error
```

لأن جميع الكلاسات التي تطبق الـ Interface أصبحت مطالبة بتنفيذ `sleep()`.

إذا كان لديك:

```text
1000 classes
```

فسيتوجب تعديلهم جميعاً.

---

# Default Methods

تم إضافتها في Java 8 لحل مشكلة كسر التوافق (Backward Compatibility).

مثال:

```java
public interface Animal {

    void makeSound();

    default void sleep() {
        System.out.println("Sleeping...");
    }

}
```

الآن:

```java
public class Dog implements Animal {

    @Override
    public void makeSound() {
        System.out.println("Woof");
    }

}
```

الكود يعمل بدون مشاكل رغم أن `Dog` لم يقم بعمل Override لـ `sleep()`.

---

## ماذا يحدث عند Implement Interface يحتوي Default Method ؟

عندما يقوم Class بتنفيذ Interface يحتوي Default Method:

```java
public class Dog implements Animal {

}
```

فإن الـ Class يرث الـ Default Method تلقائياً.

يمكن استدعاؤها مباشرة:

```java
Dog dog = new Dog();

dog.sleep();
```

النتيجة:

```text
Sleeping...
```

---

## هل يمكن عمل Override للـ Default Method ؟

نعم.

```java
public class Dog implements Animal {

    @Override
    public void makeSound() {
        System.out.println("Woof");
    }

    @Override
    public void sleep() {
        System.out.println("Dog is sleeping");
    }

}
```

النتيجة:

```text
Dog is sleeping
```

---

# Static Methods

تم إضافتها أيضاً في Java 8.

مثال:

```java
public interface Animal {

    static void printInfo() {
        System.out.println("Animals are living creatures.");
    }

}
```

---

## كيف يتم استدعاؤها ؟

يتم استدعاؤها باستخدام اسم الـ Interface نفسه:

```java
Animal.printInfo();
```

---

## هل يرثها الـ Class الذي يقوم بالـ Implement ؟

لا.

```java
public class Dog implements Animal {

}
```

لا يمكنك كتابة:

```java
Dog.printInfo();
```

ولا:

```java
Dog dog = new Dog();
dog.printInfo();
```

سيحدث:

```text
Compilation Error
```

لأن الـ Static Method ملك للـ Interface وليس للكلاس الذي يطبقه.

---

# هل يمكن عمل Override للـ Static Method ؟

لا.

لأن الـ Static Methods لا تشارك في الـ Polymorphism.

---

# مقارنة بين Default و Static Methods

|Feature|Default Method|Static Method|
|---|---|---|
|Added in Java 8|Yes|Yes|
|Has implementation|Yes|Yes|
|Inherited by implementing classes|Yes|No|
|Can be overridden|Yes|No|
|Called using object|Yes|No|
|Called using interface name|No|Yes|
|Supports polymorphism|Yes|No|

---

# Example

```java
public interface Vehicle {

    default void start() {
        System.out.println("Vehicle started");
    }

    static void company() {
        System.out.println("Toyota");
    }

}
```

```java
public class Car implements Vehicle {

}
```

استخدام الـ Default Method:

```java
Car car = new Car();

car.start();
```

Output:

```text
Vehicle started
```

---

استخدام الـ Static Method:

```java
Vehicle.company();
```

Output:

```text
Toyota
```

---

هذا غير صحيح:

```java
Car.company();
```

أو:

```java
Car car = new Car();
car.company();
```

لأن الـ Static Method ليست جزءاً من الكلاس الذي قام بالـ Implement.

---

# لماذا أضافت Java الـ Static Methods داخل Interfaces ؟

لتجميع Utility Methods المتعلقة بالـ Interface نفسه.

مثال:

```java
List.of(...)
Map.of(...)
Comparator.comparing(...)
Predicate.not(...)
```

كلها Static Methods مرتبطة بالمفهوم الخاص بالـ Interface أو الـ API بدلاً من وضعها في Utility Classes منفصلة.

---

# Interview Answer (Short Version)

### What is the difference between default and static methods in Java 8 interfaces?

- Default methods are inherited by implementing classes and can be overridden.
    
- Static methods belong to the interface itself, are not inherited, and cannot be overridden.
    
- Default methods support polymorphism while static methods do not.
    

### What existed before Java 8?

Before Java 8, interfaces could only contain:

- Abstract methods
    
- Constants (`public static final`)
    

Interfaces could not contain method implementations before Java 8.

```java 
public interface Vehicle
{
 void wheels();
 
 default engine()
 {
 System.out.println("car can has diffrent engines");
 }
 
 static void color()
 {
  System.out.println("car can has diffrent engines");
 }
}


public class Ferrari implements Vehicle
{
@Override
void wheels()
{
 System.out.println("Ferrari has four wheels");
}
@Override
public void engine()
{
 System.out.println("this is from FERRARI");
}
//we can override engine bu we cant override color 
}
```