## 🔹 يعني إيه **Java Streams API**؟

الـ **Streams API** اتضافت في Java 8، وهدفها إنك تتعامل مع البيانات (خصوصًا اللي جاية من Collections) بطريقة **Functional Programming** بدل الـ loops التقليدية.

ببساطة:

> Stream = طريقة لمعالجة البيانات (مش تخزينها) بشكل متسلسل أو متوازي.

---

# 🔹 الفرق بين Stream و Collection

|المقارنة|Collection|Stream|
|---|---|---|
|الهدف|تخزين البيانات|معالجة البيانات|
|التعديل|بتعدل في الداتا نفسها|لا تعدل المصدر|
|التنفيذ|فوري (Eager)|كسول (Lazy)|
|الاستخدام|for-loop|Functional (lambda)|
|إعادة الاستخدام|ينفع|لأ (مرة واحدة فقط)|

### مثال Collection عادي:

```java
List<String> names = Arrays.asList("Ali", "Omar", "Mona");
for(String name : names){
    if(name.startsWith("A")){
        System.out.println(name);
    }
}
```

### نفس المثال باستخدام Stream:

```java
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```

لاحظ الفرق في البساطة 👌

---

# 🔹 مكونات الـ Stream

أي Stream بيتكون من 3 مراحل:

1️⃣ Source  
2️⃣ Intermediate Operations  
3️⃣ Terminal Operation

مثال:

```java
names.stream()              // Source
     .filter(n -> n.length() > 3)  // Intermediate
     .count();               // Terminal
```

---

# 🔹 أهم الميثودز في Streams API

## ✅ 1) filter()

### ✔ بتعمل إيه؟

بتفلتر العناصر بشرط معين.

### ✔ Syntax:

```java
stream.filter(element -> condition)
```

### ✔ مثال:

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6);

numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(System.out::println);
```

📌 الناتج: 2 4 6

---

## ✅ 2) map()

### ✔ بتعمل إيه؟

بتحوّل كل عنصر لعنصر تاني.

### ✔ Syntax:

```java
stream.map(element -> transformation)
```

### ✔ مثال:

```java
numbers.stream()
       .map(n -> n * 2)
       .forEach(System.out::println);
```

📌 الناتج: 2 4 6 8 10 12

---

## ✅ 3) sorted()

### ✔ بتعمل إيه؟

بترتب العناصر.

```java
numbers.stream()
       .sorted()
       .forEach(System.out::println);
```

أو ب Comparator:

```java
.sorted((a,b) -> b - a)
```

---

## ✅ 4) distinct()

### ✔ بتشيل التكرار

```java
List<Integer> nums = Arrays.asList(1,2,2,3,3,4);

nums.stream()
    .distinct()
    .forEach(System.out::println);
```

📌 الناتج: 1 2 3 4

---

## ✅ 5) limit()

### ✔ بتجيب عدد معين من العناصر

```java
numbers.stream()
       .limit(3)
       .forEach(System.out::println);
```

---

## ✅ 6) count() 
this method returns (long) data type  (terminal operation)
### ✔ بترجع عدد العناصر

```java
long total = numbers.stream().count();
```

---

## ✅ 7) collect()

### ✔ بتحول النتيجة لـ Collection

```java
List<Integer> evenNumbers = numbers.stream()
                                    .filter(n -> n % 2 == 0)
                                    .collect(Collectors.toList());
```

---

## ✅ 8) forEach()

### ✔ بتنفيذ عملية على كل عنصر (Terminal)

```java
numbers.stream()
       .forEach(n -> System.out.println(n));
```

---

## ✅ 9) reduce()

### ✔ بتجمع العناصر في قيمة واحدة

```java
int sum = numbers.stream()
                 .reduce(0, (a,b) -> a + b);
```

📌 الناتج: مجموع الأرقام


تمام 👌  
خلينا نشرح **`reduce()`** واحدة واحدة وبطريقة مفهومة جدًا 🔥

احنا بنتكلم عن الـ Streams في Java.

---

# 🔹 يعني إيه reduce ؟

`reduce()` معناها:

> تاخد كل العناصر وتحولهم لقيمة واحدة بس.

يعني:

- تجمعهم
    
- تضربهم
    
- تطلع أكبر قيمة
    
- تطلع String واحدة من كذا String
    

هي دايمًا بترجع **قيمة واحدة**.

---

# 🔹 شكلها (Syntax)

أشهر شكل:

```java
T reduce(T identity, BinaryOperator<T> accumulator)
```

يعني:

```java
stream.reduce(قيمة_بداية, (a,b) -> عملية)
```

---

# 🧠 معنى كل جزء

|الجزء|معناه|
|---|---|
|identity|القيمة اللي نبدأ بيها|
|a|الناتج الحالي|
|b|العنصر اللي داخل دلوقتي|
|العملية|اللي بيجمع بينهم|

---

# ✅ مثال 1: جمع الأرقام

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5);

int sum = numbers.stream()
                 .reduce(0, (a,b) -> a + b);

System.out.println(sum);
```

