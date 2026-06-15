## Redis Roadmap — Spring Boot Developer

---

## Phase 1 — Redis Fundamentals

### ما هو Redis؟

- In-Memory Database — بيخزن في الـ RAM مش الـ Disk
- بيستخدم كـ Cache, Message Broker, Session Store
- أسرع بـ 100x من PostgreSQL في القراءة

### Data Structures

**String**

```bash
SET name "Ali"
GET name        # "Ali"
```

**List**

```bash
LPUSH courses "Java"
LPUSH courses "Spring"
LRANGE courses 0 -1   # ["Spring", "Java"]
```

**Set** — بدون تكرار

```bash
SADD tags "java" "spring" "java"
SMEMBERS tags   # {"java", "spring"}
```

**Hash** — زي الـ Object

```bash
HSET user name "Ali" email "ali@gmail.com"
HGET user name   # "Ali"
HGETALL user     # كل الـ fields
```

**TTL — Time To Live**

```bash
SET token "abc123"
EXPIRE token 3600    # بيتمسح بعد ساعة
TTL token            # كام ثانية فاضلة
```

---

## Phase 2 — Caching Basics

### ليه Caching؟

```
بدون Cache:
User Request → App → DB Query → Response
                      ↑ بطيء ومكلف

مع Cache:
User Request → App → Redis ✅ → Response (أسرع بكتير)
                      ↓ لو مش موجود
                      DB Query → Redis → Response
```

### Cache Strategies

|Strategy|بيعمل إيه|
|---|---|
|Cache Aside|الـ App بتتحقق من الـ Cache الأول|
|Write Through|بيكتب في الـ Cache والـ DB مع بعض|
|Write Behind|بيكتب في الـ Cache الأول والـ DB بعدين|

---

## Phase 3 — Redis مع Spring Boot

### الـ Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### الـ Configuration

```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379

spring.cache.type=redis
spring.cache.redis.time-to-live=3600000
```

### تشغيل Redis بـ Docker

```yaml
# docker-compose.yml
services:
  redis:
    image: redis:7-alpine
    container_name: skillhub_redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

volumes:
  redis_data:
```

---

## Phase 4 — Spring Cache Annotations

### تفعيل الـ Cache

```java
@SpringBootApplication
@EnableCaching  // ← مهم جداً
public class SkillHubApplication { }
```

### `@Cacheable` — جيب من الـ Cache لو موجود

```java
@Cacheable(value = "courses", key = "#id")
public CourseResponse getCourseById(Long id) {
    // لو موجود في الـ Cache → مش هيدخل هنا
    // لو مش موجود → هينفذ وبعدين يحفظ في الـ Cache
    return courseRepository.findById(id)...;
}
```

```
أول request:  DB query → نتيجة → تتحفظ في Redis
تاني request: Redis ✅ → نتيجة (من غير DB)
```

### `@CacheEvict` — امسح من الـ Cache

```java
@CacheEvict(value = "courses", key = "#id")
public void deleteCourse(Long id) {
    // بيمسح الـ Cache لما الـ course يتمسح
    courseRepository.deleteById(id);
}

// امسح كل الـ Cache
@CacheEvict(value = "courses", allEntries = true)
public void deleteAllCourses() { }
```

### `@CachePut` — حدّث الـ Cache

```java
@CachePut(value = "courses", key = "#id")
public CourseResponse updateCourse(Long id, CourseRequest request) {
    // بينفذ دايماً ويحدث الـ Cache بالنتيجة الجديدة
    ...
}
```

### الفرق بين الثلاثة

|Annotation|بينفذ الـ Method؟|بيحفظ في Cache؟|بيمسح من Cache؟|
|---|---|---|---|
|`@Cacheable`|لو مش موجود بس|✅|❌|
|`@CachePut`|دايماً ✅|✅|❌|
|`@CacheEvict`|دايماً ✅|❌|✅|

---

## Phase 5 — RedisTemplate

لما تحتاج تتحكم في Redis يدوياً:

```java
@Service
@RequiredArgsConstructor
public class TokenService {

    private final RedisTemplate<String, String> redisTemplate;

    // حفظ token مع TTL
    public void saveToken(String userId, String token) {
        redisTemplate.opsForValue()
            .set("token:" + userId, token, 24, TimeUnit.HOURS);
    }

    // جيب token
    public String getToken(String userId) {
        return redisTemplate.opsForValue()
            .get("token:" + userId);
    }

    // امسح token عند الـ logout
    public void deleteToken(String userId) {
        redisTemplate.delete("token:" + userId);
    }
}
```

### العمليات الأساسية

|Operation|الاستخدام|
|---|---|
|`opsForValue()`|Strings|
|`opsForList()`|Lists|
|`opsForSet()`|Sets|
|`opsForHash()`|Hashes|
|`opsForZSet()`|Sorted Sets|

---

## Phase 6 — Use Cases في مشروعك

|الحالة|الحل|
|---|---|
|`getCourseById`|`@Cacheable`|
|`updateCourse`|`@CachePut`|
|`deleteCourse`|`@CacheEvict`|
|JWT Blacklist|`RedisTemplate` + TTL|
|Rate Limiting|`RedisTemplate` + TTL|
|OTP codes|`RedisTemplate` + TTL قصير|

---

## الـ Roadmap بالترتيب

```
Phase 1: Redis Fundamentals (يومين)
    └── Data Structures + Redis CLI

Phase 2: Caching Basics (يوم)
    └── Cache Strategies + متى تستخدم Cache

Phase 3: Setup (ساعتين)
    └── Docker + Dependencies + Config

Phase 4: Annotations (يومين)
    └── @Cacheable + @CacheEvict + @CachePut

Phase 5: RedisTemplate (يومين)
    └── Manual Cache + JWT Blacklist

Phase 6: Apply على مشروعك (3 أيام)
    └── Cache الـ Courses + Lessons + Auth
```

---

الـ Roadmap كاملة ومرتبة ✅ عايز نبدأ بـ Phase 3 ونطبقها على مشروعك؟