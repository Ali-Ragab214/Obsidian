

1️⃣ **Main Class وشرح `@SpringBootApplication`**  


---

# 1️⃣ Main Class في Spring Boot

في أي مشروع Spring Boot هتلاقي كلاس رئيسي اسمه غالبًا:

```java
Application.java
```

مثال:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {

        SpringApplication.run(MyApplication.class, args);

    }

}
```

ده **نقطة البداية للتطبيق**.

يعني لما تشغل المشروع:

```text
Run Application
```

اللي يحصل:

```text
Java starts main()
        ↓
SpringApplication.run()
        ↓
Spring Boot starts
        ↓
Server starts
        ↓
Application ready
```

---

# ماذا يفعل `SpringApplication.run()` ؟

السطر ده:

```java
SpringApplication.run(MyApplication.class, args);
```

يقوم بـ:

1️⃣ تشغيل **Spring Context**  
2️⃣ عمل **Component Scan**  
3️⃣ تشغيل **Auto Configuration**  
4️⃣ تشغيل **Embedded Server**  
5️⃣ تشغيل التطبيق بالكامل

---

# Diagram للفهم

```
main()
  │
  ▼
SpringApplication.run()
  │
  ▼
Start Spring Container
  │
  ▼
Load Beans
  │
  ▼
Start Server
  │
  ▼
Application Ready
```

---

# 2️⃣ ما هي `@SpringBootApplication`

هي أهم Annotation في Spring Boot.

```java
@SpringBootApplication
```

هي في الحقيقة **3 annotations في واحدة**.

يعني:

```java
@SpringBootApplication
```

تعادل:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

# نشرحهم بسرعة

## 1️⃣ `@Configuration`

تعني أن الكلاس يحتوي على **Bean Configuration**.

مثال:

```java
@Bean
public MyService myService(){
    return new MyService();
}
```

---

## 2️⃣ `@ComponentScan`

Spring يبحث عن:

```
@Component
@Service
@Repository
@Controller
```

داخل نفس الباكدج.

مثال:

```
com.app
   │
   ├── controller
   ├── service
   ├── repository
```

Spring سيجدهم تلقائيًا.

---

## 3️⃣ `@EnableAutoConfiguration`

دي أهم ميزة في Spring Boot.

هي التي تقوم بـ:

```text
Auto Config
```

يعني Spring يضبط الأشياء تلقائيًا.

مثال:

لو أضفت dependency:

```
spring-boot-starter-web
```

Spring تلقائيًا يضبط:

```
DispatcherServlet
Jackson
Tomcat
Spring MVC
```

بدون أي configuration.

---

# Diagram

```
@SpringBootApplication
        │
        ▼
 ┌───────────────────────┐
 │   @Configuration      │
 │   @ComponentScan      │
 │   @EnableAutoConfig   │
 └───────────────────────┘
```

---

