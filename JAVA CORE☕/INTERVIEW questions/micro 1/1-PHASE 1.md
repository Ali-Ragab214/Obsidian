تمام، هنا كل حاجة من الـ 6 نقط، كل واحدة فيها: **ملخص مختصر ومفيد + أسئلة الانترفيو (مقالي + MCQ + Tracing) مع إجاباتها**.

---

# 1️⃣ Collection framework 
## 1)

**What is Collection Framework?**

> هي مجموعة Interfaces وClasses لتخزين وإدارة البيانات بكفاءة.

---

## 2)

**Difference between Collection and Collections?**

Collection

→ Interface.

Collections

→ Utility Class فيها Methods مثل

```
Collections.sort()

Collections.reverse()

Collections.shuffle()
```

---

## 3)

**Difference between List and Set?**

|List|Set|
|---|---|
|يسمح بالتكرار|لا يسمح|
|يوجد Index|لا يوجد|
|يحافظ على الترتيب|حسب الـ Implementation|

---

## 4)

**ArrayList vs LinkedList**

|ArrayList|LinkedList|
|---|---|
|Dynamic Array|Doubly Linked List|
|قراءة سريعة|قراءة أبطأ|
|حذف بطيء|حذف أسرع من الأطراف|

---

## 5)

**HashSet vs TreeSet**

HashSet

- أسرع.
- لا يرتب.

TreeSet

- يرتب.
- أبطأ.

---

## 6)

**HashMap vs TreeMap**

HashMap

```
O(1)
```

TreeMap

```
O(log n)
```

---

## 7)

**HashMap vs Hashtable**

HashMap

- ليس Thread Safe.
- يسمح بـ null.

Hashtable

- Thread Safe.
- لا يسمح بـ null.
- قديم ونادر الاستخدام حاليًا.

---

## 8)

**HashMap vs ConcurrentHashMap**

ConcurrentHashMap

- Thread Safe.
- أفضل أداء من Hashtable.
- يستخدم في Multi-threading.

---

## 9)

**Can HashMap have duplicate keys?**

لا.

أي `put()` بنفس الـ Key سيستبدل القيمة القديمة.

---

## 10)

**Can HashMap have duplicate values?**

نعم.

---

## 11)

**Why Set doesn't allow duplicates?**

لأنه يعتمد على:

- `hashCode()`
- `equals()`

لتحديد إذا كان العنصر موجودًا بالفعل.

---

## 12)

**What is the difference between Comparable and Comparator?**

Comparable

- ترتيب طبيعي (Natural Ordering).
- تكتب `compareTo()` داخل الكلاس.

Comparator

- ترتيب مخصص (Custom Ordering).
- كلاس منفصل يحتوي `compare()`.

---

## 13)

**What is Fail-Fast Iterator?**

لو عدلت على الـ Collection أثناء الـ Iteration مباشرة، سترى:

```
ConcurrentModificationException
```

مثال:

```
for (String s : list) {
    list.remove(s); // Exception
}
```

---

## 14)

**What is the Time Complexity of HashMap?**

- Average: `O(1)`
- Worst Case: `O(n)` (وفي الإصدارات الحديثة قد تتحسن مع استخدام الشجرة في بعض حالات التصادم).

---

## 15)

**Which Collection should you choose?**

- قراءة كثيرة → `ArrayList`
- حذف وإضافة من الأطراف → `LinkedList`
- إزالة التكرار → `HashSet`
- ترتيب تلقائي → `TreeSet`
- Key-Value → `HashMap`
- Multi-threaded Map → `ConcurrentHashMap`

---

# أهم 7 كلاسات لازم تكون حافظهم للانترفيو

1. `ArrayList`
2. `LinkedList`
3. `HashSet`
4. `TreeSet`
5. `HashMap`
6. `TreeMap`
7. `ConcurrentHashMap`

لو فهمت السبعة دول كويس، هتكون مغطّي أغلب أسئلة الـ Collection Framework في مقابلات الـ Java Backend.

---

# 2️⃣ Multithreading & Concurrency

## الملخص المختصر

