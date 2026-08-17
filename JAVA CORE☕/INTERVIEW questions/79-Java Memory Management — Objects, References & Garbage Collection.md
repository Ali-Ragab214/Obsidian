بص، الصفحة دي من أهم الصفحات في فهم **Java References + Objects + Garbage Collection**. لو فهمتها كويس، هتفهم جزء كبير من الـ OOP في Java.

خلينا نمشي مع الصورة **خطوة خطوة**.

---

# أولًا: الفكرة الأساسية

في Java عندك حاجتين مختلفتين:

### 1. Object

الـ Object بيتخزن في الـ **Heap**.

### 2. Reference Variable

الـ Reference variable بيخزن **مرجع/عنوان يوصل للـ Object**.

تخيلها كده:

```text
Reference Variable          Object
       b  ----------------->  Book #1
```

يعني `b` **مش هو الـ Book object نفسه**.

هو مجرد reference بيشير للـ object.

---

# الجزء الأول

عندنا:

```java
Book b = new Book();
Book c = new Book();
```

خلينا نفهم كل سطر.

## السطر الأول

```java
Book b = new Book();
```

بيحصل حاجتين:

### أولًا:

```java
Book b;
```

اتعمل reference variable اسمه `b`.

### ثانيًا:

```java
new Book();
```

اتعمل Object جديد في الـ Heap.

يعني:

```text
b ───────────────► Book Object #1
```

---

## السطر الثاني

```java
Book c = new Book();
```

نفس الكلام:

```text
b ───────────────► Book Object #1

c ───────────────► Book Object #2
```

إذن:

```text
References = 2
Objects = 2
```

وده بالضبط اللي مكتوب في الصورة.

---

# الجزء الثاني

دلوقتي عندنا:

```java
Book d = c;
```

وهنا بقى **النقطة المهمة جدًا**.

ناس كتير بتفتكر إن:

```java
Book d = c;
```

معناها:

> اعمل Book جديد وانسخ الـ Book اللي موجود في c.

**لا ❌**

مفيش Object جديد اتعمل.

اللي حصل إن Java أخذت **قيمة الـ reference الموجودة في `c`** وحطتها في `d`.

قبل السطر:

```text
b ───────────────► Object #1

c ───────────────► Object #2
```

بعد:

```java
Book d = c;
```

يبقى:

```text
b ───────────────► Object #1

c ───────────────► Object #2
                    ▲
d ─────────────────┘
```

يعني:

```text
c ──► Object #2
d ──► Object #2
```

**الاتنين بيشيروا لنفس الـ Object.**

---

# خد بالك من دي جدًا

بعد:

```java
Book d = c;
```

عندنا:

```text
References = 3
Objects = 2
```

ليه؟

لأن عندنا 3 reference variables:

```text
b
c
d
```

لكن عندنا Objectين فقط:

```text
Book #1
Book #2
```

`d` **لم ينشئ Object جديد**.

---

# تشبيه بسيط جدًا

تخيل إن عندك بيت:

```text
🏠 بيت
```

وعندك عنوان البيت:

```text
A = عنوان البيت
```

لو عملت:

```java
B = A;
```

أنت مش بنيت بيت جديد.

أنت فقط عملت:

```text
A = عنوان البيت
B = نفس عنوان البيت
```

فـ:

```text
A ─────► 🏠
B ─────► 🏠
```

عندك **عنوانين/References** لنفس البيت.

مش بيتين.

---

# الجزء الثالث

دلوقتي الصورة بتعمل:

```java
c = b;
```

ودي مهمة جدًا.

قبلها كان:

```text
b ─────────► Object #1

c ─────────► Object #2
              ▲
d ────────────┘
```

دلوقتي:

```java
c = b;
```

معناها:

> خُد قيمة الـ reference الموجودة في `b` وحطها في `c`.

`b` كان بيشير إلى:

```text
Object #1
```

إذن `c` هيبدأ يشير إلى نفس Object #1.

