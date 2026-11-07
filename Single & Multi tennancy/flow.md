تمام يا Ali. هنلغي التشخيص والأسئلة تماماً، وهنبدأ من الصفر كأنك أول مرة تسمع عن Multi-Tenancy.

عايزك تعتبر إن هدفنا مش إنك تفهم الصورة بس، لكن إنك بعد الشرح تعرف تعمل Architecture مشابهة بنفسك باستخدام Java وSpring Boot وHibernate وPostgreSQL.

هنمشي من الصورة الكبيرة، وبعدها نفك كل Component، ثم نمشي مع Request حقيقي، وفي النهاية نرتب خطوات بناء المشروع.

# 1. نفهم الأول: يعني إيه Multi-Tenancy؟

تخيّل إن عندك شركة بتقدم نظام لإدارة المنتجات والمخازن لعدة شركات.

عندك 3 شركات:

- Alpha Company
    
- Beta Company
    
- Gamma Company
    

كل شركة بتستخدم نفس التطبيق، لكن كل شركة لازم تشوف بياناتها فقط.

مثلاً:

|Product|Tenant|Price|
|---|---|---|
|Keyboard|Alpha|100|
|Mouse|Alpha|50|
|Laptop|Beta|500|
|Monitor|Gamma|300|

المطلوب:

- Alpha تشوف Keyboard وMouse فقط.
    
- Beta تشوف Laptop فقط.
    
- Gamma تشوف Monitor فقط.
    

المشكلة: الشركات كلها بتستخدم نفس الـ Application ونفس الـ Database، فإزاي نضمن إن كل شركة تشوف بياناتها فقط؟

هنا بيظهر مفهوم Multi-Tenancy.

Tenant: هو العميل أو الشركة أو المؤسسة اللي بتستخدم نظام الـ SaaS، وليها بيانات معزولة عن باقي العملاء.

## 1.1 استراتيجيات Multi-Tenancy

فيه 3 استراتيجيات أساسية:

