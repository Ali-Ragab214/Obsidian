# Section 09: Testing in Spring Boot (JUnit 5 & Mockito)

> **الملف ده مبني على مذكراتك في `Unit Testing/Day1.md` ومعايير اختبار تطبيقات Spring Boot.**  
> يغطي: هرم الاختبارات (Testing Pyramid)، الفرق بين Unit و Integration Testing، مكتبة Mockito، والـ Slice Testing Annotations (`@WebMvcTest`, `@DataJpaTest`, `@SpringBootTest`).

---

### Q1: What is the Testing Pyramid? Unit Testing vs Integration Testing vs End-to-End Testing.
**Level:** Junior  
**Related to:** Q2 (JUnit 5 & Mockito), Q3 (@SpringBootTest)

**Answer:**
**The Testing Pyramid (هرم الاختبارات)** هو نموذج هندسي يحدد التوزيع المثالي لأنواع الاختبارات في أي مشروع برمجي مستقر:

```
            / \
           /   \        End-to-End (E2E) Tests
          / E2E \       (بطيئة، مكلفة، تختبر النظام ككل عبر المتصفح/API)
         /───────\
        /         \     Integration Tests (@SpringBootTest)
       / Integrat. \    (متوسطة السرعة، تختبر تكامل الطبقات وقاعدة البيانات)
      /─────────────\
     /               \  Unit Tests (JUnit 5 + Mockito)
    /   Unit Tests    \ (سريعة جداً بالمللي ثانية، معزولة، تغطي Business logic)
   /───────────────────\
```

#### 1. Unit Testing (اختبار الوحدات):
- يختبر أصغر وحدة في الكود (دالة أو ميثود واحدة) **بشكل معزول تماماً عن العالم الخارجي**.
- لا يتم تشغيل Spring Context ولا يتم الاتصال بقاعدة بيانات أو شبكة.
- أي تبعية خارجية (Dependency) يتم استبدالها بـ **Mock Object** سريع في الذاكرة.

#### 2. Integration Testing (اختبار التكامل):
- يختبر كيف تعمل الطبقات المختلفة معاً (مثلاً: Controller يتصل بـ Service، و Service يكتب في Database حقيقية أو H2).
- يتطلب تحميل Spring Context أو جزء منه (Slice Test).

| وجه المقارنة | Unit Testing | Integration Testing |
| :--- | :--- | :--- |
| **سرعة التنفيذ** | فائقة (أقل من ثانية لآلاف الاختبارات) | أبطأ (ثوانٍ إلى دقائق لتحميل الفريم وورك) |
| **تحميل Spring Context** | ❌ لا يتم تحميله نهائياً | ✅ يتم تحميل Spring Container |
| **الاعتماديات الخارجية** | تُستبدل بالكامل بـ Mocks | تعمل مع قواعد بيانات حقيقية أو Testcontainers |
| **الهدف الأساسي** | اختبار صحة اللوجيك الرياضي والشرطي | اختبار صحة التكامل والـ Queries والـ Serialization |

**Short answer (30 seconds version):**
> "Unit tests isolate a single class using mocks without loading the Spring container, executing in milliseconds. Integration tests load the Spring ApplicationContext to verify real collaboration between layers, database queries, and configuration wiring."

---

### Q2: How do you write a Unit Test for the Service layer using Mockito? (`@Mock` vs `@InjectMocks`)
**Level:** Junior / Mid  
**Related to:** Q1, Q4 (@MockBean)

**Answer:**
مأخوذة من مذكراتك بملف `Unit Testing/Day1.md`:

عند اختبار الـ Service Layer، لا نريد فتح اتصال حقيقي بقاعدة البيانات، بل نتحكم في ردود الـ Repository لمحاكاة كل السيناريوهات المحتملة.

