# very important

---

## الحقيقة الأساسية الأول

> **Composition و DI مش متنافسين — هما بيكملوا بعض**
واصلا هما بيستخدموا مع بعض علطول 
||Composition|DI|
|---|---|---|
|بتجاوب على إيه؟|**هيكل الكود** — إيه اللي جوه الكلاس؟|**إزاي الأوبجكت بيتعمل؟**|
|السؤال|Car HAS-A Engine؟|مين اللي بيعمل الـ Engine دي؟|

---

## تخيل السيناريو ده

```
أنت عندك: Car تحتاج Engine
```

### الحالة 1: Composition بدون DI ❌

```java
public class Car {
    private Engine engine;

    public Car() {
        this.engine = new V8Engine(); // Car بتعمل الـ Engine بنفسها
    }
}
```

**المشكلة:**

- Car مربوطة بـ V8Engine للأبد
- مش تقدر تحط ElectricEngine
- مش تقدر تعمل Unit Test

---

### الحالة 2: Composition + DI ✅

```java
public class Car {
    private final Engine engine; // Composition: Car HAS-A Engine

    public Car(Engine engine) {  // DI: Engine بتيجي من برا
        this.engine = engine;
    }
}
//we use composition with DI 

// الاستخدام
Car gasCar      = new Car(new V8Engine());
Car electricCar = new Car(new ElectricEngine());
Car testCar     = new Car(new MockEngine()); // للـ Testing
```

**يعني:**

- **Composition** قررت إن Car لازم يبقالها Engine
- **DI** قررت إن مين بيجيب الـ Engine دي

---

## الفرق التاني: متى تستخدم إيه؟

### استخدم Composition بدون DI لما:

الـ object **جزء لا يتجزأ** ومش هيتغير أبداً

```java
public class Person {
    private final Heart heart;
    private final Brain brain;

    public Person() {
        this.heart = new Heart(); // ✅ منطقي
        this.brain = new Brain(); // ✅ منطقي
    }
    // القلب والمخ مش بيتغيروا من برا
}
```

---

### استخدم Composition + DI لما:

الـ object **ممكن يتغير** أو محتاج **مرونة**

```java
public class NotificationService {
    private final MessageSender sender; // Composition

    public NotificationService(MessageSender sender) { // DI
        this.sender = sender;
    }

    public void notify(String msg) {
        sender.send(msg);
    }
}

// تقدر تبعت Email أو SMS أو WhatsApp
new NotificationService(new EmailSender());
new NotificationService(new SmsSender());
new NotificationService(new WhatsAppSender());
```

---

## المثال الكامل اللي بيوضح الفرق

```java
// ===============================
// Interfaces
// ===============================
public interface Engine {
    void start();
}

public interface GPS {
    String getLocation();
}

// ===============================
// Implementations
// ===============================
public class V8Engine implements Engine {
    public void start() {
        System.out.println("V8 روعة 🔥");
    }
}

public class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric هادي ⚡");
    }
}

public class GoogleGPS implements GPS {
    public String getLocation() {
        return "Google: Cairo, Egypt 📍";
    }
}

// ===============================
// Car — Composition + DI معاً
// ===============================
public class Car {
    private final Engine engine;  // Composition: Car HAS-A Engine
    private final GPS gps;        // Composition: Car HAS-A GPS

    // DI: الاتنين بييجوا من برا
    public Car(Engine engine, GPS gps) {
        this.engine = engine;
        this.gps = gps;
    }

    public void drive() {
        engine.start();
        System.out.println("Location: " + gps.getLocation());
    }
}

// ===============================
// Main
// ===============================
public class Main {
    public static void main(String[] args) {

        // تركيبة 1
        Car sportsCar = new Car(new V8Engine(), new GoogleGPS());
        sportsCar.drive();
        // V8 روعة 🔥
        // Location: Google: Cairo, Egypt 📍

        // تركيبة 2
        Car electricCar = new Car(new ElectricEngine(), new GoogleGPS());
        electricCar.drive();
        // Electric هادي ⚡
        // Location: Google: Cairo, Egypt 📍
    }
}
```

---

## في Spring Boot — الصورة الكاملة

```java
// ===============================
// Repository Layer
// ===============================
public interface UserRepository extends JpaRepository<User, Long> { }

// ===============================
// Service Layer
// ===============================
@Service
public class UserService {

    // Composition: UserService HAS-A UserRepository
    private final UserRepository userRepository;

    // DI: Spring بيحقن الـ Repository أوتوماتيك
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findById(Long id) {
        return userRepository.findById(id).orElseThrow();
    }
}

// ===============================
// Controller Layer
// ===============================
@RestController
public class UserController {

    // Composition: Controller HAS-A Service
    private final UserService userService;

    // DI: Spring بيحقن الـ Service أوتوماتيك
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

---

## ملخص نهائي في جملة واحدة

```
Composition  =  قررت إن الكلاس "يحتوي على" حاجة
DI           =  قررت إن الحاجة دي "تيجي من برا"
```

| السؤال                   | الإجابة                                   |
| ------------------------ | ----------------------------------------- |
| Car لازم يبقالها Engine؟ | **Composition**                           |
| مين بيعمل الـ Engine دي؟ | **DI**                                    |
| أنهو أحسن؟               | **الاتنين مع بعض** ✅                      |
| في Spring بستخدم إيه؟    | **Composition + Constructor DI دايماً** ✅ |
| في Spring بستخدم         | Composition + DI => Always                |

---

