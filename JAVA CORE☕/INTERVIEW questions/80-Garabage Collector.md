ببساطة:

### أولًا

```java
Book b = new Book();
Book c = new Book();
```

أنشأنا **Objectين مختلفين**:

```text
b ─────► Object 1
c ─────► Object 2
```

إذن:

- Active References = 2
    
- Reachable Objects = 2
    

---

### بعد كده

```java
b = c;
```

هنا **الـ reference الموجود في `c` بيتنسخ إلى `b`**.

فتصبح:

```text
b ─────► Object 2
c ─────► Object 2

Object 1  ← مفيش حد بيشاور عليه
```

إذن:

- References = 2
    
- Reachable Objects = 1
    
- Object 1 = **Abandoned** → eligible for **Garbage Collection**
    

⚠️ مهم: `b = c` **لم يحذف Object 1 فورًا**، لكنه أصبح غير قابل للوصول، والـ GC ممكن ينظفه لاحقًا.

---

### ثانيًا

```java
c = null;
```

دلوقتي:

```text
b ─────► Object 2
c ─────► null

Object 1  ← abandoned
```

`c` **لسه Reference Variable**، لكن قيمته `null`، يعني **مش بيشاور على أي Object**.

Object 2 لسه آمن لأن `b` بيشاور عليه.

### الخلاصة 🔥

```java
b = c;
```

معناها: **b و c يشاوروا على نفس الـ Object.**

```java
c = null;
```

معناها: **c مبقاش يشاور على أي Object.**

والـ Object يدخل في نطاق Garbage Collection **فقط عندما لا يعود هناك أي reference قابل للوصول إليه**.