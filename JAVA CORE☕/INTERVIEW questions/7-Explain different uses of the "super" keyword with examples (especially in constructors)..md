
في Java، كلمة `super` من أهم الكلمات في الـ OOP، وبتستخدم عشان تشير إلى:

# الـ Parent Class (الأب)

يعني لو عندك:

```java
class Student extends Person
```

فـ `super` تشير إلى:

```java
Person
```

---

# أهم استخدامات `super` في Java 🔥

## 1️⃣ استدعاء Constructor الأب

وده أشهر استخدام.

---

# مثال

## Parent

```java
class Person {

    String name;

    public Person(String name) {
        this.name = name;

        System.out.println("Person Constructor");
    }
}
```

---

## Child

```java
class Student extends Person {

    String university;

    public Student(String name, String university) {

        // استدعاء Constructor الأب
        super(name);

        this.university = university;

        System.out.println("Student Constructor");
    }
}
```

---

## Main

```java
public class Main {

    public static void main(String[] args) {

        Student s =
                new Student("Ali", "Cairo University");
    }
}
```

---

# الناتج

```text
Person Constructor
Student Constructor
```

---

# ليه استخدمنا super هنا؟

لأن:

# الأب هو المسؤول عن تهيئة الـ fields الخاصة به

يعني:

- `name` تخص Person
    
- فـ Person Constructor هو اللي يهيئها
    

---

# مهم جدًا 🔥

## `super()` لازم تكون أول سطر داخل constructor

صح ✅

```java
super(name);
```

غلط ❌

```java
System.out.println("Hello");
super(name);
```

---

# لو مكتبتش super() ؟

Java تلقائي تعمل:

```java
super();
```

لكن:

# فقط إذا الأب عنده constructor فارغ.

---

# 2️⃣ الوصول لـ Fields الأب

---

# مثال

```java
class Animal {

    String sound = "Animal Sound";
}
```

---

```java
class Cat extends Animal {

    String sound = "Meow";

    public void printSounds() {

        System.out.println(this.sound);

        System.out.println(super.sound);
    }
}
```

---

# الناتج

```text
Meow
Animal Sound
```

---

# الفرق

|الكلمة|تشير إلى|
|---|---|
|this.sound|field الحالية|
|super.sound|field الأب|

---

# 3️⃣ استدعاء Method من الأب

---

# مثال

## Parent

```java
class Person {

    public void displayInfo() {
        System.out.println("Person Info");
    }
}
```

---

## Child

```java
class Student extends Person {

    @Override
    public void displayInfo() {

        // استدعاء Method الأب
        super.displayInfo();

        System.out.println("Student Info");
    }
}
```

---

# Main

```java
public class Main {

    public static void main(String[] args) {

        Student s = new Student();

        s.displayInfo();
    }
}
```

---

# الناتج

```text
Person Info
Student Info
```

---

# ليه استخدمنا super.method() ؟

عشان:

- نشغل behavior الأب
    
- ثم نضيف behavior جديد
    

وده شائع جدًا في:

- inheritance
    
- frameworks
    
- Spring
    
- Android
    

---

# الفرق بين this و super 🔥

|this|super|
|---|---|
|تشير للـ object الحالي|تشير للأب|
|تستخدم fields الحالية|تستخدم fields الأب|
|تستدعي constructor الحالي|تستدعي constructor الأب|

---

# مثال يجمع everything 🔥

```java
class Person {

    protected String name;

    public Person(String name) {

        this.name = name;

        System.out.println("Person Constructor");
    }

    public void display() {
        System.out.println("Name: " + name);
    }
}
```

---

```java
class Student extends Person {

    private String university;

    public Student(String name, String university) {

        super(name);

        this.university = university;

        System.out.println("Student Constructor");
    }

    @Override
    public void display() {

        super.display();

        System.out.println("University: " + this.university);
    }
}
```

---

# Main

```java
public class Main {

    public static void main(String[] args) {

        Student s =
                new Student("Ali", "Cairo University");

        s.display();
    }
}
```

---

# الناتج النهائي

```text
Person Constructor
Student Constructor

Name: Ali
University: Cairo University
```

---

# Interview Questions مشهورة 🔥

## هل super keyword ممكن تستخدم للوصول لـ private fields ؟

# لا ❌

لأن الـ private members لا تُورث.

---

## هل super() و this() ينفعوا مع بعض في نفس constructor؟

# لا ❌

لازم واحدة فقط تكون أول سطر.

---

## هل Java تضيف super() تلقائي؟

# أيوه ✅

إذا:

- الأب عنده constructor فارغ
    
- وأنت مكتبتش super()
    

---

# استخدامات super باختصار

| الاستخدام                | المثال            |
| ------------------------ | ----------------- |
| استدعاء constructor الأب | `super(name)`     |
| الوصول لـ field الأب     | `super.name`      |
| استدعاء method الأب      | `super.display()` |