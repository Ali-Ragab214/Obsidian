# Java Collections & Streams — الشرح الكامل المفصل

---

# 📦 CH09 - Java Collections

---

## 1️⃣ Collections Introduction

### ما هي الـ Collections؟

**Collections** هي مجموعة من الـ classes والـ interfaces بتساعدك تخزن وتتعامل مع مجموعات من البيانات.

### الأنواع الأساسية

```
Collection
    ├── List (ترتيب + تكرار)
    │   ├── ArrayList
    │   ├── LinkedList
    │   └── Vector
    │
    ├── Set (لا تكرار)
    │   ├── HashSet
    │   ├── LinkedHashSet
    │   └── TreeSet
    │
    └── Queue (FIFO)
        ├── PriorityQueue
        └── ArrayDeque

Map (Key-Value)
    ├── HashMap
    ├── LinkedHashMap
    └── TreeMap
```

---

## 2️⃣ Collection and Iterator Interfaces

### Collection Interface

```java
public interface Collection<E> {
    boolean add(E e);
    boolean remove(Object o);
    boolean contains(Object o);
    int size();
    boolean isEmpty();
    void clear();
    Iterator<E> iterator();
}
```

---

## 3️⃣ Iterator Interface Demo

### ما هو Iterator؟

طريقة للمرور على عناصر الـ Collection واحد واحد.

```java
import java.util.*;

public class IteratorDemo {
    public static void main(String[] args) {
        
        List<String> names = new ArrayList<>();
        names.add("Ahmed");
        names.add("Ali");
        names.add("Omar");
        names.add("Sara");
        
        // ─── الطريقة 1: Iterator ─────────────────────
        Iterator<String> iterator = names.iterator();
        
        while (iterator.hasNext()) {
            String name = iterator.next();
            System.out.println(name);
            
            // حذف أثناء التكرار
            if (name.equals("Ali")) {
                iterator.remove(); // ✅ آمن
            }
        }
        
        System.out.println("After removal: " + names);
        // [Ahmed, Omar, Sara]
        
        // ─── الطريقة 2: Enhanced For Loop ─────────────
        for (String name : names) {
            System.out.println(name);
            // names.remove(name); ❌ خطأ - ConcurrentModificationException
        }
        
        // ─── الطريقة 3: forEach (Java 8+) ─────────────
        names.forEach(name -> System.out.println(name));
    }
}
```

---

## 4️⃣ Generic Utility Methods

### Collections Class

```java
import java.util.*;

public class UtilityMethodsDemo {
    public static void main(String[] args) {
        
        List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9, 3));
        
        // ─── Sort ─────────────────────────────────────
        Collections.sort(numbers);
        System.out.println("Sorted: " + numbers);
        // [1, 2, 3, 5, 8, 9]
        
        // ─── Reverse ──────────────────────────────────
        Collections.reverse(numbers);
        System.out.println("Reversed: " + numbers);
        // [9, 8, 5, 3, 2, 1]
        
        // ─── Shuffle ──────────────────────────────────
        Collections.shuffle(numbers);
        System.out.println("Shuffled: " + numbers);
        // [3, 9, 1, 5, 8, 2] (عشوائي)
        
        // ─── Max & Min ────────────────────────────────
        int max = Collections.max(numbers);
        int min = Collections.min(numbers);
        System.out.println("Max: " + max + ", Min: " + min);
        
        // ─── Frequency ────────────────────────────────
        List<String> words = Arrays.asList("apple", "banana", "apple", "cherry", "apple");
        int count = Collections.frequency(words, "apple");
        System.out.println("apple appears " + count + " times"); // 3
        
        // ─── Binary Search ────────────────────────────
        Collections.sort(numbers);
        int index = Collections.binarySearch(numbers, 5);
        System.out.println("5 is at index: " + index);
        
        // ─── Fill ─────────────────────────────────────
        Collections.fill(numbers, 0);
        System.out.println("Filled: " + numbers);
        // [0, 0, 0, 0, 0, 0]
        
        // ─── Copy ─────────────────────────────────────
        List<Integer> source = Arrays.asList(1, 2, 3);
        List<Integer> dest = new ArrayList<>(Arrays.asList(0, 0, 0));
        Collections.copy(dest, source);
        System.out.println("Copied: " + dest); // [1, 2, 3]
        
        // ─── Swap ─────────────────────────────────────
        List<String> names = new ArrayList<>(Arrays.asList("A", "B", "C"));
        Collections.swap(names, 0, 2);
        System.out.println("Swapped: " + names); // [C, B, A]
    }
}
```