[Spring Boot Multi-tenant Architecture Overview | by Kostiantyn Dementiev | Medium](https://images.openai.com/static-rsc-4/8YS9akbf1fkCTxr2BuLEOD1T1tDTKjjk7gd1bgTLSqWsMhhIB4p0-1vgrmj6BpZ5ry5THCYufoeL8JcDQpsZCQjq64_4wIL602BC5uHhnbJnWpXi5_IfN-eSJLHGVKMrTGEi5jlKUa0lJt5q-ntlsEceBe5znSzhFijBDbp33l4?purpose=inline)

1. Database per Tenant

كل Tenant ليه Database منفصلة تماماً.

Alpha → Database Alpha

Beta → Database Beta

Strong isolation

[Schema-Based Multi-Tenancy with Spring Boot & Hibernate | by Oguz Topal | Medium](https://images.openai.com/static-rsc-4/3odTpkjFdVUdDrOIIiBRhOW6m3Kkg6Td5z3c39QF-55Zt0FIQ53csoB05DwYYMY8Yd7O31CCDaq9UwoQBUjsZejXslXR--hi71ibkMUibAM8BcfYgOO5xudrt7fkoJPYtu0aImWgNLyebVNGTF-7zJeKwJLGcgXdscaHA2LYdXM?purpose=inline)

2. Schema per Tenant

نفس الـ Database، لكن كل Tenant ليه Schema منفصلة.

Alpha → schema_alpha

Beta → schema_beta

[Multi-Tenant Database Architecture: Concepts, Patterns, and Real-World Examples](https://images.openai.com/static-rsc-4/Sg8egj5lZY3HH7wV1k6DLLfVZvpl1OpmR9bmgmS4OTHT6I-i36my97d6dNNEsrBdLagRxIY2O-TRA8aY_Lh47TU8zuVgD37AISwUZ3m49rrPNf51SKJK0YeyYIrGCXKt4dtBr8tGc43lEhoc2bTbVAIYBWE0g4YecT_u_z12phw?purpose=inline)

3. Shared Database + Shared Schema

كل الشركات في نفس Database ونفس Tables.

لكن كل Record بيكون مرتبط بـ `tenant_id`.

Approach in your course

إحنا هنركز على الاستراتيجية الثالثة، لأنها الموجودة في الـ Diagram بتاعك.

مثلاً جدول `products`:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    tenant_id VARCHAR(100) NOT NULL,
    product_name VARCHAR(255),
    quantity INT,
    price DECIMAL(10,2)
);
```

لاحظ إننا أضفنا:

```
tenant_id
```

ده المفتاح اللي بيحدد كل Record تابع لمين.

لكن وجود العمود ده لوحده مش كفاية؛ لازم نضمن إن كل Query بتتنفذ بترجع بيانات الـ Tenant الصحيح.

وده بالضبط اللي الـ Flow بتاعنا بيعمله.

# 2. الصورة الكبيرة للـ Architecture

خلينا نبص على رحلة الـ Request:

Angular Client

X-Tenant-ID: alpha

TenantFilter

Extract Tenant ID → ThreadLocal

ProductController

Receives HTTP Request

ProductService + AOP Aspect

Reads Tenant ID and enables Hibernate Filter

Hibernate ORM

Automatically adds tenant condition to SQL

PostgreSQL

Returns Alpha's records only

الفكرة الأساسية:

نحدد Tenant مرة واحدة في بداية الـ Request، ونخليه متاح لباقي أجزاء التطبيق، وبعدها Hibernate يستخدمه لعزل البيانات تلقائياً.

وده معناه إننا مش محتاجين نكتب `WHERE tenant_id = ?` بإيدنا في كل Repository Query، طالما الـ Hibernate Filter مطبق بشكل صحيح على العمليات المدعومة.

# 3. نفهم كل Component من الصفر

هنبدأ بأهم 4 مفاهيم في الصورة، لأنهم أساس الـ Architecture.

## 3.1 Servlet Filter — بوابة الدخول

أول حاجة لازم تفهمها هي إن Spring Boot مبني فوق Servlet Stack.

لما HTTP Request يوصل للتطبيق، مش بيروح للـ Controller مباشرة.

بيعدي على مجموعة اسمها:

Servlet Filter Chain

الـ Filter هو Component بيقدر يعترض الـ Request قبل وصوله للـ Controller.

أمثلة لاستخدامه:

- Authentication.
    
- Logging.
    
- CORS.
    
- Extracting Tenant ID.
    
- Request validation.
    

في مشروعنا هننشئ:

`TenantFilter`

وظيفته:

1. يستقبل الـ Request.
    
2. يقرأ `X-Tenant-ID`.
    
3. يخزن القيمة في ThreadLocal.
    
4. يسمح للـ Request يكمل.
    
5. ينظف ThreadLocal بعد انتهاء الـ Request.
    

مثال:

```
GET /api/products HTTP/1.1
Host: alpha.stockapp.com
X-Tenant-ID: alpha
Authorization: Bearer eyJhbGci...
```

الـ Filter يقرأ:

```
String tenantId =
    request.getHeader("X-Tenant-ID");
```

دلوقتي عرفنا إن الـ Request تابع لـ Alpha.

لكن السؤال: هنخزن القيمة دي فين عشان الـ Service والـ Aspect يقدروا يوصلولها؟

هنا بيظهر المفهوم التاني.

## 3.2 ThreadLocal — تخزين قيمة خاصة بكل Thread

الأول نفهم يعني إيه Thread.

الـ Thread هو مسار تنفيذ داخل البرنامج.

لما التطبيق يستقبل Requests متعددة، ممكن كل Request يتنفذ على Thread مختلف من Thread Pool.

تخيّل:

|Thread|Request|Tenant|
|---|---|---|
|Thread-42|Request 1|Alpha|
|Thread-43|Request 2|Beta|
|Thread-44|Request 3|Gamma|

المشكلة إننا محتاجين نخزن Tenant ID بطريقة تخلي كل Thread يقدر يوصل لقيمته من أي Layer داخل التطبيق.

`ThreadLocal` عبارة عن آلية في Java بتخلي كل Thread يحتفظ بنسخة مستقلة من قيمة معينة.

يعني:

- Thread-42 يقرأ Alpha.
    
- Thread-43 يقرأ Beta.
    
- Thread-44 يقرأ Gamma.
    

من غير ما القيم تتداخل.

ThreadLocal Storage

Thread-42

# Alpha

Thread-43

# Beta

Thread-44

# Gamma

كل Thread ليه قيمة مستقلة داخل نفس ThreadLocal variable.

نعمل Class اسمه `TenantContext`:

```
public class TenantContext {

    private static final ThreadLocal<String> CURRENT =
            new ThreadLocal<>();

    public static void setCurrentTenant(String tenant) {
        CURRENT.set(tenant);
    }

    public static String getCurrentTenant() {
        return CURRENT.get();
    }

    public static void clear() {
        CURRENT.remove();
    }
}
```

لاحظ 3 Methods:

|Method|الوظيفة|
|---|---|
|`set()`|تخزين Tenant ID|
|`get()`|قراءة Tenant ID|
|`remove()`|إزالة القيمة من ThreadLocal|

نقطة مهمة جداً: ليه `remove()` مش اختيارية؟

لأن Spring عادةً بيستخدم Thread Pool.

يعني Thread-42 بعد ما يخلص Request بتاع Alpha، مش بالضرورة يتدمر.

ممكن يتعاد استخدامه في Request جديد تابع لـ Beta.

لو نسينا `remove()` ممكن الـ Request الجديد يلاقي قيمة قديمة.

وده ممكن يؤدي لتسريب بيانات بين العملاء.

دي من أخطر الأخطاء في Multi-Tenancy.

## 3.3 AOP — Aspect Oriented Programming

دي نقطة مهمة جداً في الـ Diagram.

الأول نفهم AOP.

`AOP` اختصار لـ Aspect Oriented Programming.

فكرتها إنك تقدر تضيف سلوك إضافي حوالين تنفيذ Methods معينة من غير ما تحط الكود ده جوا كل Method بنفسك.

مثلاً عندك:

```
public List<Product> findAll() {
    return productRepository.findAll();
}
```

إنت عايز قبل تنفيذ `findAll()` يحصل الآتي:

1. اقرأ Tenant ID من ThreadLocal.
    
2. افتح Hibernate Session الحالية.
    
3. فعّل Hibernate Filter.
    
4. حدد قيمة Tenant ID.
    

بدل ما تكرر الكلام ده في كل Service Method، تقدر تستخدم AOP.

### يعني إيه Aspect؟

الـ Aspect عبارة عن Class بيحتوي على Logic إضافي بيتنفذ عند نقاط معينة في البرنامج.

مثلاً:

```
@Aspect
@Component
public class TenantAspect {

    @Before("execution(* com.app.service..*(..))")
    public void applyTenant() {
        String tenant =
            TenantContext.getCurrentTenant();

        // Enable Hibernate Filter here
    }
}
```

خلينا نفهم Annotation:

`@Before`

معناها إن الكود الموجود في الـ Advice هيتنفذ قبل الـ Method المستهدفة.

أما:

`execution(* com.app.service..*(..))`

دي Pointcut Expression، بتحدد الـ Methods اللي الـ Aspect هيشتغل قبلها.

بالتالي لما ننفذ:

```
productService.findAll();
```

الـ Aspect يتدخل قبل تنفيذ الـ Method.

ملحوظة: الـ AOP مش بديل للـ Filter. كل واحد ليه مسؤولية مختلفة:

- Filter: يحدد Tenant ID في بداية الـ HTTP Request.
    
- Aspect: يستخدم Tenant ID لتجهيز Hibernate قبل تنفيذ عمليات الـ Service.
    

## 3.4 Hibernate @Filter

دي أهم حلقة بتربط كل اللي فات بالـ Database.

Hibernate هو ORM (Object Relational Mapping).

يعني بيساعدنا نتعامل مع Database Tables عن طريق Java Objects وEntities.

مثلاً:

```
@Entity
@Table(name = "products")
public class Product {

    @Id
    private Long id;

    private String productName;

    @Column(name = "tenant_id")
    private String tenantId;
}
```

عايزين نقول لـ Hibernate:

أي Query على Product لازم تراعي Tenant ID الحالي.

نستخدم `@FilterDef` و`@Filter`:

```
@FilterDef(
    name = "tenantFilter",
    parameters = @ParamDef(
        name = "tenantId",
        type = String.class
    )
)
@Filter(
    name = "tenantFilter",
    condition = "tenant_id = :tenantId"
)
@Entity
public class Product {
    // Fields
}
```

هنا حصل إيه؟

- `@FilterDef`: عرّفنا Filter اسمه `tenantFilter`، وعنده Parameter اسمه `tenantId`.
    
- `@Filter`: حددنا الشرط اللي Hibernate هيضيفه للـ SQL.
    

لكن خلي بالك من حاجة مهمة:

تعريف الـ Filter مش معناه إنه اشتغل تلقائياً.

لازم نفعّله على الـ Hibernate Session.

مثلاً:

```
session.enableFilter("tenantFilter")
       .setParameter("tenantId", "alpha");
```

كده Hibernate يعرف إن الـ Filter المطلوب تطبيقه هو الخاص بـ Alpha.

# 4. دلوقتي نمشي مع Request حقيقي خطوة بخطوة

هنفترض إن عندنا Request بالشكل ده:

```
GET /api/products
X-Tenant-ID: alpha
Authorization: Bearer <token>
```

عايزين نوصل في النهاية إن Alpha تشوف منتجاتها فقط.

## Step 1 — Angular Client

الـ Client بيبعت Request ومعاه:

```
X-Tenant-ID: alpha
```

إيه اللي بيحصل؟

بيحدد الـ Tenant اللي الـ Request بيحاول يشتغل باسمه.

ليه؟

عشان الـ Backend يعرف سياق العميل المطلوب.

لو شيلنا الخطوة؟

مش هيكون عندنا Tenant ID في الـ Header، وبالتالي مش هنقدر نستخرج القيمة بالطريقة دي.

لكن مهم: الـ Header مش إثبات إن المستخدم مسموح له يدخل على Alpha. لازم نتحقق من صلاحياته وربطه بالـ Tenant من خلال Authentication.

## Step 2 — TenantFilter

الـ Request يدخل الـ Filter.

```
String tenantId =
    request.getHeader("X-Tenant-ID");

TenantContext.setCurrentTenant(tenantId);
```

بعد كده:

```
filterChain.doFilter(request, response);
```

دي معناها: كمل الـ Request لباقي الـ Filter Chain.

والتنظيف بيكون باستخدام `finally`:

```
try {
    filterChain.doFilter(request, response);
} finally {
    TenantContext.clear();
}
```

ليه finally؟

لأنها بتضمن تنفيذ التنظيف حتى لو حصل Exception أثناء تنفيذ الـ Request.

لو شيلنا الخطوة؟

مش هيبقى عندنا Tenant Context متاح للـ Service وAspect.

## Step 3 — ProductController

الـ Request يوصل للـ Controller.

```
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    @GetMapping
    public List<Product> getAll() {
        return productService.findAll();
    }
}
```

لاحظ حاجة مهمة جداً:

الـ Controller مش محتاج يعرف تفاصيل الـ Multi-Tenancy.

هو بيعمل شغله الطبيعي.

ليه؟

عشان إحنا عايزين الـ Tenant Isolation تكون مسؤولية مركزية مش مسؤولية كل Controller.

لو شيلنا الـ Controller؟

مش هيكون عندنا HTTP Endpoint يستقبل الـ Request ويربطه بالـ Service.

## Step 4 — ProductService + TenantAspect

الـ Controller ينادي:

```
productService.findAll();
```

هنا الـ AOP يتدخل قبل تنفيذ الـ Service Method.

الـ Aspect يعمل الآتي:

1. يقرأ Tenant ID من `TenantContext`.
    
2. يجيب Hibernate Session الحالية.
    
3. يفعّل `tenantFilter`.
    
4. يمرر قيمة `alpha`.
    

مثال توضيحي:

```
@Aspect
@Component
public class TenantAspect {

    @Before("execution(* com.app.service..*(..))")
    public void applyTenant() {

        String tenant = TenantContext.getCurrentTenant();

        Session session = entityManager.unwrap(Session.class);

        session.enableFilter("tenantFilter")
               .setParameter("tenantId", tenant);
    }
}
```

ليه محتاجين الخطوة دي؟

لأن Hibernate Filter مش بيقرأ ThreadLocal بنفسه.

إحنا بنعمل Bridge بين الاتنين:

`ThreadLocal → AOP → Hibernate Filter`

لو شيلناها؟

الـ Filter هيكون متعرّف في Entity، لكن مش هيكون مفعّل، وبالتالي ممكن الاستعلام يرجع بيانات كل الشركات.

## Step 5 — Hibernate ORM

الـ Service ينفذ:

```
productRepository.findAll();
```

Hibernate يستقبل طلب الاستعلام.

الـ Query المنطقية:

```
SELECT * FROM products;
```

لكن بسبب الـ Hibernate Filter المفعّل، Hibernate يضيف شرط Tenant:

```
SELECT *
FROM products
WHERE tenant_id = 'alpha';
```

ليه؟

عشان نمنع رجوع بيانات باقي الشركات.

لو شيلنا الخطوة؟

هنضطر نكتب شرط Tenant يدوياً في كل Query، وده بيزود احتمالية نسيان شرط العزل.

## Step 6 — PostgreSQL

الـ SQL يوصل للـ Database.

عندنا:

|id|tenant_id|product_name|
|---|---|---|
|1|alpha|Keyboard|
|2|alpha|Mouse|
|3|beta|Laptop|
|4|gamma|Monitor|

الـ Database تنفذ:

```
SELECT *
FROM products
WHERE tenant_id = 'alpha';
```

النتيجة:

|id|tenant_id|product_name|
|---|---|---|
|1|alpha|Keyboard|
|2|alpha|Mouse|

وبالتالي Alpha مش هتشوف بيانات Beta أو Gamma.

# 5. الدياجرام الكامل في Obsidian

ده أهم Diagram في الشرح. انسخه في Obsidian داخل Code Block من نوع Mermaid.

PostgreSQLHibernateTenantAspectProductServiceProductControllerTenantContextTenantFilterAngular ClientPostgreSQLHibernateTenantAspectProductServiceProductControllerTenantContextTenantFilterAngular Client#chatgpt-mermaid-_r_rl_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_rl_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_rl_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_rl_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-_r_rl_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rl_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_rl_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_rl_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_rl_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_rl_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_rl_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_rl_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-_r_rl_ p{margin:0;}#chatgpt-mermaid-_r_rl_ .actor{stroke:rgb(31, 78, 148);fill:rgb(9, 23, 44);stroke-width:1;}#chatgpt-mermaid-_r_rl_ rect.actor.outer-path[data-look="neo"]{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ rect.note[data-look="neo"]{stroke:rgb(58, 132, 63);fill:rgb(48, 48, 48);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ text.actor>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-_r_rl_ .actor-line{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ .innerArc{stroke-width:1.5;stroke-dasharray:none;}#chatgpt-mermaid-_r_rl_ .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ [id$="-arrowhead"] path{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ .sequenceNumber{fill:#505050;}#chatgpt-mermaid-_r_rl_ [id$="-sequencenumber"]{fill:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ [id$="-crosshead"] path{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rl_ .messageText{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-_r_rl_ .labelBox{stroke:rgba(255, 255, 255, 0.15);fill:rgb(0, 0, 0);filter:none;}#chatgpt-mermaid-_r_rl_ .labelText,#chatgpt-mermaid-_r_rl_ .labelText>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-_r_rl_ .loopText,#chatgpt-mermaid-_r_rl_ .loopText>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-_r_rl_ .sectionTitle,#chatgpt-mermaid-_r_rl_ .sectionTitle>tspan{fill:rgb(237, 237, 237);stroke:none;}#chatgpt-mermaid-_r_rl_ .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:rgba(255, 255, 255, 0.15);fill:rgba(255, 255, 255, 0.15);}#chatgpt-mermaid-_r_rl_ .note{stroke:rgb(58, 132, 63);fill:rgb(48, 48, 48);}#chatgpt-mermaid-_r_rl_ .noteText,#chatgpt-mermaid-_r_rl_ .noteText>tspan{fill:rgb(237, 237, 237);stroke:none;font-weight:normal;}#chatgpt-mermaid-_r_rl_ .activation0{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-_r_rl_ .activation1{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-_r_rl_ .activation2{fill:rgb(48, 48, 48);stroke:hsl(0, 0%, 8.8235294118%);}#chatgpt-mermaid-_r_rl_ .actorPopupMenu{position:absolute;}#chatgpt-mermaid-_r_rl_ .actorPopupMenuPanel{position:absolute;fill:rgb(9, 23, 44);box-shadow:0px 8px 16px 0px rgba(0,0,0,0.2);filter:drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));}#chatgpt-mermaid-_r_rl_ .actor-man circle,#chatgpt-mermaid-_r_rl_ line{fill:rgb(9, 23, 44);stroke-width:2px;}#chatgpt-mermaid-_r_rl_ g rect.rect{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));stroke:rgb(31, 78, 148);}#chatgpt-mermaid-_r_rl_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_rl_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_rl_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_rl_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_rl_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_rl_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_rl_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_rl_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_rl_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rl_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}AOP intercepts Service methodfinally: clear TenantContextGET /api/productsX-Tenant-ID: alpha1Extract alpha2setCurrentTenant(alpha)3Continue Filter Chain4findAll()5getCurrentTenant()6alpha7Enable tenantFilter(alpha)8productRepository.findAll()9Add tenant condition10SELECT * FROM productsWHERE tenant_id = 'alpha'11Alpha products12Products13Products14HTTP 200 Response15

## 5.1 ThreadLocal Lifecycle

الدياجرام ده مهم جداً لفهم الـ ThreadLocal:

#chatgpt-mermaid-_r_rs_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_rs_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_rs_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_rs_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-_r_rs_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_rs_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_rs_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_rs_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_rs_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_rs_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rs_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rs_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-_r_rs_ p{margin:0;}#chatgpt-mermaid-_r_rs_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .cluster-label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .cluster-label span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_rs_ .label text,#chatgpt-mermaid-_r_rs_ span{fill:rgb(237, 237, 237);color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .node rect,#chatgpt-mermaid-_r_rs_ .node circle,#chatgpt-mermaid-_r_rs_ .node ellipse,#chatgpt-mermaid-_r_rs_ .node polygon,#chatgpt-mermaid-_r_rs_ .node path{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .rough-node .label text,#chatgpt-mermaid-_r_rs_ .node .label text,#chatgpt-mermaid-_r_rs_ .image-shape .label,#chatgpt-mermaid-_r_rs_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_rs_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .rough-node .label,#chatgpt-mermaid-_r_rs_ .node .label,#chatgpt-mermaid-_r_rs_ .image-shape .label,#chatgpt-mermaid-_r_rs_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_rs_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_rs_ .root .anchor path{fill:rgb(175, 175, 175)!important;stroke-width:0;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rs_ .arrowheadPath{fill:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rs_ .edgePath .path{stroke:rgb(175, 175, 175);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .flowchart-link{stroke:rgb(175, 175, 175);fill:none;}#chatgpt-mermaid-_r_rs_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_rs_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-_r_rs_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_rs_ .labelBkg{background-color:rgba(0, 0, 0, 0.5);}#chatgpt-mermaid-_r_rs_ .cluster rect{fill:rgb(48, 48, 48);stroke:rgba(255, 255, 255, 0.15);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .cluster text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ .cluster span{color:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(48, 48, 48);border:1px solid rgba(255, 255, 255, 0.15);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_rs_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-_r_rs_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_rs_ .icon-shape,#chatgpt-mermaid-_r_rs_ .image-shape{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-_r_rs_ .icon-shape p,#chatgpt-mermaid-_r_rs_ .image-shape p{background-color:rgb(0, 0, 0);padding:2px;}#chatgpt-mermaid-_r_rs_ .icon-shape .label rect,#chatgpt-mermaid-_r_rs_ .image-shape .label rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-_r_rs_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_rs_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_rs_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_rs_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_rs_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_rs_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rs_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_rs_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_rs_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rs_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_rs_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_rs_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rs_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_rs_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_rs_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_rs_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(153, 206, 255);}#chatgpt-mermaid-_r_rs_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_rs_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_rs_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .node rect,#chatgpt-mermaid-_r_rs_ .node circle,#chatgpt-mermaid-_r_rs_ .node ellipse,#chatgpt-mermaid-_r_rs_ .node polygon,#chatgpt-mermaid-_r_rs_ .node path{fill:rgb(0, 40, 77);stroke:rgba(255, 255, 255, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_rs_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_rs_ .mermaid-decision .label-container{fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-dasharray:2,2;}#chatgpt-mermaid-_r_rs_ .edgePaths .flowchart-link{stroke:rgb(175, 175, 175);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_rs_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-_r_rs_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Thread receives RequestTenantFilter reads HeaderTenantContext.set(alpha)Controller / Service / AspectHibernate QueryRequest completesfinally:TenantContext.clear()Thread returns to ThreadPoolThread reused for anotherRequest

لاحظ إن الـ Thread مش شرط يتدمر بعد الـ Request.

هو بيرجع للـ Thread Pool عشان يتستخدم تاني.

وده سبب أهمية `clear()`.

# 6. هتبني الـ Architecture دي إزاي في Spring Boot؟

خلينا نحول الكلام لخطة تنفيذ حقيقية.

## 6.1 Project Structure

أنا أفضل Feature-based Architecture، مع فصل الـ Shared Components.

```
src/main/java/com/app
│
├── shared
│   └── tenant
│       ├── TenantContext.java
│       ├── TenantFilter.java
│       ├── TenantResolver.java
│       └── TenantContextHolder.java
│
├── config
│   ├── HibernateConfig.java
│   └── TenantAspect.java
│
├── features
│   └── product
│       ├── api
│       │   └── ProductController.java
│       ├── application
│       │   └── ProductService.java
│       ├── domain
│       │   └── Product.java
│       └── infrastructure
│           └── ProductRepository.java
│
└── Application.java
```

`TenantResolver` هنا مسؤول عن تحديد الـ Tenant والتحقق من صلاحيته، مش مجرد قراءة الـ Header.

## 6.2 ترتيب التنفيذ

متبدأش بكل حاجة مرة واحدة.

امشي بالترتيب ده:

1. Database Design
    
    اعمل `tenants` table و`products` table، وضيف `tenant_id` للـ Products.
    
    اربط الـ Product بالـ Tenant.
    
2. TenantContext
    
    اعمل Class مسؤول عن تخزين وقراءة وتنظيف Tenant ID باستخدام ThreadLocal.
    
3. TenantFilter
    
    اقرأ `X-Tenant-ID`، تحقق من وجوده وصحته، خزنه في TenantContext، وكمل الـ Filter Chain مع `finally` للتنظيف.
    
4. Product Entity
    
    ضيف `@FilterDef` و`@Filter` وعمود `tenant_id`.
    
5. Hibernate Configuration
    
    تأكد من طريقة الوصول للـ Hibernate Session والـ Transaction Management.
    
6. TenantAspect
    
    اعمل Aspect يفعّل Hibernate Filter قبل عمليات الـ Database.
    
7. ProductService + Repository
    
    نفذ CRUD واختبر الـ Isolation.
    
8. Security
    
    اربط الـ Authenticated User بالـ Tenant المسموح له باستخدامه.
    
9. Testing
    
    اختبر إن Alpha مش قادرة تقرأ أو تعدّل بيانات Beta.
    

## 6.3 تجربة باستخدام Postman أو curl

نفترض إن قاعدة البيانات فيها المنتجات الأربعة اللي اتكلمنا عنها.

أول Request:

```
curl -X GET http://localhost:8080/api/products \
-H "X-Tenant-ID: alpha" \
-H "Authorization: Bearer YOUR_TOKEN"
```

المفروض يرجع:

```
[
  {
    "id": 1,
    "tenantId": "alpha",
    "productName": "Keyboard"
  },
  {
    "id": 2,
    "tenantId": "alpha",
    "productName": "Mouse"
  }
]
```

ثاني Request:

```
curl -X GET http://localhost:8080/api/products \
-H "X-Tenant-ID: beta" \
-H "Authorization: Bearer YOUR_TOKEN"
```

المفروض يرجع:

```
[
  {
    "id": 3,
    "tenantId": "beta",
    "productName": "Laptop"
  }
]
```

Expected behavior

Alpha وBeta بيستخدموا نفس Endpoint، لكن كل واحد بيشوف بياناته فقط.

# 7. أخطاء Production لازم تاخد بالك منها

دي أهم جزئية عايزك تركز فيها؛ لأن فهم الـ Happy Path لوحده مش كفاية.

## 7.1 نسيان TenantContext.clear()

Critical

تخيّل:

1. Thread-42 استقبل Request من Alpha.
    
2. خزّن `alpha`.
    
3. خلص الـ Request.
    
4. نسينا `clear()`.
    
5. نفس Thread استقبل Request من Beta.
    
6. حصل خطأ في تحديد Tenant، والـ Context القديم لسه موجود.
    

النتيجة ممكن تكون إن Request جديد يشتغل بسياق قديم.

الحل:

```
try {
    filterChain.doFilter(request, response);
} finally {
    TenantContext.clear();
}
```

## 7.2 مشكلة findById()

دي من أهم النقاط اللي لازم تعرفها.

ممكن تفتكر إن:

```
productRepository.findById(100L);
```

هيطبق Hibernate Filter تلقائياً.

لكن خد بالك: Hibernate `@Filter` مش بيطبق تلقائياً على كل طرق تحميل الـ Entity.

في إصدارات Hibernate الحديثة فيه خاصية:

```
@Filter(
    name = "tenantFilter",
    condition = "tenant_id = :tenantId",
    applyToLoadByKey = true
)
```

الخاصية دي بتسمح بتطبيق الـ Filter على عمليات تحميل الـ Entity باستخدام المفتاح، زي `find()`، في الإصدارات اللي بتدعمها.

لو إصدار Hibernate عندك مش بيدعمها، أو عندك طريقة تحميل مش مغطاة بالـ Filter، لازم تعمل حماية إضافية.

مثلاً:

```
Optional<Product> product =
    productRepository.findById(id);

return product
    .filter(p -> p.getTenantId().equals(currentTenant));
```

ده مثال توضيحي، والأفضل في Production إنك تمنع الوصول من مستوى الاستعلام نفسه، مش تعتمد على فحص بعد تحميل البيانات فقط.

## 7.3 Async Threads

دي نقطة ناس كتير بتغفل عنها.

`ThreadLocal` مرتبط بالـ Thread نفسه.

يعني لو عملت:

```
TenantContext.setCurrentTenant("alpha");

executor.execute(() -> {
    String tenant = TenantContext.getCurrentTenant();
});
```

الـ Async Task ممكن يشتغل على Thread مختلف، وبالتالي مش هيلاقي `alpha`.

الحل إنك تنقل Tenant Context بشكل صريح للـ Async Task، وتعمل Cleanup بعد التنفيذ.

ومتفترضش إن `ThreadLocal` بينتقل تلقائياً لأي Thread جديد.

## 7.4 الاعتماد على Header فقط

Security risk

تخيّل مستخدم تابع لـ Alpha بعت:

```
X-Tenant-ID: beta
```

لو الـ Backend صدّق الـ Header من غير ما يتحقق من صلاحيات المستخدم، المستخدم ممكن يشوف بيانات Beta.

الحل:

```
Authenticated User
       |
       v
Check Tenant Membership
       |
       v
Is User allowed to access Alpha?
       |
    Yes / No
       |
       v
Set TenantContext
```

يعني الـ Header بيقولنا المستخدم طالب يشتغل على مين، لكن الـ Authentication وAuthorization هما اللي يحددوا هل مسموح له ولا لأ.

## 7.5 مشكلة AOP وHibernate Session

فيه تفصيلة مهمة في الصورة:

الـ Aspect لازم يفعّل Hibernate Filter على الـ Session اللي هتنفذ عليها الـ Query فعلاً.

كمان لازم تراجع ترتيب الـ AOP والـ Transaction.

مثلاً لو الـ Transaction بدأت على Session، والـ Aspect فعّل Filter على Session مختلفة، مش هتحصل النتيجة المطلوبة.

عشان كده لازم تختبر إن:

- الـ Filter اتفعّل على نفس الـ Session.
    
- الـ Filter اتفعّل قبل تنفيذ Query.
    
- الـ Filter فضل مفعّل طول العمليات المطلوبة.
    
- الـ Session اتقفلت أو انتهت الـ Transaction بشكل سليم.
    

ملحوظة إضافية: Hibernate Filters مش بديل كامل عن Database-level security. في الأنظمة الحساسة ممكن تستخدم PostgreSQL Row-Level Security كطبقة دفاع إضافية.

# 8. خمس مفاهيم لازم تكون فاهمها قبل باقي الكورس

|المفهوم|لازم تفهم إيه؟|
|---|---|
|Servlet Filter|اعتراض الـ Request قبل Controller|
|ThreadLocal|تخزين قيمة منفصلة لكل Thread|
|AOP|تنفيذ Logic إضافي قبل أو بعد Methods|
|Hibernate Filter|إضافة شروط تلقائية للـ SQL|
|Tenant Isolation|منع أي Tenant من الوصول لبيانات غيره|

لو فهمت العلاقة بينهم، هتقدر تفهم معظم الـ Request Flow اللي في الكورس.

## 8.1 العلاقة بينهم في سطر واحد

```
HTTP Request
    |
    v
Servlet Filter
    |
    v
ThreadLocal
    |
    v
AOP Aspect
    |
    v
Hibernate Filter
    |
    v
SQL WHERE tenant_id = ?
    |
    v
PostgreSQL
```

# 9. ملاحظات مهمة على الـ Diagram بتاع الكورس

الصورة ممتازة لتوضيح الـ Happy Path، لكن فيه شوية تفاصيل لازم متفترضش إنها مضمونة بمجرد تنفيذ الرسم:

1. `X-Tenant-ID` مش مصدر موثوق للصلاحيات؛ لازم نربطه بالـ Authenticated User.
    
2. `@Filter` مش بيحمي كل عمليات Hibernate بنفس الطريقة، خصوصاً تحميل Entity بالـ ID.
    
3. `ThreadLocal` مش بيعمل Context Propagation للـ Async Threads.
    
4. الـ AOP Pointcut لازم يستهدف Methods مناسبة، ومش كل Method داخل نفس الـ Bean هتعدي بالضرورة على Spring Proxy.
    
5. لازم تراجع ترتيب الـ Transaction والـ Aspect.
    
6. لازم تتأكد إن كل الـ Entities والـ Relationships اللي محتاجة عزل عليها الـ Tenant Filtering المناسب.
    
7. عمليات الـ Insert محتاجة حماية برضه؛ لازم نتأكد إن الـ `tenant_id` بيتحدد من الـ Trusted Tenant Context، مش من بيانات المستخدم المرسلة.
    

وفي نقطة أخيرة: الرسم بيستخدم Hibernate Filter كوسيلة لعزل القراءة، لكن ده مش معناه إنك تقدر تعتمد عليه وحده لضمان العزل الكامل في كل CRUD operations.

# 10. ملخص الفهم النهائي

خلينا نرجع للصورة الأصلية.

ليه كل Component موجود؟

|Component|مسؤول عن إيه؟|
|---|---|
|Angular Client|إرسال Tenant ID|
|TenantFilter|استخراج Tenant ID|
|TenantContext|تخزين Tenant ID في ThreadLocal|
|Controller|استقبال الـ Request|
|Service|تنفيذ Business Logic|
|AOP Aspect|تفعيل Hibernate Filter|
|Hibernate Filter|تحديد Tenant condition|
|PostgreSQL|إرجاع البيانات المطابقة|

أهم فكرة في المشروع كله:

إحنا مش بنعمل Application منفصل لكل شركة.

إحنا بنعمل Application واحدة، لكن بنحدد سياق كل Request، وبنستخدم السياق ده عشان نضمن إن كل شركة تتعامل مع بياناتها فقط.

# 11. Obsidian — صفحة المراجعة النهائية

دي النسخة المختصرة اللي ترجع لها بعد ما تخلص مشاهدة الجزء ده من الكورس.

# Multi-Tenancy — HTTP Request Flow

## 1. What is Multi-Tenancy?

Multi-Tenancy means multiple customers (Tenants) share the same application while keeping their data isolated.

### Strategies

1. Database per Tenant
    
2. Schema per Tenant
    
3. Shared Database + Shared Schema (tenant_id)
    

**Our approach:** Shared Database + Shared Schema.

## 2. Main Request Flow

```
HTTP Request
     |
     v
TenantFilter
     |
     v
TenantContext (ThreadLocal)
     |
     v
ProductController
     |
     v
ProductService
     |
     v
TenantAspect (AOP)
     |
     v
Hibernate Filter
     |
     v
SQL WHERE tenant_id = ?
     |
     v
PostgreSQL
```

## 3. Main Components

### Servlet Filter

- Intercepts HTTP Requests before Controllers.
    
- Extracts X-Tenant-ID.
    
- Sets TenantContext.
    
- Continues the Filter Chain.
    
- Clears Context in finally.
    

### ThreadLocal

- Stores a value independently for each Thread.
    
- `set()` stores Tenant ID.
    
- `get()` retrieves Tenant ID.
    
- `remove()` clears Tenant ID.
    

### AOP (Aspect Oriented Programming)

- Adds additional behavior around method execution.
    
- `@Before` executes advice before the target method.
    
- Used to activate Hibernate Filter before database queries.
    

### Hibernate @Filter

- Defines a condition applied to Hibernate queries.
    
- `@FilterDef` defines the filter and parameters.
    
- `@Filter` defines the SQL condition.
    
- `enableFilter()` activates the filter on the current Session.
    
- Defining a Filter does NOT automatically activate it.
    

## 4. Example

Request:

```
GET /api/products
X-Tenant-ID: alpha
```

Hibernate executes:

```
SELECT *
FROM products
WHERE tenant_id = 'alpha';
```

Only Alpha products are returned.

## 5. Critical Production Pitfalls

- Always clear ThreadLocal using `finally`.
    
- Hibernate Filter doesn't automatically cover every entity loading operation, including findById by default.
    
- ThreadLocal does not automatically propagate to Async Threads.
    
- Never trust X-Tenant-ID without authorization.
    
- Ensure AOP and Hibernate use the same Session.
    
- Protect INSERT, UPDATE and DELETE operations too.
    
- Consider PostgreSQL Row-Level Security as additional protection.
    

## 6. Golden Rules

1. Filter identifies the requested Tenant.
    
2. ThreadLocal stores the current Tenant Context.
    
3. AOP connects TenantContext with Hibernate.
    
4. Hibernate Filter applies Tenant conditions to supported queries.
    
5. Database returns only matching records.
    

**Remember: Tenant Isolation must be enforced across the entire data-access layer, not only Controllers.**

## إيه اللي أريدك تخرج بيه من الشرح؟

مش مطلوب تحفظ الكود.

المطلوب لما تشوف السطر ده:

```
TenantContext.setCurrentTenant(tenantId);
```

تعرف إنه بداية رحلة الـ Tenant Context.

ولما تشوف:

```
session.enableFilter("tenantFilter");
```

تعرف إننا بنوصل الـ Tenant Context بالـ Hibernate Query Execution.

ولما تشوف:

```
WHERE tenant_id = 'alpha'
```

تعرف إن دي النتيجة النهائية لكل الخطوات السابقة.

وبالنسبة لباقي الكورس: لما توصل لأي جزء جديد، ابعتلي اسمه أو صورة منه، وهربطه بالـ Architecture دي وأوضحلك المفاهيم الجديدة اللي محتاج تفهمها.

مش هعملك Quiz دلوقتي احتراماً لطلبك إننا نشرح الأول من غير أسئلة.