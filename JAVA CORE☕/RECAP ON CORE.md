
# ☕ Java Core - Full Recap للـ Backend Developer

سأشرح كل topic بالتفصيل مع أمثلة عملية موجهة لـ Spring Backend. هنبدأ من الأساس وندخل في العمق.

---

# 📌 PART 1: OOP الكامل

---


In Java, **`Throwable`** is the parent (superclass) of all exceptions and errors in the exception hierarchy.

The hierarchy looks like this:

```
Object
  └── Throwable
        ├── Exception
        │     ├── RuntimeException (unchecked)
        │     └── (checked exceptions)
        └── Error
              └── (JVM-level errors)
```
throw => throw or  create an exception
throws => declare an exception
**Why not the others?**

- **`Exception`** — is a subclass of `Throwable`, not the root
- **`Error`** — is also a subclass of `Throwable` (represents serious JVM issues like `OutOfMemoryError`)
- **`Object`** — is the root of _all_ Java classes, but it's not specific to the exception hierarchy

> `Throwable` is the only class whose objects can be used with the `throw` keyword or declared in a `catch` block.


تمام يا علي 🔥  
يلا نفهم 35 و36 و39 بعمق وبأمثلة واضحة 👌

---

# ✅ سؤال 35

## ❓ Can static method be overridden?

الإجابة الصحيحة: **B) No**

## 📌 ليه؟

الـ **static method** مش بتتعملها overriding  
لكن بيحصل لها حاجة اسمها:

> **Method Hiding**

لأنها بتتربط بالـ class نفسه مش بالـ object.

---

## 🎯 مثال

```java
class Parent {
    static void show() {
        System.out.println("Parent static method");
    }
}

class Child extends Parent {
    static void show() {
        System.out.println("Child static method");
    }
}

public class Test {
    public static void main(String[] args) {
        Parent obj = new Child();
        obj.show();
    }
}
```

### 🔎 الناتج؟

```
Parent static method
```

## ❗ ليه مش Child؟

لأن:

- static method بتتحدد وقت compile-time
    
- مش dynamic binding
    
- مش polymorphism
    

---

## 🧠 الفرق بين Override و Hiding

|Overriding|Static Hiding|
|---|---|
|Instance method|Static method|
|Runtime binding|Compile-time binding|
|Polymorphism|No polymorphism|

---

# ✅ سؤال 36

## ❓ What happens if finally has return?

الإجابة الصحيحة:  
**B) Overrides previous return**

---

## 📌 الفكرة

لو عندك return في try  
وبعدين return تاني في finally

اللي في finally هو اللي هيتنفذ.

---

## 🎯 مثال

```java
public class Test {
    static int test() {
        try {
            return 10;
        } finally {
            return 20;
        }
    }

    public static void main(String[] args) {
        System.out.println(test());
    }
}
```

### 🔎 الناتج؟

```
20
```

## ❗ ليه؟

لأن finally دايمًا بيتنفذ  
ولو فيه return → بيكسر أي return قبله.

---

## ⚠️ ودي مصيبة ليه؟

لأنها:

- بتلغي الاستثناء لو كان في exception
    
- بتخفي return الحقيقي
    
- Best practice: **متحطش return في finally**
    

---

# ✅ سؤال 39

## ❓ Which class is synchronized?

الإجابة الصحيحة:  
**B) StringBuffer**

---

## 📌 الفرق بين StringBuffer و StringBuilder

|StringBuffer|StringBuilder|
|---|---|
|Thread-safe|Not thread-safe|
|Synchronized|Not synchronized|
|أبطأ|أسرع|

---

## 🎯 مثال Multi-thread

```java
StringBuffer sb = new StringBuffer();

Runnable r = () -> {
    for(int i = 0; i < 1000; i++) {
        sb.append("A");
    }
};

Thread t1 = new Thread(r);
Thread t2 = new Thread(r);

t1.start();
t2.start();
```

### ✔ StringBuffer

آمن ومفيش corruption

### ❌ StringBuilder

ممكن يحصل data inconsistency

---

## 🧠 ليه HashMap مش synchronized؟

لأنه:

- مش thread-safe
    
- في multi-thread ممكن يحصل infinite loop (في الإصدارات القديمة)
    
- البديل:
    
    - ConcurrentHashMap
        
    - Collections.synchronizedMap()
        

---

# 🎯 ملخص سريع

|السؤال|الفكرة الأساسية|
|---|---|
|35|static methods لا تُoverride بل تُhide|
|36|return في finally يلغي أي return قبله|
|39|StringBuffer هو synchronized|

---

لو حابب أعملك:

- 🔥 امتحان Advanced أصعب بكتير
    
- 🔥 أو شرح Multi-threading بعمق
    
- 🔥 أو نحل 10 أسئلة tricky جدًا
    

قولّي 🔥
## 1.1 Classes & Objects

الـ Class هي القالب، والـ Object هو النسخة الحقيقية منه.

```java
// القالب (Class)
public class User {
    // Fields (State)
    private String name;
    private String email;
    private int age;

    // Constructor
    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }

    // Methods (Behavior)
    public String getName() { return name; }
    public String getEmail() { return email; }
    public int getAge() { return age; }

    @Override
    public String toString() {
        return "User{name='" + name + "', email='" + email + "'}";
    }
}

// استخدام
User user1 = new User("أحمد", "ahmed@gmail.com", 25);
User user2 = new User("سارة", "sara@gmail.com", 22);

System.out.println(user1); // User{name='أحمد', email='ahmed@gmail.com'}
```

