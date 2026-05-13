# Java Records 📦

## ما هو الـ Record؟

الـ `record` هو نوع خاص من الـ class اتضاف في **Java 16** عشان يحل مشكلة كتابة كود كتير (boilerplate) لما بتعمل class بتشيل بيانات بس.

> بدل ما تكتب constructor + getters + equals + hashCode + toString بإيدك، الـ record بيعملهم كلهم تلقائياً.

---

## المشكلة اللي جه يحلها

```java
// ❌ class تقليدية لشيل بيانات فقط — كود كتير جداً
class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public boolean equals(Object o) { ... }

    @Override
    public int hashCode() { ... }

    @Override
    public String toString() { ... }
}
```

```java
// ✅ نفس الحاجة بالـ record في سطر واحد
record Point(int x, int y) {}
```

---

## الصيغة الأساسية

```java
record RecordName(Type field1, Type field2, ...) {
    // اختياري: methods إضافية
}
```

### مثال بسيط

```java
record Person(String name, int age) {}

// الاستخدام
Person p = new Person("Ali", 25);

System.out.println(p.name());     // Ali
System.out.println(p.age());      // 25
System.out.println(p);            // Person[name=Ali, age=25]
```

---

## اللي بيعمله الـ Record تلقائياً

|الميزة|الوصف|
|---|---|
|`Constructor`|بيعمل canonical constructor تلقائياً|
|`Getters`|بيعمل accessor method لكل field باسمه|
|`equals()`|بيقارن بين الـ records بناءً على القيم|
|`hashCode()`|بيحسب hash بناءً على القيم|
|`toString()`|بيطبع اسم الـ record وقيم الـ fields|

---

## الـ Record Immutable

كل fields في الـ record بتكون **`final`** تلقائياً — يعني مينفعش تغير قيمتها بعد ما تتعمل.

```java
record Point(int x, int y) {}

Point p = new Point(3, 5);
p.x = 10; // ❌ ERROR! cannot assign a value to final variable
```

---

## تخصيص الـ Constructor

### Compact Constructor

```java
record Person(String name, int age) {

    // بيتنفذ قبل الـ constructor الأصلي
    Person {
        if (age < 0)
            throw new IllegalArgumentException("Age cannot be negative");
        name = name.trim(); // تعديل مسموح هنا
    }
}

Person p1 = new Person("  Ali  ", 25); // name هتبقى "Ali"
Person p2 = new Person("Sara", -1);    // ❌ Exception
```

### Custom Constructor

```java
record Rectangle(double width, double height) {

    // constructor إضافي
    Rectangle(double side) {
        this(side, side); // مربع
    }
}

Rectangle r1 = new Rectangle(4.0, 6.0);
Rectangle r2 = new Rectangle(5.0); // مربع 5x5
```

---

## إضافة Methods

```java
record Circle(double radius) {

    // instance method
    double area() {
        return Math.PI * radius * radius;
    }

    double circumference() {
        return 2 * Math.PI * radius;
    }

    // static method
    static Circle unit() {
        return new Circle(1.0);
    }
}

Circle c = new Circle(5.0);
System.out.println(c.area());          // 78.53...
System.out.println(Circle.unit());     // Circle[radius=1.0]
```

---

## الـ equals() في Record

```java
record Point(int x, int y) {}

Point p1 = new Point(3, 5);
Point p2 = new Point(3, 5);
Point p3 = new Point(1, 2);

System.out.println(p1.equals(p2)); // true  ← نفس القيم
System.out.println(p1.equals(p3)); // false ← قيم مختلفة
System.out.println(p1 == p2);      // false ← objects مختلفة في الميموري
```

> في الـ class العادية `p1.equals(p2)` كانت هترجع `false` لو ما overrideش الـ equals بإيدك.

---

## Record مع Generics

```java
record Pair<A, B>(A first, B second) {}

Pair<String, Integer> p = new Pair<>("Score", 100);
System.out.println(p.first());  // Score
System.out.println(p.second()); // 100
```

