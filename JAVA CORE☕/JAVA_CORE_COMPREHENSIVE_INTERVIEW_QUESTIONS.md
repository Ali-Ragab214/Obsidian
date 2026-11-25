# ☕ JAVA CORE — Comprehensive Interview Questions

> **من الـ Fundamentals للـ Advanced — أسئلة مترابطة بمستوى Java Backend Interview**

> **الفكرة من الملف ده مش إنه "بنك أسئلة" محفوظة.**
> كل سؤال هنا بيجمع أكتر من مفهوم من ملفات الـ Core، وبيختبرك في **ليه** القرار ده اتاخد، و**إزاي** بيتنفذ جوه الـ JVM، و**إيه** اللي يحصل لو غيّرنا حاجة، و**إيه** الفرق بينه وبين البديل.
> الأسئلة مرتبة من الأساسيات للـ Advanced، وكل سؤال مبني على اللي قبله، فلو قرأتهم بالترتيب هتطلع بصورة متصلة مش شوية تعريفات متفرقة.

*ملاحظة: ملفات Spring/JPA/DB الموجودة في الفولدر مش ضمن نطاق الأسئلة دي؛ اتستخدمت بس للمقارنة لما يكون فيه مفهوم Core متداخل (زي Record vs JPA Entity).*

---

## 📌 إزاي تستخدم الملف ده

| الاستخدام | الطريقة |
|---|---|
| **مراجعة سريعة** | اقرأ الـ **Interview Answer** والـ **Common Interview Trap** بس |
| **تحضير عميق** | اقرأ السؤال كامل + اكتب الـ Practical Example من غير ما تبص |
| **Mock Interview** | حدّد 5 أسئلة عشوائية، جاوب بصوت عالي في 2–3 دقايق لكل سؤال |
| **مراجعة قبل الإنترفيو** | الـ **Interview Preparation Checklist** في الآخر |
| **لفهم الـ WHY** | ركّز على **Deep Dive** و**Follow-up**، لأنهم بيبنوا التفكير مش الحفظ |

### 🎯 مستوى الأسئلة
- **Intermediate → Advanced Java Backend**
- الأسئلة بتفترض إنك عارف الـ Syntax، وبتختبر **التصميم، الـ Runtime Behavior، والـ Trade-offs**.
- كل سؤال فيه **Follow-ups** واقعية بتتسأل فعلاً في إنترفيوهات Backend.
- الأمثلة مكتوبة بأسلوب **Java 21**، مع توضيح الفروق المهمة (Java 8 / 11 / 17) لما تكون مؤثرة.

### 🗺️ طريقة قراءة الإجابات
- **Interview Answer** = الإجابة اللي تقولها في الإنترفيو (مختصرة، منظمة، فيها keywords).
- **Deep Dive** = اللي بيحصل تحت الغطاء (JVM / Language Spec / Data Structure).
- **Practical Example** = كود واقعي، وغالبًا فيه سيناريو Backend.
- **Common Interview Trap** = الغلطة اللي بتوقع 90% من المرشحين.

---

## 📚 Table of Contents

| Part | الموضوع | الأسئلة |
|---|---|---|
| **1** | OOP & Building Blocks | Q1 – Q4 |
| **2** | Object Identity, Immutability & Data Classes | Q5 – Q9 |
| **3** | Composition, DI & Design Principles | Q10 – Q11 |
| **4** | Strings & Wrapper Classes | Q12 – Q13 |
| **5** | Generics & Type Erasure | Q14 |
| **6** | Collections & HashMap Internals | Q15 – Q18 |
| **7** | Exceptions & Resource Management | Q19 – Q20 |
| **8** | JVM Memory, GC, Class Loading & JIT | Q21 – Q23 |
| **9** | Java 8+ : Lambdas, Streams & Optional | Q24 – Q26 |
| **10** | Concurrency Essentials | Q27 – Q28 |
| **11** | Serialization & Production Concerns | Q29 |
| **12** | Modern Java: Sealed Types & Pattern Matching | Q30 |
| **13** | Rapid-Fire Traps & Final Checklist | Q31 + Checklist |

---

## 🧱 Part 1 — OOP & Building Blocks

### Q1. إزاي تشرح الـ Four Pillars of OOP مش كتعريفات محفوظة، لكن كقرارات تصميم بتأثر على كود Backend حقيقي؟

**🎯 Related Concepts:**
Encapsulation · Inheritance · Polymorphism (Compile-time vs Runtime) · Abstraction · IS-A vs HAS-A · Liskov Substitution Principle · Coupling & Cohesion

**🗣️ Interview Answer:**

> OOP is a design paradigm, not just a set of keywords. Its four pillars answer four different design questions:

protect data from direct access
>
> - **Encapsulation**: how do I protect an object's invariants and hide its internal representation?
> - **Abstraction**: what contract do I expose, and what implementation details do I hide?
> - **Inheritance**: is there a true **IS-A** relationship that justifies coupling my class to a parent?
> - **Polymorphism**: can I treat different implementations through one type and let the runtime choose the behavior?
>
> In a backend service, this shows up as: entities that validate their own state (encapsulation), service layers that depend on interfaces (abstraction + polymorphism), and inheritance used only when the subtype is genuinely substitutable for the parent (LSP). Most of the time, **composition is preferred over inheritance** because it reduces coupling.

**🔍 Deep Dive:**

| Pillar            | السؤال اللي بيجاوب عليه                           | Backend Example                                         | الغلطة الشائعة                              |
| ----------------- | ------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------- |
| **Encapsulation** | مين يقدر يغيّر الحالة؟ وإزاي أحمي الـ invariants؟ | `BankAccount.withdraw()` بترفض السحب لو الرصيد مش كفاية | اعتبارها مجرد `private` + getters/setters   |
| **Abstraction**   | إيه اللي أعرّضه وإيه اللي أخفيه؟                  | `PaymentGateway` interface بدون تفاصيل Stripe/PayPal    | الخلط بينها وبين الـ Encapsulation          |
| **Inheritance**   | هل العلاقة فعلًا IS-A؟                            | `SavingsAccount extends Account`                        | استخدامها لمجرد إعادة استخدام كود           |
| **Polymorphism**  | مين يختار التنفيذ: الـ Compiler ولا الـ Runtime؟  | `gateway.pay()` تتنفذ حسب الـ object الفعلي             | اعتبار الـ Overloading runtime polymorphism |

**1️⃣ Encapsulation = حماية الـ Invariant، مش Getters & Setters
** 
==🔴protect data from direct access==

الـ Encapsulation الحقيقية معناها إن الكلاس **يمنع أي حالة غير صالحة**. لو عندك:

```java
public class BankAccount {
    private long balance; // بالـ cents

    public void withdraw(long amount) {
        if (amount <= 0) throw new IllegalArgumentException("amount must be positive");
        if (amount > balance) throw new IllegalStateException("insufficient funds");
        balance -= amount;
    }

    public long balance() { return balance; }
}
```

مفيش `setBalance()`، لأن السماح بتغيير الرصيد مباشرة معناه إن الـ invariant (`balance >= 0`) بقى مسؤولية كل caller، وده مش encapsulation.

**2️⃣ Abstraction = تفصل الـ "What" عن الـ "How"
**
==🔴focus on what not how== 

```java
public interface PaymentGateway {
    PaymentResult charge(Money amount, PaymentMethod method);
}
```

الـ `OrderService` بيتعامل مع `PaymentGateway`، ومش عارف ولا مهتم إذا كان Stripe ولا PayPal. ده بيخلّي الـ high-level policy مستقل عن الـ low-level detail (وده أساس الـ Dependency Inversion اللي جاي في Q11).

**3️⃣ Inheritance = علاقة IS-A حقيقية**

الوراثة مش أداة "أخد كود جاهز". لما تعمل `class Admin extends User`، أنت بتقول إن أي `Admin` **لازم** يشتغل في كل مكان متوقع فيه `User` (LSP). لو `Admin` بقت بترفض عمليات أساسية في `User`، يبقى العلاقة غلط.

**4️⃣ Polymorphism = نفس الرسالة، تنفيذ مختلف**

- **Compile-time (Static)**: Overloading — الـ compiler بيختار الـ method من الـ reference type والـ arguments.
  ال compile-time poly اسرع لان الاختيار بين ال methods بيتم اثناء ال compilation ومش هنستني لل runtime 
- **Runtime (Dynamic)**: Overriding — الـ JVM بيختار التنفيذ من الـ **actual object type** عن طريق الـ virtual method dispatch. 
 **Runtime (Dynamic)** is more flexible 

```java
PaymentGateway gateway = new StripeGateway();
gateway.charge(...); // Stripe implementation at runtime
```

**LSP — الـ Pillar الخفي:**

> أي `Subtype` لازم يقدر يحل محل الـ `Supertype` من غير ما يكسر توقعات الـ caller.

مثال كلاسيكي للانتهاك:

```java
class Rectangle {
    protected int w, h;
    public void setW(int w) { this.w = w; }
    public void setH(int h) { this.h = h; }
    public int area() { return w * h; }
}

class Square extends Rectangle {
    @Override public void setW(int w) { this.w = w; this.h = w; }
    @Override public void setH(int h) { this.w = h; this.h = h; }
}
```

`Square` مش substitutable لـ `Rectangle`، لأن كود بيتعامل مع `Rectangle` بيفترض إن تغيير العرض مش بيغيّر الطول. الحل: متعملش `Square extends Rectangle`، وخلّي الاتنين implements `Shape`.

**Coupling & Cohesion:**

- **High cohesion**: الكلاس بيعمل حاجة واحدة مترابطة.
- **Low coupling**: أقل اعتماد ممكن على تفاصيل الآخرين.
- الـ Inheritance بيزوّد الـ coupling بشدة (fragile base class)، والـ Composition بيقلله.

**💻 Practical Example:**

```java
import java.util.List;

// Abstraction: العقد
interface DiscountPolicy {
    Money discountFor(Order order);
}

// Encapsulation: الحالة محمية والقواعد جوه
final class Order {
    private final List<OrderLine> lines;
    private final Customer customer;

    Order(List<OrderLine> lines, Customer customer) {
        if (lines == null || lines.isEmpty())
            throw new IllegalArgumentException("order must have lines");
        this.lines = List.copyOf(lines);
        this.customer = customer;
    }

    Money subtotal() {
        return lines.stream()
                    .map(OrderLine::total)
                    .reduce(Money.ZERO, Money::plus);
    }

    Customer customer() { return customer; }
    List<OrderLine> lines() { return lines; } // immutable copy anyway
}

// Polymorphism: نفس الاستدعاء، سياسات مختلفة
final class VipDiscount implements DiscountPolicy {
    public Money discountFor(Order order) {
        return order.subtotal().multiply(0.15);
    }
}

final class NoDiscount implements DiscountPolicy {
    public Money discountFor(Order order) { return Money.ZERO; }
}

// High-level policy مش عارف تفاصيل الـ discount
final class CheckoutService {
    private final DiscountPolicy discountPolicy;

    CheckoutService(DiscountPolicy discountPolicy) { // DI (Q10/Q11)
        this.discountPolicy = discountPolicy;
    }

    Money total(Order order) {
        return order.subtotal().minus(discountPolicy.discountFor(order));
    }
}
```

لاحظ: `CheckoutService` مغلق قدام التعديل ومفتوح قدام الإضافة — ضفنا `RamadanDiscount` من غير ما نلمس `CheckoutService` (Open/Closed).

**🔁 Follow-up Interview Questions:**

1. إيه الفرق الدقيق بين **Abstraction** و**Encapsulation**؟ (الاتنين بيخفوا حاجة، بس واحدة بتخفي **التعقيد** والتانية بتحمي **الحالة**)
2. هل ينفع يكون عندك Polymorphism من غير Inheritance؟ (أيوه: Interfaces، وFunctional Interfaces، وDuck typing مش موجود في Java لكن السلوك متاح عبر contracts)
3. هل Java لغة OOP خالصة؟ (لأ: فيها primitives و`static` members، و`main` static)
4. ادّيني مثال LSP violation من كود حقيقي، وإزاي تصلحه.
5. إمتى تختار Inheritance بدل Composition؟ (لما يكون فيه IS-A حقيقي + سلوك مشترك ثابت + مش متوقع تغييرات في الـ base)
6. إيه الفرق بين **Compile-time** و**Runtime** polymorphism من ناحية الـ performance والـ flexibility؟

**⚠️ Common Interview Trap:**

- ❌ **"Encapsulation = private fields + getters/setters"** — ده مجرد **data hiding**، والـ encapsulation الحقيقية بتحمي الـ invariants وتمنع الحالة غير الصالحة.
- ❌ **"Inheritance is for code reuse"** — إعادة الاستخدام سبب ثانوي؛ السبب الأساسي هو **substitutability**. لو عايز reuse بس، استخدم Composition.
- ❌ **"Overloading is runtime polymorphism"** — الـ Overloading بيتحدد وقت الـ compile من الـ reference type، وممكن يحصل ambiguity.
- ❌ **"Java is purely OOP"** — لأ، فيها primitives وstatic، و`String` مش primitive لكنها special.

---

### Q2. إمتى تختار Abstract Class وإمتى Interface؟ وإزاي تتعامل مع Default Methods والـ Diamond Problem؟

**🎯 Related Concepts:**
Abstract Class · Interface · Default Methods (Java 8) · Static & Private Interface Methods (Java 9) · Multiple Inheritance of Type vs State · Diamond Problem · Template Method Pattern · Binary Compatibility

**🗣️ Interview Answer:**

> The choice is about **what you're modeling**, not about which one "has more features":
>
> - Use an **interface** when you're defining a **capability/contract** that unrelated classes can implement, and when you need multiple inheritance of type. An interface has no instance state and no constructor.
> - Use an **abstract class** when subclasses share **state, invariants, or a partial implementation** (template method), and there's a genuine IS-A relationship.
>
> Since Java 8, interfaces can have `default` and `static` methods, and since Java 9 `private` methods, so interfaces can carry behavior — but they still cannot hold instance state. The diamond problem is solved by explicit rules: a superclass method wins over an interface default; if two interfaces conflict, the class **must override** and can call `InterfaceName.super.method()`.



**🔍 Deep Dive:**

| Feature | Interface | Abstract Class |
|---|---|---|
| **Multiple inheritance** | ✅ Yes (type only) | ❌ No (single class) |
| **Instance fields** | ❌ No (only `public static final` constants) | ✅ Yes |
| **Constructor** | ❌ No | ✅ Yes (called by subclass) |
| **Method types** | abstract, default, static, private (9+) | abstract, concrete, static, final |
| **Access modifiers** | methods implicitly `public` (private allowed 9+) | any |
| **When to use** | Capability/contract, decoupling, multiple types | Shared state + partial implementation, template method |
| **Evolution** | Adding abstract method breaks implementors (default method avoids it) | Adding concrete method is safe |

**Default Method Rules (الـ Diamond):**

1. **Class wins**: لو الـ superclass فيه method بنفس التوقيع، هي اللي تتاخد، والـ interface default بتتجاهل.
2. **Most specific interface wins**: لو `interface B extends A`، وكلاهما فيه default، الـ `B` هي الأقرب.
3. **Conflict → compile error**: لو الاتنين مش مرتبطين ببعض، لازم الكلاس يعمل override.
4. **Explicit call**: `A.super.method()` للاختيار اليدوي.

```java
interface A {
    default String hello() { return "A"; }
}

interface B {
    default String hello() { return "B"; }
}

class C implements A, B {
    @Override
    public String hello() {
        return A.super.hello() + " + " + B.super.hello(); // "A + B"
    }
}
```

**Interface Evolution — ليه الـ Default Methods اتعملت؟**

قبل Java 8، لو عايز تضيف method لـ `Collection`، كل الـ implementors في الدنيا هتتكسر. الحل: `default` method بتوفر **binary compatibility** — الكلاسات القديمة تفضل شغالة، والجديد يقدر يعمل override.

لكن الـ default methods جابت مشكلة جديدة: لو كلاس implements واجهتين فيهم نفس الـ default signature، لازم تحل التعارض بنفسك.

**Static & Private Interface Methods:**

```java
interface Validator {
    boolean isValid(String value);

    static Validator notEmpty() {
        return v -> v != null && !v.isBlank();
    }

    private static void log(String msg) { /* helper */ }
}
```

- `static` methods في interface **مش بتتورث**، وبتتنادى `Validator.notEmpty()`.
- `private` methods (Java 9+) بتستخدم كـ helper جوه الـ default methods، ومش جزء من الـ contract.

**الـ Interface Fields:**

```java
interface Limits {
    int MAX_RETRIES = 3; // public static final implicitly
}
```

أي field في interface هو `public static final` تلقائيًا — يعني **constant**، ومش حالة per-object. لو محتاج حالة، يبقى abstract class أو composition.

**Template Method — الاستخدام الذكي للـ Abstract Class:**

```java
abstract class ReportExporter {
    // Template: الترتيب ثابت
    public final byte[] export(Report report) {
        validate(report);
        String content = render(report);   // hook
        byte[] bytes = compress(content);  // hook
        return sign(bytes);
    }

    protected abstract String render(Report report);
    protected abstract byte[] compress(String content);

    private void validate(Report report) {
        if (report == null) throw new IllegalArgumentException();
    }

    private byte[] sign(byte[] bytes) { /* common */ return bytes; }
}
```

- الـ `final` بيمنع الكلاس الفرعي إنه يغيّر الترتيب.
- الـ Subclass بيعمل override للـ hooks بس.
- ده مثال حقيقي لـ **"inheritance for behavior, not for reuse"**.

**💻 Practical Example:**

```java
interface Cache<K, V> {
    V get(K key);
    void put(K key, V value);
    void evict(K key);

    default V getOrCompute(K key, java.util.function.Function<K, V> loader) {
        V value = get(key);
        if (value == null) {
            value = loader.apply(key);
            put(key, value);
        }
        return value;
    }
}

abstract class AbstractCache<K, V> implements Cache<K, V> {
    protected final int maxSize;

    protected AbstractCache(int maxSize) { this.maxSize = maxSize; }

    protected void ensureCapacity() {
        if (size() >= maxSize) evictOne();
    }

    protected abstract int size();
    protected abstract void evictOne();
}

final class InMemoryCache<K, V> extends AbstractCache<K, V> {
    private final java.util.Map<K, V> store = new java.util.LinkedHashMap<>();

    InMemoryCache(int maxSize) { super(maxSize); }

    public V get(K key) { return store.get(key); }

    public void put(K key, V value) {
        ensureCapacity();
        store.put(key, value);
    }

    public void evict(K key) { store.remove(key); }

    protected int size() { return store.size(); }
    protected void evictOne() { store.keySet().stream().findFirst().ifPresent(store::remove); }
}
```

هنا الـ interface بيحدد الـ contract + default behavior، والـ abstract class بتحمل الحالة المشتركة (`maxSize`) والـ template، والـ concrete class بتنفذ التخزين.

**🔁 Follow-up Interview Questions:**

1. هل الـ Interface ممكن تحتوي على `private` methods؟ ومن أنهي إصدار؟ (أيوه، Java 9)
2. إيه اللي يحصل لو كلاس implements واجهتين فيهم نفس الـ default method؟ (compile error إلا لو override)
3. لو `abstract class` implements interface، هل لازم تنفذ كل الـ methods؟ (لأ، ممكن تسيبها abstract)
4. إزاي تضيف method لواجهة مستخدمة في production من غير ما تكسر الـ implementors؟ (default method)
5. هل الـ `default` method بتتورث في الـ subinterface؟ (أيوه، ولو حصل تعارض الـ most specific بيفوز)
6. إيه الفرق بين `abstract class` و`interface` من ناحية الـ state والـ constructor؟
7. هل ينفع تعمل `new` لـ interface؟ (لأ، إلا anonymous class — Q9)
8. إمتى تفضّل `sealed interface` على `interface` عادي؟ (Q7/Q9)

**⚠️ Common Interview Trap:**

- ❌ **"Interface can't have method bodies"** — من Java 8 فيه default/static، ومن 9 private.
- ❌ **"Interface can have instance fields"** — لأ، أي field `public static final`، يعني constant.
- ❌ **"Abstract class is always faster/better"** — مفيش "أفضل" مطلق؛ القرار حسب الحاجة للـ state والـ multiple types.
- ❌ **"Default methods solve multiple inheritance"** — هم بيحلوا **multiple inheritance of type/behavior**، لكن مش **state**، وبيعملوا تعارضات محتاجة حل يدوي.
- ❌ **"Abstract class can't have a constructor"** — لأ، وعنده constructor بيتنادى من الـ subclass، بس مش بتنشئ object لوحده.

---

### Q3. إيه الفرق بين Overloading وOverriding وHiding؟ وإزاي الـ JVM بيختار الـ Method؟

**🎯 Related Concepts:**
Static vs Dynamic Binding · Method Signature · Covariant Return Type · Field Hiding · Static Method Hiding · `@Override` · Overload Resolution Phases · Varargs

**🗣️ Interview Answer:**

> - **Overloading** is compile-time polymorphism: same method name, different parameter list, resolved by the compiler based on the **reference type** and the **static types of arguments**.
> - **Overriding** is runtime polymorphism: same signature in a subclass, resolved by the JVM based on the **actual object type**. The return type can be covariant, visibility cannot be reduced, and checked exceptions cannot be broadened.
> - **Hiding** happens with `static` methods and fields: they are not overridden, and the **reference type** decides which one is used.
>
> Constructors, `private` methods, `final` methods, and `static` methods cannot be overridden. Fields are never polymorphic.

**🔍 Deep Dive:**

| | Overloading | Overriding | Hiding |
|---|---|---|---|
| **Binding** | Compile-time | Runtime | Compile-time |
| **Applies to** | Methods | Instance methods | Static methods & fields |
| **Decided by** | Reference type + argument types | Actual object type | Reference type |
| **Signature** | Must differ in parameters | Must be identical | Same signature (static) |
| **Return type** | Can differ (with different params) | Covariant allowed | — |
| **Access** | Any | Cannot reduce | — |
| **`static`** | Allowed | ❌ Not allowed | ✅ Hiding |
| **`private`/`final`** | Allowed | ❌ Not allowed | — |

**1️⃣ Overload Resolution — الترتيب اللي الـ Compiler بيمشي عليه:**

الـ Compiler بيحاول في 3 مراحل بالترتيب:

1. **Strict invocation** (بدون boxing/varargs): exact match أو widening primitive/reference.
2. **Boxing/unboxing** (مع widening لو لازم).
3. **Varargs**.

```java
static void f(long x)     { System.out.println("long"); }
static void f(Integer x)  { System.out.println("Integer"); }

f(1); // long  ← widening beats boxing
```

```java
static void f(Object o)  { System.out.println("Object"); }
static void f(String s)  { System.out.println("String"); }

f(null); // String ← most specific reference type
```

لو مفيش "most specific" (مثلًا `f(String)` و`f(Integer)` مع `null`)، يبقى **ambiguous → compile error**.

**2️⃣ Overriding Rules — القواعد اللي بتتسأل في الإنترفيو:**

```java
class Parent {
    protected Number value() throws IOException { return 1; }
}

class Child extends Parent {
    @Override
    public Integer value() { return 2; } // ✅ covariant return, narrower access, no checked exception
}
```

- ✅ Covariant return: `Integer` subtype of `Number`.
- ✅ Visibility can be wider (`protected` → `public`)، مش أضيق.
- ✅ Checked exceptions can be narrower or removed، مش أوسع.
- ❌ `final` method → cannot override.
- ❌ `static` method → hidden, not overridden.
- ❌ `private` method → not inherited; نفس التوقيع في الابن = method جديدة.
- ❌ Constructors → not inherited/overridden.

**3️⃣ Static Hiding & Field Hiding:**

```java
class Parent {
    String name = "parent";
    static String type() { return "Parent"; }
    String instance() { return "Parent"; }
}

class Child extends Parent {
    String name = "child";
    static String type() { return "Child"; }
    @Override String instance() { return "Child"; }
}

Parent p = new Child();
System.out.println(p.name);      // parent  ← field hiding
System.out.println(p.type());    // Parent  ← static hiding
System.out.println(p.instance()); // Child   ← overriding
```

**ليه؟**
- الـ Fields والـ static methods بيتحلّوا **statistically** من الـ reference type.
- الـ Instance methods بيتحلّوا **dynamically** من الـ object type (virtual dispatch).

**4️⃣ Private Methods — مفاجأة كتير:**

```java
class Parent {
    private void secret() { System.out.println("Parent"); }
    public void call() { secret(); } // binding to Parent.secret
}

class Child extends Parent {
    public void secret() { System.out.println("Child"); } // new method, NOT override
}

new Child().call(); // Parent
```

الـ `private` method مش part of the contract، فمفيش override. نفس الاسم في الابن = method مستقلة. ولو حطيت `@Override` هتاخد compile error.

**5️⃣ `@Override` — ليه إلزامي عمليًا:**

لو غلطت في التوقيع (مثلاً `equals(MyClass o)` بدل `equals(Object o)`)، من غير `@Override` الـ compiler مش هيكشف الغلطة، وهتكون بتعمل overload مش override. `@Override` بيحوّل الغلطة لـ compile-time error.

**6️⃣ Access Modifiers — مين يشوف مين:**

| Modifier | Same Class | Same Package | Subclass (Other Package) | Everywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| **package-private** (default) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ (through inheritance) | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

- `protected` = package access + subclasses، **مش** "everywhere".
- الـ **top-level class** ينفع تكون `public` أو package-private بس.
- الـ **interface members** `public` افتراضيًا (مع `private` methods من Java 9).
- تقليل الـ visibility في الـ override **compile error** (Q3 rules).
- الـ access modifiers بتتحقق وقت الـ compile؛ الـ reflection ممكن تتجاوزها بـ `setAccessible` (مع قيود الـ modules).

**💻 Practical Example:**

```java
import java.util.*;

class Notification {
    public String channel() { return "generic"; }

    public void send(String message) {
        System.out.println("[" + channel() + "] " + message); // virtual call
    }
}

class EmailNotification extends Notification {
    @Override public String channel() { return "email"; }
}

class SmsNotification extends Notification {
    @Override public String channel() { return "sms"; }
}

public class Main {
    static void dispatch(List<Notification> notifications, String msg) {
        for (Notification n : notifications) n.send(msg); // runtime dispatch
    }

    public static void main(String[] args) {
        dispatch(List.of(new EmailNotification(), new SmsNotification()), "Order shipped");
        // [email] Order shipped
        // [sms] Order shipped
    }
}
```

**Overload + Inheritance Trap:**

```java
class Base {
    void print(Object o) { System.out.println("Base(Object)"); }
}

class Derived extends Base {
    void print(String s) { System.out.println("Derived(String)"); }
}

Base b = new Derived();
b.print("hello"); // Base(Object) ← overload chosen at compile time from Base's methods
```

الـ `Derived.print(String)` مش override للـ `Base.print(Object)`، فهو **overload** جديد، والـ compiler شايف الـ reference type `Base` فمش لاقي غير `print(Object)`.

**🔁 Follow-up Interview Questions:**

1. هل ممكن تعمل overload بفرق في return type بس؟ (لأ، الـ signature مش بيتضمن return type)
2. إيه اللي يحصل لو الـ subclass قلل الـ visibility؟ (compile error)
3. هل ممكن الـ override يرمي exception أوسع؟ (لأ للـ checked، أيوه للـ unchecked)
4. إيه الفرق بين `static` method في parent وchild؟ (hiding، والـ reference type بيحدد)
5. هل الـ Fields بتتعملها override؟ (لأ، hiding)
6. إزاي الـ compiler بيتعامل مع `f(null)` لما فيه أكتر من overload؟ (most specific، وإلا ambiguity)
7. إيه الـ bridge methods؟ وليه الـ compiler بيولّدها مع الـ Generics؟ (Q14)
8. هل `private` method ممكن تكون `final`؟ (مش لازم، هي أصلًا مش قابلة للـ override)

**⚠️ Common Interview Trap:**

- ❌ **"Static methods are overridden"** — هم **hidden**، والـ reference type بيحدد.
- ❌ **"Fields are polymorphic"** — لأ، field access بتتحل من الـ reference type.
- ❌ **"Private methods can be overridden"** — لأ، نفس التوقيع = method جديدة.
- ❌ **"Overloading is runtime"** — لأ، compile-time.
- ❌ **"Covariant return means any return type"** — لازم يكون subtype فعلي.
- ❌ **"`@Override` optional, just a warning"** — هو مش إلزامي، لكنه بيكشف أخطاء التوقيع اللي هتتحول لسلوك غلط وقت الـ runtime.

---

### Q4. إيه اللي بيحصل بالظبط وقت إنشاء Object؟ اشرح Constructors وInitialization Order و`this()`/`super()`.

**🎯 Related Concepts:**
Constructor Chaining · `this()` vs `super()` · Static vs Instance Initializer Blocks · Field Initialization · Abstract Class Constructor · Private Constructor · Final Fields · Calling Overridable Methods in Constructor

**🗣️ Interview Answer:**

