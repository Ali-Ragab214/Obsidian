بص، خلينا نفهم الـ Multi-Tenancy من الصفر، بس مش مجرد تعريفات. عايزك في الآخر تكون قادر تبص على أي SaaS System وتفهم هو Single-Tenant ولا Multi-Tenant، وتعرف تختار Architecture مناسبة لمشروعك، وتفهم إيه اللي بيتغير في الـ Code وإيه اللي مش بيتغير.

وأهم نقطة عايزك تركز فيها:

Multi-Tenancy مش معناها إننا بنعمل نسخة من الكود لكل Customer، ولا إننا بنعدل في الكود كل ما عميل جديد يشترك.

بالعكس، الفكرة إننا بنشغل نفس الـ Application لخدمة Customers كتير، مع الحفاظ على إن بيانات كل واحد منهم معزولة عن التاني.

# 1. الأول نفهم يعني إيه Tenant؟

كلمة Tenant معناها عميل أو مؤسسة بتستخدم الـ SaaS Application.

مثلاً عندك SaaS Platform لإدارة الـ Gyms والـ Fitness Coaches.

[FitSynQ - Complete Gym Management Software | Fitness Management Platform](https://images.openai.com/static-rsc-4/PcNhRvoOEXPlISxfTpsIEIp9CygYNNOHX45yOVoQzwvY2lPPO9OVZrubL9eYgD2OpWwprcvZRkoK-o3g-v6xHiQbGYPu0EtpUxZiMOO9bJ9QP_lo5WxqjF66jY0Vk0lYlA07KpaSowlg-5DWTSE6jzhHMjDUDZdhLNpkh73yL8w?purpose=inline)

[fitsynq.app](https://fitsynq.app/?utm_source=chatgpt.com)

تخيل عندك 3 عملاء:

Gym A

Tenant 1

Gym B

Tenant 2

Gym C

Tenant 3

كل Gym عنده:

- Coaches.
    
- Trainees.
    
- Workout Programs.
    
- Nutrition Plans.
    
- Subscriptions.
    
- Payments.
    

الـ Gym A مينفعش يشوف بيانات Gym B، رغم إن الاتنين بيستخدموا نفس الـ Application.

إذن:

- Tenant = Organization / Customer.
    
- User = شخص بيستخدم النظام.
    
- كل Tenant ممكن يكون عنده Users كتير.
    

ودي نقطة مهمة جداً: Tenant مش بالضرورة يكون User.

# 2. Single-Tenant Architecture

خلينا نفترض إن عندنا 3 Gyms.

في الـ Single-Tenant، كل Gym عنده Application Instance مستقلة، وغالباً Database مستقلة.
![[Pasted image 20261104135726.png]]

[Fundamentals of SaaS Architecture | by Amit Kumar Batra | Medium](https://images.openai.com/static-rsc-4/GARZnTekFz-FKMvv84ya_L5EV3UK9nrvUd2ElgHauU7CyEfX2wSx2NJYdHMXy7j3kktwV0dihkAv5Ok2FOXqTW1U95-Bdc4kmFS1XRPXxTyzcyNvipaBZHj0jHNQPCZB1M6W0nwunlvdGW90_wgRcMZPibXUFNolZrLx7tytfQs?purpose=inline)

[medium.com](https://medium.com/%40amit2067/fundamentals-of-saas-architecture-c824f82b888b?utm_source=chatgpt.com)

يعني:

|Customer|Application|Database|
|---|---|---|
|Gym A|Instance A|DB A|
|Gym B|Instance B|DB B|
|Gym C|Instance C|DB C|

كل واحد عنده نسخته المستقلة.

طيب إيه اللي بيحصل لما Gym A يطلب Feature جديدة؟

مثلاً عايز AI Workout Generator.

ممكن تعمل:

1. تعدل الكود.
    
2. تعمل Deploy لنسخة Gym A.
    
3. تعمل Deploy منفصل لباقي العملاء لو التعديل مشترك ومطلوب عندهم.
    

ولو كل عميل عنده نسخة مختلفة من الكود، إدارة الـ Versions هتبقى أصعب بكتير.

المميزات:

- عزل قوي بين العملاء.
    
- Customization أكبر لكل Customer.
    
- تقدر تخصص Resources لكل عميل.
    
- تأثير مشكلة في Instance معينة ممكن يفضل محدود فيها.
    

العيوب:

- تكلفة Infrastructure أعلى.
    
- Deployment وMaintenance أصعب.
    
- محتاج تدير Instances وDatabases كتير.
    
- Scaling ممكن يكون مكلف.
    

# 3. Multi-Tenant Architecture

هنا بقى الفكرة مختلفة تماماً.

بدل ما تعمل Application مستقلة لكل Customer، بتعمل:

- One Application.
    
- One Codebase.
    
- Multiple Tenants.
    
- Shared Infrastructure (في التصميم المشترك).

![[Pasted image 20261104140025.png]]

[Buy Multi-tenant Front-end applications with Spring Boot and Thymeleaf - Complete Source Code](https://images.openai.com/static-rsc-4/caHc311XNwX4_XyxoKhigpCCMe2FOeVSXoaSzKRnjzzHkruKGZ19ITgq2vZw_8KhCZCH-oIoUsHqcicmJBVaVB6xQjy653qIZjYSpUQWQy4ptJaSlxfQquj1gsWGeuRMFEvWM2xliXfwKYcrD6Mk8VQlFg2i6b1r0Go02OjkIk8?purpose=inline)

[tech.asimio.net](https://tech.asimio.net/products/multitenant-frontend-springboot-and-thymeleaf-ui/?utm_source=chatgpt.com)

يعني Gym A وGym B وGym C بيستخدموا نفس الـ Backend ونفس الـ Deployment.

لكن الـ Application بتعرف كل Request تابع لأنهي Tenant.

مثلاً:

```
gym-a.yourplatform.com
gym-b.yourplatform.com
gym-c.yourplatform.com
```

لما Gym A يعمل Login أو يبعت Request:

```
GET /api/workouts
```

الـ Backend بيحدد إن الـ Request تابع لـ Gym A.

وبالتالي يرجع بيانات Gym A بس.

لما Gym B يبعت نفس الـ Request، يرجع بيانات Gym B.

==🔴نفس الـ Endpoint، نفس الـ Code، لكن البيانات مختلفة حسب الـ Tenant.==

وده جوهر الـ Multi-Tenancy.

# 4. أهم سؤال: إزاي بنفصل بيانات كل Tenant عن التاني؟

دي أهم نقطة في الموضوع كله، وفيها 3 ==🟢Design Approaches== مشهورين.

## Approach 1: Database per Tenant

كل Tenant عنده Database مستقلة.
![[Pasted image 20261104140314.png]]



```
             Application
                  |
          Tenant Resolution
                  |
          +-------+-------+
          |       |       |
        Gym A   Gym B   Gym C
          |       |       |
         DB A    DB B    DB C
```

مثلاً:

```
gym_a_db
gym_b_db
gym_c_db
```

لو Gym A عنده جدول Users فيه 1000 User، وGym B عنده 500 User، كل Database مستقلة تماماً.

المميزات:

- Data Isolation قوي جداً.
    
- Backup وRestore لكل Customer بشكل مستقل.
    
- تقدر تخصص Database Configuration لكل Tenant.
    

العيوب:

- إدارة Databases كتير.
    
- Database Migrations أصعب؛ لازم تتأكد إن كل قواعد البيانات اتحدثت.
    
- Infrastructure وConnection Management ممكن يبقوا مكلفين.
    

مناسب لما يكون عندك Enterprise Customers أو متطلبات عزل قوية.

## Approach 2: Schema per Tenant

هنا عندك Database واحدة، لكن كل Tenant عنده Schema مستقلة.
![[Pasted image 20261104143143.png]]



مثلاً PostgreSQL:

```
fitness_platform_db
│
├── gym_a
│   ├── users
│   ├── workouts
│   └── subscriptions
│
├── gym_b
│   ├── users
│   ├── workouts
│   └── subscriptions
│
└── gym_c
    ├── users
    ├── workouts
    └── subscriptions
```

كل Schema فيها نفس الـ Tables تقريباً، لكن البيانات مستقلة.

المميزات:

- عزل أفضل من Shared Schema.
    
- Database واحدة أسهل في بعض عمليات الإدارة.
    
- تقدر تعمل Backup أو Migration على مستوى Schema.
    

العيوب:

- إدارة الـ Schemas والـ Migrations مع زيادة العملاء.
    
- اختيار Schema الصحيحة لكل Request محتاج Configuration مضبوط.
    
- ممكن تواجه تحديات مع Connection Pooling.
    

## Approach 3: Shared Database + Shared Schema

وده من أشهر الاختيارات في SaaS Applications.

هنا كل العملاء بيستخدموا نفس Database ونفس Tables.

لكن بنضيف عمود اسمه:

`tenant_id

![[Pasted image 20261104145135.png]]`

[Kubernetes Resource Units Explained - Ajit Singh](https://images.openai.com/static-rsc-4/WCjtI8EgBltvV-KX6ZtFoG_1_8lp6_hav1YoBC14tczGWFMhUc-AWb9psXWRsLOOmoC-JrnyJbLIXAzkegLUIy9qP0LuyoLEv3ii0u65XgvWV4QW9c3B23oW7D5hKJw73sMwqz5Lkzcr73HK_q3zQEwulxinCInOSJeS6bS4rbA?purpose=inline)

[singhajit.com](https://singhajit.com/explainer/kubernetes-resource-units/?utm_source=chatgpt.com)

مثلاً جدول Workouts:

|id|name|tenant_id|
|---|---|---|
|1|Push Day|101|
|2|Leg Day|101|
|3|Upper Body|202|
|4|Full Body|303|

لاحظ إن:

- Gym A = Tenant 101.
    
- Gym B = Tenant 202.
    
- Gym C = Tenant 303.
    

كلهم موجودين في نفس الجدول!

لكن لما Gym A يطلب Workouts:

```
SELECT *
FROM workouts
WHERE tenant_id = 101;
```

ولما Gym B يطلب:

```
SELECT *
FROM workouts
WHERE tenant_id = 202;
```

كده كل واحد بيشوف بياناته بس.

المميزات:

- Infrastructure أقل تكلفة.
    
- أسهل في إدارة عدد كبير من العملاء.
    
- Deployment واحد.
    
- أسهل في الـ Scaling من ناحية إدارة العملاء والـ Instances.
    

العيوب:

- لازم تهتم جداً بالـ Data Isolation.
    
- أي Query ناقص فيه Tenant Filter ممكن يعمل Data Leakage.
    
- العملاء بيشاركوا نفس Database Resources.
    
- ممكن يحصل Noisy Neighbor Problem.
    

## مقارنة الـ 3 Approaches

|Feature|DB per Tenant|Schema per Tenant|Shared Schema|
|---|---|---|---|
|Isolation|قوي جداً|قوي|محتاج حماية دقيقة|
|Cost|أعلى|متوسط|أقل|
|Complexity|عالية|متوسطة|متوسطة|
|Scalability|جيدة|جيدة|جيدة جداً مع تصميم مناسب|
|Maintenance|أصعب|متوسطة|أسهل غالباً|

ملحوظة: مفيش Approach أفضل في كل الحالات. الاختيار بيعتمد على حجم المشروع، ومتطلبات الأمان، والتكلفة، وطبيعة العملاء.

# 5. طيب إيه اللي بيتغير في الكود فعلياً؟

خلينا ندخل في الجزء اللي إنت مهتم بيه كمطور Spring Boot.

هنفترض إنك اخترت:

Shared Database + Shared Schema.

أول حاجة محتاج تعملها هي تعريف Tenant في الـ Domain.

مثلاً:

```java
@Entity
@Table(name = "tenants")
public class Tenant {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String subdomain;

    private boolean active;
}
```

بعد كده أي Entity بياناتها خاصة بعميل معين لازم ترتبط بالـ Tenant.

مثلاً Workout:

```java
@Entity
@Table(name = "workouts")
public class Workout {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "tenant_id", nullable = false)
    private Tenant tenant;
}
```

الـ Database هتبقى بالشكل ده:

```sql
CREATE TABLE workouts (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255),
    tenant_id BIGINT NOT NULL,

    CONSTRAINT fk_workout_tenant
        FOREIGN KEY (tenant_id)
        REFERENCES tenants(id)
);
```

لاحظ إننا مش بنعمل Entity مختلفة لكل Gym.

إحنا بنعمل Entity واحدة، وكل Record جواها بيكون مرتبط بـ Tenant معين.

## طيب مين بيحدد الـ Tenant ID؟

مش المفروض الـ Frontend يبعتلك `tenantId` وتثق فيه!

لأن أي مستخدم ممكن يغير الرقم ويحاول يوصل لبيانات Tenant تاني.

الـ Flow الصحيح ممكن يكون كده:

1. Incoming Request

gym-a.yourplatform.com/api/workouts

2. Tenant Resolution

Resolve Tenant from trusted subdomain / domain mapping

3. Authentication & Authorization

Validate JWT and verify user's tenant membership

4. Tenant Context

Set current tenant in request-scoped context

5. Service + Repository

Execute tenant-scoped database queries

Result

Return current tenant's data only

خلينا نبسط الـ Implementation:

أولاً: TenantContext

ممكن تستخدم `ThreadLocal` في تطبيق Synchronous بسيط:

```java
public final class TenantContext {

    private static final ThreadLocal<Long> CURRENT_TENANT =
            new ThreadLocal<>();

    public static void setTenantId(Long tenantId) {
        CURRENT_TENANT.set(tenantId);
    }

    public static Long getTenantId() {
        return CURRENT_TENANT.get();
    }

    public static void clear() {
        CURRENT_TENANT.remove();
    }
}
```

ثانياً: Tenant Filter

الـ Filter بيشتغل في بداية الـ Request ويحدد الـ Tenant بعد التحقق من هوية المستخدم وارتباطه به.

```java
@Component
public class TenantFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        try {
            Long tenantId = resolveAndValidateTenant(request);

            TenantContext.setTenantId(tenantId);

            filterChain.doFilter(request, response);

        } finally {
            TenantContext.clear();
        }
    }
}
```

`resolveAndValidateTenant()` هنا مجرد توضيح للفكرة، مش Method جاهزة؛ لازم تنفذ فيها الـ Tenant Resolution والتحقق من الـ Membership والـ Authentication.

ثالثاً: Repository

```java
public interface WorkoutRepository
        extends JpaRepository<Workout, Long> {

    List<Workout> findAllByTenantId(Long tenantId);

    Optional<Workout> findByIdAndTenantId(
        Long id,
        Long tenantId
    );
}
```

رابعاً: Service

```java
@Service
@RequiredArgsConstructor
public class WorkoutService {

    private final WorkoutRepository workoutRepository;

    public List<Workout> getWorkouts() {

        Long tenantId = TenantContext.getTenantId();

        return workoutRepository.findAllByTenantId(tenantId);
    }
}
```

لاحظ هنا إن الـ Service مش بتاخد Tenant ID من الـ Frontend، لكنها بتاخده من الـ Trusted Context.

مهم جداً

الكود السابق تعليمي ومش كفاية لوحده للإنتاج. لازم تحمي كل عمليات القراءة والتعديل والحذف، وتتحقق من ملكية الـ Tenant عند إنشاء العلاقات، وتستخدم Database Constraints واختبارات تمنع الـ Cross-Tenant Access. كمان `ThreadLocal` محتاج تنظيف مضمون، ومش مناسب تلقائياً مع Reactive أو Async Processing؛ في الحالات دي محتاج Context Propagation مناسب.

# 6. طيب إيه الفرق بين Advantages وChallenges اللي في الصور؟

خلينا نفهمهم عملياً بدل ما نحفظهم.

## Advantages of Multi-Tenancy

1. Reduced Costs

بدل ما تدفع تكلفة Server وDatabase لكل Customer، بتشارك Infrastructure واحدة بين مجموعة عملاء.

2. Scalability

تقدر تزود عدد العملاء بدون ما تعمل Deploy جديد لكل عميل.

لكن ده مش معناه إن الـ Scaling تلقائي؛ محتاج تراقب الـ CPU والـ Memory والـ Database Connections.

3. Faster Deployment and Updates

لما تصلح Bug أو تضيف Feature، بتعمل Deploy للـ Application مرة واحدة، والعملاء كلهم يستفيدوا من التحديث.

4. Simplified Maintenance

بدل ما تدير 100 Application Instance بإصدارات مختلفة، ممكن تدير Application واحدة.

5. Increased Efficiency

استغلال أفضل للـ Infrastructure والـ Resources المشتركة.

## Potential Challenges

1. Security Concerns

تخيل إن Gym A قدر يعمل Request ويشوف Workout بتاع Gym B!

دي كارثة اسمها Cross-Tenant Data Leakage.

عشان كده الـ Tenant Isolation لازم يكون جزء أساسي من الـ Architecture، مش مجرد شرط بنضيفه في بعض الـ Queries.

2. Customization Limitations

Gym A عايز Feature معينة، وGym B مش عايزها.

مش منطقي نعمل نسخة مختلفة من الكود لكل واحد.

الحل ممكن يكون:

- Tenant Configuration.
    
- Feature Flags.
    
- Subscription Plans.
    
- Tenant-specific Settings.
    

مثلاً:

```
Gym A:
  AI_FEATURE = true
  CUSTOM_BRANDING = true

Gym B:
  AI_FEATURE = false
  CUSTOM_BRANDING = false
```

نفس الكود، لكن الـ Configuration مختلف.

3. Noisy Neighbor Problem

تخيل Gym A عنده 50,000 Trainee، وكلهم بيعملوا Requests في نفس الوقت.

ده ممكن يستهلك Database Connections وCPU وMemory ويأثر على Gym B وGym C.

الحلول:

- Rate Limiting.
    
- Resource Quotas.
    
- Caching.
    
- Query Optimization.
    
- Queue-based Processing.
    
- Monitoring.
    

4. System Outages

بما إن العملاء بيشاركوا نفس الـ Application Infrastructure، لو حصل Failure في الـ Shared Application أو Database ممكن يؤثر على عدد كبير من العملاء.

الحلول تشمل:

- High Availability.
    
- Replication.
    
- Load Balancing.
    
- Backups.
    
- Disaster Recovery.
    

# 7. إيه علاقة الكلام ده بالـ SaaS؟

دي نقطة مهمة جداً.

SaaS = Software as a Service

يعني إنت بتقدم Software للعميل كخدمة، غالباً عن طريق Subscription.

مثلاً:

Fitness SaaS Platform

SaaS
![[Pasted image 20261104151353.png]]

[Fitness Management SaaS Platform | CRM Dashboard Gym :: Behance](https://images.openai.com/static-rsc-4/cIR0I-2XvGml6wdZdE3-1m6PaB7yWVlkQGW2RNrNE4GWvXfcIV4O1gLNRUAqCGAqGR-G8yMmnhOS1cYMhMR16Kj8Yde3zayRCRixkZnKZMjujN445BXKLmoHXT74y1GMYSLUNS7nVWh8W-MLA4C-u49FODRdzGcSQO8I0BD0QJs?purpose=inline)

[behance.net](https://www.behance.net/gallery/184463855/Fitness-Management-SaaS-Platform-CRM-Dashboard-Gym?utm_source=chatgpt.com)

Basic Plan

# $19

Monthly example

- 2 Coaches
    
- 100 Trainees
    
- Basic Programs
    

Pro Plan

# $49

Monthly example

- Unlimited Coaches
    
- 1000 Trainees
    
- AI Features
    

Customers:

- Gym A → Basic Plan
    
- Gym B → Pro Plan
    
- Gym C → Basic Plan
    

الـ SaaS مش معناه بالضرورة Multi-Tenant، لكن Multi-Tenancy Architecture اختيار شائع جداً لبناء SaaS.

ممكن يكون عندك SaaS بنظام Single-Tenant لكل Customer، وممكن يكون Multi-Tenant.

الفرق إن SaaS هو Business/Delivery Model، بينما Multi-Tenancy هو Architectural Design.

# 8. Architecture Diagram لمشروع SaaS 
==🔴الصوره دي موضحه جدا== 
حقيقي

خلينا نجمع كل اللي فات في Diagram واحدة:
![[Pasted image 20261104151508.png]]



في Architecture دي، فيه حاجات إضافية لازم تاخد بالك منها:

- Redis: لازم الـ Cache Key يحتوي على Tenant ID عشان بيانات عميل متتسربش لعميل تاني.
    
- Background Jobs: كل Job لازم يحتفظ بالـ Tenant Context بتاعه.
    
- File Storage: لازم تعزل ملفات كل Tenant.
    
- Audit Logs: تسجل مين عمل إيه، وفي أنهي Tenant.
    
- Database Constraints: استخدم Composite Unique Constraints لما تكون الـ Uniqueness على مستوى Tenant، مثلاً `(tenant_id, email)`.
    

# 9. أنهي Architecture أنصحك تبدأ بيها؟

لو بنتكلم عن منصة الـ Fitness والـ Online Coaches اللي كنت بتفكر فيها، فأنا شايف إن البداية المناسبة ليها:

My recommendation

Modular Monolith + Shared Database + Shared Schema

ليه؟

1. هتبدأ بـ Application واحدة.
    
2. PostgreSQL Database واحدة.
    
3. كل Gym أو Coaching Organization ليه `tenant_id`.
    
4. كل User ممكن يكون مرتبط بـ Tenant واحد أو أكتر حسب الـ Business Requirements.
    
5. تقدر تضيف AI Features لكل Tenant حسب اشتراكه.
    
6. تقدر تتطور مستقبلاً لـ Database per Tenant للعملاء اللي محتاجين عزل أكبر.
    

ومتبدأش بـ Microservices لمجرد إن المشروع SaaS؛ الـ Modular Monolith كفاية جداً كبداية.

# 10. اختبر فهمك

عايزك تجاوب على الأسئلة دي عشان نتأكد إن المفهوم ثبت.

Multi-Tenancy Quiz

0/4 answered

1. عندك 100 Gym بيستخدموا نفس Spring Boot Application ونفس Database، وكل Record مرتبط بـ tenant_id. ده إيه؟

Single-Tenant

Multi-Tenant

Microservices

2. Gym A بعت Request وغيّر tenantId من 101 إلى 202 في الـ Frontend. إيه التصرف الصحيح؟

نقبل الـ ID عادي

نتحقق من Membership وTenant Context الموثوق

نعمل Database جديدة

3. Gym A محتاج Feature مختلفة عن Gym B. هل نعمل Codebase مختلفة؟

نعم، لكل عميل Codebase

لا، نستخدم Configuration أو Feature Flags

ننسخ الـ Database

4. إيه المشكلة لو عملت Cache Key اسمه workouts بس لكل العملاء؟

مفيش مشكلة

ممكن بيانات Tenant تظهر لTenant تاني

الـ Database هتتمسح

Check answers

الخطوة اللي بعدها أنصحك بيها: نعمل Design فعلي لـ Multi-Tenant Architecture باستخدام Spring Boot 4 وSpring Security وJPA وPostgreSQL، ونقسمه Feature by Feature، ونفهم Tenant Resolution وTenantContext وTenant Isolation وDatabase Design، مع ER Diagram وSequence Diagrams قبل كتابة أي كود.