---

## 1.2 الـ 4 ركائز الأساسية في OOP

---

### 🔒 1 - Encapsulation (التغليف)

**الفكرة:** إخبي البيانات الداخلية، وادي access من خلال methods بس.

```java
public class BankAccount {
    private double balance;  // ← مش أي حد يقدر يعدلها مباشرة
    private String owner;

    public BankAccount(String owner, double initialBalance) {
        this.owner = owner;
        this.balance = initialBalance;
    }

    // Controlled access
    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > balance) throw new IllegalStateException("Insufficient funds");
        balance -= amount;
    }
}

// الاستخدام
BankAccount account = new BankAccount("أحمد", 1000);
account.deposit(500);
account.withdraw(200);
System.out.println(account.getBalance()); // 1300.0

// account.balance = -999999; // ← Compile Error! محمية
```

**في Spring:** كل Entity/DTO بتعملها encapsulation بنفس الطريقة دي.

---

### 🧬 2 - Inheritance (الوراثة)

**الفكرة:** Class جديدة بترث خصائص وسلوك class موجودة.

```java
// Parent Class
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child Class
public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);  // ← بيستدعي constructor الـ parent
        this.breed = breed;
    }

    // Method جديدة خاصة بالـ Dog
    public void bark() {
        System.out.println(name + " says: Woof!");
    }

    // Override method من الـ parent
    @Override
    public void eat() {
        System.out.println(name + " is eating dog food");
    }
}

// الاستخدام
Dog dog = new Dog("Rex", 3, "Labrador");
dog.eat();    // Rex is eating dog food (overridden)
dog.sleep();  // Rex is sleeping (inherited)
dog.bark();   // Rex says: Woof! (new)
```

```
        Animal
        /    \
      Dog    Cat
      /
  Labrador
```

**⚠️ قاعدة مهمة:** Java بتدعم **Single Inheritance** بس — class واحدة مش تقدر ترث من اتنين في نفس الوقت.

---

### 🎭 3 - Polymorphism (تعدد الأشكال)

**الفكرة:** نفس الـ method بتتصرف بشكل مختلف حسب الـ object.

**نوعين:**

**أ) Compile-time Polymorphism = Method Overloading**

```java
public class Calculator {
    // نفس الاسم، parameters مختلفة
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

Calculator calc = new Calculator();
System.out.println(calc.add(2, 3));        // 5
System.out.println(calc.add(2.5, 3.5));   // 6.0
System.out.println(calc.add(1, 2, 3));    // 6
```

**ب) Runtime Polymorphism = Method Overriding**

```java
public class Shape {
    public double area() {
        return 0;
    }
}

public class Circle extends Shape {
    private double radius;
    public Circle(double radius) { this.radius = radius; }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width, height;
    public Rectangle(double w, double h) { this.width = w; this.height = h; }

    @Override
    public double area() {
        return width * height;
    }
}

// ✨ هنا الـ Magic
Shape[] shapes = {
    new Circle(5),
    new Rectangle(4, 6),
    new Circle(3)
};

for (Shape shape : shapes) {
    System.out.println(shape.area()); // بيختار الـ method الصح runtime
}
// 78.54
// 24.0
// 28.27
```

---

### 🫙 4 - Abstraction (التجريد)

**الفكرة:** إخبي التفاصيل المعقدة، وبيّن بس الـ interface المهمة.

**طريقتين:**

**أ) Abstract Class**

```java
public abstract class Vehicle {
    protected String brand;
    protected int speed;

    public Vehicle(String brand) {
        this.brand = brand;
    }

    // Abstract method - لازم كل child class تعمل implementation
    public abstract void startEngine();

    // Concrete method - موروثة زي ما هي
    public void accelerate(int amount) {
        speed += amount;
        System.out.println(brand + " speed: " + speed);
    }
}

public class Car extends Vehicle {
    public Car(String brand) { super(brand); }

    @Override
    public void startEngine() {
        System.out.println(brand + ": Vroom! 🚗");
    }
}

public class ElectricCar extends Vehicle {
    public ElectricCar(String brand) { super(brand); }

    @Override
    public void startEngine() {
        System.out.println(brand + ": Silent start... 🔋");
    }
}

Vehicle car = new Car("Toyota");
Vehicle tesla = new ElectricCar("Tesla");

car.startEngine();    // Toyota: Vroom! 🚗
tesla.startEngine();  // Tesla: Silent start... 🔋
```

**ب) Interface**

```java
public interface Payable {
    void pay(double amount);           // abstract by default
    void refund(double amount);

    // Default method (Java 8+)
    default String getPaymentStatus() {
        return "Payment processed";
    }
}

public interface Notifiable {
    void sendNotification(String message);
}

// Class بتـ implement اتنين interfaces (مش ممكن في inheritance)
public class OnlineOrder implements Payable, Notifiable {

    @Override
    public void pay(double amount) {
        System.out.println("Paid: $" + amount + " via Credit Card");
    }

    @Override
    public void refund(double amount) {
        System.out.println("Refunded: $" + amount);
    }

    @Override
    public void sendNotification(String message) {
        System.out.println("Email sent: " + message);
    }
}
```