> When you create an object, Java runs initialization in a strict order:
>
> 1. **Parent static** fields/blocks (once per class, at class initialization)
> 2. **Child static** fields/blocks
> 3. **Parent instance** field initializers and instance blocks, in source order
> 4. **Parent constructor** body
> 5. **Child instance** field initializers and instance blocks
> 6. **Child constructor** body
>
> `this(...)` and `super(...)` must both be the **first statement** in a constructor, so they are mutually exclusive. If you don't write either, the compiler inserts `super()` — which fails to compile if the parent has no no-arg constructor. Constructors are **not inherited** and **not overridden**, but an abstract class can have one, and a private constructor prevents subclassing.

**🔍 Deep Dive:**

**1️⃣ الـ Order الكامل:**

```java
class Parent {
    static { System.out.println("1. Parent static block"); }
    { System.out.println("3. Parent instance block"); }
    Parent() { System.out.println("4. Parent constructor"); }
}

class Child extends Parent {
    static { System.out.println("2. Child static block"); }
    { System.out.println("5. Child instance block"); }
    Child() { System.out.println("6. Child constructor"); }
}

new Child();
```

**Output:**
```
1. Parent static block
2. Child static block
3. Parent instance block
4. Parent constructor
5. Child instance block
6. Child constructor
```

**السبب:**
- الـ Static initialization بيحصل **مرة واحدة** عند أول استخدام للكلاس (active use)، والأب قبل الابن.
- كل constructor بيبدأ بـ `super()` (صريح أو ضمني)، فالـ parent instance initialization + constructor بتخلص **قبل** أي حاجة في الابن.

**2️⃣ `this()` vs `super()`:**

```java
class Employee {
    private final String name;

    Employee() {
        this("unknown"); // ✅ constructor chaining
    }

    Employee(String name) {
        this.name = name; // implicit super() هنا
    }
}
```

- `this(...)` بتنادي constructor تاني في **نفس الكلاس**.
- `super(...)` بتنادي constructor في **الأب**.
- الاتنين لازم يكونوا **أول statement**.
- مينفعش الاتنين مع بعض في نفس الـ constructor.
- لو مفيش أي واحد، الـ compiler بيحط `super()` ضمنيًا.

**3️⃣ لو الأب مفيهوش No-Arg Constructor:**

```java
class Parent {
    Parent(int x) { }
}

class Child extends Parent {
    Child() { } // ❌ compile error: no suitable constructor Parent()
    // الحل:
    // Child() { super(0); }
}
```

**4️⃣ Abstract Class Constructor:**

```java
abstract class Shape {
    protected final String name;

    protected Shape(String name) {
        this.name = name;
    }
}

class Circle extends Shape {
    Circle() { super("circle"); }
}
```

الـ abstract class **مينفعش تتInstansiate**، لكن الـ constructor بتاعها بيتنفذ كجزء من إنشاء الـ subclass. ده مكان طبيعي لتهيئة الحالة المشتركة.

**5️⃣ Private Constructor — استخدامات حقيقية:**

```java
// 1) Utility class
final class Strings {
    private Strings() { }
    static boolean isBlank(String s) { return s == null || s.isBlank(); }
}

// 2) Singleton (Q11)
final class Config {
    private static final Config INSTANCE = new Config();
    private Config() { }
    static Config getInstance() { return INSTANCE; }
}

// 3) Factory / controlled creation
final class Money {
    private Money() { }
    static Money of(long cents) { return new Money(); }
}
```

الـ private constructor **بيمنع الوراثة** عمليًا، لأن أي subclass لازم ينادي `super()` ومش قادر.

**6️⃣ استدعاء Method قابلة للـ Override جوه Constructor — فخ خطير:**

```java
class Parent {
    Parent() {
        System.out.println("value = " + value()); // dynamic dispatch!
    }
    String value() { return "parent"; }
}

class Child extends Parent {
    private String value = "child";

    @Override String value() { return value; }
}

new Child(); // value = null
```

**ليه `null`؟**
- الـ `Parent` constructor اتنفذ **قبل** تهيئة حقول `Child`.
- الـ method اللي اتنفذت هي `Child.value()` (runtime dispatch)، لكن `Child.value` لسه `null`.
- القاعدة: **متستدعيش overridable methods من constructor**. استخدم `final` methods أو methods `private`/`static` للتهيئة.

**7️⃣ Final Fields:**

```java
class User {
    private final String id;
    private final String email;

    User(String id, String email) {
        this.id = id;
        this.email = email;
    }
}
```

- كل `final` field لازم يتعين **مرة واحدة** قبل نهاية كل constructor.
- لو فيه أكتر من constructor، كل واحد لازم يعيّنها (أو يستخدم `this(...)`).
- `static final` لازم يتعين في declaration أو static block.

**💻 Practical Example:**

```java
abstract class Entity {
    protected final long id;

    protected Entity(long id) {
        if (id <= 0) throw new IllegalArgumentException("id must be positive");
        this.id = id;
    }
}

final class Customer extends Entity {
    private static int created;
    static { System.out.println("Customer class initialized"); }

    private final String email;
    { created++; } // instance block

    Customer(long id, String email) {
        super(id); // must be first
        this.email = email;
    }

    Customer(String email) {
        this(0L, email); // ❌ سيرمي IllegalArgumentException لأن id = 0
    }
}
```

لاحظ إن `super(id)` بتحصل قبل الـ instance block، والـ validation في الأب بتحمي الـ invariant حتى لو الابن حاول يتحايل.

**🔁 Follow-up Interview Questions:**

1. هل الـ Constructor بيتورث؟ (لأ)
2. هل الـ Constructor ممكن يكون `final`/`abstract`/`static`؟ (لأ)
3. إيه اللي يحصل لو حطيت `this()` و`super()` مع بعض؟ (compile error)
4. لو الكلاس فيه static وinstance blocks، إيه الترتيب؟ (static مرة واحدة، ثم instance كل object)
5. إزاي تمنع الوراثة في Java؟ (`final class` أو private constructor)
6. ليه استدعاء overridable method من constructor خطر؟ (الحقول لسه default)
7. هل الـ `final` field ممكن يتعين في instance block؟ (أيوه، طالما مرة واحدة)
8. إيه الفرق بين `super()` و`this()` من ناحية الـ binding؟

**⚠️ Common Interview Trap:**

- ❌ **"Constructors are inherited"** — لأ، ومينفعش تعمل override.
- ❌ **"Abstract class can't have a constructor"** — لأ، وعندها constructor بيتنفذ.
- ❌ **"`super()` and `this()` can be used together"** — مستحيل، كلاهما لازم يكون أول statement.
- ❌ **"Child fields are initialized before parent constructor"** — العكس هو الصحيح.
- ❌ **"Calling an overridden method in constructor is safe"** — لأ، ممكن تشتغل على حقول لسه `null`/`0`.

---

## 🧬 Part 2 — Object Identity, Immutability & Data Classes

### Q5. اشرح `==` و`equals()` و`hashCode()` كمنظومة واحدة. إيه اللي بيحصل جوه `HashMap`/`HashSet` لما تعمل override لواحد بس؟

**🎯 Related Concepts:**
Reference Equality · Logical Equality · `equals()` Contract · `hashCode()` Contract · Buckets & Collisions · Mutable Keys · `Objects.equals/hash` · `getClass()` vs `instanceof` · Records Equality

**🗣️ Interview Answer:**

> `==` compares **references** (or primitive values), while `equals()` is meant for **logical equality** and `hashCode()` must be consistent with it.
>
> The contract is:
> - `equals` is reflexive, symmetric, transitive, consistent, and returns `false` for `null`.
> - If `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` **must** hold.
> - The reverse is not required: equal hash codes do **not** imply equal objects (collisions are allowed).
>
> Hash-based collections first use `hashCode()` to find the bucket, then `equals()` to find the exact entry. If you override `equals()` without `hashCode()`, two logically equal objects can land in different buckets, so `HashSet` will keep duplicates and `HashMap.get()` can return `null`. If you override `hashCode()` without `equals()`, collisions increase and equality still fails.

**🔍 Deep Dive:**

**1️⃣ Default Behavior:**

```java
Object.equals()   → reference equality (this == obj)
Object.hashCode() → identity-based hash (often derived from memory address, but not guaranteed)
```

يعني من غير override:

```java
var a = new User(1, "Ali");
var b = new User(1, "Ali");
a.equals(b); // false
a == b;      // false
```

**2️⃣ الـ Contract بالتفصيل:**

| Rule | Meaning |
|---|---|
| **Reflexive** | `x.equals(x)` must be `true` |
| **Symmetric** | `x.equals(y) == y.equals(x)` |
| **Transitive** | `x.equals(y) && y.equals(z)` → `x.equals(z)` |
| **Consistent** | Repeated calls return the same result if state didn't change |
| **Null-safe** | `x.equals(null)` must be `false`, not throw |
| **Hash consistency** | `x.equals(y)` → `x.hashCode() == y.hashCode()` |

**3️⃣ التنفيذ الصحيح:**

```java
import java.util.Objects;

final class UserId {
    private final long value;

    UserId(long value) { this.value = value; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                  // fast path
        if (o == null || getClass() != o.getClass()) return false; // exact type
        UserId other = (UserId) o;
        return value == other.value;
    }

    @Override
    public int hashCode() {
        return Long.hashCode(value);
    }
}
```

**ليه `getClass()` مش `instanceof`؟**

- `instanceof` بتسمح للـ subclass إنها تكون equal، وده بيكسر **symmetry** لو الـ subclass غيّرت الـ equals.
- `getClass()` بتضمن إن الاتنين نفس النوع بالظبط، فمفيش مفاجآت.
- بعض الـ frameworks بتستخدم `instanceof` (مثلاً لو عايزة proxies تشتغل)، لكن القاعدة الافتراضية: **`getClass()`** إلا لو عندك سبب واضح.

**4️⃣ اللي بيحصل جوه `HashMap`:**

```java
map.put(key, value);
```

1. يحسب `hash = key.hashCode()` ثم يوزّعه (spread) ويحسب الـ index.
2. لو الـ bucket فاضي → يدخل مباشرة.
3. لو فيه entries → يلف على الـ bucket ويقارن:
   - `hash` متساوي؟
   - `key == entry.key` أو `key.equals(entry.key)`؟
4. لو لقى equal → يستبدل القيمة.
5. لو ملقاش → يضيف entry جديدة (أو يعمل treeify لو الـ bucket كبير — Q16).

**سيناريو الكارثة: Override `equals` بس**

```java
class BadKey {
    private final String id;
    BadKey(String id) { this.id = id; }

    @Override public boolean equals(Object o) {
        return o instanceof BadKey k && id.equals(k.id);
    }
    // ❌ No hashCode override
}

var set = new HashSet<BadKey>();
set.add(new BadKey("A"));
set.add(new BadKey("A"));
System.out.println(set.size()); // 2 ❌
```

الـ `hashCode()` الافتراضي مختلف لكل object، فالاتنين راحوا buckets مختلفة، والـ `equals` ما اتناداتش أصلًا.

**سيناريو تاني: Override `hashCode` بس**

```java
class BadKey2 {
    private final String id;
    BadKey2(String id) { this.id = id; }

    @Override public int hashCode() { return id.hashCode(); }
    // ❌ No equals override
}

set.add(new BadKey2("A"));
set.add(new BadKey2("A"));
set.size(); // 2 ❌ (same bucket, but equals is identity → false)
```

**5️⃣ Mutable Keys — أخطر Bug في Production:**

```java
class MutableKey {
    String id;
    MutableKey(String id) { this.id = id; }

    @Override public boolean equals(Object o) {
        return o instanceof MutableKey k && Objects.equals(id, k.id);
    }
    @Override public int hashCode() { return Objects.hash(id); }
}

var map = new HashMap<MutableKey, String>();
var key = new MutableKey("A");
map.put(key, "value");

key.id = "B"; // ❌ hash changed after insertion

System.out.println(map.get(new MutableKey("B"))); // null
System.out.println(map.get(new MutableKey("A"))); // null
```

الـ entry موجودة في bucket القديم، لكن الـ lookup الجديد بيروح bucket جديد. **القاعدة: مفاتيح الـ HashMap لازم تكون immutable** (أو على الأقل الـ fields الداخلة في `hashCode`/`equals` متتغيّرش).

**6️⃣ `Objects` Utility:**

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Money m)) return false;
    return cents == m.cents && Objects.equals(currency, m.currency);
}

@Override
public int hashCode() {
    return Objects.hash(cents, currency); // varargs → array allocation
}
```

لاحظ: `Objects.hash(...)` بيعمل array allocation، ففي الـ hot paths ممكن تكتب الـ hash يدويًا:

```java
int result = Long.hashCode(cents);
result = 31 * result + currency.hashCode();
return result;
```

**7️⃣ Records — Equality جاهزة:**

```java
record Point(int x, int y) { }

new Point(1, 2).equals(new Point(1, 2)); // true
new Point(1, 2).hashCode() == new Point(1, 2).hashCode(); // true
```

الـ Record بيولّد `equals`/`hashCode`/`toString` من الـ components. لكن لو الـ component نفسه mutable (مثلاً `List`)، الـ equality shallow وهتتأثر بالتغييرات (Q7).

**💻 Practical Example:**

```java
import java.util.*;

final class Money {
    private final long cents;
    private final String currency;

    Money(long cents, String currency) {
        this.cents = cents;
        this.currency = Objects.requireNonNull(currency);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return cents == money.cents && currency.equals(money.currency);
    }

    @Override
    public int hashCode() {
        return 31 * Long.hashCode(cents) + currency.hashCode();
    }

    @Override
    public String toString() {
        return cents + " " + currency;
    }

    public static void main(String[] args) {
        var a = new Money(1000, "USD");
        var b = new Money(1000, "USD");

        System.out.println(a == b);        // false
        System.out.println(a.equals(b));   // true

        var set = new HashSet<Money>();
        set.add(a);
        set.add(b);
        System.out.println(set.size());    // 1

        var map = new HashMap<Money, String>();
        map.put(a, "order-1");
        System.out.println(map.get(b));    // order-1
    }
}
```

**🔁 Follow-up Interview Questions:**

1. لو عندك `equals` بس من غير `hashCode`، إيه اللي يحصل في `HashMap`؟
2. ليه `Objects.hash()` مش مثالي في الـ performance-sensitive code؟
3. إيه الفرق بين `getClass()` و`instanceof` في `equals`؟
4. هل ينفع `hashCode` يكون ثابت (return 1)؟ (أيوه، لكن الأداء هيبقى O(n))
5. إيه اللي يحصل لو غيّرت field داخل key بعد `put`؟
6. هل الـ `equals` لازم تكون symmetric مع الـ subclass؟
7. إزاي الـ Records بتولّد `equals`/`hashCode`؟
8. هل `String` و`Integer` عندهم `equals`/`hashCode` صح؟ (أيوه)

**⚠️ Common Interview Trap:**

- ❌ **"Override equals only is enough"** — لأ، لازم الاتنين مع بعض.
- ❌ **"hashCode must be unique"** — مش شرط؛ الـ collisions مسموحة، بس لازم equal objects يكون لهم نفس الـ hash.
- ❌ **"Mutable objects can be HashMap keys"** — ممكن، لكن كارثة لو الـ hash-related fields اتغيرت.
- ❌ **"`equals(null)` throws NPE"** — لازم يرجع `false` (إلا لو التنفيذ غلط).
- ❌ **"`==` compares content for objects"** — لأ، references.
- ❌ **"Records are deeply immutable"** — لأ، shallow (Q6/Q7).

---

### Q6. إيه معنى Immutability الحقيقي؟ وإزاي تعمل Defensive Copy؟ وإيه علاقة `final` وRecords بالموضوع؟

**🎯 Related Concepts:**
Immutable Objects · `final` Fields · Defensive Copy · Shallow vs Deep Copy · Unmodifiable vs Immutable Collections · Record Shallow Immutability · Thread Safety · `String` Immutability

**🗣️ Interview Answer:**

> An immutable object is one whose observable state **cannot change after construction**. That requires more than `final` fields:
> - all fields are `final`,
> - the class is `final` (or constructors are private / no accessible subclass),
> - no mutable object is exposed directly,
> - mutable inputs are **defensively copied** on the way in,
> - mutable internal state is **defensively copied** on the way out.
>
> `final` only means the **reference** can't be reassigned; it does not freeze the object it points to. `Collections.unmodifiableList()` is a **view**, not an immutable copy. Records give you shallow immutability: components are `final`, but if a component is a `List`, the list contents can still change.

**🔍 Deep Dive:**

**1️⃣ `final` ≠ Immutable:**

```java
final class Bad {
    private final List<String> items = new ArrayList<>();

    void add(String item) { items.add(item); } // ❌ mutating the object
}
```

الـ reference `items` ثابت، لكن الـ List نفسها بتتغير. فالـ object مش immutable.

**2️⃣ Immutability الكاملة:**

```java
import java.util.*;

final class Period {
    private final Date start;
    private final Date end;

    Period(Date start, Date end) {
        this.start = new Date(start.getTime()); // defensive copy in
        this.end = new Date(end.getTime());
        if (this.start.after(this.end))
            throw new IllegalArgumentException("start after end");
    }