📌 الناتج:

```
15
```

---

# 🔎 بيحصل إيه خطوة خطوة؟

الـ identity = 0

|a|b|الناتج|
|---|---|---|
|0|1|1|
|1|2|3|
|3|3|6|
|6|4|10|
|10|5|15|

في الآخر يرجع 15

---

# ✅ مثال 2: ضرب الأرقام

```java
int result = numbers.stream()
                    .reduce(1, (a,b) -> a * b);

System.out.println(result);
```

📌 الناتج:

```
120
```

ليه بدأنا بـ 1 مش 0؟  
لأن أي رقم × 0 = 0 ❌

---

# ✅ مثال 3: تجميع Strings

```java
List<String> names = Arrays.asList("Ali", "Omar", "Mona");

String result = names.stream()
                     .reduce("", (a,b) -> a + " " + b);

System.out.println(result);
```

📌 الناتج:

```
 Ali Omar Mona
```

---

# 🔹 شكل تاني لـ reduce (من غير identity)

```java
Optional<Integer> result = numbers.stream()
                                  .reduce((a,b) -> a + b);
```

ليه Optional؟  
لأن لو الـ Stream فاضية مفيش قيمة ترجعها.

علشان كده بترجع:

```
Optional<Integer>
```

وتجيب القيمة كده:

```java
result.get();
```

أو الأفضل:

```java
result.orElse(0);
```

---

# 🔥 الفرق بين reduce و count و collect

||reduce|count|collect|
|---|---|---|---|
|بيرجع|قيمة واحدة|رقم|Collection|
|استخدام|جمع / دمج|عد|تجميع|

---

# 🧠 إمتى أستخدم reduce؟

✔ لما عايز:

- مجموع
    
- حاصل ضرب
    
- أكبر قيمة
    
- أصغر قيمة
    
- دمج عناصر في قيمة واحدة
    

---

# ⚠️ ملاحظة مهمة جدًا

لو انت عايز مجموع أرقام  
الأفضل تستخدم:

```java
numbers.stream().mapToInt(n -> n).sum();
```

أسرع وأوضح من reduce.

---

# 🎯 تمرين ليك

عندك:

```java
List<Integer> nums = Arrays.asList(5, 10, 15, 20);
```

مطلوب:

- تطلع أكبر رقم باستخدام reduce
    

لو عايز أحله معاك خطوة خطوة قولّي 👌
  
System.out.print("Largest number");  
List<Integer> Nums = Arrays.asList(5, 30, 15, 20);  
int max = Nums.stream()  
        .reduce(0,(a,b)->(a>b?a:b));  
  
System.out.print(max);



سؤال جامد جدًا 👌🔥  
قيمة البداية ( **identity** ) في `reduce` دي أهم نقطة تفهمها صح.

خلينا نشرحها ببساطة 👇

---

# 🔹 يعني إيه identity أصلاً؟

هي **القيمة اللي العملية مش بتأثر عليها**  
يعني لو مفيش عناصر خالص في الـ Stream، دي اللي هترجع.

وكمان هي أول قيمة يبدأ منها الحساب.

---

# 🔥 القاعدة الذهبية

> اختار قيمة بداية "محايدة" للعملية اللي هتعملها.




تمام 🔥  
خلينا نشرح أهم **Terminal Operations** في Streams واحدة واحدة وبأمثلة واضحة 👌

كل اللي هنشرحه ده من الـ Streams API في Java.

---

# 🔹 1) average()

## ✅ بتعمل إيه؟

بتحسب **متوسط القيم**.

⚠️ مهمة جدًا:  
لازم تحول الـ Stream لـ رقمية الأول (`mapToInt` / `mapToDouble`)

---

## ✅ مثال:

```java
List<Integer> numbers = Arrays.asList(10, 20, 30, 40);

OptionalDouble avg = numbers.stream()
                            .mapToInt(n -> n)
                            .average();

System.out.println(avg.getAsDouble());
```

📌 الناتج:

```
25.0
```

---

## ❓ ليه بترجع OptionalDouble؟

عشان لو القائمة فاضية مفيش متوسط يتحسب.

الأفضل تستخدم:

```java
avg.orElse(0);
```

---

# 🔹 2) findFirst()

## ✅ بتعمل إيه؟

بترجع **أول عنصر** يحقق الشرط.

## بترجع:

```java
Optional<T>
```

---

## ✅ مثال:

```java
Optional<Integer> result = numbers.stream()
                                  .filter(n -> n > 15)
                                  .findFirst();

System.out.println(result.orElse(0));
```

📌 الناتج:

```
20
```

---

## 🧠 الفرق بينها وبين get(0) ؟

