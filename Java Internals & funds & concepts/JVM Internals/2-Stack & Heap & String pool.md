# Stack vs Heap في Java — من منظور Spring Boot Developer

---

## 1. Analogy

تخيل إنك في مطبخ:

- **Stack** = سطح الطاولة اللي بتشتغل عليه. تحط عليه الحاجات اللي محتاجها _دلوقتي_ في الخطوة الحالية. لما تخلص الخطوة، بتمسح الطاولة تلقائيًا.
- **Heap** = الثلاجة الكبيرة المشتركة. أي حاجة محتاج تحتفظ بيها أطول، تحطها هناك. بس حد لازم يتفقد الثلاجة من وقت لتاني ويرمي اللي مش محتاجه — ده الـ GC.

---

## 2. التعريف التقني

### Stack

- **Thread-specific**: كل thread عنده Stack خاص بيه.
- بيتخزن فيه:
    - **Primitive values** كاملة: `int x = 5` → الـ `5` نفسه على الـ Stack.
    - **Object references** (المؤشر): مش الـ Object نفسه، بس العنوان اللي بيشير عليه.
    - **Method frames**: كل method بتتنادي بتحجز "frame" على الـ Stack فيه الـ local variables + return address.
- بيتمسح **تلقائيًا** لما الـ method تخلص (LIFO).

### Heap

- **Shared** بين كل الـ threads.
- بيتخزن فيه:
    - كل الـ **Objects** (`new User()`, `new ArrayList()`, …).
    - الـ **instance variables** جوه الـ Objects.
- بيتحكم فيه الـ **Garbage Collector**.

---

## 3. مثال كود عملي

```java
public void processUser() {
    int age = 25;                  // (A)
    String name = "Ahmed";         // (B)
    User user = new User("Ahmed"); // (C)
}
```

|المتغير|فين الـ Reference؟|فين الـ Value/Object؟|
|---|---|---|
|`age = 25` (A)|—|الـ `25` نفسه على الـ **Stack**|
|`name = "Ahmed"` (B)|Reference على الـ **Stack**|Object في **Heap** (String Pool تحديدًا)|
|`user = new User(...)` (C)|Reference على الـ **Stack**|الـ User object على الـ **Heap**|

> لما `processUser()` تخلص → الـ Stack frame اتمسح، الـ references اختفت، الـ User object على الـ Heap بقى **eligible for GC**.

---

## 4. العلاقة بالـ Garbage Collection

**ليه الـ Stack مش محتاج GC؟** لأن عمر كل variable على الـ Stack مربوط بعمر الـ method. لما الـ method تخلص، الـ frame بيتشال تلقائيًا. مفيش memory leak ممكن هنا.

**ليه الـ Heap محتاج GC؟** الـ Objects بتعيش على الـ Heap بعد ما الـ method تخلص. مفيش آلية تلقائية تمسحها — الـ GC هو اللي بيتفقد: "في حد لسه شايل reference على الـ Object ده؟"

**متى Object يبقى eligible for GC؟** لما **مفيش أي reachable reference** واصلة له. يعني:

```java
User user = new User("Ahmed"); // Object على الـ Heap، reference على الـ Stack
user = null;                    // دلوقتي الـ Object مفيش reference بيشاور عليه → eligible for GC
```

أو ببساطة لما الـ method تخلص والـ local reference يتمسح من الـ Stack.

---

## 5. String Pool

**الـ String Pool** موجودة في الـ **Heap** (من Java 7+، قبلها كانت في PermGen).

```java
String s1 = "hello";              // (A) بتدور في الـ Pool، لو مش موجودة تخلقها
String s2 = "hello";              // (B) نفس الـ Object في الـ Pool — s1 == s2 → true

String s3 = new String("hello");  // (C) Object جديد على الـ Heap خارج الـ Pool
                                   //     s1 == s3 → false | s1.equals(s3) → true
```

**التفصيلة المهمة للـ Interview:**

- `==` بيقارن الـ references (العناوين).
- `s1 == s2` → `true` لأنهم بيشاوروا على نفس الـ Pool object.
- `s1 == s3` → `false` لأن `new String()` بيجبر Java تخلق object جديد خارج الـ Pool.
- `s3.intern()` بترجع الـ Pool reference.