**الفرق بين Abstract Class و Interface:**

```
┌─────────────────┬──────────────────┬──────────────────┐
│                 │  Abstract Class  │    Interface      │
├─────────────────┼──────────────────┼──────────────────┤
│ Constructor     │       ✅          │       ❌          │
│ Fields          │       ✅          │  constants فقط   │
│ Implementation  │ جزئية ممكن       │  default فقط     │
│ Inheritance     │  واحدة بس        │  أكتر من واحدة   │
│ متى تستخدم؟    │ "IS-A" علاقة     │ "CAN-DO" علاقة  │
└─────────────────┴──────────────────┴──────────────────┘
```

---

## 1.3 Access Modifiers

```
┌───────────────┬──────────┬──────────┬───────────┬──────────┐
│   Modifier    │   نفس    │   نفس   │  Subclass │   كل     │
│               │  Class   │ Package  │           │  مكان    │
├───────────────┼──────────┼──────────┼───────────┼──────────┤
│   private     │    ✅    │    ❌    │     ❌    │    ❌    │
│   (default)   │    ✅    │    ✅    │     ❌    │    ❌    │
│   protected   │    ✅    │    ✅    │     ✅    │    ❌    │
│   public      │    ✅    │    ✅    │     ✅    │    ✅    │
└───────────────┴──────────┴──────────┴───────────┴──────────┘
```

---

## 1.4 Static vs Instance

```java
public class Counter {
    private static int totalCount = 0;  // مشتركة بين كل Objects
    private int id;                      // خاصة بكل Object

    public Counter() {
        totalCount++;
        this.id = totalCount;
    }

    public static int getTotalCount() { return totalCount; }  // static method
    public int getId() { return id; }                          // instance method
}

Counter c1 = new Counter();
Counter c2 = new Counter();
Counter c3 = new Counter();

System.out.println(Counter.getTotalCount()); // 3 (static - بنستدعيها بالـ class)
System.out.println(c1.getId());              // 1
System.out.println(c2.getId());              // 2
```

---

## 1.5 الـ Keywords المهمة

```java
// final
final int MAX = 100;          // constant - مش تتغير
final class ImmutableClass {}  // مش تتورث
final void method() {}         // مش تتـ Override

// this
public class Person {
    String name;
    Person(String name) {
        this.name = name;  // this = الـ current object
    }
}

// super
public class Child extends Parent {
    Child() {
        super();          // استدعاء constructor الـ parent
        super.method();   // استدعاء method الـ parent
    }
}
```

---

# 📌 PART 2: Collections & Generics

---

## 2.1 الـ Collections Framework - الخريطة الكاملة

```
                    Iterable
                       │
                   Collection
                  /     │     \
               List    Set    Queue
               /  \     │      │
          ArrayList LinkedHashSet Deque
          LinkedList  HashSet
          Vector      TreeSet
```

---

## 2.2 List

**ترتيب + تكرار مسموح**

### ArrayList

```java
import java.util.ArrayList;
import java.util.List;

List<String> names = new ArrayList<>();

// إضافة
names.add("أحمد");
names.add("سارة");
names.add("محمد");
names.add(1, "خالد");  // إضافة في index معين

// قراءة
System.out.println(names.get(0));    // أحمد
System.out.println(names.size());   // 4
System.out.println(names.contains("سارة")); // true

// تعديل
names.set(0, "علي");

// حذف
names.remove("محمد");
names.remove(0);  // حذف بالـ index

// تكرار
for (String name : names) {
    System.out.println(name);
}

// في Spring - بتستخدمها كتير في Service layer
List<User> users = userRepository.findAll();
```

**متى تستخدم ArrayList؟**

```
✅ لما محتاج access سريع بالـ index
✅ لما الـ Read أكتر من الـ Write
❌ لو بتعمل إضافة/حذف في النص كتير → استخدم LinkedList
```

### LinkedList

```java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.addFirst("أول");
linkedList.addLast("آخر");
linkedList.add("وسط");

System.out.println(linkedList.getFirst()); // أول
System.out.println(linkedList.getLast());  // آخر
linkedList.removeFirst();
```

```
ArrayList:  [0][1][2][3]  ← بيدور بالـ index سريع
LinkedList: [0]→[1]→[2]→[3] ← إضافة/حذف في الأول سريع
```

---

## 2.3 Set

**مفيش تكرار - مش بيحفظ ترتيب (إلا LinkedHashSet)**

```java
// HashSet - أسرع، مش بيحفظ ترتيب
Set<String> hashSet = new HashSet<>();
hashSet.add("Java");
hashSet.add("Python");
hashSet.add("Java");     // ← مش هيتضاف مرة تانية
System.out.println(hashSet.size()); // 2

// LinkedHashSet - بيحفظ ترتيب الإضافة
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("C");
linkedHashSet.add("A");
linkedHashSet.add("B");
System.out.println(linkedHashSet); // [C, A, B]

// TreeSet - بيرتب تلقائياً
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(5);
treeSet.add(1);
treeSet.add(3);
System.out.println(treeSet); // [1, 3, 5]
```

**مثال عملي - إزالة duplicates:**

```java
List<String> withDuplicates = List.of("Java", "Python", "Java", "Go", "Python");
Set<String> unique = new HashSet<>(withDuplicates);
System.out.println(unique); // [Java, Python, Go]
```