    public Date start() {
        return new Date(start.getTime()); // defensive copy out
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```

لو رجّعت `start` مباشرة، أي caller يقدر يعمل `p.start().setTime(0)` ويكسر الـ invariant.

**3️⃣ Shallow vs Deep Copy:**

| | Shallow Copy | Deep Copy |
|---|---|---|
| **Meaning** | نسخة جديدة من الـ object، لكن الـ references الداخلية مشتركة | نسخة جديدة + نسخ كل الـ objects الداخلية |
| **Java** | `Object.clone()` الافتراضي، copy constructor سطحي | تنفيذ يدوي أو serialization/JSON |
| **Risk** | تعديل الداخلي بيأثر على الأصل | آمن لكن أغلى |

```java
class Order {
    private final List<Item> items;

    Order(List<Item> items) {
        this.items = new ArrayList<>(items); // shallow copy of the list
    }
}
```

لو `Item` نفسها mutable، لازم deep copy أو تحويلها لـ immutable/value object.

**4️⃣ Unmodifiable vs Immutable:**

```java
List<String> original = new ArrayList<>(List.of("a", "b"));
List<String> view = Collections.unmodifiableList(original);

original.add("c");
System.out.println(view); // [a, b, c] ❌ view changed
```

- `Collections.unmodifiableList` = **read-only view** فوق الـ list الأصلية. لو الأصل اتغير، الـ view بتتغير.
- `List.copyOf(original)` = **immutable copy** (snapshot) — التغييرات على الأصل مش بتأثر عليها.
- `List.of(...)` = immutable بالكامل، و**بترفض `null`**.

**5️⃣ Records — Shallow Immutability:**

```java
record Order(List<Item> items) { }

var items = new ArrayList<Item>();
items.add(new Item("book"));

var order = new Order(items);
items.add(new Item("pen"));

System.out.println(order.items().size()); // 2 ❌
order.items().add(new Item("laptop"));     // ❌ مسموح! الـ list mutable
```

الـ record بيعمل `final` للـ reference، لكن مش بيعمل defensive copy. الحل:

```java
record Order(List<Item> items) {
    Order {
        items = List.copyOf(items); // compact constructor
    }
}
```

**6️⃣ ليه Immutability مهمة في الـ Backend؟**

- **Thread safety**: الـ immutable objects بتتقري من أي thread بدون synchronization.
- **Safe keys** في `HashMap`/`HashSet` (Q5).
- **No aliasing bugs**: مفيش caller بيغيّر حالتك من تحت رجلك.
- **Easy caching**: الـ object آمن للمشاركة.
- **Value objects**: `Money`, `UserId`, `DateRange`, `Email`.

**7️⃣ Cost:**

- كل تعديل بيعمل object جديد (copy-on-write).
- Deep copies مكلفة.
- في الـ hot loops، استخدم `StringBuilder` (Q12) أو mutable builders ثم `build()` immutable.

**💻 Practical Example:**

```java
import java.util.*;

final class DateRange {
    private final java.time.LocalDate from;
    private final java.time.LocalDate to;

    DateRange(java.time.LocalDate from, java.time.LocalDate to) {
        Objects.requireNonNull(from);
        Objects.requireNonNull(to);
        if (from.isAfter(to)) throw new IllegalArgumentException("from > to");
        this.from = from; // LocalDate immutable → no copy needed
        this.to = to;
    }

    boolean contains(java.time.LocalDate date) {
        return !date.isBefore(from) && !date.isAfter(to);
    }

    DateRange overlap(DateRange other) {
        var start = from.isAfter(other.from) ? from : other.from;
        var end = to.isBefore(other.to) ? to : other.to;
        return start.isAfter(end) ? null : new DateRange(start, end);
    }
}
```

`LocalDate` immutable بالفعل، فمفيش داعي لـ defensive copy. **القاعدة: اختار types immutable أصلًا** (java.time بدل `Date`/`Calendar`).

**🔁 Follow-up Interview Questions:**

1. هل `final` field معناها immutable object؟ (لأ)
2. إيه الفرق بين `Collections.unmodifiableList` و`List.copyOf`؟
3. إزاي تعمل deep copy في Java؟ (copy constructor, serialization, JSON, manual)
4. ليه `String` immutable؟ (Q12)
5. هل الـ Records immutable بالكامل؟ (لأ، shallow)
6. إزاي تتعامل مع `Date` كـ field في immutable class؟
7. إيه الـ trade-off بين immutability والـ performance؟
8. إزاي الـ immutability بتساعد في الـ concurrency؟

**⚠️ Common Interview Trap:**

- ❌ **"`final` makes the object immutable"** — بيجمّد الـ reference بس.
- ❌ **"`unmodifiableList` is a copy"** — دي view.
- ❌ **"Record is deeply immutable"** — shallow فقط.
- ❌ **"Defensive copy is only needed on input"** — لازم على الـ output كمان لو هترجّع mutable state.
- ❌ **"Immutable objects are always slower"** — مش دايمًا؛ بتقلل synchronization وتقلل bugs.

---

### Q7. Records: إمتى تستخدمها؟ وإيه الفرق بينها وبين Class عادي وLombok؟ وإيه علاقتها بالـ Sealed Types؟

**🎯 Related Concepts:**
Record (Java 16) · Compact Constructor · Canonical Constructor · Shallow Immutability · Value Objects vs Entities · DTO · Sealed Interface · Pattern Matching · Serialization

**🗣️ Interview Answer:**

> A `record` is a concise, transparent carrier for **immutable data**: the compiler generates the canonical constructor, accessors, `equals`, `hashCode`, and `toString` from the components. It's `final`, extends `java.lang.Record`, cannot declare extra instance fields, and cannot extend another class (but can implement interfaces).
>
> Use records for **value objects and DTOs** (Money, Point, DateRange, API responses), not for JPA entities: entities need mutable state, proxies, no-arg constructors, and identity-based lifecycle. Records are only **shallowly immutable** — a `List` component is still mutable, so you need a compact constructor with `List.copyOf`.
>
> Combined with `sealed interface`, records let you model **algebraic data types** and write exhaustive `switch` expressions.

**🔍 Deep Dive:**

**1️⃣ What the Compiler Generates:**

```java
record Point(int x, int y) { }
```

بيتحول تقريبًا لـ:

```java
final class Point extends java.lang.Record {
    private final int x;
    private final int y;

    Point(int x, int y) { this.x = x; this.y = y; }

    public int x() { return x; }
    public int y() { return y; }

    @Override public boolean equals(Object o) { /* component-wise */ }
    @Override public int hashCode() { /* component-wise */ }
    @Override public String toString() { /* Point[x=1, y=2] */ }
}
```

**2️⃣ القيود المهمة:**

| Allowed | Not Allowed |
|---|---|
| ✅ Implement interfaces | ❌ Extend another class |
| ✅ Static fields/methods | ❌ Extra instance fields |
| ✅ Additional methods | ❌ Mutable component setters (by design) |
| ✅ Nested records | ❌ `native` methods |
| ✅ Generic records | ❌ Abstract records |

**3️⃣ Compact Constructor — مكان الـ Validation:**

```java
record Email(String value) {
    Email {
        if (value == null || !value.contains("@"))
            throw new IllegalArgumentException("invalid email");
        value = value.toLowerCase(); // reassignment allowed here
    }
}
```

الـ compact constructor **مش** بيعيد تعريف الحقول؛ هو بيسمح لك تتحقق وتعدّل القيم قبل الإسناد التلقائي.

**4️⃣ Canonical vs Overloaded Constructors:**

```java
record Range(int from, int to) {
    Range(int to) {
        this(0, to); // ✅ delegate to canonical
    }

    Range {
        if (from > to) throw new IllegalArgumentException();
    }
}
```

**5️⃣ Accessors — مش `getX()`:**

```java
var p = new Point(1, 2);
p.x(); // ✅
// p.getX(); ❌
```

ده مقصود: records بتتبع convention مختلف عن JavaBeans. لو محتاج JavaBean-style للـ frameworks، استخدم class عادي أو DTO.

**6️⃣ Records vs Lombok vs Normal Class:**

| | Record | Lombok `@Value` | Normal Class |
|---|---|---|---|
| **Boilerplate** | Compiler-generated | Annotation processor | Manual |
| **Immutability** | Shallow by default | Shallow | Manual |
| **Inheritance** | ❌ Can't extend class | ✅ Can | ✅ |
| **Extra fields** | ❌ Instance fields | ✅ | ✅ |
| **Frameworks (JPA)** | ❌ Not suitable | ⚠️ Depends | ✅ |
| **Serialization** | ✅ Component-based | Manual | Manual |
| **Pattern matching** | ✅ Deconstruction | ❌ | ❌ |

**7️⃣ Records + Sealed = Algebraic Data Types:**

```java
sealed interface PaymentResult permits Success, Failure, Pending { }

record Success(String transactionId) implements PaymentResult { }
record Failure(String code, String message) implements PaymentResult { }
record Pending(String reference) implements PaymentResult { }

static String describe(PaymentResult result) {
    return switch (result) {
        case Success s -> "paid: " + s.transactionId();
        case Failure f -> "failed: " + f.code();
        case Pending p -> "pending: " + p.reference();
    }; // exhaustive — no default needed
}
```

الـ compiler بيتأكد إن الـ switch غطّى كل الـ permitted subtypes. ده بيخلي إضافة نوع جديد = compile error في كل مكان محتاج تعديل، وده **أفضل من `default` صامت**.

**8️⃣ Records & Serialization:**

- Records بتتنفذ serialization عن طريق الـ **canonical constructor** والـ components، مش عن طريق `writeObject` الافتراضي.
- الـ `serialVersionUID` مش مطلوب بنفس الطريقة، لكن التوافق بيتأثر بتغيير الـ components.
- الـ validation في الـ compact constructor **بتشتغل** وقت الـ deserialization (ميزة أمان).

**9️⃣ Records vs JPA Entity:**

| Record | JPA Entity |
|---|---|
| Immutable value | Mutable lifecycle |
| No no-arg ctor requirement | Needs no-arg ctor (spec) |
| No proxies | Proxies for lazy loading |
| Identity by value | Identity by PK |
| DTO/projection | Persistent state |

**💻 Practical Example:**

```java
import java.util.List;
import java.util.Objects;

record OrderLine(String sku, int quantity, long unitPriceCents) {
    OrderLine {
        Objects.requireNonNull(sku);
        if (quantity <= 0) throw new IllegalArgumentException("quantity > 0");
        if (unitPriceCents < 0) throw new IllegalArgumentException("price >= 0");
    }

    long totalCents() { return quantity * unitPriceCents; }
}

record Order(String id, List<OrderLine> lines) {
    Order {
        Objects.requireNonNull(id);
        lines = List.copyOf(lines); // defensive copy → real immutability
        if (lines.isEmpty()) throw new IllegalArgumentException("empty order");
    }

    long totalCents() {
        return lines.stream().mapToLong(OrderLine::totalCents).sum();
    }
}
```

**🔁 Follow-up Interview Questions:**

1. هل الـ Record بيمنع الـ inheritance؟ (أيوه، هو `final` ومش بيمتد كلاس)
2. إزاي تعمل validation في Record؟ (compact constructor)
3. هل الـ Record immutable بالكامل؟ (لأ، shallow)
4. إزاي تستخدم Record كـ JPA projection؟ (constructor expression في JPQL)
5. ليه الـ Record مش مناسب كـ JPA entity؟
6. إيه اللي بيحصل لو غيّرت component في Record مستخدم في serialization؟
7. هل ينفع Record يكون `abstract`؟ (لأ)
8. إزاي Record بتساعد في pattern matching؟ (deconstruction)

**⚠️ Common Interview Trap:**

- ❌ **"Record = immutable automatically"** — shallow فقط.
- ❌ **"Record can extend a class"** — لأ، بس implements interfaces.
- ❌ **"Record is a replacement for every DTO"** — لأ، لو محتاج mutability أو JavaBean conventions، استخدم class.
- ❌ **"Records are good JPA entities"** — لأ، لغياب no-arg ctor والـ mutability والـ proxies.
- ❌ **"Compact constructor runs after field assignment"** — لأ، قبل.

---

### Q8. Enums: ليه هي أكتر من مجرد Constants؟ وإزاي تستخدمها في State Machines وSingleton؟

**🎯 Related Concepts:**
`java.lang.Enum` · Private Constructor · Constant-Specific Class Bodies · `values()`/`valueOf()` · `ordinal()` Fragility · `EnumMap`/`EnumSet` · Enum Singleton · Serialization Safety · Switch

**🗣️ Interview Answer:**

> An `enum` in Java is a **special class** whose instances are a fixed set of constants. Each constant is a singleton instance, the constructor is implicitly private, and the class implicitly extends `java.lang.Enum`. Enums can have fields, methods, implement interfaces, and even constant-specific class bodies.
>
> They're ideal for **closed sets of values** (status, roles, payment states), **state machines**, and the **singleton pattern** (one constant is serialization-safe and thread-safe by the JVM). Avoid `ordinal()` for persistence because reordering constants changes the numbers; store an explicit code or the name. Use `EnumMap`/`EnumSet` when the key/domain is an enum — they're faster and more memory-efficient than `HashMap`/`HashSet`.

**🔍 Deep Dive:**

**1️⃣ Enum is a Class:**

```java
enum OrderStatus {
    NEW, PAID, SHIPPED, DELIVERED, CANCELLED;
}
```

بيتحول تقريبًا لـ:

```java
final class OrderStatus extends java.lang.Enum<OrderStatus> {
    public static final OrderStatus NEW = new OrderStatus("NEW", 0);
    // ...
    private OrderStatus(String name, int ordinal) { super(name, ordinal); }
}
```

- الـ constructor **private** (أو package-private، لكن عمليًا private).
- الـ constants هي `public static final`.
- `values()` بترجّع array جديدة كل مرة (مش cached) — خد بالك في الـ hot paths.
- `valueOf(String)` بترمي `IllegalArgumentException` لو الاسم مش موجود.

**2️⃣ Fields, Methods, Interfaces:**

```java
enum HttpStatus implements HasCode {
    OK(200), NOT_FOUND(404), INTERNAL_ERROR(500);

    private final int code;

    HttpStatus(int code) { this.code = code; }

    @Override public int code() { return code; }
}
```

**3️⃣ Constant-Specific Bodies:**

```java
enum Operation {
    PLUS {
        @Override int apply(int a, int b) { return a + b; }
    },
    TIMES {
        @Override int apply(int a, int b) { return a * b; }
    };

    abstract int apply(int a, int b);
}
```

كل constant بتاخد **anonymous subclass** من الـ enum. ده مفيد لـ Strategy pattern، لكن لو الـ logic كبرت، الأفضل تفصلها في classes.

**4️⃣ State Machine:**

```java
enum OrderState {
    NEW {
        @Override OrderState next() { return PAID; }
    },
    PAID {
        @Override OrderState next() { return SHIPPED; }
    },
    SHIPPED {
        @Override OrderState next() { return DELIVERED; }
    },
    DELIVERED {
        @Override OrderState next() { return this; }
    };

    abstract OrderState next();

    boolean canCancel() { return this == NEW || this == PAID; }
}
```

ده بيخلي الانتقالات صريحة ومحصورة، ومفيش `switch` متكرر في كل مكان.

**5️⃣ `ordinal()` — فخ الـ Persistence:**

```java
enum Role { ADMIN, USER }        // ADMIN=0, USER=1
enum Role2 { USER, ADMIN }       // USER=0, ADMIN=1
```

لو خزنت `ordinal()` في الـ database، أي reorder أو إدخال constant جديد بيكسر البيانات القديمة. الحل:

```java
enum Role {
    ADMIN("A"), USER("U");
    private final String code;
    Role(String code) { this.code = code; }
    String code() { return code; }
}
```

**6️⃣ Enum Singleton — الأفضل:**

```java
enum Config {
    INSTANCE;

    private final Map<String, String> values = new java.util.concurrent.ConcurrentHashMap<>();

    String get(String key) { return values.get(key); }
    void put(String key, String value) { values.put(key, value); }
}
```

ليه أفضل من double-checked locking؟

- Thread-safe من غير synchronization يدوي (class initialization).
- Serialization-safe: الـ JVM بيرجّع نفس الـ instance، مش نسخة جديدة.
- Reflection-safe عمليًا (مينفعش تعمل constructor call).
- Cleaner.

**7️⃣ EnumMap / EnumSet:**

```java
EnumMap<OrderStatus, Integer> counts = new EnumMap<>(OrderStatus.class);
EnumSet<Role> adminRoles = EnumSet.of(Role.ADMIN);
```

- `EnumMap` بيستخدم **array** مفهرسة بالـ ordinal، فسريعة جدًا وذاكرتها أقل.
- `EnumSet` bitmask، سريعة جدًا.
- الاتنين **مش thread-safe**، ومش بيقبلوا `null` keys.

**8️⃣ Switch:**

```java
switch (status) {
    case NEW -> System.out.println("new");
    case PAID, SHIPPED -> System.out.println("in progress");
    case DELIVERED -> System.out.println("done");
    case CANCELLED -> System.out.println("cancelled");
}
```

الـ switch on enum **exhaustive** بطبيعته لو غطيت كل القيم (بدون `default`)، وأي constant جديد بيسبب warning/compile error حسب الحالة.

**💻 Practical Example:**

```java
enum PaymentState {
    CREATED {
        @Override PaymentState authorize() { return AUTHORIZED; }
        @Override PaymentState capture() { throw new IllegalStateException("not authorized"); }
    },
    AUTHORIZED {
        @Override PaymentState authorize() { return this; }
        @Override PaymentState capture() { return CAPTURED; }
    },
    CAPTURED {
        @Override PaymentState authorize() { throw new IllegalStateException("already captured"); }
        @Override PaymentState capture() { return this; }
    };

    abstract PaymentState authorize();
    abstract PaymentState capture();
}

final class Payment {
    private PaymentState state = PaymentState.CREATED;

    void authorize() { state = state.authorize(); }
    void capture()   { state = state.capture(); }
    PaymentState state() { return state; }
}
```

**🔁 Follow-up Interview Questions:**

1. هل الـ Enum بيورث كلاس تاني؟ (لأ، هو implicitly extends `Enum`)
2. هل ينفع Enum implements interface؟ (أيوه)
3. إزاي Enum بيضمن الـ Singleton؟ (class initialization + private ctor + serialization)
4. ليه `ordinal()` خطر؟ (reordering / inserting constants)
5. إيه الفرق بين `values()` و`valueOf()`؟ (`values` array of constants، `valueOf` lookup by name)
6. هل `EnumMap` أسرع من `HashMap`؟ (أيوه، array-based)
7. هل الـ Enum constructor ممكن يكون `public`؟ (لأ)
8. إيه اللي يحصل لو ضفت constant جديد لـ enum مستخدم في switch؟ (حسب الـ exhaustiveness)

**⚠️ Common Interview Trap:**

- ❌ **"Enum is just constants"** — هو class كامل.
- ❌ **"Enum can extend a class"** — لأ.
- ❌ **"Enum constructor can be public"** — لأ، private implicitly.
- ❌ **"ordinal is stable for persistence"** — لأ.
- ❌ **"Enum singleton is overkill"** — هو الأنسب في Java.
- ❌ **"EnumMap is thread-safe"** — لأ.

---

### Q9. إيه الفرق بين Static Nested وInner وLocal وAnonymous Classes؟ وإمتى تستخدم كل واحد؟

**🎯 Related Concepts:**
Nested Classes · Implicit Outer Reference · `Outer.this` · Effectively Final Capture · Anonymous Class vs Lambda · Memory Leak · Builder Pattern · `static` Members in Inner Classes (Java 16)

**🗣️ Interview Answer:**

> - A **static nested class** is a top-level-like class scoped inside another class; it does **not** hold a reference to an outer instance.
> - An **inner (non-static) class** is associated with an instance of the outer class and holds an implicit `Outer.this` reference, which can leak memory if the inner object outlives the outer.
> - A **local class** is declared inside a method and can capture effectively final locals.
> - An **anonymous class** is a one-off implementation, often used for callbacks; it captures effectively final locals and has its own `this`.
>
> Prefer **static nested** for helpers/builders, **lambdas** over anonymous classes for functional interfaces, and avoid long-lived inner classes in caches/listeners unless you manage the outer lifecycle.

**🔍 Deep Dive:**

| Type | Holds outer ref? | Can have static members? | Typical use |
|---|---|---|---|
| **Static nested** | ❌ | ✅ | Builder, helper, nested DTO |
| **Inner** | ✅ `Outer.this` | ⚠️ (Java 16+ allows static members) | Iterator, view, adapter tied to outer |
| **Local** | ✅ if in instance method | ⚠️ (Java 16+) | One-off helper inside method |
| **Anonymous** | ✅ if created in instance context | ❌ | Callback, one-off implementation |

**1️⃣ Static Nested — الـ Default الاختيار:**

```java
public class Order {
    private final String id;

    public Order(String id) { this.id = id; }

    public static final class Builder {
        private String id;

        public Builder id(String id) { this.id = id; return this; }

        public Order build() { return new Order(id); }
    }
}
```

الـ `Builder` مش محتاج `Order` instance، فـ static أفضل: أقل memory، ومفيش خطر leak.

**2️⃣ Inner Class — `Outer.this`:**

```java
public class LinkedList {
    private Node head;

    private static final class Node {
        int value;
        Node next;
    }

    public final class Iterator {
        private Node current = head;

        public boolean hasNext() { return current != null; }

        public int next() {
            int v = current.value;
            current = current.next;
            return v;
        }

        // Outer.this متاح هنا
        int size() { return LinkedList.this.size(); }
    }

    int size() { /* ... */ return 0; }
}
```

الـ `Iterator` محتاج الـ outer instance، فـ inner منطقي. لكن لو خزّنت `Iterator` في مكان طويل العمر، الـ `LinkedList` كلها هتفضل في الذاكرة.

**3️⃣ Local Class:**

```java
void process(List<String> items) {
    int limit = 10;

    class Validator {
        boolean valid(String s) { return s.length() <= limit; }
    }

    var validator = new Validator();
    items.stream().filter(validator::valid).forEach(System.out::println);
}
```

الـ `limit` لازم تكون **effectively final** (مش بالضرورة `final` keyword).

**4️⃣ Anonymous Class:**

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("running");
    }
};
```

- بتعمل class جديد في الـ compile time (مش runtime).
- ليها `this` خاص بيها، فالـ `this` جوه الـ anonymous ≠ الـ outer.
- بتقدر توصل للـ outer بـ `Outer.this`.

**5️⃣ Anonymous vs Lambda:**

| | Anonymous Class | Lambda |
|---|---|---|
| **`this`** | نفس الـ anonymous instance | نفس الـ enclosing instance |
| **State** | ممكن fields | مفيش fields (إلا captured) |
| **Interface** | أي interface/abstract class | Functional interface فقط |
| **Bytecode** | Class لكل استخدام | `invokedynamic` (عادة) |
| **Capture** | effectively final | effectively final |

```java
class Demo {
    void run() {
        Runnable anon = new Runnable() {
            @Override public void run() {
                System.out.println(this.getClass()); // Demo$1
            }
        };

        Runnable lambda = () -> System.out.println(this.getClass()); // Demo
    }
}
```

**6️⃣ Memory Leak الكلاسيكي:**

```java
public class EventBus {
    private final List<Runnable> listeners = new ArrayList<>();

    public void register() {
        listeners.add(new Runnable() { // inner anonymous → holds EventBus.this
            @Override public void run() { handle(); }
        });
    }

    void handle() { }
}
```

لو الـ `EventBus` اتمسح من الـ scope لكن الـ listener متخزن في static registry، الـ `EventBus` هتفضل حية بسبب الـ implicit reference. الحل: static nested + weak reference، أو إلغاء التسجيل.

**7️⃣ Java 16+ — Static Members in Inner Classes:**

قبل Java 16، الـ inner class مكانتش تقدر تعرّف `static` members (إلا الـ constant variables). من Java 16 (JEP 395)، القيد ده اتخفف وأصبح مسموح للـ inner classes تعرّف `static` members، مع إنها لسه مرتبطة بالكلاس الأصلي ومش محتاجة instance عشان الـ static member يشتغل.

**💻 Practical Example:**

```java
import java.util.*;
import java.util.function.Supplier;

public final class Pipeline<T> {
    private final List<Supplier<T>> steps = new ArrayList<>();

    public Pipeline<T> add(Supplier<T> step) {
        steps.add(step);
        return this;
    }

    public T execute() {
        T result = null;
        for (var step : steps) result = step.get();
        return result;
    }

    // static nested: مفيش outer ref
    public static final class Builder<T> {
        private final Pipeline<T> pipeline = new Pipeline<>();

        public Builder<T> add(Supplier<T> step) {
            pipeline.add(step);
            return this;
        }

        public Pipeline<T> build() { return pipeline; }
    }
}
```

**🔁 Follow-up Interview Questions:**

1. ليه الـ inner class بتحتاج outer instance؟
2. إزاي تعمل instance من inner class من بره؟ (`outer.new Inner()`)
3. إيه الفرق بين `Outer.this` و`this` في anonymous class؟
4. ليه الـ lambda مش بتحتاج effectively final بس؟
5. إزاي inner class تسبب memory leak؟
6. هل الـ static nested class بتكون member of outer؟ (أيوه، بس مش محتاجة instance)
7. إمتى تفضل anonymous class على lambda؟ (لو محتاج state/fields أو abstract class أو multiple methods)
8. هل الـ local class ممكن تكون `static`؟ (لأ)

**⚠️ Common Interview Trap:**

- ❌ **"Inner class is just a nested class"** — الفرق هو الـ implicit outer reference.
- ❌ **"Lambda can replace anonymous class everywhere"** — بس للـ functional interfaces.
- ❌ **"Anonymous class `this` refers to outer"** — لأ، ليها `this` خاص.
- ❌ **"Static nested class can't access outer private members"** — تقدر، لأنها member من نفس الـ top-level class.
- ❌ **"Captured locals must be declared final"** — effectively final كفاية.

---

## 🧩 Part 3 — Composition, DI & Design Principles

### Q10. Composition vs Inheritance vs Aggregation: إزاي تصمم علاقة بين كلاسين؟ وإيه دور Dependency Injection؟

**🎯 Related Concepts:**
IS-A vs HAS-A · Composition (Strong Ownership) · Aggregation (Weak Ownership) · Delegation · Fragile Base Class · Constructor Injection · Field Injection · Testability · Circular Dependencies

**🗣️ Interview Answer:**

> Inheritance models **IS-A** and creates tight coupling to the parent's implementation. Composition models **HAS-A** by delegating to another object, and it's usually preferred because it's more flexible and testable.
>
> Composition and aggregation are both HAS-A, but they differ in **lifecycle ownership**:
> - **Composition**: the container owns the part; the part dies with the container (e.g., `Order` owns `OrderLine`).
> - **Aggregation**: the container references an external part that can outlive it (e.g., `Team` has `Player`s).
>
> **DI is not a replacement for composition** — they solve different problems. Composition answers *"what is inside the class?"*; DI answers *"who creates and supplies those dependencies?"*. Constructor injection + composition gives you immutable, testable objects with explicit dependencies.

**🔍 Deep Dive:**

| | Inheritance | Composition | Aggregation |
|---|---|---|---|
| **Relationship** | IS-A | HAS-A (owns) | HAS-A (references) |
| **Coupling** | Tight (implementation) | Loose (interface) | Loose |
| **Lifecycle** | Subtype is a parent | Part dies with whole | Part can outlive whole |
| **Flexibility** | Limited (single parent) | High (swap at runtime) | High |
| **UML** | Hollow triangle | Filled diamond | Hollow diamond |
| **Example** | `SavingsAccount extends Account` | `Order` owns `OrderLine` | `Team` has `Player` |

**1️⃣ ليه Composition أفضل في أغلب الحالات؟**

- **Fragile base class**: تغيير في الـ parent ممكن يكسر الـ child بدون ما تعرف.
- **Single inheritance**: Java بتسمح بميراث واحد، فبتتحبس في اختيار واحد.
- **Testability**: تقدر تستبدل الـ dependency بـ mock بسهولة.
- **Runtime flexibility**: تقدر تغيّر السلوك عن طريق حقن implementation مختلف.

**2️⃣ Delegation — قلب الـ Composition:**

```java
interface Engine {
    void start();
}

final class ElectricEngine implements Engine {
    public void start() { System.out.println("electric"); }
}

final class Car {
    private final Engine engine;

    Car(Engine engine) { this.engine = engine; }

    void start() { engine.start(); } // delegation
}
```

الـ `Car` مش بترث من `Engine`، لكنها بتفوض الشغل. ده بيسمح بـ `Car(new ElectricEngine())` أو `Car(new MockEngine())` في الـ tests.

**3️⃣ Composition vs Aggregation بالكود:**

```java
// Composition: OrderLine مش ليها معنى بدون Order
final class Order {
    private final List<OrderLine> lines;

    Order(List<OrderLine> lines) {
        this.lines = List.copyOf(lines);
    }
}

// Aggregation: Player موجودة قبل Team وبعدها
final class Team {
    private final List<Player> players;

    Team(List<Player> players) {
        this.players = List.copyOf(players);
    }
}
```

الفرق مش في الكود نفسه، لكن في **ملكية الـ lifecycle** والقواعد الـ domain.

**4️⃣ DI — إزاي بتوصّل الـ dependencies؟**

| Type | Example | Verdict |
|---|---|---|
| **Constructor injection** | `Car(Engine e)` | ✅ Best: immutable, explicit, testable |
| **Setter injection** | `setEngine(e)` | ⚠️ Optional dependencies only |
| **Field injection** | `@Autowired Engine e` | ❌ Hidden deps, hard to test, reflection magic |
| **Method injection** | `execute(Engine e)` | ✅ For per-call dependencies |

**5️⃣ Composition + DI = الحل الكامل:**

```java
final class OrderService {
    private final OrderRepository repository;
    private final PaymentGateway gateway;
    private final Clock clock;

    OrderService(OrderRepository repository, PaymentGateway gateway, Clock clock) {
        this.repository = Objects.requireNonNull(repository);
        this.gateway = Objects.requireNonNull(gateway);
        this.clock = Objects.requireNonNull(clock);
    }

    Order place(OrderRequest request) {
        var order = Order.create(request, clock.instant());
        gateway.charge(order.total(), request.paymentMethod());
        return repository.save(order);
    }
}
```

- الـ `OrderService` **بتملك** الـ references (composition).
- الـ **DI container** (أو الـ `main`) هو اللي بيcreate ويpassهم (DI).
- الـ `Clock` injected عشان الـ tests تتحكم في الوقت.

**6️⃣ Circular Dependencies:**

```java
class A { A(B b) {} }
class B { B(A a) {} }
```

- Constructor injection هيكشف الدورة فورًا (compile/runtime).
- الحل: إعادة تصميم الحدود (SRP)، أو event/mediator، أو `@Lazy` كحل مؤقت (مش تصميم).
- الدورة عادة علامة إن الكلاسين المفروض يكونوا واحد أو فيه طبقة ناقصة.

**💻 Practical Example:**

```java
interface Notifier {
    void send(String to, String message);
}

final class EmailNotifier implements Notifier {
    public void send(String to, String message) {
        System.out.println("EMAIL to " + to + ": " + message);
    }
}

final class FakeNotifier implements Notifier {
    final List<String> sent = new ArrayList<>();
    public void send(String to, String message) { sent.add(to + ":" + message); }
}

final class SignupService {
    private final Notifier notifier;

    SignupService(Notifier notifier) { this.notifier = notifier; } // composition + DI

    void signup(String email) {
        // ... persist user ...
        notifier.send(email, "Welcome!");
    }
}

// Production
var service = new SignupService(new EmailNotifier());

// Test
var fake = new FakeNotifier();
new SignupService(fake).signup("ali@example.com");
assert fake.sent.size() == 1;
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين Composition وAggregation من ناحية الـ lifecycle؟
2. ليه Inheritance بتعتبر tight coupling؟
3. إمتى يكون Inheritance مبرر؟ (IS-A حقيقي + سلوك ثابت + substitutability)
4. إيه عيوب Field Injection؟
5. إزاي تحل Circular Dependency؟
6. هل DI لازم يكون framework؟ (لأ، ممكن manual wiring في الـ main)
7. إيه الفرق بين DI وService Locator؟
8. إزاي Constructor Injection بيساعد الـ unit testing؟

**⚠️ Common Interview Trap:**

- ❌ **"Composition is just having a field"** — لازم delegation وownership واضحين.
- ❌ **"DI and composition are alternatives"** — هما مكملين لبعض.
- ❌ **"Field injection is fine because Spring handles it"** — بيخفي الـ dependencies وبيصعّب الـ testing.
- ❌ **"Inheritance is always bad"** — لأ، لكنها أداة قوية لازم تُستخدم بحذر.
- ❌ **"Aggregation and composition are the same"** — الفرق في ملكية الـ lifecycle.

---

### Q11. اشرح SOLID بمثال Backend واحد، وإزاي الـ Design Patterns بتطبقها عمليًا؟

**🎯 Related Concepts:**
Single Responsibility · Open/Closed · Liskov Substitution · Interface Segregation · Dependency Inversion · Strategy · Factory · Builder · Singleton · Observer · Decorator · Template Method · Proxy

**🗣️ Interview Answer:**

> SOLID is a set of guidelines for managing dependencies and extension:
> - **SRP**: a class should have one reason to change.
> - **OCP**: open for extension, closed for modification.
> - **LSP**: subtypes must be substitutable for their base types.
> - **ISP**: clients shouldn't depend on methods they don't use.
> - **DIP**: depend on abstractions, not concrete implementations.
>
> Patterns are reusable solutions to recurring design problems, and most of them are just SOLID in action: **Strategy** enables OCP, **Factory** centralizes creation, **Builder** handles complex construction, **Observer** decouples events, **Decorator** adds behavior without modifying, **Proxy** controls access, and **Singleton** ensures one instance.

**🔍 Deep Dive:**

**1️⃣ SRP — Single Responsibility:**

```java
// ❌ Three reasons to change: persistence, formatting, emailing
class InvoiceService {
    void save(Invoice i) { }
    String format(Invoice i) { return ""; }
    void email(Invoice i) { }
}

// ✅ Split
class InvoiceRepository { void save(Invoice i) { } }
class InvoiceFormatter { String format(Invoice i) { return ""; } }
class InvoiceMailer { void email(Invoice i) { } }
```

"Reason to change" مش "number of methods" — الـ class ممكن يكون كبير بس مسؤول عن حاجة واحدة.

**2️⃣ OCP — Open/Closed:**

```java
interface Discount {
    Money apply(Order order);
}

class VipDiscount implements Discount { public Money apply(Order o) { return o.total().multiply(0.2); } }
class SeasonalDiscount implements Discount { public Money apply(Order o) { return o.total().multiply(0.1); } }

class Checkout {
    private final Discount discount;
    Checkout(Discount d) { this.discount = d; }
    Money total(Order o) { return o.total().minus(discount.apply(o)); }
}
```

إضافة `RamadanDiscount` مش بتلمس `Checkout`.

**3️⃣ LSP — Liskov Substitution:**

أي `Discount` لازم تشتغل مكان أي `Discount` تانية بدون ما تكسر `Checkout`. لو `NullDiscount.apply()` بترمي exception، يبقى بتكسر العقد.

**4️⃣ ISP — Interface Segregation:**

```java
// ❌ Fat interface
interface Worker {
    void work();
    void eat();
}

// ✅ Segregated
interface Workable { void work(); }
interface Eatable { void eat(); }

class Robot implements Workable { public void work() { } }
class Human implements Workable, Eatable {
    public void work() { }
    public void eat() { }
}
```

**5️⃣ DIP — Dependency Inversion:**

```java
// ❌ High-level depends on low-level
class OrderService {
    private final MySqlOrderRepository repo = new MySqlOrderRepository();
}

// ✅ Both depend on abstraction
interface OrderRepository { Order save(Order o); }

class OrderService {
    private final OrderRepository repo;
    OrderService(OrderRepository repo) { this.repo = repo; }
}
```

**6️⃣ Patterns — الخريطة السريعة:**

| Pattern | Problem it solves | Backend Example |
|---|---|---|
| **Strategy** | Swap algorithm at runtime | Discount, payment routing |
| **Factory** | Centralize object creation | `PaymentGatewayFactory` |
| **Builder** | Complex construction | `Order.builder()` |
| **Singleton** | One shared instance | Config, registry |
| **Observer** | Notify multiple listeners | Domain events |
| **Decorator** | Add behavior without modifying | Caching/logging repository |
| **Adapter** | Integrate incompatible APIs | Third-party payment adapter |
| **Template Method** | Fixed algorithm, variable steps | Report exporter (Q2) |
| **Proxy** | Control access / lazy load | Spring AOP, Hibernate proxy |

**7️⃣ Strategy + Factory:**

```java
interface PaymentGateway {
    PaymentResult charge(Money amount);
}

final class StripeGateway implements PaymentGateway {
    public PaymentResult charge(Money amount) { return PaymentResult.ok("stripe"); }
}

final class PaypalGateway implements PaymentGateway {
    public PaymentResult charge(Money amount) { return PaymentResult.ok("paypal"); }
}

final class PaymentGatewayFactory {
    private final Map<String, PaymentGateway> gateways;

    PaymentGatewayFactory(Map<String, PaymentGateway> gateways) {
        this.gateways = Map.copyOf(gateways);
    }

    PaymentGateway forMethod(String method) {
        var gateway = gateways.get(method);
        if (gateway == null) throw new IllegalArgumentException("unsupported: " + method);
        return gateway;
    }
}
```

**8️⃣ Builder — للـ Objects اللي فيها اختيارات كتير:**

```java
final class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;

    private HttpRequest(Builder b) {
        this.url = b.url;
        this.method = b.method;
        this.headers = Map.copyOf(b.headers);
        this.body = b.body;
    }

    static Builder builder(String url) { return new Builder(url); }

    static final class Builder {
        private final String url;
        private String method = "GET";
        private final Map<String, String> headers = new HashMap<>();
        private String body;

        Builder(String url) { this.url = Objects.requireNonNull(url); }

        Builder method(String m) { this.method = m; return this; }
        Builder header(String k, String v) { headers.put(k, v); return this; }
        Builder body(String b) { this.body = b; return this; }

        HttpRequest build() { return new HttpRequest(this); }
    }
}
```

**9️⃣ Singleton — Double-Checked Locking (الصح):**

```java
final class Registry {
    private static volatile Registry instance; // volatile إلزامي
    private final Map<String, String> values = new ConcurrentHashMap<>();

    private Registry() { }

    static Registry getInstance() {
        var local = instance;
        if (local == null) {
            synchronized (Registry.class) {
                local = instance;
                if (local == null) {
                    instance = local = new Registry();
                }
            }
        }
        return local;
    }
}
```

- `volatile` بيمنع reordering اللي ممكن يعرّض instance غير مكتمل.
- الـ **static holder** أبسط وأأمن:

```java
final class Registry {
    private Registry() { }
    private static final class Holder { static final Registry INSTANCE = new Registry(); }
    static Registry getInstance() { return Holder.INSTANCE; }
}
```

- و**Enum singleton** (Q8) أفضل لو ينفع.

**🔟 Decorator — إضافة سلوك بدون تعديل:**

```java
interface UserRepository {
    Optional<User> findById(long id);
}

final class CachedUserRepository implements UserRepository {
    private final UserRepository delegate;
    private final Map<Long, User> cache = new ConcurrentHashMap<>();

    CachedUserRepository(UserRepository delegate) { this.delegate = delegate; }

    public Optional<User> findById(long id) {
        return Optional.ofNullable(cache.computeIfAbsent(id,
                k -> delegate.findById(k).orElse(null)));
    }
}
```

**💻 Practical Example:**

**كل الـ SOLID في سيناريو واحد:**

```java
record Order(long id, Money total) { }

interface OrderRepository { Order save(Order order); }
interface PaymentGateway { void charge(Money amount); }
interface ReceiptSender { void send(Order order); }

final class PlaceOrderUseCase {
    private final OrderRepository repository;
    private final PaymentGateway gateway;
    private final ReceiptSender sender;

    PlaceOrderUseCase(OrderRepository repository,
                      PaymentGateway gateway,
                      ReceiptSender sender) {
        this.repository = repository;
        this.gateway = gateway;
        this.sender = sender;
    }

