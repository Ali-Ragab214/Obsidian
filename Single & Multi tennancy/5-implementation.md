بص يا علي، التلات Classes دول بيعملوا مع بعض جزء مهم جدًا من الـ Multi-Tenancy باستخدام Shared Database + Shared Schema.

الفكرة الأساسية إننا عايزين كل Request يدخل السيستم نعرف هو تابع لأنهي Tenant، ونخلي أي Database Query تتنفذ تشوف بيانات الـ Tenant ده بس.

خلينا نمسكهم بالترتيب الصح، وهنمشي كأننا بنبني الـ Feature من الصفر.

# أولاً: ترتيب إنشاء الـ Classes

1. TenantContext
    
    First
    
    المسؤول عن تخزين الـ Current Tenant مؤقتًا أثناء تنفيذ الـ Request باستخدام `ThreadLocal`.
    
2. TenantFilter
    
    المسؤول عن قراءة الـ Tenant ID من الـ HTTP Request وتخزينه في `TenantContext`.
    
3. TenantHibernateFilter
    
    المسؤول عن تفعيل Hibernate Filter بحيث أي Query على الـ Entities ترجع بيانات الـ Tenant الحالي فقط.
    

خلينا نشرح كل واحد بالتفصيل.

# 1. TenantContext

## الكود بالكامل

```

package ali.ragab.saas_multi_tennant_app.config;

public class TenantContext {

    private static final ThreadLocal<String> CURRENT_TENANT = new ThreadLocal<>();

    public static void setCurrentTenant(final String tenant) {
        CURRENT_TENANT.set(tenant);
    }

    public static String getCurrentTenant() {
        return CURRENT_TENANT.get();
    }

    public static void clearCurrentTenant() {
        CURRENT_TENANT.remove();
    }
}
```

## الكلاس ده مسؤول عن إيه؟

ببساطة هو عبارة عن مكان بنخزن فيه الـ `Tenant ID` الخاص بالـ Request الحالي.

لكن السؤال: ليه استخدمنا `ThreadLocal` بدل ما نعمل متغير عادي؟

### نفهم ThreadLocal الأول

تخيل إن عندك Application بيخدم شركتين:

- Company A → tenant-a
    
- Company B → tenant-b
    

والاتنين بيبعتوا Requests في نفس الوقت.

الـ Spring Boot بيقدر يعالج كل Request في Thread مختلف.

لو استخدمنا متغير `static` عادي، ممكن Thread بتاع Company A يكتب `tenant-a`، وبعده Thread بتاع Company B يكتب `tenant-b`، وبالتالي يحصل تداخل في البيانات.

لكن `ThreadLocal` بيخلي كل Thread عنده نسخة مستقلة من المتغير.
![[Pasted image 20261110212449.png]]



يعني:

|Thread|CURRENT_TENANT|
|---|---|
|Thread 1|tenant-a|
|Thread 2|tenant-b|
|Thread 3|tenant-c|

كل Thread بيشوف القيمة الخاصة بيه فقط.

## نشرح الـ Methods

1. setCurrentTenant()

```
public static void setCurrentTenant(final String tenant) {
    CURRENT_TENANT.set(tenant);
}
```

بتحط الـ Tenant ID داخل الـ ThreadLocal الخاص بالـ Thread الحالي.

مثلاً:

```
TenantContext.setCurrentTenant("tenant-a");
```

دلوقتي الـ Thread الحالي بقى عارف إنه بيخدم tenant-a.

2. getCurrentTenant()

```
public static String getCurrentTenant() {
    return CURRENT_TENANT.get();
}
```

بترجع الـ Tenant ID المخزن حاليًا.

مثلاً:

```
String tenantId = TenantContext.getCurrentTenant();
```

النتيجة:

```
tenant-a
```

3. clearCurrentTenant()

```
public static void clearCurrentTenant() {
    CURRENT_TENANT.remove();
}
```

دي مهمة جدًا.

ليه؟

لأن Spring بيستخدم Thread Pool، بمعنى إن نفس الـ Thread ممكن يخلص Request ويرجع يتستخدم في Request تاني.