---

## 5️⃣ Collections Framework Interfaces

```java
// ─── List Interface ───────────────────────────────
List<String> list = new ArrayList<>();
list.add("Ahmed");
list.add(0, "First"); // إضافة في index معين
list.get(0); // قراءة
list.set(0, "Updated"); // تعديل
list.remove(0); // حذف

// ─── Set Interface ────────────────────────────────
Set<String> set = new HashSet<>();
set.add("Ahmed");
set.add("Ahmed"); // مش هيتضاف (no duplicates)
set.contains("Ahmed"); // true

// ─── Queue Interface ──────────────────────────────
Queue<String> queue = new LinkedList<>();
queue.offer("First");  // إضافة
queue.peek();          // شوف الأول بدون حذف
queue.poll();          // شوف واحذف الأول

// ─── Map Interface ────────────────────────────────
Map<String, Integer> map = new HashMap<>();
map.put("Ahmed", 22);
map.get("Ahmed"); // 22
map.containsKey("Ahmed"); // true
```

---

## 6️⃣ Collections Concrete Classes

### ArrayList vs LinkedList

```java
import java.util.*;

public class ListComparison {
    public static void main(String[] args) {
        
        // ─── ArrayList ────────────────────────────────
        // سريع في القراءة، بطيء في الإضافة/الحذف من النص
        List<String> arrayList = new ArrayList<>();
        arrayList.add("A");
        arrayList.add("B");
        arrayList.add("C");
        arrayList.get(1); // ✅ سريع جداً O(1)
        arrayList.add(1, "X"); // ⚠️ بطيء O(n)
        
        // ─── LinkedList ───────────────────────────────
        // سريع في الإضافة/الحذف، بطيء في القراءة
        List<String> linkedList = new LinkedList<>();
        linkedList.add("A");
        linkedList.add("B");
        linkedList.add("C");
        linkedList.get(1); // ⚠️ بطيء O(n)
        linkedList.add(1, "X"); // ✅ سريع O(1)
    }
}
```

### HashSet vs TreeSet vs LinkedHashSet

```java
import java.util.*;

public class SetComparison {
    public static void main(String[] args) {
        
        // ─── HashSet ──────────────────────────────────
        // لا ترتيب، الأسرع
        Set<Integer> hashSet = new HashSet<>();
        hashSet.add(5);
        hashSet.add(2);
        hashSet.add(8);
        System.out.println("HashSet: " + hashSet);
        // [2, 5, 8] أو أي ترتيب عشوائي
        
        // ─── TreeSet ──────────────────────────────────
        // مرتب تلقائياً، أبطأ شوية
        Set<Integer> treeSet = new TreeSet<>();
        treeSet.add(5);
        treeSet.add(2);
        treeSet.add(8);
        System.out.println("TreeSet: " + treeSet);
        // [2, 5, 8] دايماً مرتب
        
        // ─── LinkedHashSet ────────────────────────────
        // بترتيب الإضافة
        Set<Integer> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add(5);
        linkedHashSet.add(2);
        linkedHashSet.add(8);
        System.out.println("LinkedHashSet: " + linkedHashSet);
        // [5, 2, 8] نفس ترتيب الإضافة
    }
}
```

---

## 7️⃣ Linked Lists Example

```java
import java.util.*;

public class LinkedListExample {
    public static void main(String[] args) {
        
        LinkedList<String> list = new LinkedList<>();
        
        // ─── إضافة ───────────────────────────────────
        list.add("Ahmed");
        list.addFirst("First");    // في الأول
        list.addLast("Last");      // في الآخر
        list.add(2, "Middle");     // في مكان معين
        
        System.out.println(list);
        // [First, Ahmed, Middle, Last]
        
        // ─── قراءة ───────────────────────────────────
        System.out.println("First: " + list.getFirst());
        System.out.println("Last: " + list.getLast());
        System.out.println("At index 2: " + list.get(2));
        
        // ─── حذف ─────────────────────────────────────
        list.removeFirst();
        list.removeLast();
        list.remove(1);
        System.out.println("After removal: " + list);
        
        // ─── استخدام كـ Queue ─────────────────────────
        LinkedList<String> queue = new LinkedList<>();
        queue.offer("First");
        queue.offer("Second");
        queue.offer("Third");
        
        System.out.println("Peek: " + queue.peek()); // First
        System.out.println("Poll: " + queue.poll()); // First (ويحذفه)
        System.out.println("Queue: " + queue); // [Second, Third]
        
        // ─── استخدام كـ Stack ─────────────────────────
        LinkedList<String> stack = new LinkedList<>();
        stack.push("Bottom");
        stack.push("Middle");
        stack.push("Top");
        
        System.out.println("Pop: " + stack.pop()); // Top
        System.out.println("Stack: " + stack); // [Middle, Bottom]
    }
}
```

