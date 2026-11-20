# شرح `MultiTenantConnectionProvider` من الصفر (ريفرنس)

## 1) الفكرة الأساسية: الـ Multi-Tenancy بـ Schema-per-Tenant

عندك تطبيق SaaS واحد بيخدم شركات كتير (tenants). لازم داتا كل شركة تبقى معزولة عن التانية. فيه 3 استراتيجيات معروفة:

| الاستراتيجية          | الفكرة                           | العزل                   |
| --------------------- | -------------------------------- | ----------------------- |
| Database-per-tenant   | داتابيز منفصلة لكل tenant        | أقوى عزل، أغلى تكلفة    |
| **Schema-per-tenant** | داتابيز واحدة، schema لكل tenant | **اللي في الكود بتاعك** |
| Discriminator column  | جداول مشتركة + عمود `tenant_id`  | أضعف عزل، أرخص          |

في PostgreSQL الـ schema هو "namespace" جوه الداتابيز. يعني ممكن يبقى عندك `tenant_a.users` و `tenant_b.users`، جدولين بنفس الاسم في أماكن مختلفة.

---

## 2) الـ `search_path` في PostgreSQL

ده أهم مفهوم عشان تفهم الكود. لما تكتب:

```sql
SELECT * FROM users;
```

PostgreSQL مش عارف `users` دي في أنهي schema، فبيدور بالترتيب في الـ schemas المكتوبة في `search_path`.

```sql
SET search_path TO tenant_a, public;
SELECT * FROM users;   -- هيدور في tenant_a.users الأول، وبعدين public.users
```


**نقطة مهمة جداً:** الـ `search_path` هو **state على مستوى الـ connection/session**. يعني لو غيرته على connection معين، هيفضل متغير لحد ما حد يغيره تاني أو الـ connection يتقفل فعلياً.

---

## 3) دور `MultiTenantConnectionProvider`

Hibernate عنده سؤال في كل مرة يفتح Session: **"أنا شغال لـ tenant اسمه X، هاخد connection منين وإزاي أجهزه؟"**

الـ interface ده هو الإجابة. Hibernate بيناديه، وأنت بتقرر إزاي تجهز الـ connection للـ tenant ده.

الـ `<String>` هو نوع الـ tenant identifier. القيمة بتيجي من `CurrentTenantIdentifierResolver` (كلاس تاني بيقرا الـ tenant من الـ request/JWT/header).

---

## 4) الميثودز واحدة واحدة

### `getAnyConnection()`

```java
return this.dataSource.getConnection();
```

بترجع connection من الـ pool **من غير ما تحدد tenant**. Hibernate بيستخدمها وقت الـ startup (قراءة metadata، validation) أو في أي شغل مش مرتبط بـ tenant.

### `releaseAnyConnection(connection)`

```java
connection.close();
```

بترجع الـ connection اللي اتفتح بـ `getAnyConnection()`. مع HikariCP، الـ `close()` مش بتقفل الاتصال الفعلي، بترجعه للـ pool.

### `getConnection(tenantIdentifier)` ← **أهم ميثود**

```java
final Connection connection = getAnyConnection();
if (tenantIdentifier != null && !tenantIdentifier.equals("public")) {
    connection.createStatement().execute("SET search_path TO " + tenantIdentifier + ", public");
}
return connection;
```

الخطوات:

1. تاخد connection من الـ pool.
2. لو الـ tenant مش `null` ومش `"public"`، تضبط الـ `search_path` على schema الـ tenant.
3. ترجع الـ connection لـ Hibernate، وكل query بعد كده هتتنفذ على الـ tenant ده.

### `releaseConnection(tenantIdentifier, connection)`

```java
connection.createStatement().execute("SET search_path TO public");
connection.close();
```

بتتنفذ لما الـ session تخلص. بترجع الـ `search_path` لـ `public` وبعدين ترجع الـ connection للـ pool.

**ليه الـ reset ضروري؟** لأن الـ connection بيتعاد استخدامه. لو رجع للـ pool وهو لسه على `tenant_a`، وجه request بعده من غير tenant أو بـ tenant بيفشل في الضبط، هيقرا داتا `tenant_a` بالغلط. ده **data leak بين العملاء**.

### `supportsAggressiveRelease()`

```java
return false;
```

