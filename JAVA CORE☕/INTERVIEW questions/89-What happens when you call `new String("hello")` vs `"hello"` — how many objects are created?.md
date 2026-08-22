

### 1. `"hello"`

```java
String s = "hello";
```

هنا Java بتدور على `"hello"` في **String Pool**.

- لو `"hello"` مش موجودة → يتم إنشاء **String object واحد** في الـ String Pool.
    
- لو موجودة بالفعل → مش بيتعمل Object جديد، و`s` بس بتشير للـ object الموجود.
    

مثال:

```java
String a = "hello";
String b = "hello";

System.out.println(a == b); // true
```

الاتنين بيشيروا لنفس الـ String object.

```text
String Pool

   ┌───────────────┐
   │   "hello"     │
   └───────────────┘
       ↑       ↑
       │       │
       a       b
```

---

### 2. `new String("hello")`

```java
String s = new String("hello");
```

هنا الموضوع مختلف.

عندنا **Object جديد** بيتعمل باستخدام `new`، وبالتالي `s` تشير إلى String object جديد في الـ Heap.

لكن `"hello"` نفسها **String literal**، وبالتالي لازم تكون موجودة في String Pool.

فلو `"hello"` لم تكن موجودة قبل كده، فغالبًا عند أول تنفيذ:

```java
String s = new String("hello");
```

هيكون عندك **2 String objects**:

```text
String Pool                  Heap

┌───────────────┐            ┌───────────────┐
│    "hello"    │            │    "hello"    │
└───────────────┘            └───────────────┘
                              ↑
                              │
                              s
```

يعني:

**1 في String Pool + 1 في Heap = 2 objects**

---

### أهم مثال

```java
String a = "hello";
String b = new String("hello");

System.out.println(a == b);
```

الناتج:

```text
false
```

ليه؟

لأن: مهم جدا في object  في  string pool   وواحد اخر في ال Heap 

```text
a ────────→ String Pool "hello"

b ────────→ Heap String "hello"
```

هما **Objectين مختلفين**.

لكن:

```java
System.out.println(a.equals(b));
```

الناتج:

```text
true
```

لأن `equals()` بتقارن **المحتوى**، بينما `==` مع الـ references بتقارن هل الاتنين بيشيروا لنفس الـ object.

---

### ⚠️ نقطة مهمة جدًا في الانترفيو

لو قلت:

```java
String s = new String("hello");
```

**هل دائمًا بيتعمل 2 objects؟**

مش بالضرورة.

لو `"hello"` موجودة بالفعل في الـ String Pool:

```java
String x = "hello";
String s = new String("hello");
```

فالـ literal `"hello"` مش محتاجة Object جديد.

ساعتها `new String(...)` هيعمل **Object واحد جديد فقط**.

يعني:

```text
String Pool                  Heap

"hello"                      new String("hello")
   ↑                              ↑
   x                              s
```

### الزتونة 🧠

|Code|Objects جديدة|
|---|--:|
|`String s = "hello";` لو `"hello"` مش موجودة|**1**|
|`String s = "hello";` لو موجودة|**0**|
|`String s = new String("hello");` والـ literal مش موجودة|**2**|
|`String s = new String("hello");` والـ literal موجودة|**1**|

وأهم قاعدة تحفظها:

> **String literal → String Pool**  
> **`new String()` → Object جديد في Heap**  
> **`==` → نفس الـ object؟**  
> **`equals()` → نفس المحتوى؟**