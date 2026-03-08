تمام يا علي. خلينا نعمل **شرح احترافي لكن واضح** لمفهوم **Beans** في Spring.

---

# ما هو Spring Bean ؟

في Spring Framework:

> **Bean هو أي Object يتم إنشاؤه وإدارته بواسطة Spring IoC Container.**

بمعنى أبسط:

```text
Any Object created and managed by Spring = Bean
```

---

# الفرق بين Object و Bean

## Object عادي

```java
Engine engine = new PetrolEngine();
```

الديفيلوبر هو الذي:

- أنشأ الـ object
    
- يتحكم في حياته
    

الدياجرام:

```
Developer
   │
   ▼
creates object
```

---

## Bean

```java
@Component
class PetrolEngine {}
```

Spring هو الذي:

- ينشئ الـ object
    
- يديره
    
- يحقنه في كلاس آخر
    

الدياجرام:

```
Spring IoC Container
        │
        ▼
creates Bean
```

---

# الدياجرام الأساسي للـ Beans

```
           Spring IoC Container
        ┌────────────────────────┐
        │                        │
        │     manages beans      │
        │                        │
        └─────────┬──────────────┘
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
   Car Bean   Engine Bean   Service Bean
```

كل هذه الكلاسات أصبحت **Beans** لأن Spring أنشأها.

---

# مثال عملي

## Engine

```java
@Component
class PetrolEngine {

    public void start(){
        System.out.println("Engine started");
    }

}
```

## Car

```java
@Component
class Car {

    private PetrolEngine engine;

    public Car(PetrolEngine engine){
        this.engine = engine;
    }

}
```

---

# ماذا يحدث عند تشغيل البرنامج؟

### Step 1

Spring يبدأ **Component Scan**

يبحث عن:

```
@Component
```

---

### Step 2

يجد الكلاسات:

```
PetrolEngine
Car
```

---

### Step 3

ينشئ Beans

```
PetrolEngine Bean
Car Bean
```

---

### Step 4

يحقن الـ Dependency

```
Car → يحتاج Engine
Spring inject PetrolEngine
```

الدياجرام:

```
Car Bean
   │
   ▼
Engine Bean
```

---

# أين يتم تخزين الـ Beans؟

Spring يخزنهم داخل:

```
IoC Container
```

تقريباً مثل **خزانة Objects**.

```
IoC Container
   │
   ├── Car Bean
   ├── Engine Bean
   ├── UserService Bean
   └── Repository Bean
```

وعندما تحتاج Bean:

```
context.getBean(Car.class)
```

Spring يعطيك الـ object.

---

# طرق إنشاء Beans في Spring

## 1️⃣ باستخدام Component

```java
@Component
class Car {}
```

---

## 2️⃣ باستخدام Configuration

```java
@Configuration
class AppConfig {

    @Bean
    public Engine engine(){
        return new PetrolEngine();
    }

}
```

---

# Lifecycle للـ Bean (باختصار)

عندما يبدأ التطبيق:

```
1. Container starts
2. Bean created
3. Dependencies injected
4. Bean ready to use
5. Container shutdown → Bean destroyed
```

---

# الخلاصة الاحترافية

Bean هو:

```
Object
created by Spring
managed by Spring
stored in IoC Container
used for Dependency Injection
```

الدياجرام النهائي:

```
Developer
   │
   │ writes classes
   ▼
Spring IoC Container
   │
   ├── creates Beans
   ├── injects dependencies
   └── manages lifecycle
```

---

لو حابب، أشرح لك بعدها **نقطة خطيرة في Spring الناس بتتلخبط فيها جدًا**:

الفرق بين

```
@Component
@Bean
```

وده من أشهر أسئلة **Spring interviews**.