لو معملناش `remove()`، ممكن الـ Request الجديد يلاقي قيمة الـ Tenant القديم.

وده ممكن يؤدي إلى Data Leakage بين الـ Tenants.

الخلاصة: `TenantContext` هو المكان اللي بنخزن ونقرأ ونمسح منه الـ Current Tenant الخاص بالـ Thread الحالي.

# 2. TenantFilter

ده المسؤول عن استقبال الـ Tenant ID من الـ HTTP Request.

## الكود بالكامل

```java

package ali.ragab.saas_multi_tennant_app.config;

import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Locale;

@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class TenantFilter implements Filter {

    private static final String TENANT_HEADER = "X-TenantID";

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain filterChain
    ) throws IOException, ServletException {

        final HttpServletRequest httpRequest =
                (HttpServletRequest) request;

        final HttpServletResponse httpResponse =
                (HttpServletResponse) response;

        final String tenantId = resolveHeader(httpRequest);

        if (tenantId == null || tenantId.isBlank()) {
            httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            httpResponse.getWriter().write(
                    "Missing or empty X-TenantID header"
            );
            return;
        }

        try {
            TenantContext.setCurrentTenant(tenantId);
            filterChain.doFilter(request, response);
        } finally {
            TenantContext.clearCurrentTenant();
        }
    }

    private String resolveHeader(final HttpServletRequest request) {
        final String tenantId = request.getHeader(TENANT_HEADER);

        if (tenantId != null && !tenantId.isBlank()) {
            return tenantId.toLowerCase(Locale.ROOT);
        }

        return null;
    }
}
```

## الكلاس ده بيعمل إيه؟

هو Servlet Filter، وده معناه إنه بيتنفذ في مرحلة مبكرة جدًا من دورة حياة الـ HTTP Request.

قبل ما الـ Request يوصل للـ Controller، بيعدي على الـ Filter.

@Component -> spring know that this class is a bean and will be stored i app context 

client ->filter chain -> dispatcher servlet ->controller 

خلينا نفهمه خطوة خطوة.

### 1. @Component

```
@Component
```

بنخلي Spring يكتشف الكلاس ويعمله Bean داخل الـ Application Context.

### 2. @Order

```
@Order(Ordered.HIGHEST_PRECEDENCE)
```

بنحدد إن الـ Filter ده يكون من أعلى الأولويات في ترتيب الـ Filters.

يعني نريده يشتغل بدري جدًا قبل باقي الـ Filters اللي ترتيبها يسمح بكده.

وده مهم لأننا عايزين الـ Tenant ID يبقى موجود قبل ما باقي أجزاء التطبيق تحتاجه.

### 3. TENANT_HEADER

```
private static final String TENANT_HEADER = "X-TenantID";
```

حددنا اسم الـ HTTP Header اللي العميل هيبعت فيه الـ Tenant ID.

مثلاً لو بنستخدم Postman:

```
GET /api/users

X-TenantID: tenant-a
```

الـ Header ده هو اللي هيحدد الـ Tenant المطلوب.

### 4. doFilter()

دي أهم Method في الكلاس كله.

```
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain filterChain
)
```

دي الـ Method اللي Servlet Container بيناديها مع كل Request بيمر على الـ Filter.

أول حاجة: تحويل الـ Request والـ Response

```
final HttpServletRequest httpRequest =
        (HttpServletRequest) request;

final HttpServletResponse httpResponse =
        (HttpServletResponse) response;
```

بنحولهم للـ HTTP-specific implementations علشان نقدر نستخدم Methods زي:

- `getHeader()`
    
- `setStatus()`
    
- `getWriter()`
    

ثاني حاجة: استخراج الـ Tenant ID

```
final String tenantId = resolveHeader(httpRequest);
```

بننادي الـ Helper Method اللي بتقرأ الـ Header.

ثالث حاجة: Validation

```
if (tenantId == null || tenantId.isBlank()) {
    httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
    httpResponse.getWriter().write(
            "Missing or empty X-TenantID header"
    );
    return;
}
```

