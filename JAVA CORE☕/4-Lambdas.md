

---

# 🧠 المشكلة الحقيقية قبل Java 8 (مهمة جداً للانترفيو)

تخيل عندك List فيها مليون عنصر وعايز تعمل:

- فلترة
    
- Sorting
    
- Mapping
    
- Aggregation
    

قبل Java 8 كنا نكتب **Loops جهنمية** 😅

## مثال واقعي قبل Java 8

عايزين نجيب كل الأرقام الزوجية من List.

```java
List<Integer> nums = Arrays.asList(1,2,3,4,5,6);

List<Integer> evens = new ArrayList<>();

for (Integer n : nums) {
    if (n % 2 == 0) {
        evens.add(n);
    }
}
```

المشكلة هنا 👇

❌ كود طويل  
❌ غير قابل لإعادة الاستخدام  
❌ صعب القراءة  
❌ صعب يعمل Parallel processing  
❌ كل مرة بنكتب loop جديد

Java كانت محتاجة طريقة تقول:

> "قولّي انت عايز تعمل ايه مش ازاي تعمله"

وده اسمه:

## Declarative Programming

---

# 💡 الحل: Lambda + Streams = ثورة Java 8

نفس المثال أصبح:

```java
List<Integer> evens =
        nums.stream()
            .filter(n -> n % 2 == 0)
            .toList();
```

لاحظ الفرق 😍  
احنا كتبنا "عايز فلترة" مش "ازاي نعمل loop".

ده أهم سبب اتعملت عشانه Lambda.

---

# 🔥 الفكرة الأساسية

Streams بتحتاج تبعت **function كـ parameter**.

مثلاً:

- filter محتاج function ترجع boolean
    
- sort محتاج function تقارن
    
- map محتاج function تحول
    

وهنا ظهرت Lambda.

---

# 🎯 أهم استخدام للـ Lambda = Collections + Streams

خلينا نفهم Stream بسرعة:

## Stream = Pipeline

زي خط إنتاج 🏭

```
Data → Filter → Map → Sort → Collect
```

---

# 1️⃣ FILTER — أهم استخدام للـ Lambda

### قبل Java 8

Loop + if + List جديدة 😭

### بعد Java 8

```java
nums.stream()
    .filter(n -> n % 2 == 0)
    .forEach(System.out::println);
```

### معنى الـ Lambda هنا

```java
n -> n % 2 == 0
```

دي Function اسمها:  
Predicate

بتاخد input وترجع boolean.

---

# 2️⃣ MAP — تحويل البيانات

عايز تحول List أسماء → أطوال الأسماء.

### قبل Java 8

```java
List<Integer> lengths = new ArrayList<>();
for (String s : names) {
    lengths.add(s.length());
}
```

### بعد Java 8

```java
names.stream()
     .map(name -> name.length())
     .toList();
```

Lambda هنا:

```java
name -> name.length()
```

دي Function بتحول قيمة لقيمة.

---

# 3️⃣ SORT — أشهر سؤال Interview 🔥

قبل Java 8:

```java
Collections.sort(list, new Comparator<Integer>() {
    public int compare(Integer a, Integer b) {
        return a - b;
    }
});
```

بعد Java 8:

```java
list.sort((a, b) -> a - b);
```

### sort بيحتاج ايه؟

Comparator → Function تقارن بين عنصرين.

---

# 4️⃣ forEach — تنفيذ كود على كل عنصر

```java
nums.forEach(n -> System.out.println(n));
```

دي Consumer.

---

# 🧠 ليه Lambda ممتازة مع Streams؟

لأن كل عمليات Streams بتستقبل:  
👉 Behavior

مش Data.

---

# ✨ أهم Pipeline Interview Example

### سؤال مشهور جداً 🔥

"هات أسماء الناس اللي سنهم أكبر من 18 وحولها UpperCase"

```java
people.stream()
      .filter(p -> p.getAge() > 18)
      .map(p -> p.getName())
      .map(name -> name.toUpperCase())
      .toList();
```

ده سؤال بيتسأل كتير.

---

# 💬 ازاي الانترفيو بيجيبهالك؟

## سؤال 1

**What problem did Lambdas solve?**

الإجابة النموذجية:

- Reduce boilerplate code
    
- Enable Functional Programming
    
- Simplify working with Collections
    
- Make Streams possible
    
- Allow passing behavior as data
    

احفظها 👌

---

## سؤال 2

**Why Lambdas are mostly used with Streams?**

الإجابة:

Because stream operations need functions:

- filter → Predicate
    
- map → Function
    
- sort → Comparator
    
- forEach → Consumer
    

---

## سؤال 3 🔥 (كلاسيك)

"حول الكود ده لـ Stream"

```java
List<String> result = new ArrayList<>();
for(String s : list){
    if(s.startsWith("A")){
        result.add(s.toUpperCase());
    }
}
```

الحل:

```java
list.stream()
    .filter(s -> s.startsWith("A"))
    .map(String::toUpperCase)
    .toList();
```

---

## سؤال 4

**الفرق بين map و filter ؟**

|map|filter|
|---|---|
|يحول العنصر|يقرر يمر ولا لا|
|يرجع نفس العدد|ممكن يقلل العدد|

---

## سؤال 5 🔥 مهم جداً

**Lazy vs Eager**

Streams lazy 👌  
يعني التنفيذ يحصل في الآخر عند:

- collect
    
- forEach
    
- toList
    

---

# ⚠️ أخطاء شائعة في الانترفيو

### ❌ استخدام stream بدون terminal operation

غلط:

```java
list.stream().filter(x -> x > 5);
```

مش هينفذ حاجة.

لازم:

```java
.toList()
```

---

### ❌ تعديل الList داخل stream

غلط جداً 🚨

```java
list.stream().forEach(x -> list.add(x));
```

---

# 🎯 خلاصة تحفظها للانترفيو

Lambda ظهرت عشان:

1️⃣ تقلل الكود  
2️⃣ تخلي Java تدعم Functional Programming  
3️⃣ تمكّن Streams  
4️⃣ تسهّل التعامل مع Collections  
5️⃣ تدعم Parallel processing

---

لو عايز نخش على **Method References + Stream operations deep dive** ونعمل تمارين زي الانترفيو بالظبط قولّي 🔥