معناها Hibernate **مش** هيقفل الـ connection بعد كل statement، هيحتفظ بيه لحد نهاية الـ transaction. ده الصح هنا لأن الـ `search_path` state على الـ connection، ولو اتحرر في النص هتضيع الإعدادات.

### `isUnwrappableAs()` و `unwrap()`

ميثودز بتخلي Hibernate يسأل: "هل الـ provider ده ممكن أحوله لنوع تاني؟". الرد `false` / `null` عادي ومش بيأثر.

### `customize(Map)` ← من `HibernatePropertiesCustomizer`

```java
hibernateProperties.put(MultiTenancySettings.MULTI_TENANT_CONNECTION_PROVIDER, this);
```

دي **مش** من الـ interface بتاع Hibernate، دي من Spring Boot. وظيفتها تسجل الـ provider بتاعك في إعدادات Hibernate. من غيرها Hibernate مش هيعرف إن عندك provider مخصص.

---

## 5) الـ Flow الكامل

```
Request جاي (header: X-Tenant-ID: tenant_a)
   ↓
Filter/Interceptor يحفظ الـ tenant في ThreadLocal/Context
   ↓
CurrentTenantIdentifierResolver.resolveCurrentTenantIdentifier() → "tenant_a"
   ↓
Hibernate يفتح Session
   ↓
provider.getConnection("tenant_a")
   → connection من الـ pool
   → SET search_path TO tenant_a, public
   ↓
الـ queries تتنفذ على tenant_a
   ↓
provider.releaseConnection("tenant_a", connection)
   → SET search_path TO public
   → connection.close() (يرجع للـ pool)
```

---

## 6) الثغرة: SQL Injection

### أين المشكلة؟

```java
connection.createStatement().execute("SET search_path TO " + tenantIdentifier + ", public");
```

الـ `tenantIdentifier` بيتحط **مباشرة كنص** جوه الـ SQL. ولو القيمة دي جاية من حاجة المستخدم يقدر يتحكم فيها (header، JWT مش متوقع، query param، subdomain)، المهاجم يكتب فيها SQL.

### مثال هجوم

لو المهاجم بعت:

```
X-Tenant-ID: tenant_a; DROP TABLE users; --
```

الـ SQL النهائي هيبقى:

```sql
SET search_path TO tenant_a; DROP TABLE users; --, public
```

والـ JDBC driver بتاع PostgreSQL بيسمح بتنفيذ أكتر من statement في نفس الـ `execute()` (simple query protocol)، يعني الـ `DROP TABLE` ممكن فعلاً يتنفذ.

### هجوم تاني أخطر: Tenant Hopping

حتى من غير ما يكتب SQL ضار، المهاجم ممكن يبعت اسم schema لـ tenant تاني:

```
X-Tenant-ID: tenant_b
```

ولو الـ app مش بتتأكد إن اليوزر ده فعلاً تابع لـ `tenant_b`، هيشوف داتا عميل تاني بالكامل. ده مش SQL injection تقني، لكنه ثغرة عزل (authorization) وأخطر من ناحية البزنس.

### ليه مينفعش `PreparedStatement`؟

`SET search_path` هو DDL/utility command، والـ parameters (`?`) مش بتشتغل مع الأوامر دي في PostgreSQL. فالحل مش parameterization، الحل **validation**.

---

## 7) الحل

### أ) Validation على الـ identifier (الحد الأدنى)

```java
private static final Pattern TENANT_PATTERN = Pattern.compile("^[a-z][a-z0-9_]{0,62}$");

private void validateTenant(String tenantIdentifier) {
    if (!TENANT_PATTERN.matcher(tenantIdentifier).matches()) {
        throw new IllegalArgumentException("Invalid tenant identifier");
    }
}
```

(63 هو الحد الأقصى لطول identifier في PostgreSQL.)

### ب) Whitelist: تأكد إن الـ tenant موجود فعلاً

خزّن الـ tenants في جدول `public.tenants` واعمل cache ليهم. أي identifier مش موجود فيه يترفض. ده بيمنع الـ injection وبيمنع استخدام schemas مش المفروض تتوصل (زي `pg_catalog` أو `information_schema`).

### ج) اربط الـ tenant بالمستخدم المُصادَق عليه

