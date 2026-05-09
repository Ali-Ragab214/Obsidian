# Functional Programming في Java

> ملاحظة: الملف ده شامل كل حاجة عن Functional Programming في Java — المفاهيم، الكود، الأمثلة الحقيقية، وأسئلة الانترفيو.

---

## 1. إيه هو Functional Programming؟

**Functional Programming (FP)** هو أسلوب برمجة بتعامل مع البرنامج كمجموعة من **دوال رياضية نقية** — كل دالة بتاخد input وبترجع output من غير ما تأثر على أي حاجة بره منها.

### OOP vs FP — الفرق الجوهري

|الموضوع|OOP|Functional Programming|
|---|---|---|
|الوحدة الأساسية|Object / Class|Function|
|الحالة (State)|Objects بيخزنوا state|تتجنب state مشترك|
|التعديل|Mutable objects|Immutable data|
|الأسلوب|"إزاي تعمل" (How)|"إيه اللي تعمله" (What)|
|التكرار|loops (for, while)|recursion + higher-order functions|

### مثال يوضح الفرق

```java
// OOP style — بتغير state
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> evens = new ArrayList<>();
for (int n : numbers) {
    if (n % 2 == 0) evens.add(n);  // بتغير evens من بره
}

// Functional style — بدون تغيير state
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // بتولد list جديدة
```

---

## 2. المبادئ الأساسية

### 2.1 Pure Functions — الدوال النقية

دالة **نقية** هي دالة:

- بترجع نفس النتيجة لنفس الـ input **دايمًا**
- مش بتغير أي حاجة بره منها (No Side Effects)

```java
// Pure function ✅
public int add(int a, int b) {
    return a + b;  // دايمًا نفس النتيجة لنفس الـ input
}

// Impure function ❌
private int counter = 0;
public int addAndCount(int a, int b) {
    counter++;            // بتغير state خارجي!
    return a + b;
}

// Impure function ❌
public int getRandom(int a) {
    return a + new Random().nextInt();  // نتيجة مختلفة كل مرة!
}
```

**فايدة الـ Pure Functions:**

- سهلة الاختبار (unit testing)
- سهلة الـ debugging
- آمنة في multithreading
- قابلة للـ caching (memoization)

---

### 2.2 Immutability — عدم التغيير

بدل ما تعدل على object موجود، بتولد object جديد بالتعديل.

```java
// Mutable ❌ — بتعدل على الـ list الأصلية
public void addDiscount(List<Double> prices) {
    for (int i = 0; i < prices.size(); i++) {
        prices.set(i, prices.get(i) * 0.9);  // خطر!
    }
}

// Immutable ✅ — بتولد list جديدة
public List<Double> addDiscount(List<Double> prices) {
    return prices.stream()
        .map(p -> p * 0.9)
        .collect(Collectors.toList());  // الـ list الأصلية محدتش
}
```

```java
// Immutable object باستخدام record (Java 16+)
public record Point(int x, int y) {
    public Point move(int dx, int dy) {
        return new Point(x + dx, y + dy);  // بيرجع point جديدة
    }
}

Point p1 = new Point(1, 2);
Point p2 = p1.move(3, 4);  // p1 لسه (1,2) — ماتغيرتش
```

---

### 2.3 First-Class Functions

في Java، الـ functions بقت **first-class citizens** من Java 8 — يعني تقدر:

- تحط function في متغير
- تبعت function كـ parameter لـ function تانية
- ترجع function من function

```java
// حط function في متغير
Function<Integer, Integer> doubler = x -> x * 2;
Function<String, Integer> getLength = String::length;

// بعت function كـ parameter
public <T, R> List<R> transform(List<T> list, Function<T, R> fn) {
    return list.stream().map(fn).collect(Collectors.toList());
}

List<String> names = Arrays.asList("ahmed", "sara", "omar");
List<Integer> lengths = transform(names, String::length);
// [5, 4, 4]

// رجّع function من function (Function Factory)
public Function<Integer, Integer> multiplier(int factor) {
    return x -> x * factor;  // بترجع Lambda
}

Function<Integer, Integer> triple = multiplier(3);
triple.apply(5);  // 15
```

---

### 2.4 Higher-Order Functions

دالة بتاخد دالة كـ parameter أو بترجع دالة.

```java
// Higher-order function — بتاخد Predicate كـ parameter
public <T> List<T> filterList(List<T> list, Predicate<T> condition) {
    return list.stream()
        .filter(condition)
        .collect(Collectors.toList());
}

// استخدام
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
List<Integer> evens  = filterList(nums, n -> n % 2 == 0);
List<Integer> bigs   = filterList(nums, n -> n > 5);
List<Integer> primes = filterList(nums, n -> isPrime(n));
```

---

### 2.5 Function Composition