    Order execute(Order order) {
        gateway.charge(order.total());          // DIP
        var saved = repository.save(order);     // SRP
        sender.send(saved);                     // OCP: swap sender without changes
        return saved;
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين SRP و"كل كلاس يعمل حاجة واحدة"؟
2. إزاي OCP بيتحقق من غير ما تزوّد تعقيد؟
3. إدّيني مثال LSP violation وإزاي تصلحه.
4. إيه الفرق بين Factory وBuilder؟
5. ليه `volatile` ضروري في DCL singleton؟
6. إيه الفرق بين Decorator وProxy؟ (الاتنين بيلفّوا object، لكن Proxy بيتحكم في الوصول، Decorator بيضيف سلوك)
7. إمتى Singleton بيبقى anti-pattern؟
8. إزاي DIP بيختلف عن DI؟

**⚠️ Common Interview Trap:**

- ❌ **"SOLID is a law"** — هي guidelines، والـ over-engineering بيضر.
- ❌ **"Singleton is always good"** — ممكن يبقى global state مخفي وصعب الاختبار.
- ❌ **"Factory = Builder"** — Factory للإنشاء، Builder للبناء التدريجي.
- ❌ **"DIP means using interfaces everywhere"** — المقصود الاتجاه (high-level لا يعتمد على low-level)، مش كثرة الواجهات.
- ❌ **"Decorator and Proxy are the same"** — النية مختلفة.

---

## 🔤 Part 4 — Strings & Wrapper Classes

### Q12. ليه `String` Immutable؟ واشرح String Pool و`new String()` وStringBuilder/StringBuffer.

**🎯 Related Concepts:**
Immutability · String Constant Pool · `intern()` · `==` vs `equals` · Compact Strings (Java 9) · `hashCode` Caching · `StringBuilder` vs `StringBuffer` · String Concatenation · Text Blocks

**🗣️ Interview Answer:**

> `String` is immutable: once created, its value cannot change. That gives **security** (paths, URLs, credentials can't be mutated after validation), **thread safety** (safe to share), **safe caching/hashing** (`hashCode` is computed once and cached), and enables the **string pool**, where identical literals share one instance.
>
> `new String("hello")` always creates a **new heap object**, even if `"hello"` is already in the pool. `intern()` returns the pooled instance. `==` compares references, `equals()` compares content.
>
> For mutable text, use `StringBuilder` (not thread-safe, faster) or `StringBuffer` (synchronized). Concatenation in a loop with `+` creates many intermediate objects, so use `StringBuilder`.

**🔍 Deep Dive:**

**1️⃣ String Representation:**

- قبل Java 9: `char[] value`.
- من Java 9 (Compact Strings): `byte[] value` + `byte coder` (Latin-1 أو UTF-16)، لتقليل الذاكرة.
- `String` هي `final class`، وقيمتها (`value`) `private final`.

**2️⃣ String Pool:**

```java
String a = "hello";          // pool
String b = "hello";          // same pool object
String c = new String("hello"); // new heap object

a == b;        // true
a == c;        // false
a.equals(c);   // true
a == c.intern(); // true
```

- الـ Pool موجود في الـ **Heap** (من Java 7، قبل كده كان PermGen).
- الـ Literals بتتحط في الـ pool وقت class loading.
- `intern()` بترجع الـ pooled instance (أو تضيف لو مش موجودة).

**3️⃣ `new String("hello")` — كام Object؟**

- لو `"hello"` مش موجودة في الـ pool: **2 objects** (literal في الـ pool + heap object).
- لو موجودة: **1 object** (الـ heap object بس).
- الجملة الشهيرة "always creates 2 objects" مش دقيقة؛ الصح "always creates at least one new heap object، وممكن literal object كمان".

**4️⃣ ليه Immutable؟**

| Reason | Explanation |
|---|---|
| **Security** | `File`, `URL`, `Socket`, DB credentials — لو اتغيرت بعد الـ validation، ممكن تحصل ثغرة |
| **Thread safety** | مفيش race conditions على القيمة |
| **Hash caching** | `hashCode` بيتحسب مرة ويتخزن، فسريع كـ HashMap key |
| **Pool safety** | لو الـ String mutable، أي reference للـ pool هتشوف التغيير |
| **Class loading** | أسماء الكلاسات والـ reflection بتعتمد على ثبات الـ String |

**5️⃣ `hashCode` Caching:**

```java
public final class String {
    private int hash; // cached, default 0
    public int hashCode() {
        int h = hash;
        if (h == 0) { /* compute */ hash = h; }
        return h;
    }
}
```

**6️⃣ Concatenation:**

```java
String s = "";
for (int i = 0; i < 10_000; i++) {
    s += i; // ❌ O(n²): each iteration creates a new String
}
```

- الـ `+` في **سطر واحد** بيتحول لـ `StringBuilder` (أو `invokedynamic`/`StringConcatFactory` من Java 9).
- في **loop**، الـ compiler مش بيقدر يدمج، فبتعمل object جديد كل مرة.
- الحل:

```java
var sb = new StringBuilder(10_000);
for (int i = 0; i < 10_000; i++) sb.append(i);
String s = sb.toString();
```

**7️⃣ StringBuilder vs StringBuffer:**

| | StringBuilder | StringBuffer |
|---|---|---|
| **Thread-safe** | ❌ | ✅ (synchronized) |
| **Performance** | أسرع | أبطأ |
| **Use case** | Local/loop building | Shared mutable buffer (نادر) |
| **Default capacity** | 16 | 16 |
| **Growth** | `old * 2 + 2` | `old * 2 + 2` |

في الـ backend، `StringBuilder` هو الافتراضي؛ `StringBuffer` مش محتاجها إلا لو فعلاً بتشارك الـ buffer بين threads (وحتى بعدها، الأفضل تصمم بشكل مختلف).

**8️⃣ Text Blocks (Java 15):**

```java
String json = """
        {
          "name": "Ali",
          "role": "admin"
        }
        """;
```

بتقلل الـ escaping، لكنها **لسه String immutable** عادي.

**9️⃣ `String` Methods المهمة:**

- `strip()` (Unicode-aware) vs `trim()` (≤ U+0020).
- `isBlank()` vs `isEmpty()`.
- `repeat(n)`.
- `lines()`.
- `formatted()`.
- `compareTo` lexicographic.
- `switch` on String (compiler uses hashCode + equals).

**💻 Practical Example:**

```java
public class StringDemo {
    public static void main(String[] args) {
        String a = "java";
        String b = "java";
        String c = new String("java");
        String d = c.intern();

        System.out.println(a == b);       // true
        System.out.println(a == c);       // false
        System.out.println(a == d);       // true
        System.out.println(a.equals(c));  // true

        // بناء نص كبير
        var sb = new StringBuilder(64);
        for (int i = 0; i < 5; i++) sb.append(i).append(',');
        System.out.println(sb); // 0,1,2,3,4,

        // مقارنة آمنة
        System.out.println(Objects.equals(a, c)); // true
    }
}
```

**🔁 Follow-up Interview Questions:**

1. كام object بيتعمل في `new String("hello")`؟
2. إيه الفرق بين `intern()` و`new String()`؟
3. ليه `String` مناسبة كـ HashMap key؟
4. إيه اللي يحصل لو استخدمت `+` في loop؟
5. إيه الفرق بين `StringBuilder` و`StringBuffer`؟
6. هل `String` بتتخزن في الـ Stack ولا Heap؟
7. إيه الفرق بين `trim()` و`strip()`؟
8. ليه `String` مش primitive؟

**⚠️ Common Interview Trap:**

- ❌ **"`new String()` always creates 2 objects"** — depends on the pool.
- ❌ **"`==` works for String content"** — لأ، إلا لو نفس الـ reference/pool.
- ❌ **"String is a char array"** — من Java 9 هي byte array + coder.
- ❌ **"StringBuilder is thread-safe"** — لأ.
- ❌ **"`intern()` is always good"** — ممكن يملأ الـ pool ويأثر على الذاكرة.

---

### Q13. Wrapper Classes وAutoboxing: ليه `Integer a = 127; Integer b = 127; a == b` بتطلع `true` وأحيانًا `false`؟

**🎯 Related Concepts:**
Primitive vs Wrapper · Autoboxing/Unboxing · Integer Cache · `valueOf` vs `parseInt` · `==` vs `equals` · NullPointerException · Overload Resolution · Performance

**🗣️ Interview Answer:**

> Autoboxing converts a primitive to its wrapper automatically, and unboxing does the reverse. The JVM caches `Integer` values from **-128 to 127** (and the same idea for `Byte`, `Short`, `Long`, `Character` up to 127, and `Boolean`), so `Integer.valueOf(127) == Integer.valueOf(127)` is `true`, but `Integer.valueOf(128) == Integer.valueOf(128)` is `false` because they're different objects.
>
> `==` compares references for wrappers, so always use `equals()` for value comparison. Unboxing a `null` wrapper throws `NullPointerException`. Autoboxing also affects overload resolution: the compiler prefers widening a primitive over boxing, and boxing over varargs.

**🔍 Deep Dive:**

**1️⃣ Cache Ranges:**

| Wrapper | Cache |
|---|---|
| `Byte` | -128..127 (all) |
| `Short` | -128..127 |
| `Integer` | -128..127 (configurable with `-XX:AutoBoxCacheMax`) |
| `Long` | -128..127 |
| `Character` | 0..127 |
| `Boolean` | `TRUE`, `FALSE` |

```java
Integer a = 127, b = 127;
System.out.println(a == b); // true (cached)

Integer c = 128, d = 128;
System.out.println(c == d); // false (new objects)

System.out.println(c.equals(d)); // true ✅
```

**2️⃣ `valueOf` vs `parseInt`:**

```java
Integer i1 = Integer.valueOf(100); // returns cached/boxed Integer
int i2 = Integer.parseInt("100");  // returns primitive int
```

- `valueOf` → `Integer` (may use cache).
- `parseInt` → `int` (no object).
- `new Integer(100)` **deprecated** (Java 9) — always creates a new object.

**3️⃣ Autoboxing في الـ Collections:**

```java
List<Integer> list = new ArrayList<>();
list.add(1);        // autoboxing: Integer.valueOf(1)
int x = list.get(0); // unboxing: intValue()
```

كل إضافة/قراءة فيها boxing/unboxing، وده بيأثر على الأداء في الـ hot loops. استخدم primitive-specialized structures لو متاح (مثلاً `int[]`, `IntStream`, Eclipse Collections، fastutil).

**4️⃣ NPE — أشهر فخ:**

```java
Integer count = null;
int x = count;        // ❌ NPE (unboxing)
count++;              // ❌ NPE
if (count > 0) { }    // ❌ NPE
```

الحل:

```java
if (count != null && count > 0) { }
// أو
int safe = count == null ? 0 : count;
```

**5️⃣ Overload Resolution — الترتيب:**

```java
static void f(long x)    { System.out.println("long"); }
static void f(Integer x) { System.out.println("Integer"); }

f(1); // long ← widening beats boxing
```

الترتيب: **exact → widening → boxing → varargs**.

```java
static void f(int... x)      { System.out.println("varargs"); }
static void f(Integer x)     { System.out.println("Integer"); }

f(1); // Integer (boxing beats varargs)
```

**6️⃣ `equals` على Wrappers — نوع مختلف = false:**

```java
Integer i = 1;
Long l = 1L;

i.equals(l); // false ❌ (different classes)
i == l;      // ❌ compile error (incomparable types)
```

**7️⃣ `Float`/`Double` — NaN:**

```java
Double.NaN == Double.NaN;          // false
Double.valueOf(Double.NaN).equals(Double.NaN); // true
```

`Double.equals` بيتعامل مع NaN كـ equal، و`0.0` و`-0.0` كـ different. ده يختلف عن `==`.

**8️⃣ `List.remove` — فخ الـ Overload:**

```java
List<Integer> list = new ArrayList<>(List.of(10, 20, 30));
list.remove(1);              // removes index 1 → [10, 30]
list.remove(Integer.valueOf(10)); // removes value 10 → [30]
```

`remove(int index)` vs `remove(Object o)` — الـ autoboxing/overload بيحدد.

**9️⃣ Performance:**

- Boxing بيخصص objects (إلا المخزّن).
- Unboxing بيعمل null check + method call.
- في الـ tight loops، استخدم primitives.
- الـ `Integer` cache بيقلل الـ allocation للقيم الصغيرة.

**💻 Practical Example:**

```java
import java.util.*;

public class WrapperDemo {
    public static void main(String[] args) {
        Integer a = 100;
        Integer b = 100;
        Integer c = 200;
        Integer d = 200;
        System.out.println(a == b);        // true
        System.out.println(c == d);        // false
        System.out.println(c.equals(d));   // true

        List<Integer> nums = new ArrayList<>(List.of(1, 2, 3));
        nums.remove(1);                    // index
        System.out.println(nums);          // [1, 3]

        Integer maybeNull = null;
        try {
            int bad = maybeNull;           // NPE
        } catch (NullPointerException e) {
            System.out.println("unboxing null!");
        }
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الـ cache range لـ `Integer`؟ وهل ممكن تغيّرها؟
2. ليه `Integer a = 128; Integer b = 128; a == b` false؟
3. إيه الفرق بين `Integer.parseInt` و`Integer.valueOf`؟
4. إزاي الـ autoboxing بيأثر على overload resolution؟
5. إيه اللي يحصل لو عملت unboxing لـ `null`؟
6. هل `Double.NaN == Double.NaN`؟ و`equals`؟
7. ليه `list.remove(1)` بتحذف index مش value؟
8. إمتى تستخدم primitive بدل wrapper؟

**⚠️ Common Interview Trap:**

- ❌ **"`==` compares values for wrappers"** — لأ، references (إلا الـ cached).
- ❌ **"Autoboxing is free"** — لأ، فيه allocation/CPU.
- ❌ **"`Integer` cache is always -128..127"** — الـ upper bound قابل للتغيير.
- ❌ **"`new Integer()` is fine"** — deprecated، استخدم `valueOf`.
- ❌ **"Unboxing null returns 0"** — لأ، NPE.

---

## 🧮 Part 5 — Generics & Type Erasure

### Q14. اشرح Generics من الـ Type Safety للـ Type Erasure، وإزاي تستخدم Wildcards صح (PECS)؟

**🎯 Related Concepts:**
Generic Classes/Methods · Type Parameters · Bounded Types · Type Erasure · Bridge Methods · Wildcards (`? extends`, `? super`) · PECS · Heap Pollution · Raw Types · `@SafeVarargs` · Reification

**🗣️ Interview Answer:**

> Generics provide **compile-time type safety** and eliminate casts, but they're implemented with **type erasure**: type parameters are removed at compile time and replaced by their bounds (or `Object`). That's why you can't do `new T()`, `new T[]`, `instanceof List<String>`, or overload on `List<String>` vs `List<Integer>`.
>
> For wildcards, use **PECS**: **P**roducer **E**xtends, **C**onsumer **S**uper. If a structure only produces values, use `? extends T`; if it only consumes them, use `? super T`. The compiler enforces this with the "producer/consumer" rule.
>
> Erasure also explains **bridge methods** and why generic varargs can cause **heap pollution** warnings.

**🔍 Deep Dive:**

**1️⃣ Generic Class:**

```java
final class Box<T> {
    private T value;

    void set(T value) { this.value = value; }
    T get() { return value; }
}

Box<String> box = new Box<>();
box.set("hello");
String s = box.get(); // no cast
```

**2️⃣ Generic Method:**

```java
static <T extends Comparable<T>> T max(List<T> list) {
    if (list.isEmpty()) throw new IllegalArgumentException();
    T max = list.get(0);
    for (T item : list) if (item.compareTo(max) > 0) max = item;
    return max;
}
```

**3️⃣ Type Erasure — اللي بيحصل فعليًا:**

```java
// Source
List<String> list = new ArrayList<>();
list.add("x");
String s = list.get(0);

// After erasure (تقريبًا)
List list = new ArrayList();
list.add("x");
String s = (String) list.get(0);
```

- `T` → `Object` (أو الـ bound).
- الـ cast بيتحط في الـ bytecode.
- الـ `ArrayList<String>` و`ArrayList<Integer>` نفس الـ class في الـ runtime.

**4️⃣ What Erasure Prevents:**

```java
// ❌ لا يمكن
T instance = new T();
T[] array = new T[10];
if (obj instanceof List<String>) { }

// ❌ same erasure: الاتنين بيتحولوا لـ m(List)
void m(List<String> x) { }
void m(List<Integer> x) { }

// ❌ نفس الكلاس مينفعش implements نفس الواجهة بـ type arguments مختلفة
class C implements Comparable<String>, Comparable<Integer> { }
```

**5️⃣ Bounds:**

```java
// Upper bound
static <T extends Number> double sum(List<T> nums) {
    return nums.stream().mapToDouble(Number::doubleValue).sum();
}

// Multiple bounds: class first, then interfaces
static <T extends Number & Comparable<T>> T max(Collection<T> c) { ... }
```

**6️⃣ Wildcards & PECS:**

```java
// Producer: نقرأ منه فقط
static double sum(List<? extends Number> numbers) {
    double total = 0;
    for (Number n : numbers) total += n.doubleValue();
    return total;
}

// Consumer: نكتب فيه فقط
static void addInts(List<? super Integer> sink) {
    sink.add(1);
    sink.add(2);
}
```

| Wildcard | Read | Write | Use |
|---|---|---|---|
| `List<? extends Number>` | ✅ as `Number` | ❌ (إلا `null`) | Producer |
| `List<? super Integer>` | ✅ as `Object` | ✅ `Integer` | Consumer |
| `List<?>` | ✅ as `Object` | ❌ | Unknown type |

**7️⃣ PECS — المثال الكلاسيكي:**

```java
static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (T item : src) dest.add(item);
}
```

- `src` producer → `extends`.
- `dest` consumer → `super`.

**8️⃣ Bridge Methods:**

```java
class Parent implements Comparable<Parent> {
    public int compareTo(Parent o) { return 0; }
}
```

الـ compiler بيولّد bridge method:

```java
public int compareTo(Object o) {
    return compareTo((Parent) o);
}
```

عشان الـ polymorphism يشتغل مع الـ erased signature. ده اللي بيظهر في الـ stack traces أحيانًا كـ synthetic method.

**9️⃣ Heap Pollution:**

```java
@SafeVarargs
static <T> List<T> listOf(T... items) {
    return List.of(items);
}
```

- الـ generic varargs بيتحول لـ array من `Object`، وممكن يسبب `ClassCastException` لاحقًا.
- `@SafeVarargs` بتقول للـ compiler إن الطريقة آمنة (بتستخدم الـ array للقراءة فقط).
- `List.of(items)` آمن لأنها بتعمل copy.

**🔟 Raw Types:**

```java
List raw = new ArrayList(); // ❌ raw
raw.add("x");
raw.add(1); // يشتغل، لكن بيكسر type safety
```

استخدم `List<Object>` أو `List<?>` بدل raw types. الـ raw types موجودة للتوافق مع pre-Java 5.

**💻 Practical Example:**

```java
import java.util.*;

public class GenericsDemo {
    // Producer
    static double average(List<? extends Number> nums) {
        return nums.stream().mapToDouble(Number::doubleValue).average().orElse(0);
    }

    // Consumer
    static void fillWithZeros(List<? super Integer> list, int count) {
        for (int i = 0; i < count; i++) list.add(0);
    }

    // Generic method + bound
    static <T extends Comparable<T>> T max(List<T> list) {
        return list.stream().max(Comparator.naturalOrder())
                   .orElseThrow(NoSuchElementException::new);
    }

    public static void main(String[] args) {
        System.out.println(average(List.of(1, 2, 3)));       // 2.0
        System.out.println(average(List.of(1.5, 2.5)));      // 2.0

        List<Number> numbers = new ArrayList<>();
        fillWithZeros(numbers, 3);
        System.out.println(numbers); // [0, 0, 0]

        System.out.println(max(List.of("a", "c", "b")));     // c
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه اللي بيتمسح بالظبط في الـ type erasure؟
2. ليه مينفعش `new T[]`؟
3. إيه الفرق بين `List<Object>` و`List<?>`؟
4. ليه `List<String>` مش subtype من `List<Object>`؟
5. إيه الـ bridge methods وليه موجودة؟
6. اشرح PECS بمثال.
7. إيه الـ heap pollution؟
8. إمتى تستخدم `@SafeVarargs`؟
9. هل الـ Generics بتشتغل مع primitives؟ (لأ، لازم wrapper)
10. إيه الفرق بين `? extends T` و`? super T` من ناحية القراءة والكتابة؟

**⚠️ Common Interview Trap:**

- ❌ **"Generics exist at runtime"** — لأ، erasure.
- ❌ **"`List<Object>` accepts `List<String>`"** — لأ، مفيش covariance.
- ❌ **"`? extends T` allows adding"** — لأ، إلا `null`.
- ❌ **"Raw types are just old syntax"** — هي بتكسر type safety.
- ❌ **"Type erasure means no type checks"** — الـ compiler بيعمل checks ويحط casts، لكن الـ runtime معلوماته محدودة.

---

## 📦 Part 6 — Collections & HashMap Internals

### Q15. إزاي تختار الـ Collection المناسبة؟ واشرح ArrayList vs LinkedList وFail-Fast Iterators و`Arrays.asList` vs `List.of`.

**🎯 Related Concepts:**
Collection Hierarchy · List/Set/Queue/Map · Dynamic Array vs Linked Nodes · Amortized Complexity · Fail-Fast vs Fail-Safe · `Iterator.remove` · `Arrays.asList` · `List.of` · `Collections.unmodifiableList` · `subList` View

**🗣️ Interview Answer:**

> Start from the **contract** you need:
> - ordered, index-based, duplicates → `List` (`ArrayList` by default, `LinkedList` only for heavy ends operations).
> - uniqueness → `Set` (`HashSet` for speed, `LinkedHashSet` for insertion order, `TreeSet` for sorting).
> - FIFO/priority → `Queue`/`Deque` (`ArrayDeque`, `PriorityQueue`).
> - key-value → `Map` (`HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap`).
>
> `ArrayList` is a resizable array: O(1) random access, amortized O(1) append, O(n) insert/remove in the middle. `LinkedList` is a doubly-linked list: O(1) at the ends **if you already hold the node**, but O(n) random access and high memory overhead per node.
>
> Iterators are **fail-fast**: structural modification outside the iterator throws `ConcurrentModificationException` (best-effort, not a guarantee). `Arrays.asList` is a **fixed-size view** backed by the array; `List.of` is a truly immutable, null-hostile list.

**🔍 Deep Dive:**

**1️⃣ Hierarchy:**

```text
Iterable
 └── Collection
      ├── List      → ArrayList, LinkedList, Vector, CopyOnWriteArrayList
      ├── Set       → HashSet, LinkedHashSet, TreeSet, EnumSet
      └── Queue     → ArrayDeque, PriorityQueue, BlockingQueue
Map (not a Collection) → HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap
```

**2️⃣ ArrayList vs LinkedList:**

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(i)` | **O(1)** | O(n) |
| `add(end)` | Amortized O(1) | O(1) |
| `add(0, x)` | O(n) | O(1) |
| `remove(0)` | O(n) | O(1) |
| `remove(middle)` | O(n) | O(n) (search) + O(1) unlink |
| Memory | Array + spare capacity | Node object + 2 refs per element |
| Cache locality | ✅ Good | ❌ Poor |

**القاعدة العملية:** `ArrayList` هو الافتراضي. `LinkedList` مفيدة كـ `Deque` (لكن `ArrayDeque` أسرع منها في معظم الحالات).

**3️⃣ ArrayList Growth:**

```java
// Default capacity = 10, allocated lazily on first add
// Growth: oldCapacity + (oldCapacity >> 1) → 1.5x
```

لو عارف الحجم مقدمًا:

```java
var list = new ArrayList<String>(10_000); // avoids repeated resizing
```

**4️⃣ Fail-Fast Iterators:**

```java
var list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    if (s.equals("b")) list.remove(s); // ❌ ConcurrentModificationException
}
```

- الـ `modCount` بيتغير مع أي structural modification.
- الـ iterator بيتحقق منه في `next()`/`remove()`.
- الحل:

```java
var it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("b")) it.remove(); // ✅
}
```

أو `list.removeIf(s -> s.equals("b"));`.

**ملاحظة:** الـ fail-fast **best-effort**، مش ضمانة. مينفعش تعتمد عليه للـ concurrency.

**5️⃣ `Arrays.asList` vs `List.of`:**

```java
String[] arr = {"a", "b"};
var fixed = Arrays.asList(arr);
fixed.set(0, "x");        // ✅
fixed.add("c");           // ❌ UnsupportedOperationException
arr[0] = "y";             // ✅ الـ list view بتتغير

var immutable = List.of("a", "b");
immutable.set(0, "x");    // ❌ UnsupportedOperationException
immutable.add("c");       // ❌
```

| | `Arrays.asList` | `List.of` |
|---|---|---|
| **Size** | Fixed | Fixed |
| **Mutable elements** | ✅ `set` | ❌ |
| **Backed by array** | ✅ view | ❌ copy |
| **Null** | ✅ | ❌ |
| **Primitive array** | ⚠️ `List<int[]>` | ⚠️ same |

**فخ الـ primitives:**

```java
int[] nums = {1, 2, 3};
var list = Arrays.asList(nums);
System.out.println(list.size()); // 1 ❌ (single int[] element)
```

لأن الـ varargs بيعمل `int[][]` مش `Integer[]`. الحل: `Arrays.stream(nums).boxed().toList()`.

**6️⃣ `subList` — View مش Copy:**

```java
var list = new ArrayList<>(List.of("a", "b", "c", "d"));
var sub = list.subList(1, 3); // [b, c] view
list.set(1, "x");
System.out.println(sub); // [x, c] ← changed
list.add("e");           // ❌ possible ConcurrentModificationException on sub
```

**7️⃣ `Collections.unmodifiableList` vs `List.copyOf`:**

```java
var original = new ArrayList<>(List.of("a"));
var view = Collections.unmodifiableList(original);
original.add("b");
System.out.println(view); // [a, b] ← view

var snapshot = List.copyOf(original);
original.add("c");
System.out.println(snapshot); // [a, b] ← unchanged
```

**8️⃣ Thread-Safe Collections:**

| Class | Behavior |
|---|---|
| `Vector`, `Hashtable` | Legacy, synchronized, slow |
| `Collections.synchronizedList` | Wrapper, needs external sync for iteration |
| `CopyOnWriteArrayList` | Snapshot per write, great for read-heavy |
| `ConcurrentHashMap` | High concurrency, no null keys/values |
| `BlockingQueue` | Producer-consumer |

**💻 Practical Example:**

```java
import java.util.*;

public class CollectionChoice {
    public static void main(String[] args) {
        // Read-heavy, index access
        List<String> names = new ArrayList<>(100);
        names.add("Ali");
        names.add("Mona");
        System.out.println(names.get(0));

        // Dedup + insertion order
        Set<String> unique = new LinkedHashSet<>(names);
        unique.add("Ali"); // ignored
        System.out.println(unique); // [Ali, Mona]

        // Sorting
        var sorted = new TreeSet<>(Comparator.reverseOrder());
        sorted.addAll(unique);
        System.out.println(sorted); // [Mona, Ali]

        // Safe removal
        names.removeIf(n -> n.startsWith("M"));
        System.out.println(names); // [Ali]
    }
}
```

**🔁 Follow-up Interview Questions:**

1. ليه `ArrayList` أسرع من `LinkedList` في معظم السيناريوهات؟
2. إيه اللي يحصل لو زوّدت عن capacity الـ ArrayList؟
3. إيه الفرق بين fail-fast وfail-safe؟
4. هل `ConcurrentModificationException` مضمونة؟
5. إيه الفرق بين `Arrays.asList` و`List.of`؟
6. ليه `Arrays.asList(int[])` size = 1؟
7. إيه الفرق بين `subList` و`List.copyOf(subList)`؟
8. إمتى تستخدم `CopyOnWriteArrayList`؟
9. إيه الفرق بين `Collections.synchronizedList` و`CopyOnWriteArrayList`؟

**⚠️ Common Interview Trap:**

- ❌ **"LinkedList is always faster for insert/remove"** — لأ، لأن الوصول للـ node O(n).
- ❌ **"ArrayList grows by doubling"** — 1.5x.
- ❌ **"Fail-fast is guaranteed"** — best-effort.
- ❌ **"`Arrays.asList` is immutable"** — fixed-size، لكن `set` مسموح.
- ❌ **"`List.of` allows null"** — لأ.
- ❌ **"`subList` returns a copy"** — view.

---

### Q16. اشرح HashMap من الداخل: Hashing، Buckets، Collisions، Treeification، Resize، وليه مش Thread-Safe؟

**🎯 Related Concepts:**
Hash Function · Bucket Array · Separate Chaining · Load Factor · Threshold · Resize/Rehash · Treeify (Java 8) · `equals`/`hashCode` · Null Key · `ConcurrentHashMap`

**🗣️ Interview Answer:**

> `HashMap` is an array of buckets. For each key, it computes `hashCode()`, spreads the bits (`h ^ (h >>> 16)`), and maps it to an index with `(n - 1) & hash`. Each bucket holds a linked list of nodes; since Java 8, a bucket with many collisions is converted to a **red-black tree** (treeified) when it reaches 8 entries and the table capacity is at least 64, improving worst-case lookup from O(n) to O(log n).
>
> The default capacity is 16 and the load factor is 0.75, so the threshold is 12; when size exceeds it, the table doubles and entries are rehashed. Average `put/get/remove` is O(1), worst case O(log n) after treeification.
>
> `HashMap` allows one `null` key and multiple `null` values, is **not thread-safe**, and has no guaranteed iteration order. For concurrency, use `ConcurrentHashMap`, which disallows null keys/values.

**🔍 Deep Dive:**

**1️⃣ Structure:**

```java
transient Node<K,V>[] table; // buckets

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next; // linked list
}

static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    // red-black tree node
}
```

**2️⃣ Hash Spreading:**

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

الهدف: خلط البتات العالية مع المنخفضة، لأن الـ index بيستخدم البتات المنخفضة فقط (`(n-1) & hash`).

**3️⃣ Index Calculation:**

```java
int index = (table.length - 1) & hash;
```

- الـ capacity دايمًا **power of two**، فالـ `&` بيعمل modulo أسرع.
- `hash & (n-1)` = `hash % n` لما `n` power of two.

**4️⃣ `put` Flow:**

1. لو `table` فاضية → `resize()` (initial capacity 16).
2. احسب `hash` و`index`.
3. لو الـ bucket فاضي → ضيف `Node`.
4. لو موجود:
   - قارن `hash` ثم `key ==` أو `equals`.
   - لو equal → استبدل القيمة.
   - لو الـ bucket list → أضف في الآخر (Java 8) وتحقق من treeify.
5. `++size > threshold` → `resize()`.

**5️⃣ Collisions & Treeification:**

- **Separate chaining**: كل bucket فيها linked list.
- لما الـ list توصل 8 entries **و** capacity ≥ 64 → `treeifyBin` (red-black tree).
- لو capacity < 64 → `resize()` بدل treeify.
- عند الحذف، لو الـ tree صغرت لـ 6 → untreeify.

**ليه 8 و6؟** توازن بين احتمال التصادم (Poisson) وتكلفة الـ tree. الـ tree أغلى في الذاكرة والـ constant factor.

**6️⃣ Resize:**

```java
newCapacity = oldCapacity << 1; // double
newThreshold = newCapacity * loadFactor;
```

- إعادة توزيع الـ nodes على buckets الجديدة.
- في Java 8، الـ resize بيستخدم خاصية الـ power-of-two: كل node إما تفضل في نفس الـ index أو تروح `index + oldCapacity`، فمفيش إعادة حساب hash كاملة.
- الـ resize مكلف O(n)، لكنه amortized O(1) لكل إدخال.

**7️⃣ Load Factor:**

- default 0.75.
- أقل → collisions أقل، ذاكرة أكتر.
- أعلى → ذاكرة أقل، collisions أكتر.
- `HashMap(int initialCapacity, float loadFactor)`.

**8️⃣ `null` Key:**

```java
map.put(null, "value"); // hash = 0 → bucket 0
```

- `HashMap` بتسمح بـ `null` key واحد و`null` values كتير.
- `Hashtable` و`ConcurrentHashMap` **ما بتسمحش** بـ null keys/values.

**9️⃣ Iteration Order:**

- **مش مضمون**، وممكن يتغير مع resize.
- `LinkedHashMap` بتحافظ على insertion order (أو access order).
- `TreeMap` بترتب بالـ key.

**🔟 Thread Safety:**

```java
var map = new HashMap<>();
// thread A: map.put(k1, v1);
// thread B: map.put(k2, v2);
```

- مفيش synchronization.
- ممكن يحصل **lost update**، **infinite loop** (في Java 7 resize)، أو **corrupted structure**.
- الحل: `ConcurrentHashMap` (CAS + synchronized bins)، أو `Collections.synchronizedMap` (لكن قفل عام).

**1️⃣1️⃣ Capacity Constructor:**

```java
new HashMap<>(1000); // tableSizeFor(1000) = 1024
```

`HashMap` بيقرّب للـ power of two الأعلى. لو عارف عدد الـ entries، استخدم:

```java
int capacity = (int) Math.ceil(expected / 0.75);
new HashMap<>(capacity);
```

**💻 Practical Example:**

```java
import java.util.*;

