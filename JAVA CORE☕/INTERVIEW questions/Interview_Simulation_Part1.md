تمام، دلوقتي إحنا في **Java Backend Interview**، وأنا الـ Interviewer.

الجزء ده **Part1: Java Core**.

هسألك زي ما بتتسأل فعلاً، وتحت كل سؤال هتلاقي الإجابة المثالية + follow-up + هو كان بيختبر إيه.

---

# Section 1: Java Basics

---

# السؤال الأول

> **What happens when we run a Java program? From `.java` to execution?**

إجابة الانترفيو:

> **Source `.java` is compiled by `javac` into bytecode `.class`, then the JVM loads it via ClassLoader, verifies it, and executes it via Interpreter + JIT compiler, with objects on Heap and calls on Stack, until GC cleans up.**

يعني ببساطة...

```
Hello.java
  ↓ javac
Hello.class (bytecode)
  ↓ java Hello
ClassLoader -> Verifier -> Execution
```

خطوة خطوة:

```text
.java
 ↓
javac
 ↓
.class (bytecode, مش machine code)
 ↓
ClassLoader (يحمل الكلاس)
 ↓
Verifier (يتأكد مفيش كود خبيث)
 ↓
static main يدخل Stack
 ↓
Objects تتخلق في Heap
 ↓
Interpreter يشغل + JIT يحول السخن لـ native
 ↓
GC ينضف
```

تخيلها زي مطعم:

```
المنيو (.java)
 ↓
الشيف يجهز الطبق (.class)
 ↓
الجرسون يتأكد الطبق سليم (Verifier)
 ↓
الزبون ياكل (Execution)
```

**Follow-up:**

> **What is the file extension of compiled bytecode?**

```text
.class
```

> **Is `javac` the JVM?**

لا.

`javac` هو الـ compiler اللي بيحول source لـ bytecode.

الـ JVM هي اللي بتشغل الـ bytecode.

**What the interviewer is really testing:** هل فاهم إن Java مش compiled مباشرة للـ machine زي C++، وإن الـ portability جاية من الـ bytecode + JVM.

---

# السؤال التاني

> **Why is the `main` method `static`? And can you overload it?**

إجابة الانترفيو:

> **`main` is static because the JVM needs to call it without creating an object first. And yes, you can overload `main`, but the JVM only calls `public static void main(String[] args)`.**

يعني...

الـ JVM بتبدأ من غير أي Object.

لو `main` مش static، كانت هتحتاج:

```
object عشان تنادي main
 ↓
بس عشان تعمل object محتاج constructor
 ↓
طب مين ينادي الـ constructor؟
```

Chicken-egg problem.

عشان كده:

```java
Main.main(args); // من غير new
```

جوه `main` تقدر تعمل object عادي:

```java
public static void main(String[] args) {
    new Main().print();
}
```

## طيب Overload؟

```java
public static void main(String[] args) {
    System.out.println("JVM calls this");
}

public static void main(int x) {
    System.out.println("Manual only: " + x);
}
```

لو شلت الـ signature الأصلية:

```text
Main method not found
```

**Follow-up:**

> **Why `public`? Why `void`? Why `String[] args`?**

- `public` عشان JVM من package تاني تقدر تناديه.
- `void` عشان مش بيرجع حاجة للـ OS (الـ exit code عبر `System.exit`).
- `String[] args` عشان command-line arguments.

**What the interviewer is really testing:** فاهم static lifecycle ولا حافظ الـ signature خلاص.

---

# السؤال التالت

> **Is Java pass-by-value or pass-by-reference?**

ده من أشهر أسئلة Java على الإطلاق، والإجابة الغلط بتطيرك.

إجابة الانترفيو:

> **Java is always pass-by-value. For primitives the value itself is copied, for objects a copy of the reference is passed.**

يعني...

## حالة Primitive

```java
int x = 10;
go(x);
System.out.println(x); // 10
```

```
x = 10
 ↓ copy
param = 10 (غير فيه براحتك، الأصل ثابت)
```

## حالة Object