---

## 8️⃣ Maps

### Map Demo

```java
import java.util.*;

public class MapDemo {
    public static void main(String[] args) {
        
        // ─── HashMap ──────────────────────────────────
        Map<String, Integer> ages = new HashMap<>();
        
        // إضافة
        ages.put("Ahmed", 22);
        ages.put("Ali", 25);
        ages.put("Sara", 20);
        ages.put("Omar", 23);
        
        // قراءة
        System.out.println("Ahmed's age: " + ages.get("Ahmed")); // 22
        
        // تحديث
        ages.put("Ahmed", 23); // بيستبدل القيمة القديمة
        
        // حذف
        ages.remove("Omar");
        
        // التحقق
        System.out.println("Contains Ali? " + ages.containsKey("Ali")); // true
        System.out.println("Contains age 25? " + ages.containsValue(25)); // true
        
        // حجم
        System.out.println("Size: " + ages.size()); // 3
        
        // ─── التكرار على Map ──────────────────────────
        
        // طريقة 1: keySet
        for (String name : ages.keySet()) {
            System.out.println(name + " is " + ages.get(name));
        }
        
        // طريقة 2: values
        for (Integer age : ages.values()) {
            System.out.println("Age: " + age);
        }
        
        // طريقة 3: entrySet (الأفضل)
        for (Map.Entry<String, Integer> entry : ages.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
        
        // طريقة 4: forEach (Java 8+)
        ages.forEach((name, age) -> {
            System.out.println(name + " is " + age + " years old");
        });
    }
}
```

---

## 9️⃣ Map Interface Methods

```java
import java.util.*;

public class MapMethods {
    public static void main(String[] args) {
        
        Map<String, Integer> scores = new HashMap<>();
        
        // ─── put & putIfAbsent ────────────────────────
        scores.put("Ahmed", 90);
        scores.put("Ahmed", 95); // بيستبدل
        scores.putIfAbsent("Ali", 80); // بيضيف لو مش موجود
        scores.putIfAbsent("Ahmed", 100); // مش هيضيف لأن موجود
        
        // ─── getOrDefault ─────────────────────────────
        int score = scores.getOrDefault("Omar", 0);
        System.out.println("Omar's score: " + score); // 0
        
        // ─── compute ──────────────────────────────────
        scores.compute("Ahmed", (key, value) -> value + 5);
        System.out.println("Ahmed after +5: " + scores.get("Ahmed")); // 100
        
        // ─── computeIfAbsent ──────────────────────────
        scores.computeIfAbsent("Sara", k -> 85);
        
        // ─── computeIfPresent ─────────────────────────
        scores.computeIfPresent("Ali", (k, v) -> v + 10);
        
        // ─── merge ────────────────────────────────────
        scores.merge("Ahmed", 10, (oldVal, newVal) -> oldVal + newVal);
        // لو موجود بيدمجهم، لو مش موجود بيضيف القيمة الجديدة
        
        // ─── replace ──────────────────────────────────
        scores.replace("Ali", 90);
        scores.replace("Ali", 90, 95); // لو القيمة 90 استبدلها بـ 95
        
        System.out.println("Final scores: " + scores);
    }
}
```

---

## 🔟 Algorithms

