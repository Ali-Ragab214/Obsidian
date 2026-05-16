
---

# أولاً: يعني ايه Decorator Pattern ؟

هو **Structural Design Pattern**.

فكرته الأساسية:

> نضيف Features / Behavior جديد للكائن **من غير ما نعدل الكلاس نفسه**.

يعني بدل ما نعمل inheritance كتير ونبوّظ الدنيا…  
نلفّ الـ object داخل wrapper يضيفله سلوك جديد.

اسمها Decorator = كأنك “بتزيّن” object بخصائص إضافية.

---

# ليه أصلاً نستخدمه؟

تخيل عندك Class ثابت ومينفعش تعدل عليه (أو مش عايز تكسره).

لو استخدمت inheritance بس → هتعمل classes كتير جدًا 😵‍💫  
وده اسمه **Class Explosion**

مثال بسيط:  
عندك Coffee system:

عايز:

- Coffee
    
- Coffee + Milk
    
- Coffee + Sugar
    
- Coffee + Milk + Sugar
    
- Coffee + Milk + Caramel
    
- Coffee + Milk + Sugar + Caramel 😵‍💫😵‍💫😵‍💫
    

لو inheritance:

```java
CoffeeWithMilk
CoffeeWithSugar
CoffeeWithMilkAndSugar
CoffeeWithMilkSugarCaramel
...
```

كابوس.

Decorator حل المشكلة 👌

---

# الفكرة ببساطة

بدل ما نعمل subclass لكل combination  
نعمل **Decorators** نركبهم فوق بعض runtime.

زي كده:

```java
Coffee coffee =
    new MilkDecorator(
        new SugarDecorator(
            new BasicCoffee()
        )
    );
```

كل decorator يضيف feature.

---

# تركيب الـ Pattern (Structure)

الـ Decorator بيكون مكوّن من 4 أجزاء:

1. Component (Interface)
    
2. Concrete Component (الكلاس الأساسي)
    
3. Decorator (Abstract wrapper)
    
4. Concrete Decorators (الإضافات)
    

---

# مثال من أرض الواقع ☕ (مهم جداً)

## سيناريو: Coffee Shop

الزبون يطلب:

- قهوة عادية
    
- يضيف لبن
    
- يضيف سكر
    
- يضيف كراميل
    

كل إضافة = Decorator

---

# Step 1) Component Interface

ده الشكل العام لأي Coffee.

```java
public interface Coffee {
    String getDescription();
    double getCost();
}
```

---

# Step 2) Concrete Component

القهوة الأساسية.

```java
public class BasicCoffee implements Coffee {

    @Override
    public String getDescription() {
        return "Basic Coffee";
    }

    @Override
    public double getCost() {
        return 5.0;
    }
}
```

---

# Step 3) Abstract Decorator

ده أهم جزء في الباترن.

لاحظ:  
Decorator **يمسك object من نفس النوع**.

دي أهم نقطة في الباترن كله.

```java
public abstract class CoffeeDecorator implements Coffee {

    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}
```

ليه؟  
عشان يقدر يلف حوالين أي Coffee.

---

# Step 4) Concrete Decorators

## Milk Decorator

```java
public class MilkDecorator extends CoffeeDecorator {

    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 2.0;
    }
}
```

---

## Sugar Decorator

```java
public class SugarDecorator extends CoffeeDecorator {

    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + Sugar";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 1.0;
    }
}
```

---

## Caramel Decorator

```java
public class CaramelDecorator extends CoffeeDecorator {

    public CaramelDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + Caramel";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 3.0;
    }
}
```

---

# Step 5) الاستخدام الحقيقي

```java
public class Main {
    public static void main(String[] args) {

        Coffee coffee = new BasicCoffee();
        coffee = new MilkDecorator(coffee);
        coffee = new SugarDecorator(coffee);
        coffee = new CaramelDecorator(coffee);

        System.out.println(coffee.getDescription());
        System.out.println(coffee.getCost());
    }
}
```

Output:

```
Basic Coffee + Milk + Sugar + Caramel
11.0
```

🔥 شفت القوة؟  
قدرنا نركّب features runtime.

---

# أهم ميزة في الباترن

بدل ما تعمل inheritance  
عملت **composition + wrapping**

وده يحقق:  
Open/Closed Principle

> Open for extension  
> Closed for modification

---

# مثال Real جداً من البرمجة 💻

## Java I/O Streams

دي أشهر implementation للDecorator.

مثال:

```java
InputStream file = new FileInputStream("data.txt");
InputStream buffered = new BufferedInputStream(file);
InputStream data = new DataInputStream(buffered);
```

هنا:

- FileInputStream → الأساس
    
- BufferedInputStream → Decorator
    
- DataInputStream → Decorator
    

كل واحد بيضيف behavior.

ده Decorator حقيقي في Java نفسها 🔥

---

# امثلة من الواقع غير القهوة

### 🎮 Game Characters

Character → basic player  
Decorators:

- ArmorDecorator
    
- WeaponDecorator
    
- SpeedBoostDecorator
    

### 💳 Payment System

Payment → interface  
Decorators:

- LoggingPaymentDecorator
    
- FraudCheckDecorator
    
- DiscountDecorator
    

### 📦 E-commerce Order

Order → base order  
Decorators:

- GiftWrapDecorator
    
- ExpressShippingDecorator
    
- InsuranceDecorator
    

---

# امتى استخدم Decorator؟

استخدمه لما:

- عايز تضيف behavior runtime
    
- مش عايز inheritance explosion
    
- عايز تضيف features optional
    
- عايز تبني system extensible
    

---

# الفرق بين Decorator و Inheritance

|Inheritance|Decorator|
|---|---|
|Compile time|Runtime|
|rigid|flexible|
|class explosion|scalable|
|static behavior|dynamic behavior|

---

# عيوب الباترن ⚠️

لازم تبقى فاهمها كسينيور:

1. بيزود عدد الكلاسات
    
2. debugging أصعب شوية
    
3. الكود بيبقى فيه wrapping كتير
    

لكن فوائده أكبر بكتير.

---

# خلاصة السينيور 👌

Decorator =  
"طريقة ذكية تضيف Features للكائن من غير ما تكسر الكود أو تعمل inheritance مرعب"

لو فهمته كويس هتستخدمه كتير في:

- Logging
    
- Security
    
- Caching
    
- Validation
    
- Middleware
    

---

لو حابب بعد كده أشرحلك الفرق بين  
Decorator vs Proxy vs Adapter  
دي أهم 3 patterns بيتلخبطوا ببعض 🔥