---

## 2.4 Map

**Key-Value pairs - مش بيكرر الـ keys**

```java
// HashMap - الأشيع
Map<String, Integer> scores = new HashMap<>();
scores.put("أحمد", 95);
scores.put("سارة", 88);
scores.put("محمد", 92);

// قراءة
System.out.println(scores.get("أحمد"));           // 95
System.out.println(scores.getOrDefault("علي", 0)); // 0 (مش موجود)
System.out.println(scores.containsKey("سارة"));   // true
System.out.println(scores.size());                // 3

// تكرار
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Java 8+ - forEach
scores.forEach((name, score) ->
    System.out.println(name + " scored " + score));

// تعديل
scores.put("أحمد", 99);  // update
scores.putIfAbsent("خالد", 75);  // بيضيف بس لو مش موجود

// حذف
scores.remove("محمد");
```

**أنواع الـ Map:**

```
HashMap      → أسرع، مش بيحفظ ترتيب
LinkedHashMap → بيحفظ ترتيب الإضافة
TreeMap       → بيرتب بالـ key تلقائياً
```

**مثال عملي - عد التكرار:**

```java
String text = "hello world hello java world";
String[] words = text.split(" ");

Map<String, Integer> wordCount = new HashMap<>();
for (String word : words) {
    wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
}

System.out.println(wordCount);
// {hello=2, world=2, java=1}
```

---

## 2.5 Queue & Stack

```java
// Queue (FIFO - أول واحد جه أول واحد يطلع)
Queue<String> queue = new LinkedList<>();
queue.offer("أول");
queue.offer("تاني");
queue.offer("تالت");

System.out.println(queue.poll()); // أول (ويحذفه)
System.out.println(queue.peek()); // تاني (من غير حذف)

// Stack (LIFO - آخر واحد جه أول واحد يطلع)
Deque<String> stack = new ArrayDeque<>();
stack.push("أول");
stack.push("تاني");
stack.push("تالت");

System.out.println(stack.pop());  // تالت
System.out.println(stack.peek()); // تاني
```

---

## 2.6 Generics

**الفكرة:** نكتب code مرة واحدة يشتغل مع أي type.

```java
// من غير Generics - خطر
List list = new ArrayList();
list.add("hello");
list.add(123);  // مش هيطلع error!
String s = (String) list.get(1); // ClassCastException runtime!

// مع Generics - آمن
List<String> safeList = new ArrayList<>();
safeList.add("hello");
// safeList.add(123); // ← Compile Error فوراً ✅
```

**كتابة Generic Class:**

```java
public class Box<T> {
    private T content;

    public Box(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}

Box<String> stringBox = new Box<>("Hello");
Box<Integer> intBox = new Box<>(42);
Box<User> userBox = new Box<>(new User("أحمد", "ahmed@gmail.com", 25));

System.out.println(stringBox.getContent()); // Hello
System.out.println(intBox.getContent());    // 42
```

**Generic Method:**

```java
public class Utils {
    // T ممكن يكون أي type
    public static <T> List<T> repeat(T item, int times) {
        List<T> result = new ArrayList<>();
        for (int i = 0; i < times; i++) {
            result.add(item);
        }
        return result;
    }
}

List<String> strings = Utils.repeat("Java", 3);
System.out.println(strings); // [Java, Java, Java]

List<Integer> numbers = Utils.repeat(0, 5);
System.out.println(numbers); // [0, 0, 0, 0, 0]
```

**Bounded Type Parameters:**

```java
// T لازم يكون Number أو subclass منه
public static <T extends Number> double sum(List<T> list) {
    double total = 0;
    for (T item : list) {
        total += item.doubleValue();
    }
    return total;
}

System.out.println(sum(List.of(1, 2, 3)));         // 6.0
System.out.println(sum(List.of(1.5, 2.5, 3.0)));  // 7.0
```

**Wildcards:**

```java
// ? extends T = قرا بس (Producer)
public void printNumbers(List<? extends Number> list) {
    for (Number n : list) {
        System.out.println(n);
    }
}

// ? super T = اكتب بس (Consumer)
public void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}
```

---

## 2.7 Collections Utility Class

```java
List<Integer> numbers = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

Collections.sort(numbers);              // ترتيب تصاعدي
Collections.reverse(numbers);          // عكس الترتيب
Collections.shuffle(numbers);          // خلط عشوائي
Collections.max(numbers);              // أكبر قيمة
Collections.min(numbers);              // أصغر قيمة
Collections.frequency(numbers, 1);     // كم مرة اتكررت
Collections.unmodifiableList(numbers); // مش تتعدل
```

---

# 📌 PART 3: Java 8+ Features

---

## 3.1 Lambda Expressions

**الفكرة:** طريقة مختصرة لكتابة anonymous functions.

```java
// قبل Java 8 - Verbose
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running...");
    }
};

// Java 8+ - Lambda
Runnable r2 = () -> System.out.println("Running...");

// Comparator قبل Java 8
Comparator<String> comp1 = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
};

// Comparator مع Lambda
Comparator<String> comp2 = (a, b) -> a.compareTo(b);

// Method Reference (أبسط لما بنستدعي method موجودة)
Comparator<String> comp3 = String::compareTo;
```