```java
import java.util.*;

public class AlgorithmsDemo {
    public static void main(String[] args) {
        
        List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9, 3, 7, 4, 6));
        
        // ─── Sorting ──────────────────────────────────
        Collections.sort(numbers);
        System.out.println("Sorted: " + numbers);
        
        // ترتيب عكسي
        Collections.sort(numbers, Collections.reverseOrder());
        System.out.println("Reverse sorted: " + numbers);
        
        // ─── Searching ────────────────────────────────
        Collections.sort(numbers); // لازم يكون مرتب
        int index = Collections.binarySearch(numbers, 5);
        System.out.println("5 found at index: " + index);
        
        // ─── Finding Max/Min ──────────────────────────
        int max = Collections.max(numbers);
        int min = Collections.min(numbers);
        System.out.println("Max: " + max + ", Min: " + min);
        
        // ─── Rotating ─────────────────────────────────
        Collections.rotate(numbers, 2);
        System.out.println("Rotated: " + numbers);
        // [8, 9, 1, 2, 3, 4, 5, 6, 7]
        
        // ─── Disjoint ─────────────────────────────────
        List<Integer> list1 = Arrays.asList(1, 2, 3);
        List<Integer> list2 = Arrays.asList(4, 5, 6);
        boolean noCommon = Collections.disjoint(list1, list2);
        System.out.println("No common elements? " + noCommon); // true
        
        // ─── AddAll ───────────────────────────────────
        List<String> names = new ArrayList<>();
        Collections.addAll(names, "Ahmed", "Ali", "Omar");
        System.out.println("Names: " + names);
    }
}
```

---

# 🌊 CH10 - Java Stream API

---

## 1️⃣ Stream API Introduction

```java
import java.util.*;
import java.util.stream.*;

public class StreamIntro {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // ❌ الطريقة القديمة
        List<Integer> evenNumbers = new ArrayList<>();
        for (int num : numbers) {
            if (num % 2 == 0) {
                evenNumbers.add(num * num);
            }
        }
        System.out.println("Old way: " + evenNumbers);
        
        // ✅ الطريقة الحديثة - Stream
        List<Integer> result = numbers.stream()
            .filter(n -> n % 2 == 0)        // فلتر الزوجية
            .map(n -> n * n)                // مربع كل رقم
            .collect(Collectors.toList());
        
        System.out.println("Stream way: " + result);
        // [4, 16, 36, 64, 100]
    }
}
```

---

## 2️⃣ Stream Operations

### Intermediate Operations

```java
import java.util.*;
import java.util.stream.*;

public class IntermediateOps {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Ahmed", "Ali", "Sara", "Omar", "Zainab", "Ali");
        
        // ─── filter ───────────────────────────────────
        List<String> longNames = names.stream()
            .filter(name -> name.length() > 4)
            .collect(Collectors.toList());
        System.out.println("Long names: " + longNames);
        
        // ─── map ──────────────────────────────────────
        List<String> upperNames = names.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());
        System.out.println("Upper: " + upperNames);
        
        // ─── distinct ─────────────────────────────────
        List<String> uniqueNames = names.stream()
            .distinct()
            .collect(Collectors.toList());
        System.out.println("Unique: " + uniqueNames);
        
        // ─── sorted ───────────────────────────────────
        List<String> sorted = names.stream()
            .sorted()
            .collect(Collectors.toList());
        System.out.println("Sorted: " + sorted);
        
        // ─── limit ────────────────────────────────────
        List<String> first3 = names.stream()
            .limit(3)
            .collect(Collectors.toList());
        System.out.println("First 3: " + first3);
        
        // ─── skip ─────────────────────────────────────
        List<String> afterFirst3 = names.stream()
            .skip(3)
            .collect(Collectors.toList());
        System.out.println("After first 3: " + afterFirst3);
        
        // ─── peek ─────────────────────────────────────
        List<String> result = names.stream()
            .peek(name -> System.out.println("Processing: " + name))
            .filter(name -> name.startsWith("A"))
            .collect(Collectors.toList());
    }
}
```

---

### Terminal Operations

