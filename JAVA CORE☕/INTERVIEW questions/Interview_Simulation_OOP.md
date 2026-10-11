ك
في اولاتمام، دلوقتي إحنا في **Java Backend Interview**، وأنا الـ Interviewer.

الجزء ده **OOP كامل**، أهم ملف في التلاتة.

أي حاجة مكتوب جنبها `[Added by agent (not in my original files)]` معناها مكنتش عندك واتضافت عشان الانترفيو الحقيقي بيحتاجها.

---

# Section 1: The 4 Pillars

---

# السؤال الأول

> **What are the 4 pillars of OOP? Explain each with Java.**

إجابة الانترفيو:

> **Encapsulation (hide state + expose behavior), Inheritance (reuse via IS-A), Polymorphism (same interface different behavior), Abstraction (what not how).**

## 1- Encapsulation (مش getters و setters وخلاص)

الغلط:

```java
class Account {
    double balance; // مفتوح، أي حد يعمل balance = -100
}
```

الصح:

```java
class Account {
    private double balance;
    void withdraw(double amt) {
        if(amt <= 0 || amt > balance) throw new IllegalArgumentException("invalid");
        balance -= amt;
    }
}
```

- **What:** تجميع state + behavior + التحكم في الوصول.
- **Why:** validation و invariant (الرصيد مينفعش سالب).
- **When NOT:** DTO/Record للـ transfer مش محتاج logic، هنا getters/setters مقبولة.
==🟢Control Object access==

## 2- Inheritance (IS-A)

```java
class Animal { void eat() {} }
class Dog extends Animal { void bark() {} }
```

`Dog IS-A Animal`.

## 3- Polymorphism

```java
Animal a = new Dog();
a.sound(); // Dog barks
a = new Cat();
a.sound(); // Cat meows
```

نفس الـ `sound()` سلوك مختلف حسب الـ Object.

## 4- Abstraction

```java
interface Pay { void pay(double amt); } // WHAT
class CashPay implements Pay { public void pay(double a) { /* HOW */ } }
```

الـ Controller يعتمد على `Pay` مش على `CashPay` (هنرجع لها في DIP).

**Follow-up: Compile-time vs Runtime polymorphism؟**

- **Compile-time (static):** overloading، الـ compiler يختار بالـ signature، أسرع، مش محتاج وراثة.
- **Runtime (dynamic):** overriding، الـ JVM تختار حسب الـ object الحقيقي (late binding)، محتاج وراثة.

```java
calc.add(1,2); // static
Animal a = new Dog(); a.sound(); // dynamic
```

**What the interviewer is really testing:** هل فاهم الـ pillars كـ problem-solving مش تعريفات محفوظة.

---

# Section 2: Abstract Class vs Interface

---


```
Abstract class : only can inherit one class, can be private ,can have private method , can have fields and private fields ,can have constructor ,==🔴cant be final due to it is created to be extended== 
```
# السؤال الأول

> **When to use abstract class vs interface? And what about `default`/`static`/`private` methods?**

إجابة الانترفيو:

> **Abstract = IS-A with shared state and partial implementation (single inheritance). Interface = CAN-DO contract (multiple), with `default` for evolution, `static` for utilities, `private` for reuse inside interface (Java 9+).**

```java
abstract class Animal {
    String name; // state مشتركة
    Animal(String n) { this.name = n; } // ctor (بيتنادى من الـ child)
    abstract void sound(); // WHAT
    void sleep() { System.out.println("sleeping"); } // HOW مشترك
}
```

```java
interface Flyable {
    void fly(); // public abstract ضمنياً
    default void land() { System.out.println("landing"); } // تتورث وتتoverride
    static void info() { System.out.println("flyers"); } // Flyable.info() فقط
    private void check() { System.out.println("check"); } // [Added by agent] Java 9+، reuse جوه الـ interface
}
```

