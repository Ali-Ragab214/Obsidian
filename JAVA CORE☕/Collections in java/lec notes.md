Linked Lists

Arrays and ArrayList are used frequently when we are developing different applications.

However, arrays and array lists suffer from a major drawback.

Removing an element from the middle of an array is expensive since all array elements
beyond the removed one must be moved toward the beginning of the array. The same is
true for inserting elements in the middle.

Another well-known data structure, the linked list, solves this problem.



// LinkedList implements List

List<String> staff = new LinkedList<>();

staff.add("First");

staff.add(new String("Second"));
staff.add("Third");

Iterator iter = staff.iterator();

String first = iter.next(); // visit first element

String second = iter.next(); // visit second element

iter.remove(); // remove last visited element


تمام يا علي 👌  
خلينا نشرح الكود ده خطوة خطوة + نفهم هو بيعمل إيه بالظبط في الـ **LinkedList** 🔥

---

# 🔹 الكود

```java
// LinkedList implements List

List<String> staff = new LinkedList<>();

staff.add("First");

staff.add(new String("Second"));
staff.add("Third");

Iterator iter = staff.iterator();

String first = iter.next(); // visit first element

String second = iter.next(); // visit second element

iter.remove(); // remove last visited element
```

---

# 🔥 أول سطر

```java
List<String> staff = new LinkedList<>();
```

هنا عملنا:

- Reference من نوع `List`
    
- Object من نوع `LinkedList`
    

✔ ده اسمه Polymorphism  
✔ نستخدم Interface بدل Concrete class

---

# 🔹 إضافة عناصر

```java
staff.add("First");
staff.add(new String("Second"));
staff.add("Third");
```

الـ LinkedList هتبقى كده في الذاكرة:

```
[First] <-> [Second] <-> [Third]
```

كل عنصر Node فيه:

- data
    
- pointer للعنصر اللي بعده
    
- pointer للعنصر اللي قبله
    

---

# 🔹 إنشاء Iterator

```java
Iterator iter = staff.iterator();
```

الـ Iterator ده أداة تمشي بيها على العناصر واحدة واحدة.

زي مؤشر واقف على أول عنصر.

⚠ الأفضل تكتبها كده:

```java
Iterator<String> iter = staff.iterator();
```

علشان تتجنب Raw Type.

---

# 🔹 أول next()

```java
String first = iter.next();
```

ده معناه:

- روح لأول عنصر
    
- رجعه
    
- حرّك المؤشر للعنصر اللي بعده
    

الـ first = "First"

المؤشر بقى واقف عند "Second"

---

# 🔹 تاني next()

```java
String second = iter.next();
```

يرجع "Second"  
ويحرك المؤشر لـ "Third"

---

# 🔥 دلوقتي أهم سطر

```java
iter.remove();
```

السطر ده بيحذف:

> آخر عنصر تم زيارته بـ next()

وآخر عنصر زرناه كان:

```
"Second"
```

يبقى اللي هيتمسح هو:

```
Second
```

القائمة هتبقى:

```
[First] <-> [Third]
```

---

# 🔥 ليه بنستخدم remove من Iterator مش من List؟

لو كنت كتبت:

```java
staff.remove("Second");
```

وأنت بتلف بالـ Iterator  
كان ممكن يطلع:

```
ConcurrentModificationException
```

لكن لما تستخدم:

```java
iter.remove();
```

ده آمن 100% أثناء الـ iteration

---

# 🔥 مهم جدًا تعرف

`iter.remove()`:

- ينفع تتنادى مرة واحدة بعد next()
    
- لو ناديتها مرتين ورا بعض → IllegalStateException
    
- لو ناديتها قبل next() → IllegalStateException
    

---

# 🎯 ملخص الحركة

|العملية|العنصر|
|---|---|
|next()|First|
|next()|Second|
|remove()|يمسح Second|

---

# 🔥 ليه LinkedList مناسبة هنا؟

لأن:

- حذف عنصر في النص O(1)
    
- عكس ArrayList اللي بتحرك عناصر بعدها O(n)
    

---

لو تحب أشرحلك:

- الفرق في الأداء بين LinkedList و ArrayList
    
- أو إزاي LinkedList شغالة داخليًا بالـ Nodes
    
- أو نعمل مثال يرمي ConcurrentModificationException ونفهمه
    

قوللي ونكمل 🔥