```java
import java.util.*;
import java.util.stream.*;

public class TerminalOps {
    public static void main(String[] args) {
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // ─── forEach ──────────────────────────────────
        numbers.stream()
            .filter(n -> n % 2 == 0)
            .forEach(n -> System.out.println(n));
        
        // ─── collect ──────────────────────────────────
        List<Integer> list = numbers.stream()
            .filter(n -> n > 5)
            .collect(Collectors.toList());
        
        Set<Integer> set = numbers.stream()
            .collect(Collectors.toSet());
        
        // ─── count ────────────────────────────────────
        long count = numbers.stream()
            .filter(n -> n > 5)
            .count();
        System.out.println("Count: " + count); // 5
        
        // ─── reduce ───────────────────────────────────
        int sum = numbers.stream()
            .reduce(0, (a, b) -> a + b);
        System.out.println("Sum: " + sum); // 55
        
        int product = numbers.stream()
            .reduce(1, (a, b) -> a * b);
        System.out.println("Product: " + product);
        
        // ─── min & max ────────────────────────────────
        Optional<Integer> min = numbers.stream()
            .min(Integer::compare);
        System.out.println("Min: " + min.get()); // 1
        
        Optional<Integer> max = numbers.stream()
            .max(Integer::compare);
        System.out.println("Max: " + max.get()); // 10
        
        // ─── anyMatch, allMatch, noneMatch ────────────
        boolean hasEven = numbers.stream()
            .anyMatch(n -> n % 2 == 0);
        System.out.println("Has even? " + hasEven); // true
        
        boolean allPositive = numbers.stream()
            .allMatch(n -> n > 0);
        System.out.println("All positive? " + allPositive); // true
        
        boolean noNegative = numbers.stream()
            .noneMatch(n -> n < 0);
        System.out.println("No negative? " + noNegative); // true
        
        // ─── findFirst & findAny ──────────────────────
        Optional<Integer> first = numbers.stream()
            .filter(n -> n > 5)
            .findFirst();
        System.out.println("First > 5: " + first.get()); // 6
        
        Optional<Integer> any = numbers.stream()
            .filter(n -> n > 5)
            .findAny();
        System.out.println("Any > 5: " + any.get());
    }
}
```

---

## 3️⃣ Stream API Demo

```java
import java.util.*;
import java.util.stream.*;

class Product {
    String name;
    double price;
    String category;
    
    Product(String name, double price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
    }
    
    // Getters
    String getName() { return name; }
    double getPrice() { return price; }
    String getCategory() { return category; }
    
    @Override
    public String toString() {
        return name + " - $" + price + " (" + category + ")";
    }
}

public class StreamDemo {
    public static void main(String[] args) {
        
        List<Product> products = Arrays.asList(
            new Product("Laptop", 1200, "Electronics"),
            new Product("Phone", 800, "Electronics"),
            new Product("Shirt", 30, "Clothing"),
            new Product("Shoes", 80, "Clothing"),
            new Product("Watch", 200, "Accessories"),
            new Product("Headphones", 150, "Electronics")
        );
        
        // ─── 1. Electronics أرخص من 1000 ─────────────
        List<Product> cheapElectronics = products.stream()
            .filter(p -> p.getCategory().equals("Electronics"))
            .filter(p -> p.getPrice() < 1000)
            .collect(Collectors.toList());
        
        System.out.println("Cheap Electronics:");
        cheapElectronics.forEach(System.out::println);
        
        // ─── 2. أسماء المنتجات بس ──────────────────────
        List<String> names = products.stream()
            .map(Product::getName)
            .collect(Collectors.toList());
        System.out.println("\nProduct names: " + names);
        
        // ─── 3. مجموع الأسعار ──────────────────────────
        double totalPrice = products.stream()
            .mapToDouble(Product::getPrice)
            .sum();
        System.out.println("\nTotal price: $" + totalPrice);
        
        // ─── 4. متوسط السعر ────────────────────────────
        double avgPrice = products.stream()
            .mapToDouble(Product::getPrice)
            .average()
            .orElse(0);
        System.out.println("Average price: $" + avgPrice);
        
        // ─── 5. أغلى منتج ──────────────────────────────
        Optional<Product> mostExpensive = products.stream()
            .max(Comparator.comparing(Product::getPrice));
        System.out.println("\nMost expensive: " + mostExpensive.get());
        
        // ─── 6. Group by category ──────────────────────
        Map<String, List<Product>> byCategory = products.stream()
            .collect(Collectors.groupingBy(Product::getCategory));
        
        System.out.println("\nGrouped by category:");
        byCategory.forEach((category, prods) -> {
            System.out.println(category + ": " + prods.size() + " products");
        });
        
        // ─── 7. Count by category ──────────────────────
        Map<String, Long> countByCategory = products.stream()
            .collect(Collectors.groupingBy(
                Product::getCategory,
                Collectors.counting()
            ));
        System.out.println("\nCount by category: " + countByCategory);
        
        // ─── 8. أرخص 3 منتجات ─────────────────────────
        List<Product> top3Cheapest = products.stream()
            .sorted(Comparator.comparing(Product::getPrice))
            .limit(3)
            .collect(Collectors.toList());
        
        System.out.println("\nTop 3 cheapest:");
        top3Cheapest.forEach(System.out::println);
    }
}
```

---

## 4️⃣ Advanced Collect

