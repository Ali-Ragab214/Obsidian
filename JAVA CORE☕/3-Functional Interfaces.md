# Functional Interfaces & Lambda Expressions in Java

> **الهدف:** فهم الـ Functional Interfaces من الصفر وإزاي Lambda بتشتغل معاهم

---

## 1. المشكلة اللي Lambda جات تحلها

قبل Java 8، لو عايز تعدي "سلوك" لدالة تانية كنت محتاج تعمل **Anonymous Inner Class** — وده كان مرهق جداً:

```java
// قبل Java 8 ❌ — كود كتير لحاجة بسيطة
List<String> names = Arrays.asList("ALi", "ALi", "Sara");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});
```

```java
// بعد Java 8 ✅ — نفس النتيجة في سطر واحد
Collections.sort(names, (a, b) -> a.compareTo(b));
```

---

## 2. إيه هو الـ Functional Interface؟

> **تعريف:** Interface فيها **method واحدة بس abstract** — مش أكتر.

```java
@FunctionalInterface
public interface MyInterface {
    void doSomething();  // method واحدة بس ✅
}
```

الـ `@FunctionalInterface` annotation اختيارية — بس بتخلي الـ compiler يتأكد إن مفيش أكتر من method واحدة.

### ليه method واحدة بس؟

لأن الـ Lambda بتعمل implement لـ method واحدة — لو في أكتر من method مش هتعرف هي بتعمل implement لأنهي واحدة منهم.

```java
@FunctionalInterface
interface Greeter {
    String greet(String name);  // method واحدة
}

// Lambda بتعمل implement لـ greet
Greeter g = name -> "Hello, " + name + "!";
g.greet("Ahmed");  // "Hello, Ahmed!"
```

---

## 3. الـ Lambda Syntax

```
(parameters) -> expression
(parameters) -> { statements; }
```

### الحالات المختلفة

```java
// بدون parameters
Runnable r = () -> System.out.println("Hello!");

// parameter واحد — الأقواس اختيارية
Consumer<String> c = name -> System.out.println(name);

// أكتر من parameter
Comparator<Integer> comp = (a, b) -> a - b;

// body أكتر من سطر — محتاج {} و return
Function<Integer, String> f = (n) -> {
    if (n > 0) return "positive";
    if (n < 0) return "negative";
    return "zero";
};
```

---

## 4. الـ Functional Interfaces الجاهزة في Java

Java 8 جابت package كامل جاهز: `java.util.function`

---

### 4.1 `Predicate<T>`

> **بياخد T ويرجع boolean** — بيستخدم في الفلترة

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

```

#### أمثلة

```java
// تحقق إن الرقم موجب
Predicate<Integer> isPositive = n -> n > 0;
isPositive.test(5);   // true
isPositive.test(-3);  // false

// تحقق إن الـ String مش فاضية
Predicate<String> notEmpty = s -> !s.isEmpty();
notEmpty.test("Hello");  // true
notEmpty.test("");       // false

// تحقق إن الـ String طولها أكتر من 5
Predicate<String> isLong = s -> s.length() > 5;
```

#### تركيب Predicates

```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven     = n -> n % 2 == 0;

// and — الاتنين لازم يكونوا true
Predicate<Integer> positiveAndEven = isPositive.and(isEven);
positiveAndEven.test(4);   // true
positiveAndEven.test(-4);  // false
positiveAndEven.test(3);   // false

// or — واحد منهم كافي
Predicate<Integer> positiveOrEven = isPositive.or(isEven);
positiveOrEven.test(3);   // true  (موجب)
positiveOrEven.test(-4);  // true  (زوجي)
positiveOrEven.test(-3);  // false (لا موجب ولا زوجي)

// negate — عكس النتيجة
Predicate<Integer> isNegative = isPositive.negate();
isNegative.test(-5);  // true
isNegative.test(5);   // false
```

#### مع Stream

```java
List<Integer> numbers = Arrays.asList(1, -2, 3, -4, 5, 6);

Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven     = n -> n % 2 == 0;

List<Integer> positiveEvens = numbers.stream()
    .filter(isPositive.and(isEven))
    .collect(Collectors.toList());
// [6]
```

---

### 4.2 `Function<T, R>`

> **بياخد T ويرجع R** — بيستخدم في التحويل

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

#### أمثلة

```java
// حول String لـ Integer
Function<String, Integer> strToInt = s -> Integer.parseInt(s);
strToInt.apply("42");  // 42

