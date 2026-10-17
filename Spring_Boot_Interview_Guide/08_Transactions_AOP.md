# Section 08: Transactions & AOP (Aspect-Oriented Programming)

> **الملف ده مبني على مذكراتك في `Spring Data jpa/15-Transactions.md`، `Courses_Pilot/NOTES/6-Transactional & @RequiredArgsConstructor.md`، وملاحظاتك حول ACID في مجلد `System Design/15-ACID.md`.**  
> يغطي: كيفية عمل `@Transactional` عبر البروكسي (Proxy Pattern)، مستويات الـ Propagation والـ Isolation، الحالات التي يفشل فيها التراجع (Rollback gotchas)، ومفاهيم الـ AOP الأساسية.

---

### Q1: What is `@Transactional` and how does it work under the hood? (Spring AOP Proxy)
**Level:** Mid  
**Related to:** Q2 (Propagation), Q4 (Self-invocation Gotcha), Q6 (AOP Concepts)

**Answer:**
مأخوذة من مذكراتك بملف `Spring Data jpa/15-Transactions.md`:

`@Transactional` هي أداة Spring للتحكم في معاملات قواعد البيانات (Database Transactions) برمجياً وبشكل تصريحي (**Declarative Transaction Management**).

#### كيف تعمل تحت الغطاء؟ (The Proxy Mechanism):
عندما تضع `@Transactional` على كلاس أو ميثود، Spring لا يستدعي الكود الخاص بك مباشرة، بل ينشئ **CGLIB / Dynamic Proxy Wrapper** يحيط بالـ Bean الخاص بك:

```
  Caller (e.g. UserController)
              │
              ▼
  ┌────────────────────────────────────────────────────────┐
  │              Spring Transactional Proxy                │
  │                                                        │
  │  1. TransactionManager.getTransaction()                │
  │     - يفتح Connection مع قاعدة البيانات                │
  │     - connection.setAutoCommit(false)                  │
  │                                                        │
  │  2. استدعاء الكود الحقيقي الخاص بك:                    │
  │     try {                                              │
  │         userService.transferMoney();                   │
  │         3. connection.commit(); // تم بنجاح!           │
  │     } catch (RuntimeException | Error ex) {            │
  │         4. connection.rollback(); // تراجع فوري!       │
  │         throw ex;                                      │
  │     } finally {                                        │
  │         5. connection.close(); // إعادة للـ Pool       │
  │     }                                                  │
  └────────────────────────────────────────────────────────┘
```

**Short answer (30 seconds version):**
> "@Transactional uses Spring AOP dynamic proxies to manage database transactions declaratively. The proxy intercepts the method invocation, begins a database transaction by setting autoCommit to false, executes the target method, and automatically commits on successful completion or rolls back if an unhandled RuntimeException occurs."

---

### Q2: What are Transaction Propagation types? Explain `REQUIRED` vs `REQUIRES_NEW`.
**Level:** Mid / Senior  
**Related to:** Q1, Q3 (Isolation Levels)

**Answer:**
الـ **Propagation (انتشار المعاملة)** يحدد: **ماذا يحدث لو استدعت ميثود ذات Transaction ميثوداً أخرى ذات Transaction أيضاً؟ هل تنضمان لنفس المعاملة أم تنشأ معاملة جديدة؟**

Spring يوفر 7 أنواع Propagation، ولكن أهم نوعين يطرحان في 99% من المقابلات هما:

#### 1. `Propagation.REQUIRED` (الافتراضي - Default):
- **القاعدة:** "لو فيه Transaction قائمة بالفعل، انضم إليها وشاركها؛ لو مفيش أي Transaction، ابدأ Transaction جديدة".
- **النتيجة:** كل العمليات تشترك في نفس الـ Commit والـ Rollback. لو فشلت أي خطوة فرعية، تتراجع العملية بالكامل!

#### 2. `Propagation.REQUIRES_NEW` (معاملة مستقلة معلقة):
- **القاعدة:** "دائماً ابدأ Transaction جديدة مستقلة تماماً؛ لو فيه Transaction قائمة، قم بتعليقها (Suspend) مؤقتاً حتى تنتهي المعاملة الجديدة أولاً".
- **الاستخدام المثالي:** لعمليات الـ **Audit Logs** أو الـ **Payment Notification Record**؛ حيث نريد حفظ سجل المحاولة حتى لو فشلت العملية الرئيسية وتراجعت!