- **Thread**: أصغر وحدة تنفيذ، بتتعمل بـ `Thread` class أو `Runnable` interface.
- **`synchronized`**: بيمنع أكتر من Thread من الدخول لنفس الـ block/method في نفس الوقت (Mutual Exclusion) عشان يمنع Race Conditions.
- **`ExecutorService`**: بيدير مجموعة Threads (Thread Pool) بدل ما تعمل Thread بنفسك يدويًا، بيوفر عليك overhead إنشاء وتدمير الـ Threads.
- **`Future` & `CompletableFuture`**: بتمثل نتيجة عملية Async هتخلص في المستقبل. `CompletableFuture` بيدّيك تحكم أكتر (chaining, combining multiple async calls).
- **`volatile`**: بيضمن إن القيمة بتتقرا من الـ Main Memory مباشرة مش من الـ CPU Cache، بيحل مشكلة الـ Visibility بس مش الـ Atomicity.
- **Deadlock**: لما اتنين Threads بيستنوا بعض على موارد (locks) بشكل دائري.

## أسئلة الانترفيو

**مقالي:**

1. إيه الفرق بين `Thread` و `Runnable`؟

> `Thread` هي class لو ورثت منها معنتهاش وراثة من حاجة تانية (Java single inheritance)، أما `Runnable` interface بتقدر تعمله implement وتفضل حر تورث من class تاني كمان، فبشكل عام استخدام `Runnable` بيفضل أفضل practice.

2. إيه الفرق بين `synchronized` method و `synchronized` block؟

> `synchronized method` بتقفل على مستوى الـ object أو الـ class كله، أما `synchronized block` بتقفل بس على جزء معين من الكود (وممكن تحدد الـ lock object نفسه)، وده بيدي أداء أحسن لأنك مش بتقفل حاجات مش محتاجة تتقفل.

3. إيه هو الـ Deadlock وإزاي تتجنبه؟

> Deadlock بيحصل لما Thread A مستني lock ماسكه Thread B، وThread B مستني lock ماسكه Thread A، فمفيش حد بيتحرك. بتتجنبه عن طريق: ترتيب ثابت لاكتساب الـ locks، استخدام `tryLock()` مع timeout، أو تقليل عدد الـ locks المطلوبة.

4. إيه الفرق بين `Future` و `CompletableFuture`؟

> `Future` بيدّيك نتيجة async بس مفيهوش طريقة سهلة تعمل chaining أو تجمع بين نتائج كذا async call. `CompletableFuture` بيدّيك methods زي `thenApply`, `thenCombine`, `thenAccept` عشان تبني pipeline كامل من العمليات الـ Async.

**MCQ:** 5. أنهي كلمة مفتاحية بتضمن الـ Visibility بس مش الـ Atomicity؟ a) synchronized b) volatile c) transient d) final

> **b) volatile**

6. إيه اللي بيحصل لو استخدمت `ExecutorService` بدل ما تعمل Thread يدويًا؟ a) أداء أسوأ b) إعادة استخدام Threads وتقليل الـ overhead c) مفيش فرق d) مينفعش تعمل Async

> **b) إعادة استخدام Threads وتقليل الـ overhead**

**Tracing:**

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});
System.out.println("Before get");
System.out.println(future.get());
executor.shutdown();
```

> الإجابة: هيطبع `Before get` الأول فورًا، وبعدها بعد ثانية هيطبع `42`، لأن `future.get()` بتعمل block لحد ما النتيجة تكون جاهزة.

---

# 3️⃣ الموضوع ده من أكتر المواضيع اللي بتتسأل في انترفيوهات Java وSpring Boot، لكن أغلب الناس بتحفظه بدل ما تفهمه. خلينا نشرحه كأننا في انترفيو.

---

# أولاً: يعني إيه Exception؟

الـ **Exception** هو حدث غير متوقع (Unexpected Event) بيحصل أثناء تنفيذ البرنامج ويؤدي إلى تعطله أو تغيير مسار التنفيذ.

مثال:

```java
int x = 10 / 0;
```

هيحصل

```text
ArithmeticException: / by zero
```

أو

```java
String name = null;
System.out.println(name.length());
```

هيحصل

```text
NullPointerException
```

---

# ليه Java عملت Exception Handling؟

تخيل عندك كود:

```java
public void readFile() {
    FileReader file = new FileReader("data.txt");
    ...
}
```

لو الملف مش موجود؟

من غير Exception Handling البرنامج كله هيقف.

لكن باستخدام Exception Handling:

```java
try {
    FileReader file = new FileReader("data.txt");
} catch (IOException e) {
    System.out.println("File not found");
}
```

البرنامج هيكمل شغال.

---

# Hierarchy

لازم تكون حافظها

```text
Object
   │
