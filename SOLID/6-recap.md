تمام يا علي، هشرحها بطريقة بسيطة جدًا كأنك في إنترفيو.

# 14. What are the SOLID Principles?

SOLID عبارة عن 5 قواعد تساعدك تكتب كود سهل التعديل والصيانة.

## S — Single Responsibility Principle

**الكلاس يكون له سبب واحد فقط للتغيير.**

❌ مثال غلط:

```java
class UserService {
    void registerUser() {}
    void sendEmail() {}
}
```

الكلاس مسؤول عن:

- تسجيل المستخدم
    
- إرسال الإيميل
    

يعني مسؤوليتين.

✅ الصح:

```java
class UserService {
    void registerUser() {}
}

class EmailService {
    void sendEmail() {}
}
```

كل كلاس له مسؤولية واحدة.

---

## O — Open/Closed Principle

**مفتوح للإضافة ومغلق للتعديل.**

❌

```java
if(type.equals("car")) {
   // ...
}
else if(type.equals("bike")) {
   // ...
}
```

كل مرة تضيف نوع جديد تعدل الكود.

✅

```java
interface Vehicle {
   void move();
}
```

ثم:

```java
class Car implements Vehicle {}
class Bike implements Vehicle {}
class Bus implements Vehicle {}
```

أضف أنواع جديدة بدون تعديل الكود القديم.

---

## L — Liskov Substitution Principle

**أي Child Class يجب أن تحل محل الـ Parent بدون مشاكل.**

❌

```java
class Bird {
   void fly(){}
}

class Penguin extends Bird {
   void fly(){
      throw new RuntimeException();
   }
}
```

البطريق لا يطير.

إذن Penguin ليست Bird مناسبة لهذا التصميم.

---

## I — Interface Segregation Principle

**لا تجبر الكلاس على تنفيذ ميثود لا يحتاجها.**

❌

```java
interface Worker{
   work();
   eat();
}
```

لو عندي Robot:

```java
class Robot implements Worker
```

الروبوت لا يأكل!

---

✅

```java
interface Workable {
   work();
}

interface Eatable {
   eat();
}
```

كل كلاس يطبق ما يحتاجه فقط.

---

## D — Dependency Inversion Principle

**اعتمد على Interface وليس Class مباشر.**

❌

```java
class UserService {
   EmailService emailService =
      new EmailService();
}
```

UserService مربوط بـ EmailService مباشرة.

---

✅

```java
interface MessageService {}

class EmailService
       implements MessageService {}
```

```java
class UserService {

   MessageService service;

   UserService(MessageService service){
      this.service = service;
   }
}
```

يمكنك تغيير Email إلى SMS بسهولة.

---

# 15. Give two real-world examples where SOLID is violated

## Example 1: Restaurant System

❌

```java
class OrderService {

    void createOrder(){}

    void saveOrder(){}

    void sendEmail(){}

    void printInvoice(){}
}
```

المشكلة:

الكلاس يعمل كل شيء.

هذا يخالف:

```text
SRP
```

---

✅ الحل

```java
OrderService
```

لإنشاء الطلب

```java
EmailService
```

للإيميل

```java
InvoiceService
```

للفواتير

كل كلاس له وظيفة واحدة.

---

## Example 2: Payment System

❌

```java
class PaymentService {

   void pay(String type){

      if(type.equals("visa")){
      }

      if(type.equals("paypal")){
      }

      if(type.equals("cash")){
      }
   }
}
```

كلما أضفت طريقة دفع جديدة ستعدل الكود.

هذا يخالف:

```text
OCP
```

---

✅ الحل

```java
interface PaymentMethod {
    void pay();
}
```

```java
class VisaPayment
        implements PaymentMethod {}
```

```java
class PaypalPayment
        implements PaymentMethod {}
```

```java
class CashPayment
        implements PaymentMethod {}
```

لو أضفت:

```java
class VodafoneCashPayment
```

لن تعدل أي كود قديم.

---

### إجابة إنترفيو قصيرة

> SOLID is a set of five object-oriented design principles:
> 
> - S: Single Responsibility Principle
>     
> - O: Open/Closed Principle
>     
> - L: Liskov Substitution Principle
>     
> - I: Interface Segregation Principle
>     
> - D: Dependency Inversion Principle
>     
> 
> They help developers build maintainable, scalable, and loosely coupled applications.

احفظ الجملة الأخيرة دي لأنها تُقال كثيرًا في الإنترفيوهات.