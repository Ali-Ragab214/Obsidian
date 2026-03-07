
الإجابة الصحيحة هي:

**C) Lazy ✅**

---

## شرح الـ **Stream** في Java

### 1️⃣ ماذا يعني Lazy؟

- العمليات على الـ Stream **لا تنفذ مباشرة** عند تعريفها.
    
- التنفيذ يتم **فقط عند terminal operation** مثل `collect()`, `forEach()`, `reduce()`.
    
- هذا يسمح بالـ **efficient processing** وخاصة مع البيانات الكبيرة.
    

---

### 2️⃣ مثال عملي

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

Stream<Integer> stream = numbers.stream()
                                .filter(n -> {
                                    System.out.println("Filtering " + n);
                                    return n % 2 == 0;
                                });

// حتى هنا، لا شيء تم تنفيذه بعد
stream.collect(Collectors.toList());  // الآن يشتغل الفلتر
```

> Output:
> 
> ```
> Filtering 1
> Filtering 2
> Filtering 3
> Filtering 4
> Filtering 5
> ```

---

### 3️⃣ ليه باقي الاختيارات غلط؟

❌ A) Reusable → غلط، Stream **once consumed** ما ينفعش تستخدمه تاني  
❌ B) Mutable → غلط، Stream نفسه **immutable**، ما تغيرش source collection  
❌ D) Thread-safe → غلط، Streams **مش thread-safe بشكل افتراضي**

---

لو تحب أعمل لك **ملخص سريع لكل خصائص الـ Stream مع مثال لكل خاصية** هيبقى سهل تذاكره للانترفيو.

هل أعمله لك؟

## ليه يحصل IllegalStateException؟

في Java، الـ **Stream**:

- ✅ يُستخدم **مرة واحدة فقط**
    
- ❌ غير قابل لإعادة الاستخدام (Not reusable)
    
- بعد تنفيذ **terminal operation** يصبح stream مغلق (consumed)
    

---

## مثال يوضح المشكلة

List<Integer> list = List.of(1, 2, 3, 4);  
  
Stream<Integer> stream = list.stream();  
  
stream.forEach(System.out::println);  // أول استخدام ✔️  
  
// إعادة الاستخدام ❌  
stream.forEach(System.out::println);  // IllegalStateException

هيظهر:

java.lang.IllegalStateException: stream has already been operated upon or closed

---

## ليه كده؟

الـ Stream مصمم ليكون:

- Lazy
    
- Single-use pipeline
    
- يشبه Iterator في فكرة إنه يستهلك البيانات مرة واحدة