public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> ages = new HashMap<>();
        ages.put("Ali", 30);
        ages.put("Mona", 25);
        ages.put("Ali", 31); // overwrite
        System.out.println(ages); // {Ali=31, Mona=25}

        System.out.println(ages.get("Ali"));       // 31
        System.out.println(ages.getOrDefault("X", 0)); // 0

        ages.putIfAbsent("Omar", 20);
        ages.computeIfAbsent("Sara", k -> 22);

        // Iteration (order not guaranteed)
        for (var e : ages.entrySet()) {
            System.out.println(e.getKey() + " = " + e.getValue());
        }

        // Null key
        ages.put(null, 99);
        System.out.println(ages.get(null)); // 99
    }
}
```

**🔁 Follow-up Interview Questions:**

1. ليه capacity لازم تكون power of two؟
2. إيه اللي يحصل لو الـ load factor = 1؟
3. إيه الـ treeify threshold؟ وليه 8؟
4. إزاي `HashMap` بيتعامل مع `null` key؟
5. إيه الفرق بين `HashMap` و`Hashtable`؟
6. ليه `HashMap` مش thread-safe؟ وإيه اللي ممكن يحصل؟
7. إزاي `ConcurrentHashMap` بيحقق concurrency أعلى؟
8. إيه الفرق بين `HashMap` و`LinkedHashMap`؟
9. إيه اللي يحصل لو الـ key mutable واتغير بعد الـ put؟
10. إزاي تختار initial capacity مناسبة؟

**⚠️ Common Interview Trap:**

- ❌ **"HashMap is always O(1)"** — متوسط، والـ worst case O(log n) بعد treeify (أو O(n) قبلها).
- ❌ **"Capacity = number of entries"** — capacity هي حجم الـ table، والـ size هو عدد الـ entries.
- ❌ **"HashMap preserves insertion order"** — لأ.
- ❌ **"HashMap allows multiple null keys"** — key واحد بس.
- ❌ **"HashMap is thread-safe for reads"** — مش آمن عمومًا لو فيه writes متزامنة.
- ❌ **"Treeification happens at 8 regardless of capacity"** — لازم capacity ≥ 64.

---

### Q17. HashSet/LinkedHashSet/TreeSet وHashMap/LinkedHashMap/TreeMap: إيه الفروق؟ وإزاي تبني LRU Cache؟

**🎯 Related Concepts:**
Hashing vs Ordering · Red-Black Tree · `Comparable` vs `Comparator` · `compareTo` Consistency with `equals` · Access-Order · `removeEldestEntry` · Null Handling

**🗣️ Interview Answer:**

> The three sets differ by **ordering and underlying structure**:
> - `HashSet` is backed by `HashMap`, O(1) average, no order.
> - `LinkedHashSet` keeps **insertion order** via a doubly-linked list running through the entries.
> - `TreeSet` is backed by a `TreeMap` (red-black tree), sorted, O(log n).
>
> Same idea for maps: `HashMap` unordered, `LinkedHashMap` insertion/access order, `TreeMap` sorted by `Comparable` or `Comparator`.
>
> `Comparable` defines the **natural order** inside the class (`compareTo`), while `Comparator` is an external strategy. `TreeSet`/`TreeMap` use `compareTo`/`compare` for equality, not `equals`, so inconsistency between them can silently drop elements. A `LinkedHashMap` with `accessOrder=true` and `removeEldestEntry` gives you a simple LRU cache.

**🔍 Deep Dive:**

| | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| **Backing** | `HashMap` | `LinkedHashMap` | `TreeMap` |
| **Order** | None | Insertion | Sorted |
| **Complexity** | O(1) avg | O(1) avg | O(log n) |
| **Null** | 1 null | 1 null | ❌ natural ordering |
| **Equality** | `equals`/`hashCode` | `equals`/`hashCode` | `compareTo`/`compare` |

| | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| **Order** | None | Insertion/access | Sorted |
| **Complexity** | O(1) avg | O(1) avg | O(log n) |
| **Null key** | 1 | 1 | ❌ (natural) |
| **Use case** | General | LRU, predictable order | Range queries, sorted |

**1️⃣ `HashSet` — إزاي بتستخدم `HashMap`:**

```java
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

الـ value ثابت (`PRESENT`)، والـ set بتستخدم الـ keys بس.

**2️⃣ `LinkedHashMap` — Insertion vs Access Order:**

```java
var insertion = new LinkedHashMap<String, Integer>();
insertion.put("a", 1);
insertion.put("b", 2);
insertion.put("c", 3);
System.out.println(insertion); // {a=1, b=2, c=3}

insertion.get("a");
System.out.println(insertion); // {a=1, b=2, c=3} (insertion order)

var access = new LinkedHashMap<String, Integer>(16, 0.75f, true);
access.put("a", 1);
access.put("b", 2);
access.put("c", 3);
access.get("a"); // a becomes most-recently-used
System.out.println(access); // {b=2, c=3, a=1}
```

**3️⃣ LRU Cache:**

```java
final class LruCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    LruCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

var cache = new LruCache<String, Integer>(2);
cache.put("a", 1);
cache.put("b", 2);
cache.get("a");       // a is now MRU
cache.put("c", 3);    // evicts b
System.out.println(cache); // {a=1, c=3}
```

**ملاحظة:** `LruCache` مش thread-safe. للـ concurrency، استخدم `ConcurrentHashMap` + قائمة، أو مكتبة (Caffeine).

**4️⃣ `Comparable` vs `Comparator`:**

```java
// Natural order inside the class
final class User implements Comparable<User> {
    private final String name;
    private final int age;

    User(String name, int age) { this.name = name; this.age = age; }

    @Override
    public int compareTo(User other) {
        return Integer.compare(this.age, other.age);
    }
}

// External strategy
var byName = Comparator.comparing(User::getName);
var byAgeDesc = Comparator.comparingInt(User::getAge).reversed();
var complex = Comparator.comparing(User::getName)
                        .thenComparingInt(User::getAge);
```

**5️⃣ الـ Consistency Trap:**

```java
var set = new TreeSet<>(Comparator.comparingInt(String::length));
set.add("aa");
set.add("bb"); // compare == 0 → considered duplicate!
System.out.println(set.size()); // 1 ❌
```

`TreeSet` بتستخدم `compare` مش `equals`، فأي comparator بيرجع 0 يعتبرهم نفس العنصر. لازم الـ comparator يكون **consistent with equals**.

**6️⃣ Range Queries مع `TreeMap`:**

```java
var map = new TreeMap<Integer, String>();
map.put(1, "a");
map.put(5, "e");
map.put(10, "j");

System.out.println(map.subMap(1, true, 5, true)); // {1=a, 5=e}
System.out.println(map.headMap(5));                // {1=a}
System.out.println(map.tailMap(5));                // {5=e, 10=j}
System.out.println(map.firstKey());                // 1
System.out.println(map.lastKey());                 // 10
```

**7️⃣ Null Handling:**

- `HashMap`/`LinkedHashMap`: null key/value مسموح.
- `TreeMap`: null key ممنوع مع natural ordering (NPE)، وnull values مسموحة.
- `Hashtable`/`ConcurrentHashMap`: لا null key ولا value.

**💻 Practical Example:**

```java
import java.util.*;

public class OrderingDemo {
    record Product(String name, int price) { }

    public static void main(String[] args) {
        var products = List.of(
            new Product("pen", 10),
            new Product("book", 50),
            new Product("bag", 100)
        );

        // Sorted by price
        var byPrice = new TreeSet<>(Comparator.comparingInt(Product::price));
        byPrice.addAll(products);
        System.out.println(byPrice);

        // Insertion order
        var insertion = new LinkedHashMap<String, Integer>();
        products.forEach(p -> insertion.put(p.name(), p.price()));
        System.out.println(insertion);
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إزاي `HashSet` بتمنع التكرار؟
2. إيه الفرق بين `LinkedHashMap` insertion order وaccess order؟
3. إزاي تبني LRU cache؟
4. إيه الفرق بين `Comparable` و`Comparator`؟
5. ليه `TreeSet` ممكن تعتبر عنصرين مختلفين duplicates؟
6. إمتى تستخدم `TreeMap` بدل `HashMap`؟
7. هل `TreeMap` بتسمح بـ null key؟
8. إيه الـ complexity بتاعة `TreeMap`؟

**⚠️ Common Interview Trap:**

- ❌ **"TreeSet uses equals"** — تستخدم `compareTo`.
- ❌ **"LinkedHashMap is sorted"** — insertion/access order مش sorted.
- ❌ **"HashSet is ordered"** — لأ.
- ❌ **"Comparator can be inconsistent with equals"** — ممكن، لكنه بيكسر `TreeSet`/`TreeMap`.
- ❌ **"LRU cache is thread-safe"** — `LinkedHashMap` مش thread-safe.

---

### Q18. Queue/Deque/PriorityQueue/BlockingQueue: إمتى تستخدم كل واحد؟ وإيه الفرق بين `add` و`offer`؟

**🎯 Related Concepts:**
FIFO vs LIFO · `Queue` Contract · `Deque` · `ArrayDeque` vs `Stack` · Binary Heap · `PriorityQueue` · `BlockingQueue` · Producer-Consumer · `DelayQueue`

**🗣️ Interview Answer:**

> `Queue` is FIFO; `Deque` is a double-ended queue that can act as both FIFO and LIFO. The API has two flavors: `add/remove/element` throw exceptions on failure, while `offer/poll/peek` return `false`/`null`.
>
> `ArrayDeque` is the best general-purpose queue/stack: resizable array, no synchronization, no nulls, faster than `LinkedList` and `Stack`. `PriorityQueue` is a binary heap: `offer/poll` are O(log n), `peek` is O(1), but iteration is **not sorted**.
>
> `BlockingQueue` adds blocking `put/take` for producer-consumer; implementations include `ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`, `PriorityBlockingQueue`, and `DelayQueue`.

**🔍 Deep Dive:**

**1️⃣ Queue API:**

| Operation | Throws | Returns special value |
|---|---|---|
| Insert | `add(e)` | `offer(e)` → `false` |
| Remove | `remove()` | `poll()` → `null` |
| Examine | `element()` | `peek()` → `null` |

**2️⃣ Deque API:**

```java
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("a");
deque.addLast("b");
deque.peekFirst(); // a
deque.peekLast();  // b
deque.pollFirst(); // a
deque.pollLast();  // b
```

**3️⃣ ArrayDeque vs Stack vs LinkedList:**

| | `ArrayDeque` | `Stack` | `LinkedList` |
|---|---|---|---|
| **Thread-safe** | ❌ | ✅ (legacy) | ❌ |
| **Null** | ❌ | ✅ | ✅ |
| **Performance** | ✅ Fast | ⚠️ synchronized | ⚠️ node overhead |
| **Use** | Queue/Stack | Legacy | Deque/List |

**قاعدة:** استخدم `ArrayDeque` للـ Stack بدل `Stack` القديمة.

**4️⃣ PriorityQueue:**

```java
var pq = new PriorityQueue<Integer>();
pq.offer(5);
pq.offer(1);
pq.offer(3);

System.out.println(pq);        // [1, 5, 3] ← heap order, not sorted
System.out.println(pq.poll()); // 1
System.out.println(pq.poll()); // 3
```

- Binary heap.
- `offer`/`poll` O(log n), `peek` O(1).
- **مش thread-safe**.
- ممنوع `null`.
- لو محتاج ترتيب كامل، استخدم `TreeSet` أو `Arrays.sort`.

**5️⃣ Custom Comparator:**

```java
var tasks = new PriorityQueue<Task>(
    Comparator.comparing(Task::priority).reversed()
);
```

**6️⃣ BlockingQueue — Producer/Consumer:**

```java
BlockingQueue<Order> queue = new ArrayBlockingQueue<>(100);

// Producer
queue.put(order); // blocks if full

// Consumer
Order order = queue.take(); // blocks if empty
```

| Implementation | Characteristic |
|---|---|
| `ArrayBlockingQueue` | Bounded, array, fair option |
| `LinkedBlockingQueue` | Optionally bounded, linked nodes |
| `SynchronousQueue` | No storage, handoff |
| `PriorityBlockingQueue` | Unbounded, priority |
| `DelayQueue` | Elements available after delay |
| `LinkedTransferQueue` | Transfer/handoff |

**7️⃣ `DelayQueue` — Scheduled Work:**

```java
class DelayedTask implements Delayed {
    private final long readyAt;
    DelayedTask(long delayMs) { this.readyAt = System.currentTimeMillis() + delayMs; }