```java
Person p = new Person("Ali"); // 0x100
rename(p);
System.out.println(p.name); // Ahmed (اتغير!)
```

ليه؟

```text
p ----> 0x100 ----> Ali
              |
copy reference
              |
param ---> 0x100 ----> Ali (نفس الـ Object!)
```

فلما تعمل:

```java
param.name = "Ahmed"; // بيغير نفس الـ Object
```

لكن:

```java
param = new Person("Omar"); // بيغير الـ copy بس
System.out.println(p.name); // لسه Ahmed
```

الـ Spring context:

```java
@Transactional
public void update(User u) {
    u.setName("Ali"); // هيتعملها dirty checking لأن نفس الـ reference
    u = new User(); // دي ملهاش لازمة، الـ caller مش هيشوفها
}
```

**Follow-up:**

> **If I reassign inside method, does caller see it?**

لا.

Mutation للـ fields باين، reassignment مش باين.

**What the interviewer is really testing:** هل هتقع في فخ "objects are pass-by-reference" ولا فاهم الـ reference-copy.

---

# السؤال الرابع

> **Is Java completely object-oriented?**

إجابة الانترفيو:

> **No, because of the 8 primitives (`int`, `boolean`, etc.) which are not objects and live on Stack for speed and memory efficiency. Wrappers bridge the gap.**

```java
int x = 10; // Stack، مفيش methods
Integer y = 10; // Object، فيه methods
```

|Primitive|Wrapper|
|---|---|
|int|Integer|
|boolean|Boolean|
|double|Double|

الـ autoboxing بيحول تلقائي:

```java
list.add(10); // -> Integer.valueOf(10)
int z = list.get(0); // -> intValue()
```

ليه Java عملت كده؟

- **What:** primitives أسرع وأخف.
- **Why:** مليون `int` كـ Objects هياكلوا Heap و GC على الفاضي.
- **When NOT:** لما تحتاج Collections/Generics/null لازم Wrapper.

**What the interviewer is really testing:** فاهم trade-off الأداء ولا شايفها نقص في اللغة.

---

# السؤال الخامس

> **What is a method signature? Does return type count?**

إجابة الانترفيو:

> **Signature = method name + number, types, and order of parameters. Return type and modifiers are NOT part of it.**

```java
int sum(int a, int b) // signature: sum(int,int)
```

وده غلط compile error:

```java
int sum(int a, int b) { ... }
double sum(int a, int b) { ... } // نفس الـ signature!
```

الـ compiler بيستخدمها عشان يختار overload.

**What the interviewer is really testing:** مقدمة لسؤال overloading with null.

---

# ملاحظة سريعة (❌ مش أسئلة حقيقية)

## `for` vs `while`؟

الاتنين نفس الـ bytecode، الفرق style بس:

- `for` لما عارف العدد.
- `while` لما مستني condition.

مفيش interviewer محترم بيسأله لوحده، اتذكر هنا عشان كان في ملفاتك (`59-Difference...md`) ومش مستاهل مذاكرة عميقة.

## `break` vs `continue` vs `return`؟

```java
for(int i=1;i<=5;i++) {
    if(i==3) break; // يخرج من الـ loop: 1,2
    // if(i==3) continue; // يتخطى 3: 1,2,4,5
    // if(i==3) return; // يخرج من الـ method كلها
}
```

سؤال Junior فقط (`94-...md`)، اعرفه في سطر وعدي.

break => يخرج من اللوب 
return => تخرج من الميثود

---

# Section 2: Strings (أهم Section)

---

# السؤال الأول

> **Can you explain the difference between String, StringBuilder, and StringBuffer?**

الإجابة المثالية:

> **All three represent character sequences. String is immutable, StringBuilder is mutable and not thread-safe (fastest), StringBuffer is mutable and thread-safe via synchronized (slower).**

## أولاً: String Immutable

```java
String s = "Hello";
s.concat(" World");
System.out.println(s); // Hello
```

ليه؟

`concat()` عملت Object جديد، مغيرتش القديم.