فتصبح:

```text
b ─────────► Object #1
              ▲
c ────────────┘

d ─────────► Object #2
```

---

# طيب Object #2 حصل له إيه؟

قبل:

```text
c ─────────► Object #2
d ─────────► Object #2
```

بعد:

```java
c = b;
```

`c` بطل يشير إلى Object #2.

لكن `d` لسه بيشير إليه:

```text
d ─────────► Object #2
```

إذن **Object #2 لسه موجود ومش Garbage**.

ودي نقطة مهمة جدًا.

---

# طيب إمتى Object يبقى Garbage؟

الـ Garbage Collector بيشوف:

> هل في أي Reference ممكن توصل للـ Object ده؟

لو آه:

```text
Reference ─────► Object
```

يبقى الـ Object **reachable** ولسه مستخدم.

لو مفيش أي Reference بتوصله:

```text
Reference variables

b
c
d

       ❌

Object #2
```

يبقى Object #2 أصبح **eligible for Garbage Collection**.

---

# مثال يكمل الصورة

لو بعد كل اللي فات عملنا:

```java
d = b;
```

كان عندنا:

```text
b ─────► Object #1
c ─────► Object #1
d ─────► Object #2
```

بعد:

```java
d = b;
```

يبقى:

```text
b ─────► Object #1
          ▲
c ────────┤
          │
d ────────┘

Object #2
```

Object #2 **مبقاش في أي Reference بتشاور عليه**.

إذن:

```text
Object #2 → Unreachable
```

وبالتالي:

> Object #2 أصبح **eligible for Garbage Collection**.

---

# أهم حاجة في الصفحة

احفظ الجملة دي:

> **Assignment between reference variables copies the reference value, not the object.**

يعني:

```java
Book d = c;
```

لا تعني:

```text
Create new Book
Copy Book
```

لكن تعني:

```text
Copy the reference stored in c into d
```

---

# وده يفسر حاجة مهمة جدًا

لو عندك:

```java
Book b = new Book();
Book c = b;
```

يبقى:

```text
b ───────► Book Object
           ▲
c ─────────┘
```

لو عملت:

```java
c.someField = 10;
```

وبعدين:

```java
System.out.println(b.someField);
```

هتلاقي:

```text
10
```

ليه؟

لأن `b` و `c` بيشاوروا على **نفس الـ Object**.

---

# طب لو عملنا Object جديد؟

لو كتبت:

```java
Book c = new Book();
```

ده مختلف تمامًا.

هنا:

```text
b ─────────► Object #1

c ─────────► Object #2
```

عندك Objectين مختلفين.

لكن:

```java
Book c = b;
```

هنا:

```text
b ───────► Object #1
           ▲
c ─────────┘
```

Object واحد وReference اتنين.

---

# ملخص الصورة كلها

### البداية:

```java
Book b = new Book();
Book c = new Book();
```

```text
b ─────► Object #1
c ─────► Object #2

References = 2
Objects = 2
```

### بعد:

```java
Book d = c;
```

```text
b ─────► Object #1

c ─────► Object #2
          ▲
d ────────┘

References = 3
Objects = 2
```

### بعد:

```java
c = b;
```

```text
b ─────► Object #1
          ▲
c ────────┘

d ─────► Object #2

References = 3
Objects = 2
```

**Object #2 لم يتم حذفه** لأن `d` لسه بيشاور عليه.

---

## ⚠️ وأهم 3 جمل تحفظهم للإنترفيو

1. **A reference variable does not contain the object itself; it contains a reference to the object.**
    
2. **When you assign one reference variable to another, Java copies the reference value, not the object.**
    
3. **An object becomes eligible for garbage collection when it is no longer reachable through any live reference.**
    

ولو عايز تثبت الموضوع تمامًا، أهم خطوة بعد الصفحة دي هي نفهم **`==` مع الـ Objects وReferences**؛ لأنها مبنية مباشرة على نفس الفكرة دي.