Throwable
   │
───────────────
│             │
Error      Exception
                 │
        RuntimeException
```

---

## Throwable

أبو كل الـ Exceptions والـ Errors.

أي حاجة ممكن تتعملها

```java
throw
```

لازم تكون وارثة من Throwable.

---

# Error

دي أخطاء JVM نفسها.

مثلاً

```text
OutOfMemoryError

StackOverflowError
```

دي غالبًا مش بتتعالج.

---

# Exception

دي الأخطاء اللي البرنامج ممكن يتعامل معاها.

وده اللي إحنا بنستخدمه.

---

# أنواع Exceptions

## 1) Checked Exception

دي Exceptions الـ Compiler بيجبرك تتعامل معاها.

مثال

```java
FileReader file = new FileReader("test.txt");
```

هيقولك

```text
Unhandled exception
```

لازم تعمل

```java
try {
    FileReader file = new FileReader("test.txt");
}
catch(IOException e){

}
```

أو

```java
throws IOException
```

---

## أشهر Checked Exceptions

```text
IOException

SQLException

FileNotFoundException

ClassNotFoundException
```

---

## لماذا اسمها Checked؟

لأن

الـ Compiler

بيعمل Check عليها.

---

# 2) Unchecked Exception

دي بترث من

```java
RuntimeException
```

والـ Compiler

مش هيجبرك تمسكها.

مثال

```java
String s = null;

s.length();
```

البرنامج هيCompile.

لكن هيقع وقت التشغيل.

---

## أشهر Runtime Exceptions

```text
NullPointerException

ArithmeticException

ArrayIndexOutOfBoundsException

IllegalArgumentException

IllegalStateException

NumberFormatException
```

---

# مقارنة

|Checked|Unchecked|
|---|---|
|Compile Time|Runtime|
|لازم try-catch أو throws|مش إجباري|
|يرث من Exception|يرث من RuntimeException|
|IOException|NullPointerException|

---

# throw vs throws

من أشهر الأسئلة.

---

## throw

بترمي Exception.

مثال

```java
throw new RuntimeException("Error");
```

Exception واحدة.

داخل الـ Method.

---

## throws

بتعلن إن الـ Method ممكن ترمي Exception.

```java
public void read()
throws IOException
{
}
```

---

# try-catch

مثال

```java
try{

    int x = 10 / 0;

}

catch(ArithmeticException e){

    System.out.println(e.getMessage());

}
```

بدل ما البرنامج يقع.

يمسك الـ Exception.

---

# Multiple Catch

```java
try{

}
catch(IOException e){

}
catch(SQLException e){

}
```

---

# Multi Catch

من Java 7

```java
catch(IOException | SQLException e){

}
```

---

# finally

من أشهر الأسئلة.

finally

بيتنفذ دائمًا.

حتى لو

```java
return
```

---

مثال

```java
public static int test(){

    try{

        return 10;

    }

    finally{

        System.out.println("Finally");

    }

}
```

الناتج

```text
Finally

10
```

---

ليه؟

لأن الـ JVM لما تشوف `return` داخل `try`، **بتحسب قيمة الـ return وتحفظها مؤقتًا**، لكن قبل ما تخرج من الـ Method لازم تنفذ `finally`. بعد انتهاء `finally` ترجع القيمة المحفوظة.

---

مثال أخطر

```java
public static int test(){

    try{

        return 10;

    }

    finally{

        return 20;

    }

}
```

الناتج

```text
20
```

لأن

finally

عملت Override

للـ Return.

> **Best Practice:** لا تكتب `return` داخل `finally` لأنه يخفي النتيجة الأصلية وقد يخفي Exceptions أيضًا.

---

# هل finally دائمًا يتنفذ؟

لا.

في حالات نادرة.

مثل

```java
System.exit(0);
```

أو لو الـ JVM انهارت.

---

# Custom Exception

دي أهم حاجة في Spring.

مثلاً

بتعمل Banking System.

لو الرصيد مش كفاية.

ينفع أعمل

```java
throw new RuntimeException();
```

ينفع.

لكن ده سيئ.

---

الأفضل

```java
throw new InsufficientBalanceException();
```

بقى واضح جدًا.

---

مثال

```java
public class ProductNotFoundException
extends RuntimeException{

    public ProductNotFoundException(String message){

        super(message);

    }

}
```

---

الاستخدام

```java
if(product==null){

    throw new ProductNotFoundException(

        "Product not found"

    );

}
```

---

# لماذا غالبًا نورث من RuntimeException؟

في Spring Boot

الـ Business Errors

زي

```text
Product Not Found