```java
s = s.concat(" World"); // دلوقتي s بيشاور على الجديد
System.out.println(s); // Hello World
```

الذاكرة:

```text
Heap / String Pool

+---------+
| Hello   |  <- s كان هنا
+---------+

+------------------+
| Hello World      |  <- s بقى هنا
+------------------+
```

## لماذا String Immutable؟ (سؤال ذهبي)

## 1- Security

```java
String username = "admin";
```

لو Mutable أي method تقدر تغيرها، خطر.

## 2- String Pool

```java
String a = "Java";
String b = "Java";
```

الاتنين نفس الـ Object.

```text
a ─────┐
       ▼
     "Java"
       ▲
b ─────┘
```

لو Mutable وتغير `a`، يبقى `b` اتغير كارثة.

## 3- Thread Safety

Immutable مش محتاج synchronization.

## 4- HashCode Caching

`HashMap` و `HashSet` بيعتمدوا على hash ثابت.

لو String بتتغير الـ Map كله يبوظ.

## StringBuilder Mutable

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
System.out.println(sb); // Hello World (نفس الـ Object)
```

ليه أسرع؟

مفيش:

```text
New Object + Allocation + GC
```

كل مرة.

أهم Methods:

```java
append()
insert()
delete()
replace()
reverse()
capacity()
```

Capacity:

```java
new StringBuilder() // 16
new StringBuilder("Hello") // 16 + 5 = 21
// لما تتملي: newCapacity = oldCapacity * 2 + 2
```

## StringBuffer

نفس Builder بس:
String is imuatable unlike StringBuilder and StringBuffer ,and stringBuilder is the fastest


```java
public synchronized StringBuffer append(...)
```

Thread واحد يدخل، الباقي يستنى Lock.

عشان كده أبطأ.

|Feature|String|StringBuilder|StringBuffer|
|---|---|---|---|
|Mutable|❌|✅|✅|
|Thread Safe|✅ بالـ Immutability|❌|✅|
|Performance|بطيء مع التعديل|الأسرع|أبطأ من Builder|
|يستخدم مع|نص ثابت|Single Thread|Multi Thread|

**Follow-up:**

> **Which one should you prefer?**

> **Prefer StringBuilder unless you specifically need thread safety.**

في المشاريع الحديثة StringBuffer نادر، بنستخدم Concurrent structures أو local Builder.

**What the interviewer is really testing:** فاهم immutability و trade-off السرعة والأمان ولا حافظ جدول.

---

# السؤال التاني

> **Why is the String class declared `final`? Is it the same as immutability?**

إجابة الانترفيو:

> **`final` prevents inheritance, which protects immutability. If String were extendable, a subclass could break Security, Pool sharing, and hashCode caching.**

لأ، مش نفس الحاجة:

- `final class` = مينفعش تورث.
- `immutable` = المحتوى مبيتغيرش.

الـ `final` بتحمي الـ `immutable`:

```java
// تخيل لو مسموح:
class EvilString extends String {
    // يكسر الـ Pool ويغير السلوك
}
```

الأسباب الأربعة نفس اللي فوق (Security/Pool/Thread-safety/hashCode).

**What the interviewer is really testing:** بيفرق بين `final` كـ mechanism و immutability كـ guarantee.

---

# السؤال التالت

> **What happens when you call `new String("hello")` vs `"hello"` — how many objects?**

إجابة الانترفيو:

> **Literal `"hello"` reuses the Pool (0 new if exists, else 1). `new String("hello")` creates Pool literal + Heap object (1 or 2 new).**
> string literal reuses the pool ,

```java
String a = "Java";
String b = "Java";
System.out.println(a == b); // true (نفس الـ Pool Object)
```

```java
String c = new String("Java");
System.out.println(a == c); // false (Heap vs Pool)
System.out.println(a.equals(c)); // true (المحتوى واحد)
```

`==` بتقارن References.

`equals()` في String بتقارن المحتوى.

لو:

```java
String x = new String("hello");
```

والـ `"hello"` مش موجودة قبل كده = **2 Objects** (واحد Pool + واحد Heap).

لو موجودة = **1 Object** (الـ Heap بس).

**Follow-up:**

> **Can StringBuilder be converted to String? Does it modify the String?**

```java
String s = sb.toString(); // String جديدة مستقلة
```

لا، هو عنده `char[]` داخلي قابل للتعديل، و `toString()` بتعمل نسخة immutable مستقلة.

**What the interviewer is really testing:** فاهم الـ Pool ولا فاكر كل `new` زي بعضه.

---

# السؤال الرابع

> **What is the difference between `==` and `equals()` with Integer cache?**

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true (Cache -128..127)
```

