أيوه. والجزء اللي بعته ده مهم جدًا لأنه بينقلك من مجرد معرفة **class / object / variable / method** لفهم: **إزاي الـ Object بيتصرف حسب الـ State بتاعه، وإزاي الـ Methods ممكن تغيّر الـ State**.

هشرحهولك كأننا ماشيين في الجزء ده من **Head First Java**، من غير ما نسيب نقطة من اللي ذكرتها.

---

# 1. How Objects Behave

الجملة الأساسية في الجزء ده:

> **State affects behavior, behavior affects state.**

يعني:

```text
        State
          ↓
      Behavior
          ↓
     changes State
```

الـ Object عنده حاجتين:

### State

الحالة الحالية للـ object.

بتمثلها غالبًا:

```java
instance variables
```

### Behavior

الحاجات اللي الـ object يقدر يعملها.

بتمثلها:

```java
methods
```

مثال:

```java
class Dog {

    int weight;

    void bark() {
        if (weight < 14) {
            System.out.println("Yip yip!");
        } else {
            System.out.println("Woof!");
        }
    }
}
```

هنا:

```java
weight
```

دي **State**.

و:

```java
bark()
```

ده **Behavior**.

---

# 2. State affects Behavior

بص على المثال:

```java
Dog d = new Dog();

d.weight = 10;
d.bark();
```

الناتج:

```text
Yip yip!
```

ليه؟

لأن:

```text
weight = 10
```

والـ method عملت:

```java
if (weight < 14)
```

---

لكن:

```java
d.weight = 20;
d.bark();
```

الناتج:

```text
Woof!
```

إذن **نفس الـ object ونفس الـ method** لكن الـ behavior اختلف بسبب الـ state.

```text
weight = 10
     ↓
bark()
     ↓
Yip yip!


weight = 20
     ↓
bark()
     ↓
Woof!
```

وده المقصود بـ:

> **An object's behavior uses an object's unique state.**

---

# 3. Behavior affects State



الموضوع مش اتجاه واحد.

الـ method نفسها ممكن تغير الـ state.

مثلاً:

```java
class Dog {

    int weight;

    void eat() {
        weight++;
    }
}
```

عندنا:

```java
Dog d = new Dog();

d.weight = 10;

System.out.println(d.weight);

d.eat();

System.out.println(d.weight);
```

الناتج:

```text
10
11
```

يعني:

```text
State
weight = 10

      ↓
   eat()

      ↓

State
weight = 11
```

إذن:

> **Behavior affects state.**

---

# 4. Declaring a Variable

لما تكتب:

```java
int x;
```

أنت بتقول للـ Java:

> اعمل variable اسمه `x` ونوعه `int`.

لكن لسه **مفيش قيمة اتخزنت فيه**.

مثلاً:

```java
int x;
```

بعدها:

```java
x = 10;
```

دلوقتي:

```text
x
↓
10
```

ممكن تعمل الاتنين مع بعض:

```java
int x = 10;
```

وده اسمه:

**Declaration + Initialization**

---

# 5. "I'd like a double mocha, no, make it an int."

دي طريقة Head First في شرح **type**.

لما تقول:

```java
int x;
```

أنت حددت إن `x` هيخزن integer.

مينفعش تعمل:

```java
int x = 10.5;
```

لأن:

```text
10.5 → double
```

مش `int`.

لكن:

```java
double x = 10.5;
```

صح.

---

## Primitive Types

Java عندها 8 primitive types:

```text
byte
short
int
long
float
double
char
boolean
```

مثلاً:

```java
int age = 23;
double salary = 15000.5;
boolean active = true;
char grade = 'A';
```

---

# 6. Back away from that keyword!

دي نقطة مهمة جدًا.

في Java عندك:

```java
new
```

لما تستخدم:

```java
Dog d = new Dog();
```

فيه حاجتين حصلوا:

### 1. Reference variable

```java
Dog d
```

عملنا variable اسمه `d` نوعه `Dog`.

### 2. Object

```java
new Dog()
```

أنشأنا object جديد.

إذن:

```java
Dog d = new Dog();
```

ممكن نفصلها:

```java
Dog d;
d = new Dog();
```

---

# 7. Object Reference

دي من أهم أفكار الكتاب.

لما تعمل:

```java
Dog d = new Dog();
```