---

## Record مع Interfaces

```java
interface Describable {
    String describe();
}

record Product(String name, double price) implements Describable {

    @Override
    public String describe() {
        return name + " costs $" + price;
    }
}

Product laptop = new Product("Laptop", 999.99);
System.out.println(laptop.describe()); // Laptop costs $999.99
```

---

## Record جوّا Record

```java
record Address(String city, String country) {}

record Employee(String name, Address address) {}

Employee emp = new Employee("Ali", new Address("Cairo", "Egypt"));

System.out.println(emp.name());            // Ali
System.out.println(emp.address().city());  // Cairo
```

---

## Record كـ Local Record

```java
void printInfo() {
    record Info(String key, String value) {}  // local record جوّا method

    Info info = new Info("Language", "Java");
    System.out.println(info); // Info[key=Language, value=Java]
}
```

---

## المقارنة: Record vs Class

||`record`|`class`|
|---|---|---|
|**الهدف**|شيل بيانات فقط|أي منطق أو سلوك|
|**Immutability**|إجبارية|اختيارية|
|**Boilerplate**|لا يوجد|كتير|
|**Inheritance**|❌ مش ينفع يورّث|✅ ينفع|
|**extends**|❌ مش ينفع|✅ ينفع|
|**implements**|✅ ينفع|✅ ينفع|
|**Fields إضافية**|❌ static فقط|✅ أي نوع|
|**equals/hashCode**|تلقائي بالقيم|يدوي أو تلقائي|

---

## إمتى تستخدم Record؟

### ✅ استخدم Record لما:

- بتعمل **Data Transfer Object (DTO)** بين الـ layers
- بتعمل **Response/Request objects** في الـ API
- بتمثل **قيمة** زي إحداثيات أو نقطة أو نطاق
- عايز **immutable data** بدون تعقيد
- بتعمل **temporary grouping** لبيانات مع بعض

```java
// DTO مثال
record UserDTO(Long id, String username, String email) {}

// API Response
record ApiResponse<T>(boolean success, T data, String message) {}

// Value object
record DateRange(LocalDate from, LocalDate to) {}
```

### ❌ متستخدمش Record لما:

- محتاج تغير الـ fields بعد الإنشاء (mutable state)
- محتاج تورّث من class تانية
- الـ object عنده سلوك معقد أكتر من البيانات
- محتاج fields instance إضافية

---

## اللي مش مسموح في Record

```java
record Bad(int x) {
    int extra;              // ❌ instance field إضافية مش مسموح
    static int count = 0;  // ✅ static field مسموح
}

record Child(int z) extends Bad {} // ❌ record مش بتورّث
```

---

## مثال عملي كامل

```java
record Student(String name, int grade, double gpa) {

    // validation
    Student {
        if (grade < 1 || grade > 4)
            throw new IllegalArgumentException("Grade must be 1-4");
        if (gpa < 0.0 || gpa > 4.0)
            throw new IllegalArgumentException("GPA must be 0-4");
    }

    // computed method
    String status() {
        return gpa >= 2.0 ? "Passing" : "Failing";
    }

    // static factory
    static Student freshman(String name) {
        return new Student(name, 1, 0.0);
    }
}

// الاستخدام
Student s1 = new Student("Ali", 3, 3.7);
Student s2 = Student.freshman("Sara");

System.out.println(s1);           // Student[name=Ali, grade=3, gpa=3.7]
System.out.println(s1.status());  // Passing
System.out.println(s2);           // Student[name=Sara, grade=1, gpa=0.0]
```

---

## ملخص سريع

```
Record = Immutable Data Class بدون Boilerplate

✅ تلقائياً: constructor + getters + equals + hashCode + toString
✅ ينفع: implements interface, static fields, methods, generics
❌ ما ينفعش: extends class, instance fields إضافية, mutable state
```

---

## Tags

`#Java` `#OOP` `#Record` `#DataClass` `#Immutable` `#Java16`