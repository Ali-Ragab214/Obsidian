## مثال كامل - IoC + ApplicationContext + DI في Spring Boot 🚀

---

### هيكل المشروع 📁

```
src/
└── main/
    └── java/
        └── com/example/
            ├── Main.java
            ├── Engine.java          (Interface)
            ├── PetrolEngine.java    (Implementation)
            ├── ElectricEngine.java  (Implementation)
            └── Car.java             (Component)
```

---

### 1️⃣ الـ Interface

```java
package com.example;

public interface Engine {
    void start();
    String getFuelType();
}
```

**ليه Interface؟**

```
Car هتتكلم مع Engine بس
مش هتعرف PetrolEngine أو ElectricEngine
= Loose Coupling ✅
```

---

### 2️⃣ الـ Implementations 

```java
package com.example;

import org.springframework.stereotype.Component;

@Component  // Spring اعمل Bean من الكلاس دي
public class PetrolEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Petrol Engine Started 🔥");
    }

    @Override
    public String getFuelType() {
        return "Petrol ⛽";
    }
}
```

```java
package com.example;

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Primary;

@Component
@Primary  // لو فيه أكتر من Engine - استخدم الأولوية دي
public class ElectricEngine implements Engine {

    @Override
    public void start() {
        System.out.println("Electric Engine Started ⚡");
    }

    @Override
    public String getFuelType() {
        return "Electricity 🔋";
    }
}
```

---

### 3️⃣ الـ Car

```java
package com.example;

import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;

@Component
public class Car {

    private Engine engine;  // مش عارف النوع!

    @Autowired  // Spring حقن الـ Engine تلقائي!
    public Car(Engine engine) {
        this.engine = engine;
        System.out.println("Car Bean Created! 🚗");
        System.out.println("Engine Injected: " + engine.getClass().getSimpleName());
    }

    public void drive() {
        System.out.println("\n--- Car is Starting ---");
        engine.start();
        System.out.println("Fuel Type: " + engine.getFuelType());
        System.out.println("Car is Driving! 🚗💨");
    }
}
```

---

### 4️⃣ الـ Main

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class Main {
    public static void main(String[] args) {

        // هنا بيتعمل الـ IoC Container!
        ApplicationContext context = 
            SpringApplication.run(Main.class, args);

        System.out.println("\n====== Container Ready ======");

        // اطلب الـ Bean من الـ Container
        Car car = context.getBean(Car.class);

        // استخدمها!
        car.drive();

        System.out.println("\n====== All Beans in Container ======");
        // اطبع كل الـ Beans اللي الـ Container عملها
        String[] beans = context.getBeanDefinitionNames();
        for (String bean : beans) {
            if (!bean.contains("org.springframework")) {
                System.out.println("Bean: " + bean);
            }
        }
    }
}
```

---

### Output ✅

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

====== Container Ready ======
Car Bean Created! 🚗
Engine Injected: ElectricEngine   ← @Primary اتحقنت

====== Car Starting ======
--- Car is Starting ---
Electric Engine Started ⚡
Fuel Type: Electricity 🔋
Car is Driving! 🚗💨

====== All Beans in Container ======
Bean: petrolEngine
Bean: electricEngine
Bean: car
```

---

### إيه اللي حصل خطوة بخطوة؟ 🔍

```
1. Main.java اشتغل
        ↓
2. SpringApplication.run() اتنفذ
        ↓
3. IoC Container اتعمل
        ↓
4. Spring لف على كل الكلاسات
   ولقى @Component على:
   ✅ PetrolEngine
   ✅ ElectricEngine
   ✅ Car
        ↓
5. عمل Bean لكل واحدة
        ↓
6. Car محتاجة Engine في الـ Constructor
   Spring لقى @Autowired
        ↓
7. شاف إن ElectricEngine عندها @Primary
   حقنها في Car تلقائي!
        ↓
8. Car جاهزة للاستخدام ✅
```

---

### لو عايز تغير لـ PetrolEngine؟ 🔄

```java
// مش هتعدل في Car خالص!
// بس شيل @Primary من ElectricEngine
// وحطها على PetrolEngine

@Component
@Primary  // ← نقلتها هنا
public class PetrolEngine implements Engine {
    ...
}
```

**Output:**

```
Car Bean Created! 🚗
Engine Injected: PetrolEngine   ← اتغيرت!

--- Car is Starting ---
Petrol Engine Started 🔥
Fuel Type: Petrol ⛽
Car is Driving! 🚗💨
```

**Car.java مش اتغيرت ولا سطر! 🎉**

---

### الصورة الكاملة 🎯

```
┌─────────────────────────────────────────────┐
│           IoC Container                     │
│                                             │
│  @Component        @Component               │
│  PetrolEngine      ElectricEngine(@Primary) │
│       │                  │                  │
│       └────────┬─────────┘                  │
│                ↓                            │
│           @Autowired                        │
│           Car ← Engine injected!            │
│                                             │
│  انت:  context.getBean(Car.class)           │
│  Spring: اتفضل Car جاهزة! ✅               │
└─────────────────────────────────────────────┘
```

---

### خلاصة الـ Annotations 📝

|Annotation|معناها|
|---|---|
|`@Component`|اعمل Bean من الكلاس دي|
|`@Autowired`|حقن الـ Dependency تلقائي|
|`@Primary`|لو فيه أكتر من Implementation استخدم دي|
|`@SpringBootApplication`|ابدأ الـ IoC Container|

عايز نكمل على **@Qualifier** لما يكون عندك أكتر من Engine وعايز تختار بالاسم؟ 🎯