```java
import java.util.*;
import java.util.stream.*;

public class AdvancedCollect {
    public static void main(String[] args) {
        
        List<String> names = Arrays.asList("Ahmed", "Ali", "Sara", "Omar", "Zainab", "Ahmed");
        
        // ─── toList ───────────────────────────────────
        List<String> list = names.stream()
            .collect(Collectors.toList());
        
        // ─── toSet ────────────────────────────────────
        Set<String> set = names.stream()
            .collect(Collectors.toSet());
        
        // ─── toMap ────────────────────────────────────
        Map<String, Integer> nameLength = names.stream()
            .distinct()
            .collect(Collectors.toMap(
                name -> name,          // key
                name -> name.length()  // value
            ));
        System.out.println("Name lengths: " + nameLength);
        
        // ─── joining ──────────────────────────────────
        String joined = names.stream()
            .collect(Collectors.joining(", "));
        System.out.println("Joined: " + joined);
        // Ahmed, Ali, Sara, Omar, Zainab, Ahmed
        
        String withPrefix = names.stream()
            .collect(Collectors.joining(", ", "[", "]"));
        System.out.println("With brackets: " + withPrefix);
        // [Ahmed, Ali, Sara, Omar, Zainab, Ahmed]
        
        // ─── counting ─────────────────────────────────
        long count = names.stream()
            .collect(Collectors.counting());
        
        // ─── summingInt ───────────────────────────────
        int totalLength = names.stream()
            .collect(Collectors.summingInt(String::length));
        System.out.println("Total length: " + totalLength);
        
        // ─── averagingInt ─────────────────────────────
        double avgLength = names.stream()
            .collect(Collectors.averagingInt(String::length));
        System.out.println("Average length: " + avgLength);
        
        // ─── summarizingInt ───────────────────────────
        IntSummaryStatistics stats = names.stream()
            .collect(Collectors.summarizingInt(String::length));
        System.out.println("Stats: " + stats);
        // IntSummaryStatistics{count=6, sum=30, min=3, average=5.0, max=6}
        
        // ─── groupingBy ───────────────────────────────
        Map<Integer, List<String>> byLength = names.stream()
            .collect(Collectors.groupingBy(String::length));
        System.out.println("Grouped by length: " + byLength);
        // {3=[Ali], 4=[Sara, Omar], 5=[Ahmed, Ahmed], 6=[Zainab]}
        
        // ─── partitioningBy ───────────────────────────
        Map<Boolean, List<String>> partitioned = names.stream()
            .collect(Collectors.partitioningBy(name -> name.length() > 4));
        System.out.println("Long names: " + partitioned.get(true));
        System.out.println("Short names: " + partitioned.get(false));
    }
}
```

---

## 5️⃣ Parallel Streams

```java
import java.util.*;
import java.util.stream.*;

public class ParallelStreamDemo {
    public static void main(String[] args) {
        
        List<Integer> numbers = new ArrayList<>();
        for (int i = 1; i <= 1000000; i++) {
            numbers.add(i);
        }
        
        // ─── Sequential Stream ────────────────────────
        long startSeq = System.currentTimeMillis();
        
        long sumSeq = numbers.stream()
            .mapToLong(n -> n)
            .sum();
        
        long endSeq = System.currentTimeMillis();
        System.out.println("Sequential: " + (endSeq - startSeq) + "ms");
        System.out.println("Sum: " + sumSeq);
        
        // ─── Parallel Stream ──────────────────────────
        long startPar = System.currentTimeMillis();
        
        long sumPar = numbers.parallelStream()
            .mapToLong(n -> n)
            .sum();
        
        long endPar = System.currentTimeMillis();
        System.out.println("Parallel: " + (endPar - startPar) + "ms");
        System.out.println("Sum: " + sumPar);
        
        // ─── متى تستخدم Parallel ──────────────────────
        
        // ✅ كويس لما:
        // - البيانات كتيرة جداً (آلاف أو ملايين)
        // - العملية معقدة وبتاخد وقت
        // - مفيش تعديل على shared state
        
        // ❌ متستخدموش لما:
        // - البيانات قليلة
        // - العملية بسيطة
        // - فيه shared state
    }
}
```

---

## 📊 مقارنة الصعوبة والاستخدام

