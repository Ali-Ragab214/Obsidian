# 1) Why can't Static Methods be Overridden?

أولًا لازم نفهم:

## ما هو Method Overriding ؟

الـ Overriding يعني إن الابن يكتب نسخة جديدة من Method موجودة في الأب بنفس:

- الاسم
    
- البراميترز
    
- الـ return type (أو Covariant Return Type)
    

مثال:

```java
class Animal {
    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

---

## لماذا يعمل الـ Overriding هنا؟

لأن الـ JVM وقت التشغيل تستخدم:

```text
Dynamic Dispatch
```

يعني:

```java
Animal animal = new Dog();

animal.sound();
```

الـ JVM ستنظر لنوع الـ Object الحقيقي:

```text
Dog
```

وليس نوع الـ Reference:

```text
Animal
```

لذلك تطبع:

```text
Bark
```

---

# لكن ماذا عن static methods؟

```java
class Animal {

    static void print() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {

    static void print() {
        System.out.println("Dog");
    }
}
```

---

الآن:

```java
Animal animal = new Dog();

animal.print();
```

الناتج:

```text
Animal
```

وليس:

```text
Dog
```

---

## لماذا؟

لأن الـ Static Methods تنتمي إلى:

```text
Class
```

وليس إلى:

```text
Object
```

---

يعني:

```java
Animal.print();
Dog.print();
```

وليس:

```java
animal.print();
```

---

## لذلك ما يحدث ليس Overriding

بل يسمى:

```text
Method Hiding
```

وليس:

```text
Method Overriding
```

---

# Memory Diagram

```text
Animal class
    |
    ---> static print()

Dog class
    |
    ---> static print()
```

لا يوجد Runtime Polymorphism هنا.

---

# Rule

```text
Static methods are resolved at compile time.
```

بينما:

```text
Instance methods are resolved at runtime.
```

---

# 2) Why Constructors Cannot Be Overridden

## Constructor Job

الـ Constructor وظيفته:

```text
Create and initialize object
```

---

مثال:

```java
class Animal {

    Animal() {
        System.out.println("Animal Constructor");
    }
}
```

---

عندما تكتب:

```java
Dog dog = new Dog();
```

يحدث:

```text
Allocate Memory
↓
Call Parent Constructor
↓
Call Child Constructor
```

---

لو كان الـ Constructor قابل للـ Overriding فسيظهر سؤال:

> كيف يمكن للـ JVM أن تعرف أي Constructor ستستخدم قبل إنشاء الـ Object أصلاً؟

الـ Object لم يُنشأ بعد.

إذن:

```text
No Object
↓
No Runtime Polymorphism
↓
No Overriding
```

---

## لذلك Constructors ليست Members أصلاً

الـ Members هم:

- Fields
    
- Methods
    

أما Constructor:

```text
Special Initialization Block
```

---

# 3) Why Private Methods Cannot Be Overridden

```java
class Animal {

    private void print() {
        System.out.println("Animal");
    }
}
```

---

الـ Child لا يرى الـ Method أساسًا.

```java
class Dog extends Animal {

    private void print() {
        System.out.println("Dog");
    }
}
```

هذه ليست Override.

بل Method جديدة تمامًا.

---

## لماذا؟

لأن:

```text
Private methods are not inherited.
```

---

بدون Inheritance لا يوجد Overriding.

---

## مثال

```java
class Parent {

    private void hello() {}
}

class Child extends Parent {

