

**Convention over Configuration** يعني:

> **الإطار (framework) يضع قواعد افتراضية جاهزة، ولو اتبعتها مش هتحتاج تكتب configuration كتير.**
> it assumes logical configuration 

يعني بدل ما تكتب إعدادات لكل حاجة،  
النظام **يفترض إعدادات قياسية (default)**.

لو مشيت عليها → كل شيء يشتغل تلقائيًا.

---

# الفكرة ببساطة

بدل ما تقول للنظام كل تفصيلة:

❌ “استخدم هذا المسار”  
❌ “ابحث عن الكلاسات هنا”  
❌ “استعمل هذه الإعدادات”

النظام يقول لك:

✅ "أنا عندي قواعد جاهزة، امشي عليها وخلاص".

---

# مثال بسيط

في **Spring Boot** لما تنشئ مشروع:

هيكون فيه structure مثل:

```
src/main/java
src/main/resources
```

Spring Boot **يفترض تلقائيًا**:

- الكود في `java`
    
- الإعدادات في `resources`
    
- `application.properties` في `resources`
    

فلا تحتاج تقول له أين يبحث.

هذا هو **Convention**.

---

# مثال آخر

لو كتبت:

```java
@RestController
public class UserController {
}
```

Spring Boot يفترض أن:

- هذا **Controller**
    
- يعالج **HTTP Requests**
    
- يرسل **JSON**
    

بدون configuration.

---

# مثال مع قاعدة البيانات

لو كتبت في:

```
application.properties
```

فقط:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/app
spring.datasource.username=root
spring.datasource.password=123
```

Spring Boot **يفترض تلقائيًا**:

- أنك تريد **DataSource**
    
- تستخدم **Hibernate**
    
- تريد **JPA**
    

بدون إعدادات إضافية.

---

# الفرق بين المفهومين

|||
|---|---|
|Configuration|أنت تكتب الإعدادات|
|Convention|النظام يفترض الإعدادات|

---

# العلاقة مع Auto Configuration

المفهومين مرتبطين جدًا:

- **Convention over Configuration**  
    → يوفر قواعد افتراضية
    
- **Auto Configuration**  
    → يطبق هذه القواعد تلقائيًا
    

كلاهما جزء من فلسفة **Spring Boot**.

---

# مثال بسيط للتوضيح

بدل ما تكتب 50 سطر configuration لتشغيل السيرفر،

يكفي:

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}
```

والباقي يعمل تلقائيًا.

---

# الزتونة (مهم للانترفيو) 🔥

**Convention over Configuration يعني أن framework يضع قواعد افتراضية جاهزة تقلل الحاجة لكتابة configuration.**

---

لو حابب نمشي **بالترتيب الحقيقي لفهم Spring Boot بعمق**، التوبيك الجاي المفروض يكون:

1️⃣ **Spring Boot Starters** (مهم جدًا)  
2️⃣ **Spring Boot Architecture**  
3️⃣ **Application.properties**  
4️⃣ **Embedded Server**

وصدقني **Starters** تحديدًا هي اللي بتخلي Spring Boot سهل جدًا.