مش معنى كده إن:

```text
d = Dog Object
```

بالمعنى الحرفي.

الأدق:

```text
d
│
│ reference
▼
┌─────────────┐
│   Dog       │
│             │
│ weight = 10 │
└─────────────┘
```

`d` عبارة عن **reference** بيشير للـ object.

---

# 8. An Object Reference is Just Another Variable Value

دي النقطة اللي الكتاب عايز يثبتها.

إحنا متعودين:

```java
int x = 7;
```

يعني:

```text
x
↓
7
```

لكن ممكن variable يحتوي **reference**:

```java
Dog d = new Dog();
```

يعني:

```text
d
↓
reference
↓
Dog object
```

فالـ reference نفسه عبارة عن **value** للـ variable.

---

# 9. Copying a Variable

الجزء اللي عندك:

```text
00000111 int X
00000111 int Z
```

بيشرح إن لو عندك:

```java
int x = 7;
int z = x;
```

فالقيمة بتتنسخ:

```text
x = 7

z = x

x = 7
z = 7
```

لكن الاتنين مستقلين.

لو:

```java
z = 20;
```

يبقى:

```text
x = 7
z = 20
```

---

# 10. But Objects Are Different

بص:

```java
Dog x = new Dog();
Dog z = x;
```

هنا **مش بنعمل Dog جديد**.

عندنا object واحد:

```text
x ───────┐
         │
         ▼
     ┌─────────┐
     │  Dog    │
     │ weight  │
     └─────────┘
         ▲
         │
z ───────┘
```

يعني:

```java
x == z
```

تساوي:

```java
true
```

لأن الاتنين بيشاوروا على نفس object.

---

# 11. foo.go(x)

مثلاً:

```java
void foo(Dog x) {
    x.weight = 20;
}
```

واستخدمناها:

```java
Dog d = new Dog();

d.weight = 10;

foo(d);

System.out.println(d.weight);
```

الناتج:

```text
20
```

ليه؟

لأن reference للـ Dog اتبعت للـ method.

الـ parameter:

```java
x
```

بقى reference لنفس الـ object.

```text
d ───────┐
         ↓
      Dog Object
         ↑
         │
x ───────┘
```

فلما:

```java
x.weight = 20;
```

أنت بتعدل نفس الـ object.

---

# 12. `void go(int z)`

مثلاً:

```java
void go(int z) {
    z = 20;
}
```

وعندنا:

```java
int x = 10;

go(x);

System.out.println(x);
```

الناتج:

```text
10
```

ليه؟

لأن الـ primitive اتبعت **بالقيمة**.

```text
x = 10

      ↓ copy

z = 10
```

بعد:

```java
z = 20;
```

بقى:

```text
x = 10
z = 20
```

---

# 13. Java Passes Arguments by Value

دي نقطة Interview مهمة جدًا:

> **Java is always pass-by-value.**

لكن مع الـ objects، الـ value اللي بتتنسخ هي **reference value**.

مثال:

```java
void change(Dog d) {
    d.weight = 50;
}
```

ده يقدر يعدل الـ object.

لكن:

```java
void change(Dog d) {
    d = new Dog();
    d.weight = 50;
}
```

ده مش هيغير reference الأصلي عند الـ caller.

مثلاً:

```java
Dog dog = new Dog();
dog.weight = 10;

change(dog);

System.out.println(dog.weight);
```

هيفضل:

```text
10
```

لأن:

```text
caller dog ─────→ Object A

inside method:

d ─────→ Object B
```

إنت غيرت نسخة الـ reference، مش الـ reference الأصلي.

---

# 14. Life on the Garbage-Collectible Heap

لما تعمل:

```java
Dog d = new Dog();
```

الـ object بيتعمل في الـ **heap**.

تقريبًا:

```text
Stack
┌───────────┐
│ d         │
│ reference │
└─────┬─────┘
      │
      ▼
Heap
┌──────────────┐
│ Dog Object   │
│ weight = 10  │
└──────────────┘
```

لما ميبقاش فيه أي reference للـ object:

```text
Dog object
    ↑
    │
   no references
```

يبقى object أصبح **eligible for garbage collection**.

مثلاً:

```java
Dog d = new Dog();

d = null;
```

دلوقتي الـ Dog القديم مفيش حاجة بتشاور عليه.

