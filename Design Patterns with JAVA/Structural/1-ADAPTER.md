
---

# 📌 أولاً: Adapter Pattern يعني ايه؟

هو واحد من:

👉 **Structural Design Patterns**

يعني بيهتم بـ:

> ربط كلاسات مختلفة ببعض رغم إنهم مش متوافقين

---

# 💡 الفكرة ببساطة جدًا

تخيل عندك:

- كودك بيشتغل بطريقة معينة (Interface معين)
    
- لكن عندك كلاس جاهز/قديم شغال بطريقة مختلفة
    

❌ المشكلة: الاتنين مش شبه بعض  
✔️ الحل: نعمل **Adapter** يخلّيهم يشتغلوا مع بعض (converter)

---

# 🧠 تشبيه واقعي

- موبايل فيه Type-C ⚡
    
- شاحن قديم USB 🔌
    

مينفعش توصلهم مباشرة  
لكن تستخدم **Adapter** (وصلة)

👉 نفس فكرة التصميم بالظبط

---

# 💻 نطبق المثال بتاعنا ونفهمه جزء جزء

## 🎯 السيناريو

إنت عندك نظام Payment جديد

النظام بتاعك متوقع كده:

---

## 1️⃣ Target (اللي النظام مستنيه)

ده الـ Interface اللي الكود الجديد شغال عليه:

```java
public interface Payment {
    void pay();
}
```

📌 معناه:

- أي Payment لازم يكون فيه method اسمها pay()
    

---

# ❗ المشكلة

عندك نظام قديم جاهز:

## 2️⃣ Adaptee (الكود القديم)

```java
public class LegacyPaymentSystem {
    public void makePayment() {
        System.out.println("Payment done using legacy system");
    }
}
```

📌 هنا المشكلة:

- اسم الميثود مختلف (`makePayment` بدل `pay`)
    
- مش implements للـ interface الجديد
    
- يعني مش compatible مع النظام الجديد
    

---

# ⚙️ هنا ييجي دور Adapter 🔥

## 3️⃣ Adapter (المحول)

```java
public class PaymentAdapter implements Payment {

    private LegacyPaymentSystem legacy;

    public PaymentAdapter(LegacyPaymentSystem legacy) {
        this.legacy = legacy;
    }

    @Override
    public void pay() {
        legacy.makePayment();
    }
}
```

---

# 🧠 نفهم الـ Adapter سطر بسطر

## ✔️ implements Payment

يعني:

> أنا هخلي الكلاس ده يبان للنظام إنه Payment طبيعي

---

## ✔️ private LegacyPaymentSystem legacy;

ده اسمه:  
👉 Composition

معناه:

> أنا جوا الـ Adapter ماسك الكلاس القديم

---

## ✔️ Constructor

```java
public PaymentAdapter(LegacyPaymentSystem legacy)
```

ده معناه:

> هديلي النظام القديم وأنا هلفه لك

---

## ✔️ pay()

```java
public void pay() {
    legacy.makePayment();
}
```

هنا السحر 🔥

إنت قولت للنظام:

- “نادِ pay()”  
    لكن أنا من جوا حولتها إلى:
    
- `makePayment()`
    

👉 كده عملنا **Translation / Mapping**

---

# 🚀 الاستخدام (Client Code)

```java
public class Main {
    public static void main(String[] args) {

        LegacyPaymentSystem legacy = new LegacyPaymentSystem();

        Payment payment = new PaymentAdapter(legacy);

        payment.pay();
    }
}
```

---

# 🧠 نفهم اللي حصل فعلياً

## الخطوات:

### 1️⃣ النظام القديم اتعمل

```java
LegacyPaymentSystem legacy = new LegacyPaymentSystem();
```

---

### 2️⃣ لفيناه داخل Adapter

```java
Payment payment = new PaymentAdapter(legacy);
```

يعني:

> حولناه لشكل Payment

---

### 3️⃣ استخدمناه بشكل طبيعي

```java
payment.pay();
```

لكن في الخلفية:

```java
legacy.makePayment();
```

---

# 🎯 مين هو كل جزء في Pattern؟

|الجزء|اسمه|وظيفته|
|---|---|---|
|Payment|Target|الشكل اللي النظام عايزه|
|LegacyPaymentSystem|Adaptee|الكود القديم المختلف|
|PaymentAdapter|Adapter|اللي بيترجم بينهم|
|Main|Client|اللي بيستخدم النظام|

---

# 💡 الفكرة المهمة جداً

Adapter مش بيغير الكود القديم ❌  
Adapter بيعمل:  
👉 Wrapper (لفّة) حوالين الكود القديم

---

# ⭐ إمتى تستخدم Adapter؟

استخدمه لما:

✔️ عندك legacy system قديم  
✔️ بتدمج API خارجي مختلف  
✔️ عندك interface ومحتاج توافقه مع كلاس مش شبهه  
✔️ عايز تعيد استخدام كود بدون تعديله

---

# ⚠️ أهم فرق لازم تفهمه

## Adapter vs Factory

|Adapter|Factory|
|---|---|
|يربط كودين مختلفين|ينشئ objects|
|يحول interface|يصنع object جديد|

---

# 🧠 خلاصة قوية جدًا

Adapter Pattern يعني:

> “خلي كلاس غير متوافق يشتغل مع نظامك بدون ما تغيره”

عن طريق:

- Interface جديد (Target)
    
- كلاس قديم (Adaptee)
    
- كلاس وسيط (Adapter)
    

---

سؤال ممتاز 👌 ده أهم جزء في فهم الـ Adapter فعلاً.

