# Interview Question: What happens internally when a Spring Boot application starts?

This is one of the most common interview questions for **Spring Boot** developers. Interviewers ask it to evaluate whether you understand the framework beyond just writing annotations.

---

# Question

> **Explain what happens internally when a Spring Boot application starts.**

---

# Short Interview Answer (2–3 minutes)

> When a Spring Boot application starts, the JVM loads the application's main class and executes the `main()` method. The `SpringApplication.run()` method then bootstraps the Spring framework by creating an `ApplicationContext`, loading configuration, performing component scanning, creating and wiring beans using Dependency Injection, applying auto-configuration, starting the embedded web server (if it's a web application), and finally making the application ready to serve requests.

---

# Let's go through the startup process step by step.

---

# Step 1: JVM Starts

Suppose we have:

```java
@SpringBootApplication
public class EcommerceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EcommerceApplication.class, args);
    }

}
```

The first thing that happens is:

```
User
   │
   ▼
java -jar app.jar
   │
   ▼
JVM Starts
```

The JVM

- Loads classes
    
- Creates the Main Thread
    
- Executes
    

```java
main()
```

---

# Step 2: SpringApplication.run()

Now this line executes:

```java
SpringApplication.run(EcommerceApplication.class, args);
```

This is the heart of Spring Boot startup.

Internally it creates

```java
SpringApplication application =
        new SpringApplication(EcommerceApplication.class);
```

Then

```java
application.run(args);
```

---

# Step 3: Determine Application Type

Spring Boot determines which type of application this is.

It checks the classpath.

If it finds

```
spring-webmvc
```

↓

It creates

```
Servlet Web Application
```

If it finds

```
spring-webflux
```

↓

It creates

```
Reactive Application
```

Otherwise

```
Console Application
```

---

# Step 4: Create ApplicationContext

Spring now creates the appropriate IoC Container.

For MVC applications:

```
AnnotationConfigServletWebServerApplicationContext
```

This container is responsible for

- Creating beans
    
- Managing bean lifecycle
    
- Dependency Injection
    
- Event publishing
    

Think of it as the brain of Spring.

---

# Step 5: Read application.properties

Spring loads configuration from

```
application.properties
```

or

```
application.yml
```

Example

```properties
server.port=8081

spring.datasource.url=...

spring.jpa.hibernate.ddl-auto=update
```

These values become available through

```java
@Value

@ConfigurationProperties
```

---

# Step 6: Process @SpringBootApplication

This annotation is actually three annotations combined.

```java
@SpringBootApplication
```

=

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

## @Configuration

Tells Spring

"This class contains bean definitions."

---

## @ComponentScan

Spring starts scanning packages.

Example

```
com.company

│
├── controller

├── service

├── repository

├── config

└── security
```

It looks for

```
@Component

@Service

@Repository

@Controller

@RestController
```

---

# Step 7: Bean Definitions

Suppose Spring finds

```java
@Service
public class UserService {

}
```

Spring does NOT immediately create the object.

First it creates a

```
Bean Definition
```

Think of it like metadata.

```
Bean Name

Bean Type

Scope

Constructor

Dependencies
```

---

# Step 8: Auto Configuration

One of Spring Boot's biggest features.

Spring checks

```
META-INF/spring.factories
```

(or the newer `AutoConfiguration.imports` mechanism in recent versions)

It sees hundreds of auto-configurations.

Example

```
DataSourceAutoConfiguration

JpaAutoConfiguration

SecurityAutoConfiguration

DispatcherServletAutoConfiguration

JacksonAutoConfiguration
```

Spring asks:

"Do the required classes exist?"

Example

```
Is spring-data-jpa present?

YES

↓

Configure Hibernate
```

---

# Step 9: Create Singleton Beans

Now Spring begins creating beans.

Example

```java
@Service
public class UserService {

}
```

Spring creates

```
UserService Object
```

Stores it inside the IoC Container.

---

# Step 10: Dependency Injection

Suppose

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring first creates

```
UserRepository
```

Then

```
UserService
```

It injects

```
repository
```

through the constructor.

---

# Step 11: Bean Lifecycle

Every bean goes through several phases.

```
Instantiate Bean
        │
        ▼
Dependency Injection
        │
        ▼
Aware Interfaces
        │
        ▼
@PostConstruct
        │
        ▼
BeanPostProcessor
        │
        ▼
Ready
```

If you have

```java
@PostConstruct
public void init() {

}
```

It executes here.

---

# Step 12: Embedded Server Starts

If it's a web application,

Spring starts

```
Tomcat

Jetty

Undertow
```

Example

```
Tomcat Started on Port 8080
```

Now Tomcat begins listening.

```
8080

↓

HTTP Requests
```

---

# Step 13: DispatcherServlet

Spring registers

```
DispatcherServlet
```

This is the Front Controller.

Every request goes through it.

```
Browser

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository

↓

Database
```

---

# Step 14: Application Ready

Finally Spring publishes

```
ApplicationReadyEvent
```

At this point

```
Application Started Successfully
```

---

# Complete Startup Flow

```
java -jar
      │
      ▼
JVM Starts
      │
      ▼
main()
      │
      ▼
SpringApplication.run()
      │
      ▼
Determine Application Type
      │
      ▼
Create ApplicationContext
      │
      ▼
Load Properties
      │
      ▼
Component Scan
      │
      ▼
Bean Definitions
      │
      ▼
Auto Configuration
      │
      ▼
Create Beans
      │
      ▼
Dependency Injection
      │
      ▼
@PostConstruct
      │
      ▼
Embedded Tomcat Starts
      │
      ▼
DispatcherServlet Registered
      │
      ▼
ApplicationReadyEvent
      │
      ▼
Ready to Serve Requests
```

---

# Common Follow-up Interview Questions

### 1. What is the IoC Container?

The IoC (Inversion of Control) Container is responsible for creating, configuring, wiring, and managing the lifecycle of beans. In Spring Boot, this is typically an implementation of `ApplicationContext`.

---

### 2. What is the difference between Bean Definition and Bean Instance?

- **Bean Definition:** Metadata describing how Spring should create a bean (class, scope, dependencies, etc.).
    
- **Bean Instance:** The actual Java object created from that definition and stored in the IoC container.
    

---

### 3. What is Auto Configuration?

Auto Configuration is a Spring Boot feature that automatically configures beans based on the libraries present on the classpath and the application's configuration. It reduces manual configuration.

---

### 4. When are singleton beans created?

By default, singleton beans are **eagerly initialized** during application startup, before the application is ready to serve requests. Beans annotated with `@Lazy` are created only when first needed.

---

### 5. Why is Constructor Injection preferred?

Constructor Injection ensures all required dependencies are available when the bean is created, allows fields to be `final`, improves testability, and guarantees the bean is fully initialized before use.

---

# Interview Answer (5-minute Version)

> When Spring Boot starts, the JVM executes the `main()` method, which calls `SpringApplication.run()`. Spring determines the application type, creates an appropriate `ApplicationContext`, loads configuration files, and processes `@SpringBootApplication`, which enables component scanning, Java configuration, and auto-configuration. It scans the classpath to discover components, creates bean definitions, and then instantiates singleton beans while resolving dependencies through constructor injection. During bean creation, lifecycle callbacks such as `@PostConstruct` and `BeanPostProcessor` are executed. If the application is web-based, Spring Boot starts the embedded servlet container (such as Tomcat), registers the `DispatcherServlet` as the front controller, publishes an `ApplicationReadyEvent`, and the application becomes ready to receive HTTP requests. This startup process is driven by the IoC container, which manages the complete lifecycle of all Spring beans.