تركيب دالتين أو أكتر مع بعض عشان تعمل دالة جديدة.

```java
Function<Integer, Integer> times2  = x -> x * 2;
Function<Integer, Integer> plus10  = x -> x + 10;
Function<String,  Integer> strLen  = String::length;
Function<Integer, String>  intStr  = Object::toString;

// compose: بتعمل g(f(x)) — بتشغّل الـ function التانية الأول
Function<Integer, Integer> plus10ThenTimes2 = times2.compose(plus10);
plus10ThenTimes2.apply(5);  // (5+10)*2 = 30

// andThen: بتعمل f(g(x)) — بتشغّل الـ function الأولى الأول
Function<Integer, Integer> times2ThenPlus10 = times2.andThen(plus10);
times2ThenPlus10.apply(5);  // (5*2)+10 = 20

// سلسلة طويلة
Function<String, String> pipeline = strLen
    .andThen(times2)
    .andThen(plus10)
    .andThen(intStr);

pipeline.apply("Hello");  // len=5 → 10 → 20 → "20"
```

---

### 2.6 Closures

Lambda بتقدر "تحبس" variables من الـ scope اللي بره منها — بس لازم تكون **effectively final**.

```java
// Closure — Lambda بتحبس discount
public Function<Double, Double> makeDiscounter(double discount) {
    // discount محبوسة جوه الـ Lambda
    return price -> price * (1 - discount);
}

Function<Double, Double> tenOff    = makeDiscounter(0.10);
Function<Double, Double> twentyOff = makeDiscounter(0.20);

tenOff.apply(100.0);     // 90.0
twentyOff.apply(100.0);  // 80.0
```

```java
// effectively final — مينفعش تعدل المتغير بعد ما استخدمته في Lambda
int multiplier = 3;
Function<Integer, Integer> fn = x -> x * multiplier;
// multiplier = 4; // compile error!
```

---

## 3. الـ Functional Interfaces الأساسية

### الخمسة اللي لازم تحفظهم

```java
// 1. Predicate<T> — بياخد T ويرجع boolean
Predicate<String>  notEmpty   = s -> !s.isEmpty();
Predicate<Integer> isPositive = n -> n > 0;
Predicate<String>  isLong     = s -> s.length() > 10;

// تركيب Predicates
Predicate<String> notEmptyAndLong = notEmpty.and(isLong);
Predicate<String> emptyOrShort    = notEmpty.negate().or(isLong.negate());

// 2. Function<T, R> — بياخد T ويرجع R
Function<String,  Integer> length  = String::length;
Function<Integer, String>  toStr   = Object::toString;
Function<String,  String>  upper   = String::toUpperCase;

// 3. Consumer<T> — بياخد T ومش بيرجع حاجة
Consumer<String>  print  = System.out::println;
Consumer<List<?>> clear  = List::clear;

// Chaining consumers
Consumer<String> printAndLog = print.andThen(s -> log(s));

// 4. Supplier<T> — مش بياخد حاجة وبيرجع T
Supplier<String>      greeting = () -> "Hello, World!";
Supplier<List<String>> newList = ArrayList::new;
Supplier<LocalDate>   today    = LocalDate::now;

// 5. Comparator<T> — للترتيب
Comparator<String>  byLength  = Comparator.comparingInt(String::length);
Comparator<String>  reversed  = byLength.reversed();
Comparator<String>  thenAlpha = byLength.thenComparing(Comparator.naturalOrder());
```

### Bi-variants

```java
// BiFunction<T, U, R>
BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
repeat.apply("ha", 3);  // "hahaha"

// BiPredicate<T, U>
BiPredicate<String, Integer> longerThan = (s, n) -> s.length() > n;
longerThan.test("Hello", 3);  // true

// BiConsumer<T, U>
BiConsumer<String, Integer> printN = (s, n) -> System.out.println(s.repeat(n));
```

### UnaryOperator و BinaryOperator

```java
// UnaryOperator<T> — نفس الـ input والـ output type
UnaryOperator<String>  trim    = String::trim;
UnaryOperator<Integer> square  = x -> x * x;
UnaryOperator<List<Integer>> sort = list -> {
    List<Integer> copy = new ArrayList<>(list);
    Collections.sort(copy);
    return copy;
};

// BinaryOperator<T> — بياخد اتنين T ويرجع T
BinaryOperator<Integer> add  = Integer::sum;
BinaryOperator<String>  join = (a, b) -> a + " " + b;
BinaryOperator<Integer> max  = Integer::max;
```

---

## 4. Lambda Expressions

### الـ Syntax كامل