خلينا نفك الكود بتاعك خطوة خطوة وبالراحة.

---

## أولاً: الميثود `pay()` هتشتغل إزاي؟

الكود بتاعك:

```java
public class PaymentAdapter implements Payment {

    private LegacyPaymentSystem legacy;

    public PaymentAdapter(LegacyPaymentSystem legacy) {
        this.legacy = legacy;
    }

    @Override
    public void pay() {
        legacy.makePayment();
    }
}
```

### مين بينادي الميثود دي أصلاً؟

السيستم الجديد عندك كله متفق إنه يتعامل مع Interface اسمه `Payment`:

```java
public interface Payment {
    void pay();
}
```

يعني أي مكان في السيستم هيستخدم الدفع هيكتب:

```java
Payment payment = .....
payment.pay();
```

السيستم **مش عارف ولا مهتم** ده PayPal ولا Visa ولا Cash.

هو عايز حاجة واحدة بس:

> كائن يحقق interface اسمه Payment.

---

### اللي بيحصل وقت التشغيل (Runtime flow)

خلينا نشوف السيناريو الحقيقي:

```java
Payment payment = new PaymentAdapter(new LegacyPaymentSystem());
payment.pay();
```

لما تنادي:

```java
payment.pay();
```

اللي بيحصل تحت الكواليس 👇

1️⃣ JVM تشوف نوع الكائن الحقيقي → `PaymentAdapter`  
2️⃣ تنفذ `PaymentAdapter.pay()`  
3️⃣ الميثود دي تنادي على:

```java
legacy.makePayment();
```

4️⃣ يتم تنفيذ الكود القديم بتاع الـ legacy system

يبقى حصل إيه؟  
السيستم فاكر إنه بيستخدم `Payment`  
لكن فعلياً شغّل **LegacyPaymentSystem** بدون ما يعرف عنه حاجة 😎

دي فايدة الـ Adapter.

---

## طب أنا استفدت إيه فعلياً؟

خلينا نشوف الفرق قبل وبعد.

### ❌ بدون Adapter

كل مكان في السيستم هيبقى كدا:

```java
LegacyPaymentSystem paypal = new LegacyPaymentSystem();
paypal.makePayment();
```

المشكلة:

- كل السيستم مربوط بالـ Legacy class
    
- لو غيرت provider هتكسر كل الكود 💥
    

---

### ✅ باستخدام Adapter

السيستم كله:

```java
Payment payment = container.getPayment();
payment.pay();
```

لو غيرت طريقة الدفع؟  
ولا سطر يتغير في السيستم 👌

---

# أهم سؤال سألت عنه 🔥

## لو عندي أكتر من طريقة دفع أعمل إيه؟

ده السيناريو الحقيقي جداً في الشغل.

عايز تدعم:

- PayPal
    
- Stripe
    
- Paymob
    
- Vodafone Cash
    

السيستم محتاج Interface واحدة بس:

```java
public interface Payment {
    void pay(double amount);
}
```

---

## عندك Providers مختلفين (Legacy / Third-party)

### PayPal Library

```java
class PaypalGateway {
    public void sendPayment(double money) {
        System.out.println("Paid with PayPal");
    }
}
```

### Stripe Library

```java
class StripeGateway {
    public void makeCharge(double amount) {
        System.out.println("Paid with Stripe");
    }
}
```

### Vodafone Cash API

```java
class VodafoneCashService {
    public void payWithVodafone(double value) {
        System.out.println("Paid with Vodafone Cash");
    }
}
```

شايف الكارثة؟ 😅  
كل واحد باسم ميثود مختلف.

---

## نعمل Adapter لكل واحد 👇

### PaypalAdapter

```java
class PaypalAdapter implements Payment {

    private PaypalGateway paypal = new PaypalGateway();

    public void pay(double amount) {
        paypal.sendPayment(amount);
    }
}
```

### StripeAdapter

```java
class StripeAdapter implements Payment {

    private StripeGateway stripe = new StripeGateway();

    public void pay(double amount) {
        stripe.makeCharge(amount);
    }
}
```

### VodafoneAdapter

```java
class VodafoneAdapter implements Payment {

    private VodafoneCashService vodafone = new VodafoneCashService();

    public void pay(double amount) {
        vodafone.payWithVodafone(amount);
    }
}
```

---

## السيستم هيستخدمهم ازاي؟ 😎

```java
public class PaymentService {

    private Payment payment;

    public PaymentService(Payment payment) {
        this.payment = payment;
    }

    public void checkout(double amount) {
        payment.pay(amount);
    }
}
```

---

## اختيار طريقة الدفع وقت التشغيل

```java
Payment payment = new StripeAdapter(); // أو PaypalAdapter أو VodafoneAdapter
PaymentService service = new PaymentService(payment);
service.checkout(500);
```

لو العميل اختار PayPal؟  
غير السطر ده بس 👇

```java
Payment payment = new PaypalAdapter();
```

خلصت ✨

ولا سطر اتغير في:

- Checkout
    
- Business logic
    
- Controllers
    
- Services
    

وده سبب إن الشركات بتحب الـ Adapter.

---

## ملخص الفكرة الكبيرة 🎯

بدل ما السيستم يعتمد على:

```
Stripe
Paypal
Vodafone
```

السيستم يعتمد على حاجة واحدة:

```
Payment Interface
```

وكل Provider ليه Adapter.

ده اسمه:

> Plug & Play architecture 🔌

---

لو تحب بعد كده نحل سؤال انترفيو كلاسيك:  
"Design a multi payment system using Adapter" خطوة بخطوة؟ 😄