# Section 02: Spring Boot Fundamentals

> **الملف ده مبني على مذكراتك في مجلد `Spring Boot/` وملاحظاتك حول السيرفرات والـ Servlets في `50-Web Server vs Application Server.md` و `55-Servlet vs Spring.md`.**  
> يغطي الفلسفة الهندسية لـ Spring Boot وكيف يبدأ التطبيق تحت الغطاء.

---

### Q1: What is Spring Boot, and what major problems does it solve compared to Spring Framework?
**Level:** Junior  
**Related to:** Q2 (Auto-configuration), Q3 (Starters), Q5 (Convention over Configuration)

**Answer:**
قبل ظهور Spring Boot، كان العمل مع Spring Framework أشبه بكابوس إعدادات:
- كنت مضطراً لكتابة ملفات XML ضخمة أو كلاسات `@Configuration` لا نهائية لتعريف كل شيء (DataSource, TransactionManager, ViewResolver, DispatcherServlet).
- كنت تضطر لضبط إصدارات المكتبات في `pom.xml` يدوياً ومواجهة جحيم تعارض الإصدارات (**Dependency Hell / JAR Hell**).
- كنت تبني ملف WAR ثم تبحث عن Web Server خارجي (زي Tomcat أو WildFly) وتعمل Deploy يدوي.

**Spring Boot** جاء كطبقة فوق Spring Framework، هدفه تبسيط وتسريع التطوير (**Rapid Application Development - RAD**) بدون التخلي عن قوة Spring Framework.

#### الركائز الأربعة التي بني عليها Spring Boot (من مذكراتك):
1. **Auto-Configuration:** يضبط الإعدادات تلقائياً حسب الـ Dependencies الموجودة في الـ Classpath.
2. **Starter Dependencies:** يجمع المكتبات المتوافقة مع بعضها في تبعية واحدة مع ضمان عدم تعارض الإصدارات.
3. **Embedded Web Servers:** يدمج سيرفر الويب (Tomcat/Jetty/Undertow) داخل التطبيق، ويولد ملف JAR تنفيذي يعمل بـ `java -jar app.jar`.
4. **Production-Ready Features (Actuator):** مراقبة صحة التطبيق، الـ Metrics، والـ Logging بدون كود يدوي.

| وجه المقارنة | Spring Framework التقليدي | Spring Boot |
| :--- | :--- | :--- |
| **طريقة الإعداد (Configuration)** | يدوي بالكامل (XML أو Java Config مفصلة) | تلقائي (Auto-configuration) وقابل للتعديل |
| **إدارة التبعيات** | المطور يحدد كل إصدار بنفسه | Starter POMs مع Parent BOM تضبط النسخ المتوافقة |
| **السيرفر (Web Server)** | يحتاج سيرفر خارجي (External Tomcat/JBoss) | سيرفر مدمج (Embedded Tomcat/Undertow) |
| **نوع الـ Packaging** | ملف WAR للـ Deployment | ملف Fat/Uber JAR تنفيذي مستقل |

**Short answer (30 seconds version):**
> "Spring Boot is an opinionated framework built on top of the Spring Framework. It eliminates boilerplate configuration through auto-configuration, simplifies dependency management with starter POMs, provides embedded web servers for standalone execution, and provides production-ready monitoring tools out of the box."

---

### Q2: How does Spring Boot Auto-configuration work under the hood?
**Level:** Mid  
**Related to:** Q1, Q4 (@SpringBootApplication), Q6 (Conditional Annotations)

**Answer:**
**Auto-Configuration** تعني: Spring Boot يفحص الـ Classpath والـ Properties والـ Beans المسجلة، وبناءً عليها يقرر تلقائياً تسجيل الـ Beans المناسبة بدون تدخل منك.

