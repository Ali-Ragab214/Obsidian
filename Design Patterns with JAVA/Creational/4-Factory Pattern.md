

---

# 🏭 Factory Design Pattern – شرح شامل

## 📌 الفكرة الأساسية

بدل ما تكتب: 

```java
new Object()
```

نخلي **كلاس مسؤول عن إنشاء الـ objects** بدل الكود العادي. 

اسم الفكرة:

> **Encapsulate Object Creation**

يعني نخبي عملية إنشاء الكائنات في مكان واحد.

---

# 🤯 المشكلة قبل Factory

تخيل عندك سيستم Notifications:

- EmailNotification
    
- SMSNotification
    
- PushNotification
    

لو كتبت كده 👇

```java
if(type.equals("email"))
    notification = new EmailNotification();
else if(type.equals("sms"))
    notification = new SMSNotification();
else
    notification = new PushNotification();
```

## ❌ المشاكل:

1. Tight Coupling
    
2. كل ما نضيف نوع جديد نعدّل الكود
    
3. يخالف Open/Closed Principle
    
4. كود صعب الصيانة
    

---

# 💡 الحل = Factory Pattern

بدل ما الكود ينشئ objects بنفسه  
نخلي **Factory Class** هي اللي تنشئهم.

---

# 🧠 مكونات الـ Pattern

1️⃣ Product (interface)  
2️⃣ Concrete Products (implementations)  
3️⃣ Factory (object creator)

---

# 👨‍💻 مثال كامل خطوة خطوة

## 1️⃣ Product Interface

```java
interface Notification {
    void send(String message);
}
```

---

## 2️⃣ Concrete Products

```java
class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SMSNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class PushNotification implements Notification {
    public void send(String message) {
        System.out.println("Push: " + message);
    }
}

class CustomNotification implements Notification{
public void send(String message)
{
System.out.println("Custom notification");
}
}
```

---

## 3️⃣ Factory Class (قلب الباترن ❤️)

```java
class NotificationFactory {

    public static Notification createNotification(String type) {

        if(type.equalsIgnoreCase("email"))
            return new EmailNotification();

        if(type.equalsIgnoreCase("sms"))
            return new SMSNotification();

        if(type.equalsIgnoreCase("push"))
            return new PushNotification();

        throw new IllegalArgumentException("Unknown type");
    }
}
```

---

## 4️⃣ استخدام الباترن

```java
public class Main {
    public static void main(String[] args) {

        Notification notification =
            NotificationFactory.createNotification("email");

        notification.send("Hello Ali");
    }
}
```

---

# 🔥 إيه اللي حصل هنا؟

الـ Main:

- لا يعرف Email ولا SMS
    
- لا يستخدم new
    
- يعتمد فقط على interface
    

ده اسمه:

> Loose Coupling ❤️

---

# 🆚 قبل vs بعد Factory

| بدون Factory   | باستخدام Factory      |
| -------------- | --------------------- |
| new everywhere | creation في مكان واحد |
| تعديل صعب      | سهل التوسعه           |
| Tight coupling | Loose coupling        |
| مخالف OCP      | يحقق OCP              |

---

# 🎯 إمتى أستخدم Factory؟

استخدمه عندما:

- عندك أنواع كثيرة من نفس الـ interface
    
- نوع الـ object بيتحدد runtime
    
- عايز تفصل creation عن business logic
    

أمثلة حقيقية:

- Payments
    
- Notifications
    
- Database drivers
    
- Loggers
    
- UI components
    

---

# 🧠 أهم مزايا Factory

1️⃣ Decoupling  
2️⃣ قابل للتوسعة  
3️⃣ نظافة الكود  
4️⃣ يحقق SOLID (خصوصاً OCP & DIP)

---

# 🎤 أسئلة انترفيو + الإجابات

---

## 💬 Q1: ما هو Factory Pattern؟

**الإجابة المثالية:**

> Pattern ينقل مسؤولية إنشاء objects إلى class منفصل بدلاً من استخدام new مباشرة، مما يقلل الترابط ويزيد قابلية التوسعة.

---

## 💬 Q2: ما المشكلة في استخدام new مباشرة؟

- Tight coupling
    
- صعب التوسعة
    
- يخالف OCP
    

---

## 💬 Q3: متى تستخدم Factory؟

- عند وجود multiple implementations لنفس interface
    
- عند تحديد النوع runtime
    

---

## 💬 Q4: ما الفرق بين Factory و Constructor؟

# Constructor ينشئ object لنفس الكلاس  
# Factory يمكنه إنشاء **أنواع مختلفة**

---

## 💬 Q5: ما علاقة Factory بـ SOLID؟

- يحقق OCP
    
- يحقق DIP
    

---

## 💬 Q6: أعطني مثال واقعي

- Payment systems
    
- Logging
    
- Database connections
    

---

لو حابب بعدها نشرح **Factory Method الحقيقي (بـ inheritance)** لأنه المستوى اللي بعد ده مباشرة 👌

---
---
---

this is called simple factory : =>

```java 
class ShapeFactory
{
public static Shape create(String type)
{
return switch(type)
 {
case "circle" -> new Circle();
case "square"->new Square();
default -> throw new IllegalArgumentException();
    };
  }
}
Shape s = ShapeFactory.create("circle");
```