---

## 6. في سياق Spring Boot

### Singleton Beans → Heap طول عمر الـ Application

```java
@Service
public class UserService { // Singleton bean
    private final UserRepository repo; // instance variable → على الـ Heap
}
```

الـ ApplicationContext شايل reference على كل الـ Singleton beans → مش هيتعمل لهم GC طول ما الـ app شغال. **ده بالظبط سبب الـ memory overhead لو عندك beans كتير أو beans بيخزنوا state غلط.**

---

### طول الـ Request Lifecycle — مين على الـ Stack؟

```java
@RestController
public class UserController {

    @GetMapping("/user/{id}")
    public UserDTO getUser(@PathVariable Long id) {
        // (1) id → على الـ Stack (primitive-like, local variable)
        UserDTO dto = userService.findById(id); // (2) reference على الـ Stack، Object على الـ Heap
        return dto; // (3) لما الـ method تخلص، Stack frame بيتمسح
    }
}
```

كل request بيشتغل في thread عنده Stack خاص → الـ local variables معزولة. ده جزء من سبب إن Spring thread-safe للـ local variables (مش للـ instance variables في الـ Beans).

---

### StackOverflowError في Spring

أشهر حالتين:

**1. Circular Dependency + Proxy recursion:**

```java
@Service
public class A {
    @Autowired B b;
    public void doSomething() { b.callBack(); }
}

@Service
public class B {
    @Autowired A a;
    public void callBack() { a.doSomething(); } // infinite recursion → StackOverflow
}
```

**2. Recursive method من غير base case:**

```java
public int calculate(int n) {
    return calculate(n - 1); // مفيش base case → Stack frames تتراكم لحد ما تفضل
}
```

كل call بتحجز frame جديد على الـ Stack → Stack بيتملى → `StackOverflowError`.

---

### OutOfMemoryError: Java Heap Space في Spring

```java
@Service
public class ReportService {
    public List<Order> generateReport() {
        return orderRepository.findAll(); // لو الـ DB فيها 10 مليون record → Heap بتتملى
    }
}
```

أسباب شائعة في Spring:

- `findAll()` على table ضخمة من غير pagination.
- Memory leak في Singleton bean بيخزن data في instance variable (List/Map بيكبر مع كل request).
- Cache من غير eviction policy.

---

## 7. الغلطات الشائعة

|الغلطة|الصح|
|---|---|
|ظن إن الـ Object نفسه على الـ Stack|الـ Object دايمًا على الـ Heap — على الـ Stack بس الـ reference|
|مقارنة Strings بـ `==`|استخدم `.equals()` دايمًا|
|تخزين state في Singleton Bean (instance variable)|الـ Singleton مشترك بين كل الـ threads — استخدم method-local variables أو `@RequestScope`|
|استدعاء `findAll()` من غير Pagination|Heap بتتملى في بيانات كبيرة|
|اعتقاد إن الـ GC بيشتغل فورًا بعد `= null`|GC بيشتغل على حسب الـ JVM policy، مش فوري|

---

## 8. جدول المقارنة

| Aspect           | Stack                                 | Heap                                |
| ---------------- | ------------------------------------- | ----------------------------------- |
| **المحتوى**      | Primitives, References, Method frames | Objects, Instance variables         |
| **الإدارة**      | JVM تلقائي (LIFO)                     | Garbage Collector                   |
| **الـ Scope**    | Thread-specific                       | Shared بين كل الـ threads           |
| **السرعة**       | أسرع                                  | أبطأ نسبيًا                         |
| **الحجم**        | صغير (default ~512KB–1MB per thread)  | كبير (JVM Heap `-Xmx`)              |
| **الـ Error**    | `StackOverflowError`                  | `OutOfMemoryError: Java heap space` |
| **الـ Lifetime** | محدود بالـ method                     | يعيش لحد ما GC يمسحه                |
| **String Pool**  | —                                     | موجودة هنا (من Java 7+)             |