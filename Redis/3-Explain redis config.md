أيوه يا علي، الكود ده **شغال ومظبوط بشكل عام**، لكن فيه ملاحظتين مهمين هقولك عليهم بعد ما أشرحلك كل سطر بيعمل إيه، لأن فهم Redis Cache مهم جدًا في الـ Spring Boot.

---

# أولاً: ما هو Redis Cache Manager أصلاً؟

تخيل إن عندك Endpoint زي:

```java
@GetMapping("/courses")
public List<CourseResponse> getAllCourses() {
    return courseService.getAllCourses();
}
```

والـ Service:

```java
@Cacheable("courses")
public List<CourseResponse> getAllCourses() {
    return repository.findAll();
}
```

أول Request:

```
Client
 ↓
Spring Cache
 ↓
Redis (مش لاقي البيانات)
 ↓
Database
 ↓
Redis
 ↓
Client
```

ثاني Request:

```
Client
 ↓
Spring Cache
 ↓
Redis (البيانات موجودة)
 ↓
Client
```

يعني بدل ما كل مرة تضرب الـ Database، بتجيب من Redis.

---

# الكود بالكامل

```java
@Configuration
public class RedisConfig {
```

## ماذا يفعل؟

بيقول لـ Spring:

> "الكلاس ده فيه Beans عايزك تضيفها في Application Context."

يعني Spring هيقرأ الكلاس ده وقت تشغيل التطبيق.

---

## الـ Bean

```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory factory)
```

هنا بتقول:

> "أنا مش عايز الـ CacheManager الافتراضي، عايز أعمل واحد مخصص."

Spring هيحقن:

```java
RedisConnectionFactory factory
```

وده المسؤول عن الاتصال بـ Redis.

تقريبًا:

```java
RedisConnectionFactory
↓
Lettuce
↓
Redis Server
```

---

# ObjectMapper

```java
ObjectMapper mapper = new ObjectMapper();
```

الـ ObjectMapper مسؤول عن:

```
Java Object
↔
JSON
```

مثلاً:

```java
CourseResponse response =
    new CourseResponse(1L, "Java");
```

يتحول إلى:

```json
{
  "id":1,
  "title":"Java"
}
```

---

# JavaTimeModule

```java
mapper.registerModule(new JavaTimeModule());
```

مهم جدًا.

بدونه:

```java
LocalDateTime.now()
```

ممكن Jackson ميعرفش يحوله.

مثلاً:

```java
EnrollmentResponse(
    1L,
    80,
    LocalDateTime.now()
)
```

هيترمي Exception.

مع JavaTimeModule:

هيتحول إلى:

```json
"2026-06-16T18:30:00"
```

---

# تعطيل Timestamp

```java
mapper.disable(
    SerializationFeature.WRITE_DATES_AS_TIMESTAMPS
);
```

بدون السطر ده:

```json
1748764545000
```

(Unix Timestamp)

أما معه:

```json
"2026-06-16T18:30:00"
```

وده أوضح للبشر.

---

# Serializer

```java
JacksonJsonRedisSerializer<Object> serializer =
        new JacksonJsonRedisSerializer<>(Object.class);
```

ده مسؤول عن:

```
Java Object
↓
JSON
↓
Redis
```

وبالعكس.

مثلاً:

عندك:

```java
CourseResponse(
    1L,
    "Spring"
)
```

هيتخزن في Redis كده:

```json
{
  "id":1,
  "title":"Spring"
}
```

---

# المشكلة هنا ⚠️

أنت عامل:

```java
ObjectMapper mapper = new ObjectMapper();
```

لكن **مش مستخدمه**.

يعني الـ serializer بيستخدم الـ ObjectMapper الافتراضي.

وده معناه إن:

```java
JavaTimeModule
WRITE_DATES_AS_TIMESTAMPS
```

مش هيتطبقوا.

---

كان المفروض:

```java
serializer.setObjectMapper(mapper);
```

لكن...

في الإصدارات الحديثة من Spring Data Redis:

```java
setObjectMapper()
```

اتعملها Deprecated.

---