**صيغ الـ Lambda:**

```java
// بدون parameters
() -> System.out.println("Hello")

// parameter واحد (أقواس اختيارية)
x -> x * 2

// أكتر من parameter
(x, y) -> x + y

// أكتر من سطر واحد
(x, y) -> {
    int sum = x + y;
    return sum * 2;
}
```

---

## 3.2 Functional Interfaces

Interface عندها method واحدة abstract بس — وعليها @FunctionalInterface.

**الأهم:**

```java
// 1. Predicate<T> → بياخد T، بيرجع boolean
Predicate<Integer> isPositive = n -> n > 0;
Predicate<String> isLong = s -> s.length() > 5;

System.out.println(isPositive.test(5));     // true
System.out.println(isPositive.test(-3));    // false
System.out.println(isLong.test("Hello"));   // false
System.out.println(isLong.test("Java Programming")); // true

// Predicate Composition
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);
System.out.println(isPositiveAndEven.test(4));  // true
System.out.println(isPositiveAndEven.test(-4)); // false

// 2. Function<T, R> → بياخد T، بيرجع R
Function<String, Integer> getLength = s -> s.length();
Function<Integer, String> intToStr = n -> "Number: " + n;

System.out.println(getLength.apply("Java"));    // 4

// Function Chaining
Function<String, String> process = getLength.andThen(intToStr);
System.out.println(process.apply("Java")); // Number: 4

// 3. Consumer<T> → بياخد T، مش بيرجع حاجة
Consumer<String> print = s -> System.out.println(s);
Consumer<String> printUpper = s -> System.out.println(s.toUpperCase());

print.accept("hello");       // hello
printUpper.accept("hello");  // HELLO

// 4. Supplier<T> → مش بياخد حاجة، بيرجع T
Supplier<String> greeting = () -> "Hello, World!";
Supplier<Double> random = () -> Math.random();

System.out.println(greeting.get()); // Hello, World!
System.out.println(random.get());   // 0.xxxxx

// 5. BiFunction<T, U, R> → بياخد اتنين، بيرجع واحد
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
System.out.println(repeat.apply("Java ", 3)); // Java Java Java
```

---

## 3.3 Streams API ⭐ (الأهم في Java 8)

**الفكرة:** معالجة collections بشكل functional وأنيق.

```
Source → [filter] → [map] → [collect]
Data   → تصفية   → تحويل → نتيجة
```

**المصادر:**

```java
// من Collection
Stream<String> s1 = list.stream();

// من Array
Stream<String> s2 = Arrays.stream(array);

// إنشاء مباشر
Stream<String> s3 = Stream.of("a", "b", "c");

// Infinite Stream
Stream<Integer> s4 = Stream.iterate(0, n -> n + 1);
```

### Intermediate Operations (بترجع Stream - Lazy)

```java
List<String> names = List.of("Ahmed", "Sara", "Mohamed", "Ali", "Khaled", "Aya");

// filter - تصفية
names.stream()
     .filter(name -> name.length() > 3)
     .forEach(System.out::println);
// Ahmed, Sara, Mohamed, Khaled

// map - تحويل
names.stream()
     .map(String::toUpperCase)
     .forEach(System.out::println);
// AHMED, SARA, MOHAMED, ALI, KHALED, AYA

// sorted
names.stream()
     .sorted()
     .forEach(System.out::println);
// Ahmed, Ali, Aya, Khaled, Mohamed, Sara

// distinct - إزالة التكرار
List<Integer> nums = List.of(1, 2, 2, 3, 3, 3, 4);
nums.stream()
    .distinct()
    .forEach(System.out::print);
// 1234

// limit & skip
Stream.iterate(1, n -> n + 1)
      .skip(5)    // اتخطى أول 5
      .limit(5)   // خد 5 بس
      .forEach(System.out::print);
// 6 7 8 9 10

// flatMap - لما عندك Stream of Streams
List<List<Integer>> nested = List.of(
    List.of(1, 2, 3),
    List.of(4, 5, 6),
    List.of(7, 8, 9)
);

nested.stream()
      .flatMap(List::stream)
      .forEach(System.out::print);
// 123456789
```

### Terminal Operations (بتنهي الـ Stream)

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// collect - الأهم
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(evenNumbers); // [2, 4, 6, 8, 10]

// collect to Set
Set<Integer> evenSet = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toSet());

// collect to Map
List<String> namesList = List.of("Ahmed", "Sara", "Mohamed");
Map<String, Integer> nameLengths = namesList.stream()
    .collect(Collectors.toMap(
        name -> name,           // key
        name -> name.length()   // value
    ));
System.out.println(nameLengths); // {Ahmed=5, Sara=4, Mohamed=7}

// joining
String joined = namesList.stream()
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(joined); // [Ahmed, Sara, Mohamed]

// count
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();
System.out.println(count); // 5

// reduce - تجميع
int sum = numbers.stream()
    .reduce(0, Integer::sum);
System.out.println(sum); // 55

int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
System.out.println(product); // 3628800

// min & max
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
System.out.println(max.get()); // 10
System.out.println(min.get()); // 1

// anyMatch, allMatch, noneMatch
boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);    // true
boolean allPositive = numbers.stream().allMatch(n -> n > 0);     // true
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);   // true

// findFirst & findAny
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();
System.out.println(first.get()); // 6