|الموضوع|الصعوبة|متى تستخدمه|الأهمية|
|---|---|---|---|
|**ArrayList**|⭐ سهل|القراءة كتير|⭐⭐⭐⭐⭐|
|**LinkedList**|⭐⭐ متوسط|الإضافة/الحذف كتير|⭐⭐⭐|
|**HashSet**|⭐ سهل|لا تكرار|⭐⭐⭐⭐|
|**TreeSet**|⭐⭐ متوسط|لا تكرار + ترتيب|⭐⭐⭐|
|**HashMap**|⭐⭐ متوسط|Key-Value|⭐⭐⭐⭐⭐|
|**TreeMap**|⭐⭐⭐ صعب|Key-Value مرتب|⭐⭐|
|**Iterator**|⭐⭐ متوسط|حذف أثناء loop|⭐⭐⭐|
|**Streams**|⭐⭐⭐ صعب|فلترة/تحويل|⭐⭐⭐⭐⭐|
|**Parallel Streams**|⭐⭐⭐⭐ صعب جداً|بيانات ضخمة|⭐⭐|

---

## 🎯 أيهم أفضل؟

### للمبتدئين — ابدأ بدول:

1. ✅ **ArrayList** — الأسهل والأشيع
2. ✅ **HashMap** — مهم جداً
3. ✅ **HashSet** — بسيط ومفيد
4. ✅ **Streams (filter, map, collect)** — حديث ومهم

### للمتقدمين:

5. **LinkedList** — لما تحتاج Queue/Stack
6. **TreeSet/TreeMap** — لما تحتاج ترتيب
7. **Advanced Streams** — groupingBy, reducing
8. **Parallel Streams** — للأداء العالي

---

## 📝 مثال شامل — كل حاجة مع بعض

```java
import java.util.*;
import java.util.stream.*;

class Student {
    String name;
    int age;
    String major;
    double gpa;
    
    Student(String name, int age, String major, double gpa) {
        this.name = name;
        this.age = age;
        this.major = major;
        this.gpa = gpa;
    }
    
    // Getters
    String getName() { return name; }
    int getAge() { return age; }
    String getMajor() { return major; }
    double getGpa() { return gpa; }
    
    @Override
    public String toString() {
        return name + " (" + major + ", GPA: " + gpa + ")";
    }
}

public class ComprehensiveExample {
    public static void main(String[] args) {
        
        // ─── Data ─────────────────────────────────────
        List<Student> students = Arrays.asList(
            new Student("Ahmed", 22, "CS", 3.8),
            new Student("Ali", 20, "Math", 3.5),
            new Student("Sara", 21, "CS", 3.9),
            new Student("Omar", 23, "Physics", 3.2),
            new Student("Zainab", 22, "CS", 3.7),
            new Student("Fatima", 20, "Math", 3.6),
            new Student("Hassan", 24, "Physics", 3.4)
        );
        
        // ─── 1. CS طلاب الـ GPA > 3.7 ─────────────────
        System.out.println("=== Top CS Students ===");
        students.stream()
            .filter(s -> s.getMajor().equals("CS"))
            .filter(s -> s.getGpa() > 3.7)
            .forEach(System.out::println);
        
        // ─── 2. متوسط GPA لكل تخصص ────────────────────
        System.out.println("\n=== Average GPA by Major ===");
        Map<String, Double> avgGpaByMajor = students.stream()
            .collect(Collectors.groupingBy(
                Student::getMajor,
                Collectors.averagingDouble(Student::getGpa)
            ));
        avgGpaByMajor.forEach((major, avg) -> 
            System.out.println(major + ": " + String.format("%.2f", avg))
        );
        
        // ─── 3. أعلى 3 طلاب GPA ───────────────────────
        System.out.println("\n=== Top 3 Students ===");
        students.stream()
            .sorted(Comparator.comparing(Student::getGpa).reversed())
            .limit(3)
            .forEach(System.out::println);
        
        // ─── 4. عدد الطلاب في كل تخصص ─────────────────
        System.out.println("\n=== Students per Major ===");
        Map<String, Long> countByMajor = students.stream()
            .collect(Collectors.groupingBy(
                Student::getMajor,
                Collectors.counting()
            ));
        System.out.println(countByMajor);
        
        // ─── 5. أسماء الطلاب مرتبة ─────────────────────
        System.out.println("\n=== Sorted Names ===");
        String names = students.stream()
            .map(Student::getName)
            .sorted()
            .collect(Collectors.joining(", "));
        System.out.println(names);
        
        // ─── 6. إحصائيات الأعمار ───────────────────────
        System.out.println("\n=== Age Statistics ===");
        IntSummaryStatistics ageStats = students.stream()
            .mapToInt(Student::getAge)
            .summaryStatistics();
        System.out.println(ageStats);
        
        // ─── 7. Group by age range ─────────────────────
        System.out.println("\n=== Students by Age Range ===");
        Map<String, List<Student>> byAgeRange = students.stream()
            .collect(Collectors.groupingBy(s -> {
                if (s.getAge() <= 21) return "Young (≤21)";
                else return "Older (>21)";
            }));
        byAgeRange.forEach((range, list) -> {
            System.out.println(range + ": " + list.size() + " students");
        });
        
        // ─── 8. هل كل الطلاب GPA > 3.0؟ ───────────────
        boolean allPassing = students.stream()
            .allMatch(s -> s.getGpa() >= 3.0);
        System.out.println("\nAll passing? " + allPassing);
        
        // ─── 9. أي طالب GPA = 4.0؟ ────────────────────
        boolean hasPerfect = students.stream()
            .anyMatch(s -> s.getGpa() == 4.0);
        System.out.println("Any perfect GPA? " + hasPerfect);
        
        // ─── 10. Partition by GPA ──────────────────────
        Map<Boolean, List<Student>> partitioned = students.stream()
            .collect(Collectors.partitioningBy(s -> s.getGpa() >= 3.5));
        
        System.out.println("\n=== High achievers (GPA ≥ 3.5) ===");
        partitioned.get(true).forEach(System.out::println);
    }
}
```