```java
// 1. بدون parameters
Runnable r = () -> System.out.println("Hello");

// 2. parameter واحد — الأقواس اختيارية
Consumer<String> c = s -> System.out.println(s);

// 3. أكتر من parameter
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// 4. body متعدد السطور
Function<Integer, String> classify = n -> {
    if (n < 0)  return "negative";
    if (n == 0) return "zero";
    return "positive";
};

// 5. مع type declaration صريح
Comparator<String> comp = (String a, String b) -> a.compareTo(b);
```

### Method References — الأنواع الأربعة

```java
// 1. Static method reference
Function<String, Integer>  parse  = Integer::parseInt;
Function<Double, Double>   abs    = Math::abs;

// 2. Instance method على الـ object نفسه
Function<String, String>   upper  = String::toUpperCase;
Function<String, Integer>  len    = String::length;
Predicate<String>          empty  = String::isEmpty;

// 3. Instance method على object خارجي
PrintStream out    = System.out;
Consumer<String>   print = out::println;

MyService service  = new MyService();
Runnable   task    = service::run;

// 4. Constructor reference
Supplier<ArrayList<String>>  newList = ArrayList::new;
Function<String, StringBuilder> sb  = StringBuilder::new;
BiFunction<String, Integer, char[]> charArr = char[]::new;
```

---

## 5. Stream API بالتفصيل

### إيه هو الـ Stream؟

الـ Stream مش data structure — هو **pipeline لمعالجة البيانات** بطريقة functional. بياخد source، بيطبق عليه عمليات متسلسلة، وبيطلع نتيجة.

```
Source → [intermediate ops] → [terminal op] → Result
```

### إنشاء Streams

```java
// من Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> s1 = list.stream();
Stream<String> s2 = list.parallelStream();

// من Array
int[]          arr = {1, 2, 3, 4, 5};
IntStream      s3  = Arrays.stream(arr);
Stream<String> s4  = Arrays.stream(new String[]{"x", "y"});

// Stream.of
Stream<Integer> s5 = Stream.of(1, 2, 3, 4, 5);

// Stream.iterate — infinite stream
Stream<Integer> evens = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, 6, ...

// Stream.generate — infinite stream
Stream<Double>  randoms = Stream.generate(Math::random);

// Range
IntStream oneToTen    = IntStream.rangeClosed(1, 10);  // 1..10
IntStream zeroToNine  = IntStream.range(0, 10);        // 0..9
```

---

### Intermediate Operations (Lazy)

```java
// filter — فلترة
list.stream().filter(s -> s.startsWith("A"))

// map — تحويل
list.stream().map(String::toUpperCase)
list.stream().map(s -> s.length())

// mapToInt / mapToDouble / mapToLong
list.stream().mapToInt(String::length)

// flatMap — فرد nested collections
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2), Arrays.asList(3, 4)
);
nested.stream().flatMap(Collection::stream);  // [1, 2, 3, 4]

// distinct
Stream.of(1,2,2,3,3,3).distinct();  // [1, 2, 3]

// sorted
list.stream().sorted()
list.stream().sorted(Comparator.reverseOrder())
list.stream().sorted(Comparator.comparingInt(String::length))

// peek — للـ debugging فقط
list.stream()
    .peek(s -> System.out.println("before: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("after: " + s))
    .collect(Collectors.toList())

// limit و skip
stream.limit(5)   // أول 5 عناصر
stream.skip(3)    // اتخطى أول 3

// takeWhile و dropWhile (Java 9+)
Stream.of(1,2,3,4,5,1,2)
    .takeWhile(n -> n < 4);  // [1, 2, 3]
Stream.of(1,2,3,4,5,1,2)
    .dropWhile(n -> n < 4);  // [4, 5, 1, 2]
```

---

### Terminal Operations

```java
// collect — الأهم
List<String>   toList    = stream.collect(Collectors.toList());
Set<String>    toSet     = stream.collect(Collectors.toSet());
String         joined    = stream.collect(Collectors.joining(", ", "[", "]"));
long           count     = stream.collect(Collectors.counting());
Double         avg       = stream.collect(Collectors.averagingInt(String::length));

// forEach
stream.forEach(System.out::println);
stream.forEachOrdered(System.out::println);  // بيحافظ على الترتيب في parallel

// count
long n = stream.count();

// findFirst / findAny
Optional<String> first = stream.findFirst();
Optional<String> any   = stream.findAny();  // أسرع في parallel

// min / max
Optional<Integer> min = stream.min(Comparator.naturalOrder());
Optional<Integer> max = stream.max(Integer::compareTo);

// anyMatch / allMatch / noneMatch
boolean hasNeg  = stream.anyMatch(n -> n < 0);
boolean allPos  = stream.allMatch(n -> n > 0);
boolean noneNeg = stream.noneMatch(n -> n < 0);

// reduce
int sum     = stream.reduce(0, Integer::sum);
int product = stream.reduce(1, (a, b) -> a * b);

Optional<Integer> max2 = stream.reduce(Integer::max);

// toArray
Object[]    arr1 = stream.toArray();
String[]    arr2 = stream.toArray(String[]::new);
```