#### الكود العملي لاختبار `UserService`:
```java
@ExtendWith(MockitoExtension.class) // يفعل Mockito بدون تحميل Spring
class UserServiceTest {

    @Mock
    private UserRepository userRepository; // كائن وهمي بديل لقاعدة البيانات

    @InjectMocks
    private UserService userService; // الكلاس الحقيقي محل الاختبار، ويحقن Mockito جواه الـ Repository الوهمي

    @Test
    @DisplayName("Should return user when valid ID is provided")
    void shouldReturnUser_WhenValidId() {
        // 1. Arrange (تجهيز التوقعات والبيانات)
        Long userId = 1L;
        User mockUser = User.builder().id(userId).email("ali@test.com").name("Ali").build();
        
        // عندما يستدعي اللوجيك userRepository.findById(1L)، أرجع له mockUser
        Mockito.when(userRepository.findById(userId)).thenReturn(Optional.of(mockUser));

        // 2. Act (تنفيذ الميثود الحقيقية)
        User result = userService.getUserById(userId);

        // 3. Assert (التأكد من صحة النتيجة)
        Assertions.assertNotNull(result);
        Assertions.assertEquals("ali@test.com", result.getEmail());
        
        // التأكد من أن الميثود في الـ Mock تم استدعاؤها مرة واحدة فقط
        Mockito.verify(userRepository, Mockito.times(1)).findById(userId);
    }

    @Test
    @DisplayName("Should throw ResourceNotFoundException when user does not exist")
    void shouldThrowException_WhenUserNotFound() {
        // Arrange
        Mockito.when(userRepository.findById(99L)).thenReturn(Optional.empty());

        // Act & Assert
        Assertions.assertThrows(ResourceNotFoundException.class, () -> {
            userService.getUserById(99L);
        });
    }
}
```

| Annotation | الوظيفة |
| :--- | :--- |
| **`@Mock`** | ينشئ كائن Dummy فارغ تماماً؛ ميثوداته ترجع `null` أو `false` ما لم تبرمجها بـ `when().thenReturn()`. |
| **`@InjectMocks`** | ينشئ نسخة حقيقية من الكلاس المراد اختباره، ويحقن بداخله كل الـ Mocks المعرفة بـ `@Mock`. |

**Short answer (30 seconds version):**
> "In service unit tests with Mockito, @Mock creates simulated dummy instances of dependencies like repositories. @InjectMocks instantiates the real service under test and injects the created mocks into it, allowing testing in complete isolation without starting Spring or a database."

---

### Q3: What is `@SpringBootTest`? When should you avoid using it?
**Level:** Mid  
**Related to:** Q1, Q4 (@WebMvcTest)

**Answer:**
`@SpringBootTest` هي الـ Annotation الأكبر والأشمل في اختبارات Spring:
- تقوم بتحميل **الـ كامل ApplicationContext للتطبيق** كما لو كنت تشغله في الحقيقة.
- تبحث عن `@SpringBootApplication`، وتشغل كل الـ Beans والـ Configurations والـ Auto-configurations.

#### متى تستخدمها؟
- في **الـ Full Integration Tests**؛ مثلاً اختبار يرسل HTTP Request، ويمر عبر الـ Filter Chain والـ Controller والـ Service، ويكتب في قاعدة بيانات تجريبية.

#### لماذا يجب تجنب استخدامها في كل اختبار؟ (Warning ⚠️):
- **بطيئة جداً (Heavy Startup Cost):** تحميل التطبيق بالكامل قد يستغرق 5-10 ثوانٍ لكل Test Class. لو كان لديك 500 اختبار في المشروع، سيستغرق الـ CI/CD Pipeline ساعة كاملة فقط لتشغيل الاختبارات!
- **الحل الهندسي:** استخدام **Test Slices** (اختبار شرائح معينة فقط من التطبيق) مثل `@WebMvcTest` و `@DataJpaTest`.

**Short answer (30 seconds version):**
> "@SpringBootTest bootstraps the entire Spring ApplicationContext for comprehensive integration testing. It should be avoided for routine testing because it is slow and resource-heavy; slice testing annotations should be preferred to test isolated architectural layers."

---

### Q4: Explain Spring Boot Slice Testing: `@WebMvcTest`, `@DataJpaTest`, and `@MockBean`.
**Level:** Mid / Senior  
**Related to:** Q3

**Answer:**
**Slice Testing (اختبار الشرائح)** هي ميزة ذكية جداً في Spring Boot تتيح لك تحميل جزء محدد فقط من الـ ApplicationContext الخاص بطبقة معينة دون تحميل باقي التطبيق:

