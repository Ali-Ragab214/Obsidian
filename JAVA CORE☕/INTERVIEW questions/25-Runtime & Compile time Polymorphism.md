

---

# 🧠 What is Polymorphism?

**Polymorphism = نفس الـ method name لكن سلوك مختلف**

يعني:

> “Same interface, different behavior”
> ## 🧠 يعني إيه Polymorphism باختصار؟

**Polymorphism = إن نفس الـ method أو نفس الـ reference يدي أكتر من سلوك مختلف حسب الكائن اللي بيشتغل عليه.**

---

## ⚡ الجملة:

> “Same interface, different behavior”

## معناها ببساطة:

- **Same interface** = نفس الطريقة/الـ method name
    
- **Different behavior** = التنفيذ بيختلف حسب النوع الحقيقي للكائن
    

---

## 💡 مثال سريع جدًا:

```java
Animal a = new Dog();
a.sound();  // Dog barks

a = new Cat();
a.sound();  // Cat meows
```

👉 نفس `sound()`  
👉 لكن النتيجة مختلفة حسب object

---

## 🧠 الخلاصة في سطر واحد:

> Polymorphism يعني إنك تستخدم نفس الـ method لكن السلوك بيتغير حسب نوع الـ object اللي بيتنفذ عليه.
> 

---

# ⚙️ Types of Polymorphism in Java

Java فيها نوعين أساسيين:

1. **Compile-time Polymorphism (Static Polymorphism)**
    
2. **Runtime Polymorphism (Dynamic Polymorphism)**
    

---

# 🟦 1. Compile-time Polymorphism

## 📌 Definition:

هو إن Java تحدد أي method هتتنفذ أثناء **compile time** (وقت الكومبايل)

---

## ⚙️ بيحصل عن طريق:

### 👉 Method Overloading

---

## 📌 Example:

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

---

## 📌 Usage:

```java
Calculator calc = new Calculator();

calc.add(2, 3);
calc.add(2, 3, 4);
calc.add(2.5, 3.5);
```

---

## 🧠 What happens?

- Java تختار method المناسبة **وقت compilation**
    
- بناءً على:
    
    - number of parameters
        
    - type of parameters
        

---

## ✅ Features:

- Faster (no runtime decision)
    
- Determined at compile time
    
- Achieved via method overloading
    
- No inheritance required
    

---

# 🟩 2. Runtime Polymorphism

## 📌 Definition:

هو إن Java تحدد أي method هتتنفذ أثناء **runtime** (وقت التشغيل)

---

## ⚙️ بيحصل عن طريق:

### 👉 Method Overriding

---

## 📌 Example:

```java
class Animal {
    void sound() {
        System.out.println("Animal makes sound");
    }
}
```

```java
class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}
```

```java
class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}
```

---

## 📌 Usage:

```java
Animal a;

a = new Dog();
a.sound();

a = new Cat();
a.sound();
```

---

## 🧠 What happens?

- Java تختار method أثناء runtime
    
- بناءً على **actual object type**
    

---

## Output:

```text
Dog barks
Cat meows
```

---

## ✅ Features:

- Slower than compile-time (decision at runtime)
    
- Requires inheritance
    
- Achieved via method overriding
    
- Supports dynamic behavior
    

---

# ⚡ Main Differences (Interview Table)

|Feature|Compile-time|Runtime|
|---|---|---|
|Binding|Early binding|Late binding|
|When decided|Compile time|Runtime|
|Technique|Method overloading|Method overriding|
|Speed|Faster|Slightly slower|
|Inheritance required|❌ No|✅ Yes|
|Flexibility|Low|High|

---

# 🧠 Simple intuition

## Compile-time polymorphism:

> "I know which method I’ll call before running the program"

## Runtime polymorphism:

> "I’ll decide which method to call while the program is running"

---

# 🔥 Real-world analogy

## Compile-time:

📞 Calling a restaurant menu:

- Press 1 → Pizza
    
- Press 2 → Burger  
    👉 System already knows options
    

---

## Runtime:

🚗 Uber:

- You book ride
    
- Any driver may accept  
    👉 Decision happens after request
    

---

# 💣 Interview-ready answer (short version)

> Compile-time polymorphism is method overloading where method resolution happens during compilation based on method signature.  
> Runtime polymorphism is method overriding where method resolution happens at runtime based on the actual object type.  
> Compile-time polymorphism is faster and static, while runtime polymorphism is more flexible and dynamic.

---

لو عايز، أقدر أديك:

- أسئلة interview trick questions على polymorphism
    
- أو explain الفرق بين overloading vs overriding في deep level
    
- أو MCQ style interview practice
    

قولّي 👍