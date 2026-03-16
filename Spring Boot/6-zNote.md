

```
this is the ioc Container for spring =>
ApplicationContext
      │
      ├── EmailNotificationService
      ├── SmsNotificationService
      └── NotificationManager
```

السؤال ده ممتاز لأنه يوضح الفرق بين **Java العادية** وازاي Spring Framework يدير الـ objects.

خلينا نقارن السطرين.

---

# 1️⃣ الكود الأول (Spring IoC)

```java
NotificationManager manager =
        context.getBean(NotificationManager.class);
```

هنا أنت **بتطلب object من Spring IoC Container**.

يعني:

1. Spring Boot شغّل التطبيق
    
2. أنشأ **IoC Container**
    
3. عمل **Component Scan**
    
4. أنشأ الـ beans
    

بعد كده أنت بتقول:

```text
يا Spring اديني الـ Bean من نوع NotificationManager
```

Spring يرجع لك **object هو اللي أنشأه**.

والـ dependency injection حصل تلقائي.

يعني Spring عمليًا عمل:

```java
new NotificationManager(new EmailNotificationService());
```

لكن **بشكل تلقائي**.

---

# 2️⃣ الكود الثاني (Java عادية)

```java
NotificationManager manager =
        new NotificationManager(new EmailNotificationService());
```

هنا:

- أنت اللي بتنشىء object
    
- أنت اللي بتحدد الـ dependency
    
- مفيش IoC
    
- مفيش Dependency Injection
    
- مفيش Container
    

يعني الكود **مربوط بقوة** بـ `EmailNotificationService`.

---

# الفرق الحقيقي

||Spring IoC|Java عادية|
|---|---|---|
|Object Creation|Spring|أنت|
|Dependency Injection|تلقائي|يدوي|
|Loose Coupling|نعم|لا|
|Bean Lifecycle|Spring يديره|أنت|
|Testing|أسهل|أصعب|

---

# مثال يوضح قوة Spring

لو عايز تغير من Email إلى SMS.

في Java العادية:

```java
new NotificationManager(new SmsNotificationService());
```

لازم تغير الكود.

لكن مع Spring:

```java
@Primary
SmsNotificationService
```

بس.

مش هتغير أي كود في:

```java
NotificationManager
```

---

# الفكرة الأساسية

Spring يطبق مبدأ اسمه:

```
Inversion of Control (IoC)
```

بدل ما التطبيق يتحكم في إنشاء الـ objects،  
**الـ framework هو اللي يتحكم**.

---

# تصور بسيط

بدون Spring:

```
Application
   │
   ▼
new NotificationManager()
   │
   ▼
new EmailNotificationService()
```

مع Spring:

```
Application
   │
   ▼
Spring IoC Container
   │
   ├── NotificationManager
   ├── EmailNotificationService
   └── SmsNotificationService
```

وأنت بس تطلب:

```
getBean()
```

---

بص عندنا مشكله دلوقت وهنفهم حلها 

```
package com.notification.notificationapp;  
  
public interface INotificationService {  
    void sendNotification(String message);  
  
}


```
```

```



```
package com.notification.notificationapp;  
  
import org.springframework.context.annotation.Primary;  
import org.springframework.stereotype.Service;  
  
@Service  
public class EmailNotificationService  implements  INotificationService{  
    @Override  
    public void sendNotification(String message) {  
        System.out.println("Email :" + message);  
    }  
}
```



```
package com.notification.notificationapp;  
  
import org.springframework.stereotype.Service;  
  
@Service  
public class SmsNotificationService implements  INotificationService{  
    @Override  
    public void sendNotification(String message) {  
        System.out.println("SMS :" +message);  
    }  
}
```



```
package com.notification.notificationapp;  
  
import org.springframework.stereotype.Component;  
  
@Component  
public class NotificationManager {  
    final private INotificationService notificationService;  
     public NotificationManager(INotificationService notificationService)  
     {  
         this.notificationService= notificationService;  
     }  
  
    public void notifyUser() {  
         notificationService.sendNotification("Hello from Spring Boot");  
     }  
  
}
```


 المشكله هي ان ال email and sms from type INotificationService يعني كدا لما جينا فال  main هو مش عارف هيرجعلك اي منهم فهيدي   exception  فدي الحلو 



أيوه 👍 لو شلت `@Primary` من `EmailNotificationService` فيه **أكتر من حل** يخلي Spring Framework ميطلعش error، لأن المشكلة الأساسية إن Spring لقى **2 Beans من نفس النوع** (`INotificationService`) ومش عارف يختار واحد.

خلينا نشوف الحلول الأشهر.

---

# 1️⃣ الحل الأشهر: `@Qualifier`
Best Practice 
تحدد بنفسك أنهي Bean يتاخد.

في `NotificationManager`:

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class NotificationManager {

    private final INotificationService notificationService;

    public NotificationManager(
            @Qualifier("emailNotificationService")
            INotificationService notificationService) {

        this.notificationService = notificationService;
    }

    public void notifyUser() {
        notificationService.sendNotification("Hello from Spring Boot");
    }
}
```

Spring هنا هيختار:

```
emailNotificationService
```

اسم الـ Bean بيكون **اسم الكلاس بحرف صغير في الأول**.

---

# 2️⃣ الحل الثاني: تغيير اسم الـ Bean

ممكن تسمي الـ Bean بنفسك.

مثال:

```java
@Service("emailService")
public class EmailNotificationService implements INotificationService {
```

وبعدين تستخدمه:

```java
@Qualifier("emailService")
```

---

# 3️⃣ الحل الثالث: حقن كل الخدمات في List

دا الافضل 
#but here will show u all types from the interface 

ممكن Spring يديك **كل implementations** مرة واحدة.

```java
@Component
public class NotificationManager {

    private final List<INotificationService> services;

    public NotificationManager(List<INotificationService> services) {
        this.services = services;
    }

    public void notifyUser() {

        for (INotificationService service : services) {
            service.sendNotification("Hello from Spring Boot");
        }

    }
}
```

هنا Spring هيحقن:

```
EmailNotificationService
SmsNotificationService
```

---

# 4️⃣ الحل الرابع: `ApplicationContext`

تجيب الـ bean بنفسك:

```java
INotificationService service =
        context.getBean("emailNotificationService", INotificationService.class);
```

لكن ده **مش Best Practice**.

---

# مقارنة الحلول

|الحل|الأفضلية|
|---|---|
|`@Qualifier`|⭐ الأفضل|
|`@Primary`|جيد|
|`List<INotificationService>`|مفيد لو عايز كل الخدمات|
|`ApplicationContext`|أقل استخدام|

---

# نصيحة للـ Interviews

في انترفيوهات Spring Framework غالبًا يسألوك:

```
What happens if multiple beans of the same type exist?
```

الإجابة:

```
Spring throws NoUniqueBeanDefinitionException
```

والحل:

```
@Primary or @Qualifier
```

---

لو حابب، أوريك **حركة قوية جدًا في Spring Core**:  
إزاي Spring يعرف **يختار Bean معين تلقائيًا حسب اسم الـ variable** حتى من غير `@Qualifier` (دي حاجة ناس كتير متعرفهاش).