Email Already Exists

Invalid Password

Order Already Shipped
```

دي كلها Runtime.

مش عايزين كل Method

تكتب

```java
throws ProductNotFoundException
```

---

# Spring Boot Exception Handling

بدل

```java
try{

}
catch(){

}
```

في كل Controller

نعمل

Global Handler.

---

```java
@ControllerAdvice
```

يعني

كل Controllers.

---

```java
@ExceptionHandler
```

يعني

لما Exception معين يحصل.

نفذ الـ Method دي.

---

مثال

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<String> handle(

        ProductNotFoundException ex

    ){

        return ResponseEntity

                .status(404)

                .body(ex.getMessage());

    }

}
```

---

لو حصل

```java
throw new ProductNotFoundException("Not Found");
```

الـ Client

هيشوف

```http
404

Not Found
```

بدل

```http
500
```

---

# أفضل Practice

بدل String

اعمل

```java
ErrorResponse
```

```java
public class ErrorResponse{

    private String message;

    private LocalDateTime timestamp;

    private int status;

}
```

وده اللي بيحصل في المشاريع.

---

# أشهر أسئلة الانترفيو

## 1)

**What is the difference between Error and Exception?**

> Error يمثل مشاكل في الـ JVM غالبًا لا يمكن التعافي منها مثل `OutOfMemoryError`، أما Exception فهو يمثل أخطاء يمكن للتطبيق التعامل معها.

---

## 2)

**What is the difference between Checked and Unchecked Exceptions?**

> Checked Exceptions يتم التحقق منها أثناء الـ Compile ويجب التعامل معها أو الإعلان عنها بـ `throws`. أما Unchecked Exceptions فترث من `RuntimeException` ولا يجبرك الـ Compiler على التعامل معها.

---

## 3)

**Why do RuntimeExceptions exist?**

> لأنها تمثل أخطاء برمجية (Programming Errors) مثل `NullPointerException` أو `IllegalArgumentException`، ومن الأفضل إصلاح السبب بدلًا من إجبار كل Method على كتابة `try-catch`.

---

## 4)

**Difference between throw and throws?**

|throw|throws|
|---|---|
|يرمي Exception|يعلن أن الـ Method قد ترمي Exception|
|داخل الـ Method|في Signature|
|Exception واحدة|يمكن إعلان أكثر من Exception|

---

## 5)

**Can we have multiple catch blocks?**

نعم.

---

## 6)

**Can finally execute after return?**

نعم.

finally

يتنفذ أولًا.

---

## 7)

**Can finally not execute?**

نعم.

في حالات مثل

```java
System.exit()
```

أو انهيار الـ JVM.

---

## 8)

**Why create Custom Exceptions?**

> لتمثيل أخطاء الـ Business Logic بأسماء واضحة، وتحسين قابلية القراءة، وتسهيل الـ Global Exception Handling.

---

## 9)

**Why do most Spring Boot custom exceptions extend RuntimeException?**

> لأن معظم أخطاء الـ Business Logic هي Unchecked Exceptions، ولا نريد إجبار كل Service وController على كتابة `throws` أو `try-catch` في كل مكان.

---

## 10)

**What is @ControllerAdvice?**

> هو Global Exception Handler يلتقط الـ Exceptions من جميع الـ Controllers في مكان واحد.

---

## 11)

**What is @ExceptionHandler?**

> Annotation تحدد أي Exception سيتم التعامل معه داخل Method معينة.

---

## 12)

**What happens if no one catches an Exception?**

> تنتقل للأعلى في سلسلة الاستدعاءات (Call Stack). وإذا لم يتم التعامل معها حتى تصل إلى الـ JVM، ينتهي تنفيذ الـ Thread ويُطبع الـ Stack Trace.

---

# سؤال Tracing مشهور

```java
public static void main(String[] args) {

    try {

        System.out.println("A");

        int x = 10 / 0;

        System.out.println("B");

    } catch (ArithmeticException e) {

        System.out.println("C");

    } finally {

        System.out.println("D");

    }

    System.out.println("E");

}
```

**الإجابة:**

```text
A
C
D
E
```