```java
Integer c = 128;
Integer d = 128;
System.out.println(c == d); // false (Objects مختلفة)
System.out.println(c.equals(d)); // true
```

القاعدة: استخدم `equals()` للقيم، `==` للـ references فقط.

**What the interviewer is really testing:** وقعت في الـ Cache trap قبل كده ولا لأ.

---

# Section 3: static Family + Hiding (Traps)

---

# السؤال الأول

> **What are static variables and static methods?**

إجابة الانترفيو:

> **Static members belong to the class, not objects. One copy shared across all instances, accessible without `new`, but static methods can only touch static data.**
> static methods can only touch static data.

```java
class Student {
    static int count = 0;
    Student() { count++; }
}
```

```java
Student s1 = new Student(); // count=1
Student s2 = new Student(); // count=2 (نفس الـ copy)
```

```java
Math.sqrt(16); // من غير new Math()
```

الغلط المشهور:

```java
static void print() {
    System.out.println(this.name); // ❌ مفيش this في static
}
```

ليه مينفعش `static` جوه method؟

```java
void foo() {
    static int x = 10; // ❌ Compile error
}
```

لأن الـ local عايش على Stack per-call، والـ static عايش في Method Area طول البرنامج، مينفعش يتحجز جوه call مؤقت.

**Spring context:** الـ `static` عكس الـ Bean (Bean = instance مدار من Spring، static = class-level خارج الـ container، صعب تعملها mock في التست).

**What the interviewer is really testing:** فاهم memory الفرق بين class-level و object-level.

static methods can only touch static data , and static data can only be in static methods 

---

# السؤال التاني

> **Explain execution order: static block vs instance block vs constructor? When does static block run?**

```java
class Demo {
    static { System.out.println("static"); }
    { System.out.println("instance"); }
    Demo() { System.out.println("ctor"); }
}
```

```java
new Demo();
new Demo();
```

الناتج:

```text
static (مرة واحدة)
instance
ctor
instance
ctor
```

القاعدة:

```text
Static (مرة عند class load)
 ↓
Instance (مع كل new)
 ↓
Constructor (مع كل new)
```

ومع الوراثة:

```text
Parent static -> Child static -> Parent instance/ctor -> Child instance/ctor
```

إمتى الـ static block بيتنفذ؟

مش مع بداية البرنامج، لكن مع أول استخدام للكلاس:

- `new`
- static method/var access
- `Class.forName()`

لو الكلاس متلمسش خالص، مش هيتنفذ.

**What the interviewer is really testing:** حافظ الترتيب ولا فاهم الـ class loading.



```java 
class Sara 
{
static {System.out.println("Static block");}
{System.out.println("Instance block");}
Sara(){System.out.println("Ctor block");}
}

public static void main(String[] args)
{
System.out.println("Main block");
Sara s = new Sara();
}
```

---

# السؤال التالت

> **What happens if a static method and instance method have the same name in the same class? Can static be overridden?**

إجابة الانترفيو:

> **If signatures are identical it is a compile error because `static` is not part of the signature. If params differ it is overloading and allowed. Static methods are hidden, not overridden.**

```java
class A {
    static void print() {}
    void print() {} // ❌ Compile error
}
```

لكن:

```java
class A {
    static void print() {}
    void print(String s) {} // ✅ Overloading
}
```

## Method hiding

```java
class Animal { static void sound() { System.out.println("Animal"); } }
class Dog extends Animal { static void sound() { System.out.println("Dog"); } }
```