---

### Collectors بالتفصيل

```java
// toMap
Map<String, Integer> nameLengths = names.stream()
    .collect(Collectors.toMap(
        s -> s,           // key
        String::length    // value
    ));

// toMap مع merge function (لو في duplicate keys)
Map<String, Long> wordCount = words.stream()
    .collect(Collectors.toMap(
        s -> s,           // key
        s -> 1L,          // value
        Long::sum         // merge: لو نفس الـ key اجمعهم
    ));

// groupingBy
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));

// groupingBy مع downstream collector
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

Map<String, Optional<Employee>> topEarnerByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary))
    ));

// partitioningBy
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 30_000));

List<Employee> highEarners = partitioned.get(true);
List<Employee> lowEarners  = partitioned.get(false);

// summarizingInt
IntSummaryStatistics stats = grades.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
// stats.getCount(), getSum(), getMin(), getMax(), getAverage()
```

---

## 6. Optional

### إيه هو Optional؟

بدل ما ترجع `null` وتخاطر بـ `NullPointerException`، بترجع `Optional<T>` — container ممكن يكون فيه قيمة أو فاضي.

```java
// إنشاء
Optional<String> present = Optional.of("Hello");          // لازم تكون مش null
Optional<String> nullable = Optional.ofNullable(getName()); // ممكن تكون null
Optional<String> empty    = Optional.empty();              // فاضي

// فحص
present.isPresent();   // true
empty.isEmpty();       // true (Java 11+)

// استخراج القيمة
String val1 = present.get();                    // throw لو فاضي
String val2 = present.orElse("default");        // قيمة افتراضية
String val3 = present.orElseGet(() -> compute()); // lazy default
String val4 = present.orElseThrow();             // throw NoSuchElementException لو فاضي
String val5 = present.orElseThrow(() -> new RuntimeException("Not found"));

// تحويل وفلترة
Optional<Integer> length = present.map(String::length);        // Optional<Integer>
Optional<String>  upper  = present.map(String::toUpperCase);
Optional<String>  long_  = present.filter(s -> s.length() > 3);

// flatMap — لو الـ map بترجع Optional
Optional<String> result = present.flatMap(s -> findInDB(s));

// ifPresent / ifPresentOrElse
present.ifPresent(System.out::println);
present.ifPresentOrElse(
    s -> System.out.println("Found: " + s),
    () -> System.out.println("Not found")
);
```

### أمثلة حقيقية

```java
// بدون Optional ❌
public String getCity(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            return address.getCity();
        }
    }
    return "Unknown";
}

// مع Optional ✅
public String getCity(User user) {
    return Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .orElse("Unknown");
}
```

```java
// في Repository pattern
public Optional<User> findById(int id) {
    return users.stream()
        .filter(u -> u.getId() == id)
        .findFirst();
}

// الاستخدام
userRepo.findById(42)
    .map(User::getName)
    .orElse("Guest");
```

---

## 7. أنماط Functional Programming في Java

### 7.1 Strategy Pattern — بـ Lambda

```java
// التقليدي — interface + classes منفصلة
interface SortStrategy { List<Integer> sort(List<Integer> list); }
class BubbleSort implements SortStrategy { ... }
class QuickSort  implements SortStrategy { ... }

// Functional — Lambda مباشرة
Function<List<Integer>, List<Integer>> bubbleSort = list -> { ... };
Function<List<Integer>, List<Integer>> quickSort  = list -> { ... };

// أو Comparator
Comparator<Employee> byName   = Comparator.comparing(Employee::getName);
Comparator<Employee> bySalary = Comparator.comparingDouble(Employee::getSalary);
Comparator<Employee> byDeptThenSalary = 
    Comparator.comparing(Employee::getDepartment)
              .thenComparingDouble(Employee::getSalary);
```

### 7.2 Builder Pattern — Functional style