// جيب طول الـ String
Function<String, Integer> getLength = s -> s.length();
getLength.apply("Hello");  // 5

// حول Integer لـ String
Function<Integer, String> intToStr = n -> "Number: " + n;
intToStr.apply(10);  // "Number: 10"
```

#### تركيب Functions

```java
Function<String, Integer> getLength = s -> s.length();
Function<Integer, Boolean> isLong   = n -> n > 5;

// andThen — شغّل getLength الأول، ثم isLong على النتيجة
Function<String, Boolean> isStringLong = getLength.andThen(isLong);
isStringLong.apply("Hello");      // 5 → false
isStringLong.apply("Hello World"); // 11 → true

// compose — عكس andThen
Function<Integer, Integer> times2 = x -> x * 2;
Function<Integer, Integer> plus3  = x -> x + 3;

Function<Integer, Integer> plus3ThenTimes2 = times2.compose(plus3);
plus3ThenTimes2.apply(4);  // (4+3)*2 = 14

Function<Integer, Integer> times2ThenPlus3 = times2.andThen(plus3);
times2ThenPlus3.apply(4);  // (4*2)+3 = 11
```

#### مع Stream

```java
List<String> names = Arrays.asList("ahmed", "sara", "omar");

Function<String, String> toUpper  = String::toUpperCase;
Function<String, Integer> getLen  = String::length;

// map بياخد Function
List<String> upperNames = names.stream()
    .map(toUpper)
    .collect(Collectors.toList());
// ["AHMED", "SARA", "OMAR"]

List<Integer> lengths = names.stream()
    .map(getLen)
    .collect(Collectors.toList());
// [5, 4, 4]
```

---

### 4.3 `Consumer<T>`

> **بياخد T ومش بيرجع حاجة (void)** — بيستخدم للطباعة والتعديل

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

#### أمثلة

```java
// اطبع الـ String
Consumer<String> print = s -> System.out.println(s);
print.accept("Hello!");  // Hello!

// اطبع بـ uppercase
Consumer<String> printUpper = s -> System.out.println(s.toUpperCase());
printUpper.accept("hello");  // HELLO

// سجّل في الـ database (مثال)
Consumer<User> saveUser = user -> userRepository.save(user);
```

#### andThen — سلسلة consumers

```java
Consumer<String> print    = s -> System.out.println("Print: " + s);
Consumer<String> log      = s -> System.out.println("Log: " + s);
Consumer<String> printAndLog = print.andThen(log);

printAndLog.accept("Ahmed");
// Print: Ahmed
// Log: Ahmed
```

#### مع Stream

```java
List<String> names = Arrays.asList("Ahmed", "Sara", "Omar");

// forEach بياخد Consumer
names.stream()
    .forEach(name -> System.out.println("Hello, " + name));
// Hello, Ahmed
// Hello, Sara
// Hello, Omar
```

---

### 4.4 `Supplier<T>`

> **مش بياخد حاجة وبيرجع T** — بيستخدم للإنشاء والتوليد

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

#### أمثلة

```java
// رجّع string ثابتة
Supplier<String> greeting = () -> "Hello, World!";
greeting.get();  // "Hello, World!"

// رجّع الوقت الحالي
Supplier<LocalDate> today = () -> LocalDate.now();
today.get();  // 2026-05-09

// رجّع list جديدة
Supplier<List<String>> newList = () -> new ArrayList<>();
newList.get();  // []

// Factory pattern
Supplier<User> userFactory = () -> new User("Guest", "guest@email.com");
User guest = userFactory.get();
```

#### مع Optional

```java
// orElseGet بياخد Supplier — lazy evaluation
Optional<String> name = Optional.empty();

// ❌ orElse — بيتنفذ دايماً حتى لو Optional فيه قيمة
String result1 = name.orElse(expensiveOperation());

// ✅ orElseGet — بيتنفذ بس لو Optional فاضي
String result2 = name.orElseGet(() -> expensiveOperation());
```

---

### 4.5 `Comparator<T>`

> **بياخد T, T ويرجع int** — بيستخدم للترتيب

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    // سالب → o1 قبل o2
    // صفر  → متساويين
    // موجب → o2 قبل o1
}
```

#### أمثلة