```java
Animal a = new Dog();
a.sound(); // Animal (الـ Reference هو اللي يحدد، compile-time)
```

قارن مع overriding:

```java
class Animal { void walk() { System.out.println("Animal"); } }
class Dog extends Animal { void walk() { System.out.println("Dog"); } }
```

```java
Animal a = new Dog();
a.walk(); // Dog (الـ Object هو اللي يحدد، runtime)
```

القاعدة الذهبية:

```text
variables + static methods -> Reference Type (compile-time)
instance methods -> Actual Object (runtime)
```

لو حطيت `@Override` على static هيعمل error، وده إثبات إنه hiding مش overriding.

نادي صح كده:

```java
Animal.sound();
Dog.sound();
```

**Follow-up:**

> **What about `print(null)` with `print(Object)` vs `print(String)`?**

```java
void print(Object o) { ... }
void print(String s) { ... }
print(null); // String (الأكثر تخصصاً، String IS-A Object)
```

لكن:

```java
void print(String s) { ... }
void print(Integer i) { ... }
print(null); // ❌ Ambiguous compile error
```

**What the interviewer is really testing:** أشهر trap في Java، بيفرق اللي جرب بإيده.

---

# السؤال الرابع

> **Explain variable hiding with `Bike`/`Honda3`?**

```java
class Bike { int speedLimit = 90; }
class Honda3 extends Bike { int speedLimit = 150; }
```

```java
Bike obj = new Honda3();
System.out.println(obj.speedLimit); // 90 (!!)
```

ليه؟

الـ Object فيه الاتنين (90 و 150)، بس الـ variables بتتربط بالـ Reference.

لو method كانت هتطبع 150 (runtime).

```text
variables = Reference decides
methods = Object decides
```

لو الأسماء مختلفة (`speedLimit` vs `speedLimits`) ده مش hiding أصلاً، دي variable جديدة، والـ `Bike` ref مش شايفها خالص (compile error).

**What the interviewer is really testing:** نفس فكرة hiding، بس على fields.

---

# Section 4: Collections + HashMap + Wrappers

---

# السؤال الأول

> **What is HashMap? How does `put()`/`get()` work internally?**

إجابة الانترفيو:

> **HashMap is a Key-Value structure with O(1) average put/get/remove. It hashes the key to find a bucket, then uses `equals()` inside the bucket. Collisions use linked-list, converted to Red-Black Tree in Java 8.**

تخيل مكتبة فيها مليون كتاب، بدل ما تمشي على الرفوف، بتدخل ID يقولك Shelf 15 مباشرة.

جوه الـ HashMap فيه Array، كل خانة اسمها Bucket:

```text
0
1
2 -> Ahmed->30
3
4
5 -> Ali->20
6
7
```

`put("Ali",20)`:

```
1- hashCode("Ali")
 ↓
2- bucket = hash % length (فعلياً bitwise أسرع)
 ↓
3- فاضي؟ حط. مش فاضي؟ شوف Key موجود؟ آه Update، لأ حطه آخر الـ Chain
```

`get("Ali")` نفس الفكرة: hash -> bucket -> دور بـ `equals()` -> رجع Value.

## Collision

اتنين Keys نزلوا نفس الـ Bucket.

Java 7: LinkedList (`O(n)`).

Java 8: لو زاد عن 8 في Bucket مناسبة يتحول Red-Black Tree (`O(log n)`).

## Null؟

```java
map.put(null, "Ali"); // مسموح، Null Key واحد بس
map.put(1, null); // مسموح، Null values براحتك
```

ليه Key واحد؟

لأن `null` ملهاش `hashCode()`، فبتتخزن في Bucket مخصص.

## Capacity و Load Factor

```
Initial = 16
Load Factor = 0.75
16 * 0.75 = 12 -> أول ما توصل 12 يحصل Resize لـ 32 + Rehashing
```

ليه Resize؟

عشان يقلل الـ Collision.

## Thread-safe؟

لا.

> **HashMap is not thread-safe. Use ConcurrentHashMap.**