// forEach
numbers.stream().forEach(System.out::println);
```

### Collectors المتقدمة

```java
List<Product> products = List.of(
    new Product("Laptop", "Electronics", 1500.0),
    new Product("Phone", "Electronics", 800.0),
    new Product("Shirt", "Clothing", 50.0),
    new Product("Pants", "Clothing", 70.0),
    new Product("TV", "Electronics", 1200.0)
);

// groupingBy - تجميع بمعيار
Map<String, List<Product>> byCategory = products.stream()
    .collect(Collectors.groupingBy(Product::getCategory));
// {Electronics=[Laptop, Phone, TV], Clothing=[Shirt, Pants]}

// counting per group
Map<String, Long> countByCategory = products.stream()
    .collect(Collectors.groupingBy(
        Product::getCategory,
        Collectors.counting()
    ));
// {Electronics=3, Clothing=2}

// averagingDouble per group
Map<String, Double> avgPriceByCategory = products.stream()
    .collect(Collectors.groupingBy(
        Product::getCategory,
        Collectors.averagingDouble(Product::getPrice)
    ));
// {Electronics=1166.67, Clothing=60.0}

// partitioningBy - تقسيم لـ true/false
Map<Boolean, List<Product>> expensive = products.stream()
    .collect(Collectors.partitioningBy(p -> p.getPrice() > 500));
// {true=[Laptop, Phone, TV], false=[Shirt, Pants]}

// summarizingDouble
DoubleSummaryStatistics stats = products.stream()
    .collect(Collectors.summarizingDouble(Product::getPrice));
System.out.println("Sum: " + stats.getSum());
System.out.println("Avg: " + stats.getAverage());
System.out.println("Max: " + stats.getMax());
System.out.println("Min: " + stats.getMin());
```

---

## 3.4 Optional ⭐

**الفكرة:** wrapper يتجنب NullPointerException.

```java
// المشكلة القديمة
public User findUser(int id) {
    return null; // ← NullPointerException في أي حتة!
}

User user = findUser(5);
System.out.println(user.getName()); // 💥 NullPointerException

// الحل مع Optional
public Optional<User> findUser(int id) {
    // لو موجود
    if (id == 1) return Optional.of(new User("أحمد", "ahmed@gmail.com", 25));
    // لو مش موجود
    return Optional.empty();
}
```

**الاستخدام:**

```java
Optional<User> userOpt = findUser(1);

// isPresent & get (مش الأفضل)
if (userOpt.isPresent()) {
    System.out.println(userOpt.get().getName());
}

// ifPresent - أحسن
userOpt.ifPresent(user -> System.out.println(user.getName()));

// orElse - قيمة افتراضية لو مش موجود
User user = userOpt.orElse(new User("Unknown", "", 0));

// orElseGet - بيتنفذ لو مش موجود بس (Lazy)
User user2 = userOpt.orElseGet(() -> createDefaultUser());

// orElseThrow - throw exception لو مش موجود
User user3 = userOpt.orElseThrow(() ->
    new RuntimeException("User not found"));

// map - تحويل لو موجود
Optional<String> name = userOpt.map(User::getName);
Optional<String> upperName = userOpt.map(User::getName)
                                    .map(String::toUpperCase);
System.out.println(upperName.orElse("N/A")); // AHMED

// filter - تصفية
Optional<User> activeUser = userOpt.filter(u -> u.getAge() >= 18);

// flatMap - لو الـ method بترجع Optional
Optional<String> email = userOpt.flatMap(u -> Optional.ofNullable(u.getEmail()));
```

**في Spring - Repository مثال:**

```java
// Spring Repository
Optional<User> user = userRepository.findById(id);

User result = user.orElseThrow(() ->
    new EntityNotFoundException("User with id " + id + " not found"));
```

---

## 3.5 Method References

```java
// 4 أنواع:

// 1. Static Method Reference: ClassName::staticMethod
Function<String, Integer> parseInt = Integer::parseInt;
System.out.println(parseInt.apply("42")); // 42

// 2. Instance Method Reference (على object معين): instance::method
String prefix = "Hello, ";
Function<String, String> greet = prefix::concat;
System.out.println(greet.apply("World")); // Hello, World

// 3. Instance Method Reference (على أي object): ClassName::instanceMethod
Function<String, String> toUpper = String::toUpperCase;
Function<String, Integer> length = String::length;
System.out.println(toUpper.apply("java")); // JAVA

// 4. Constructor Reference: ClassName::new
Supplier<ArrayList<String>> listFactory = ArrayList::new;
ArrayList<String> newList = listFactory.get();

Function<String, StringBuilder> sbFactory = StringBuilder::new;
StringBuilder sb = sbFactory.apply("Hello");
```

---

## 3.6 Default & Static Methods في Interface

```java
public interface Logger {
    void log(String message);  // abstract

    // Default method - بيجي مع implementation
    default void logError(String message) {
        log("ERROR: " + message);
    }

    default void logInfo(String message) {
        log("INFO: " + message);
    }

    // Static method
    static Logger consoleLogger() {
        return message -> System.out.println("[LOG] " + message);
    }
}

public class FileLogger implements Logger {
    @Override
    public void log(String message) {
        // كتابة في file
        System.out.println("Writing to file: " + message);
    }
    // مش محتاج implement logError و logInfo - بيورثهم
}

