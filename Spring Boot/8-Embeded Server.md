
---

# Embedded Servers

قبل Spring Boot كان لازم:

```
Install Tomcat
Deploy WAR
Configure Server
```

عملية طويلة.

---

## Spring Boot حل المشكلة

Spring Boot يأتي مع **Embedded Server**.

يعني السيرفر **موجود داخل التطبيق**.

---

# أشهر Embedded Servers

|Server|الاستخدام|
|---|---|
|**Apache Tomcat**|الافتراضي في Spring Boot|
|**Jetty**|أسرع في بعض الحالات|
|**Undertow**|خفيف وسريع|

---

# كيف يعمل Embedded Server ؟

عند تشغيل التطبيق:

```text
Run Application
       ↓
Spring Boot starts
       ↓
Embedded Tomcat starts
       ↓
Application runs on port 8080
```

---

# مثال Console

عند تشغيل التطبيق سترى:

```
Tomcat started on port(s): 8080
```

يعني:

```
http://localhost:8080
```

التطبيق يعمل.

---

# مثال Controller

```java
@RestController
public class HelloController {

    @GetMapping("/")
    public String hello(){
        return "Hello Spring Boot";
    }

}
```

عند فتح:

```
http://localhost:8080
```

سيعمل السيرفر مباشرة بدون تثبيت Tomcat.

---

# الفرق الكبير

قبل Spring Boot:

```
Build WAR
Install Tomcat
Deploy WAR
Start Server
```

مع Spring Boot:

```
Run Application
```

فقط.

---

# الزتونة (مهم للمراجعة)

### Main Class

هي نقطة تشغيل التطبيق.

```
SpringApplication.run()
```

تشغل Spring Boot بالكامل.

---

### `@SpringBootApplication`

تساوي:

```
@Configuration
@ComponentScan
@EnableAutoConfiguration
```

---

### Embedded Server

Spring Boot يحتوي على سيرفر داخلي مثل:

```
Tomcat
Jetty
Undertow
```

ويعمل التطبيق مباشرة.

---

لو حابب يا علي، أعمل لك بعدها **ريكاب كامل لـ Spring Boot في خريطة واحدة** (حوالي 15 توبيك مرتبين بطريقة يفهمك Spring Boot كله بسرعة).