|HashMap|Hashtable|
|---|---|
|Not thread-safe|Thread-safe|
|أسرع|أبطأ|
|يسمح null|لا يسمح|

## ليه الـ Key لازم Immutable؟

لو استخدمت `User` mutable وغيرت `name` بعد ما دخل Map، الـ `hashCode` هيتغير ومش هتعرف توصله تاني.

عشان كده `String` أفضل Key.

**Follow-up:**

> **Difference between HashMap, LinkedHashMap, TreeMap?**

- `HashMap`: الأسرع O(1)، غير مرتب.
- `LinkedHashMap`: بيحافظ على ترتيب الإدخال.
- `TreeMap`: مرتب حسب Key، O(log n).

> **When to use HashMap in real project?**

```java
Map<Long, User> users = new HashMap<>(); // ID -> User
Map<String, Integer> count = new HashMap<>(); // Frequency count
count.put(w, count.getOrDefault(w, 0) + 1);
```

**What the interviewer is really testing:** ده السؤال اللي بيحدد مستواك، لو جاوبت الـ internals صح بتعدي 90% من أسئلة Collections.

---

# السؤال التاني

> **What is the difference between `equals()` and `hashCode()`? Why override both?**

إجابة الانترفيو:

> **`equals()` checks logical equality by content, `hashCode()` returns an int used to find the bucket in hash collections. Contract: if equal then same hash. Collections call hash first, then equals only on collision.**

الـ `Object` الأصلي:

```java
public boolean equals(Object obj) {
    return this == obj; // Reference comparison
}
```

عشان كده:

```java
Employee e1 = new Employee(1,"Ali");
Employee e2 = new Employee(1,"Ali");
System.out.println(e1.equals(e2)); // false (من غير Override)
```

بعد Override:

```java
@Override
public boolean equals(Object obj) {
    if(this == obj) return true;
    if(obj == null) return false;
    if(getClass() != obj.getClass()) return false;
    Employee other = (Employee) obj;
    return id == other.id && Objects.equals(name, other.name);
}
```

بقت `true`.

## ليه الاتنين مع بعض؟

```java
HashSet<Employee> set = new HashSet<>();
set.add(new Employee(1));
set.add(new Employee(1));
System.out.println(set.size()); // من غير hashCode = 2 (غلط)، مع hashCode = 1 (صح)
```

لأن الـ Set بيعمل `hashCode()` الأول عشان يلاقي الـ Bucket، لو الـ hash مختلف مش بينادي `equals()` أصلاً.

القاعدة الذهبية:

> **لو `a.equals(b)==true` يبقى لازم `a.hashCode()==b.hashCode()`. العكس مش شرط (Collision عادي).**

```java
@Override
public int hashCode() {
    return Objects.hash(id, name);
}
```

**What the interviewer is really testing:** هل فاهم الـ flow (hash -> bucket -> equals) ولا حافظ القاعدة.

---

# السؤال التالت

> **What is autoboxing? What are the risks?**

إجابة الانترفيو:

> **Autoboxing is automatic conversion between primitives and wrappers (`int` <-> `Integer`) via `valueOf()`/`intValue()`. Risks: NPE on null unboxing, performance in loops, and `==` cache trap.**

```java
Integer x = 10; // -> Integer.valueOf(10)
int y = x; // -> x.intValue()
```

## 1- NPE

```java
Integer z = null;
int w = z; // NullPointerException!
```

## 2- Performance

```java
Integer sum = 0;
for(int i=0;i<1000000;i++) sum += i; // boxing/unboxing مليون مرة + Objects
```

استخدم primitive في الـ loops.

## 3- `==` trap

```java
Integer a = 127, b = 127; // true (Cache)
Integer c = 128, d = 128; // false (استخدم equals)
```

## 4- Overload resolution

```java
test(int) vs test(Integer)
test(10); // int (exact match يكسب، مش boxing)
```

القاعدة: Exact > Widening > Boxing > Varargs.

**Spring context:** الـ `@RequestParam int id` لو الـ param ناقص هيضرب، لكن `Integer id` يقبل null وتقدر ت validate.