```java
@Service
public class OrderService {
    @Autowired private PaymentService paymentService;
    @Autowired private AuditLogService auditLogService;

    @Transactional // REQUIRED (Transaction A)
    public void checkoutOrder(Order order) {
        orderRepository.save(order);
        
        try {
            paymentService.processPayment(order); // تشترك في Transaction A
        } catch (PaymentFailedException ex) {
            // نريد تسجيل أن المحاولة فشلت حتى لو تراجع الأوردر!
            auditLogService.logFailure(order.getId()); // REQUIRES_NEW (Transaction B مستقلة!)
            throw ex; // Transaction A ستتراجع، ولكن Transaction B تم حفظها بالفعل!
        }
    }
}
```

| نوع الـ Propagation | السلوك لو وُجدت Transaction سابقة | السلوك لو لم توجد أي Transaction |
| :--- | :--- | :--- |
| **`REQUIRED` (Default)** | ينضم إليها | ينشئ Transaction جديدة |
| **`REQUIRES_NEW`** | يعلقها مؤقتاً وينشئ جديدة مستقلة | ينشئ Transaction جديدة |
| **`MANDATORY`** | ينضم إليها | يرمي Exception فوراً (يرفض العمل بدون ترانزاكشن) |
| **`SUPPORTS`** | ينضم إليها | يعمل بشكل عادي بدون ترانزاكشن |
| **`NOT_SUPPORTED`** | يعلقها ويعمل بدون ترانزاكشن | يعمل بدون ترانزاكشن |
| **`NEVER`** | يرمي Exception لو وُجدت | يعمل بدون ترانزاكشن |

**Short answer (30 seconds version):**
> "Transaction propagation defines the transaction boundaries across collaborating methods. REQUIRED (default) joins the existing physical transaction or creates a new one if none exists. REQUIRES_NEW always suspends the current transaction and executes in a completely independent, isolated transaction with its own commit/rollback lifecycle."

---

### Q3: What are Transaction Isolation Levels? What read phenomena do they prevent?
**Level:** Senior  
**Related to:** System Design ACID (15-ACID.md)

**Answer:**
مستوحاة من مذكراتك بملف `System Design/15-ACID.md`:

الـ **Isolation (العزل)** يحدد درجة حماية المعاملات المتزامنة التي تقرأ وتعدل نفس البيانات في نفس اللحظة لمنع تشوه البيانات.

#### الظواهر غير المرغوبة (Read Phenomena):
1. **Dirty Read:** معاملة تقرأ بيانات غير مؤكدة تم تعديلها بواسطة معاملة أخرى لم تنته بعد، ثم قامت المعاملة الأخرى بعمل Rollback!
2. **Non-Repeatable Read:** معاملة تقرأ نفس الصف مرتين داخل نفس الترانزاكشن، فتحصل على قيم مختلفة لأن معاملة أخرى قامت بعمل Update و Commit في المنتصف.
3. **Phantom Read:** معاملة تستعلم عن مجموعة صفوف (`WHERE age > 30`)، فتعيد قراءة الاستعلام وتجد صفوفاً جديدة ظهرت لأن معاملة أخرى قامت بعمل Insert و Commit في المنتصف.

#### مستويات العزل الأربعة ومعالجة الظواهر:
| مستوى العزل (Isolation Level) | Dirty Read | Non-Repeatable Read | Phantom Read | الأداء |
| :--- | :---: | :---: | :---: | :---: |
| **`READ_UNCOMMITTED`** | ❌ مهدد | ❌ مهدد | ❌ مهدد | الأسرع (أقل حماية) |
| **`READ_COMMITTED`** (Postgres/Oracle Default) | ✅ محمي | ❌ مهدد | ❌ مهدد | متوازن وممتاز |
| **`REPEATABLE_READ`** (MySQL InnoDB Default) | ✅ محمي | ✅ محمي | ❌ مهدد | حماية عالية |
| **`SERIALIZABLE`** | ✅ محمي | ✅ محمي | ✅ محمي | الأبطأ تماماً (أقفال كاملة) |

في كود Spring نحدده هكذا:
```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void calculateFinancialReport() { ... }
```