```java
public class Query {
    private final Predicate<Employee>    filter;
    private final Comparator<Employee>   sorter;
    private final Function<Employee, ?> mapper;
    private final int limit;

    private Query(Builder b) {
        this.filter = b.filter;
        this.sorter = b.sorter;
        this.mapper = b.mapper;
        this.limit  = b.limit;
    }

    public List<?> execute(List<Employee> employees) {
        Stream<Employee> stream = employees.stream();
        if (filter != null) stream = stream.filter(filter);
        if (sorter != null) stream = stream.sorted(sorter);
        if (limit  > 0)     stream = stream.limit(limit);
        if (mapper != null) return stream.map(mapper).collect(Collectors.toList());
        return stream.collect(Collectors.toList());
    }

    public static class Builder {
        private Predicate<Employee>   filter;
        private Comparator<Employee>  sorter;
        private Function<Employee, ?> mapper;
        private int limit = 0;

        public Builder filter(Predicate<Employee> f) { this.filter = f; return this; }
        public Builder sortBy(Comparator<Employee> s) { this.sorter = s; return this; }
        public Builder map(Function<Employee, ?> m)   { this.mapper = m; return this; }
        public Builder limit(int n)                    { this.limit  = n; return this; }
        public Query build() { return new Query(this); }
    }
}

// الاستخدام
List<String> result = new Query.Builder()
    .filter(e -> e.getDepartment().equals("Engineering"))
    .sortBy(Comparator.comparingDouble(Employee::getSalary).reversed())
    .map(Employee::getName)
    .limit(5)
    .build()
    .execute(employees);
```

### 7.3 Pipeline Pattern

```java
// بناء pipeline من خطوات قابلة للتركيب
public class Pipeline<T> {
    private final Function<T, T> process;

    private Pipeline(Function<T, T> fn) {
        this.process = fn;
    }

    public static <T> Pipeline<T> of(Function<T, T> fn) {
        return new Pipeline<>(fn);
    }

    public Pipeline<T> pipe(Function<T, T> fn) {
        return new Pipeline<>(process.andThen(fn));
    }

    public T execute(T input) {
        return process.apply(input);
    }
}

// مثال — معالجة نص
Pipeline<String> textProcessor = Pipeline.of(String::trim)
    .pipe(String::toLowerCase)
    .pipe(s -> s.replaceAll("\\s+", " "))
    .pipe(s -> s.replaceAll("[^a-z0-9 ]", ""));

String clean = textProcessor.execute("  Hello, World!!! ");
// "hello world"
```

### 7.4 Memoization

```java
// Cache نتائج Pure Functions
public class Memoizer {
    public static <T, R> Function<T, R> memoize(Function<T, R> fn) {
        Map<T, R> cache = new ConcurrentHashMap<>();
        return input -> cache.computeIfAbsent(input, fn);
    }
}

// مثال
Function<Integer, Long> fibonacci = null;
Map<Integer, Long> cache = new HashMap<>();

// مع memoization
Function<Integer, Long> fib = Memoizer.memoize(n -> {
    if (n <= 1) return (long) n;
    return fib.apply(n - 1) + fib.apply(n - 2);
});
```

---

## 8. أمثلة حقيقية من مشاريع

### مثال 1 — نظام E-Commerce

```java
// Order processing pipeline
public class OrderService {

    // Filter + Sort + Map في سطور قليلة
    public List<OrderSummary> getTopPendingOrders(int limit) {
        return orderRepository.findAll().stream()
            .filter(o -> o.getStatus() == OrderStatus.PENDING)
            .filter(o -> o.getAmount() > 100)
            .sorted(Comparator.comparingDouble(Order::getAmount).reversed())
            .limit(limit)
            .map(o -> new OrderSummary(o.getId(), o.getCustomer(), o.getAmount()))
            .collect(Collectors.toList());
    }

    // Revenue by category
    public Map<String, Double> getRevenueByCategory() {
        return orderRepository.findAll().stream()
            .filter(o -> o.getStatus() == OrderStatus.DELIVERED)
            .collect(Collectors.groupingBy(
                Order::getCategory,
                Collectors.summingDouble(Order::getAmount)
            ));
    }

    // إيجاد أعلى order لكل عميل
    public Map<String, Optional<Order>> getTopOrderPerCustomer() {
        return orderRepository.findAll().stream()
            .collect(Collectors.groupingBy(
                Order::getCustomer,
                Collectors.maxBy(Comparator.comparingDouble(Order::getAmount))
            ));
    }

    // تقرير نصي للـ orders
    public String generateReport(List<Order> orders) {
        return orders.stream()
            .filter(o -> o.getStatus() == OrderStatus.PENDING)
            .map(o -> String.format("#%d (%s) → %.2f LE", o.getId(), o.getCustomer(), o.getAmount()))
            .collect(Collectors.joining("\n", "=== Pending Orders ===\n", "\n=== End ==="));
    }
}
```

### مثال 2 — نظام HR