|Abstract|Interface|
|---|---|
|extends واحد|implements متعدد|
|فيه state + ctor|مفيش state (constants بس)|
|IS-A (Dog is Animal)|CAN-DO (Bird can Fly)|
|لما فيه base مشترك|لما contract متعدد|

**Spring context:** الـ Service layer دايماً `interface CourseService` + `class CourseServiceImpl` عشان DIP + proxy + mock في التست.

**Follow-up:**

> **Can abstract class have constructor? Can you do `new Animal()`? Can class be abstract + final?**

- آه فيه ctor، بيتنادى أول حاجة لما تعمل `new Dog()` عبر `super(name)` عشان يهيئ الجزء بتاع الأب.
- لا مينفعش `new Animal()` ولا `new Flyable()`، لكن ينفع `Animal a = new Dog()` و anonymous `new Flyable(){...}`.
- لا مينفعش abstract + final مع بعض (abstract عايز يتورث، final يمنع الوراثة، contradiction).

**What the interviewer is really testing:** بيفرق بين reuse و contract ولا بيختار بالحظ.

---

# Section 3: Overloading vs Overriding (بالقواعد والـ Edge Cases)

---

# السؤال الأول

> **What are the rules for return types, access modifiers, and exceptions in overriding? And overloading vs overriding?**

إجابة الانترفيو:

> **Overloading = same name different params in same class (compile-time). Overriding = same signature in child (runtime) with covariant return, wider-or-same access, same-or-narrower checked exceptions.**

## Overloading

```java
add(int,int)
add(double,double)
add(String,String)
```

الـ compiler يختار بالـ signature.

## Overriding rules

### 1- Return: نفس النوع أو subtype (covariant) [Added by agent - covariant detail]

```java
class P { Object get() { ... } }
class C extends P { @Override String get() { ... } } // ✅ String IS-A Object
```

### 2- Access: نفس أو أوسع فقط

```text
private < default < protected < public
```

```java
class P { protected void f() {} }
class C extends P { public void f() {} } // ✅ أوسع
class C2 extends P { private void f() {} } // ❌ أضيق
```

### 3- Exceptions: نفس أو أضيق (checked)، والـ unchecked براحتك

```java
class P { void f() throws IOException {} }
class C extends P { void f() throws FileNotFoundException {} } // ✅ أضيق
class C2 extends P { void f() throws Exception {} } // ❌ أوسع
```

لو الـ parent مش بيرمي checked، الـ child مينفعش يرمي checked جديدة (لكن unchecked مسموح).

لو الـ parent ctor بيرمي checked، الـ child ctor لازم `throws` نفسها أو ي handleها (سؤال Q19 في بنك الأسئلة).

**Follow-up: `test(int)` vs `test(Integer)`؟**

Exact match يكسب:

```java
test(10); // test(int) مش boxing
```

**What the interviewer is really testing:** حفظت الجدول ولا جربت الـ compile errors بنفسك.

---


# Section 4: `this`, `super`, Constructors, Chaining, Order

---

# السؤال الأول

> **Explain `super` uses, `this()` vs `super()`, and initialization order?**

## استخدامات `super`

```java
class Person {
    String name;
    Person(String n) { this.name = n; }
    void hi() { System.out.println("hi " + name); }
}
class Student extends Person {
    int grade;
    Student(String n, int g) {
        super(n); // 1- ctor الأب (لازم أول سطر)
        this.grade = g;
    }
    void hi() {
        super.hi(); // 2- method الأب + إضافة
        System.out.println("grade " + grade);
    }
    void print() {
        System.out.println(super.name); // 3- field الأب لو مخفي
    }
}

```

- `super(...)` لازم أول statement (الأب يتهيأ قبل الابن).
- لو مكتبتهاش والـ parent عنده no-arg، الـ compiler يحط `super()` ضمنياً.
- لو الـ parent معندوش no-arg؟ لازم تنادي `super("Ali")` صريح وإلا compile error.