FileLogger logger = new FileLogger();
logger.log("Custom message");       // Writing to file: Custom message
logger.logError("Something wrong"); // Writing to file: ERROR: Something wrong
logger.logInfo("Started");          // Writing to file: INFO: Started

Logger consoleLogger = Logger.consoleLogger();
consoleLogger.log("Hello"); // [LOG] Hello
```

---

## 3.7 Comparable vs Comparator

```java
// Comparable - الـ class نفسها بتعرف ترتيب نفسها
public class Student implements Comparable<Student> {
    private String name;
    private double gpa;

    public Student(String name, double gpa) {
        this.name = name;
        this.gpa = gpa;
    }

    @Override
    public int compareTo(Student other) {
        // ترتيب حسب الـ GPA تنازلياً
        return Double.compare(other.gpa, this.gpa);
    }
}

List<Student> students = new ArrayList<>(List.of(
    new Student("أحمد", 3.5),
    new Student("سارة", 3.9),
    new Student("محمد", 3.2)
));

Collections.sort(students); // بيستخدم compareTo
// سارة 3.9, أحمد 3.5, محمد 3.2

// Comparator - ترتيب خارجي مرن
Comparator<Student> byName = Comparator.comparing(s -> s.name);
Comparator<Student> byGPA = Comparator.comparingDouble(s -> s.gpa);
Comparator<Student> byGPADesc = Comparator.comparingDouble(Student::getGpa).reversed();

students.sort(byName);      // أبجدي
students.sort(byGPADesc);   // أعلى GPA الأول

// ترتيب مركب
students.sort(Comparator.comparingDouble(Student::getGpa)
                        .reversed()
                        .thenComparing(Student::getName));
```

---

# 📌 PART 4: Exception Handling

---

## 4.1 أنواع الـ Exceptions

```
Throwable
├── Error (JVM مشكلة - مش هتتعامل معاها)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── Checked Exceptions (لازم تتعامل معاها)
    │   ├── IOException
    │   ├── SQLException
    │   └── FileNotFoundException
    └── RuntimeException (Unchecked)
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── ClassCastException
        └── IllegalArgumentException
```

---

## 4.2 Try-Catch-Finally

```java
public void readFile(String path) {
    FileReader reader = null;
    try {
        reader = new FileReader(path);
        // قراءة
        int data = reader.read();
        System.out.println("Data: " + data);

    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + path);
    } catch (IOException e) {
        System.err.println("Error reading file: " + e.getMessage());
    } catch (Exception e) {
        System.err.println("Unexpected error: " + e.getMessage());
    } finally {
        // بيتنفذ دايماً - سواء حصل exception أو لأ
        if (reader != null) {
            try { reader.close(); } catch (IOException e) { }
        }
    }
}

// أحسن - Try-with-resources (Java 7+)
public void readFileBetter(String path) {
    try (FileReader reader = new FileReader(path)) {
        int data = reader.read();
        System.out.println("Data: " + data);
    } catch (IOException e) {
        System.err.println("Error: " + e.getMessage());
    }
    // reader بيتغلق تلقائياً
}
```

---

## 4.3 Custom Exceptions

```java
// Checked Exception
public class InsufficientFundsException extends Exception {
    private double amount;

    public InsufficientFundsException(double amount) {
        super("Insufficient funds. Need: " + amount);
        this.amount = amount;
    }

    public double getAmount() { return amount; }
}

// Unchecked Exception
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}

// الاستخدام
public User getUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

// في Spring هتلاقيها كتير
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

---

# 📌 PART 5: String & Immutability

---

## 5.1 String Pool و Immutability

```java
// String immutable - بيتخزن في String Pool
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

System.out.println(s1 == s2);       // true (نفس الـ object في Pool)
System.out.println(s1 == s3);       // false (object جديد في Heap)
System.out.println(s1.equals(s3));  // true (نفس الـ content)

// دايماً استخدم equals() مع Strings!

// String مش بتتغير
String original = "Hello";
String modified = original.toUpperCase();
System.out.println(original); // Hello (ما اتغيرتش)
System.out.println(modified); // HELLO (object جديد)
```

---

## 5.2 StringBuilder vs String

```java
// String concatenation في loop - بطيء جداً!
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i;  // بيعمل object جديد كل مرة!
}

// StringBuilder - أسرع بكتير
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String result2 = sb.toString();

// String Methods المهمة
String text = "  Hello, Java World!  ";
text.trim();                          // "Hello, Java World!"
text.strip();                         // نفس trim بس أحدث
text.toLowerCase();                   // "  hello, java world!  "
text.toUpperCase();                   // "  HELLO, JAVA WORLD!  "
text.replace("Java", "Python");       // تبديل
text.contains("Java");                // true
text.startsWith("Hello");            // false (فيه مسافات)
text.strip().startsWith("Hello");    // true
text.split(",");                      // ["  Hello", " Java World!  "]
text.substring(2, 7);                 // "Hello"
text.indexOf("Java");                // 9
text.isEmpty();                       // false
text.isBlank();                       // false
"".isEmpty();                         // true
"   ".isBlank();                      // true (Java 11+)

// String.format
String formatted = String.format("Name: %s, Age: %d, GPA: %.2f",
    "أحمد", 25, 3.567);
System.out.println(formatted); // Name: أحمد, Age: 25, GPA: 3.57

// Java 15+ - Text Blocks
String json = """
    {
        "name": "أحمد",
        "age": 25
    }
    """;
```

