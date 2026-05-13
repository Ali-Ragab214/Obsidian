## Dependency Injection vs Composition in Java

---

### 1. التعريف البسيط

**Composition** = أنت بتبني الأوبجكت جوه الكلاس نفسه 
**Dependency Injection** = حد تاني بيديلك الأوبجكت من برا

---

### 2. Composition

الكلاس بيخلق الـ dependency بنفسه ويتحكم فيها

```java
public class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine(); // ❌ Car بتعمل Engine بنفسها
    }
}
```

**المشكلة:**

- لو عايز تغير نوع الـ Engine؟ لازم تعدل جوه Car
- مش ممكن تعمل Unit Test بسهولة
- الكلاسات مربوطين ببعض بشكل صعب **(Tight Coupling)**

---

### 3. Dependency Injection

حد تاني بيديلك الـ dependency من برا

```java
public class Car {
    private Engine engine;

    public Car(Engine engine) { // ✅ بتستقبل Engine من برا
        this.engine = engine;
    }
}

// الاستخدام
Engine v8 = new V8Engine();
Car myCar = new Car(v8); // تقدر تحط أي engine
```

**الفايدة:**

- تقدر تبدل الـ Engine من غير ما تلمس Car
- سهل جداً في الـ Testing
- **Loose Coupling**

---

### 4. أنواع الـ DI

**Constructor Injection** ← الأفضل ✅

```java
public class Car {
    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

**Setter Injection**

```java
public class Car {
    private Engine engine;

    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```

**Field Injection** ← في Spring بس، مش الأفضل ⚠️

```java
@Component
public class Car {
    @Autowired
    private Engine engine; // Spring بيحقنها أوتوماتيك
}
```

---

### 5. مثال واقعي كامل

```java
// Interface
public interface Engine {
    void start();
}

// Implementations
public class V8Engine implements Engine {
    public void start() {
        System.out.println("V8 engine started 🔥");
    }
}

public class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric engine started ⚡");
    }
}

// Car لا تعرف نوع الـ Engine
public class Car {
    private final Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is moving...");
    }
}

// Main
public class Main {
    public static void main(String[] args) {
        Car gasCar = new Car(new V8Engine());
        gasCar.drive(); // V8 engine started 🔥

        Car electricCar = new Car(new ElectricEngine());
        electricCar.drive(); // Electric engine started ⚡
    }
}
```

---

### 6. المقارنة الكاملة

||Composition|Dependency Injection|
|---|---|---|
|مين بيعمل الأوبجكت؟|الكلاس نفسه|حد تاني من برا|
|Coupling|Tight ❌|Loose ✅|
|Testing|صعب|سهل جداً ✅|
|Flexibility|قليلة|عالية ✅|
|متى تستخدمه؟|الأوبجكت جزء أساسي لا يتغير|الأوبجكت ممكن يتغير أو يتبدل|

---

### 7. امتى تستخدم إيه؟

**استخدم Composition لما:**

- الـ object جزء لا يتجزأ من الكلاس
- مش محتاج تبدله أبداً
- مثال: `Human` و `Heart` ← القلب مش بيتغير

**استخدم DI لما:**

- محتاج مرونة في التبديل
- بتكتب Unit Tests
- بتشتغل على Spring أو أي Framework
- مثال: `UserService` و `UserRepository` ← الـ repo ممكن يتغير

---

### 8. في Spring Boot الواقع

```java
@Service
public class UserService {

    private final UserRepository userRepository; // interface مش class

    @Autowired // أو بدونها في الـ versions الحديثة
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUser(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}
```

Spring بيعمل الـ injection أوتوماتيك، ومش محتاج تعمل `new` بنفسك أبداً.

---

### 9. أسئلة الإنترفيو 🎯

**Q: What is the difference between Composition and DI?**

> Composition creates dependencies internally, DI receives them externally — promoting loose coupling and testability.

**Q: Why is Constructor Injection preferred over Field Injection?**

> It makes dependencies explicit, supports immutability with `final`, and works without a framework.

**Q: How does Spring handle DI?**

> Spring uses an IoC container that manages object creation and injects dependencies automatically via `@Autowired` or constructor injection.

---

عايز أكمل على **IoC Container** أو **DI Frameworks** زي Spring بالتفصيل؟