```java
public class HRService {

    // تقرير إحصائي كامل
    public void printDepartmentReport(List<Employee> employees) {
        employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.summarizingDouble(Employee::getSalary)
            ))
            .forEach((dept, stats) ->
                System.out.printf("%s: count=%d, avg=%.0f, max=%.0f%n",
                    dept, (long)stats.getCount(), stats.getAverage(), stats.getMax())
            );
    }

    // ترقية الموظفين المستحقين
    public List<Employee> promoteEligible(List<Employee> employees) {
        Predicate<Employee> eligible = e ->
            e.getYearsExp() >= 5 &&
            e.getPerformance() >= 4.0 &&
            e.getLevel() != Level.SENIOR;

        return employees.stream()
            .filter(eligible)
            .map(e -> new Employee(
                e.getName(),
                e.getDepartment(),
                e.getSalary() * 1.20,       // زيادة 20%
                e.getYearsExp(),
                Level.values()[e.getLevel().ordinal() + 1]  // ترقية
            ))
            .collect(Collectors.toList());
    }

    // تقسيم الموظفين بحسب أهلية الترقية
    public Map<Boolean, List<String>> partitionByPromotion(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.partitioningBy(
                e -> e.getYearsExp() >= 5,
                Collectors.mapping(Employee::getName, Collectors.toList())
            ));
    }
}
```

### مثال 3 — Data Processing

```java
public class DataProcessor {

    // معالجة CSV
    public List<Product> parseCsv(List<String> lines) {
        return lines.stream()
            .skip(1)  // skip header
            .filter(line -> !line.isBlank())
            .map(line -> line.split(","))
            .filter(parts -> parts.length == 4)
            .map(parts -> new Product(
                parts[0].trim(),
                parts[1].trim(),
                Double.parseDouble(parts[2].trim()),
                Integer.parseInt(parts[3].trim())
            ))
            .collect(Collectors.toList());
    }

    // word frequency
    public Map<String, Long> wordFrequency(List<String> sentences) {
        return sentences.stream()
            .flatMap(s -> Arrays.stream(s.toLowerCase().split("\\s+")))
            .filter(w -> w.length() > 3)
            .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
    }

    // أكتر 10 كلمات تكرارًا
    public List<String> topWords(List<String> sentences, int n) {
        return wordFrequency(sentences).entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(n)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

---

## 9. Parallel Streams

```java
// Sequential — شغل على thread واحد
List<Integer> result = list.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Parallel — Java بتقسم الشغل على الـ cores تلقائياً
List<Integer> result = list.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// أو بـ parallel()
List<Integer> result = list.stream()
    .parallel()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// مثال واقعي — معالجة صور
List<String> processedImages = imagePaths.parallelStream()
    .map(path -> loadImage(path))      // بطيء — I/O
    .map(img -> resizeImage(img))      // بطيء — CPU
    .map(img -> applyFilter(img))
    .map(img -> saveImage(img))
    .collect(Collectors.toList());
```

### متى تستخدم Parallel وامتى لا؟

```java
// ✅ استخدم Parallel لما:
// — الـ data كبيرة جداً (100k+ عنصر)
// — الـ operations مكلفة CPU
// — مفيش shared mutable state

// ❌ متستخدمش Parallel لما:
// — الـ data صغيرة (overhead بيعدي الفايدة)
// — فيه I/O (database, files) — thread pool بيتعكس
// — فيه shared state — race conditions
// — الترتيب مهم — parallel مش بيضمن ترتيب

// مثال على الخطر — shared state ❌
List<Integer> results = new ArrayList<>();  // NOT thread-safe!
list.parallelStream()
    .forEach(results::add);  // race condition!

// الصح ✅
List<Integer> results = list.parallelStream()
    .collect(Collectors.toList());  // thread-safe collector
```

---

## 10. أسئلة الانترفيو

### المستوى الأول — الأساسيات

**س: إيه هو Functional Programming وإيه الفرق بينه وبين OOP؟**

> FP هو أسلوب برمجة بيعتمد على دوال نقية وبيتجنب الـ mutable state والـ side effects. في OOP بنفكر في Objects وحالتها، في FP بنفكر في تحويل البيانات من شكل لشكل. Java بتدعم الاتنين من Java 8.

---

**س: إيه هي الـ Pure Function؟ اديني مثال.**

> دالة بترجع نفس النتيجة لنفس الـ input دايمًا وما بتعملش أي side effect.

```java
// Pure ✅
public int multiply(int a, int b) { return a * b; }

// Impure ❌ — بتاثر على state خارجي
private List<String> log = new ArrayList<>();
public int add(int a, int b) {
    log.add(a + "+" + b);  // side effect!
    return a + b;
}
```

---

**س: إيه هو Functional Interface وإزاي بيشتغل مع Lambda؟**

> Interface فيه method واحدة abstract بس، بيتعلم بـ `@FunctionalInterface`. الـ Lambda بتولد implementation لـ الـ method دي في runtime.

```java
@FunctionalInterface
interface Transformer<T, R> {
    R transform(T input);
}

Transformer<String, Integer> getLen = s -> s.length();
getLen.transform("Hello");  // 5
```

---

**س: فرّق بين `map()` و `flatMap()`.**

```java
// map — بيحول كل عنصر لعنصر تاني (1:1)
Stream<Integer> lengths = names.stream().map(String::length);
// ["Hello", "Hi"] → [5, 2]

