أوك، هبدأ معاك من الأول للآخر بالترتيب.

---

# Redis Caching في Spring Boot

## أولاً: إضافة الـ Dependencies

في `pom.xml` محتاج تضيف اتنين:

```xml
<!-- Spring Cache Abstraction -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**ليه الاتنين دول؟**

- `spring-boot-starter-cache` ده الـ abstraction بتاع Spring — بيخليك تستخدم annotations زي `@Cacheable` من غير ما تتعامل مع Redis مباشرةً
- `spring-boot-starter-data-redis` ده الـ implementation الفعلي — بيربط Spring بـ Redis

---

## ثانياً: تشغيل Redis

محتاج Redis يكون شغال على جهازك، أسهل طريقة:

```bash
docker run -d --name redis -p 6379:6379 redis:latest
```

ده بيشغل Redis على port `6379` وهو الـ default.

---

## ثالثاً: الـ Configuration في `application.properties`

```properties
# Redis Connection
spring.data.redis.host=localhost
spring.data.redis.port=6379
# لو عندك password
# spring.data.redis.password=yourpassword

# Cache Type
spring.cache.type=redis

# Default TTL (وقت انتهاء الـ cache) - بالميللي ثانية
spring.cache.redis.time-to-live=600000
```

**الـ TTL مهم جداً** — ده بيحدد بعد كام وقت الـ cached data تتحذف أوتوماتيك. هنا 600000ms = 10 دقايق.

---

## رابعاً: تفعيل الـ Cache في Spring Boot

في الـ main class بتاعتك:

```java
@SpringBootApplication
@EnableCaching  // <-- دي المهمة
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

`@EnableCaching` بتقول لـ Spring "ابدأ تراقب الـ caching annotations".

---

## خامساً: الـ Redis Configuration Class

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {

        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

        JacksonJsonRedisSerializer<Object> serializer =
                new JacksonJsonRedisSerializer<>(Object.class);

        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(
                                new StringRedisSerializer()
                        )
                )
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(
                                serializer
                        )
                );

        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
    }
}
```

**ليه الـ Serialization ده مهم؟** Redis بيخزن كل حاجة كـ bytes — محتاج تقوله:

- الـ **key** يتحول لـ String عادي
- الـ **value** يتحول لـ JSON عشان تفهمه لما تجيبه تاني

---

## سادساً: استخدام الـ Cache في الـ Service

دي الجزء الأهم — الـ annotations:

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // ✅ @Cacheable: لو الداتا موجودة في cache → رجّعها من غير ما تروح للـ DB
    // لو مش موجودة → روح جيب من الـ DB وحطها في الـ cache
    @Cacheable(value = "products", key = "#id")
    public Product getProductById(Long id) {
        System.out.println("Fetching from DB..."); // مش هتشوف ده غير أول مرة
        return productRepository.findById(id).orElseThrow();
    }

    // ✅ @CachePut: دايماً بيروح للـ DB ويـ update الـ cache بعدها
    // بتستخدمه لما تعمل update على داتا
    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }

    // ✅ @CacheEvict: بيمسح الداتا من الـ cache
    // بتستخدمه لما تحذف item
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }

    // مسح كل الـ cache بتاع "products" دفعة واحدة
    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductsCache() {}
}
```

---

## الـ Object لازم يكون Serializable

```java
public class Product implements Serializable { // <-- مهم
    private Long id;
    private String name;
    private Double price;
    // getters & setters
}
```

ده عشان Java تقدر تحول الـ object لـ bytes وتبعته لـ Redis.

---

## الصورة الكاملة — إزاي بيشتغل كل ده مع بعض

```
Request جه لـ getProductById(1L)
        │
        ▼
@Cacheable بتسأل: في cache اسمه "products" فيه key = "1"؟
        │
   ┌────┴────┐
  YES        NO
   │          │
   ▼          ▼
رجّع         روح الـ DB
الداتا       جيب الداتا
من Redis     وحطها في Redis
             وبعدين رجّعها
```

---

## ملخص الـ Annotations

|Annotation|بتعمل إيه|
|---|---|
|`@Cacheable`|جيب من cache، لو مفيش روح DB وحط|
|`@CachePut`|روح DB دايماً وـ update الـ cache|
|`@CacheEvict`|امسح من الـ cache|
|`@Caching`|لو محتاج تجمع أكتر من annotation|

---

ده كل اللي محتاجه فعلاً كـ developer عشان تستخدم Redis في مشروعك. في أي خطوة محتاج توضيح أكتر؟ 