#### كيف يعمل السحر تحت الغطاء؟
1. في الـ Main Class توجد annotation `@EnableAutoConfiguration` (وهي جزء من `@SpringBootApplication`).
2. يبحث Spring Boot داخل ملف `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (أو `spring.factories` في الإصدارات الأقدم).
3. يجد قائمة بمئات كلاسات الـ Configuration التلقائية، مثل:
   - `DataSourceAutoConfiguration`
   - `WebMvcAutoConfiguration`
   - `SecurityAutoConfiguration`
4. **السر في الـ Conditional Annotations:**
   كل كلاس Auto-config لا يتم تفعيله عشوائياً، بل يحتوي على شروط صارمة:
   - `@ConditionalOnClass(DataSource.class)`: لا تفعل إعدادات قاعدة البيانات إلا إذا كان درايفر قاعدة البيانات موجوداً في الـ Classpath.
   - `@ConditionalOnMissingBean(DataSource.class)`: لو المطور كتب بنفسه `@Bean public DataSource dataSource()`، تراجع يا Spring Boot ولا تسجل الـ Bean التلقائي! (Giving the developer full control).
   - `@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")`: تفعيل الإعداد فقط لو قيمة بروبرتي معينة مطابقة.

```
تطبيقك يبدأ ──► يقرأ الـ Classpath ──► يجد H2 / MySQL Driver
     │
     ▼
يفحص: هل المطور عرّف DataSource بنفسه؟
 ├── نعم  ──► Spring Boot يتنحى جانباً (Developer wins)
 └── لا   ──► DataSourceAutoConfiguration ينشئ HikariCP DataSource تلقائياً!
```

**Short answer (30 seconds version):**
> "Auto-configuration works via @EnableAutoConfiguration which scans Spring Boot's internal autoconfigure imports file. It conditionally registers beans using @Conditional annotations (like @ConditionalOnClass and @ConditionalOnMissingBean), providing sensible defaults while gracefully backing off if the developer provides a custom bean."

---

### Q3: What are Spring Boot Starters, and how do they prevent dependency conflicts?
**Level:** Junior  
**Related to:** Q1, Q2

**Answer:**
في الماضي، إذا أردت بناء REST API مع Spring و JSON و Validation، كنت تحتاج كتابة 10 إلى 15 سطر في `pom.xml`:
- `spring-web`
- `spring-webmvc`
- `jackson-databind`
- `hibernate-validator`
- `tomcat-embed-core`
... وكنت مطالباً بالتأكد أن نسخة Jackson متوافقة بنسبة 100% مع نسخة Spring Web وإلا سيحدث `NoSuchMethodError` وقت التشغيل!

**Spring Boot Starter** هو مجرد **Descriptor POM (Meta-dependency)** يجمع كل الـ dependencies المرتبطة بـ Goal معين تحت مظلة واحدة:
- `spring-boot-starter-web`: يجمع Spring MVC, Jackson, Tomcat Mدمج, Validation.
- `spring-boot-starter-data-jpa`: يجمع Hibernate, Spring Data JPA, HikariCP Connection Pool.
- `spring-boot-starter-security`: يجمع Spring Security Core, Web filter chain.

#### من يدير الإصدارات؟
ملف الـ Parent POM:
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.x.x</version>
</parent>
```
الـ Parent يستورد **Bill of Materials (BOM)** الذي يحتوي على جدول توافق دقيق لجميع إصدارات المكتبات، ولذلك في الـ `pom.xml` الخاص بك، أنت لا تكتب `<version>` لأي مكتبة رسمية مدعومة!

**Short answer (30 seconds version):**
> "Spring Boot Starters are curated sets of dependency descriptors that bundle all necessary libraries for a specific functionality into a single dependency. Managed under the starter parent BOM, they ensure compatible, pre-tested library versions and eliminate version mismatch errors."

---

### Q4: Explain `@SpringBootApplication` and its 3 core composite annotations.
**Level:** Junior  
**Related to:** Q1, Q2, Section 01 - Q9

**Answer:**
الـ Annotation الرئيسية لأي تطبيق Spring Boot هي `@SpringBootApplication`، وهي عبارة عن **Composite Meta-annotation** تجمع 3 Annotations رئيسية:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration     // 1. يحدد الكلاس كـ Configuration Bean (بديل @Configuration)
@EnableAutoConfiguration     // 2. يشغل محرك الـ Auto-configuration
@ComponentScan               // 3. يفعل الـ Component Scanning في الباكيج الحالية والباكيجات الفرعية
public @interface SpringBootApplication {
    // ...
}
```