لأن السطر الذي يطبع `B` لن يُنفذ بعد حدوث الـ Exception، وينتقل التنفيذ إلى `catch` ثم `finally` ثم يكمل البرنامج بعد كتلة `try-catch-finally`.

---

## الإجابة الذهبية في الانترفيو (30 ثانية)

> **Java Exception Handling is a mechanism for handling runtime problems without crashing the application. Exceptions are divided into Checked Exceptions, which must be handled or declared, and Unchecked Exceptions, which extend RuntimeException and are typically caused by programming errors. In Spring Boot, business-related exceptions are usually implemented as custom RuntimeExceptions and handled centrally using @ControllerAdvice and @ExceptionHandler to return consistent HTTP error responses and keep controllers clean.**
---

# 4️⃣ 
---

# 6️⃣ Java 8+ Features (Streams, Lambda, Optional)

## الملخص المختصر

- **Lambda Expressions**: طريقة مختصرة لكتابة Anonymous Functions، بتستخدم مع Functional Interfaces (زي `Runnable`, `Comparator`, `Function<T,R>`).
- **Streams API**: بتدّيك طريقة functional-style لمعالجة الـ Collections (filter, map, reduce, collect) بدل الـ for loops التقليدية.
- **Intermediate vs Terminal operations**: Intermediate (زي `filter`, `map`) بترجع Stream تاني ولازيّ (Lazy)، Terminal (زي `collect`, `forEach`) بتنفذ العملية فعليًا.
- **`Optional<T>`**: Container objects بتستخدم عشان تتجنب `NullPointerException`، بيوضح إن القيمة ممكن تكون موجودة أو لأ.
- **Method References**: اختصار أكتر للـ Lambda لما بتنادي method موجودة بالفعل (زي `String::toUpperCase`).

## أسئلة الانترفيو

**مقالي:**

1. إيه الفرق بين Intermediate و Terminal Operations في Stream؟

> Intermediate operations (زي `filter`, `map`, `sorted`) بترجع Stream جديد وبتتنفذ بطريقة Lazy، يعني مش بتتنفذ فعليًا إلا لما يتنادى Terminal operation. Terminal operations (زي `collect`, `forEach`, `reduce`) هي اللي بتبدأ تنفيذ السلسلة كلها وبترجع نتيجة نهائية (مش Stream).

2. ليه نستخدم `Optional` بدل ما نرجع `null` مباشرة؟

> عشان نجبر الـ caller إنه يتعامل بشكل صريح مع احتمالية إن القيمة مش موجودة (باستخدام `isPresent()`, `orElse()`, `map()`)، وده بيقلل احتمالية حصول `NullPointerException` بشكل كبير لأن الكود بيوضح النية بشكل واضح.

3. إيه الفرق بين `map()` و `flatMap()` في Stream؟

> `map()` بتحول كل عنصر لعنصر تاني (1-to-1)، بينما `flatMap()` بتستخدم لما كل عنصر نفسه فيه Collection أو Stream، فبتـ"فرد" (flatten) النتيجة كلها في Stream واحد بدل Stream of Streams.

**MCQ:** 4. أنهي دول Terminal Operation في Streams؟ a) filter b) map c) collect d) sorted

> **c) collect**

5. إيه اللي بيرجعه `Optional.empty()`؟ a) null b) Object فاضي بيمثل عدم وجود قيمة c) Exception d) 0

> **b) Object فاضي بيمثل عدم وجود قيمة**

**Tracing:**

```java
List<String> names = Arrays.asList("Ahmed", "Sara", "Omar", "Mona");
List<String> result = names.stream()
    .filter(n -> n.length() > 4)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
System.out.println(result);
```

> الإجابة: `[AHMED]` — بيفلتر الأسماء اللي طولها أكبر من 4 حروف بس (Ahmed=5 ✓, Sara=4 ✗, Omar=4 ✗, Mona=4 ✗)، وبعدين يحول الناتج لـ Uppercase.

```java
Optional<String> opt = Optional.ofNullable(null);
System.out.println(opt.orElse("Default Value"));
```

> الإجابة: `Default Value` — لأن الـ Optional فاضي (empty)، فبيرجع القيمة الافتراضية المحددة في `orElse()`.

---

عايزني أكمل بنفس الطريقة على المحور الجاي (Spring Ecosystem)، ولا نركز أكتر في حاجة معينة هنا الأول؟