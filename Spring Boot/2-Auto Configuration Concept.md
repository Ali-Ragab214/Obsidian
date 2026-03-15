

---

# ما هو Auto Configuration؟

**Auto Configuration** يعني:

> Spring Boot يقوم بعمل **configuration تلقائي للتطبيق** بناءً على ال dependencies الموجودة في المشروع.

يعني بدل ما تكتب إعدادات كتير،  
Spring Boot **يخمّن الإعدادات المناسبة ويعملها لوحده**.

**What is the difference between Spring Framework and Spring Boot?**

✅ **الإجابة:**

| Spring Framework   | Spring Boot        |
| ------------------ | ------------------ |
| framework أساسي    | مبني فوق Spring    |
| configuration كثير | configuration قليل |
| setup يدوي         | auto configuration |

---

# الفكرة ببساطة

Spring Boot ينظر إلى:

- **dependencies** الموجودة في المشروع
    
- **properties** في `application.properties`
    
- **libraries** الموجودة
    

وبناءً على ذلك يقرر:

- ما الذي يجب تشغيله
    
- وما الذي يجب تهيئته
    

---

# مثال عملي

لو أضفت dependency:

```xml
spring-boot-starter-web
```

Spring Boot تلقائيًا يعمل:

1️⃣ يشغل **Embedded Tomcat**  
2️⃣ يجهز **Spring MVC**  
3️⃣ يجهز **JSON converter**  
4️⃣ يجهز **DispatcherServlet**

بدون أي configuration منك.

---

# مثال آخر (Database)

لو أضفت:

```xml
spring-boot-starter-data-jpa
mysql-connector
```

Spring Boot تلقائيًا يعمل:

- إنشاء **DataSource**
    
- إعداد **Hibernate**
    
- إنشاء **EntityManager**
    

كل اللي عليك فقط:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/app
spring.datasource.username=root
spring.datasource.password=123
```

---

# كيف يعرف Spring Boot ماذا يفعل؟

Spring Boot يعتمد على:

### `@EnableAutoConfiguration`

وهذا annotation جزء من:

```java
@SpringBootApplication
```

يعني لما تكتب:

```java
@SpringBootApplication
```

فأنت فعليًا فعلت:

- Component Scan
    
- Configuration
    
- **Auto Configuration**
    

---

# ماذا يحدث داخليًا؟

Spring Boot يحتوي على **Classes كثيرة مسؤولة عن Auto Configuration** مثل:

```
DataSourceAutoConfiguration
WebMvcAutoConfiguration
SecurityAutoConfiguration
```

Spring Boot يقرأ ال dependencies  
ويقرر أي configuration class يجب تشغيله.

---

# مثال

لو لم تضف dependency للويب:

❌ لن يتم تشغيل Tomcat  
❌ لن يتم إعداد REST

لأن Auto Configuration لن يفعل Web Configuration.

---

# الزتونة (مهم للانترفيو) 🔥

**Auto Configuration في Spring Boot هي آلية تقوم تلقائيًا بإعداد التطبيق بناءً على ال dependencies الموجودة في المشروع.**

---

لو حابب نمشي **بالترتيب الصح اللي بيخلّي Spring Boot مفهوم بعمق** فالتوبيك اللي بعده المفروض يكون:

**Spring Boot Starters**

وده مهم جدًا لأن:

> **Starters هي التي تجعل Auto Configuration يعمل أصلاً.**