مش معنى كده إنه اتمسح فورًا.

لكن أصبح:

> **eligible for garbage collection**

والـ JVM / Garbage Collector هو اللي بيتولى تنظيف الـ heap.

---

# 15. Arrays — A Tray of Cups

الكتاب بيشبه الـ array بـ **tray of cups**.

لو:

```java
int[] nums = new int[4];
```

عندك 4 أماكن:

```text
index:

0     1     2     3
┌───┬───┬───┬───┐
│   │   │   │   │
└───┴───┴───┴───┘
```

مثلاً:

```java
nums[0] = 10;
nums[1] = 20;
nums[2] = 30;
nums[3] = 40;
```

---

# 16. Array of Objects

مهم جدًا.

لما تعمل:

```java
Dog[] dogs = new Dog[3];
```

أنت **مش عملت 3 Dogs**.

أنت عملت array فيها 3 references.

```text
dogs

0       1       2
┌──────┬──────┬──────┐
│ null │ null │ null │
└──────┴──────┴──────┘
```

لازم تعمل الـ objects:

```java
dogs[0] = new Dog();
dogs[1] = new Dog();
dogs[2] = new Dog();
```

دلوقتي:

```text
dogs
 │
 ├── 0 ─────→ Dog A
 │
 ├── 1 ─────→ Dog B
 │
 └── 2 ─────→ Dog C
```

---

# 17. Dog Example

خلينا نجمع الكلام كله:

```java
class Dog {

    int size;

    void bark() {
        if (size > 60) {
            System.out.println("Woof!");
        } else if (size > 14) {
            System.out.println("Ruff!");
        } else {
            System.out.println("Yip!");
        }
    }

    void eat() {
        size++;
    }
}
```

استخدامه:

```java
Dog d = new Dog();

d.size = 10;

d.bark();
```

الناتج:

```text
Yip!
```

بعد:

```java
d.size = 30;
d.bark();
```

الناتج:

```text
Ruff!
```

وبعد:

```java
d.size = 70;
d.bark();
```

الناتج:

```text
Woof!
```

لاحظ:

```text
size
 ↓
affects
 ↓
bark behavior
```

---

# 18. A Class Describes What an Object Knows and Does

دي جملة مهمة جدًا من الكتاب:

> **A class describes what an object knows and what an object does.**

يعني:

### What an object knows

= variables

```java
int size;
String name;
```

### What an object does

= methods

```java
bark();
eat();
run();
```

مثلاً:

```java
class Dog {

    String name;  // knows

    int size;     // knows

    void bark() { // does
        System.out.println("Woof");
    }

    void eat() {  // does
        size++;
    }
}
```

---

# 19. The Size Affects the Bark

دي نقطة مهمة في المثال.

```java
void bark() {

    if (size > 60) {
        System.out.println("Woof!");
    } else if (size > 14) {
        System.out.println("Ruff!");
    } else {
        System.out.println("Yip!");
    }
}
```

لاحظ إن:

```java
bark()
```

معندهاش parameter اسمه `size`.

لكنها تقدر تستخدم:

```java
size
```

ليه؟

لأن `size` **instance variable** للـ object.

يعني:

```java
Dog d1 = new Dog();
d1.size = 10;

Dog d2 = new Dog();
d2.size = 70;
```

لما:

```java
d1.bark();
```

يستخدم:

```text
d1.size = 10
```

ولما:

```java
d2.bark();
```

يستخدم:

```text
d2.size = 70
```

نفس الـ class.

نفس الـ method.

لكن كل object عنده **state مختلف**.

---

# 20. You Can Send Things to a Method

يعني الـ method ممكن تاخد parameters.

مثلاً:

```java
void bark(int times) {
    for (int i = 0; i < times; i++) {
        System.out.println("Woof");
    }
}
```

تستخدمها:

```java
dog.bark(3);
```

أنت بعت:

```text
3
```

للـ method.

---

# 21. Parameter vs Argument

مثلاً:

```java
void bark(int times) {
}
```

`times` هنا:

> Parameter

لكن:

```java
dog.bark(3);
```

`3` هنا:

> Argument

يعني:

```text
Parameter = variable defined in method
Argument = actual value passed to method
```

---

# 22. You Can Get Things Back From a Method

مش لازم الـ method تكون:

```java
void
```

ممكن ترجع value.