- get(0) → من الـ List مباشرة
    
- findFirst() → بعد الفلترة أو المعالجة
    

---

# 🔹 3) findAny()

## ✅ بتعمل إيه؟

بترجع **أي عنصر** يحقق الشرط.

📌 غالبًا زي findFirst  
لكن في `parallelStream()` ممكن يرجع عنصر مختلف.

---

## مثال:

```java
Optional<Integer> result = numbers.stream()
                                  .filter(n -> n > 15)
                                  .findAny();
```

---

## 🧠 الفرق المهم:

||findFirst|findAny|
|---|---|---|
|ترتيب|يحترم الترتيب|مش مهم|
|أسرع في parallel|أبطأ|أسرع|

---

# 🔹 4) anyMatch()

## ✅ بتعمل إيه؟

بتسأل:

> هل فيه عنصر واحد على الأقل يحقق الشرط؟

## بترجع:

```java
boolean
```

---

## مثال:

```java
boolean result = numbers.stream()
                        .anyMatch(n -> n > 35);

System.out.println(result);
```

📌 الناتج:

```
true
```

---

# 🔹 5) allMatch()

## ✅ بتعمل إيه؟

بتسأل:

> هل كل العناصر تحقق الشرط؟

---

## مثال:

```java
boolean result = numbers.stream()
                        .allMatch(n -> n > 5);

System.out.println(result);
```

📌 الناتج:

```
true
```

---

# 🔹 6) noneMatch() (مكملة معاهم)

## ✅ معناها:

ولا عنصر يحقق الشرط

```java
boolean result = numbers.stream()
                        .noneMatch(n -> n < 0);
```

📌 الناتج:

```
true
```

---

# 🔥 الفرق بينهم باختصار

|الميثود|بترجع|المعنى|
|---|---|---|
|average|OptionalDouble|متوسط|
|findFirst|Optional|أول عنصر|
|findAny|Optional|أي عنصر|
|anyMatch|boolean|هل فيه عنصر واحد؟|
|allMatch|boolean|هل الكل؟|
|noneMatch|boolean|ولا واحد؟|

---

# 🧠 نقطة مهمة جدًا

الميثودز دي كلها **Terminal**  
يعني أول ما تكتب واحدة فيهم:

```java
.stream().......
```

الـ Stream بتنتهي ومينفعش تكمل بعدها.

---

# 🎯 تمرين يثبت الفكرة

```java
List<String> names = Arrays.asList("Ali", "Omar", "Mona", "Adam");
```

مطلوب:

1️⃣ هل فيه اسم بيبدأ بحرف A؟  
2️⃣ هل كل الأسماء طولها أكبر من 3؟  
3️⃣ رجع أول اسم طوله أكبر من 4

لو حابب أحله معاك خطوة خطوة 👌























---

# 🔹 تمرين عملي شامل 🔥

## 🎯 المطلوب:

عندك ليستة طلاب، كل طالب ليه اسم ودرجة.

عايز:

1. تجيب الطلاب اللي درجتهم أكبر من 80
    
2. ترتبهم تنازلي
    
3. تجيب الأسماء بس
    
4. تحطهم في ليستة جديدة
    

---

### الكود:

```java
class Student {
    String name;
    int grade;

    Student(String name, int grade){
        this.name = name;
        this.grade = grade;
    }

    public int getGrade() { return grade; }
    public String getName() { return name; }
}
```

```java
List<Student> students = Arrays.asList(
    new Student("Ali", 90),
    new Student("Omar", 70),
    new Student("Mona", 85)
);

List<String> result = students.stream()
        .filter(s -> s.getGrade() > 80)
        .sorted((a,b) -> b.getGrade() - a.getGrade())
        .map(Student::getName)
        .collect(Collectors.toList());

System.out.println(result);
```

📌 الناتج: [Ali, Mona]

---

# 🔹 مميزات Streams API

✅ كود أقصر وأنظف  
✅ قابل للتوازي (parallelStream)  
✅ Functional Style  
✅ Lazy Evaluation

---

# 🔹 الفرق بين stream() و parallelStream()

```java
list.stream();         // sequential
list.parallelStream(); // multi-threaded
```

parallelStream بيستخدم تعدد الخيوط (Multi-threading) لتحسين الأداء مع البيانات الكبيرة.

---

# 🔹 الخلاصة

- Collection = تخزين
    
- Stream = معالجة
    
- Stream مش بتعدل الداتا الأصلية
    
- Intermediate operations بترجع Stream
    
- Terminal operation بتنهي العملية
    

---

لو حابب 👌  
أبعتلك:

- خريطة ذهنية للـ Streams
    
- تمارين محلولة زيادة
    
- أسئلة Interview عن Streams
    
- مقارنة بينها وبين الـ for-loop بالأداء
    

قولّي تحب إيه 👨‍💻🔥

