حلو 🔥 دلوقتي هنحوّل الـ Decorator من كلام نظري → **تاسك عملي بجافا** زي اللي ممكن ييجي في انترفيو.

هديك **Scenario + Requirements + Steps** وانت تنفذه ✨

---

# 💼 Task — Online Food Delivery (Decorator Pattern)

## 🎬 Scenario

إنت شغال على سيستم **طلب أكل أونلاين**.

العميل بيطلب أكلة أساسية (Base meal)  
وبعد كده يقدر يضيف **Extras**:

- Cheese
    
- Ketchup
    
- Fries
    
- Cola
    

المهم 👇  
الإضافات دي **اختيارية** وبتتضاف فوق بعض بأي ترتيب.

وده بالظبط use case مثالي للـ Decorator.

---

# 🎯 المطلوب منك

عايزين نعرف:

- وصف الطلب (Description)
    
- السعر النهائي (Cost)
    

---

# 🧩 Step 1 — اعمل Component Interface

اعمل interface اسمه:

```java
Meal
```

يحتوي على:

```java
String getDescription();
double getCost();
```

---

# 🧩 Step 2 — اعمل Concrete Components (الأكلات الأساسية)

اعمل 2 meals:

### 🍔 Burger

- Description: "Burger"
    
- Cost: 50
    

### 🍕 Pizza

- Description: "Pizza"
    
- Cost: 70
    

---

# 🧩 Step 3 — اعمل الـ Decorator Abstract Class

اعمل abstract class اسمها:

```
MealDecorator
```

- تعمل implement لـ Meal
    
- تحتوي على:
    

```java
protected Meal meal;
```

وتستقبل meal في constructor.

---

# 🧩 Step 4 — اعمل Concrete Decorators (الإضافات)

اعمل 4 إضافات:

|Decorator|السعر|
|---|---|
|CheeseDecorator|+10|
|KetchupDecorator|+5|
|FriesDecorator|+20|
|ColaDecorator|+15|

كل واحد:

- يزوّد السعر
    
- يزوّد الوصف
    

مثال:

```
Burger + Cheese + Cola
```

---

# 🧪 Step 5 — اعمل Main Class للتجربة

عايزك تعمل سيناريوهات زي:

### Order 1

Burger + Cheese + Fries

Expected:

```
Description: Burger, Cheese, Fries
Cost: 80
```

---

### Order 2

Pizza + Cheese + Ketchup + Cola

Expected:

```
Description: Pizza, Cheese, Ketchup, Cola
Cost: 100
```

---

# ⭐ Bonus (مهم للانترفيو)

بعد ما تخلص:  
جاوب على السؤال ده 👇

**ليه استخدمنا Decorator بدل inheritance؟**

ده أهم جزء في التاسك 😎

---

ابدأ اكتب الكود، ولما تخلص ابعتهولي أراجعهولك زي code review بالظبط 👨‍💻