```java
// ترتيب تصاعدي
Comparator<Integer> asc = (a, b) -> a - b;

// ترتيب تنازلي
Comparator<Integer> desc = (a, b) -> b - a;

// ترتيب Strings حسب الطول
Comparator<String> byLength = (a, b) -> a.length() - b.length();

// ترتيب Students حسب الـ GPA
Comparator<Student> byGpa = (s1, s2) -> Double.compare(s1.getGpa(), s2.getGpa());
```

#### Comparator Helper Methods

```java
List<String> names = Arrays.asList("Omar", "Ahmed", "Sara", "Ali");

// comparing — أنيق وآمن
names.sort(Comparator.comparing(String::length));
// ["Ali", "Sara", "Omar", "Ahmed"]

// reversed
names.sort(Comparator.comparing(String::length).reversed());
// ["Ahmed", "Omar", "Sara", "Ali"]

// thenComparing — ترتيب على أكتر من field
List<Student> students = ...;
students.sort(
    Comparator.comparing(Student::getDepartment)
              .thenComparingDouble(Student::getGpa)
              .reversed()
);

// naturalOrder / reverseOrder
names.sort(Comparator.naturalOrder());   // A → Z
names.sort(Comparator.reverseOrder());  // Z → A
```

---

### 4.6 `UnaryOperator<T>`

> **بياخد T ويرجع T** — نفس النوع — extension من Function

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    T apply(T t);
}
```

#### أمثلة

```java
// ضاعف الرقم
UnaryOperator<Integer> doubler = x -> x * 2;
doubler.apply(5);  // 10

// حول الـ String لـ uppercase
UnaryOperator<String> upper = String::toUpperCase;
upper.apply("hello");  // "HELLO"

// أضف prefix
UnaryOperator<String> addPrefix = s -> "Mr. " + s;
addPrefix.apply("Ahmed");  // "Mr. Ahmed"
```

#### مع List.replaceAll

```java
List<String> names = new ArrayList<>(Arrays.asList("ahmed", "sara", "omar"));

// replaceAll بياخد UnaryOperator
names.replaceAll(String::toUpperCase);
// ["AHMED", "SARA", "OMAR"]

names.replaceAll(s -> s + "!");
// ["AHMED!", "SARA!", "OMAR!"]
```

---

### 4.7 `BinaryOperator<T>`

> **بياخد T, T ويرجع T** — extension من BiFunction

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    T apply(T t1, T t2);
}
```

#### أمثلة

```java
// جمع رقمين
BinaryOperator<Integer> add = (a, b) -> a + b;
add.apply(3, 5);  // 8

// الأكبر من رقمين
BinaryOperator<Integer> max = Integer::max;
max.apply(10, 20);  // 20

// دمج Strings
BinaryOperator<String> concat = (a, b) -> a + " " + b;
concat.apply("Hello", "World");  // "Hello World"
```

#### مع Stream.reduce

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// reduce بياخد BinaryOperator
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// 15

int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// 120

// أو بـ method reference
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);
```

---

### 4.8 `BiFunction<T, U, R>`

> **بياخد T و U ويرجع R** — زي Function بس بتاخد parameter تاني

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
```

#### أمثلة

```java
// جمع String وInteger
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
repeat.apply("ha", 3);  // "hahaha"

// دمج اسمين
BiFunction<String, String, String> fullName = (first, last) -> first + " " + last;
fullName.apply("Ahmed", "Ali");  // "Ahmed Ali"

// حساب مساحة مستطيل
BiFunction<Double, Double, Double> area = (w, h) -> w * h;
area.apply(5.0, 3.0);  // 15.0
```

---

### 4.9 `BiPredicate<T, U>`

> **بياخد T و U ويرجع boolean**

```java
// تحقق إن الـ String أطول من الرقم المعطى
BiPredicate<String, Integer> longerThan = (s, n) -> s.length() > n;
longerThan.test("Hello", 3);   // true
longerThan.test("Hi", 3);      // false

// تحقق إن الرقم الأول أكبر من التاني
BiPredicate<Integer, Integer> greaterThan = (a, b) -> a > b;
greaterThan.test(10, 5);  // true
```

---

### 4.10 `BiConsumer<T, U>`

> **بياخد T و U ومش بيرجع حاجة**

