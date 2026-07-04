# IoC (Inversion of Control) و Dependency Injection في Spring

دي من أشهر أسئلة الإنترفيوهات في Java و Spring، وغالبًا السؤال بيبدأ بـ:

> What is IoC?
> 
> What is Dependency Injection?
> 
> What is the relationship between them?

---

# أولاً: المشكلة قبل IoC

افترض عندك:

```java
class Engine {
    void start() {
        System.out.println("Engine Started");
    }
}

class Car {

    private Engine engine = new Engine();

    void drive() {
        engine.start();
        System.out.println("Car is moving");
    }
}
```

## مين المسؤول عن إنشاء الـ Engine؟

الإجابة:

```text
Car class itself
```

يعني:

```text
Car ---> creates ---> Engine
```

Diagram:

```text
+----- Car -----+
|               |
| new Engine()  |
|               |
+-------|-------+
        |
        v
     Engine
```

---

## المشكلة هنا؟

### 1) Tight Coupling

الـ Car مربوط مباشرة بـ Engine.

لو غيرنا:

```text
Engine -> ElectricEngine
```

هنضطر نعدل داخل الـ Car.

---

### 2) صعب نعمل Testing

لو عايز تعمل Mock للـ Engine:

```java
MockEngine engine = new MockEngine();
```

مش هتعرف لأن Car هي اللي بتعمل:

```java
new Engine()
```

---

# الحل: IoC

## ما معنى Inversion of Control؟

الترجمة:

```text
عكس التحكم
```

أو بشكل أدق:

```text
بدل ما الـ Object ينشئ Dependencies بنفسه،
حد تاني مسؤول عن إنشائها وإعطائها له.
```

---

بدلاً من:

```text
Car controls Engine creation
```

هيبقى:

```text
Spring controls Engine creation
```

---

Diagram:

قبل IoC:

```text
Car ---- creates ----> Engine
```

بعد IoC:

```text
Spring Container
      |
      |
      v
Engine ---> Car
```

---

## لذلك يسمى:

```text
Inversion Of Control
```

لأن التحكم انتقل من:

```text
Car
```

إلى:

```text
Spring Container
```

---

# ما هو Spring Container؟

هو المسؤول عن:

- إنشاء Objects
    
- إدارة دورة حياتهم
    
- حقن Dependencies
    
- إدارة الـ Beans
    

---

# ما هي Dependency Injection؟

Dependency Injection هي:

> الطريقة التي يحقق بها Spring الـ IoC.

بمعنى:

```text
IoC = Concept
DI = Implementation of that concept
```

---

## Diagram

```text
IoC
 |
 +----> Dependency Injection
```

---

# مثال DI

بدلاً من:

```java
class Car {

    private Engine engine = new Engine();

}
```

هنكتب:

```java
@Component
class Engine {

}
```

```java
@Component
class Car {

    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

}
```

Spring هيعمل:

```text
1) Create Engine Bean
2) Create Car Bean
3) Inject Engine into Car
```

---

Diagram:

```text
Spring Container
      |
      +---- creates Engine
      |
      +---- creates Car
                 |
                 v
               Engine
```

---

# أنواع Dependency Injection

---

# 1) Constructor Injection (الأفضل)

```java
@Service
class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

---

### المميزات

✅ Immutable

✅ Easy Testing

✅ Dependencies واضحة

✅ Recommended by Spring

---

---

# 2) Setter Injection

```java
@Service
class UserService {

    private UserRepository repository;

    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
}
```

---

المميزات:

- Optional Dependencies
    

العيوب:

- ممكن الـ Object يفضل ناقص Dependency.
    

---

---

# 3) Field Injection

```java
@Service
class UserService {

    @Autowired
    private UserRepository repository;
}
```

---

المميزات:

- سريع في الكتابة
    

العيوب:

❌ صعب في الـ Testing

❌ Hidden Dependencies

❌ لا ينصح به في Production

---

# ترتيبهم من الأفضل للأسوأ

```text
Constructor Injection
        ↓
Setter Injection
        ↓
Field Injection
```

---

# كيف يعمل Spring داخلياً؟

لنفترض:

```java
@Service
class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring يفعل تقريباً:

```java
UserRepository repo = new UserRepository();

UserService service =
        new UserService(repo);
```

---

# Diagram

```text
+----------------------+
| Spring Container     |
|                      |
| create Repository    |
| create Service       |
| inject Repository    |
+----------|-----------+
           |
           v
     UserService
```

---

# العلاقة بين IoC و DI

|IoC|DI|
|---|---|
|Concept|Technique|
|Inversion of object creation responsibility|Inject dependencies|
|Implemented by Spring Container|Mechanism used by container|

---

## الجملة المهمة للإنترفيو

> IoC is a design principle where the control of object creation and dependency management is transferred from the application code to the framework.

---

> Dependency Injection is a technique used to implement IoC by providing objects with their required dependencies from an external source rather than creating them internally.

---

# أسئلة إنترفيو مشهورة

## 1)

### What is the difference between IoC and Dependency Injection?

الإجابة:

> IoC is the principle, while Dependency Injection is one of the techniques used to achieve it.

---

## 2)

### Is Dependency Injection the only way to implement IoC?

لا.

يوجد:

- Dependency Injection
    
- Service Locator
    

لكن Spring يعتمد أساساً على DI.

---

## 3)

### Why is Constructor Injection preferred?

الإجابة:

- Makes dependencies mandatory
    
- Supports immutability
    
- Easier testing
    
- Prevents NullPointerException
    

---

## 4)

### Can Spring inject dependencies into static fields?

لا.

لأن الـ static fields ليست جزءًا من Bean instance.

---

## 5)

### What happens if Spring finds multiple beans of the same type?

مثال:

```java
@Component
class PaypalPaymentService {}

@Component
class StripePaymentService {}
```

```java
@Service
class CheckoutService {

    public CheckoutService(PaymentService paymentService) {

    }
}
```

هيظهر:

```text
NoUniqueBeanDefinitionException
```

والحل:

```java
@Qualifier("stripePaymentService")
```

أو:

```java
@Primary
```

---

# ملخص سريع للإنترفيو

```text
IoC:
Framework controls object creation.

DI:
Framework injects required dependencies.

IoC = Principle
DI = Implementation

Spring Container manages both.
```

لو حفظت وفهمت الجزء ده كويس، تقدر تجاوب على معظم أسئلة الـ IoC و DI في أي Java/Spring interview.