// flatMap — بيحول كل عنصر لـ stream وبيفردهم (1:many)
Stream<String> words = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")));
// ["Hello World", "Hi Java"] → ["Hello", "World", "Hi", "Java"]
```

---

**س: إيه هو Optional وليه بنستخدمه؟**

> Optional بيمثل قيمة ممكن تكون موجودة أو لأ، بيتجنب الـ `NullPointerException` وبيخلي الكود أوضح في نيته.

```java
// بدون Optional — ممكن NPE
String city = user.getAddress().getCity();

// مع Optional ✅
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

---

### المستوى الثاني — المتوسط

**س: فرّق بين Intermediate وTerminal operations في الـ Streams.**

> Intermediate operations (زي `filter`, `map`, `sorted`) بتكون **lazy** — مش بتشتغل لحد ما في terminal operation. Terminal operations (زي `collect`, `forEach`, `reduce`) بتشغّل الـ pipeline وبترجع نتيجة. الـ Stream بعد الـ terminal operation بيتقفل ومينفعش يتستخدم تاني.

```java
// الـ pipeline ده مش بيشتغل لحد ما collect()
Stream<String> stream = names.stream()
    .filter(s -> s.startsWith("A"))  // lazy
    .map(String::toUpperCase);        // lazy

// هنا بيشتغل كله
List<String> result = stream.collect(Collectors.toList());  // terminal
```

---

**س: إزاي بتعمل Function Composition في Java؟**

```java
Function<Integer, Integer> times2 = x -> x * 2;
Function<Integer, Integer> plus3  = x -> x + 3;

// andThen: times2 الأول، ثم plus3
Function<Integer, Integer> f1 = times2.andThen(plus3);
f1.apply(4);  // (4*2)+3 = 11

// compose: plus3 الأول، ثم times2
Function<Integer, Integer> f2 = times2.compose(plus3);
f2.apply(4);  // (4+3)*2 = 14
```

---

**س: إيه الفرق بين `findFirst()` و `findAny()`؟**

> في sequential stream، الاتنين بيرجعوا أول عنصر. في parallel stream، `findFirst()` بيرجع أول عنصر حسب الترتيب الأصلي، و`findAny()` بيرجع أي عنصر أسرع — لأنه مش محتاج يربط على الترتيب.

---

**س: إيه هي الـ Effectively Final وليه مهمة في Lambda؟**

> متغير effectively final هو متغير ما اتعدلش بعد ما اتعمل. الـ Lambda بتقدر تحبس variables من الـ enclosing scope بس لو كانت effectively final.

```java
int base = 10;
Function<Integer, Integer> addBase = x -> x + base;  // ✅

int count = 0;
Runnable r = () -> count++;  // ❌ compile error — count بتتعدل
```

---

**س: متى تستخدم `reduce()` بدل `collect()`؟**

> `reduce()` لما تحتاج قيمة واحدة من تجميع عناصر (sum, max, concat)، و`collect()` لما تحتاج container زي List أو Map.

```java
// reduce — ترجع قيمة واحدة
int sum = numbers.stream().reduce(0, Integer::sum);
Optional<String> longest = words.stream().reduce((a, b) -> a.length() >= b.length() ? a : b);

// collect — ترجع container
List<Integer> evens = numbers.stream().filter(n -> n%2==0).collect(Collectors.toList());
```

---

### المستوى التالت — المتقدم

**س: إيه الفرق بين `map()` و `peek()`؟**

> `map()` terminal operation بترجع stream جديد بقيم محولة. `peek()` intermediate operation بتنفذ side effect على كل عنصر (للـ debugging) من غير ما تغير القيم.

```java
// peek للـ debugging
List<String> result = list.stream()
    .peek(s -> System.out.println("before: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("after: " + s))
    .collect(Collectors.toList());
```

---

**س: إزاي بتتعامل مع Exceptions في Lambda؟**

```java
// المشكلة — checked exceptions مش مسموحة في Lambda مباشرة
list.stream()
    .map(s -> Integer.parseInt(s))  // ✅ unchecked
    .map(path -> Files.readString(path))  // ❌ IOException checked

// الحل — wrapper function
private static <T, R> Function<T, R> wrap(CheckedFunction<T, R> fn) {
    return input -> {
        try {
            return fn.apply(input);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

@FunctionalInterface
interface CheckedFunction<T, R> {
    R apply(T t) throws Exception;
}

// الاستخدام
list.stream()
    .map(wrap(path -> Files.readString(path)))  // ✅
    .collect(Collectors.toList());
```

---

**س: إيه هو Short-circuit في Streams؟**

> بعض operations بتوقف الـ processing لما تلاقي النتيجة بدون ما تشوف كل العناصر.