**What the interviewer is really testing:** استخدمت Wrappers في Collections و API ولا لسه بتتلخبط في NPE.

---

# السؤال الرابع (سريع)

> **Array vs List? When to use?**

- `Array`: ثابت الحجم، primitives، O(1) مباشر.
- `List` (ArrayList): ديناميك، Wrappers، API غني، resize تلقائي amortized O(1).

95% من شغل Spring بـ List.

```java
User[] arr = new User[10]; // ثابت
List<User> list = new ArrayList<>(); // بيكبر
```

**What the interviewer is really testing:** Junior check سريع.

---

# Section 5: Exceptions

---

# السؤال الأول

> **What is the difference between Checked and Unchecked Exceptions?**

إجابة الانترفيو:

> **Checked are verified by compiler and must be handled (`IOException`), for recoverable external failures. Unchecked are `RuntimeException`s that compile fine but fail at runtime and usually indicate bugs.**

الـ Hierarchy اللي لازم تحفظها:

```text
                Throwable
               /         \
          Error       Exception
                          |
              -----------------------
              |                     |
      RuntimeException         Checked Exceptions
              |
      NullPointerException
      ArithmeticException
      ...
```

## Checked

```java
FileReader r = new FileReader("data.txt"); // ❌ Unhandled exception
```

لازم:

```java
try { ... } catch(IOException e) { ... }
// أو
throws IOException
```

أمثلة: `IOException`, `SQLException`, `FileNotFoundException`.

## Unchecked

```java
String s = null;
s.length(); // compile عادي، Run: NullPointerException
```

أمثلة: `NPE`, `ArithmeticException`, `IndexOutOfBounds`.

|Checked|Unchecked|
|---|---|
|Compiler يجبرك|Compiler ساكت|
|Recoverable (ملف ناقص)|Bug (null)|
|try/catch أو throws|يفضل تصلح الكود|

**Spring context:** الـ Spring بيحب `RuntimeException` + `@ControllerAdvice` عشان متلوثش الـ Service signatures بـ `throws` في كل حتة.

**Follow-up:**

> **Difference between `final`, `finally`, `finalize`?**

- `final`: keyword لمنع التغيير (var/method/class).
- `finally`: block بيتنفذ دايماً للـ cleanup.
- `finalize()`: deprecated، كانت بتتنادى قبل GC، دلوقتي استخدم try-with-resources.

**What the interviewer is really testing:** فاهم فلسفة Java في الـ recoverable vs programming errors.

---

# السؤال التاني

> **What happens if an exception is thrown inside a `finally` block? What if try/catch/finally all throw?**

إجابة الانترفيو:

> **The `finally` exception wins and hides the original. Never throw from `finally`; use try-with-resources which preserves via `getSuppressed()`.**

```java
try { throw new RuntimeException("TRY"); }
catch(Exception e) { throw new RuntimeException("CATCH"); }
finally { throw new RuntimeException("FINALLY"); }
// اللي هيتشاف: FINALLY فقط!
```

القاعدة:

> **Finally can override.**

الصح:

```java
try(FileReader r = new FileReader("a.txt")) {
    // use
} // close() لو ضربت هتتسجل suppressed مش هتضيع الأصلية
```

**What the interviewer is really testing:** Senior trap، هل عارف إن الـ original exception بتضيع.

---

# Section 6: Memory / JVM + References + Copy

---

# السؤال الأول

> **Explain Stack vs Heap? Where do objects and references live?**

إجابة الانترفيو:

> **Stack is per-thread, stores frames and locals, auto-freed, fast (`StackOverflowError`). Heap is shared, stores objects via `new`, managed by GC (`OutOfMemoryError`).**

|Stack|Heap|
|---|---|
|per-thread|مشترك|
|frames + local vars|Objects|
|سريع + تلقائي|GC|
|StackOverflow (recursion)|OutOfMemory (leak)|

```java
void foo() {
    int x = 10; // Stack
    Dog d = new Dog(); // d (reference) في Stack، الـ Object في Heap
}
```

