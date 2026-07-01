# 37. What is `ApplicationContext` in Spring?

## 📌 Short Definition (Interview Answer)

`ApplicationContext` is the **Spring IoC Container** responsible for:

- Creating beans
    
- Managing bean lifecycle
    
- Injecting dependencies
    
- Reading configuration
    
- Providing access to application objects
    

In simple words:

> `ApplicationContext` is the brain of the Spring application.

---

# 🧠 What is IoC Container?

Before understanding `ApplicationContext`, you need to understand the problem it solves.

## Without Spring

```java
public class OrderService {

    private EmailService emailService = new EmailService();

}
```

Here:

- `OrderService` creates `EmailService` itself.
    
- Classes become tightly coupled.
    
- Changing implementation becomes difficult.
    

---

## With Spring

```java
@Service
public class EmailService {

}
```

```java
@Service
public class OrderService {

    private final EmailService emailService;

    public OrderService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

Spring does:

1. Create `EmailService`
    
2. Create `OrderService`
    
3. Inject `EmailService` into `OrderService`
    

Who does all this?

## `ApplicationContext`

---

# What does ApplicationContext do?

## 1. Creates Beans

When Spring starts:

```java
@Service
public class UserService {

}
```

Spring creates:

```text
UserService Object
```

and stores it inside `ApplicationContext`.

---

## 2. Stores Beans

Think of it as a huge map:

```text
ApplicationContext
│
├── UserService Bean
├── CourseService Bean
├── UserRepository Bean
├── JwtService Bean
└── SecurityConfig Bean
```

Whenever you need a bean:

```java
@Autowired
private UserService userService;
```

Spring looks inside `ApplicationContext`.

---

## 3. Dependency Injection

Example:

```java
@Service
public class CourseService {

    private final UserRepository repository;

    public CourseService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Spring process:

```text
Step 1 → Create UserRepository
Step 2 → Create CourseService
Step 3 → Inject UserRepository
```

---

## 4. Manages Bean Lifecycle

Spring controls:

```text
Bean Creation
↓
Dependency Injection
↓
@PostConstruct
↓
Bean Usage
↓
@PreDestroy
↓
Bean Destruction
```

---

## 5. Reads Configuration

Example:

```properties
server.port=8080
jwt.secret=abc123
```

You can access them:

```java
@Value("${jwt.secret}")
private String secret;
```

Spring gets this value using `ApplicationContext`.

---

## 6. Component Scanning

Spring scans:

```text
@Component
@Service
@Repository
@Controller
@Configuration
```

and registers them as beans.

Without `ApplicationContext`, none of these annotations work.

---

## 7. Event Publishing

Spring supports events:

```java
applicationContext.publishEvent(
        new UserRegisteredEvent(user)
);
```

Listeners:

```java
@EventListener
public void handle(UserRegisteredEvent event) {

}
```

---

## 8. Internationalization (i18n)

`ApplicationContext` provides message resources:

```java
applicationContext.getMessage(
        "welcome.message",
        null,
        Locale.ENGLISH
);
```

---

## 9. Resource Loading

Spring can load files:

```java
Resource resource =
        applicationContext.getResource(
                "classpath:data.txt"
        );
```

---

# How is ApplicationContext Created?

In Spring Boot:

```java
@SpringBootApplication
public class SkillHubApplication {

    public static void main(String[] args) {
        ApplicationContext context =
                SpringApplication.run(
                        SkillHubApplication.class,
                        args
                );
    }
}
```

---

## What does `SpringApplication.run()` do?

Internally:

```text
1. Create ApplicationContext
2. Scan components
3. Create beans
4. Resolve dependencies
5. Configure auto configuration
6. Start embedded server
7. Return ApplicationContext
```

---

# Example

```java
ApplicationContext context =
        SpringApplication.run(
                Application.class,
                args
        );

UserService userService =
        context.getBean(UserService.class);

userService.test();
```

Output:

```text
UserService bean created
```

---

# BeanFactory vs ApplicationContext

|Feature|BeanFactory|ApplicationContext|
|---|---|---|
|Bean Management|✅|✅|
|Dependency Injection|✅|✅|
|Events|❌|✅|
|Internationalization|❌|✅|
|Resource Loading|❌|✅|
|Auto BeanPostProcessor|❌|✅|
|Recommended Today|❌|✅|

---

## Relationship

```text
BeanFactory
    ↑
ApplicationContext
```

`ApplicationContext` extends `BeanFactory`.

---

# Real Life Analogy

Imagine a company:

## Employees

```text
UserService
CourseService
ReviewService
JwtService
```

## Company Manager

```text
ApplicationContext
```

The manager:

- Hires employees (creates beans)
    
- Assigns tasks (dependency injection)
    
- Manages lifecycle
    
- Fires employees on shutdown
    

Without the manager, employees must create and manage each other manually.

---

# ApplicationContext Lifecycle

```text
Application Starts
        ↓
ApplicationContext Created
        ↓
Component Scan
        ↓
Beans Created
        ↓
Dependencies Injected
        ↓
@PostConstruct Executed
        ↓
Application Ready
        ↓
Application Stops
        ↓
@PreDestroy Executed
```

---

# What happens if ApplicationContext does not exist?

Without it:

- No Dependency Injection
    
- No `@Autowired`
    
- No `@Service`
    
- No `@Repository`
    
- No `@Controller`
    
- No `@Configuration`
    
- No Auto Configuration
    
- No Bean Lifecycle Management
    

The application becomes a plain Java application.

---

# Interview Answer (30 Seconds)

> `ApplicationContext` is Spring's IoC container that creates, manages, and stores beans, performs dependency injection, manages bean lifecycle, loads configuration, publishes events, and provides additional enterprise features like internationalization and resource loading.

---

# One-Line Summary

> `ApplicationContext` is the central container that manages the entire Spring application and all of its beans.