لو الـ Client مبعتش الـ Tenant ID أو بعت قيمة فاضية:

- بنرجع HTTP 400 Bad Request.
    
- بنوقف تنفيذ الـ Request باستخدام `return`.
    

يعني مش هنسمح للـ Request يكمل للـ Controller.

رابع حاجة: تخزين الـ Tenant

```
try {
    TenantContext.setCurrentTenant(tenantId);
    filterChain.doFilter(request, response);
}
```

هنا بنعمل حاجتين:

1. بنخزن الـ Tenant ID في `TenantContext`.
    
2. بنسمح للـ Request يكمل رحلته لباقي الـ Filters والـ Controller والـ Service.
    

خد بالك من:

```
filterChain.doFilter(request, response);
```

دي معناها: كمل تنفيذ الـ Filter Chain.

مش معناها إنك بتنادي الـ Controller بنفسك.

خامس حاجة: تنظيف الـ Context

```
finally {
    TenantContext.clearCurrentTenant();
}
```

بمجرد ما تنفيذ الـ Request يخلص، سواء نجح أو حصل Exception، بنمسح الـ Tenant ID من الـ ThreadLocal.

وده بيضمن إن الـ Thread لما يتعاد استخدامه ميكونش شايل Tenant قديم.

### 5. resolveHeader()

```
private String resolveHeader(final HttpServletRequest request) {
    final String tenantId = request.getHeader(TENANT_HEADER);

    if (tenantId != null && !tenantId.isBlank()) {
        return tenantId.toLowerCase(Locale.ROOT);
    }

    return null;
}
```

دي Helper Method بتعمل الآتي:

1. تقرأ الـ Header.
    
2. تتأكد إنه مش null أو blank.
    
3. تحول القيمة إلى lowercase.
    
4. ترجع الـ Tenant ID.
    

استخدمنا `Locale.ROOT` علشان تحويل الأحرف يكون ثابتًا بغض النظر عن الـ Locale الخاص بالسيرفر.

الخلاصة: `TenantFilter` بيستخرج الـ Tenant ID من الـ HTTP Request ويحطه في `TenantContext`، وبعد انتهاء الـ Request بيمسحه.

# 3. TenantHibernateFilter

وصلنا بقى للجزء اللي بيربط الـ Tenant بالـ Database Queries.

## الكود بالكامل

```

package ali.ragab.saas_multi_tennant_app.config;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.hibernate.Session;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class TenantHibernateFilter {

    @PersistenceContext
    private EntityManager entityManager;

    @Before("execution(* ali.ragab.saas_multi_tennant_app.services.*.*(..))")
    public void activateFilter() {

        final String tenantId = TenantContext.getCurrentTenant();

        if (tenantId != null && !tenantId.isBlank()) {

            final Session session =
                    entityManager.unwrap(Session.class);

            // Activate the filter to enable multi-tenancy support
            session.enableFilter("tenantFilter")
                    .setParameter("tenantId", tenantId);
        }
    }
}
```

## الأول نفهم هو بيحل مشكلة إيه؟

لحد دلوقتي إحنا عملنا الآتي:

- استقبلنا Tenant ID من الـ Request.
    
- خزناه في ThreadLocal.
    

لكن هل كده الـ Database عرفت إننا عايزين بيانات tenant-a فقط؟

لأ!

إحنا لسه محتاجين نربط الـ Tenant ID بالـ Hibernate Queries.

وهنا بييجي دور الكلاس ده.

### 1. @Aspect

```
@Aspect
@Component
```

هنا بنستخدم Spring AOP.

الـ Aspect بيسمح لنا ننفذ Logic معين قبل أو بعد Methods معينة، من غير ما نضطر نكرر نفس الكود داخل كل Service.

مثلاً عندنا:

```
UserService
CourseService
SubscriptionService
```

بدل ما نكتب كود تفعيل الـ Hibernate Filter في كل Service، بنعمل Aspect واحد مسؤول عن الموضوع.

### 2. EntityManager

```
@PersistenceContext
private EntityManager entityManager;
```

الـ EntityManager هو الـ JPA interface المسؤول عن التعامل مع الـ Persistence Context وإدارة الـ Entities.