#### تفصيل المكونات:
1. **`@SpringBootConfiguration`:**
   - متفرعة من `@Configuration`.
   - تعلن أن هذا الكلاس هو كلاس إعدادات يمكنه احتواء ميثودز عليها `@Bean`.

2. **`@EnableAutoConfiguration`:**
   - توجه Spring Boot للبحث عن المكتبات في الـ Classpath وتطبيق الإعدادات التلقائية (Auto-configuration).

3. **`@ComponentScan`:**
   - توجه Spring للبحث عن الـ Stereotype Annotations (`@Component`, `@Service`, `@Repository`, `@RestController`).
   - ⚠️ **قاعدة ذهبية في الباكيج:** الـ Component Scan الافتراضي يبدأ من الباكيج التي تحتوي على الكلاس الذي يحمل `@SpringBootApplication` وينزل للـ Sub-packages فقط. لو وضعت كلاس في باكيج موازية أو أعلى، لن يتم اكتشافه كـ Bean!

**Short answer (30 seconds version):**
> "@SpringBootApplication is a composite annotation comprising @SpringBootConfiguration (marking the class as a source of bean definitions), @EnableAutoConfiguration (triggering Spring Boot's automatic configuration engine), and @ComponentScan (scanning thecurrent package and its sub-packages for Spring components)."

---

### Q5: What is "Convention Over Configuration" in Spring Boot?
**Level:** Junior  
**Related to:** Q1, Q2

**Answer:**
**Convention Over Configuration (العرف قبل الإعداد):** هي فلسفة برمجية تعني:
"النظام يفترض أفضل الاختيارات القياسية (Defaults) المتعارف عليها في الصناعة تلقائياً، ولا يطلب من المطور كتابة إعدادات إلا إذا أراد الخروج عن المألوف أو تخصيص سلوك محدد."

#### أمثلة حية في Spring Boot:
1. **اسم ومكان ملف الإعدادات:**
   - العرف: ضع إعداداتك داخل `src/main/resources/application.properties` أو `application.yml`، و Spring Boot سيقرأه تلقائياً بدون أن توجهه لمسار الملف.
2. **منفذ السيرفر (Port):**
   - العرف: تشغيل السيرفر على بورت `8080`. لا تحتاج لتحديده إلا إذا أردت بورت مختلف مثل `server.port=9090`.
3. **مكتبة الـ Connection Pool:**
   - العرف: استخدام **HikariCP** تلقائياً لأنها الأسرع والأكثر استقراراً في Java.
4. **موقع الـ Static Resources:**
   - العرف: أي ملف داخل `resources/static/` أو `resources/public/` يُقدم مباشرة للمتصفح.
5. **Naming Conventions في الـ JPA:**
   - كلاس `UserOrder` يقابله تلقائياً جدول `user_order` في قاعدة البيانات.

**Short answer (30 seconds version):**
> "Convention Over Configuration means Spring Boot provides pre-configured sensible defaults for development (default port 8080, HikariCP, standard folder structures). Developers only need to write configuration when they explicitly need to override these defaults."

---

### Q6: What is an Embedded Server? (Embedded Tomcat vs External Application Server)
**Level:** Junior / Mid  
**Related to:** Section 02 - Q7, Q8

**Answer:**
مأخوذة من مذكراتك في ملف `50-Web Server vs Application Server.md` و `8-Embeded Server.md`:

#### الطريقة القديمة (Traditional External Server):
- كنت تبني التطبيق كملف `WAR` (Web Application Archive).
- تذهب لسيرفر خارجي (Standalone Tomcat, JBoss, WebLogic) تم تنصيبه مسبقاً على السيرفر.
- تضع ملف الـ WAR داخل مجلد `webapps` وتقوم بعمل Restart للسيرفر.
- عيوبها: مشاكل في اختلاف إصدارات جافا والسيرفرات بين جهاز المطور وجهاز الـ Production.