```java
// findFirst — بيوقف لما يلاقي أول عنصر
Optional<Integer> first = Stream.iterate(1, n -> n+1)
    .filter(n -> n > 1000)
    .findFirst();  // مش هيعدي أكتر من اللازم

// anyMatch — بيوقف لما يلاقي match
boolean found = list.stream()
    .anyMatch(s -> s.equals("target"));  // بيوقف فور ما يلاقيها

// limit — بيوقف بعد n عناصر
List<Integer> first10 = Stream.iterate(1, n -> n+1)
    .limit(10)
    .collect(Collectors.toList());  // [1..10] من غير ما يكمل infinity
```

---

**س: اكتب custom Collector.**

```java
// Collector بيجمع كل String في Set مرتبة
Collector<String, TreeSet<String>, TreeSet<String>> toSortedSet =
    Collector.of(
        TreeSet::new,           // supplier — ييجي الـ container
        TreeSet::add,           // accumulator — يضيف عنصر
        (s1, s2) -> { s1.addAll(s2); return s1; },  // combiner — للـ parallel
        Collector.Characteristics.IDENTITY_FINISH    // characteristics
    );

TreeSet<String> sorted = names.stream().collect(toSortedSet);
```

---

**س: إيه الفرق بين Predicate.and() وعمل filter مرتين؟**

```java
// مكافئين في النتيجة، بس الأول أوضح semantically
stream.filter(p1.and(p2));
stream.filter(p1).filter(p2);

// الفرق: Predicate composition بتسمح بـ reuse وتركيب معقد
Predicate<Employee> senior   = e -> e.getLevel() == Level.SENIOR;
Predicate<Employee> highPaid = e -> e.getSalary() > 50_000;
Predicate<Employee> inEng    = e -> e.getDept().equals("Engineering");

Predicate<Employee> topEng = senior.and(highPaid).and(inEng);
// ممكن يتستخدم في أماكن كتير
```

---

**س: ليه Streams بتدعم lazy evaluation وإيه فايدتها؟**

> Intermediate operations مش بتشتغل لحد ما في terminal operation. الفايدة:
> 
> - بتشغل الكود اللازم بس
> - بتدعم infinite streams
> - بتسمح للـ JVM يعمل optimizations

```java
// من غير lazy — كان هيعمل filter على كل الـ list الأول ثم map
// مع lazy — بيشوف كل عنصر مرة واحدة في الـ pipeline

// infinite stream — ممكن بسبب laziness
OptionalInt firstPrime = IntStream.iterate(2, n -> n + 1)
    .filter(HRService::isPrime)
    .findFirst();  // بيوقف بعد ما يلاقي أول prime
```

---

## 11. ملخص سريع — Cheat Sheet

### Functional Interfaces

|Interface|Method|استخدام|
|---|---|---|
|`Predicate<T>`|`boolean test(T)`|filter, conditions|
|`Function<T,R>`|`R apply(T)`|map, transform|
|`Consumer<T>`|`void accept(T)`|forEach, side effects|
|`Supplier<T>`|`T get()`|lazy creation|
|`Comparator<T>`|`int compare(T,T)`|sorting|
|`UnaryOperator<T>`|`T apply(T)`|transform same type|
|`BinaryOperator<T>`|`T apply(T,T)`|reduce, merge|

### Stream Operations

|Operation|نوع|ترجع|
|---|---|---|
|`filter(Predicate)`|Intermediate|`Stream<T>`|
|`map(Function)`|Intermediate|`Stream<R>`|
|`flatMap(Function)`|Intermediate|`Stream<R>`|
|`sorted(Comparator)`|Intermediate|`Stream<T>`|
|`distinct()`|Intermediate|`Stream<T>`|
|`limit(n)`|Intermediate|`Stream<T>`|
|`peek(Consumer)`|Intermediate|`Stream<T>`|
|`collect(Collector)`|Terminal|any|
|`forEach(Consumer)`|Terminal|void|
|`reduce(BinaryOp)`|Terminal|`Optional<T>`|
|`count()`|Terminal|long|
|`findFirst()`|Terminal|`Optional<T>`|
|`anyMatch(Predicate)`|Terminal|boolean|
|`min/max(Comparator)`|Terminal|`Optional<T>`|

### Collectors الأهم

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap(keyFn, valueFn)
Collectors.joining(delimiter, prefix, suffix)
Collectors.groupingBy(classifier)
Collectors.groupingBy(classifier, downstream)
Collectors.partitioningBy(predicate)
Collectors.counting()
Collectors.summingInt/Double/Long(fn)
Collectors.averagingInt/Double/Long(fn)
Collectors.summarizingInt/Double/Long(fn)
```

---

_آخر تحديث: 2026 — Java 17+_