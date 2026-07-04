# Static Block vs Instance Block vs Constructor in Java

الصورة دي بتشرح واحدة من أهم المواضيع في Java Interviews، وهي:

> **ترتيب تنفيذ الـ Static Block والـ Instance Block والـ Constructor أثناء إنشاء الـ Object.**

خلينا نشرحه من البداية كأنه Topic كامل.

---

# أولاً: الكود

```java
public class Employee {

    {
        System.out.println("Instance Block");
    }

    static {
        System.out.println("Static Block");
    }

    Employee() {
        System.out.println("Constructor");
    }
}
```

---

# أولاً: ما هو الـ Static Block؟

هو بلوك بيتكتب بالشكل ده:

```java
static {
    // code
}
```

### خصائصه

- بيتنفذ مرة واحدة فقط.
    
- بيتنفذ أول مرة الـ Class يتحمل في الـ JVM.
    
- لا يعتمد على وجود Object.
    
- يستخدم لتهيئة البيانات الخاصة بالـ Class.
    

---

## Use Cases للـ Static Block

### 1. تحميل Driver

```java
static {
    Class.forName("com.mysql.jdbc.Driver");
}
```

---

### 2. قراءة Configuration

```java
static {
    properties.load(...);
}
```

---

### 3. تهيئة Static Variables

```java
static {
    tax = 14;
}
```

---

### 4. إنشاء Connection Pool

بدل ما يتعمل مع كل Object.

---

# ما هو الـ Instance Block؟

هو بلوك بدون كلمة static.

```java
{
    System.out.println("Instance Block");
}
```

---

## خصائصه

- يتنفذ مع كل Object جديد.
    
- يتنفذ قبل الـ Constructor.
    
- يستخدم لتجميع الكود المشترك بين أكثر من Constructor.
    

---

## لماذا موجود أصلاً؟

افترض عندك:

```java
Employee(){}

Employee(String name){}

Employee(int id){}
```

كل Constructor محتاج يعمل نفس الكود.

بدل ما تكرره:

```java
connect();
loadData();
```

في كل Constructor،

تكتب:

```java
{
    connect();
    loadData();
}
```

وسيتنفذ قبل أي Constructor.

---

# ما هو الـ Constructor؟

هو المسئول عن إنشاء الـ Object وتهيئته.

```java
Employee() {
    System.out.println("Constructor");
}
```

---

# السؤال المهم

لو كتبنا

```java
public static void main(String[] args) {

    new Employee();

}
```

إيه اللي هيحصل؟

---

## الخطوة الأولى

الـ JVM تشوف:

> هل الـ Class متحمل؟

لو لأ

تحمله.

وأثناء التحميل

تنفذ

```java
static {

}
```

إذن أول حاجة:

```
Static Block
```

---

## الخطوة الثانية

الذاكرة تخصص مساحة للـ Object.

---

## الخطوة الثالثة

تنفيذ Instance Block

```
Instance Block
```

---

## الخطوة الرابعة

تنفيذ Constructor

```
Constructor
```

---

# إذن الـ Output

```
Static Block
Instance Block
Constructor
```

---

# لماذا الـ Static أولاً؟

لأن الـ Object لا يمكن إنشاؤه إلا بعد تحميل الـ Class.

وترتيب الـ JVM هو:

```
Load Class

↓

Static Block

↓

Create Object

↓

Instance Block

↓

Constructor
```

---

# ماذا يحدث لو أنشأنا Objectين؟

```java
public static void main(String[] args) {

    new Employee();

    new Employee();

}
```

النتيجة

```
Static Block

Instance Block
Constructor

Instance Block
Constructor
```

لاحظ:

```
Static Block
```

ظهرت مرة واحدة فقط.

بينما

```
Instance Block
```

ظهرت مرتين.

و

```
Constructor
```

مرتين.

---

# لماذا؟

لأن الـ Static خاص بالـ Class.

أما الباقي خاص بكل Object.

---

# ماذا لو أنشأنا ثلاثة Objects؟

```java
new Employee();

new Employee();

new Employee();
```

النتيجة

```
Static Block

Instance Block
Constructor

Instance Block
Constructor

Instance Block
Constructor
```

---

# ماذا لو لم ننشئ أي Object؟

```java
public static void main(String[] args) {

}
```

هل سينفذ Static Block؟

الإجابة:

نعم، لأن تشغيل البرنامج يتطلب تحميل الكلاس الذي يحتوي على `main()`، وبالتالي سيتم تنفيذ الـ Static Block عند تحميل الكلاس.

---

# ماذا لو استدعينا Static Method فقط؟

```java
Employee.print();
```

إذا كانت:

```java
static void print(){}
```

النتيجة:

سيتم تحميل الـ Class أولًا.

ثم

```
Static Block
```

ثم

```
print()
```

---

# ماذا لو كان عندي أكثر من Static Block؟

```java
static{
    System.out.println("1");
}

static{
    System.out.println("2");
}

static{
    System.out.println("3");
}
```

النتيجة

```
1
2
3
```

بالترتيب المكتوب داخل الملف.

---

# ماذا لو كان عندي أكثر من Instance Block؟

```java
{
    System.out.println("A");
}

{
    System.out.println("B");
}
```

ثم

```java
new Employee();
```

النتيجة

```
Static Block

A

B

Constructor
```

---

# ماذا لو كان عندي أكثر من Constructor؟

```java
Employee(){

}

Employee(String name){

}
```

وكتبت

```java
new Employee("Ali");
```

الترتيب

```
Static Block

Instance Block

Employee(String)
```

---

# العلاقة مع الـ Inheritance

لو عندنا:

```java
class Animal {

    static{
        System.out.println("Animal Static");
    }

    {
        System.out.println("Animal Instance");
    }

    Animal(){
        System.out.println("Animal Constructor");
    }
}

class Dog extends Animal{

    static{
        System.out.println("Dog Static");
    }

    {
        System.out.println("Dog Instance");
    }

    Dog(){
        System.out.println("Dog Constructor");
    }
}
```

ثم

```java
new Dog();
```

الترتيب سيكون:

```
Animal Static

Dog Static

Animal Instance

Animal Constructor

Dog Instance

Dog Constructor
```

### لماذا؟

لأن Java دائمًا تنفذ من الأب إلى الابن:

1. تحميل كلاس الأب (Static Blocks).
    
2. تحميل كلاس الابن (Static Blocks).
    
3. إنشاء جزء الأب من الكائن (Instance Block ثم Constructor).
    
4. إنشاء جزء الابن من الكائن (Instance Block ثم Constructor).
    

---

# ملخص سريع

|العنصر|يتنفذ إمتى؟|عدد مرات التنفيذ|
|---|---|---|
|Static Block|عند تحميل الـ Class|مرة واحدة فقط|
|Instance Block|عند إنشاء كل Object وقبل الـ Constructor|مرة لكل Object|
|Constructor|بعد الـ Instance Block مباشرة|مرة لكل Object|

## الترتيب النهائي

```
Load Class
        ↓
Static Block
        ↓
Create Object
        ↓
Instance Block
        ↓
Constructor
```

### قاعدة سهلة للحفظ

**Static → Instance → Constructor**

- **Static**: مرة واحدة مع تحميل الكلاس.
    
- **Instance**: مع كل كائن جديد، قبل الـ Constructor.
    
- **Constructor**: آخر خطوة في تهيئة الكائن.