#### 1. `@WebMvcTest` (لاختبار الـ Controller Layer فقط):
- يقوم بتحميل المكونات المسؤولة عن الويب فقط: `DispatcherServlet`, `Controller`, `Filters`, `Validation`, و `Jackson`.
- لا يقوم بتحميل الـ Services أو الـ Repositories نهائياً (مما يجعله سريعاً جداً!).
- نستخدم معه **`MockMvc`** لإرسال HTTP Requests افتراضية وفحص الـ Status Code والـ JSON Path:

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean // يسجل Mock Service داخل الـ Slice Context
    private UserService userService;

    @Test
    void shouldReturnUserJson() throws Exception {
        Mockito.when(userService.getUserById(1L))
               .thenReturn(new UserDto(1L, "Ali", "ali@test.com"));

        mockMvc.perform(get("/api/v1/users/1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data.name").value("Ali"))
                .andExpect(jsonPath("$.data.email").value("ali@test.com"));
    }
}
```

#### 2. `@DataJpaTest` (لاختبار الـ Repository Layer فقط):
- يقوم بتحميل الـ Entities و الـ Repositories ومحرك Hibernate فقط.
- يجهز تلقائياً قاعدة بيانات في الذاكرة (In-memory H2) بدلاً من قاعدة بيانات الإنتاج.
- **تلقائياً يجعل كل Test Method بـ `@Transactional` ويقوم بعمل Rollback فوري بعد انتهاء الاختبار** لكي يترك قاعدة البيانات نظيفة للاختبار التالي!

#### 3. ما الفرق بين `@Mock` و `@MockBean`؟ (سؤال إنترفيو شهير!):
| المعيار | `@Mock` (Mockito) | `@MockBean` (Spring Boot Test) |
| :--- | :--- | :--- |
| **المصدر** | مكتبة Mockito النقية | حزمة `org.springframework.boot.test.mock.mockito` |
| **علاقته بـ Spring** | لا يعلم أي شيء عن Spring Context | يضع الـ Mock داخل الـ Spring ApplicationContext |
| **الاستخدام** | في الـ Fast Unit Tests البسيطة | في الـ Integration Tests مع `@WebMvcTest` أو `@SpringBootTest` لاستبدال Real Bean بـ Mock |

**Short answer (30 seconds version):**
> "Slice testing loads only components relevant to a specific architectural layer. @WebMvcTest focuses exclusively on Spring MVC controllers and MockMvc without loading services. @DataJpaTest configures only JPA repositories with an in-memory database and auto-rollback transactions. @MockBean places a mock directly inside the Spring ApplicationContext to satisfy bean dependencies."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What is `Testcontainers` and why is it preferred over H2 for JPA testing? (Testcontainers runs real Docker containers like PostgreSQL, avoiding subtle SQL dialect incompatibilities of H2).*
2. *What does `@DirtiesContext` do? (Instructs Spring to destroy and recreate the ApplicationContext after the test, used when a test mutates bean state).*
3. *What is the difference between `spy()` and `mock()` in Mockito? (`mock` simulates behavior completely; `spy` wraps a real instance, calling real methods unless stubbed).*

---

## 🔗 System Design Connection: CI/CD Pipelines & Testcontainers

### Real-world System Design Application:
في بيئات الـ CI/CD الحديثة (GitHub Actions / GitLab CI)، الاختبارات هي خط الدفاع الأول قبل عمل Deploy للإنتاج:
1. **The In-Memory DB Trap (H2 vs Postgres):**
   - الاعتماد على قاعدة بيانات H2 في الاختبارات خطر جداً لأن H2 لا تدعم ميزات Postgres المتقدمة (مثل JSONB, CTEs, Window Functions, Case-sensitive collations).
2. **Testcontainers Solution:**
   - أثناء مرحلة الـ Integration Test في الـ CI/CD، يقوم Testcontainers بتشغيل حاوية Docker حقيقية لـ PostgreSQL أو Kafka أو Redis في ثوانٍ، وتنفيذ الاختبارات ضدها ثم إتلافها؛ مما يضمن تطابق بيئة الاختبار مع بيئة الإنتاج بنسبة 100%.

```
  Git Push (PR) ──► CI/CD Pipeline (GitHub Actions)
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
     Fast Unit Tests             Integration Tests
     (JUnit 5 + Mockito)         (@SpringBootTest)
     Time: 15 seconds                    │
                                         ▼
                                  Spins Docker Container
                                  (Real PostgreSQL 16)
                                         │
                                         ▼
                               All Passed ──► Safe to Deploy!
```