## ليه مينفعش `this()` و `super()` مع بعض؟

الاتنين لازم أول سطر، مستحيل يجتمعوا.

```java
Student() {
    // this("Ali", 90); // lec chain داخلي
    // super("Ali"); // ❌ مينفعش الاتنين
}
```

الـ chaining الصح:

```java
Student() { this("Ali", 90); }
Student(String n, int g) { super(n); this.grade = g; }
```

## ترتيب التهيئة الكامل [Added by agent - consolidated order]

```text
Parent static -> Child static
 -> Parent instance blocks -> Parent ctor
 -> Child instance blocks -> Child ctor
```

**What the interviewer is really testing:** فاهم ليه `super()` أول سطر (الأب قبل الابن) ولا حافظ القاعدة.

---

# Section 5: Composition vs Inheritance + IS-A vs HAS-A + Aggregation

---

# السؤال الأول

> **Composition vs Aggregation vs Inheritance? When to use?**

إجابة الانترفيو:

> **Inheritance = IS-A reuse (tight). Composition = strong HAS-A (child inside, dies with parent). Aggregation = weak HAS-A (child outside/injected, lives independently). Prefer composition/DI over inheritance.**

```java
// Inheritance (IS-A):
class Dog extends Animal { }

// Composition (Strong HAS-A, child جوه):
class Car {
    private final Engine e = new Engine(); // بتموت مع العربية
}

// Aggregation (Weak HAS-A, child من بره):
class Dept {
    private final List<Teacher> ts; // بتتحقن، عايشة لو القسم اتقفل
    Dept(List<Teacher> ts) { this.ts = ts; }
}
```

إزاي تميز من الكود؟

```
new جوه الكلاس = Composition
param / field من بره = Aggregation
extends = Inheritance
```

ليه DI هو Aggregation؟

لأن الـ dependency جاية من بره (container) مش مخلوقة جوه، فـ loose coupling + testable (`EnrollmentServiceImpl` بتاخد Repo حقن مش `new`).

- **When Inheritance:** true IS-A + Liskov سليم (Square مش دايماً Rectangle، هنرجع لها).
- **When NOT:** لمجرد reuse الكود (استخدم composition)، ولما الـ hierarchy هتتعقد (favor composition).

**What the interviewer is really testing:** أشهر سؤال تصميم، بيكشف هتعمل hierarchies عميقة تخنق المشروع ولا لا.



```java
interface MessageService{
 void service();
}

class EmailService implements  MessageService
{
void service()
{
System.out.println("Email");
}
}

class SmsService implements  MessageService
{
void service()
{
System.out.println("Sms");
}
}

class Notification 
{
private final MessageService messageservice;
Notification(MessageService messageservice)
{
  this.messageservice = messageservice;
}
}
```

---

# Section 6: Modifiers + `static` + `final` + Blocks

- **Access:** `private` (الكلاس) < `default` (الـ package) < `protected` (+ الـ children) < `public` (الكل).
- **`static`:** للكلاس مش الـ object (count/utils)، مينفعش `this` جواه.
- **`final`:** var = ثابت، method = مينفعش override (لكن تتورث وتتنادى عادي)، class = مينفعش extends (زي String).
- **Instance initializer `{...}`:** بيتنفذ قبل الـ ctor مع كل `new` (للكود المشترك بين ctors).

```java
final class Config { static final int MAX = 100; }
```

`static` + `final` مع بعض = constant حقيقي.

---

# Section 7: Object Methods: equals/hashCode/toString/clone

---

# السؤال الأول

> **Explain `equals()`, `hashCode()`, `toString()`, `clone()` contracts?**

## equals + hashCode (اتشرح في Part1 بالتفصيل)

القاعدة: equal -> نفس hash، مش العكس.

## `toString()` [Added by agent (not in my original files)]

- **What:** تمثيل نصي للـ debugging/logging.
- **Why:** default `User@6d06d69c` ملهاش لازمة.
- **How:**

