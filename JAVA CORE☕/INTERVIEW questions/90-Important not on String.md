أيوه، لو ده **السطر الوحيد**:

```java
String a = new String("hello");
```

==فالإجابة في الحالة المعتادة: **2 Objects**.== =>
 الحالة الافتراضية (لو "hello" مش موجودة قبل كده في الـ String Pool)

### ليه؟

Java بتتعامل مع الجزئين دول:

```java
"hello"
```

و

```java
new String(...)
```

بشكل مختلف.

#### 1. `"hello"`

دي **String Literal**، فـ Java بتحطها في **String Pool**:

```text
String Pool
┌─────────────┐
│   "hello"   │ ← Object #1
└─────────────┘
```

#### 2. `new String("hello")`

`new` معناها: **اعمل Object جديد**.

فيتعمل Object تاني في الـ Heap:

```text
Heap
┌─────────────┐
│   "hello"   │ ← Object #2
└─────────────┘
       ↑
       a
```

إذن:

```text
"hello"                    → Object #1 (String Pool)
new String("hello")        → Object #2 (Heap)

Total = 2 Objects
```

### 🔥 الزتونة

```java
String a = "hello";
```

→ **1 Object**

أما:

```java
String a = new String("hello");
```

→ **2 Objects** لو `"hello"` لم تكن موجودة بالفعل في الـ String Pool.

**ملاحظة دقيقة:** في Java الحديثة، الـ JVM قد يكون عندها تفاصيل تنفيذية/تحسينات داخلية، لكن **إجابة الانترفيو المتوقعة لهذا المثال هي 2**.