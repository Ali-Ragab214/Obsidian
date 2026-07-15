# Final vs Finally vs Finalize (سؤال إنترفيو كلاسيكي)

## السؤال المتوقع

> "What's the difference between final, finally, and finalize in Java?"

## الإجابة المبسطة

الثلاثة كلمات بتتلخبط مع بعض لأنهم بيبدأوا بنفس الحروف، بس كل واحدة ليها دور مختلف تمامًا ومفيش علاقة بينهم غير الاسم.

---

### 1️⃣ `final` — Keyword (كلمة محجوزة)

بتُستخدم عشان تمنع التعديل / التغيير. بتتحط على 3 حاجات مختلفة:

|تستخدمها على|المعنى|
|---|---|
|**Variable**|القيمة تتحط مرة واحدة بس، مينفعش تتغير تاني (constant)|
|**Method**|الميثود دي مينفعش تتعمللها Override في subclass|
|**Class**|الكلاس ده مينفعش يتعمله extend (inheritance)|

**مثال:**

```java
final int MAX_USERS = 100; // constant, مينفعش يتغير
// MAX_USERS = 200; ❌ compile error

class Animal {
    final void breathe() { // مينفعش override
        System.out.println("Breathing...");
    }
}

final class Utils { } // مينفعش أعمل extend Utils
```

---

### 2️⃣ `finally` — Block (في الـ exception handling)

جزء من الـ `try-catch`، وبيتنفذ **دايمًا** سواء حصل exception أو لأ، وسواء الـ exception اتعالجت أو لأ. بيُستخدم للـ cleanup (زي قفل connection أو file).

**مثال:**

```java
try {
    int result = 10 / 0; // هيرمي exception
} catch (ArithmeticException e) {
    System.out.println("Error: " + e.getMessage());
} finally {
    System.out.println("This always runs!"); // بتتنفذ في كل الحالات
}
```

**Real-world use case:**

```java
Connection conn = null;
try {
    conn = dataSource.getConnection();
    // ... use connection
} finally {
    if (conn != null) conn.close(); // تقفل الـ connection مهما حصل
}
```

---

### 3️⃣ `finalize()` — Method (في الـ Object class)

ميثود جوه كلاس `Object`، بتتنفذ نظريًا قبل ما الـ Garbage Collector يشيل الـ object من الـ memory. **مش مضمونة تتنفذ أصلًا**، وده سبب إنها **deprecated** من Java 9 ومشالوها خالص في Java 18+.

**مثال (تاريخي/قديم، متستخدمهاش دلوقتي):**

```java
@Override
protected void finalize() throws Throwable {
    System.out.println("Object is being garbage collected");
}
```

⚠️ في الإنترفيو لازم تقول: "finalize() قديمة جدًا و deprecated، البديل الحديث هو `try-with-resources` أو `AutoCloseable` interface."

---

## الجدول اللي هيفضل في دماغك بسرعة

||final|finally|finalize|
|---|---|---|---|
|النوع|Keyword|Block|Method|
|الاستخدام|منع التعديل (variable/method/class)|تنفيذ كود دايمًا بعد try-catch|تنظيف قبل الـ garbage collection|
|مرتبط بـ|OOP / Constants|Exception Handling|Garbage Collector / Object class|
|Deprecated؟|لأ|لأ|✅ Deprecated (Java 9+)|

---

## رد سريع لو سألوك في الإنترفيو (elevator pitch)

> "final بتمنع تغيير حاجة — variable, method, أو class. finally هو block بيتنفذ دايمًا في الـ try-catch عشان الـ cleanup. أما finalize فهي method قديمة كانت بتتنفذ قبل الـ garbage collection، بس دلوقتي deprecated ومحدش بيستخدمها."