    private void hello() {}
}
```

هنا يوجد:

```text
2 independent methods
```

---

# Summary

|Method Type|Can Override?|Reason|
|---|---|---|
|Instance Method|Yes|Runtime Polymorphism|
|Static Method|No|Compile Time Binding|
|Constructor|No|Object not created yet|
|Private Method|No|Not inherited|

---

# 4) Type Casting in Java

الـ Type Casting يعني:

```text
Converting one type to another type
```

---

# Implicit Casting (Widening)

يتم تلقائيًا.

```java
int x = 10;

double y = x;
```

---

Memory:

```text
int
32 bits
↓
double
64 bits
```

لا يوجد فقد للبيانات.

---

مثال:

```java
int age = 25;

double salary = age;

System.out.println(salary);
```

الناتج:

```text
25.0
```

---

# Explicit Casting (Narrowing)

العكس.

```java
double x = 10.7;

int y = (int) x;
```

الناتج:

```text
10
```

تم فقد:

```text
0.7
```

---

# Memory Diagram

```text
double
64 bits
↓
int
32 bits
```

---

# Object Type Casting

## Upcasting

```java
Dog dog = new Dog();

Animal animal = dog;
```

---

يحدث تلقائيًا.

```text
Child → Parent
```

---

## Downcasting

```java
Animal animal = new Dog();

Dog dog = (Dog) animal;
```

---

يحتاج Cast صريح.

---

## خطر

```java
Animal animal = new Animal();

Dog dog = (Dog) animal;
```

---

الناتج:

```text
ClassCastException
```

---

لذلك نستخدم:

```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
}
```

---

# 5) Booleans in Java

النوع:

```java
boolean
```

القيم:

```java
true
false
```

---

مثال:

```java
boolean isLoggedIn = true;
```

---

استخدام:

```java
if (isLoggedIn) {
    System.out.println("Welcome");
}
```

---

## Comparison Operators

```java
==
!=
>
<
>=
<=
```

تعيد دائمًا:

```text
boolean
```

---

مثال:

```java
int age = 20;

boolean result = age >= 18;
```

---

الناتج:

```text
true
```

---

# Logical Operators

## AND

```java
&&
```

```java
true && true
```

الناتج:

```text
true
```

---

## OR

```java
||
```

---

## NOT

```java
!
```

---

Example:

```java
boolean isAdmin = true;
boolean active = false;

System.out.println(isAdmin && active);
```

الناتج:

```text
false
```

---

# 6) Multiple Variables

بدل:

```java
int x = 10;
int y = 20;
int z = 30;
```

يمكن:

```java
int x = 10, y = 20, z = 30;
```

---

مثال:

```java
String first = "Ali",
       middle = "Ahmed",
       last = "Ragab";
```

---

# 7) var Keyword

تم إضافته في:

```text
Java 10
```

---

مثال:

```java
var name = "Ali";
```

الـ Compiler يحولها إلى:

```java
String name = "Ali";
```

---

مثال:

```java
var age = 25;
```

تصبح:

```java
int age = 25;
```

---

## ليست Dynamic Typing

هذا خطأ شائع.

```java
var age = 25;

age = "hello";
```

خطأ.

---

النوع يحدد مرة واحدة فقط.

---

## متى نستخدمها؟

جيد مع الأنواع الطويلة:

```java
var map = new HashMap<String,List<User>>();
```

بدل:

```java
HashMap<String,List<User>> map =
        new HashMap<>();
```

---

## متى لا نستخدمها؟

سيء هنا:

```java
var x = getData();
```

لا أحد يعرف نوع x.

---

# Best Practice

استخدمها عندما يكون النوع واضحًا من الجهة اليمنى.

---

مثال جيد:

```java
var users = new ArrayList<User>();
```

---

مثال سيء:

```java
var result = calculate();
```

---

# 8) Constants in Java

الثابت يعني:

```text
Value cannot change
```

---

مثال:

```java
final double PI = 3.14159;
```

---

محاولة التعديل:

```java
PI = 5;
```

خطأ Compilation.

---

## Naming Convention

```java
MAX_USERS
DATABASE_URL
PI
```

---

مثال:

```java
public class AppConstants {

    public static final int MAX_USERS = 100;

    public static final String APP_NAME =
            "SkillHub";
}
```

---

استخدام:

```java
System.out.println(
        AppConstants.MAX_USERS
);
```

---

## لماذا static و final معًا؟

### static

يوجد نسخة واحدة فقط لكل Class.

---

### final

لا يمكن تغيير القيمة.

---

لذلك:

```java
public static final
```

هو الشكل القياسي للـ Constants في Java.

---

# Summary Table

|Concept|Key Idea|
|---|---|
|Static Method|Class level, hidden not overridden|
|Constructor|Used to create object|
|Private Method|Not inherited|
|Widening Casting|Automatic|
|Narrowing Casting|Explicit|
|Upcasting|Child → Parent|
|Downcasting|Parent → Child|
|boolean|true or false|
|var|Type inferred at compile time|
|final constant|Value cannot change|
