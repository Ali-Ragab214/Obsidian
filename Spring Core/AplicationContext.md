تمام يا علي. خلينا نفهم **ApplicationContext** في Spring بشكل مترتب وربطه بكل المفاهيم اللي ذاكرتها: **IoC, Beans, DI, Component**.

---

# ما هو ApplicationContext ؟

في Spring Framework:

> **ApplicationContext هو الـ Implementation الأساسي لـ IoC Container في Spring.**

يعني ببساطة:

```
ApplicationContext = Spring IoC Container
```

هو المسؤول عن:

1. إنشاء الـ **Beans**
    
2. إدارة الـ **Beans**
    
3. عمل **Dependency Injection**
    
4. تخزين الـ Beans
    
5. إعطائك الـ Beans عندما تطلبها
    

---

# الصورة الكبيرة

```
                 ApplicationContext
            (Spring IoC Container)
        ┌────────────────────────────┐
        │                            │
        │        manages beans       │
        │                            │
        └─────────────┬──────────────┘
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
     Car Bean     Engine Bean    Service Bean
        │
        ▼
   Dependency Injection
```

---

# أين نستخدم ApplicationContext ؟

عندما يبدأ البرنامج.

مثال:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext("com.example");
```

ما الذي يحدث هنا؟

Spring يقوم بـ:

```
1. إنشاء IoC Container
2. عمل Component Scan
3. إنشاء Beans
4. عمل Dependency Injection
```

---

# مثال كامل

## Engine

```java
@Component
public class PetrolEngine {

    public void start(){
        System.out.println("Engine started");
    }

}
```

---

## Car

```java
@Component
public class Car {

    private PetrolEngine engine;

    public Car(PetrolEngine engine){
        this.engine = engine;
    }

    public void drive(){
        engine.start();
        System.out.println("Car driving");
    }

}
```

---

# Main Class

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext("com.example");

Car car = context.getBean(Car.class);

car.drive();
```

---

# ماذا يحدث داخل ApplicationContext ؟

## Step 1 — إنشاء Container

```
ApplicationContext created
```

---

## Step 2 — Component Scan

Spring يبحث عن:

```
@Component
```

في الباكيج.

الدياجرام:

```
Project
 ├── Car
 ├── PetrolEngine
 └── Main
```

Spring يجد:

```
@Component
```

---

## Step 3 — إنشاء Beans

```
PetrolEngine Bean
Car Bean
```

الدياجرام:

```
ApplicationContext
    │
    ├── PetrolEngine Bean
    └── Car Bean
```

---

## Step 4 — Dependency Injection

Spring يرى:

```
Car يحتاج PetrolEngine
```

فيحقنه.

الدياجرام:

```
Car Bean
   │
   ▼
PetrolEngine Bean
```

---

# ماذا يفعل getBean ؟

```java
Car car = context.getBean(Car.class);
```

يعني:

```
اذهب إلى Container
وأعطني Bean من نوع Car
```

الدياجرام:

```
Application
   │
   ▼
ApplicationContext
   │
   ▼
returns Car Bean
```

---

# علاقة ApplicationContext بالمفاهيم الأخرى

## 1 — IoC

```
ApplicationContext
    =
IoC Container
```

هو الذي يعكس التحكم.

---

## 2 — Beans

```
Beans stored inside ApplicationContext
```

الدياجرام:

```
ApplicationContext
   │
   ├── Car Bean
   ├── Engine Bean
   └── Service Bean
```

---

## 3 — Dependency Injection

ApplicationContext هو الذي يقوم بـ:

```
inject dependencies
```

مثال:

```
Car → Engine
```

---

# أنواع ApplicationContext

أهم نوعين:

## 1 — AnnotationConfigApplicationContext

نستخدمه عندما نستخدم:

```
@Component
@Configuration
```

مثال:

```java
ApplicationContext context =
new AnnotationConfigApplicationContext("com.example");
```

---

## 2 — ClassPathXmlApplicationContext

لو استخدمت XML.

مثال:

```java
ApplicationContext context =
new ClassPathXmlApplicationContext("beans.xml");
```

---

# الفرق بين BeanFactory و ApplicationContext

كلاهما **IoC Container** لكن:

|BeanFactory|ApplicationContext|
|---|---|
|بسيط|متقدم|
|Lazy loading|eager loading|
|ميزات أقل|ميزات أكثر|

في المشاريع الحديثة نستخدم:

```
ApplicationContext
```

تقريباً دائماً.

---

# الصورة النهائية

```
Developer writes classes
          │
          ▼
     ApplicationContext
  (Spring IoC Container)
          │
          ├── scans components
          ├── creates beans
          ├── injects dependencies
          └── manages lifecycle
```

---

# خلاصة احترافية

ApplicationContext هو:

```
Central container in Spring
```

مسؤول عن:

```
1. Creating Beans
2. Managing Beans
3. Dependency Injection
4. Bean lifecycle
5. Providing beans when requested
```

---

لو حابب يا علي أوريك بعدها **ديجرام واحد كبير يشرح Spring Core بالكامل في صفحة واحدة** (IoC + DI + Beans + ApplicationContext) بطريقة تخليك تربط كل حاجة مع بعض.