

**Spring Boot** هو **Framework مبني فوق Spring Framework** هدفه إنه يخليك تبني **Java applications بسرعة وبإعدادات أقل**.

ببساطة:~~~

**Spring Boot = Spring Framework + Auto Configuration + Ready to run application**
Spring Framework = مجموعة Tools ومكتبات تساعدك تبني تطبيقات Java كبيرة بطريقة منظمة.

يعني بدل ما تضيع وقت في إعدادات كتير في Spring،  
Spring Boot بيعملها **automatic**.



Structure
GymApp/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/gym/GymApp/
│   │   │       └── GymAppApplication.java  ← الـ Main Class
│   │   └── resources/
│   │       └── application.properties      ← الـ Config
├── pom.xml  ← الـ Dependencies





---

# المشكلة قبل Spring Boot

قبل Spring Boot لو كنت بتستخدم **Spring Framework** بس، كنت بتحتاج:

- تكتب **XML configuration كتير**
    
- تعمل **setup للـ server**
    
- تضبط dependencies بنفسك
    
- تضبط database configuration
    
- تضبط logging
    

يعني مشروع بسيط ممكن ياخد **وقت طويل في الإعدادات بس**.

# ما هو Configuration؟

**Configuration** يعني:

> **إعدادات تخلي التطبيق يعرف يشتغل إزاي.**

يعني أنت بتقول للتطبيق:

- يستخدم أي Database
    
- يشتغل على أي Port
    
- يستخدم أي Classes
    
- يربط أي Objects ببعض
- 
# مثال بسيط خارج Spring

لو عندك برنامج Java عادي وعايز يتصل بقاعدة بيانات.

لازم تحدد:

`database url username password driver`

دي كلها **configuration**.

# مثال آخر

لو عايز تحدد السيرفر يشتغل على port معين:

`server.port=8081`

ده برضو **configuration**.
كل ده اسمه **Configuration**.


# أنواع Configuration في Spring

أهم 3 أنواع:

### 1️⃣ XML Configuration (قديم)

`<bean id="service" class="UserService"/>`

---

### 2️⃣ Java Configuration

`@Configuration public class AppConfig {      @Bean     public UserService userService(){         return new UserService();     }  }`

---

### 3️⃣ Properties / YAML

مثل:

`server.port=8080`

---

# الزتونة 🔥

**Configuration = إعدادات تخبر البرنامج كيف يعمل ويربط مكوناته ببعض.**

---

# الحل مع Spring Boot

Spring Boot بيحل المشاكل دي عن طريق:

### 1️⃣ Auto Configuration

Spring Boot بيعمل **configuration تلقائي** حسب ال dependencies الموجودة.

مثال:

لو ضفت dependency بتاع:

- MySQL
    
- JPA
    

Spring Boot تلقائيًا:

- يعمل DataSource
    
- يعمل EntityManager
    
- يجهز Hibernate
    

بدون ما تكتب configuration كتير.

---

### 2️⃣ Starter Dependencies

بدل ما تضيف 10 dependencies، تضيف **dependency واحدة**.

مثال:

```
spring-boot-starter-web
```

ده بيضيف:

- Spring MVC
    
- Jackson
    
- Validation
    
- Embedded server
    

كلهم مرة واحدة.

---

### 3️⃣ Embedded Server

مش محتاج تنزل server خارجي زي:

- Tomcat
    
- Jetty
    

Spring Boot بيشغل **server جواه**.

يعني تشغل التطبيق فقط:

```
Run main method
```

ويشتغل السيرفر.

---

### 4️⃣ Production Ready Features

Spring Boot بيديك features جاهزة للإنتاج مثل:

- health checks
    
- metrics
    
- monitoring
    
- logging
    

عن طريق:

```
spring-boot-starter-actuator
```

---

# مثال بسيط

ده أبسط Spring Boot Application:

```java
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

}
```

لما تشغل الكود:

- Spring يبدأ
    
- السيرفر يشتغل
    
- التطبيق يبدأ
    

---

# مثال Controller

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "Hello Spring Boot";
    }

}
```

لو فتحت:

```
localhost:8080/hello
```

هتلاقي:

```
Hello Spring Boot
```

---

# ليه الشركات بتحب Spring Boot

لأنه:

- سريع في التطوير
    
- يقلل ال configuration
    
- production ready
    
- سهل يعمل REST APIs
    
- سهل integration مع databases
    

عشان كده أغلب **Microservices** في Java معمولة بـ Spring Boot.

---

# الفرق في سطر واحد

|Spring|Spring Boot|
|---|---|
|configuration كتير|configuration قليل|
|setup يدوي|auto configuration|
|server خارجي|embedded server|

---

# الزتونة

Spring Boot هو:

**Tool يخليك تبني Spring Applications بسرعة وبأقل configuration.**

---

لو عايز نكمل **التوبيك اللي بعده** في Spring Boot يبقى:

1️⃣ **Spring Boot Architecture**  
2️⃣ **Spring Boot Starters**  
3️⃣ **Auto Configuration**  
4️⃣ **Application.properties**  
5️⃣ **Embedded Server**

