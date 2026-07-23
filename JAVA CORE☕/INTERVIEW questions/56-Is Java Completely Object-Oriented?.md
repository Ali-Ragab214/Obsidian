ده من أشهر أسئلة Java Interviews، وكتير من الناس بتجاوب عليه إجابة ناقصة.

---

# Interview Question

## Is Java Completely Object-Oriented?

### Short Answer

**No, Java is not a completely Object-Oriented Programming (OOP) language.**

Although Java is designed around object-oriented principles, it is **not purely object-oriented** because it supports **primitive data types** that are not objects.

---

# Why Isn't Java Fully Object-Oriented?

في اللغات الـ Pure Object-Oriented، **كل شيء عبارة عن Object**.

لكن في Java، عندك نوعين من البيانات:

1. Primitive Data Types
    
2. Reference Types (Objects)
    

---

# 1. Primitive Data Types

Java تحتوي على 8 Primitive Types:

```java
byte
short
int
long
float
double
char
boolean
```

مثال:

```java
int age = 22;
double salary = 5000.50;
boolean active = true;
```

لاحظ أن:

```java
age
```

ليس Object.

يعني لا يمكنك كتابة:

```java
age.someMethod();
```

لأنه مجرد قيمة مخزنة في الـ Stack.

وهذا يخالف مفهوم الـ Pure OOP.

---

# 2. Objects in Java

أما الـ Objects فهي Instances من Classes.

مثال:

```java
String name = "Ali";
```

هنا:

```java
name
```

هو Object.

لذلك يمكن استدعاء Methods عليه.

```java
System.out.println(name.length());
System.out.println(name.toUpperCase());
```

---

# Primitive vs Object

```java
int x = 10;
Integer y = 10;
```

الفرق:

```java
x
```

Primitive Value.

بينما

```java
y
```

Object من Wrapper Class اسمها:

```java
Integer
```

ولذلك تستطيع كتابة:

```java
System.out.println(y.compareTo(20));
```

أما:

```java
x.compareTo(20);
```

فسيعطي Compile Error.

---

# Wrapper Classes

لكل Primitive يوجد Wrapper Class.

|Primitive|Wrapper Class|
|---|---|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|
|boolean|Boolean|

هذه الـ Wrapper Classes تعتبر Objects.

---

# Why Did Java Introduce Primitive Types?

السؤال ده مهم جدًا في المقابلات.

لو كانت Java جعلت كل شيء Object، لكان الأداء أبطأ واستهلاك الذاكرة أكبر.

مثال:

```java
int x = 5;
```

يستهلك ذاكرة قليلة جدًا.

أما:

```java
Integer x = 5;
```

فهو Object.

يحتاج:

- Object Header
    
- Metadata
    
- Heap Allocation
    
- Reference
    

وبالتالي:

- يستهلك Memory أكثر.
    
- أبطأ في الإنشاء.
    

لذلك Java استخدمت Primitive Types لتحسين الأداء.

---

# Autoboxing & Unboxing

ابتداءً من Java 5 أضافت ميزة اسمها:

## Autoboxing

تحويل Primitive إلى Wrapper تلقائيًا.

```java
int x = 10;

Integer y = x;
```

في الحقيقة Java تعمل:

```java
Integer y = Integer.valueOf(x);
```

---

## Unboxing

العكس.

```java
Integer y = 10;

int x = y;
```

في الحقيقة:

```java
int x = y.intValue();
```

---

# Does Everything in Java Extend Object?

لا.

مثال:

```java
Object obj = new Object();
```

يعمل.

لكن:

```java
Object obj = 10;
```

الذي يحدث فعليًا هو:

```java
Object obj = Integer.valueOf(10);
```

بسبب الـ Autoboxing.

أما الـ Primitive نفسه فلا يرث من `Object`.

---

# Comparison with Pure Object-Oriented Languages

في لغات مثل:

- Smalltalk
    
- Ruby
    

كل شيء Object.

حتى الرقم:

```text
5
```

هو Object.

يمكنك استدعاء Methods عليه مباشرة.

أما Java:

```java
int x = 5;
```

ليس Object.

---

# Is String a Primitive?

لا.

رغم أننا نكتبه بسهولة:

```java
String name = "Ali";
```

إلا أن `String` Class.

يعني:

```java
name.length();
name.toUpperCase();
```

كلاهما Methods خاصة بالـ Object.

---

# Interview Follow-up

### If Java is not completely object-oriented, why is it called an object-oriented language?

لأن معظم خصائص اللغة مبنية على مفاهيم الـ OOP مثل:

- Classes
    
- Objects
    
- Encapsulation
    
- Inheritance
    
- Polymorphism
    
- Abstraction
    

لكن وجود Primitive Types يجعلها **ليست Pure Object-Oriented**.

---

# Common Interview Mistakes

❌ "Java is completely object-oriented."

هذه إجابة خاطئة.

---

❌ "Everything in Java extends Object."

خاطئة أيضًا.

Primitive Types لا ترث من `Object`.

---

❌ "int is an object."

خطأ.

`int` Primitive Type.

أما:

```java
Integer
```

فهو Object.

---

# Interview Answer (30 Seconds)

> **No, Java is not completely object-oriented. Although Java follows the core principles of object-oriented programming, it is not a pure object-oriented language because it supports eight primitive data types (`int`, `char`, `boolean`, etc.), which are not objects. These primitive types were introduced to improve performance and reduce memory usage. To work with primitives as objects, Java provides Wrapper Classes such as `Integer`, `Double`, and `Character`. Therefore, Java is considered an object-oriented language, but not a purely object-oriented one.**

---

## نقطة قوية جدًا لو اتسألت Follow-up

لو الإنترفيوور سألك:

> **"طيب بما إن `int` مش Object، إزاي الكود ده بيشتغل؟"**

```java
List<Integer> numbers = new ArrayList<>();
numbers.add(10);
```

الإجابة هي:

لأن Java بتعمل **Autoboxing** تلقائيًا، وتحول:

```java
numbers.add(10);
```

إلى:

```java
numbers.add(Integer.valueOf(10));
```

وده من أشهر أسئلة الـ Java Core Interviews لأنه بيربط بين **Primitive Types** و**Wrapper Classes** و**Collections**.