إحنا محتاجينه علشان نوصل للـ Hibernate Session.

### 3. @Before

```
@Before("execution(* ali.ragab.saas_multi_tennant_app.services.*.*(..))")
```

دي مهمة جدًا.

الـ Pointcut Expression بتحدد الـ Methods اللي الـ Aspect هيشتغل قبل تنفيذها.

نفكها:

|الجزء|معناه|
|---|---|
|`execution`|بنحدد Methods بناءً على توقيعها|
|`*`|أي Return Type|
|`ali.ragab...services`|الـ Package المستهدف|
|`*`|أي Class داخل الـ Package|
|`*`|أي Method داخل الكلاس|
|`(..)`|أي عدد من الـ Parameters وبأي Types|

يعني باختصار:

قبل تنفيذ أي Method داخل أي Class موجود مباشرة في services package، نفذ activateFilter().

ملحوظة: الـ Pointcut بالشكل ده مش بيشمل Sub-packages تلقائيًا.

### 4. activateFilter()

دي الـ Advice Method اللي هتتنفذ قبل الـ Service Method.

أول حاجة:

```
final String tenantId = TenantContext.getCurrentTenant();
```

بنجيب الـ Tenant ID اللي اتخزن في الخطوة السابقة بواسطة `TenantFilter`.

مثلاً:

```
tenant-a
```

ثاني حاجة:

```
if (tenantId != null && !tenantId.isBlank())
```

بنتأكد إن فيه Tenant ID صالح من ناحية عدم كونه null أو blank.

ثالث حاجة:

```
final Session session =
        entityManager.unwrap(Session.class);
```

هنا بنجيب الـ Hibernate Session من الـ EntityManager.

ليه؟

لأن `enableFilter()` دي Hibernate-specific functionality، مش موجودة في JPA EntityManager API.

رابع حاجة، ودي أهم سطرين:

```
session.enableFilter("tenantFilter")
       .setParameter("tenantId", tenantId);
```

هنا بنعمل حاجتين:

1. بنفعل Hibernate Filter اسمه `tenantFilter`.
    
2. بنمرر له قيمة الـ Tenant ID الحالي.
    

مثلاً:

```
tenantId = tenant-a
```

وبالتالي Hibernate هيطبق الـ Filter المحدد على الـ Entity المرتبطة بيه.

خلي بالك: الكلاس ده لوحده مش كفاية. لازم تكون عرّفت الـ Filter على الـ Entity نفسها باستخدام `@FilterDef` و`@Filter`.

مثال:

```
@Entity
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
public class User extends BaseEntity {

    // Fields
}
```

هنا Hibernate بقى عارف إن:

- اسم الـ Filter هو `tenantFilter`.
    
- عنده Parameter اسمه `tenantId`.
    
- الشرط اللي هيطبقه هو إن عمود `tenant_id` يساوي قيمة الـ Tenant الحالي.
    

مثلاً لو عندنا البيانات دي:

|id|name|tenant_id|
|---|---|---|
|1|Ahmed|tenant-a|
|2|Mohamed|tenant-b|
|3|Ali|tenant-a|

والـ Current Tenant هو `tenant-a`، فالـ Query هترجع:

|id|name|tenant_id|
|---|---|---|
|1|Ahmed|tenant-a|
|3|Ali|tenant-a|

يعني Hibernate بيضيف شرط الـ Tenant للـ Queries التي يطبق عليها الـ Filter.

# رابعاً: التلاتة بيشتغلوا مع بعض إزاي؟

خلينا نمشي في سيناريو حقيقي.

افترض إن عندنا Request جاي من Company A.

## Request Lifecycle

1. HTTP Request

X-TenantID: tenant-a

2. TenantFilter

يقرأ Header ويتأكد إنه موجود.

Tenant ID = tenant-a

3. TenantContext

يخزن tenant-a في ThreadLocal.

4. Controller → Service

الـ Request يكمل للـ Business Logic.

5. TenantHibernateFilter

Aspect يقرأ Tenant ID من TenantContext.

