
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
✔️ الحل: نعمل **Adapter** يخلّيهم يشتغلوا مع بعض

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

لو حابب أشرحلك **Real Spring Boot example (زي RestTemplate + APIs قديمة)** هتفهمه أسرع جداً قولّي 👍