#### طريقة Spring Boot (Embedded Server):
- سيرفر الويب (افتراضياً Apache Tomcat) أصبح عبارة عن مكتبة جافا داخل ملف الـ `JAR` نفسه (`tomcat-embed-core`).
- عند تشغيل الميثود `main`، الكود البرمجي هو الذي ينشئ الـ Tomcat ويقوم بتشغيله داخلياً!
- النتيجة: التطبيق أصبح **Self-contained** (مستقل تماماً) ويعمل بأمر واحد:
  ```bash
  java -jar myapp.jar
  ```

#### كيف تغير السيرفر المدمج؟ (مثلاً من Tomcat إلى Jetty أو Undertow):
في `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

**Short answer (30 seconds version):**
> "An embedded server is a web server packaged directly inside the application archive (Fat JAR) as a library. Instead of deploying a WAR file to an external web container, the application bootstraps its own HTTP server (Tomcat by default) programmatically upon startup."

---

### Q7: Web Server vs Application Server vs Servlet Container — What is Tomcat?
**Level:** Mid  
**Related to:** Q6 (Embedded Server), Q8 (Servlet vs Spring)

**Answer:**
من مذكراتك بملف `50-Web Server vs Application Server.md`:

#### 1. Web Server (سيرفر الويب):
- وظيفته: التعامل مع بروتوكول HTTP وتقديم الملفات الثابتة (**Static Content** مثل HTML, CSS, JS, Images).
- أمثلة: **Nginx, Apache HTTP Server**.
- لا يفهم كود Java ولا ينفذ Business Logic معقد.

#### 2. Servlet Container (حاوية السيرفلت):
- وظيفته: ترجمة الـ HTTP Requests وتمريرها لكود جافا يسمى **Servlet**، وإعادة الـ Dynamic Content للمتصفح.
- مثال: **Apache Tomcat, Jetty**.

#### 3. Application Server (سيرفر التطبيقات الشامل):
- وظيفته: يحتوي على Servlet Container و Web Server ويضيف تطبيق كافة مواصفات **Jakarta EE (Enterprise Java)** كاملة:
  - EJB (Enterprise JavaBeans)
  - JMS (Java Message Service)
  - JTA (Distributed Transactions)
- أمثلة: **WildFly (JBoss), Oracle WebLogic, IBM WebSphere**.

#### ما هو Apache Tomcat تحديداً؟
> 📌 **معلومة إنترفيو مهمة من مذكراتك:**
> **Tomcat ليس Full Application Server!**  
> هو عبارة عن **Web Server + Servlet Container** فقط. وبما أن Spring Boot يحتوي على حلول بديلة لكل ما كان يقدمه الـ Application Server (مثل Spring Data و Spring Messaging و Transactions المدارة بواسطة Spring)، فنحن لسنا بحاجة لـ Full Application Server عملاق؛ ولهذا Tomcat المدمج كافٍ جداً ومثالي للـ Microservices.

**Short answer (30 seconds version):**
> "A Web Server serves static files (Nginx). A Servlet Container runs Java servlets and generates dynamic web content (Tomcat). An Application Server implements the complete enterprise Java EE stack including EJBs and JTA (WildFly). Tomcat is fundamentally a Servlet Container and Web Server, not a full Java EE Application Server."

---

### Q8: Servlet vs Spring MVC — What is DispatcherServlet?
**Level:** Mid  
**Related to:** Section 04 (Spring MVC)

**Answer:**
مأخوذة من ملف مذكراتك `55-Servlet vs Spring.md`:

#### 1. ما هو الـ Servlet؟
- الـ Servlet هو **Java Specification / API** منخفض المستوى للتعامل مع الـ HTTP Requests في جافا.
- بدون Spring: لو تطبيقك فيه 100 endpoint، كنت ستضطر لإنشاء 100 كلاس يمتد من `HttpServlet` وتسجيل كل واحد في ملف `web.xml`!
```java
// الطريقة القديمة المؤلمة:
public class UserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String id = req.getParameter("id");
        // معالجة البيانات يدوي وكتابة JSON بالـ PrintWriter...
        resp.getWriter().write("{\"name\":\"Ali\"}");
    }
}
```

#### 2. ما هو DispatcherServlet في Spring؟
- Spring MVC لم يلغِ الـ Servlet، بل بُني **فوقه**!
- استخدم نمطاً تصميمياً عبقرياً يسمى **Front Controller Pattern**.
- في Spring MVC، يوجد **Servlet واحد فقط مركزي** يستقبل كل الطلبات القادمة من Tomcat اسمه: **`DispatcherServlet`**.

```
 المتصفح (Client)
       │ HTTP Request
       ▼
 Apache Tomcat (Port 8080)
       │
       ▼