```java
@Override
public String toString() { return "User(id=" + id + ",name=" + name + ")"; }
```

في Spring Boot الـ logs والـ Actuator بيعتمدوا عليها، و Lombok `@ToString` بتعملها (احذر circular مع relations).

## `clone()` + Shallow/Deep

- `clone()` default = shallow (الـ nested مشترك).
- عايز deep؟ copy-constructor أو manual clone للـ nested.
- **Contract:** لو عملت `Cloneable` من غير `clone()` هترمي `CloneNotSupportedException`، ويفضل copy-constructor أوضح.

---

# Section 8: Immutability

---

# السؤال الأول

> **How to create an immutable class? Why? [Partially added by agent]**

```java
public final class Money {
    private final double amount;
    private final String currency;
    private final List<String> tags; // mutable! لازم defensive copy
    public Money(double a, String c, List<String> t) {
        this.amount = a;
        this.currency = c;
        this.tags = List.copyOf(t); // نسخة دفاعية
    }
    public double amount() { return amount; }
    public List<String> tags() { return tags; } // unmodifiable أصلاً
    // مفيش setters، مفيش extends (final)
}
```

القواعد:

```
final class + private final fields + ctor init + no setters + defensive copy للـ mutable
```

- **Why:** thread-safety من غير lock + آمن كـ HashMap key + String Pool نفس الفكرة.
- **When NOT:** الـ Entity بتاعة JPA لازم mutable (dirty checking + proxy)، استخدم record/DTO للـ immutable.

`record UserDto(Long id, String name)` هو immutable مختصر (shallow).

---

# Section 9: Inner Classes [Added by agent (not in my original files)]

> **What are the types of inner classes? Static vs non-static nested?**

```java
class Outer {
    static class StaticNested { } // من غير Outer instance: new Outer.StaticNested()
    class Inner { } // لازم Outer: outer.new Inner()، شايفة fields الأب
    void m() {
        class Local { } // جوه method فقط
        Runnable r = new Runnable() { // anonymous (مرة واحدة)
            public void run() { System.out.println("hi"); }
        };
    }
}
```

- **Static nested:** زي utils، مش محتاج outer.
- **Inner:** محتاج outer، شايف `Outer.this`.
- **Local/Anonymous:** للـ one-off (listeners/callbacks)، والـ lambda بديل الأنضف للـ functional.

متى تستخدم؟ Builder كـ static nested هو الأشهر.

---

# Section 10: Coupling & Cohesion [Added by agent (not in my original files)]

> **What is coupling and cohesion?**

- **Coupling:** قد إيه الكلاسات لازقة في بعض (عايزه **low**).
- **Cohesion:** قد إيه الكلاس مركز في مسئولية واحدة (عايزه **high**).

```java
// High coupling (وحش):
class OrderSvc { SqlOrderRepo r = new SqlOrderRepo(); } // لازق في SQL

// Low coupling + High cohesion (حلو):
@Service
class OrderSvc {
    private final OrderRepo repo; // interface، حقن
    void place() { /* order بس */ }
}
```

DI والـ interfaces هما أدواتك لتقليل coupling.

---

# Section 11: SOLID with Java Examples [Added by agent - full, only LSP fragment was in files]

> **Explain SOLID with Java?**

## S - Single Responsibility

```java
// غلط: Controller بي validate + يحسب ضريبة + يبعت ميل
// صح: Controller -> OrderService -> TaxCalc + MailSender (كل واحد شغلانة)
```

## O - Open/Closed

```java
interface Discount { double apply(double total); }
class VipDiscount implements Discount { ... } // جديد من غير تعديل القديم
```

## L - Liskov

```java
Animal a = new Dog(); a.sound(); // لازم الـ child يشتغل مكان الأب من غير كسر
```

الكسر المشهور: `Square extends Rectangle` مع `setWidth` بيكسر التوقع (الأشهر في الانترفيو).

## I - Interface Segregation