**Short answer (30 seconds version):**
> "Isolation levels control data visibility among concurrent transactions. Standard ANSI SQL defines four levels: READ_UNCOMMITTED (allows dirty reads), READ_COMMITTED (prevents dirty reads), REPEATABLE_READ (prevents dirty and non-repeatable reads), and SERIALIZABLE (fully prevents phantom reads via strict locking at the cost of concurrency)."

---

### Q4: When does `@Transactional` FAIL to roll back? (Common Gotchas & Self-invocation)
**Level:** Mid / Senior  
**Related to:** Q1 (Proxy), Q5 (Rollback Rules)

**Answer:**
هذا هو السؤال المفضل لاختبار هل المطور يفهم Spring بعمق أم مجرد مستخدم سطحي:

#### 1. الـ Self-Invocation Problem (استدعاء الميثود داخلياً):
لو استدعيت ميثود عليها `@Transactional` من داخل نفس الكلاس:
```java
@Service
public class UserService {

    public void registerUser(User user) {
        // ❌ استدعاء داخلي مباشر!
        saveWithTransaction(user); 
    }

    @Transactional
    public void saveWithTransaction(User user) {
        userRepository.save(user);
        throw new RuntimeException("Error!"); // لن يحدث Rollback نهائياً!
    }
}
```
- **لماذا فشل؟** لأن الاستدعاء الداخلي يمر عبر مرجع `this` المباشر في الجافا، متجاوزاً الـ Spring Proxy بالكامل! بما أن الـ Proxy لم يعترض الطلب، فلن يتم فتح أو إدارة أي Transaction.
- **الحل:** وضع `@Transactional` على الميثود المستدعاة من الخارج (`registerUser`)، أو فصل الميثود في كلاس Service مستقل.

#### 2. رمي Checked Exception افتراضياً:
افتراضياً، Spring يقوم بعمل Rollback **فقط للـ Unchecked Exceptions** (أي كلاس يرث `RuntimeException` أو `Error`).
لو رميت **Checked Exception** (مثل `IOException` أو `SQLException` أو `Exception` عادية):
```java
@Transactional
public void process() throws IOException {
    userRepository.save(user);
    throw new IOException("Disk error"); // ⚠️ Spring سيقوم بعمل COMMIT ولن يتراجع!
}
```
- **الحل الصحيح دائماً:** حدد خاصية `rollbackFor`:
```java
@Transactional(rollbackFor = Exception.class) // يضمن التراجع مع أي Exception مهما كان نوعه!
public void process() throws Exception { ... }
```

#### 3. استخدام `@Transactional` على ميثود `private`:
- الـ Proxies في Spring لا تستطيع عمل Override للـ `private methods`. وضع الأنوتيشن فوق ميثود خاصة سيتم تجاهله بصمت بدون تحذير ولن تعمل الترانزاكشن.

**Short answer (30 seconds version):**
> "@Transactional fails to roll back in three primary scenarios: 1) Self-invocation (calling a transactional method from within the same class bypasses the Spring proxy), 2) Throwing checked exceptions (Spring only rolls back on RuntimeException by default, resolved by specifying rollbackFor = Exception.class), and 3) Applying it to private methods where proxies cannot intercept calls."

---

### Q5: What is AOP (Aspect-Oriented Programming)? Explain Aspect, Advice, Pointcut, and JoinPoint.
**Level:** Mid  
**Related to:** Q1 (Transactional Proxy)

**Answer:**
في البرمجة كائنية التوجه (OOP)، نقسم النظام إلى كلاسات رأسية (User, Order, Payment). ولكن توجد وظائف عامة تتقاطع أفقياً مع جميع هذه الطبقات تسمى **Cross-Cutting Concerns** مثل:
- الـ Logging (تسجيل العمليات)
- الـ Security (فحص الصلاحيات)
- الـ Transaction Management (إدارة المعاملات)
- قياس زمن تنفيذ الأوامر (Performance Profiling)

**AOP** هي فلسفة برمجية تسمح بفصل هذه الوظائف المشتركة في كلاسات منفصلة وحقنها تلقائياً بدون تلويث الـ Business Logic.

#### مصطلحات الـ AOP الأساسية بالتشبيه والأمثلة:
1. **Aspect (الجانب / المكون):**
   - الكلاس الذي يجمع المنطق المشترك (يُعلم بـ `@Aspect` و `@Component`).