---

# 📌 PART 6: Enums & Records

---

## 6.1 Enum

```java
public enum OrderStatus {
    PENDING("قيد الانتظار"),
    PROCESSING("قيد المعالجة"),
    SHIPPED("تم الشحن"),
    DELIVERED("تم التسليم"),
    CANCELLED("ملغي");

    private final String arabicName;

    OrderStatus(String arabicName) {
        this.arabicName = arabicName;
    }

    public String getArabicName() {
        return arabicName;
    }

    public boolean isTerminal() {
        return this == DELIVERED || this == CANCELLED;
    }
}

// الاستخدام
OrderStatus status = OrderStatus.SHIPPED;
System.out.println(status.getArabicName()); // تم الشحن
System.out.println(status.isTerminal());    // false

// في Switch
switch (status) {
    case PENDING -> System.out.println("Waiting...");
    case SHIPPED -> System.out.println("On the way!");
    case DELIVERED -> System.out.println("Done!");
    default -> System.out.println("Other status");
}
```

---

## 6.2 Record (Java 16+)

```java
// قبل Record - Boilerplate كتير
public class UserDTO {
    private final String name;
    private final String email;

    public UserDTO(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }

    @Override public boolean equals(Object o) { /* ... */ }
    @Override public int hashCode() { /* ... */ }
    @Override public String toString() { /* ... */ }
}

// مع Record - نفس الحاجة في سطر واحد!
public record UserDTO(String name, String email) {}

// بيجي مجاناً: constructor, getters, equals, hashCode, toString
UserDTO dto = new UserDTO("أحمد", "ahmed@gmail.com");
System.out.println(dto.name());    // أحمد
System.out.println(dto.email());   // ahmed@gmail.com
System.out.println(dto);           // UserDTO[name=أحمد, email=ahmed@gmail.com]
```

---

# 📌 PART 7: كل حاجة تانية مهمة

---

## 7.1 Varargs

```java
public static int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) total += n;
    return total;
}

System.out.println(sum(1, 2, 3));       // 6
System.out.println(sum(1, 2, 3, 4, 5)); // 15
System.out.println(sum());              // 0
```

---

## 7.2 Autoboxing & Unboxing

```java
// Autoboxing - primitive → Wrapper
Integer i = 42;          // int → Integer
Double d = 3.14;         // double → Double
Boolean b = true;        // boolean → Boolean

// Unboxing - Wrapper → primitive
int primitive = i;        // Integer → int

// ⚠️ تحذير
Integer a = 127;
Integer b2 = 127;
System.out.println(a == b2); // true (مكاش من -128 لـ 127)

Integer c = 128;
Integer d2 = 128;
System.out.println(c == d2); // false! ← استخدم equals دايماً
System.out.println(c.equals(d2)); // true ✅
```

---

## 7.3 Sealed Classes (Java 17+)

```java
// بتحدد مين مسموح يرث
public sealed class Shape
    permits Circle, Rectangle, Triangle {
    public abstract double area();
}

public final class Circle extends Shape {
    private double radius;
    public Circle(double radius) { this.radius = radius; }

    @Override
    public double area() { return Math.PI * radius * radius; }
}

public final class Rectangle extends Shape {
    private double width, height;
    public Rectangle(double w, double h) { this.width = w; this.height = h; }

    @Override
    public double area() { return width * height; }
}

// Pattern Matching مع Sealed Classes
public String describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with area: " + c.area();
        case Rectangle r -> "Rectangle with area: " + r.area();
        case Triangle t -> "Triangle with area: " + t.area();
    };
}
```

---

## 7.4 Pattern Matching (Java 16+)

```java
// قبل
Object obj = "Hello";
if (obj instanceof String) {
    String s = (String) obj;  // casting يدوي
    System.out.println(s.length());
}

// بعد - Pattern Matching
if (obj instanceof String s) {
    System.out.println(s.length()); // s متاحة هنا مباشرة
}

// مع switch (Java 21)
Object value = 42;
String result = switch (value) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case null -> "null value";
    default -> "Unknown type";
};
```

---

# 🗺️ Roadmap - ترتيب المواضيع للـ Spring Developer

```
الأساس (اتعلمته دلوقتي) ✅
├── OOP كامل
├── Collections & Generics
├── Java 8+ (Streams, Lambda, Optional)
├── Exception Handling
└── String & Records

الخطوة الجاية 🎯
├── Multithreading & Concurrency
│   ├── Thread basics
│   ├── synchronized & volatile
│   ├── ExecutorService
│   └── CompletableFuture ← مهم جداً في Spring
├── Reflection & Annotations
│   └── الـ Spring بيشتغل عليهم كلهم
├── Design Patterns
│   ├── Singleton
│   ├── Factory
│   ├── Builder ← بتشوفه مع Lombok
│   ├── Strategy
│   └── Observer
└── Spring Framework
    ├── IoC & Dependency Injection
    ├── Spring MVC
    ├── Spring Data JPA
    └── Spring Security
```

---

عايز أكمل على أي topic من دول؟ **Multithreading** أو **Design Patterns** أو تبدأ مباشرة في **Spring**؟ 🚀