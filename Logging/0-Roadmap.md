لو هدفك **Spring Boot Backend Interviews**، فدي الـ Roadmap المختصرة جدًا:

## 1. أساسيات Logging

- إيه هو Logging وليه بنستخدمه بدل `System.out.println()`.
    
- Log Levels:
    
    - `TRACE`
        
    - `DEBUG`
        
    - `INFO` ✅ (الأكثر استخدامًا)
        
    - `WARN`
        
    - `ERROR`
        

---

## 2. SLF4J

اعرف إن:

- **SLF4J** = Logging Facade (واجهة).
    
- مش هو اللي بيكتب الـ Logs.
    

---

## 3. Logback

اعرف إن:

- **Logback** = Default Logging Implementation في Spring Boot.
    
- هو اللي بيكتب الـ Logs فعلًا.
    

---

## 4. Lombok

اعرف:

```java
@Slf4j
```

بتولد:

```java
private static final Logger log =
    LoggerFactory.getLogger(MyClass.class);
```

---

## 5. استخدام Logger

اعرف تستخدم:

```java
log.info("User created");
log.debug("User id {}", id);
log.warn("Low memory");
log.error("Failed", ex);
```

واستخدم `{}` بدل String Concatenation.

---

## 6. Logging Configuration

اعرف تغير مستوى الـ Logs.

```properties
logging.level.root=INFO

logging.level.com.myapp=DEBUG
```

---

## 7. Exceptions

اعرف تسجل Exception صح.

```java
log.error("Database failed", exception);
```

مش:

```java
log.error(exception.getMessage());
```

---

## 8. في Spring Boot

اعرف إن:

```
Application
↓

SLF4J

↓

Logback

↓

Console / File
```

---

# لو عندك وقت زيادة

اتعلم:

- Log Files
    
- Rolling Files
    
- MDC (Correlation ID / Request ID)
    
- Structured Logging (JSON Logs)
    

---

## لو انترفيو Backend Java

يكفي جدًا تعرف:

- ✅ Logging Basics
    
- ✅ Log Levels
    
- ✅ SLF4J
    
- ✅ Logback
    
- ✅ `@Slf4j`
    
- ✅ Configuration (`logging.level`)
    
- ✅ تسجيل Exceptions صح
    

**دي تمثل حوالي 90% من اللي بيتسأل عنه في انترفيوهات Java/Spring Boot.**