**الأهم:** متثقش في header بيبعته الـ client. خد الـ tenant من الـ JWT الموقّع (claim جوه التوكن)، بحيث المستخدم مايقدرش يغيره.

### د) Quote الـ identifier كطبقة حماية إضافية

```java
private String quote(String identifier) {
    return "\"" + identifier.replace("\"", "\"\"") + "\"";
}
```

ده بيتعامل مع الـ identifier كاسم schema بس، لكن **ما يغنيش عن الـ validation**.

---

## 8) كود محسّن (نسخة مرجعية)

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class MultiTenantConnectionProviderImpl
        implements MultiTenantConnectionProvider<String>, HibernatePropertiesCustomizer {

    private static final Pattern TENANT_PATTERN = Pattern.compile("^[a-z][a-z0-9_]{0,62}$");
    private static final String DEFAULT_SCHEMA = "public";

    private final DataSource dataSource;

    @Override
    public Connection getAnyConnection() throws SQLException {
        return dataSource.getConnection();
    }

    @Override
    public void releaseAnyConnection(Connection connection) throws SQLException {
        connection.close();
    }

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        log.debug("Getting connection for tenant: {}", tenantIdentifier);
        final Connection connection = getAnyConnection();
        try {
            if (tenantIdentifier != null && !DEFAULT_SCHEMA.equals(tenantIdentifier)) {
                if (!TENANT_PATTERN.matcher(tenantIdentifier).matches()) {
                    throw new SQLException("Invalid tenant identifier");
                }
                try (Statement stmt = connection.createStatement()) {
                    stmt.execute("SET search_path TO \"" + tenantIdentifier + "\", public");
                }
            }
            return connection;
        } catch (SQLException | RuntimeException e) {
            log.error("Error getting connection for tenant: {}", tenantIdentifier, e);
            connection.close();   // متسيبش connection مفتوح لو فشل الضبط
            throw e;
        }
    }

    @Override
    public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
        try (Statement stmt = connection.createStatement()) {
            stmt.execute("SET search_path TO public");
        } catch (SQLException e) {
            log.error("Error releasing connection for tenant: {}", tenantIdentifier, e);
        } finally {
            connection.close();
        }
    }

    @Override public boolean supportsAggressiveRelease() { return false; }
    @Override public boolean isUnwrappableAs(Class<?> unwrapType) { return false; }
    @Override public <T> T unwrap(Class<T> unwrapType) { return null; }

    @Override
    public void customize(Map<String, Object> props) {
        props.put(MultiTenancySettings.MULTI_TENANT_CONNECTION_PROVIDER, this);
    }
}
```

---

## 9) ملاحظات إضافية للمراجعة

1. **الـ `Statement` لازم يتقفل** (try-with-resources)، وإلا بيحصل resource leak.
2. **في `getConnection`:** لو الـ `SET search_path` فشل، الكود الأصلي كان بيرمي الـ exception **من غير ما يقفل الـ connection**، يعني connection leak في الـ pool. الحل بالـ `connection.close()` في الـ catch.
3. **في `releaseConnection`:** لو الـ reset فشل، الـ connection ممكن يرجع للـ pool على schema غلط. في الأنظمة الحساسة ممكن تفكر تعمل invalidate للـ connection بدل ما ترجعه.
4. **رسالة اللوج** في `releaseConnection` كانت "Error getting" والأصح "Error releasing".
5. **اختبر العزل:** اعمل integration test بيتأكد إن request لـ `tenant_a` مايقدرش يشوف داتا `tenant_b`.

---

## الخلاصة

- الـ provider بيجهز connection خاص بكل tenant عن طريق `search_path`.
- الـ `search_path` state على الـ connection، فلازم يتعمله reset عند الإرجاع.
- الثغرة: دمج قيمة جاية من الـ request في SQL، ومفيش `PreparedStatement` بيحلها هنا، فالحل **validation + whitelist + ربط الـ tenant بالـ JWT**.
- الأمان الحقيقي بيجي من إن الـ tenant مش جاي من إدخال المستخدم أصلاً، وإن الـ identifier مضمون قبل ما يوصل للـ SQL.

لو حابب، الخطوة الجاية نشرح `CurrentTenantIdentifierResolver` وإزاي تاخد الـ tenant من الـ JWT، أو إزاي تعمل provisioning لـ schema جديد بـ Flyway.