```java
// غلط: interface Robot { work(); eat(); }
// صح: interface Workable { work(); } + interface Eatable { eat(); }
```

متجبرش كلاس ينفذ methods مش محتاجها.

## D - Dependency Inversion

```java
// غلط: Controller -> CourseServiceImpl (concrete)
// صح: Controller -> CourseService (interface) <- Impl (الحقن interface)
```

ده اللي ملفاتك سمته "Interface = What, Impl = How".

---

# Section 12: Design Patterns (OOP-related)

---

# السؤال الأول

> **Explain Singleton, Factory, Builder, Strategy with Java/Spring?**

## Singleton (عندك + هنا الخلاصة)

```java
class Single {
    private static final Single I = new Single();
    private Single() {} // privacy تمنع new + inheritance
    public static Single get() { return I; }
}
```

في Spring الـ default scope singleton (one bean per context)، لكن ده container-singleton مش GoF.

## Factory

```java
Notification byChannel(String ch) {
    return switch(ch) { case "sms" -> new Sms(); default -> new Email(); };
}
```

بتخبي `new` عن الـ client (Spring `FactoryBean` نفس الفكرة).

## Builder (الأشهر مع Lombok)

```java
User u = User.builder().name("Ali").age(20).build();
```

لما الـ ctor يبقى فيه 5+ params، الـ Builder أنضف من telescoping ctors.

## Strategy

```java
interface PayStrat { void pay(double a); }
class Cash implements PayStrat { ... }
class Card implements PayStrat { ... }
// الـ Context ياخد الاستراتيجية حقن ويبدل runtime
```

في Spring: كل strategy `@Component` + `Map<String,PayStrat>` حقن وتختار بالـ key.

**What the interviewer is really testing:** استخدمت pattern لحل مشكلة حقيقية ولا بتسمع أسماء.

---

# Section 13: Tricky & Edge Cases (اللي بيحبهم الـ Interviewer)

---

# السؤال الأول

> **Diamond problem? Covariant returns? Can you override private/static/final?**

## 1- Diamond

```text
    A (print)
   / \
  B   C (الاتنين override print)
   \ /
    D (مين يورث؟)
```

Classes: ممنوع multiple inheritance عشان الغموض ده + state/ctor conflicts.

Interfaces: مسموح، لكن لو الاتنين default متشابه لازم تحسم:

```java
class D implements B,C {
    public void print() { B.super.print(); }
}
```

## 2- Covariant return

مسموح (String مكان Object) لأن IS-A محفوظ.

## 3- Can you override...؟

|السؤال|الإجابة|
|---|---|
|private method|لا (مش متشافة أصلاً، دي method جديدة مش override)|
|static method|لا (hiding بس، Reference يحدد)|
|final method|تتورث وتتنادى آه، override لأ|
|final class|extends لأ (String)|
|constructor|لا (لا يتورث أصلاً، لكن overloading آه)|
|`this()` + `super()` مع بعض|لا (الاتنين أول سطر)|

## 4- Variable/Method hiding خلاصة

```java
Bike b = new Honda3();
b.speedLimit // 90 (Reference)
b.drive() // Honda (Object)
```

## 5- ` super` خارج method/ctor؟

مينفعش (`8.1-super...md`)، فقط: أول سطر ctor + parent method + parent field.

---

# الإجابة الذهبية (OOP - 30 ثانية)

> **Encapsulation hides state behind behavior, inheritance reuses via IS-A, polymorphism varies behavior by actual object with overloading static and overriding dynamic under covariant and widening rules, abstraction exposes what via abstract and interfaces with default for evolution. Favor composition and DI for low coupling and high cohesion, code to interfaces per SOLID, use immutable value objects and records for safety, and remember hiding follows reference while overriding follows object with diamonds resolved explicitly.**

خلصت المحاكاة كاملة. راجع ملخص ليلة الانترفيو في `Interview_Simulation.md`.
