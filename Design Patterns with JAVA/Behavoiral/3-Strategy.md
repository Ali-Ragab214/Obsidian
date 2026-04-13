تمام 👌 هشرح لك **Strategy Pattern** من الصفر وبشكل عملي، وبعدها نطبّق على المثال اللي في الصورة سطر بسطر.

---

# 🔹 أولًا: يعني إيه Strategy Pattern؟

الـ **Strategy Pattern** هو واحد من **Design Patterns** (أنماط التصميم)، وهدفه:

> **تفصل “طريقة تنفيذ حاجة معينة” عن الكود الأساسي وتخليها قابلة للتغيير بسهولة**

يعني بدل ما تكتب كود فيه `if/else` أو `switch` علشان تختار طريقة التنفيذ، بتخلي كل طريقة لوحدها (Strategy)، وتبدّل بينهم بسهولة.

The **Strategy Pattern** allows you to define a family of algorithms, encapsulate each one in a separate class, and make them interchangeable.  
It lets you change the behavior of an object at runtime without modifying its code.
---

# 🔹 المشكلة اللي بيحلها

تخيل عندك سيستم دفع:

```java
if(paymentType.equals("credit")) {
    // credit logic
} else if(paymentType.equals("paypal")) {
    // paypal logic
}
```

❌ المشكلة:

- كل ما تضيف طريقة دفع جديدة → تعدل الكود
    
- الكود يكبر ويتعقد
    
- مخالف لمبدأ **Open/Closed Principle**
    

---

# 🔹 الحل باستخدام Strategy Pattern

بدل ده:

✔ نعمل Interface  
✔ كل طريقة دفع تكون Class  
✔ نبدّل بينهم وقت التشغيل

---

# 🔹 مكونات الـ Strategy Pattern

1. **Strategy Interface**
    
    - تعريف عام للسلوك
        
2. **Concrete Strategies**
    
    - كل تنفيذ فعلي
        
3. **Context**
    
    - الكلاس اللي بيستخدم الـ Strategy
        

---

# 🔹 تطبيق عملي بسيط

## 1. Interface

```java
public interface PaymentStrategy {
    void pay(int amount);
}
```

---

## 2. Strategies

```java
public class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}
```

```java
public class PayPalPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}
```

---

## 3. Context

```java
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

---

# 🔥 دلوقتي نخش على المثال اللي في الصورة

## 📁 Structure

```
strategy/
 ├── PaymentStrategy.java
 ├── CreditCardPayment.java
 ├── PayPalPayment.java
 ├── ShoppingCart.java
 └── StrategyDemo.java
```

---

# 🧠 شرح كل ملف بالتفصيل

---

## ✅ 1. PaymentStrategy.java

ده الـ Interface الأساسي

```java
public interface PaymentStrategy {
    void pay(int amount);
}
```

🔹 وظيفته:

- يحدد شكل الـ methods
    
- أي طريقة دفع لازم تطبّق `pay()`
    

---

## ✅ 2. CreditCardPayment.java

```java
public class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}
```

🔹 ده Strategy رقم 1  
🔹 بيطبّق طريقة الدفع بالكارت

---

## ✅ 3. PayPalPayment.java

```java
public class PayPalPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}
```

🔹 Strategy رقم 2  
🔹 طريقة دفع مختلفة

---

## ✅ 4. ShoppingCart.java (Context)

```java
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

🔹 ده أهم جزء 👇

### بيعمل إيه؟

- مش مهتم بطريقة الدفع نفسها
    
- بياخد أي Strategy وينفذها
    

### الميزة:

✔ تقدر تغيّر طريقة الدفع في runtime  
✔ بدون تعديل الكود

---

## ✅ 5. StrategyDemo.java

```java
public class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();

        cart.setPaymentStrategy(new CreditCardPayment());
        cart.checkout(100);

        cart.setPaymentStrategy(new PayPalPayment());
        cart.checkout(200);
    }
}
```

---

# 🔥 شرح التنفيذ خطوة خطوة

### 1️⃣ إنشاء الكارت

```java
ShoppingCart cart = new ShoppingCart();
```

---

### 2️⃣ اختيار Strategy

```java
cart.setPaymentStrategy(new CreditCardPayment());
```

✔ هنا اخترت طريقة الدفع (كارت)

---

### 3️⃣ تنفيذ العملية

```java
cart.checkout(100);
```

✔ هيستدعي:

```java
paymentStrategy.pay(100);
```

---

### 4️⃣ تغيير Strategy وقت التشغيل

```java
cart.setPaymentStrategy(new PayPalPayment());
```

✔ غيرت الطريقة بدون ما تغيّر الكود

---

# 💡 ليه ده مهم جدًا؟

### بدل:

```java
if(type == "paypal") ...
```

### بقي:

```java
cart.setPaymentStrategy(new PayPalPayment());
```

✔ كود أنضف  
✔ سهل التوسعة  
✔ بدون تعديل على القديم

---

# ⚖️ المميزات

✅ فصل المسؤوليات  
✅ سهل إضافة Strategy جديدة  
✅ بيقلل التعقيد  
✅ قابل للاختبار

---

# ⚠️ العيوب

❌ عدد Classes بيزيد  
❌ محتاج تفهمه كويس الأول

---

# 🧠 مثال حياتي بسيط

تخيل Google Maps:

- مرة تمشي 🚶
    
- مرة عربية 🚗
    
- مرة مواصلات 🚌
    

كل واحدة Strategy  
والـ App يبدّل بينهم

---

# 🔚 الخلاصة

- Strategy Pattern = تغيير السلوك وقت التشغيل
    
- بدل if/else → Classes مستقلة
    
- Context يستخدم Strategy بدون ما يعرف تفاصيلها
    

---

لو حابب 👌 أشرح لك الفرق بينه وبين:

- Factory Pattern
    
- أو Template Method
    

أو نطبقه على Spring Boot عملي 👨‍💻 قولّي 👍