```text
Stack          Heap
d ──────>   [Dog Object]
x=10
```

**Follow-up: Reference vs Object؟**

```java
Dog d = new Dog(); // reference ≠ object
Dog c = d; // نسخ الـ reference مش الـ Object (alias)
c = null; // الـ reference بقى null، الـ Object لسه عايش لو حد تاني مشاور عليه
```

الـ Object يبقى eligible للـ GC فقط لما **صفر** references تشاور عليه، مش لما تعمل null مباشرة.

```java
Book b = new Book(); // O1
Book c = new Book(); // O2
Book d = c; // d -> O2
c = b; // c -> O1 (O2 لسه عايش عبر d)
```

**What the interviewer is really testing:** أساس فهم `new` و alias و GC.

---

# السؤال التاني

> **What is shallow copy vs deep copy? How to achieve each?**

```java
class Person { Address address; }
```

Shallow: بتنسخ الـ Person بس، والـ `address` لسه مشترك:

```text
p1.address ──┐
             ▼
         [Address]
             ▲
p2.address ──┘
p1.address == p2.address // true
```

Deep: بتنسخ كل حاجة:

```text
p1.address -> [Address1]
p2.address -> [Address2] (نسخة مستقلة)
```

إزاي؟

- Shallow: `clone()` default.
- Deep: manual copy-constructor (المفضل) أو `clone()` مع نسخ الـ nested.

```java
Person(Person other) {
    this.address = new Address(other.address); // deep
}
```

خلي بالك `String` immutable فمشاركتها أمان، المشكلة في الـ mutable nested.

**What the interviewer is really testing:** فاهم الـ aliasing خطر إمتى.

---

# Section 7: Java 8+

---

# السؤال الأول

> **What changed in Java 8 interfaces? `default` vs `static`?**

إجابة الانترفيو:

> **Before 8 only abstract + constants. `default` allows backward-compatible evolution (inherited, overridable), `static` belongs to the interface itself (not inherited/overridden).**

المشكلة قبل 8:

لو عندك `List` وعايز تضيف `sort()`، كل الـ implementations هتتكسر.

الحل:

```java
interface Vehicle {
    default void horn() { System.out.println("beep"); } // تتورث وتتoverride
    static void info() { System.out.println("vehicles"); } // Vehicle.info() فقط
}
```

- `default`: polymorphism عادي.
- `static`: `Interface.method()`، مفيش inheritance.

**Follow-up: Diamond مع default؟**

لو `class D implements B,C` والاتنين عندهم نفس الـ default، لازم تعمل override صريح وتحسم:

```java
B.super.print();
```

**What the interviewer is really testing:** فاهم ليه Java ضافت default أصلاً.

---

# السؤال التاني

> **What is a method reference? What are its 4 types?**

إجابة الانترفيو:

> **Shorthand `::` for a lambda that just calls one method, only for functional interfaces.**

```java
list.forEach(x -> System.out.println(x));
list.forEach(System.out::println); // نفس الحاجة أقصر
```

الأربعة:

```java
Integer::parseInt // 1- static method
printer::print // 2- instance of particular object
String::compareToIgnoreCase // 3- instance of arbitrary type
User::new // 4- constructor
```

شرط: مفيش logic زيادة، مجرد pass-through.

**What the interviewer is really testing:** بتكتب Streams نضيفة ولا lambdas مطولة على الفاضي.

---

# الإجابة الذهبية في الانترفيو (Part1 - 30 ثانية)

> **Java compiles to bytecode and runs on JVM with Stack for calls and Heap for objects. Strings are immutable with Pool for Security and performance, use StringBuilder for mutations. HashMap hashes keys to buckets O(1) with Tree fallback in Java 8, guarded by equals/hashCode contract. Checked exceptions are compiler-enforced for recoverable failures, finally exceptions override originals. Static belongs to class with hiding at compile-time, instance methods override at runtime. Java 8 defaults evolve interfaces without breaking, and method references clean up Streams.**

خلصت Part1، خش على `Interview_Simulation_Part2.md`.