يفعل Hibernate Filter على الـ Session.

6. Database Query

```
SELECT *
FROM users
WHERE tenant_id = 'tenant-a';
```

Tenant A data only

7. TenantFilter - finally

يمسح الـ Tenant ID من ThreadLocal.

# خامساً: حاجات مهمة جدًا لازم تاخد بالك منها

فيه شوية نقاط Production-level مهمة في التصميم ده:

1. الـ Header مش وسيلة Authentication

مينفعش تثق في أي `X-TenantID` جاي من الـ Client.

ممكن أي مستخدم يبعت:

```
X-TenantID: tenant-b
```

حتى لو هو تابع لـ tenant-a.

لازم تتحقق إن الـ Tenant موجود وإن المستخدم authenticated ومصرح له بالوصول للـ Tenant ده، ويفضل تربط الـ Tenant بالـ authenticated principal أو تتحقق منه من خلال الـ Security Layer.

2. ترتيب الـ AOP والـ Transaction مهم

الـ Aspect لازم يفعل الـ Hibernate Filter على نفس الـ Session اللي هتنفذ عليها الـ Queries.

لو عندك `@Transactional`، لازم تراجع ترتيب الـ AOP advice والـ Transaction interceptor علشان تضمن إن الـ Filter اتفعل على الـ Session المستخدمة في الـ Transaction قبل أي Query.

3. Hibernate Filter مش حماية شاملة لوحده

الـ Filter مش بيتطبق تلقائيًا على كل أنواع الوصول للبيانات؛ لازم تاخد بالك من native queries وبعض طرق تحميل الـ Entities والـ associations والـ bulk operations.

كمان لازم تراعي تفعيل الـ Filter عند إنشاء البيانات، مش بس قراءتها، علشان متسمحش بإنشاء Entity تحت Tenant غلط.

4. ThreadLocal مش مناسب تلقائيًا للـ Async

لو استخدمت `@Async` أو Background Jobs، الـ Thread الجديد مش هيورث الـ Tenant ID تلقائيًا. لازم تنقل الـ Tenant Context بطريقة آمنة ومقصودة.

# الخلاصة اللي تحفظها للإنترفيو

|Class|مسؤوليته|
|---|---|
|`TenantContext`|Stores the current Tenant ID using ThreadLocal.|
|`TenantFilter`|Extracts Tenant ID from HTTP Header and sets the TenantContext.|
|`TenantHibernateFilter`|Uses Spring AOP to enable Hibernate Filter before executing Service methods.|

احفظ التسلسل ده:

`HTTP Header → TenantFilter → TenantContext → AOP → Hibernate Session → Hibernate Filter → Database`

## اختبر فهمك بقى

عايز أتأكد إنك فاهم العلاقة بين التلاتة، مش مجرد حافظ الكود.

Mini Quiz

5 Questions

1. ليه استخدمنا ThreadLocal في TenantContext بدل static String عادي؟

علشان كل Thread يكون له Tenant ID مستقل

علشان نخزن Tenant ID في Database

علشان نمنع إنشاء Threads جديدة

2. إيه اللي بيحصل لو نسينا clearCurrentTenant() في finally؟

Hibernate هيوقف الـ Filter

ممكن Thread يتعاد استخدامه ويحتفظ بـ Tenant قديم

الـ Database هتحذف بيانات الـ Tenant

3. ليه TenantHibernateFilter بيستخدم EntityManager.unwrap(Session.class)؟

علشان Hibernate Session توفر enableFilter()

علشان يحول Entity إلى DTO

علشان يفتح Connection جديدة

4. إمتى TenantFilter بيخزن الـ Tenant ID؟

بعد تنفيذ الـ Database Query

قبل filterChain.doFilter()

بعد ما الـ Response يرجع للـ Client

5. هل مجرد تفعيل TenantHibernateFilter كفاية لتطبيق شرط tenant_id على كل Entities؟

أيوه، تلقائيًا على كل Entities

لأ، لازم تعريف Hibernate Filter على الـ Entities المطلوبة

أيوه، طالما عندنا @Entity

Submit Answers