

---

## الإجابة الصح: **B**

> `==` بتقارن الـ reference، و`.equals()` بتقارن الـ content

---

## الشرح بالتفصيل 🔍

### الـ `==`

- بتقارن **عنوان الميموري** (reference)
- بتسأل: "هل الاتنين بيشاوروا على **نفس الأوبجكت**؟"
- بتشتغل مع **primitives والـ objects** الاتنين

```java
int a = 5, b = 5;
System.out.println(a == b); // true ✅ (primitives قيمة)

String s1 = new String("Hello");
String s2 = new String("Hello");
System.out.println(s1 == s2); // false ❌ (عناوين مختلفة)
```

---

### الـ `.equals()`

- بتقارن **المحتوى**
- بتسأل: "هل الاتنين **متساويين في القيمة**؟"

```java
String s1 = new String("Hello");
String s2 = new String("Hello");
System.out.println(s1.equals(s2)); // true ✅
```

---

### ليه D غلط؟ 🤔

```java
int a = 5;
int b = 5;
System.out.println(a == b); // true
// == شغالة مع primitives ✅

String s1 = "Hi";
String s2 = "Hi";
System.out.println(s1 == s2); // true !!
// ليه؟ لأن Java بتستخدم String Pool 🎯
```

---

## String Pool مهم تعرفه! 🎯

```
String s1 = "Hello"     ← بتروح String Pool
String s2 = "Hello"     ← نفس العنوان في Pool
s1 == s2 ← true ✅

String s3 = new String("Hello") ← Heap جديد
s1 == s3 ← false ❌
```

---

## القاعدة الذهبية 💡

> في الـ Interview، لما يسألوك عن مقارنة Strings **دايماً استخدم `.equals()`** مش `==`

---