2. **JoinPoint (نقطة الربط):**
   - أي نقطة في تنفيذ البرنامج يمكن اعتراضها (في Spring AOP هي دائماً استدعاء Method).
3. **Pointcut (محدد النقاط):**
   - التعبير البرمجي (Expression) الذي يحدد بدقة **أي الميثودز نريد استهدافها** (مثل: "كل الميثودز في باكيج الـ Service").
4. **Advice (الإجراء المنفذ):**
   - الكود الفعلي الذي سينفذ، وتوقيت تنفيذه بالنسبة للميثود الأصلية:
     - `@Before`: قبل تنفيذ الميثود.
     - `@AfterReturning`: بعد انتهاء الميثود بنجاح.
     - `@AfterThrowing`: لو رمت الميثود Exception.
     - `@After`: دائماً بعد انتهاء الميثود (مثل `finally`).
     - `@Around` (الأقوى والأكثر استخداماً): يحيط بالميثود بالكامل ويتحكم في تنفيذها وقياس وقتها.

#### كود عملي: Aspect لقياس زمن تنفيذ الـ Services:
```java
@Aspect
@Component
@Slf4j
public class PerformanceMonitoringAspect {

    // Pointcut: استهداف جميع الميثودز داخل باكيج الـ service
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayerMethods() {}

    // Advice: قياس الوقت يحيط بالميثود (Around)
    @Around("serviceLayerMethods()")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        try {
            Object result = joinPoint.proceed(); // تنفيذ الميثود الأصلية
            return result;
        } finally {
            long duration = System.currentTimeMillis() - startTime;
            log.info("Method [{}] executed in {} ms", joinPoint.getSignature().toShortString(), duration);
        }
    }
}
```

**Short answer (30 seconds version):**
> "AOP modularizes cross-cutting concerns (logging, security, transactions). An Aspect encapsulates the behavior. A JoinPoint is the execution point intercepted (method call). A Pointcut is an expression matching target JoinPoints. An Advice is the action executed at that point (@Before, @After, or @Around)."

---

## 📌 Follow-up Questions the Interviewer May Ask
1. *What is the difference between JDK Dynamic Proxies and CGLIB in Spring? (JDK proxies require interfaces; CGLIB creates subclasses and is default in Spring Boot).*
2. *What is the purpose of `@Transactional(readOnly = true)`? (Optimizes performance by telling Hibernate to skip dirty checking and informs read-replicas).*
3. *How do distributed transactions work across microservices? (Two-Phase Commit vs Saga Pattern).*

---

## 🔗 System Design Connection: ACID Properties & The Saga Pattern

### Real-world System Design Application:
في المونوليث (Monolith)، تضمن قاعدة البيانات المركزية خواص الـ **ACID** عبر `@Transactional` المحلية:
- **Atomicity:** إما أن تنفذ كل الخطوات (خصم الرصيد + تحديث المخزن) أو يتراجع كل شيء.
- **Consistency:** الانتقال من حالة صالحة لقاعدة البيانات إلى حالة صالحة أخرى.
- **Isolation:** عزل المعاملات المتزامنة.
- **Durability:** بمجرد عمل Commit، البيانات محفوظة على الديسك حتى لو انقطعت الكهرباء.

#### المشكلة في الـ Microservices (Distributed Transactions):
عند تقسيم النظام لخدمات مستقلة (`OrderService` بقاعدة بياناتها، و `PaymentService` بقاعدتها، و `InventoryService` بقاعدتها)، تصبح `@Transactional` عاجزة تماماً لأنها تعمل على اتصال DB محلي واحد فقط!

**الحل الهندسي: The Saga Pattern (نمط المعاملات الموزعة):**
نقسم المعاملة الكبيرة إلى سلسلة من المعاملات المحلية الصغيرة غير المتزامنة عبر الـ Message Broker (Kafka)، مع وجود **Compensating Transactions (معاملات تعويضية تراجعية)** في حالة الفشل:

```
  1. Create Order (Pending)
             │
             ▼
      Order Service ──► Kafka: OrderCreatedEvent
                            │
                            ▼
                     Payment Service
                      ├── نجاح: خصم المبلغ ──► Kafka: PaymentSuccess ──► Complete Order!
                      └── فشل: رصيد غير كاف ──► Kafka: PaymentFailed
                                                    │
                                                    ▼
                                           Order Service (Compensating Action)
                                           - Cancel Order & Release Stock!
```