مثلاً:

```java
int getSize() {
    return size;
}
```

استخدامها:

```java
int x = dog.getSize();
```

هنا:

```text
dog.getSize()
      ↓
returns int
      ↓
x
```

---

# 23. `return`

مثلاً:

```java
int add(int x, int y) {
    return x + y;
}
```

استخدام:

```java
int result = add(10, 20);
```

الناتج:

```text
result = 30
```

والـ return type هنا:

```java
int
```

لأن method بترجع `int`.

---

# 24. You Can Send More Than One Thing

الـ method ممكن تاخد أكتر من parameter:

```java
void move(int x, int y) {
    System.out.println(x);
    System.out.println(y);
}
```

استخدام:

```java
move(10, 20);
```

عندنا:

```text
x = 10
y = 20
```

---

# 25. Parameters Can Have Different Types

مثلاً:

```java
void createUser(String name, int age, boolean active) {

}
```

تقدر تعمل:

```java
createUser("Ali", 23, true);
```

يبقى:

```text
name   = "Ali"
age    = 23
active = true
```

---

# 26. Cool Things You Can Do With Parameters and Return Types

ممكن تعمل method تاخد data وترجع data.

مثلاً:

```java
int calculateAge(int birthYear, int currentYear) {
    return currentYear - birthYear;
}
```

استخدام:

```java
int age = calculateAge(2003, 2026);
```

النتيجة:

```text
23
```

ممكن كمان:

```java
Dog createDog(String name) {

    Dog d = new Dog();
    d.name = name;

    return d;
}
```

وبالتالي:

```java
Dog dog = createDog("Rocky");
```

الـ method رجعت **object reference**.

---

# 27. Encapsulation

دي من أهم أفكار OOP.

المشكلة إنك لو خليت الـ state public:

```java
class BankAccount {

    public double balance;
}
```

أي حد يقدر يعمل:

```java
account.balance = -100000;
```

وده ممكن يبقى غلط.

الأفضل:

```java
class BankAccount {

    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

دلوقتي:

```java
account.deposit(1000);
```

لكن مينفعش:

```java
account.balance = -1000;
```

لأن:

```java
balance
```

`private`.

---

# 28. Encapsulation مش معناها Getters/Setters وخلاص

دي نقطة مهمة في الانترفيو.

Encapsulation معناها إنك:

> **تجمع الـ state والbehavior المرتبطين ببعض داخل object، وتتحكم في الوصول للـ internal state.**

يعني بدل:

```java
account.balance = -100;
```

تخلي الـ object هو اللي يتحكم:

```java
account.withdraw(100);
```

وهو يقرر هل العملية valid ولا لأ.

---

# 29. How Do Objects in an Array Behave?

مثلاً:

```java
Dog[] dogs = new Dog[3];

dogs[0] = new Dog();
dogs[1] = new Dog();
dogs[2] = new Dog();
```

كل عنصر عبارة عن reference لـ object مختلف.

```text
dogs
 │
 ├── [0] ───→ Dog A
 │
 ├── [1] ───→ Dog B
 │
 └── [2] ───→ Dog C
```

تقدر تعمل:

```java
dogs[0].size = 10;
dogs[1].size = 30;
dogs[2].size = 70;
```

وبعدين:

```java
for (Dog dog : dogs) {
    dog.bark();
}
```

كل Dog هيستخدم الـ state الخاص بيه.

الناتج:

```text
Yip!
Ruff!
Woof!
```

---

# 30. Declaring and Initializing Instance Variables

مثلاً:

```java
class Dog {

    String name;
    int size;
    boolean hungry;
}
```

دي instance variables.

لو عملت:

```java
Dog d = new Dog();
```

Java بتحط default values.

مثلاً:

```text
int      → 0
double   → 0.0
boolean  → false
reference → null
```

يعني:

```java
class Dog {

    String name;
    int size;
    boolean hungry;
}
```

بعد:

```java
Dog d = new Dog();
```

تقريبًا:

```text
name    = null
size    = 0
hungry  = false
```

---

# 31. Instance Variables vs Local Variables

دي مهمة جدًا.

### Instance variable

```java
class Dog {

