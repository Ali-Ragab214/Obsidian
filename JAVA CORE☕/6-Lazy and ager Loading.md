
---

# 🧠 يعني اي Lazy Loading و Eager Loading ؟

تخيل عندك App فيه **User** و **Orders**.

السؤال:  
لما أجيب User… هل أجيب الطلبات معه فورًا؟  
ولا أجيبها لما أحتاجها فقط؟

هنا ظهر المفهومين 👇

---

# ⚡ Eager Loading

يعني:

> حمّل البيانات فورًا حتى لو مش محتاجها دلوقتي.

📦 "هات كل حاجة مرة واحدة"

---

# 🐢 Lazy Loading

يعني:

> حمّل البيانات لما أطلبها فقط.

⏳ "هات الحاجة وقت ما أحتاجها"

---

# 🎯 مثال من الحياة

تخيل بتفتح Netflix:

- Eager → يحمل كل الأفلام والصور مرة واحدة 😱
    
- Lazy → يحمل اللي ظاهر بس… والباقي لما تعمل Scroll 😉
    

طبعًا Lazy أفضل هنا.

---

# 💻 مثال عملي في Java (بدون frameworks)

## عندنا Order class

```java
class Order {
    public Order() {
        System.out.println("Order loaded");
    }
}
```

---

## 1️⃣ Eager Loading Example

```java
class UserEager {
    private Order order = new Order(); // يتعمل فورًاEAGER 

    public UserEager() {
        System.out.println("User created");
    }
}
```

### التشغيل:

```java
UserEager user = new UserEager();
```

### الناتج:

```
Order loaded
User created
```

❗ لاحظ:  
الـ Order اتعمل حتى لو مستخدمناهش.

---

## 2️⃣ Lazy Loading Example

```java
class UserLazy {
    private Order order;

    public UserLazy() {
        System.out.println("User created");
    }

    public Order getOrder() {
        if(order == null) {
            order = new Order();
        }
        return order;
    }
}
```

### التشغيل:

```java
UserLazy user = new UserLazy();
```

الناتج:

```
User created
```

الـ Order لسه متعملش 👌

لما نحتاجه:

```java
user.getOrder();
```

الناتج:

```
Order loaded
```

💥 اتعمل وقت الحاجة فقط.

---

# 🔥 فين بنستخدمهم فعليًا؟

أهم استخدام ليهم:

- قواعد البيانات (ORM)
    
- Hibernate / JPA
    
- Entity Framework
    
- APIs
    
- تحميل الصور والملفات
    

---

# 🧠 مثال واقعي جدًا (Database)

تخيل:

User عنده 1000 Orders.

لو عملنا Eager:  
❌ هنحمل 1000 Order حتى لو مش محتاجهم.

لو Lazy:  
✔️ نجيب User بس  
✔️ لما يطلب Orders نجيبهم.

وده فرق أداء ضخم جدًا.

---

# ⚖️ مقارنة قوية

|المقارنة|Eager|Lazy|
|---|---|---|
|وقت التحميل|فورًا|عند الطلب|
|استهلاك الميموري|عالي|قليل|
|الأداء|أبطأ غالبًا|أسرع غالبًا|
|سهولة الاستخدام|أسهل|يحتاج تحكم|

---

# 🎯 مين الأكثر استخدامًا؟

في المشاريع الكبيرة:  
👉 **Lazy Loading هو الأكثر استخدامًا**

ليه؟

- يوفر memory
    
- يحسن الأداء
    
- يمنع تحميل بيانات ضخمة بدون داعي
    

لكن…

⚠️ Eager مفيد لما تكون:

- البيانات صغيرة
    
- أو متأكد أنك هتحتاجها دائمًا
    

---

# 🏁 الخلاصة

- Eager = حمّل كل حاجة فورًا
    
- Lazy = حمّل عند الحاجة
    
- الأفضل غالبًا = Lazy Loading
    
- الأفضل للأداء = Lazy Loading
    

---

لو حابب أشرح لك Lazy vs Eager في Hibernate/JPA (مهم جدًا لو هتشتغل Backend) قولّي 👌