┌──────────────────────────────────────────────┐
│  DispatcherServlet (The Front Controller)    │
└──────────────────────┬───────────────────────┘
                       │ 1. أين يذهب هذا الـ Request؟ (HandlerMapping)
                       ▼
             ┌───────────────────┐
             │  UserController   │  (@GetMapping("/users"))
             └─────────┬─────────┘
                       │ 2. استدعاء الـ Business Logic
                       ▼
             ┌───────────────────┐
             │    UserService    │
             └───────────────────┘
```

**Short answer (30 seconds version):**
> "A Servlet is the foundational Java standard for handling HTTP requests. Spring MVC is built directly on top of servlets using a single master servlet called the DispatcherServlet (Front Controller pattern), which intercepts all incoming requests and routes them to appropriate @Controller handler methods."

---

### Q9: `application.properties` vs `application.yml` and Spring Profiles
**Level:** Junior  
**Related to:** Q1, Q5

**Answer:**
#### 1. Properties vs YAML:
كلاهما يُستخدم لضبط إعدادات التطبيق في Spring Boot.

**Properties (Flat Key-Value):**
```properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=secret
```

**YAML (Hierarchical & Cleaner):**
```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: root
    password: secret
```
- الـ YAML يتميز بدعم التسلسل الهرمي (Hierarchical structure)، وتجنب تكرار البوادئ، ودعم الـ Lists بشكل طبيعي.

#### 2. Spring Profiles (إدارة بيئات العمل):
الـ Profile يسمح لك بتشغيل كود معين أو تطبيق إعدادات معينة بناءً على البيئة الحالية (`dev`, `test`, `prod`).

- يمكنك إنشاء ملفات منفصلة:
  - `application-dev.yml` (قاعدة بيانات محلية H2 / MySQL، الـ Logs على وضع `DEBUG`).
  - `application-prod.yml` (قاعدة بيانات PostgreSQL في الكلاود، الـ Logs على وضع `WARN/ERROR`).

**تفعيل الـ Profile:**
- في `application.yml`:
  ```yaml
  spring:
    profiles:
      active: dev
  ```
- أو عبر الـ Environment Variable وقت التشغيل في الإنتاج (Best Practice for Docker):
  ```bash
  java -jar -Dspring.profiles.active=prod app.jar
  ```
- أو باستخدام `@Profile` على مستوى الـ Beans:
  ```java
  @Component
  @Profile("dev")
  public class MockEmailService implements EmailService { ... }
  ```

**Short answer (30 seconds version):**
> "Both formats store application configurations; YAML is preferred for hierarchical readability and list handling. Spring Profiles allow segregating configuration for different environments (dev, test, prod), activated via 'spring.profiles.active' or environment variables."

---

### Q10: What happens internally when you run `SpringApplication.run(Main.class, args)`? [ADDED - not in my notes]
**Level:** Senior / Mid  
**Related to:** Q1, Q2, Section 01 - Q5

**Answer:**
هذا السؤال هو المفضل للمقابلات المتقدمة:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### المراحل بالتسلسل الداخلي:
1. **تحديد نوع التطبيق (Application Type Deduction):**
   - يفحص Spring هل نحن في بيئة Web عادية (`Servlet`) أم Reactive (`WebFlux`) أم تطبيق عادي سطر أوامر (`None`).
2. **تحميل الـ Initializers والـ Listeners:**
   - يقرأ `spring.factories` لتحميل الـ `ApplicationContextInitializer` و `ApplicationListener`.
3. **تجهيز البيئة (Prepare Environment):**
   - قراءة الـ Command-line arguments، ملفات الـ `application.yml`، الـ System variables، والـ Profiles المفعلة.
4. **طباعة الـ Banner:** طباعة لوجو Spring Boot الشهير في الـ Terminal.
5. **إنشاء الـ ApplicationContext:**
   - ينشئ `AnnotationConfigServletWebServerApplicationContext` لتطبيقات الويب.
6. **تجهيز الـ Context وتطبيق الـ Auto-configuration:**
   - تحميل الـ Bean Definitions وتشغيل الـ `BeanDefinitionLoader`.
7. **تشغيل وتنشيط الـ Context (`refresh()`):**
   - هذه أهم وأضخم مرحلة في Spring Core:
     - إنشاء وبدء الـ Embedded Tomcat Server.
     - تشغيل الـ BeanPostProcessors.
     - إنشاء كل الـ Singleton Beans وعمل Dependency Injection لها.
8. **استدعاء الـ Runners:**
   - استدعاء أي كلاس يعمل `CommandLineRunner` أو `ApplicationRunner`.
9. **التطبيق جاهز لاستقبال الطلبات!**

**Short answer (30 seconds version):**
> "SpringApplication.run() initializes the environment, creates the appropriate ApplicationContext (Servlet-based or Reactive), triggers the refresh phase to instantiate and wire all singleton beans, boots the embedded Tomcat web server, and finally invokes any ApplicationRunners."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *How do you disable a specific Auto-configuration class? (`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})`)*
2. *How does Spring Boot read external configurations? What is the order of precedence? (Command line > System properties > Environment variables > application.yml)*
3. *What is the difference between `@Value` and `@ConfigurationProperties`? (Type safety and relaxed binding)*
4. *Can you run a Spring Boot application without an embedded server? (Yes, by setting `spring.main.web-application-type=none`)*

---

## 🔗 System Design Connection: 12-Factor App & Cloud-Native Deployments

### Real-world System Design Application:
تطوير التطبيقات الحديثة الموجهة للـ Cloud والحاويات (Docker & Kubernetes) يعتمد على مبادئ **The Twelve-Factor App**.

Spring Boot صُمم خصيصاً ليتطابق مع هذه المبادئ:
1. **Config in Environment (Factor III):**
   - بدلاً من عمل Re-build للـ JAR مع كل تغيير في الباسورد أو الـ URLs، نمرر الإعدادات عبر Environment Variables:
     ```bash
     SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/prod
     ```
2. **Port Binding (Factor VII):**
   - التطبيق الـ Cloud-Native لا يعتمد على خادم خارجي مستضاف؛ بل هو بنفسه يعمل Export لمنفذ الـ HTTP مباشرة بفضل الـ Embedded Tomcat.
3. **Disposability & Fast Startup (Factor IX):**
   - ملف الـ JAR الذاتي يسهل تشغيله في ثوانٍ داخل Kubernetes Pod وعمل Scale Out/In بحرية عند زيادة الضغط.

```
                    ┌────────────────────────────┐
                    │     Kubernetes Pod         │
                    │                            │
                    │   ┌────────────────────┐   │
  HTTP Request ────►│   │  myapp.jar (App)   │   │
  Port: 8080        │   │  (Embedded Tomcat) │   │
                    │   └─────────┬──────────┘   │
                    │             │ reads        │
                    │             ▼              │
                    │    ENV: DB_URL / PROFILES  │
                    └────────────────────────────┘
```
