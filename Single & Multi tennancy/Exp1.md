

**الفكرة الأساسية:**

- **Single-Tenant:** كل عميل (tenant) ليه نسخة كاملة منفصلة من التطبيق، يعني سيرفر وداتابيز خاصين بيه.
- **Multi-Tenant:** نسخة واحدة من التطبيق شغالة، وكل العملاء بيستخدموها مع بعض، والبيانات بتتفصل منطقيًا بـ `tenant_id`.

**التصحيح:** الفرق مش إن الكود نفسه مختلف. الكود ممكن يكون واحد في الحالتين. الفرق في **عدد النسخ المتشغلة (deployments)**. عشان كده في Multi-Tenant لما تعمل deploy مرة واحدة، التعديل بيوصل لكل العملاء في نفس اللحظة. وفي Single-Tenant لازم تعمل deploy لكل عميل لوحده.

هرسملك الصورة الأول:

الرسمة دي بتوضح الفرق في الشكل المعماري نفسه:

جرّب الـ demo ده بنفسك. اضغط على الزرار في كل جهة وشوف إيه اللي بيحصل لما تعمل update:

**اللي بيوضحه الـ demo:**
![[Pasted image 20261104135241.png]]

في الـ Single-Tenant بتـ deploy لكل عميل لوحده. ده معناه إن تحكمك أكبر، وتقدر تأجل الـ upgrade لعميل enterprise معين. بس في المقابل بيحصل `version drift`، يعني عميل على v1.4 وعميل على v1.7، وكل واحد عنده bugs مختلفة، والـ support بيتعب جدًا.

في الـ Multi-Tenant بتـ deploy مرة واحدة والكل يتحدث. ده أسرع وأريح، بس لو فيه bug هيوصل للكل في نفس اللحظة (`blast radius` كبير).

وعلى الـ 3 مستويات اللي في الرسمة التانية: كل ما تروح يمين، العزل (isolation) بيزيد والتكلفة بتزيد معاه.

## التحديات اللي في الـ slide، وإزاي تتعامل معاها

|التحدي|الحل العملي|
|---|---|
|Security concerns (تسريب داتا بين العملاء)|فلترة تلقائية بـ `tenant_id`، واختبارات تتأكد إن tenant مش بيشوف داتا غيره، وPostgreSQL Row Level Security كطبقة حماية تانية|
|Customization limitations|بدل ما تعمل fork للكود لكل عميل، استخدم feature flags وجدول settings لكل tenant|
|Noisy neighbor|rate limiting وquotas لكل tenant، وحدود على الـ connection pool|
|System outages|canary وblue/green deployments، وfeature flags عشان تقفل الميزة الجديدة بسرعة|

## إزاي تنفذها في Spring Boot (shared schema)

بالنسبة للـ Spring Boot 3 (Hibernate 6)، الفكرة في 3 خطوات. أولًا، `TenantContext` بيحفظ الـ tenant الحالي في `ThreadLocal`:

```java
public final class TenantContext {
    private static final ThreadLocal<String> CURRENT = new ThreadLocal<>();
    public static void set(String id) { CURRENT.set(id); }
    public static String get() { return CURRENT.get(); }
    public static void clear() { CURRENT.remove(); }
}
```

ثانيًا، Filter بيقرأ الـ tenant من الـ request (في الـ production خده من الـ JWT claim مش من header عادي):

```java
@Component
public class TenantFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req,
            HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        try {
            TenantContext.set(req.getHeader("X-Tenant-Id"));
            chain.doFilter(req, res);
        } finally {
            TenantContext.clear();
        }
    }
}
```

ثالثًا، الـ Entity بـ `@TenantId`، وHibernate بيملّي العمود ويفلتر الـ queries لوحده:

```java
@Entity
public class Order {
    @Id @GeneratedValue private Long id;
    @TenantId private String tenantId;
    private BigDecimal total;
}

@Component
public class TenantResolver implements CurrentTenantIdentifierResolver<String> {
    public String resolveCurrentTenantIdentifier() {
        String t = TenantContext.get();
        return t != null ? t : "system";
    }
    public boolean validateExistingCurrentSessions() { return true; }
}

@Bean
HibernatePropertiesCustomizer tenantCustomizer(TenantResolver r) {
    return p -> p.put(AvailableSettings.MULTI_TENANT_IDENTIFIER_RESOLVER, r);
}
```

خد بالك إن الـ native SQL queries مش بتتفلتر أوتوماتيك، فأي native query لازم تضيف فيها `tenant_id` بإيدك. ده مصدر تسريب الداتا الأشهر.

## توصيتي لأول SaaS ليك

1. ابدأ بـ shared schema مع `tenant_id` على كل جدول، وحط index على `(tenant_id, ...)`.
2. خلّي تحديد الـ tenant في مكان واحد بس (الـ filter)، وماتفرّقوش في الكود.
3. صمّم من البداية إنك تقدر تنقل عميل كبير لـ database لوحده بعدين. ده الـ hybrid model اللي معظم شركات الـ SaaS بتوصله.
4. الـ Single-Tenant ماتروحلوش إلا لو عميل enterprise طلب عزل كامل (compliance) وعايز يدفع مقابله.

تحب أعملك مشروع Spring Boot صغير شغال بـ schema-per-tenant عشان تشوف الفرق بين المستويين عمليًا؟