    public long getDelay(TimeUnit unit) {
        return unit.convert(readyAt - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    public int compareTo(Delayed o) {
        return Long.compare(readyAt, ((DelayedTask) o).readyAt);
    }
}
```

**8️⃣ `BlockingDeque`:**

`LinkedBlockingDeque` بيجمع blocking + double-ended.

**💻 Practical Example:**

```java
import java.util.concurrent.*;

public class ProducerConsumer {
    public static void main(String[] args) throws Exception {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);

        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    queue.put(i);
                    System.out.println("produced " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    System.out.println("consumed " + queue.take());
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين `add` و`offer`؟
2. ليه `ArrayDeque` أفضل من `Stack`؟
3. هل `PriorityQueue` بترتب العناصر؟
4. إيه الـ complexity بتاعة `PriorityQueue`؟
5. إيه الفرق بين `BlockingQueue` و`Queue` العادية؟
6. إمتى تستخدم `SynchronousQueue`؟
7. إزاي تتعامل مع `InterruptedException`؟
8. هل `PriorityQueue` thread-safe؟

**⚠️ Common Interview Trap:**

- ❌ **"PriorityQueue iteration is sorted"** — لأ، heap order.
- ❌ **"Stack is the best stack"** — legacy وsynchronized.
- ❌ **"ArrayDeque allows null"** — لأ.
- ❌ **"Queue.add never fails"** — بترمي exception لو ممتلئة.
- ❌ **"BlockingQueue is just a Queue"** — فيها blocking semantics.

---

## 🚨 Part 7 — Exceptions & Resource Management

### Q19. اشرح Exception Hierarchy وقواعد الـ Checked vs Unchecked، وإزاي تصمم Exception مخصصة وتستخدم try-with-resources صح؟

**🎯 Related Concepts:**
`Throwable` · `Error` vs `Exception` · Checked vs Unchecked · Custom Exceptions · Exception Chaining · Multi-Catch · try-with-resources · Suppressed Exceptions · Fail-Fast vs Fail-Safe APIs

**🗣️ Interview Answer:**

> The hierarchy is `Throwable → Error` and `Throwable → Exception → RuntimeException`.
> - `Error` represents JVM-level problems (`OutOfMemoryError`, `StackOverflowError`); don't catch it.
> - **Checked** exceptions extend `Exception` but not `RuntimeException`; the compiler forces you to catch or declare them (`IOException`, `SQLException`). They model **recoverable, expected** failures at the API boundary.
> - **Unchecked** exceptions extend `RuntimeException`; they model **programming errors** or precondition violations (`NullPointerException`, `IllegalArgumentException`).
>
> For custom exceptions, extend the most meaningful type, add context fields, preserve the cause (`super(message, cause)`), and avoid leaking internal details. Use **try-with-resources** for anything `AutoCloseable`: it closes resources in reverse order and attaches close failures as **suppressed** exceptions instead of losing the original one.

**🔍 Deep Dive:**

**1️⃣ Hierarchy:**

```text
Throwable
├── Error
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── AssertionError
└── Exception
    ├── IOException              (checked)
    ├── SQLException             (checked)
    ├── ReflectiveOperationException (checked)
    └── RuntimeException         (unchecked)
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── IllegalStateException
        ├── IndexOutOfBoundsException
        └── ConcurrentModificationException
```

**2️⃣ Checked vs Unchecked:**

| | Checked | Unchecked |
|---|---|---|
| **Extends** | `Exception` (not `RuntimeException`) | `RuntimeException` |
| **Compiler enforcement** | ✅ catch or declare | ❌ |
| **Represents** | Recoverable external failure | Programming error / invalid state |
| **Examples** | `IOException`, `SQLException` | `NPE`, `IllegalArgumentException` |
| **API design** | Expected failure mode | Bug / precondition |

**3️⃣ Custom Exception:**

```java
public class InsufficientFundsException extends RuntimeException {
    private final long accountId;
    private final long requested;
    private final long available;

    public InsufficientFundsException(long accountId, long requested, long available) {
        super("Account %d: requested %d but only %d available"
                .formatted(accountId, requested, available));
        this.accountId = accountId;
        this.requested = requested;
        this.available = available;
    }

    public long accountId() { return accountId; }
    public long requested() { return requested; }
    public long available() { return available; }
}
```

- لو الـ caller **متوقع** يعالجها (مثلاً API boundary) → ممكن checked.
- لو هي **bug/precondition** → unchecked.
- في الـ Spring/Backend، غالبًا unchecked + `@ControllerAdvice` بيحوّلها لـ HTTP response.

**4️⃣ Exception Chaining:**

```java
try {
    repository.save(order);
} catch (SQLException e) {
    throw new OrderPersistenceException("failed to save order " + order.id(), e);
}
```

- الـ cause بيحفظ الـ stack trace الأصلي.
- `getCause()` / `getStackTrace()`.
- متعملش `throw new RuntimeException(e.getMessage())` لأنك بتضيع الـ cause.

**5️⃣ try-with-resources:**

```java
try (var in = new FileInputStream("data.bin");
     var out = new FileOutputStream("copy.bin")) {
    in.transferTo(out);
} // out.close() then in.close()
```

- بيقفل بترتيب **عكسي**.
- لو الـ body رمى exception والـ close رمى كمان، الـ close exception بتتحط في **suppressed**.
- `getSuppressed()` بترجعها.

```java
try (var resource = new FailingResource()) {
    throw new RuntimeException("body");
} catch (RuntimeException e) {
    for (Throwable s : e.getSuppressed()) {
        System.out.println("suppressed: " + s);
    }
}
```

**6️⃣ Multi-Catch:**

```java
try {
    process();
} catch (IOException | SQLException e) {
    log.error("failed", e);
}
```

- الـ variable `e` effectively final.
- مينفعش أنواع ليها علاقة inheritance (مثلاً `IOException | FileNotFoundException`).

**7️⃣ Best Practices:**

- ❌ متعملش `catch (Exception e) {}` (swallow).
- ❌ متستخدمش `e.printStackTrace()` في production؛ استخدم logger.
- ✅ سجّل الـ context (order id, user id) مش بس الرسالة.
- ✅ استخدم exceptions للـ **exceptional** حالات، مش control flow.
- ✅ عرّف exceptions على حدود الـ module، وغلّف الـ low-level.
- ✅ استخدم `Objects.requireNonNull` للـ preconditions.
- ❌ مترميش `Throwable`/`Error`.
- ✅ في الـ streams، الـ checked exceptions مش مسموحة في الـ lambda؛ استخدم wrapper أو `UncheckedIOException`.

**8️⃣ Exceptions في الـ Lambdas/Streams:**

```java
// ❌ compile error: checked exception in lambda
list.stream().map(s -> readFile(s)); // if readFile throws IOException

// ✅ wrap
list.stream().map(s -> {
    try { return readFile(s); }
    catch (IOException e) { throw new UncheckedIOException(e); }
});
```

**💻 Practical Example:**

```java
import java.io.*;
import java.nio.file.*;

public class FileService {
    public String read(String path) {
        try {
            return Files.readString(Path.of(path));
        } catch (NoSuchFileException e) {
            throw new IllegalArgumentException("file not found: " + path, e);
        } catch (IOException e) {
            throw new UncheckedIOException("failed to read " + path, e);
        }
    }

    public void copy(String from, String to) {
        try (var in = Files.newInputStream(Path.of(from));
             var out = Files.newOutputStream(Path.of(to))) {
            in.transferTo(out);
        } catch (IOException e) {
            throw new UncheckedIOException("copy failed", e);
        }
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين `Error` و`Exception`؟
2. ليه `NullPointerException` unchecked؟
3. إمتى تعمل custom checked exception؟
4. إيه اللي بيحصل لو الـ `close()` رمى exception جوه try-with-resources؟
5. إيه الـ suppressed exceptions؟
6. إيه الفرق بين `throw` و`throws`؟
7. إزاي تتعامل مع checked exception في stream؟
8. ليه `catch (Exception)` ممارسة سيئة؟
9. إيه الفرق بين `final`, `finally`, `finalize`؟ (Q20)

**⚠️ Common Interview Trap:**

- ❌ **"Checked exceptions are always better"** — لأ، بتزوّد الـ boilerplate وبتكسر الـ functional style.
- ❌ **"Catch Exception is safe"** — بتبلع bugs وبتخفي الـ Errors.
- ❌ **"printStackTrace is logging"** — لأ، بيروح stderr ومش منظم.
- ❌ **"finally always runs"** — إلا مع `System.exit`/crash (Q20).
- ❌ **"Custom exception must extend Exception"** — ممكن RuntimeException حسب الحاجة.

---

### Q20. إيه اللي بيحصل بالظبط في `finally`؟ وإيه حكاية `return` جواه و`finalize()`؟

**🎯 Related Concepts:**
`finally` Semantics · Exception Replacement · Return Override · Suppressed Exceptions · `System.exit` · `finalize()` Deprecation · `Cleaner` · Deterministic Cleanup

**🗣️ Interview Answer:**

> `finally` runs after the `try` (and `catch`) regardless of whether an exception was thrown or a `return` happened — **except** when the JVM terminates (`System.exit`, `Runtime.halt`), the process crashes, or the thread is killed.
>
> Two dangerous behaviors:
> - A `return` inside `finally` **overrides** any previous return value and **discards** a pending exception.
> - An exception thrown inside `finally` **replaces** the original exception, so you lose the root cause.
>
> `finalize()` is deprecated for removal; it's non-deterministic, can resurrect objects, and slows GC. Use try-with-resources, `AutoCloseable`, or `java.lang.ref.Cleaner` instead.

**🔍 Deep Dive:**

**1️⃣ Execution Order:**

```java
static int test() {
    try {
        return 1;
    } finally {
        System.out.println("finally");
    }
}
// prints "finally", returns 1
```

**2️⃣ `return` in `finally` — كارثة:**

```java
static int bad() {
    try {
        return 1;
    } finally {
        return 2; // ❌ overrides
    }
}
// returns 2
```

```java
static int worse() {
    try {
        throw new RuntimeException("boom");
    } finally {
        return 42; // ❌ swallows the exception
    }
}
// returns 42, exception lost
```

**3️⃣ Exception in `finally` — بيستبدل الأصل:**

```java
try {
    throw new RuntimeException("original");
} finally {
    throw new IllegalStateException("from finally");
}
// caller sees IllegalStateException; "original" is lost
```

الحل: متعملش throw جوه `finally`، أو استخدم try-with-resources (اللي بيحفظ الأصل كـ suppressed).

**4️⃣ `System.exit` — `finally` مش هتتنفذ:**

```java
try {
    System.exit(0);
} finally {
    System.out.println("never printed");
}
```

كذلك `Runtime.getRuntime().halt(0)`، وJVM crash، و`kill -9`.

**5️⃣ `finally` مع `try-with-resources`:**

```java
try (var r = new Resource()) {
    return 1;
} finally {
    System.out.println("after close");
}
```

الترتيب: body → close() → finally.

**6️⃣ `finalize()` — ليه اتشالت:**

```java
@Override
protected void finalize() throws Throwable {
    // ❌ non-deterministic, may never run
    super.finalize();
}
```

- مش مضمون إنها تشتغل.
- ممكن تعيد إحياء الـ object.
- بتأخر الـ GC.
- بترمي exceptions بدون handling.
- Deprecated since Java 9, deprecated for removal in 18.

**7️⃣ `Cleaner` (Java 9+):**

```java
import java.lang.ref.Cleaner;

final class Resource implements AutoCloseable {
    private static final Cleaner CLEANER = Cleaner.create();
    private final Cleaner.Cleanable cleanable;

    Resource() {
        this.cleanable = CLEANER.register(this, () -> {
            // release native resource
        });
    }

    @Override public void close() { cleanable.clean(); }
}
```

- `Cleaner` بيشتغل كـ safety net، مش بديل عن `close()`.
- الـ cleanup action لازم **ما تمسكش** الـ object نفسه (وإلا مش هيتجمع).

**8️⃣ Best Practice:**

- استخدم try-with-resources لكل `AutoCloseable`.
- متحطش `return`/`throw` في `finally`.
- خلي `finally` للتنظيف البسيط (logging, restore state).
- لو محتاج cleanup مضمون، explicit `close()` في `try`/`finally` أو try-with-resources.

**💻 Practical Example:**

```java
public class FinallyDemo {
    static int safe() {
        int result = 0;
        try {
            result = 1;
            return result;
        } finally {
            System.out.println("cleanup"); // no return here
        }
    }

    static int dangerous() {
        try {
            return 1;
        } finally {
            return 2; // ❌ don't do this
        }
    }

    public static void main(String[] args) {
        System.out.println(safe());      // cleanup, 1
        System.out.println(dangerous()); // 2
    }
}
```

**🔁 Follow-up Interview Questions:**

1. هل `finally` بتتنفذ لو `return` في `try`؟ (أيوه)
2. إيه اللي يحصل لو `finally` فيها `return`؟
3. إيه اللي يحصل لو `finally` رمت exception؟
4. إمتى `finally` مش بتتنفذ؟
5. إيه الفرق بين `finalize` و`Cleaner`؟
6. ليه `finalize` deprecated؟
7. إزاي try-with-resources بتحفظ الـ exception الأصلية؟
8. هل ينفع `finally` من غير `catch`؟ (أيوه)

**⚠️ Common Interview Trap:**

- ❌ **"finally always runs"** — مش مع `System.exit`/crash.
- ❌ **"return in finally is fine"** — بيكسر الـ semantics.
- ❌ **"finally exception is added as suppressed"** — لأ، بيستبدل (إلا في try-with-resources close).
- ❌ **"finalize is a good destructor"** — لأ.
- ❌ **"try-with-resources needs catch"** — لأ، ممكن `try (...) { }` لوحده مع `throws`.

---

## ⚙️ Part 8 — JVM Memory, GC, Class Loading & JIT

### Q21. اشرح Stack vs Heap، أنواع الـ References، وإزاي تحصل Memory Leak في Java؟

**🎯 Related Concepts:**
Stack Frame · Heap Generations · Metaspace · Strong/Soft/Weak/Phantom References · GC Roots · Escape Analysis · `OutOfMemoryError` vs `StackOverflowError` · ThreadLocal Leak

**🗣️ Interview Answer:**

> Each thread has its own **stack**, holding frames with local variables, parameters, and references. Objects live on the **heap**, which is shared across threads and managed by the GC. Class metadata lives in **Metaspace** (since Java 8, replacing PermGen).
>
> References control reachability: **strong** (normal), **soft** (cleared under memory pressure, good for caches), **weak** (cleared at the next GC, used by `WeakHashMap`), and **phantom** (notification only, used for cleanup).
>
> Java can still leak: static collections that grow forever, `ThreadLocal` values in pooled threads, unclosed resources, listeners/caches holding references, and inner classes holding their outer instance. A leak means the object is still **reachable**, so the GC can't collect it.

**🔍 Deep Dive:**

**1️⃣ Stack:**

```java
void a() {
    int x = 10;        // primitive on stack
    User u = new User(); // reference on stack, object on heap
    b();
}
```

- Each thread has its own stack.
- Frame per method call: locals, operand stack, return address.
- LIFO; automatically freed on return.
- `-Xss` controls stack size.
- `StackOverflowError` from deep/infinite recursion.

**2️⃣ Heap:**

- Shared by all threads.
- Objects and arrays.
- Generations: **Young** (Eden + Survivor S0/S1) and **Old/Tenured**.
- `-Xms` initial, `-Xmx` max.
- `OutOfMemoryError: Java heap space` when full.

**3️⃣ Metaspace:**

- Class metadata, method bytecode, constant pool.
- Native memory (not heap), `-XX:MaxMetaspaceSize`.
- `OutOfMemoryError: Metaspace` from classloader leaks / too many generated classes.

**4️⃣ References:**

| Type | Class | When cleared | Use |
|---|---|---|---|
| **Strong** | normal | Never while reachable | Default |
| **Soft** | `SoftReference` | Before OOM | Memory-sensitive cache |
| **Weak** | `WeakReference` | Next GC | `WeakHashMap`, canonical maps |
| **Phantom** | `PhantomReference` | After finalization, before reclaim | Cleanup notification |

```java
var strong = new User();
var weak = new WeakReference<>(new User());
var soft = new SoftReference<>(new User());
var phantom = new PhantomReference<>(new User(), queue);
```

- `WeakHashMap` keys are weak: when the key is no longer strongly referenced, the entry is removed.
- `ThreadLocal` uses weak keys, but **values** can leak in thread pools (call `remove()`).

**5️⃣ GC Roots:**

- Local variables in active stack frames.
- Static fields.
- JNI references.
- Active threads.
- Synchronized monitors.
- Class loaders (in some cases).

أي object **مش reachable** من root → قابل للجمع.

**6️⃣ Memory Leaks الشائعة:**

```java
// 1) Static collection
static final List<Order> ALL = new ArrayList<>(); // grows forever

// 2) ThreadLocal in a pool
static final ThreadLocal<byte[]> BUF = ThreadLocal.withInitial(() -> new byte[1024 * 1024]);
// thread pool thread keeps it alive → must remove()

// 3) Listener not unregistered
eventBus.register(this); // bus holds this

// 4) Inner class holds outer
class Outer { class Inner { } } // Inner → Outer

// 5) Cache without eviction
Map<Key, Value> cache = new HashMap<>(); // unbounded
```

**7️⃣ Escape Analysis:**

الـ JIT ممكن يكتشف إن object مش بيهرب من الـ method، فيعمل **scalar replacement** (يفككه) أو **stack allocation** — يعني مش كل object لازم يروح الـ heap فعليًا. ده optimization مش قاعدة لغوية.

**8️⃣ `OutOfMemoryError` vs `StackOverflowError`:**

| | `OutOfMemoryError` | `StackOverflowError` |
|---|---|---|
| **Cause** | Heap/Metaspace/native memory exhausted | Stack frames exhausted |
| **Typical** | Too many objects, leak, huge array | Infinite recursion |
| **Fix** | Fix leak, increase `-Xmx`, profile | Fix recursion, increase `-Xss` |

**💻 Practical Example:**

```java
import java.lang.ref.WeakReference;
import java.util.*;

public class MemoryDemo {
    public static void main(String[] args) {
        var user = new User("Ali");
        var weak = new WeakReference<>(user);

        System.out.println(weak.get()); // User@...
        user = null;
        System.gc(); // hint only
        System.out.println(weak.get()); // null (likely)

        // Leak-prone cache
        Map<String, byte[]> cache = new HashMap<>();
        for (int i = 0; i < 10_000; i++) {
            cache.put("key-" + i, new byte[1024 * 1024]); // ❌ unbounded
        }
    }

    record User(String name) { }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين Stack وHeap؟
2. إيه اللي بيتخزن في Metaspace؟
3. إيه أنواع الـ References واستخداماتها؟
4. إيه الـ GC roots؟
5. إزاي `ThreadLocal` بيعمل leak؟
6. إيه الفرق بين `OutOfMemoryError` و`StackOverflowError`؟
7. إيه اللي بيعمله `System.gc()`؟
8. إيه Escape Analysis؟

**⚠️ Common Interview Trap:**

- ❌ **"Objects are allocated on the stack"** — الـ JVM ممكن تعمل scalar replacement، لكن ده optimization مش القاعدة.
- ❌ **"GC frees objects immediately"** — غير حتمي.
- ❌ **"System.gc() forces GC"** — hint فقط.
- ❌ **"Java has no memory leaks"** — فيه reachable leaks.
- ❌ **"Weak references are cleared immediately"** — عند الـ GC التالي.

---

### Q22. اشرح Garbage Collection: Generations، Algorithms، Collectors (G1/ZGC)، وإزاي تتعامل مع OOM؟

**🎯 Related Concepts:**
Mark-Sweep-Compact · Copying · Generational Hypothesis · Minor/Major/Full GC · G1 · ZGC · Shenandoah · Stop-the-World · GC Tuning · `-Xmx`/`-XX:+UseG1GC`

**🗣️ Interview Answer:**

> GC finds unreachable objects from **GC roots**, then reclaims them. Most collectors use the **generational hypothesis**: most objects die young. So the heap is split into a young generation (Eden + two Survivor spaces) and an old generation.
>
> - **Minor GC** collects the young generation (copying live objects between survivors, promoting long-lived ones).
> - **Major/Full GC** collects the old generation (mark-sweep-compact), usually more expensive.
>
> Modern default is **G1** (since Java 9): region-based, predictable pauses, compacts by copying. **ZGC** and **Shenandoah** target very low pauses (sub-millisecond to a few ms) for large heaps. Tuning starts with measuring: heap sizing, GC logs, allocation rate, and pause goals — not guessing flags.

**🔍 Deep Dive:**

**1️⃣ Generational Heap:**

```text
┌─────────────────────────────┬──────────────────────┐
│         Young               │        Old           │
│  Eden | S0 | S1             │      Tenured         │
└─────────────────────────────┴──────────────────────┘
```

- New objects → Eden.
- Minor GC: live objects → Survivor، والعمر بيزيد.
- بعد عدد معين من النجاة (tenuring threshold) → Old.
- Humongous objects (G1) → بتتحجز في **humongous regions** وغالبًا بتتعامل كـ old.

**2️⃣ Algorithms:**

| Algorithm | How | Trade-off |
|---|---|---|
| **Mark-Sweep** | Mark reachable, sweep unreachable | Fragmentation |
| **Mark-Sweep-Compact** | + compact live objects | Slower, no fragmentation |
| **Copying** | Copy live to another space | Needs extra space, fast for young |
| **Generational** | Combine by age | Best practical trade-off |

**3️⃣ Collectors:**

| Collector | Flag | Characteristic |
|---|---|---|
| **Serial** | `-XX:+UseSerialGC` | Single-thread, small heaps |
| **Parallel** | `-XX:+UseParallelGC` | Throughput, multi-thread |
| **CMS** | `-XX:+UseConcMarkSweepGC` | Low pause, deprecated/removed |
| **G1** | `-XX:+UseG1GC` | Default since 9, region-based, pause target |
| **ZGC** | `-XX:+UseZGC` | Ultra-low pause, large heaps |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Low pause, concurrent compaction |
| **Epsilon** | `-XX:+UseEpsilonGC` | No-op, benchmarking |

**4️⃣ G1 — كيف بيشتغل:**

- Heap مقسم لـ **regions** (1–32 MB).
- Young regions + old regions + humongous.
- بيعمل **evacuation**: ينسخ الـ live objects من regions مليانة garbage.
- بيعطي **pause target** (`-XX:MaxGCPauseMillis=200`).
- بيختار الـ regions اللي فيها أكبر فايدة (Garbage-First).

**5️⃣ ZGC:**

- Concurrent، low-latency.
- Colored pointers + load barriers.
- Pause عادة < 1ms، مستقل تقريبًا عن حجم الـ heap.
- مش الأفضل للـ throughput الخام.

**6️⃣ Stop-the-World:**

- كل الـ application threads بتتوقف أثناء بعض مراحل الـ GC.
- الـ concurrent collectors بيقللوا الـ STW، لكن مش بيشيلوه بالكامل.

**7️⃣ Tuning — المنهج الصح:**

1. **Measure** قبل أي تغيير: GC logs (`-Xlog:gc*`)، allocation rate، pause times.
2. **Fix leaks first** — زيادة الـ heap بتأجل المشكلة مش بتحلها.
3. **Size the heap**: `-Xms` = `-Xmx` في production لتقليل resizing.
4. **Set a pause goal** مع G1.
5. **اختبر تحت load واقعي**.

**8️⃣ OOM — الأنواع:**

| Error | Cause | Fix |
|---|---|---|
| `Java heap space` | Heap full / leak | Fix leak, increase `-Xmx` |
| `Metaspace` | Classloader leak / too many classes | Fix classloading, increase Metaspace |
| `GC overhead limit exceeded` | GC spends too much time, little reclaimed | Fix leak / heap |
| `Direct buffer memory` | NIO direct buffers | Limit `-XX:MaxDirectMemorySize` |
| `unable to create native thread` | Thread limit / native memory | Reduce threads, check ulimit |

**💻 Practical Example:**

```bash
# Run with G1 and GC logging
java -Xms2g -Xmx2g -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -Xlog:gc*:file=gc.log:time,uptime,level,tags \
     -jar app.jar
```

```java
// Allocation-heavy loop → high GC pressure
for (int i = 0; i < 10_000_000; i++) {
    var s = "order-" + i; // creates many temporary strings
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين Minor وMajor GC؟
2. ليه الـ Generational Hypothesis؟
3. إيه الفرق بين G1 وZGC؟
4. إيه الـ Stop-the-World؟
5. إزاي تقرأ GC logs؟
6. إزاي تحدد heap size مناسب؟
7. إيه أسباب `OutOfMemoryError: Metaspace`؟
8. هل زيادة الـ heap بتحل الـ memory leak؟

**⚠️ Common Interview Trap:**

- ❌ **"GC guarantees no OOM"** — لأ.
- ❌ **"More heap = better"** — ممكن يزوّد pause time ويخفي leak.
- ❌ **"ZGC is always best"** — throughput أقل في بعض الحالات.
- ❌ **"System.gc() is a fix"** — hint فقط.
- ❌ **"finalize frees memory deterministically"** — لأ.

---

### Q23. اشرح Class Loading، Parent Delegation، Bytecode، وJIT. وإيه اللي بيحصل للـ `static` initialization؟

**🎯 Related Concepts:**
ClassLoader · Loading/Linking/Initialization · Parent Delegation · `ClassNotFoundException` vs `NoClassDefFoundError` · Bytecode Instructions · JIT (C1/C2) · Tiered Compilation · Compile-Time Constants · Safe Publication

**🗣️ Interview Answer:**

> Class loading has three phases: **loading** (find and read the `.class`), **linking** (verification, preparation, resolution), and **initialization** (run static initializers). The JVM uses a **parent delegation** model: a classloader asks its parent first, then tries itself, which prevents core classes from being overridden.
>
> Bytecode is a stack-based instruction set (`invokevirtual`, `invokestatic`, `invokedynamic`, etc.), interpreted initially and then compiled by the **JIT** (C1 for fast startup, C2 for optimized code, with tiered compilation). `static` initialization runs **once per classloader**, on first active use, and is thread-safe. Compile-time constants (`static final` primitives/Strings) may be **inlined** and not trigger initialization.

**🔍 Deep Dive:**

**JDK vs JRE vs JVM (سؤال أساسي بيتكرر):**

| Component | Contains |
|---|---|
| **JVM** | Execution engine + runtime data areas + GC + bytecode interpreter/JIT |
| **JRE** | JVM + Java core classes/libraries (needed to **run** a Java app) |
| **JDK** | JRE + development tools (`javac`, `jar`, `javadoc`, `jshell`, debugger) |

من Java 9، الـ modularization خلّى الفرق أقل وضوحًا في التوزيع، بس المفهوم لسه بيتسأل.

**1️⃣ Phases:**

```text
Loading → Linking (Verification → Preparation → Resolution) → Initialization
```

- **Loading**: find the binary, create `Class` object.
- **Verification**: bytecode validity/safety.
- **Preparation**: allocate static fields, default values.
- **Resolution**: symbolic references → direct references.
- **Initialization**: static initializers + static field assignments.

**2️⃣ ClassLoaders:**

| Loader | Loads |
|---|---|
| **Bootstrap** | `java.lang.*`, core JDK |
| **Platform** (Java 9+) | JDK modules |
| **Application/System** | Classpath |
| **Custom** | App servers, plugins |

**Parent Delegation:**

```text
App → Platform → Bootstrap
```

لو الـ parent لاقي الكلاس، بيرجعه. ده بيمنع إن تطبيق يعرّف `java.lang.String` خاص به.

**3️⃣ `ClassNotFoundException` vs `NoClassDefFoundError`:**

- `ClassNotFoundException`: `Class.forName()`/`ClassLoader.loadClass()` ملقاش الكلاس (checked).
- `NoClassDefFoundError`: الكلاس كان موجود وقت الـ compile لكن مش موجود/فشل تحميله وقت الـ runtime (linkage error).

**4️⃣ Static Initialization:**

```java
class Config {
    static final int MAX = 10;                 // compile-time constant
    static final String NAME = "app";          // compile-time constant
    static final List<String> LIST = List.of(); // runtime initialization

    static {
        System.out.println("Config initialized");
    }
}
```

- الـ `static final` primitives/Strings اللي قيمتها **compile-time constant** بتتحط في الـ constant pool وتتinline، فممكن الكلاس **ميتحملش** أصلًا عند استخدامها.
- الـ static block بيتنفذ مرة واحدة عند أول **active use** (new, static method, non-constant static field).
- الـ JVM بتضمن thread safety للـ static init.

**Active use أمثلة:**

- `new Config()`
- `Config.staticMethod()`
- `Config.LIST` (non-constant)
- `Class.forName("Config")`

**مش active use:**

- `Config.MAX` لو compile-time constant (ممكن يتinline).
- الوصول لـ `Config.class`.
- الوراثة من غير instantiation.

**5️⃣ Bytecode:**

```bash
javap -c MyClass
```

```text
0: aload_0
1: invokespecial #1  // Object.<init>
4: return
```

- Stack-based.
- `invokevirtual` (virtual dispatch), `invokestatic`, `invokespecial` (private/super/ctor), `invokeinterface`, `invokedynamic` (lambdas, string concat).
- الـ JIT بيترجم الـ hot methods.

**6️⃣ JIT:**

| Compiler | Purpose |
|---|---|
| **Interpreter** | Immediate execution, low startup |
| **C1 (client)** | Fast compilation, moderate optimization |
| **C2 (server)** | Aggressive optimization, peak performance |
| **Tiered** | C1 → C2 gradually (default) |

Optimizations: inlining, escape analysis, loop unrolling, dead code elimination, branch prediction, devirtualization.

**7️⃣ Deoptimization:**

لو الـ JIT افترض افتراض غلط (مثلاً monomorphic call site بقى polymorphic)، بيرجع للـ interpreter/less optimized code. ده طبيعي ومش bug.

**8️⃣ `final` & Safe Publication:**

- `final` fields ليها **freeze** semantics: أي thread بتقرأ object بعد construction (بشكل صحيح) هتشوف القيم النهائية.
- `volatile`/`synchronized` بتضمن الـ visibility (Q28).

**💻 Practical Example:**

```java
public class ClassInitDemo {
    static class Parent {
        static { System.out.println("Parent static"); }
    }

    static class Child extends Parent {
        static final int CONST = 42; // compile-time constant
        static { System.out.println("Child static"); }
    }

    public static void main(String[] args) {
        System.out.println(Child.CONST); // may NOT initialize Child
        System.out.println("---");
        new Child(); // initializes Parent then Child
    }
}
```

**ملاحظة:** السلوك الفعلي للـ constant inlining بيعتمد على الـ compiler؛ الـ JLS بيسمح بالـ inlining للـ constant variables.

**🔁 Follow-up Interview Questions:**

1. إيه مراحل الـ Class Loading؟
2. إيه Parent Delegation وليه؟
3. إيه الفرق بين `ClassNotFoundException` و`NoClassDefFoundError`؟
4. إمتى الـ static block بيتنفذ؟
5. هل الـ static init thread-safe؟
6. إيه اللي بيتinline من الـ constants؟
7. إيه الفرق بين C1 وC2؟
8. إيه `invokedynamic`؟
9. إيه اللي يحصل في deoptimization؟
10. إزاي `final` بتساعد في safe publication؟

**⚠️ Common Interview Trap:**

- ❌ **"Class is loaded when the JVM starts"** — lazy, on first active use.
- ❌ **"Static blocks run per object"** — مرة واحدة per classloader.
- ❌ **"Compile-time constants trigger initialization"** — عادة لأ (inlined).
- ❌ **"JIT compiles everything"** — hot code فقط.
- ❌ **"NoClassDefFoundError means the class never existed"** — كان موجود وقت الـ compile.

---

## 🧪 Part 9 — Java 8+ : Lambdas, Streams & Optional

### Q24. اشرح Functional Interfaces وLambdas وMethod References، وإزاي بتتعامل مع الـ Capture والـ `this`؟

**🎯 Related Concepts:**
`@FunctionalInterface` · SAM · Built-in Interfaces · Composition (`andThen`/`compose`/`and`/`or`) · Effectively Final Capture · Closures · Method References · Lambda vs Anonymous Class · Checked Exceptions

**🗣️ Interview Answer:**

> A functional interface is an interface with **exactly one abstract method** (SAM). Default, static, and `Object` methods don't count. Java provides the core ones in `java.util.function`: `Function`, `BiFunction`, `Supplier`, `Consumer`, `Predicate`, `UnaryOperator`, `BinaryOperator`, plus primitive specializations.
>
> A lambda is a concise implementation of that single method. It captures local variables only if they are **effectively final** (Java captures the value, not the variable). Unlike an anonymous class, a lambda has **no own `this`** — `this` refers to the enclosing instance. Method references (`Type::method`, `obj::method`, `Type::new`) are shorthand when the lambda only delegates.
>
> Checked exceptions don't fit the standard functional interfaces, so you either wrap them or define a custom functional interface that declares the exception.

**🔍 Deep Dive:**

**1️⃣ Built-in Functional Interfaces:**

| Interface | Method | Meaning |
|---|---|---|
| `Function<T,R>` | `R apply(T)` | Transform |
| `BiFunction<T,U,R>` | `R apply(T,U)` | Two-arg transform |
| `Supplier<T>` | `T get()` | Provide |
| `Consumer<T>` | `void accept(T)` | Consume |
| `Predicate<T>` | `boolean test(T)` | Test |
| `UnaryOperator<T>` | `T apply(T)` | Same-type transform |
| `BinaryOperator<T>` | `T apply(T,T)` | Same-type combine |

Primitive variants: `IntFunction`, `ToIntFunction`, `IntPredicate`, `IntSupplier`, `IntBinaryOperator`, etc. — بتقلل boxing.

**2️⃣ `@FunctionalInterface`:**

```java
@FunctionalInterface
interface Validator<T> {
    boolean validate(T value);

    default Validator<T> and(Validator<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }

    static <T> Validator<T> alwaysTrue() { return value -> true; }
}
```

- لو حطيت method abstract تانية → compile error.
- الـ annotation مش إلزامية، لكنها بتمنع الأخطاء.

**3️⃣ Lambdas:**

```java
Function<String, Integer> length = String::length;
Supplier<List<String>> factory = ArrayList::new;
Consumer<String> printer = System.out::println;
Predicate<String> empty = String::isEmpty;

BinaryOperator<Integer> add = (a, b) -> a + b;
```

**4️⃣ Capture — Effectively Final:**

```java
int limit = 10;
Predicate<Integer> small = n -> n < limit; // ✅ limit effectively final

int counter = 0;
Runnable r = () -> counter++; // ❌ compile error: counter not effectively final
```

- الـ lambda بتقفل على **القيمة** وقت الإنشاء، مش على المتغير.
- لو محتاج تعديل، استخدم `AtomicInteger` أو array/holder (مع الحذر).
- `this` مش captured — هو الـ enclosing instance.

**5️⃣ `this` في Lambda vs Anonymous:**

```java
class Demo {
    void run() {
        Runnable lambda = () -> System.out.println(this);     // Demo
        Runnable anon = new Runnable() {
            public void run() { System.out.println(this); }   // Demo$1
        };
    }
}
```

**6️⃣ Method References — 4 أنواع:**

```java
// 1) Static
Function<String, Integer> parse = Integer::parseInt;

// 2) Bound instance
var list = new ArrayList<String>();
Consumer<String> add = list::add;

// 3) Unbound instance (first arg is receiver)
Function<String, Integer> len = String::length;

// 4) Constructor
Supplier<ArrayList<String>> ctor = ArrayList::new;
```

**7️⃣ Composition:**

```java
Function<Integer, Integer> inc = x -> x + 1;
Function<Integer, Integer> dbl = x -> x * 2;

inc.andThen(dbl).apply(3); // (3+1)*2 = 8
inc.compose(dbl).apply(3); // (3*2)+1 = 7

Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> shortEnough = s -> s.length() < 10;
Predicate<String> valid = notEmpty.and(shortEnough);

Consumer<String> log = System.out::println;
Consumer<String> upperLog = log.andThen(s -> System.out.println(s.toUpperCase()));
```

**8️⃣ Checked Exceptions:**

```java
@FunctionalInterface
interface ThrowingFunction<T, R> {
    R apply(T t) throws Exception;
}

static <T, R> Function<T, R> unchecked(ThrowingFunction<T, R> f) {
    return t -> {
        try { return f.apply(t); }
        catch (Exception e) { throw new RuntimeException(e); }
    };
}
```

**9️⃣ Lambda vs Anonymous Class:**

| | Lambda | Anonymous Class |
|---|---|---|
| **Target** | Functional interface | Any interface/abstract class |
| **`this`** | Enclosing instance | Anonymous instance |
| **State** | No fields | Can have fields |
| **Bytecode** | `invokedynamic` (usually) | Separate class |
| **Capture** | Effectively final | Effectively final |

**💻 Practical Example:**

```java
import java.util.*;
import java.util.function.*;

public class FunctionalDemo {
    public static void main(String[] args) {
        List<String> names = List.of("Ali", "Mona", "Omar", "Sara");

        Predicate<String> startsWithA = s -> s.startsWith("A");
        Function<String, String> upper = String::toUpperCase;
        Consumer<String> print = System.out::println;

        names.stream()
             .filter(startsWithA)
             .map(upper)
             .forEach(print); // ALI

        // Custom composition
        Function<String, Integer> length = String::length;
        Function<Integer, Boolean> gt2 = n -> n > 2;
        var longName = length.andThen(gt2);
        System.out.println(longName.apply("Ali")); // true
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه تعريف الـ Functional Interface؟
2. هل `default` methods بتكسر الـ SAM؟
3. ليه الـ lambda بتقدر تعدّلش المتغير الـ captured؟
4. إيه الفرق بين `andThen` و`compose`؟
5. إيه أنواع الـ Method References؟
6. إيه الفرق بين lambda وanonymous class من ناحية `this`؟
7. إزاي تتعامل مع checked exception في lambda؟
8. إيه فايدة الـ primitive functional interfaces؟
9. هل الـ lambda بتعمل class جديد؟

**⚠️ Common Interview Trap:**

- ❌ **"Any interface with one method is functional"** — لازم **abstract** method واحد بالظبط.
- ❌ **"Lambdas can modify captured locals"** — لأ.
- ❌ **"`this` inside lambda is the lambda"** — لأ، الـ enclosing.
- ❌ **"Method reference always shorter/clearer"** — مش دايمًا؛ لو فيه منطق، lambda أوضح.
- ❌ **"Lambdas are always faster"** — usually, but not guaranteed; `invokedynamic` bootstrap cost.

---

### Q25. اشرح Streams Pipeline: Lazy Evaluation، Short-Circuiting، Parallel Streams، وCollectors.

**🎯 Related Concepts:**
Stream Pipeline · Intermediate vs Terminal · Laziness · Short-Circuit · Single-Use · Stateless vs Stateful Operations · `map` vs `flatMap` · `reduce` · `Collectors` · Parallel Streams · `ForkJoinPool`

**🗣️ Interview Answer:**

> A stream pipeline has a **source**, zero or more **intermediate** operations (lazy), and one **terminal** operation (eager). Nothing runs until the terminal operation is invoked. Intermediate operations are either stateless (`map`, `filter`) or stateful (`sorted`, `distinct`), and some terminal operations are **short-circuiting** (`findFirst`, `anyMatch`, `limit`).
>
> Streams are **single-use**; reusing one throws `IllegalStateException`. `map` is one-to-one, `flatMap` is one-to-many. `reduce` folds to a single value; `collect` gathers into a container using `Collectors`.
>
> Parallel streams use the common `ForkJoinPool` and only help for **CPU-bound, large, easily splittable** data. They hurt with blocking I/O, shared mutable state, ordering costs, and small datasets.

**🔍 Deep Dive:**

**1️⃣ Pipeline:**

```java
list.stream()                 // source
    .filter(s -> !s.isBlank()) // intermediate (lazy)
    .map(String::trim)         // intermediate (lazy)
    .sorted()                  // intermediate (stateful)
    .collect(Collectors.toList()); // terminal (triggers)
```

**2️⃣ Laziness:**

```java
Stream.of("a", "b", "c")
      .peek(System.out::println) // nothing printed yet
      .filter(s -> true);
// no terminal → no execution
```

**3️⃣ Short-Circuit:**

```java
Stream.iterate(1, i -> i + 1)
      .filter(i -> i % 2 == 0)
      .findFirst(); // 2 — stops early

Stream.of(1, 2, 3, 4)
      .anyMatch(i -> i == 2); // true, may not process 3,4
```

**4️⃣ `map` vs `flatMap`:**

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3));
List<Integer> flat = nested.stream()
                           .flatMap(List::stream)
                           .toList(); // [1, 2, 3]
```

**5️⃣ `reduce`:**

```java
int sum = Stream.of(1, 2, 3).reduce(0, Integer::sum);
Optional<Integer> max = Stream.of(1, 2, 3).reduce(Integer::max);
```

- `reduce(identity, accumulator)` → value.
- `reduce(accumulator)` → `Optional`.
- في parallel، لازم يكون الـ accumulator **associative** و**stateless**.

**6️⃣ Collectors:**

```java
var names = users.stream().map(User::name).collect(Collectors.toList());

var byCity = users.stream()
                  .collect(Collectors.groupingBy(User::city));

var counts = users.stream()
                  .collect(Collectors.groupingBy(User::city, Collectors.counting()));

var joined = names.stream().collect(Collectors.joining(", ", "[", "]"));

var avgAge = users.stream().collect(Collectors.averagingInt(User::age));

var partition = users.stream()
                     .collect(Collectors.partitioningBy(u -> u.age() >= 18));
```

**7️⃣ `toMap` — فخ التكرار:**

```java
Map<String, User> byEmail = users.stream()
    .collect(Collectors.toMap(User::email, Function.identity()));
// ❌ IllegalStateException if duplicate email
```

الحل:

```java
.collect(Collectors.toMap(
    User::email,
    Function.identity(),
    (a, b) -> a // or merge
));
```

**8️⃣ `teeing` (Java 12):**

```java
var stats = users.stream().collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.averagingInt(User::age),
    (count, avg) -> new Stats(count, avg)
));
```

**9️⃣ `Stream.toList()` (Java 16):**

```java
var list = stream.toList(); // unmodifiable
```

- `Collectors.toList()` عادة mutable (لكن مش مضمون).
- `toList()` بترجع unmodifiable.

**🔟 Parallel Streams:**

```java
long count = list.parallelStream()
                 .filter(this::expensive)
                 .count();
```

**إمتى ينفع؟**
- ✅ CPU-bound.
- ✅ بيانات كبيرة وقابلة للتقسيم (ArrayList, array, IntStream.range).
- ✅ عمليات stateless وassociative.
- ❌ I/O blocking.
- ❌ shared mutable state.
- ❌ `LinkedList` (splitting سيء).
- ❌ عمليات مرتبة ومكلفة.

**ملاحظة:** الـ parallel streams بتستخدم `ForkJoinPool.commonPool()`، ولو حصل blocking ممكن تعمل starvation لكل التطبيق. في الـ backend، الأفضل `ExecutorService` + `CompletableFuture` للـ IO.

**1️⃣1️⃣ Infinite Streams:**

```java
Stream.iterate(0, i -> i + 2)
      .limit(5)
      .forEach(System.out::println); // 0 2 4 6 8

Stream.generate(Math::random).limit(3);
```

**1️⃣2️⃣ Stateful Operations:**

- `sorted`, `distinct`, `limit`, `skip` بتحتاج تحتفظ بحالة.
- في parallel، بتقلل الفايدة أو بتفرض buffers.

**💻 Practical Example:**

```java
import java.util.*;
import java.util.stream.*;

record Order(String customer, String city, int amount) { }

public class StreamDemo {
    public static void main(String[] args) {
        var orders = List.of(
            new Order("Ali", "Cairo", 100),
            new Order("Mona", "Alex", 200),
            new Order("Omar", "Cairo", 50),
            new Order("Sara", "Alex", 300)
        );

        // Total per city
        Map<String, Integer> totals = orders.stream()
            .collect(Collectors.groupingBy(
                Order::city,
                Collectors.summingInt(Order::amount)));

        System.out.println(totals); // {Alex=500, Cairo=150}

        // Top customer by amount
        var top = orders.stream()
            .max(Comparator.comparingInt(Order::amount))
            .map(Order::customer)
            .orElse("none");

        System.out.println(top); // Sara
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين intermediate وterminal operations؟
2. إيه معنى lazy evaluation؟
3. إيه اللي يحصل لو أعدت استخدام stream؟
4. إيه الفرق بين `map` و`flatMap`؟
5. إمتى parallel stream بتضر؟
6. إزاي تتعامل مع duplicate keys في `toMap`؟
7. إيه الفرق بين `Collectors.toList()` و`Stream.toList()`؟
8. إيه الـ short-circuit operations؟
9. ليه `reduce` لازم يكون associative في parallel؟
10. إيه خطر الـ `peek` في production؟

**⚠️ Common Interview Trap:**

- ❌ **"Streams are always faster"** — لأ، ممكن أبطأ من loop بسيط.
- ❌ **"Stream can be reused"** — single-use.
- ❌ **"Parallel is always faster"** — لأ.
- ❌ **"`forEach` preserves order in parallel"** — لأ، استخدم `forEachOrdered`.
- ❌ **"`peek` is for production logic"** — هو للـ debugging.
- ❌ **"`Collectors.toList()` is immutable"** — عادة mutable، و`toList()` هي unmodifiable.

---

### Q26. Optional: إمتى تستخدمها وإمتى تكون Anti-Pattern؟ واشرح `orElse` vs `orElseGet`.

**🎯 Related Concepts:**
`Optional` as Return Type · `of`/`ofNullable`/`empty` · `map`/`flatMap`/`filter` · `orElse`/`orElseGet`/`orElseThrow` · `ifPresentOrElse` · `or`/`stream` (Java 9) · Null-Hostile · Serialization

**🗣️ Interview Answer:**

> `Optional<T>` is a container that explicitly models **"a value may be absent"**, primarily as a **return type** for methods that can legitimately find nothing. It forces the caller to handle absence instead of risking a `NullPointerException`.
>
> Use `ofNullable` for nullable values, `map` for transformations, `flatMap` when the mapper already returns an `Optional`, and `orElseThrow` when absence is an error.
>
> `orElse` is **eager**: it evaluates the fallback even when the value is present. `orElseGet` is **lazy**: it calls the supplier only when empty. Don't use `Optional` for fields, method parameters, or collections — and never call `get()` without checking.

**🔍 Deep Dive:**

**1️⃣ Creation:**

```java
Optional<String> present = Optional.of("value");
Optional<String> nullable = Optional.ofNullable(maybeNull);
Optional<String> empty = Optional.empty();

Optional.of(null); // ❌ NPE
```

**2️⃣ Core Methods:**

| Method | Purpose |
|---|---|
| `isPresent()` / `isEmpty()` (11) | Check |
| `get()` | Unwrap (avoid) |
| `orElse(T)` | Eager fallback |
| `orElseGet(Supplier)` | Lazy fallback |
| `orElseThrow()` | Throw if empty (10+) |
| `ifPresent(Consumer)` | Act if present |
| `ifPresentOrElse(Consumer, Runnable)` (9) | Both branches |
| `map(Function)` | Transform if present |
| `flatMap(Function<Optional>)` | Avoid nested Optional |
| `filter(Predicate)` | Keep if matches |
| `or(Supplier<Optional>)` (9) | Alternative Optional |
| `stream()` (9) | 0 or 1 element |

**3️⃣ `orElse` vs `orElseGet`:**

```java
String slow() {
    System.out.println("computing...");
    return "fallback";
}

Optional.of("value").orElse(slow());    // prints "computing..." ❌
Optional.of("value").orElseGet(() -> slow()); // does not print ✅
```

- `orElse` بيقيّم الـ argument دايمًا.
- `orElseGet` بيقيّم الـ supplier عند الحاجة فقط.
- لو الـ fallback مكلف (DB call, object creation)، استخدم `orElseGet`.

**4️⃣ `map` vs `flatMap`:**

```java
Optional<User> user = findUser();

Optional<String> city = user.map(User::city);          // Optional<String>
Optional<String> city2 = user.flatMap(User::cityOpt);  // if cityOpt returns Optional
```

**5️⃣ Anti-Patterns:**

```java
// ❌ Field
class User {
    private Optional<String> nickname; // use nullable field + getter returns Optional
}

// ❌ Parameter
void update(Optional<String> name) { } // overload instead

// ❌ Collection of Optional
List<Optional<String>> list; // filter nulls instead

// ❌ isPresent + get
if (opt.isPresent()) return opt.get(); // use map/orElse

// ❌ Optional.of(null)
```

**6️⃣ Proper Usage:**

```java
public Optional<User> findByEmail(String email) {
    return repository.findByEmail(email); // repository returns Optional
}

// Caller
var name = service.findByEmail(email)
                  .map(User::name)
                  .orElse("guest");
```

**7️⃣ `Optional` in Streams:**

```java
var names = users.stream()
                 .map(User::nickname)       // Stream<Optional<String>>
                 .flatMap(Optional::stream) // Stream<String>
                 .toList();
```

**8️⃣ Serialization:**

- `Optional` **not Serializable**.
- متستخدمهاش في DTOs اللي بتتسلسل (Java serialization / بعض الـ frameworks).
- في JSON، الأفضل `null` أو `@JsonInclude(NON_NULL)`.

**9️⃣ Performance:**

- `Optional` بيعمل object allocation.
- في الـ hot paths، ممكن تكون overhead.
- `OptionalInt`/`OptionalLong`/`OptionalDouble` للـ primitives.

**💻 Practical Example:**

```java
import java.util.*;

public class OptionalDemo {
    record User(String email, String nickname) { }

    static Optional<User> find(String email) {
        if ("ali@example.com".equals(email))
            return Optional.of(new User(email, "Ali"));
        return Optional.empty();
    }

    public static void main(String[] args) {
        String name = find("ali@example.com")
                .map(User::nickname)
                .orElse("guest");
        System.out.println(name); // Ali

        String missing = find("x@example.com")
                .map(User::nickname)
                .orElseGet(() -> "guest");
        System.out.println(missing); // guest

        find("x@example.com").ifPresentOrElse(
            u -> System.out.println("found"),
            () -> System.out.println("not found")
        );
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين `orElse` و`orElseGet`؟
2. ليه `Optional.get()` ممارسة سيئة؟
3. هل `Optional` بتتحل مشكلة NPE بالكامل؟
4. إمتى تستخدم `Optional` كـ field/parameter؟
5. إيه الفرق بين `map` و`flatMap`؟
6. إيه `Optional.stream()`؟
7. هل `Optional` Serializable؟
8. إيه الفرق بين `Optional.of` و`ofNullable`؟
9. إمتى تستخدم `OptionalInt`؟

**⚠️ Common Interview Trap:**

- ❌ **"Optional replaces null everywhere"** — هو للـ return types.
- ❌ **"orElse is lazy"** — لأ.
- ❌ **"Optional.get is fine"** — بيكسر الغرض.
- ❌ **"Optional is Serializable"** — لأ.
- ❌ **"Optional<List<T>> is better than empty list"** — لأ، استخدم empty list.

---

## 🧵 Part 10 — Concurrency Essentials

### Q27. اشرح Threads وExecutors وCompletableFuture. وإزاي تحدد حجم Thread Pool مناسب؟

**🎯 Related Concepts:**
Thread Lifecycle · `Runnable` vs `Callable` · `ExecutorService` · Thread Pool Sizing · `Future` · `CompletableFuture` · `thenApply` vs `thenCompose` · Virtual Threads · `ThreadLocal`

**🗣️ Interview Answer:**

> A thread goes through `NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED`. `Runnable` runs without returning a value or throwing checked exceptions; `Callable<V>` returns a value and can throw.
>
> Don't create threads manually — use `ExecutorService`. `execute()` takes a `Runnable`; `submit()` returns a `Future`; `invokeAll`/`invokeAny` run batches. Always shut down the pool (`shutdown()` + `awaitTermination`, or `shutdownNow()`).
>
> Pool sizing: CPU-bound ≈ number of cores; I/O-bound ≈ cores × (1 + wait time / compute time), because threads spend time blocked. For blocking I/O at scale, **virtual threads** (Java 21) are a better fit than huge platform-thread pools.
>
> `CompletableFuture` composes async work: `thenApply` for sync transformation, `thenCompose` for chaining another async call, `thenCombine` for independent results, and `exceptionally`/`handle` for errors.

**🔍 Deep Dive:**

**1️⃣ Thread Lifecycle:**

```text
NEW → RUNNABLE → TERMINATED
         ↕
   BLOCKED / WAITING / TIMED_WAITING
```

- `BLOCKED`: waiting for a monitor lock.
- `WAITING`: `wait()`, `join()`, `park()`.
- `TIMED_WAITING`: `sleep()`, `wait(timeout)`.

**2️⃣ Runnable vs Callable:**

```java
Runnable r = () -> System.out.println("run");
Callable<String> c = () -> "result";
```

| | `Runnable` | `Callable<V>` |
|---|---|---|
| **Return** | `void` | `V` |
| **Checked exceptions** | ❌ | ✅ |
| **Executor** | `execute`/`submit` | `submit` |

**3️⃣ ExecutorService:**

```java
ExecutorService pool = Executors.newFixedThreadPool(8);

Future<Integer> f = pool.submit(() -> 42);
try {
    Integer result = f.get(); // blocks
} finally {
    pool.shutdown();
    pool.awaitTermination(5, TimeUnit.SECONDS);
}
```

**4️⃣ Pool Sizing:**

```text
CPU-bound:  N_threads ≈ N_cores
I/O-bound:  N_threads ≈ N_cores × (1 + wait/compute)
```

- Little's Law: `concurrency = throughput × latency`.
- Too few → queuing.
- Too many → context switching + memory.
- استخدم metrics (queue size, latency) مش تخمين.

**5️⃣ Future Limitations:**

- `get()` blocking.
- No composition.
-No exception handling fluent API.
-No manual completion (pre-Java 8).

**6️⃣ CompletableFuture:**

```java
CompletableFuture<User> userFuture =
    CompletableFuture.supplyAsync(() -> userService.load(id), executor);

CompletableFuture<Order> orderFuture =
    CompletableFuture.supplyAsync(() -> orderService.load(id), executor);

CompletableFuture<Dashboard> dashboard =
    userFuture.thenCombine(orderFuture, Dashboard::new)
              .exceptionally(ex -> Dashboard.empty());
```

| Method | Use |
|---|---|
| `thenApply` | Transform result (sync) |
| `thenCompose` | Chain another `CompletableFuture` (flatMap) |
| `thenCombine` | Combine two independent futures |
| `allOf` | Wait for all |
| `anyOf` | First to complete |
| `exceptionally` | Recover |
| `handle` | Result + exception |
| `whenComplete` | Side effect, preserves result |
| `join` | Like `get`, unchecked |

**7️⃣ Common Pool:**

- `supplyAsync` بدون executor → `ForkJoinPool.commonPool()`.
- الـ common pool مش مناسب للـ blocking I/O (ممكن يعمل starvation).
- مرّر executor مخصص.

**8️⃣ Virtual Threads (Java 21):**

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> blockingIo());
}
```

- خفيفة جدًا (millions ممكنة).
- مناسبة للـ blocking I/O.
- مش مناسبة للـ CPU-bound (الفايدة قليلة).
- Pinning مع `synchronized` في الإصدارات الأقدم (تحسن في Java 24).

**9️⃣ ThreadLocal:**

```java
static final ThreadLocal<SimpleDateFormat> FORMAT =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```

- لكل thread نسخة.
- في thread pools، لازم `remove()` وإلا leak (Q21).

**💻 Practical Example:**

```java
import java.util.concurrent.*;

public class AsyncDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(4);
        try {
            CompletableFuture<String> a =
                CompletableFuture.supplyAsync(() -> "user", pool);
            CompletableFuture<String> b =
                CompletableFuture.supplyAsync(() -> "orders", pool);

            String result = a.thenCombine(b, (x, y) -> x + "+" + y)
                             .thenApply(String::toUpperCase)
                             .exceptionally(ex -> "fallback")
                             .join();

            System.out.println(result); // USER+ORDERS
        } finally {
            pool.shutdown();
        }
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين `Runnable` و`Callable`؟
2. إيه الفرق بين `execute` و`submit`؟
3. إزاي تحدد pool size؟
4. إيه الفرق بين `thenApply` و`thenCompose`؟
5. إيه خطر الـ common pool؟
6. إزاي تتعامل مع exception في `CompletableFuture`؟
7. إيه الفرق بين `shutdown` و`shutdownNow`؟
8. إمتى تستخدم virtual threads؟
9. إيه خطر `ThreadLocal` في pool؟
10. إيه الفرق بين `Future.get` و`join`؟

**⚠️ Common Interview Trap:**

- ❌ **"More threads = more throughput"** — لأ، بعد حد معين بيقل.
- ❌ **"`newCachedThreadPool` is always fine"** — ممكن يخلق threads بلا حدود.
- ❌ **"CompletableFuture uses your executor by default"** — لأ، common pool.
- ❌ **"Virtual threads are for CPU-bound"** — لأ.
- ❌ **"Future.get is non-blocking"** — blocking.
- ❌ **"Thread pool doesn't need shutdown"** — لازم.

---

### Q28. اشرح Java Memory Model: `synchronized` و`volatile` وAtomics وDeadlock.

**🎯 Related Concepts:**
Race Condition · Happens-Before · Monitor Lock · Visibility vs Atomicity · CAS · `AtomicInteger`/`LongAdder` · Deadlock · `ReentrantLock` · `ConcurrentHashMap` · Immutable Objects

**🗣️ Interview Answer:**

> The Java Memory Model defines **happens-before** relationships that guarantee visibility and ordering between threads. Without them, one thread may not see another thread's writes.
>
> - `synchronized` provides **mutual exclusion + visibility**: entering a monitor sees writes from the previous unlock, and leaving publishes your writes.
> - `volatile` provides **visibility + ordering** for a single variable, but **not atomicity** — `count++` is still a race.
> - Atomics (`AtomicInteger`, `LongAdder`) use **CAS** for lock-free atomic updates.
>
> Deadlock happens when threads wait on each other's locks in a cycle. Prevent it with consistent lock ordering, timeouts (`tryLock`), or by reducing shared mutable state. Immutable objects and confinement are the simplest concurrency strategies.

**🔍 Deep Dive:**

**1️⃣ Race Condition:**

```java
class Counter {
    private int count;
    void increment() { count++; } // read-modify-write, not atomic
}
```

`count++` = read + add + write. Two threads ممكن يضيعوا تحديث.

**2️⃣ `synchronized`:**

```java
class Counter {
    private int count;
    synchronized void increment() { count++; }
    synchronized int get() { return count; }
}
```

- Instance method → locks `this`.
- Static method → locks `Class` object.
- Block → locks the given object.
- **Reentrant**: نفس thread تقدر تدخل تاني.
- **Happens-before**: unlock → lock.

**3️⃣ `volatile`:**

```java
class Flag {
    private volatile boolean running = true;

    void stop() { running = false; }
    void run() {
        while (running) { /* ... */ }
    }
}
```

- Visibility: أي thread بتقرأ هتشوف آخر write.
- Ordering: بيمنع reordering حول الـ access.
- **مش atomic** لـ `count++`.

**4️⃣ Atomics:**

```java
class Counter {
    private final AtomicInteger count = new AtomicInteger();

    void increment() { count.incrementAndGet(); }
    int get() { return count.get(); }
}
```

- CAS loop: `compareAndSet(expected, new)`.
- `LongAdder` أفضل في الـ high contention (بيقسّم العدّاد).
- `AtomicReference` للـ objects.

**5️⃣ Happens-Before — القواعد:**

1. **Program order**: كل statement قبل اللي بعدها في نفس thread.
2. **Monitor**: unlock → lock.
3. **Volatile**: write → read.
4. **Thread start**: `start()` → `run()`.
5. **Thread join**: كل حاجة في thread → `join()` return.
6. **Transitivity**.

**6️⃣ Deadlock:**

```java
// Thread 1: lock A then B
// Thread 2: lock B then A
```

شروط Deadlock الأربعة (Coffman):
- Mutual exclusion.
- Hold and wait.
- No preemption.
- Circular wait.

**الحل:**
- ترتيب أقفال ثابت (lock ordering).
- `tryLock(timeout)`.
- تقليل الـ shared state.
- استخدام concurrent collections.

**7️⃣ `ReentrantLock`:**

```java
private final ReentrantLock lock = new ReentrantLock();

void update() {
    if (lock.tryLock()) {
        try { /* ... */ }
        finally { lock.unlock(); }
    }
}
```

مزايا على `synchronized`:
- `tryLock` مع timeout.
- Fairness.
- Interruptible lock acquisition.
- Multiple `Condition`s.

**8️⃣ `ConcurrentHashMap`:**

```java
map.computeIfAbsent(key, k -> load(k)); // atomic
```

- CAS + synchronized bins.
- No null keys/values.
- `size()` تقديري في الـ concurrency.
- متعملش عمليات طويلة جوه `computeIfAbsent`.

**9️⃣ Immutability & Confinement:**

- Immutable objects آمنة للمشاركة بدون synchronization.
- Thread confinement: كل thread عندها نسختها.
- Stack confinement: locals.

**🔟 `wait/notify`:**

```java
synchronized (lock) {
    while (!condition) lock.wait();
    // ...
    lock.notifyAll();
}
```

- لازم تكون ماسك الـ monitor.
- استخدم `while` مش `if` (spurious wakeups).
- `notifyAll` أأمن من `notify`.

**💻 Practical Example:**

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

public class ConcurrencyDemo {
    static class UnsafeCounter {
        int count;
        void inc() { count++; }
    }

    static class SafeCounter {
        private final AtomicInteger count = new AtomicInteger();
        void inc() { count.incrementAndGet(); }
        int get() { return count.get(); }
    }

    public static void main(String[] args) throws Exception {
        var unsafe = new UnsafeCounter();
        var safe = new SafeCounter();

        try (var pool = Executors.newFixedThreadPool(8)) {
            for (int i = 0; i < 10_000; i++) {
                pool.submit(() -> { unsafe.inc(); safe.inc(); });
            }
        }

        System.out.println("unsafe ≈ " + unsafe.count); // often < 10000
        System.out.println("safe = " + safe.get());     // 10000
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه الفرق بين visibility وatomicity؟
2. ليه `volatile` مش كافية لـ `count++`؟
3. إيه الـ happens-before؟
4. إيه الفرق بين `synchronized` و`ReentrantLock`؟
5. إيه شروط الـ deadlock؟
6. إزاي تمنع deadlock؟
7. إيه الفرق بين `AtomicInteger` و`LongAdder`؟
8. إيه الـ CAS؟ وإيه ABA problem؟
9. هل `ConcurrentHashMap` آمنة بالكامل؟
10. ليه immutable objects thread-safe؟

**⚠️ Common Interview Trap:**

- ❌ **"volatile makes operations atomic"** — لأ.
- ❌ **"synchronized on different objects still coordinates"** — لأ، لازم نفس الـ lock.
- ❌ **"String literals are good locks"** — لأ، interned ومشترك عالميًا.
- ❌ **"More synchronization = safer"** — ممكن deadlock وbottlenecks.
- ❌ **"ConcurrentHashMap allows null"** — لأ.
- ❌ **"Thread.stop is fine"** — unsafe، deprecated.

---

## 🔐 Part 11 — Serialization & Production Concerns

### Q29. اشرح Java Serialization: `serialVersionUID`، `transient`، Custom `writeObject`، والمخاطر الأمنية.

**🎯 Related Concepts:**
`Serializable` Marker · `serialVersionUID` · `transient` · `writeObject`/`readObject` · `Externalizable` · `readResolve` · Deserialization Gadgets · JEP 290 Filters · Records & Enums Serialization · JSON vs Java Serialization

**🗣️ Interview Answer:**

> Java serialization converts an object graph into bytes and back. A class opts in by implementing the marker interface `Serializable`. `serialVersionUID` is an explicit version identifier: if it doesn't match between serialization and deserialization, you get `InvalidClassException`. Without it, the JVM computes one from the class structure, so any change can break compatibility.
>
> `transient` excludes a field from serialization; after deserialization it gets the default value (`null`, `0`, `false`). `static` fields are never serialized.
>
> For custom logic, implement `private void writeObject(ObjectOutputStream)` and `private void readObject(ObjectInputStream)`. Deserialization **bypasses constructors**, so you must validate invariants in `readObject`. Java serialization is dangerous for untrusted input (gadget chains); prefer JSON/DTOs, and use `ObjectInputFilter` if you must.

**🔍 Deep Dive:**

**1️⃣ Basics:**

```java
import java.io.*;

class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;
    private transient String password; // not serialized
    private static int count;          // not serialized
}
```

- `serialVersionUID` لازم يكون `static final long`.
- لو مش موجود، الـ JVM بيحسبه من الأسماء/الأنواع/الـ modifiers، وأي تغيير بيكسره.
- `transient` بيستثني الحقل.
- `static` مش جزء من الـ object state.

**2️⃣ `transient` — القيم بعد الـ Deserialization:**

```java
class Session implements Serializable {
    private String user;
    private transient String token; // → null after deserialization
    private transient int retries;  // → 0
}
```

لو محتاج تعيد بناء القيمة، استخدم `readObject`:

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject();
    this.token = generateToken(); // restore transient
}
```

**3️⃣ Custom Serialization:**

```java
class Account implements Serializable {
    private static final long serialVersionUID = 1L;

    private String id;
    private transient String secret;

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeObject(encrypt(secret));
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        this.secret = decrypt((String) in.readObject());
    }
}
```

- `defaultWriteObject()` بيكتب الحقول غير الـ transient.
- تقدر تضيف حقول إضافية.
- الترتيب في `readObject` لازم يطابق `writeObject`.

**4️⃣ `Externalizable`:**

```java
class Point implements Externalizable {
    private int x, y;

    public Point() { } // public no-arg required

    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeInt(x);
        out.writeInt(y);
    }

    public void readExternal(ObjectInput in) throws IOException {
        x = in.readInt();
        y = in.readInt();
    }
}
```

- تحكم كامل في الـ format.
- لازم `public no-arg constructor`.
- الـ `readExternal` بتتنفذ بعد الـ no-arg construction.

**5️⃣ `readResolve` — Singleton Safety:**

```java
class Config implements Serializable {
    private static final Config INSTANCE = new Config();
    private Config() { }

    private Object readResolve() { return INSTANCE; }
}
```

- بتمنع إن deserialization تعمل instance جديدة.
- الـ Enum مش محتاجها (الـ JVM بتضمن).

**6️⃣ Inheritance Rules:**

- لو الـ superclass مش `Serializable`، لازم يكون عندها **accessible no-arg constructor**.
- حقول الـ superclass غير الـ serializable **مش بتتسلسل**، والـ constructor بتاعها بيتنفذ وقت الـ deserialization.
- لو الـ superclass `Serializable`، الـ constructor مش بيتنفذ.

**7️⃣ Records & Enums:**

- **Records**: لو الـ record بيعمل `implements Serializable`، بيتسلسل عن طريق الـ components، والـ **canonical constructor** بيتنفذ (فالـ validation بتشتغل). `serialVersionUID` للـ record هو `0L` إلا لو معرّف صراحة.
- **Enums**: بتتسلسل بالاسم، والـ JVM بترجع نفس الـ constant (singleton guarantee).

**8️⃣ Security Risks:**

- **Gadget chains**: deserialization of untrusted bytes can execute arbitrary code via classes on the classpath.
- **Bypass constructors**: invariants ممكن تتكسر.
- **Sensitive data**: passwords/tokens in the stream.
- **DoS**: huge object graphs.

**الحلول:**

- متستخدمش Java serialization للـ external/untrusted data.
- استخدم JSON/DTO مع validation.
- لو لازم، استخدم `ObjectInputFilter` (JEP 290):

```java
var filter = ObjectInputFilter.Config.createFilter("com.example.*;!*");
ObjectInputFilter.Config.setSerialFilter(filter);
```

- `transient` للـ secrets، وشفّر لو لازم.

**9️⃣ `serialVersionUID` — Best Practice:**

```java
private static final long serialVersionUID = 1L;
```

- صريح دايمًا.
- غيّره عند تغيير **incompatible**.
- متغيّرهوش لمجرد إضافة حقل (متوافق).

**🔟 Java Serialization vs JSON:**

| | Java Serialization | JSON (Jackson/Gson) |
|---|---|---|
| **Readable** | ❌ binary | ✅ text |
| **Cross-language** | ❌ | ✅ |
| **Schema evolution** | ⚠️ fragile | ✅ flexible |
| **Security** | ❌ risky | ✅ safer with validation |
| **Performance** | ✅ often fast | ⚠️ depends |
| **Use case** | Legacy/internal | APIs, DTOs |

**💻 Practical Example:**

```java
import java.io.*;

public class SerializationDemo {
    static class User implements Serializable {
        private static final long serialVersionUID = 1L;

        private final String name;
        private transient String password; // not serialized

        User(String name, String password) {
            this.name = name;
            this.password = password;
        }

        String name() { return name; }
        String password() { return password; }
    }

    public static void main(String[] args) throws Exception {
        var user = new User("Ali", "secret");

        var bytes = new ByteArrayOutputStream();
        try (var out = new ObjectOutputStream(bytes)) {
            out.writeObject(user);
        }

        try (var in = new ObjectInputStream(new ByteArrayInputStream(bytes.toByteArray()))) {
            var restored = (User) in.readObject();
            System.out.println(restored.name());     // Ali
            System.out.println(restored.password()); // null (transient)
        }
    }
}
```

**🔁 Follow-up Interview Questions:**

1. إيه فايدة `serialVersionUID`؟
2. إيه اللي بيحصل لو مش موجود؟
3. إيه الفرق بين `transient` و`static`؟
4. إزاي تعمل custom serialization؟
5. إيه الفرق بين `Serializable` و`Externalizable`؟
6. ليه deserialization خطيرة؟
7. إزاي `readResolve` بتحمي الـ Singleton؟
8. إزاي الـ Records بتتسلسل؟
9. إيه اللي يحصل لو الـ superclass مش Serializable؟
10. إمتى تختار JSON بدل Java serialization؟

**⚠️ Common Interview Trap:**

- ❌ **"serialVersionUID is optional, so skip it"** — هيّ optional تقنيًا، لكن إلزامية عمليًا.
- ❌ **"transient fields are restored automatically"** — لأ، default values.
- ❌ **"Deserialization calls the constructor"** — لأ (إلا records/Externalizable).
- ❌ **"Serializable is safe"** — لأ، خصوصًا مع untrusted input.
- ❌ **"Enums need readResolve"** — لأ.
- ❌ **"Records serialize like normal classes"** — لأ، components + canonical ctor.

---

## 🧷 Part 12 — Modern Java: Sealed Types & Pattern Matching

### Q30. إزاي تصمم Closed Hierarchy بـ `sealed` وتستخدم Pattern Matching في `switch` exhaustive؟

**🎯 Related Concepts:**
Sealed Classes/Interfaces (Java 17) · `permits` · `final`/`sealed`/`non-sealed` · Exhaustive Switch (Java 21) · Type Patterns · Record Patterns · Guards (`when`) · Dominance · `case null` · Algebraic Data Types

**🗣️ Interview Answer:**

> A `sealed` type restricts which classes/interfaces may extend or implement it. The permitted subtypes are listed in `permits` (or inferred when they're in the same source file), and every direct subtype must be `final`, `sealed`, or `non-sealed`.
>
> The payoff is a **closed hierarchy** that the compiler knows completely. That enables **exhaustive `switch`**: you can handle every permitted subtype without a `default`, and adding a new subtype becomes a **compile error** everywhere that needs updating — which is exactly what you want for domain results, ASTs, and state machines.
>
> Pattern matching adds type patterns (`case String s`), record deconstruction (`case Point(int x, int y)`), guards (`when`), and explicit `null` handling (`case null`). The compiler also enforces **dominance**: an earlier broader pattern can make a later one unreachable.

**🔍 Deep Dive:**

**1️⃣ Sealed Syntax:**

```java
sealed interface Shape permits Circle, Square, Rectangle { }

record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }
final class Rectangle implements Shape {
    private final double w, h;
    Rectangle(double w, double h) { this.w = w; this.h = h; }
    double w() { return w; }
    double h() { return h; }
}
```

- `permits` بيحدد الـ direct subtypes.
- كل direct subtype لازم يكون:
  - `final` (مينفعش يتمدد)، أو
  - `sealed` (يكمل السلسلة)، أو
  - `non-sealed` (يفتح الباب لأي subclass).
- الـ subtypes لازم يكونوا في **نفس الـ module**، أو **نفس الـ package** لو مفيش module.
- لو كل الـ subtypes في نفس الملف، ممكن تحذف `permits` والـ compiler هيستنتجها.

**2️⃣ `non-sealed` — Opt-out صريح:**

```java
sealed interface PaymentMethod permits Card, Cash { }

final class Cash implements PaymentMethod { }

non-sealed class Card implements PaymentMethod { } // أي حد يقدر يعمل extends Card

class Visa extends Card { }
```

`non-sealed` معناها: "أنا جزء من الـ hierarchy المغلقة، لكني بسمح بالوراثة مني". ده مفيد لو عايز تسمح بامتدادات في طبقة معينة.

**3️⃣ Pattern Matching for `instanceof` (Java 16):**

```java
static String describe(Object o) {
    if (o instanceof String s && !s.isBlank()) {
        return "non-blank string: " + s;
    }
    return "other";
}
```

- مفيش cast يدوي.
- الـ pattern variable `s` في الـ scope بس حيث الـ compiler متأكد إن الـ match حصل (flow scoping).

```java
if (!(o instanceof String s)) return;
System.out.println(s.length()); // ✅ s in scope here
```

**4️⃣ Type Patterns في `switch` (Java 21):**

```java
static String type(Object o) {
    return switch (o) {
        case Integer i -> "int: " + i;
        case Long l    -> "long: " + l;
        case String s  -> "string: " + s;
        case null      -> "null";
        default        -> "other";
    };
}
```

- `case null` مطلوب لو الـ selector ممكن يكون `null`، وإلا `switch` هترمي `NullPointerException`.
- `default` مطلوب لو الـ selector مش sealed.

**5️⃣ Exhaustive Switch مع Sealed:**

```java
static double area(Shape shape) {
    return switch (shape) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Square s    -> s.side() * s.side();
        case Rectangle r -> r.w() * r.h();
    }; // ✅ no default
}
```

- الـ compiler عارف كل الـ permitted subtypes.
- إضافة `Triangle implements Shape` → **compile error** هنا، وده بيجبرك تحدّث كل الأماكن.
- ده أفضل من `default` صامت بيخفي الحالات الجديدة.

**6️⃣ Record Patterns (Java 21):**

```java
record Point(int x, int y) { }
record Line(Point start, Point end) { }

static String describe(Line line) {
    return switch (line) {
        case Line(Point(int x1, int y1), Point(int x2, int y2)) ->
            "from (" + x1 + "," + y1 + ") to (" + x2 + "," + y2 + ")";
    };
}
```

- الـ deconstruction بتفكك الـ record components مباشرة.
- ممكن تت nesting (record جوه record).
- بتشتغل مع `instanceof` كمان:

```java
if (obj instanceof Point(int x, int y)) {
    System.out.println(x + "," + y);
}
```

**7️⃣ Guards — `when`:**

```java
static String size(Shape shape) {
    return switch (shape) {
        case Circle c when c.radius() > 10 -> "big circle";
        case Circle c                      -> "small circle";
        case Square s                      -> "square";
        case Rectangle r                   -> "rectangle";
    };
}
```

- الـ guard بيتقيّم **بعد** نجاح الـ pattern.
- الترتيب مهم: الحالة الأخص قبل الأعم.
- لو الـ guard فشل، الـ switch بيكمل للحالة اللي بعدها.

**8️⃣ Dominance — Compiler بيمنع الـ Unreachable:**

```java
return switch (o) {
    case Object obj -> "any";   // ❌ بيسيطر على كل حاجة
    case String s   -> "string"; // unreachable → compile error
    default         -> "other";
};
```

- أي pattern أوسع قبل أضيق بيخلي الأضيق **dominated**.
- الـ compiler بيرفضه، مش بيسيبه للـ runtime.

**9️⃣ `case null`:**

```java
return switch (o) {
    case null       -> "null";
    case String s   -> "string";
    default         -> "other";
};
```

- من غير `case null`، `switch(null)` بترمي NPE.
- ممكن تدمج: `case null, default -> "null or other";` (Java 21).

**🔟 Sealed + Records = Algebraic Data Types:**

```java
sealed interface PaymentResult permits Success, Failure, Pending { }

record Success(String transactionId) implements PaymentResult { }
record Failure(String code, String message) implements PaymentResult { }
record Pending(String reference) implements PaymentResult { }

static String render(PaymentResult result) {
    return switch (result) {
        case Success s -> "paid: " + s.transactionId();
        case Failure f -> "failed: " + f.code() + " - " + f.message();
        case Pending p -> "pending: " + p.reference();
    };
}
```

ده closest thing في Java لـ **sum types** في functional languages.

**1️⃣1️⃣ Sealed vs Enum:**

| | Sealed | Enum |
|---|---|---|
| **Closed set of** | Types | Instances |
| **State per case** | أي حقول تحبها | حقول مشتركة + constant-specific bodies |
| **Pattern matching** | ✅ exhaustive | ✅ exhaustive |
| **Extensibility** | `non-sealed` ممكن | ❌ |
| **Use** | Domain results, AST | Fixed constants, status |

**1️⃣2️⃣ Sealed ميزة Compile-Time مش Security:**

- مش بتمنع reflection أو serialization hacks.
- مش بديل عن validation.
- القيمة الأساسية: **modeling** و**exhaustiveness**.

**💻 Practical Example:**

```java
sealed interface Notification permits Email, Sms, Push { }

record Email(String address, String subject) implements Notification { }
record Sms(String phone, String text) implements Notification { }
record Push(String deviceId, String title) implements Notification { }

final class NotificationSender {
    String send(Notification notification) {
        return switch (notification) {
            case Email e when e.address().isBlank() ->
                throw new IllegalArgumentException("empty email");
            case Email e -> "email to " + e.address() + ": " + e.subject();
            case Sms s   -> "sms to " + s.phone() + ": " + s.text();
            case Push p  -> "push to " + p.deviceId() + ": " + p.title();
        };
    }
}
```

لو ضفت `WhatsApp`، الـ compiler هيفشل الـ switch ويجبرك تضيف الحالة — ده **safe evolution**.

**🔁 Follow-up Interview Questions:**

1. إيه فايدة `sealed` عن `enum`؟
2. إمتى تحذف `permits`؟
3. إيه الفرق بين `final` و`sealed` و`non-sealed`؟
4. ليه الـ switch على sealed مش محتاج `default`؟
5. إيه الـ dominance؟
6. إيه اللي يحصل لو الـ selector `null` ومفيش `case null`؟
7. إزاي record patterns بتشتغل؟
8. إيه فايدة الـ guards (`when`)؟
9. هل `sealed` بتمنع الوراثة نهائيًا؟
10. إزاي sealed + records بيمثلوا algebraic data types؟

**⚠️ Common Interview Trap:**

- ❌ **"Sealed prevents all extension"** — `non-sealed` بتفتح الباب.
- ❌ **"Switch on sealed always needs default"** — لأ، لو exhaustive.
- ❌ **"Sealed is a security boundary"** — لأ، هي modeling tool.
- ❌ **"Pattern variable is in scope in the whole method"** — flow scoping.
- ❌ **"`case Object o` is a safe fallback"** — ممكن يعمل dominance ويكسر الحالات الأخص.
- ❌ **"`switch(null)` hits default"** — لأ، NPE إلا مع `case null`.

---

## 🎯 Part 13 — Rapid-Fire Traps & Final Checklist

### Q31. إيه أشهر Java Core Traps اللي بتوقّع المرشحين في الإنترفيو؟ وإزاي تتعامل معاها؟

**🎯 Related Concepts:**
Pass-by-Value · Integer Overflow · Floating Point · `BigDecimal` · Autoboxing Overloads · `switch` Fall-Through · `String` Pool · `finally` Return · Mutable Keys · Stream Reuse · `volatile` Atomicity · `ThreadLocal` Leaks

**🗣️ Interview Answer:**

> Most Java traps come from confusing **references with values**, **compile-time with runtime**, and **syntax with semantics**. Java is always **pass-by-value** (the value of a reference is copied for objects). `Integer` overflow wraps silently. `0.1 + 0.2 != 0.3` because of binary floating point. `finally` can swallow exceptions. `volatile` doesn't make `count++` atomic. Streams are single-use. And `List.of` rejects nulls while `Arrays.asList` is only fixed-size.
>
> The defense is simple: know the **contract**, test the edge case, and prefer immutable value types and explicit APIs.

**🔍 Deep Dive:**

**The Trap Table:**

| # | Trap | Behavior | Why / Fix |
|---|---|---|---|
| 1 | **Pass-by-value** | Java always copies the argument | Primitives copy value; objects copy the reference. Reassigning the parameter doesn't affect the caller; mutating the object does. |
| 2 | **`i = i++`** | Keeps the old value | Post-increment returns the old value, then assigns it back. |
| 3 | **Integer overflow** | Wraps silently | `Integer.MAX_VALUE + 1 == Integer.MIN_VALUE`; use `Math.addExact` or `long`. |
| 4 | **`Math.abs(Integer.MIN_VALUE)`** | Negative | `-Integer.MIN_VALUE` overflows; use `Math.absExact` (Java 15+) or `long`. |
| 5 | **`0.1 + 0.2`** | `0.30000000000000004` | Binary floating point; use `BigDecimal` for money. |
| 6 | **`BigDecimal.equals` vs `compareTo`** | `1.0` ≠ `1.00` with `equals` | `equals` compares scale; `compareTo` compares value. |
| 7 | **`char + int`** | `int`, not `char` | Numeric promotion; cast back. |
| 8 | **`switch` fall-through** | Executes subsequent cases | Use `break` or arrow syntax (`case X ->`). |
| 9 | **`switch(null)`** | `NullPointerException` | Guard before switch. |
| 10 | **`list.remove(1)`** | Removes index | Overload resolution picks `remove(int)`, not `remove(Object)`. |
| 11 | **`Arrays.asList(int[])`** | Size 1 | Primitive array is a single varargs element; use `Arrays.stream(...).boxed()`. |
| 12 | **`List.of(null)`** | `NullPointerException` | Null-hostile; use `Arrays.asList` or filter. |
| 13 | **`println(null)`** | Ambiguous → compile error | `println(char[])` vs `println(String)`; cast to `(String) null`. |
| 14 | **`Integer` cache** | `127 == 127` true, `128 == 128` false | Cache range -128..127; use `equals`. |
| 15 | **`new String("x") == "x"`** | `false` | New heap object; use `equals`/`intern`. |
| 16 | **`finally` return** | Overrides return/exception | Never return/throw in `finally`. |
| 17 | **`System.exit` + `finally`** | `finally` skipped | JVM terminates; use explicit cleanup before exit. |
| 18 | **`equals` without `hashCode`** | Broken `HashMap`/`HashSet` | Override both. |
| 19 | **Mutable `HashMap` key** | Lost entry | Hash changes after insertion; use immutable keys. |
| 20 | **`TreeSet` comparator returns 0** | Element treated as duplicate | Comparator must be consistent with equals for intended semantics. |
| 21 | **Stream reuse** | `IllegalStateException` | Streams are single-use; create a new one. |
| 22 | **`Optional.orElse`** | Fallback evaluated eagerly | Use `orElseGet` for expensive fallback. |
| 23 | **`volatile count++`** | Lost updates | Volatile isn't atomic; use `AtomicInteger`. |
| 24 | **`ThreadLocal` in pool** | Memory leak | Call `remove()` in `finally`. |
| 25 | **`var` is not dynamic** | Compile-time inferred type | It's still statically typed. |
| 26 | **`record` with `List`** | Shallow immutability | Defensive copy in compact constructor. |
| 27 | **`sealed` subclass** | Must be `final`/`sealed`/`non-sealed` | Compiler enforces. |
| 28 | **`String` `+` in loop** | O(n²) allocations | Use `StringBuilder`. |
| 29 | **`==` on wrappers** | Reference comparison (outside cache) | Use `equals`. |
| 30 | **`try-with-resources` close order** | Reverse order | Design resources accordingly. |

**1️⃣ Pass-by-Value — المثال الكلاسيكي:**

```java
static void modify(int x) { x = 99; }
static void modify(User u) { u.setName("new"); }
static void reassign(User u) { u = new User("new"); }

int x = 1;
modify(x); // x still 1

var u = new User("old");
modify(u);   // u.name == "new"
reassign(u); // u still points to the old object
```

**2️⃣ `i = i++`:**

```java
int i = 0;
i = i++; // i stays 0
```

**3️⃣ Floating Point & Money:**

```java
double d = 0.1 + 0.2; // 0.30000000000000004
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
a.add(b); // 0.3
```

- `new BigDecimal(double)` بيعكس التمثيل الثنائي؛ استخدم `BigDecimal.valueOf(double)` أو الـ String constructor.

**4️⃣ `finally`:**

```java
static int f() {
    try { return 1; }
    finally { return 2; } // returns 2
}
```

**5️⃣ `volatile`:**

```java
volatile int count;
count++; // ❌ not atomic
```

**💻 Practical Example:**

**Trap Detector:**

```java
import java.math.BigDecimal;
import java.util.*;

public class TrapDemo {
    public static void main(String[] args) {
        System.out.println(0.1 + 0.2);                 // 0.30000000000000004
        System.out.println(new BigDecimal("0.1")
                .add(new BigDecimal("0.2")));          // 0.3

        int i = 0;
        i = i++;
        System.out.println(i);                         // 0

        Integer a = 127, b = 127;
        Integer c = 128, d = 128;
        System.out.println(a == b);                    // true
        System.out.println(c == d);                    // false

        var list = new ArrayList<>(List.of(10, 20, 30));
        list.remove(1);                                // index
        System.out.println(list);                      // [10, 30]

        int[] nums = {1, 2, 3};
        System.out.println(Arrays.asList(nums).size()); // 1
    }
}
```

**🔁 Follow-up Interview Questions:**

1. هل Java pass-by-reference؟ (لأ)
2. إزاي تتجنب integer overflow؟
3. ليه `0.1 + 0.2 != 0.3`؟
4. إمتى تستخدم `BigDecimal`؟
5. إيه الفرق بين `i++` و`++i`؟
6. إيه اللي يحصل لو comparator بيرجع 0؟
7. ليه `volatile` مش كافية للـ counter؟
8. إزاي تتعامل مع `List.of(null)`؟
9. إيه الفرق بين `var` و`dynamic`؟
10. ليه `finally` return خطر؟

**⚠️ Common Interview Trap:**

الطريقة الوحيدة إنك متقعش في الـ traps دي هي إنك **تختبر الـ edge cases** وتفهم الـ semantics مش الـ syntax. لو مش متأكد، اكتب الـ snippet وشغّله — الـ compiler والـ JVM هما الحكم.

---

## ✅ Interview Preparation Checklist

### 🧱 1. OOP & Language Fundamentals

- [ ] أشرح الـ four pillars بمثال Backend واحد، مش تعريفات.
- [ ] أفرّق بين **Abstraction** و**Encapsulation** بدقة.
- [ ] أشرح **LSP** وأدّي مثال violation وحل.
- [ ] أختار بين **Interface** و**Abstract Class** بمبرر.
- [ ] أشرح **default methods** والـ diamond resolution (`X.super.m()`).
- [ ] أفرّق بين **Overloading / Overriding / Hiding** (methods + fields + static).
- [ ] أعرف قواعد **covariant return** وvisibility وchecked exceptions.
- [ ] أشرح **constructor chaining** و**init order** بالكامل.
- [ ] أعرف ليه استدعاء overridable method في constructor خطر.
- [ ] أشرح `this()` vs `super()` ومتى كل واحد.
- [ ] أفرّق بين الـ **4 access modifiers** (خصوصًا `protected`).
- [ ] أشرح الفرق بين **JDK / JRE / JVM**.

### 🧬 2. Objects, Immutability & Data Classes

- [ ] أشرح عقد **`equals`/`hashCode`** بالكامل.
- [ ] أعرف اللي يحصل في `HashMap` لو override واحد بس.
- [ ] أشرح **mutable keys** والمشكلة.
- [ ] أفرّق بين **`getClass()`** و**`instanceof`** في `equals`.
- [ ] أبني **immutable class** صح (defensive copy in/out).
- [ ] أفرّق بين **shallow/deep copy**.
- [ ] أفرّق بين **unmodifiable view** و**immutable copy**.
- [ ] أشرح **Records** وقيودها وshallow immutability.
- [ ] أستخدم **compact constructor** للvalidation.
- [ ] أفرّق بين Record وJPA Entity وDTO.
- [ ] أشرح **Enums** كclasses، و`ordinal` fragility، و`EnumMap`/`EnumSet`.
- [ ] أشرح **Enum singleton** ليه الأفضل.
- [ ] أفرّق بين **static nested / inner / local / anonymous**.
- [ ] أعرف خطر الـ **implicit outer reference**.
- [ ] أفرّق بين **lambda** و**anonymous class** من ناحية `this`.

### 🧩 3. Design & DI

- [ ] أفرّق بين **Inheritance / Composition / Aggregation**.
- [ ] أشرح **fragile base class**.
- [ ] أشرح **constructor injection** ومزاياه.
- [ ] أعرف عيوب **field injection**.
- [ ] أشرح **circular dependency** وحلولها.
- [ ] أشرح **SOLID** بمثال واحد متكامل.
- [ ] أفرّق بين **Strategy/Factory/Builder/Singleton/Observer/Decorator/Proxy**.
- [ ] أعرف ليه `volatile` ضروري في **DCL singleton**.
- [ ] أشرح **static holder** و**enum singleton**.

### 🔤 4. Strings & Wrappers

- [ ] أشرح **String immutability** وأسبابها.
- [ ] أشرح **string pool** و`intern()`.
- [ ] أعرف كام object في `new String("x")`.
- [ ] أفرّق بين `==` و`equals` للـ String.
- [ ] أشرح **StringBuilder vs StringBuffer**.
- [ ] أعرف ليه `+` في loop مشكلة.
- [ ] أشرح **autoboxing/unboxing** و**Integer cache**.
- [ ] أعرف NPE من unboxing `null`.
- [ ] أشرح **overload resolution** (widening → boxing → varargs).
- [ ] أفرّق بين `parseInt` و`valueOf`.
- [ ] أعرف فخ `list.remove(1)`.

### 🧮 5. Generics

- [ ] أشرح **type erasure** ونتائجه.
- [ ] أعرف ليه `new T[]` ممنوع.
- [ ] أشرح **bounded types**.
- [ ] أشرح **PECS** بمثال.
- [ ] أفرّق بين `List<Object>` و`List<?>`.
- [ ] أشرح **bridge methods**.
- [ ] أعرف **heap pollution** و`@SafeVarargs`.
- [ ] أعرف ليه raw types خطر.

### 📦 6. Collections

- [ ] أختار الـ Collection المناسبة حسب الـ contract.
- [ ] أفرّق بين **ArrayList** و**LinkedList** بالـ complexity.
- [ ] أشرح **fail-fast** و`ConcurrentModificationException`.
- [ ] أفرّق بين **`Arrays.asList` / `List.of` / `List.copyOf` / `unmodifiableList`**.
- [ ] أعرف فخ `Arrays.asList(int[])`.
- [ ] أشرح **HashMap internals**: hash spread، buckets، collisions، treeify، resize.
- [ ] أعرف **load factor** و**capacity** و**threshold**.
- [ ] أشرح **null key** و**thread safety**.
- [ ] أفرّق بين **HashMap / LinkedHashMap / TreeMap**.
- [ ] أفرّق بين **HashSet / LinkedHashSet / TreeSet**.
- [ ] أشرح **Comparable vs Comparator** وconsistency مع equals.
- [ ] أبني **LRU cache** بـ `LinkedHashMap`.
- [ ] أشرح **Queue/Deque/PriorityQueue/BlockingQueue**.
- [ ] أعرف `add` vs `offer` و`remove` vs `poll`.

### 🚨 7. Exceptions

- [ ] أشرح **Throwable hierarchy**.
- [ ] أفرّق بين **checked/unchecked** ومتى كل واحد.
- [ ] أبني **custom exception** مع context وcause.
- [ ] أشرح **try-with-resources** وترتيب الإغلاق.
- [ ] أعرف **suppressed exceptions**.
- [ ] أشرح **multi-catch** وقيوده.
- [ ] أعرف أفضل ممارسات الـ logging/rethrow.
- [ ] أشرح **`finally` semantics**.
- [ ] أعرف خطر `return`/`throw` في `finally`.
- [ ] أعرف متى `finally` مش بتتنفذ.
- [ ] أشرح ليه `finalize` deprecated وبديله `Cleaner`.

### ⚙️ 8. JVM

- [ ] أشرح **Stack vs Heap vs Metaspace**.
- [ ] أشرح **StackOverflowError vs OutOfMemoryError**.
- [ ] أشرح **GC roots**.
- [ ] أفرّق بين **strong/soft/weak/phantom** references.
- [ ] أشرح **memory leaks** في Java وأسبابها.
- [ ] أشرح **minor/major/full GC**.
- [ ] أفرّق بين **G1/ZGC/Shenandoah**.
- [ ] أعرف إزاي أقرأ **GC logs** وأظبط الـ heap.
- [ ] أشرح **class loading phases** و**parent delegation**.
- [ ] أفرّق بين **ClassNotFoundException** و**NoClassDefFoundError**.
- [ ] أشرح **static initialization** ومتى بتتنفذ.
- [ ] أشرح **JIT/C1/C2/tiered**.
- [ ] أعرف **compile-time constants** والـ inlining.

### 🧪 9. Java 8+ & Modern Java

- [ ] أعرّف **functional interface** وSAM.
- [ ] أستخدم **Function/Supplier/Consumer/Predicate** وprimitive variants.
- [ ] أشرح **lambda capture** وeffectively final.
- [ ] أفرّق بين **lambda** وanonymous class من ناحية `this`.
- [ ] أستخدم **method references** بأنواعها.
- [ ] أشرح **stream pipeline** وlaziness.
- [ ] أفرّق بين **intermediate/terminal/short-circuit**.
- [ ] أشرح **map vs flatMap**.
- [ ] أستخدم **Collectors** (groupingBy, partitioningBy, toMap, teeing).
- [ ] أعرف فخ **duplicate keys** في `toMap`.
- [ ] أشرح **parallel streams** ومتى تضر.
- [ ] أفرّق بين `Collectors.toList()` و`Stream.toList()`.
- [ ] أستخدم **Optional** صح.
- [ ] أفرّق بين **orElse** و**orElseGet**.
- [ ] أعرف **Optional anti-patterns**.
- [ ] أشرح **sealed / permits / non-sealed**.
- [ ] أكتب **switch exhaustive** على sealed hierarchy.
- [ ] أستخدم **type patterns / record patterns / guards**.
- [ ] أعرف **dominance** و**case null**.

### 🧵 10. Concurrency

- [ ] أشرح **thread lifecycle**.
- [ ] أفرّق بين **Runnable/Callable**.
- [ ] أستخدم **ExecutorService** وأقفله صح.
- [ ] أشرح **pool sizing** (CPU vs I/O).
- [ ] أفرّق بين **Future** و**CompletableFuture**.
- [ ] أشرح **thenApply vs thenCompose vs thenCombine**.
- [ ] أعرف خطر **common pool**.
- [ ] أشرح **virtual threads** ومتى تستخدمها.
- [ ] أشرح **race condition** و**`count++`**.
- [ ] أشرح **synchronized** و**volatile** والفرق.
- [ ] أشرح **happens-before**.
- [ ] أشرح **CAS** و**AtomicInteger/LongAdder**.
- [ ] أشرح **deadlock** وشروطه وحلوله.
- [ ] أفرّق بين **ReentrantLock** وsynchronized.
- [ ] أعرف **ThreadLocal leak**.

### 🔐 11. Serialization & Production

- [ ] أشرح **Serializable** و`serialVersionUID`.
- [ ] أشرح **transient** وstatic.
- [ ] أستخدم **custom writeObject/readObject**.
- [ ] أفرّق بين **Serializable** و**Externalizable**.
- [ ] أشرح **readResolve**.
- [ ] أعرف **security risks** و**ObjectInputFilter**.
- [ ] أشرح **records/enums serialization**.
- [ ] أختار **JSON vs Java serialization**.

### 🎤 12. Interview Communication

- [ ] أجاوب بـ **structure**: تعريف → مثال → trade-off → production note.
- [ ] أستخدم **keywords** صح (happens-before, type erasure, load factor, LSP...).
- [ ] أقول **"it depends"** مع مبرر، مش إجابة مطلقة.
- [ ] أذكر **trade-offs** مش حل واحد.
- [ ] أربط الإجابة بـ **Backend scenario** (API, DB, concurrency, memory).
- [ ] أسأل **clarifying questions** لو السؤال عام.
- [ ] أعترف لو مش متأكد، وأشرح إزاي أتحقق.
- [ ] أتجنب **الجدال** مع الـ interviewer.
- [ ] أختبر الكود ذهنيًا قبل ما أقول إنه يشتغل.
- [ ] أذكر **الـ version** (Java 8 vs 17 vs 21) لما يفرق.

---

### 🚩 Red Flags — لو لقيت نفسك بتعملها، راجع

| Red Flag | الصح |
|---|---|
| بتحفظ تعريفات من غير أمثلة | اربط كل مفهوم بسيناريو Backend |
| بتقول "Inheritance for reuse" | اشرح IS-A وLSP وComposition |
| بتخلط بين Overloading وOverriding | حدد binding time |
| بتقول "HashMap O(1) دايمًا" | اذكر average + worst case + treeify |
| بتقول "volatile makes it thread-safe" | اشرح visibility vs atomicity |
| بتقول "finally always runs" | اذكر الاستثناءات |
| بتقول "Optional solves NPE" | اشرح حدوده |
| بتقول "Parallel is faster" | اشرح الشروط والمخاطر |
| بتقول "Java is pass-by-reference for objects" | الصح pass-by-value للـ reference |
| بتقول "Records are fully immutable" | shallow immutability |

---

### 🗓️ خطة مراجعة 7 أيام

| Day | Focus |
|---|---|
| **1** | OOP + Interface/Abstract + Overloading/Overriding/Hiding + Constructors |
| **2** | equals/hashCode + Immutability + Records + Sealed + Enums + Nested Classes |
| **3** | Composition/DI + SOLID + Patterns + Strings + Wrappers |
| **4** | Generics + Collections + HashMap + Comparable/Comparator |
| **5** | Exceptions + JVM Memory/GC + Class Loading/JIT |
| **6** | Lambdas + Streams + Optional + Concurrency |
| **7** | Serialization + Rapid-Fire Traps + Mock Interviews + Checklist |

---

### 🎯 Final Words

> **الفرق بين مرشح "بيحفظ" ومرشح "بيفهم" هو الـ WHY.**
>
> أي حد يقدر يقول إن `HashMap` بتستخدم `hashCode`. المرشح القوي بيشرح:
> - ليه الـ capacity power of two،
> - إيه اللي يحصل عند الـ collision،
> - إمتى الـ bucket بتتحول لشجرة،
> - وليه key mutable بيكسر الـ map.
>
> ذاكر كل سؤال كأنك **بتصمم نظام**، مش كأنك **بتعدّي اختبار**. ولو نسيت تفصيلة، ابدأ من الـ contract والـ semantics، والإجابة هتتبني لوحدها.

---

*آخر تحديث: نسخة شاملة مبنية على محتوى مجلد `JAVA CORE☕` — الأسئلة مترابطة، والمصطلحات التقنية محفوظة بالإنجليزية، والشرح بالعامية المصرية.*