# الأفضل حالياً

بدل:

```java
JacksonJsonRedisSerializer
```

استخدم:

```java
GenericJackson2JsonRedisSerializer
```

زي كده:

```java
ObjectMapper mapper = new ObjectMapper();

mapper.registerModule(new JavaTimeModule());
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

GenericJackson2JsonRedisSerializer serializer =
        new GenericJackson2JsonRedisSerializer(mapper);
```

وده أفضل وأكثر استخدامًا.

---

# Cache Configuration

```java
RedisCacheConfiguration config =
    RedisCacheConfiguration.defaultCacheConfig()
```

دي الإعدادات الافتراضية للـ Cache.

بعدها بتخصصها.

---

# TTL

```java
.entryTtl(Duration.ofMinutes(10))
```

دي من أهم الحاجات.

TTL = Time To Live

يعني:

> البيانات هتعيش في Redis لمدة 10 دقائق فقط.

مثال:

الساعة 5:00

أول Request:

```
DB → Redis
```

الساعة 5:08:

```
Redis → Client
```

الساعة 5:11:

الـ Cache انتهت.

```
DB → Redis → Client
```

---

# Key Serializer

```java
.serializeKeysWith(
    RedisSerializationContext.SerializationPair
        .fromSerializer(
            new StringRedisSerializer()
        )
)
```

الـ Key بيتخزن كـ String.

مثلاً:

لو عندك:

```java
@Cacheable("courses")
```

هيبقى:

بدلاً من:

```
\xac\xed\x00...
```

هيبقى:

```
courses::SimpleKey[]
```

وده أسهل جدًا في القراءة.

---

# Value Serializer

```java
.serializeValuesWith(
    RedisSerializationContext.SerializationPair
        .fromSerializer(serializer)
)
```

يعني:

> القيم نفسها تتخزن باستخدام Jackson JSON.

بدلاً من Java Serialization.

---

## Java Serialization

شكلها:

```
¬í 0005sr...
```

غير مقروءة.

---

## Jackson JSON

شكلها:

```json
{
  "id":1,
  "title":"Spring"
}
```

مقروءة.

أخف.

أسرع غالبًا.

أفضل.

---

# إنشاء الـ CacheManager

```java
return RedisCacheManager.builder(factory)
        .cacheDefaults(config)
        .build();
```

هنا بتقول:

> "ابنِ لي CacheManager باستخدام Redis وبالإعدادات دي."

فيصبح Spring Cache:

```
@Cacheable
@CachePut
@CacheEvict
```

كلهم يستخدموا Redis بالإعدادات اللي أنت حددتها.

---

# هل الكود صحيح؟

## نعم، لكن عندي ملاحظتين:

### 1) ObjectMapper غير مستخدم

أنت عامل:

```java
ObjectMapper mapper
```

لكن الـ serializer لا يستخدمه.

فالسطرين دول حاليًا بلا فائدة:

```java
mapper.registerModule(new JavaTimeModule());

mapper.disable(
    SerializationFeature.WRITE_DATES_AS_TIMESTAMPS
);
```

---

### 2) الأفضل استخدام GenericJackson2JsonRedisSerializer

بدل:

```java
JacksonJsonRedisSerializer<Object>
```

استخدم:

```java
GenericJackson2JsonRedisSerializer
```

لأنه:

- يتعامل أفضل مع الأنواع المختلفة.
    
- يحافظ على معلومات النوع (Type Information).
    
- هو الأكثر استخدامًا حاليًا مع Spring Cache.
    

---

# الشكل النهائي لتدفق الكاش

```
Client
   ↓
@Cacheable
   ↓
RedisCacheManager
   ↓
Redis
   ↓ (Miss)
Database
   ↓
Redis (10 min TTL)
   ↓
Client
```

فباختصار: الكود عندك "هيشتغل"، لكن لو عندك Objects فيها `LocalDateTime` أو Records معقدة، أنصحك تعدله لـ `GenericJackson2JsonRedisSerializer` عشان تستفيد فعلًا من الـ `ObjectMapper` اللي أنت جهزته.