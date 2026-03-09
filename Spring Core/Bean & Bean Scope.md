

---

# أولاً: ما هو الـ Bean في Spring ؟

ببساطة:

**Bean = Object يتم إنشاؤه وإدارته بواسطة Spring Container**

يعني بدل ما أنت تعمل:

```java
UserService service = new UserService();
```

Spring هو الذي يقوم بإنشاء الـ object وإدارته.

### التعريف الرسمي

**Bean هو أي Object يقوم Spring Container بإنشائه وإدارته والتحكم في دورة حياته.**

---

# مثال بسيط

لو عندك كلاس:

```java
public class UserService {
    public void print() {
        System.out.println("User Service");
    }
}
```

لكي تجعل Spring ينشئ منه Bean تضيف Annotation مثل:

```java
@Service
public class UserService {
    public void print() {
        System.out.println("User Service");
    }
}
```

أو

```java
@Component
public class UserService {
}
```

هنا Spring سيقوم بـ:

1. إنشاء Object من الكلاس
    
2. تخزينه داخل **Application Context**
    
3. إعطائه للـ classes الأخرى عند الحاجة
    

---

# كيف نحصل على Bean ؟

من خلال **ApplicationContext**

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

UserService service = context.getBean(UserService.class);

service.print();
```

هنا:

Spring هو الذي أنشأ الـ object وليس أنت.

---

# دورة حياة الـ Bean (باختصار)

عندما يبدأ Spring:

1️⃣ يقرأ الـ configuration  
2️⃣ ينشئ Beans  
3️⃣ يعمل Dependency Injection  
4️⃣ يستخدم الـ Beans  
5️⃣ عند إغلاق التطبيق يتم تدميرها

---

# ما هو Bean Scope ؟

الـ **Scope** يحدد:

**كم Object سيتم إنشاؤه من هذا الـ Bean**

هل:

- Object واحد فقط
    
- Object جديد كل مرة
    
- Object لكل request
    
- Object لكل session
    

---

# أنواع Bean Scope

## 1️⃣ Singleton (الافتراضي)

أكثر نوع مستخدم.

**Object واحد فقط في التطبيق كله**

```
Application
   |
   |---- UserService (نفس الـ object لكل الاستخدامات)
```

### مثال

```java
@Service
@Scope("singleton")
public class UserService {
}
```

لكن هذا هو الافتراضي أصلاً.

### مثال توضيحي

لو طلبت الـ bean مرتين:

```java
UserService s1 = context.getBean(UserService.class);
UserService s2 = context.getBean(UserService.class);
```

النتيجة:

```
s1 == s2  -> true
```

نفس الـ object.

---

## 2️⃣ Prototype

كل مرة تطلب الـ bean  
Spring ينشئ **object جديد**

### مثال

```java
@Component
@Scope("prototype")
public class UserService {
}
```

### اختبار

```java
UserService s1 = context.getBean(UserService.class);
UserService s2 = context.getBean(UserService.class);
```

النتيجة:

```
s1 == s2 -> false
```

Objects مختلفة.

---

## 3️⃣ Request Scope

يستخدم في **Web Applications**

كل **HTTP Request** يحصل على object خاص به.

```
Request 1 -> Bean 1
Request 2 -> Bean 2
```

مثال:

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST)
public class RequestBean {
}
```

---

## 4️⃣ Session Scope

كل **User Session** لها object خاص بها.

```
User A session -> Bean A
User B session -> Bean B
```

مثال:

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION)
public class SessionBean {
}
```

---

# ملخص سريع

|Scope|ماذا يحدث|
|---|---|
|Singleton|Object واحد في التطبيق|
|Prototype|Object جديد كل مرة|
|Request|Object لكل HTTP request|
|Session|Object لكل user session|

---

# متى نستخدم كل Scope ؟

### Singleton (الأكثر استخداماً)

مثل:

- Service
    
- Repository
    
- Controller
    

لأنهم **Stateless**

---

### Prototype

عندما تحتاج:

```
Object جديد كل مرة
```

مثال:

- Objects تحتوي state
    
- Objects مؤقتة
    

---

### Request / Session

في تطبيقات الويب فقط.

---

# مثال كامل بسيط

```java
@Component
@Scope("singleton")
public class Engine {
    public Engine(){
        System.out.println("Engine Created");
    }
}
```

```java
public class MainApp {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        Engine e1 = context.getBean(Engine.class);
        Engine e2 = context.getBean(Engine.class);

        System.out.println(e1 == e2);
    }
}
```

النتيجة:

```
Engine Created
true
```

لأن **singleton**

---

# معلومة مهمة في الانترفيو

**Spring Beans are singleton by default**

---

# سؤال مشهور في الانترفيو

ما الفرق بين:

```
singleton vs prototype
```

الإجابة:

|Singleton|Prototype|
|---|---|
|Object واحد|Object جديد كل مرة|
|Default في Spring|يجب تحديده|
|أفضل للأداء|يستهلك memory أكثر|

---

لو حابب بعد كده أشرحلك كمان:

- **Bean Lifecycle (خطوة بخطوة)**
    
- الفرق بين  
    `@Component vs @Service vs @Repository`
    
- إزاي Spring يعمل **Bean creation internally**
    

وهم من أهم أسئلة **Spring interviews**.