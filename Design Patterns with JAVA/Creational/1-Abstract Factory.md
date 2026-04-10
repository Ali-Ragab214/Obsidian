حلو 👌 دلوقتي هنفهم Abstract Factory بأقرب مثال ليك كـ Backend Developer  
هنعمل مثال **E-commerce Payments** 💳

وهنسمي كل حاجة بالاسم الصح داخل الـ Pattern.

ركز معايا خطوة خطوة ❤️

---

# تخيل السيناريو

عندك Shopping Cart 🛒  
واليوزر يقدر يدفع بـ:

- Credit Card 💳
    
- PayPal 🌐
    

لكن في كل طريقة دفع فيه **حاجتين مرتبطين ببعض**:

1. PaymentMethod → طريقة الدفع نفسها
    
2. PaymentProcessor → الجهة اللي تنفذ عملية الدفع
    

ليه الاتنين؟  
لأن في الواقع:

- كل طريقة دفع لها Processor مختلف.
    
- مينفعش تستخدم CreditCardMethod مع PayPalProcessor ❌
    

لازم يكونوا **عائلة واحدة** ✔️

وده بالظبط دور Abstract Factory 🔥

---

# أول حاجة نفهمها:

احنا عندنا **عائلتين من المنتجات**

## العائلة الأولى

Credit Card Family:

- CreditCardPaymentMethod
    
- CreditCardPaymentProcessor
    

## العائلة الثانية

PayPal Family:

- PaypalPaymentMethod
    
- PaypalPaymentProcessor
    

يبقى ده اسمه:  
👉 Families of related objects

---

# دلوقتي نطبق الـ Pattern خطوة خطوة

---

# 1️⃣ Abstract Products (Interfaces)

دي الواجهات العامة لكل نوع منتج.

## المنتج الأول: PaymentMethod

```java
public interface IPaymentMethod {
    void pay(double amount);
}
```

## المنتج الثاني: PaymentProcessor

```java
public interface IPaymentProcessor {
    void processPayment(double amount);
}
```

إيه معنى كده؟

- أي طريقة دفع لازم فيها pay()
    
- أي Processor لازم فيه processPayment()
    

دي اسمها:  
👉 Abstract Products

---

# 2️⃣ Concrete Products (Credit Card Family)

دي المنتجات الحقيقية الخاصة بالـ Credit Card.

## CreditCardPaymentMethod

```java
public class CreditCardPayment implements IPaymentMethod {

    private IPaymentProcessor processor;

    public CreditCardPayment(IPaymentProcessor processor) {
        this.processor = processor;
    }

    public void pay(double amount) {
        System.out.println("Paying with Credit Card...");
        processor.processPayment(amount);
    }
}
```

## CreditCardPaymentProcessor

```java
public class CreditCardProcessor implements IPaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing Credit Card payment: " + amount);
    }
}
```

دي اسمها:  
👉 Concrete Products (CreditCard Family)

---

# 3️⃣ Concrete Products (PayPal Family)

## PaypalPaymentMethod

```java
public class PaypalPayment implements IPaymentMethod {

    private IPaymentProcessor processor;

    public PaypalPayment(IPaymentProcessor processor) {
        this.processor = processor;
    }

    public void pay(double amount) {
        System.out.println("Paying with PayPal...");
        processor.processPayment(amount);
    }
}
```

## PaypalPaymentProcessor

```java
public class PaypalProcessor implements IPaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment: " + amount);
    }
}
```

دي اسمها:  
👉 Concrete Products (PayPal Family)

---

# 4️⃣ Abstract Factory 🔥

المصنع العام اللي بيحدد المنتجات اللي لازم تتصنع.

```java
public interface PaymentFactory {
    IPaymentMethod createPaymentMethod();
    IPaymentProcessor createPaymentProcessor();
}
```

لاحظ 👀  
المصنع بينشئ **المنتجين مع بعض**.

ده قلب الـ Abstract Factory.

---

# 5️⃣ Concrete Factories

## CreditCardFactory

```java
public class CreditCardFactory implements PaymentFactory {

    public IPaymentProcessor createPaymentProcessor() {
        return new CreditCardProcessor();
    }

    public IPaymentMethod createPaymentMethod() {
        return new CreditCardPayment(createPaymentProcessor());
    }
}
```

## PayPalFactory

```java
public class PayPalFactory implements PaymentFactory {

    public IPaymentProcessor createPaymentProcessor() {
        return new PaypalProcessor();
    }

    public IPaymentMethod createPaymentMethod() {
        return new PaypalPayment(createPaymentProcessor());
    }
}
```

دي اسمها:  
👉 Concrete Factories

---

# 6️⃣ Client (ShoppingCart) 🛒

ده الكلاس اللي بيستخدم المصنع فقط.

```java
public class ShoppingCart {

    private IPaymentMethod paymentMethod;

    public ShoppingCart(PaymentFactory factory) {
        paymentMethod = factory.createPaymentMethod();
    }

    public void checkout(double amount) {
        paymentMethod.pay(amount);
    }
}
```

ده اسمه:  
👉 Client

لاحظ:  
ShoppingCart لا يعرف CreditCard ولا PayPal 😍  
يعرف المصنع فقط.

---

# 7️⃣ تشغيل البرنامج

```java
public class Main {
    public static void main(String[] args) {

        PaymentFactory factory = new CreditCardFactory(); 
        // جرب تغيرها PayPalFactory

        ShoppingCart cart = new ShoppingCart(factory);
        cart.checkout(500);
    }
}
```

لو غيرنا المصنع فقط 👇

```java
PaymentFactory factory = new PayPalFactory();
```

النظام كله يتحول PayPal بدون تغيير أي كود تاني 🔥

---

# دلوقتي نسمي كل جزء باسمه في الـ Pattern

|الكلاس|اسمه في Pattern|
|---|---|
|IPaymentMethod|Abstract Product|
|IPaymentProcessor|Abstract Product|
|CreditCardPayment|Concrete Product|
|CreditCardProcessor|Concrete Product|
|PaypalPayment|Concrete Product|
|PaypalProcessor|Concrete Product|
|PaymentFactory|Abstract Factory|
|CreditCardFactory|Concrete Factory|
|PayPalFactory|Concrete Factory|
|ShoppingCart|Client|

---

# أهم فايدة وصلت لها الآن 💡

- ShoppingCart لا يعرف أي implementation
    
- نقدر نضيف ApplePay بسهولة جداً
    
- منعنا خلط العائلات (CreditCard مع PayPal)
    

وده هو Abstract Factory ببساطة ❤️

لو حابب نرسم Diagram بسيط يثبت الفكرة في دماغك قولّي.