---

فاهم دلوقتي؟ أي جزء عايز تفصيل أكتر فيه؟ 🚀






تمام يا علي 👌  
خلينا نفهم **`Arrays.asList()`** صح وبشكل عملي.

---

# 🔹 يعني إيه `Arrays.asList()` ؟

دي method موجودة في كلاس:

```java
java.util.Arrays
```

وظيفتها:

> بتحول Array إلى List بسرعة في سطر واحد.

---

# 🔹 مثال بسيط

بدل ما تكتب:

```java
List<String> names = new ArrayList<>();
names.add("Ali");
names.add("Omar");
names.add("Mona");
```

تقدر تكتب:

```java
List<String> names = Arrays.asList("Ali", "Omar", "Mona");
```

خلصت في سطر 👌

---

# 🔹 مثال مع أرقام

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
```

---

# 🔥 أهم حاجة لازم تعرفها (مهمة جدًا)

الـ List اللي بترجع من `Arrays.asList()`:

❌ مش List عادية  
❌ مش ArrayList حقيقية  
❌ حجمها ثابت (Fixed Size)

يعني:

```java
numbers.add(6);    // ❌ Runtime Error
numbers.remove(1); // ❌ Runtime Error
```

هيطلع:

```java
UnsupportedOperationException
```

---

# 🔹 ليه بيحصل كده؟

لأن `Arrays.asList()` بترجع:

> List مربوطة بالـ Array الأصلي

فمش مسموح تزود أو تشيل عناصر  
لكن مسموح تعدل القيمة نفسها:

```java
numbers.set(0, 10); // ✔ ينفع
```

---

# 🔹 لو عايز List تقدر تعدل حجمها؟

اعمل كده:

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(1,2,3,4));
```

كده بقت ArrayList حقيقية  
وتقدر تعمل:

```java
numbers.add(5); // ✔
```

---

# 🔹 Arrays.asList vs List.of

لو بتستخدم Java 9+

```java
List<String> names = List.of("Ali", "Omar");
```

⚠️ دي immutable بالكامل  
لا add ولا set ولا remove

---

# 🔹 تستخدمها إمتى؟

✔ لما تعمل Test سريع  
✔ لما تعمل Stream بسرعة  
✔ لما تعمل List ثابتة

مثال مع Stream:

```java
Arrays.asList(1,2,3,4)
      .stream()
      .filter(n -> n > 2)
      .forEach(System.out::println);
```

---

# 🔥 الفرق بينهم بسرعة

|Method|حجم ثابت؟|تقدر تعدل العنصر؟|تقدر تزود/تحذف؟|
|---|---|---|---|
|Arrays.asList|✔|✔|❌|
|new ArrayList<>(...)|❌|✔|✔|
|List.of|✔|❌|❌|

---

# 🎯 خلاصة تحفظها للانترفيو

> `Arrays.asList()` converts an array into a fixed-size List backed by the original array.

لو تحب، أشرحلك الفرق بينها وبين ArrayList في الذاكرة وإزاي الاتنين بيشتغلوا داخليًا 🔥