```java
// اطبع key-value
BiConsumer<String, Integer> printEntry = (k, v) -> 
    System.out.println(k + " = " + v);
printEntry.accept("age", 25);  // age = 25

// Map.forEach بياخد BiConsumer
Map<String, Integer> scores = Map.of("Ahmed", 95, "Sara", 88);
scores.forEach((name, score) -> 
    System.out.println(name + ": " + score)
);
```

---

## 5. Method References — اختصار أنيق للـ Lambda

لما الـ Lambda بتعمل method call بس — تقدر تختصرها بـ `::`

### الأنواع الأربعة

```java
// 1. Static Method Reference
// Lambda:   s -> Integer.parseInt(s)
Function<String, Integer> parse = Integer::parseInt;

// Lambda:   n -> Math.abs(n)
Function<Integer, Integer> abs = Math::abs;

// 2. Instance Method على نفس الـ object
// Lambda:   s -> s.toUpperCase()
Function<String, String> upper = String::toUpperCase;

// Lambda:   s -> s.length()
Function<String, Integer> len = String::length;

// Lambda:   s -> s.isEmpty()
Predicate<String> empty = String::isEmpty;

// 3. Instance Method على object خارجي
PrintStream out = System.out;
// Lambda:   s -> out.println(s)
Consumer<String> print = System.out::println;

MyService service = new MyService();
// Lambda:   () -> service.run()
Runnable task = service::run;

// 4. Constructor Reference
// Lambda:   () -> new ArrayList<>()
Supplier<List<String>> newList = ArrayList::new;

// Lambda:   name -> new User(name)
Function<String, User> createUser = User::new;
```

---

## 6. ملخص سريع — Cheat Sheet

|Interface|Method|Input|Output|استخدام|
|---|---|---|---|---|
|`Predicate<T>`|`test(T)`|T|boolean|filter, conditions|
|`Function<T,R>`|`apply(T)`|T|R|map, transform|
|`Consumer<T>`|`accept(T)`|T|void|forEach, side effects|
|`Supplier<T>`|`get()`|—|T|factory, lazy creation|
|`Comparator<T>`|`compare(T,T)`|T, T|int|sort|
|`UnaryOperator<T>`|`apply(T)`|T|T|replaceAll, same type transform|
|`BinaryOperator<T>`|`apply(T,T)`|T, T|T|reduce, merge|
|`BiFunction<T,U,R>`|`apply(T,U)`|T, U|R|two inputs transform|
|`BiPredicate<T,U>`|`test(T,U)`|T, U|boolean|two inputs condition|
|`BiConsumer<T,U>`|`accept(T,U)`|T, U|void|Map.forEach|

---

## 7. عمل Functional Interface بتاعك

```java
@FunctionalInterface
interface Transformer<T, R> {
    R transform(T input);
    
    // default methods مسموحة — مش بتأثر على الـ rule
    default Transformer<T, R> andLog() {
        return input -> {
            R result = this.transform(input);
            System.out.println("Transformed: " + input + " → " + result);
            return result;
        };
    }
}

// استخدام
Transformer<String, Integer> getLength = s -> s.length();
getLength.transform("Hello");  // 5

Transformer<String, Integer> withLog = getLength.andLog();
withLog.transform("Hello");
// Transformed: Hello → 5
```

---

## 8. مثال شامل — ربط كل حاجة مع بعض

```java
// البيانات
List<String> names = Arrays.asList(
    "Ahmed", "Sara", "Ali", "Mohamed", "Nour", ""
);

// Predicate — فلترة
Predicate<String> notEmpty  = s -> !s.isEmpty();
Predicate<String> longName  = s -> s.length() > 3;

// Function — تحويل
Function<String, String> toUpper    = String::toUpperCase;
Function<String, String> addGreeting = s -> "Hello, " + s + "!";

// Consumer — طباعة
Consumer<String> print = System.out::println;

// Comparator — ترتيب
Comparator<String> byLength = Comparator.comparingInt(String::length);

// Pipeline كاملة
names.stream()
    .filter(notEmpty)                    // ازل الفاضي
    .filter(longName)                    // أطول من 3 حروف
    .map(toUpper)                        // حول لـ uppercase
    .map(addGreeting)                    // أضف تحية
    .sorted(byLength)                    // رتب حسب الطول
    .forEach(print);                     // اطبع

// Output:
// Hello, SARA!
// Hello, NOUR!
// Hello, AHMED!
// Hello, MOHAMED!
```

---

_آخر تحديث: 2026 — Java 17+_