    int size;
}
```

الـ JVM بتديها default value.

---

### Local variable

```java
void test() {

    int x;

    System.out.println(x);
}
```

ده **Compile Error**.

لأن local variables لازم يتم initialization قبل استخدامها:

```java
int x = 10;
```

---

# 32. Comparing Variables

عندك نوعين أساسيين:

```text
Primitive
Reference
```

والـ comparison بينهم مختلف.

---

# 33. Comparing Primitives

مثلاً:

```java
int x = 10;
int y = 10;

System.out.println(x == y);
```

الناتج:

```text
true
```

لأن `==` مع primitive بتقارن **القيم**.

مثلاً:

```java
int x = 10;
int y = 20;

x == y
```

→ `false`

---

# 34. Comparing References

دي مهمة جدًا.

```java
Dog d1 = new Dog();
Dog d2 = new Dog();
```

لما تعمل:

```java
d1 == d2
```

النتيجة:

```text
false
```

ليه؟

لأن عندك objectين مختلفين:

```text
d1 ─────→ Dog A

d2 ─────→ Dog B
```

حتى لو الـ objects عندهم نفس البيانات.

---

# 35. `==` With References

`==` مع references بتسأل:

> هل الاتنين بيشاوروا على **نفس object**؟

مثلاً:

```java
Dog d1 = new Dog();
Dog d2 = d1;
```

دلوقتي:

```text
d1 ─────┐
        ↓
      Dog A
        ↑
d2 ─────┘
```

إذن:

```java
d1 == d2
```

→ `true`

---

# 36. `equals()`

لو عايز تقارن **logical/content equality** غالبًا تستخدم:

```java
equals()
```

مثلاً:

```java
String s1 = new String("Ali");
String s2 = new String("Ali");
```

هنا:

```java
s1 == s2
```

غالبًا:

```text
false
```

لكن:

```java
s1.equals(s2)
```

→

```text
true
```

لأن `String` عملت override لـ `equals()` بحيث تقارن المحتوى.

---

# 37. الصورة الكبيرة للجزء كله

خلينا نربط كل الكلام:

```text
                    CLASS
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
       STATE                   BEHAVIOR
          │                       │
          ↓                       ↓
 Instance Variables            Methods
          │                       │
          └──────────┬────────────┘
                     ↓
                  OBJECT
                     │
                     ↓
          Behavior uses State
                     │
                     ↓
            Behavior can change
                     │
                     ↓
                   State
```

مثال:

```java
class Dog {

    private int size;

    void bark() {
        if (size < 14) {
            System.out.println("Yip!");
        } else {
            System.out.println("Woof!");
        }
    }

    void eat() {
        size++;
    }
}
```

هنا:

```text
size
 ↓
State
```

و:

```text
bark()
eat()
 ↓
Behavior
```

`bark()`:

```text
State → Behavior
```

و`eat()`:

```text
Behavior → State
```

---

# 38. أهم نقطة في الجزء كله

لو داخل انترفيو واتسألت:

### What is the relationship between state and behavior?

قول:

> **An object's state is represented by its instance variables, while its behavior is represented by its methods. The object's state can affect how its methods behave, and its methods can also change the object's state.**

مثال:

```java
class Dog {

    int size;

    void bark() {
        if (size < 14) {
            System.out.println("Yip!");
        } else {
            System.out.println("Woof!");
        }
    }

    void eat() {
        size++;
    }
}
```

هنا:

- `size` → state
    
- `bark()` → behavior يعتمد على state
    
- `eat()` → behavior يغير state
    

---

# 🔥 خلاصة الجزء اللي تذاكره

احفظ العلاقات دي:

```text
Class
 ↓
Blueprint

Object
 ↓
Instance of a class

State
 ↓
Instance variables

Behavior
 ↓
Methods

Reference
 ↓
Value that points to an object

new
 ↓
Creates an object

Array of objects
 ↓
Array of references

Java
 ↓
Always pass-by-value

Primitive == 
 ↓
Compare values

Reference ==
 ↓
Compare object identity

equals()
 ↓
Logical equality (depending on implementation)

Encapsulation
 ↓
Protect/control object state
```

وأهم 3 جمل من الجزء ده:

> **A class describes what an object knows and what an object does.**

> **State affects behavior.**

> **Behavior affects state.**

لو فهمت التلاتة دول كويس، هتكون فهمت أساس الـ **Object-Oriented Programming** اللي الكتاب بيبني عليه بعد كده.