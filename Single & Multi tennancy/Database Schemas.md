in hibernate we have 
ي Hibernate فيه مفهوم اسمه:

MultiTenantConnectionProvider

وده مسؤول عن توفير Connection مناسبة للـ Tenant المطلوب.

وفيه:

CurrentTenantIdentifierResolver

وده مسؤول عن تحديد الـ Tenant الحالي.


بص يا علي، خلينا نفهم الـ Schema من الصفر، ونربطها بـ PostgreSQL وموضوع الـ Multi-Tenancy اللي بتدرسه، لأن دي من أهم المفاهيم اللي لازم تكون فاهمها كويس قبل ما تختار Database Architecture لأي SaaS Application.

# 1. يعني إيه Database Schema أصلاً؟

تخيل إن عندك Database اسمها `learning_academy`.

الـ Database دي زي مبنى كبير، وجواه مجموعة من الأقسام، وكل قسم مسؤول عن تنظيم مجموعة معينة من البيانات.

الأقسام دي هي الـ Schemas.

الـ Schema في PostgreSQL هي عبارة عن Logical Namespace بيجمع وينظم Database Objects تحت اسم واحد.

يعني الـ Schema ممكن تحتوي على:

- Tables
    
- Views
    
- Indexes
    
- Sequences
    
- Functions
    
- Procedures
    
- Types
    

بمعنى إن الـ Schema مش مجرد مجموعة Tables، لكنها مساحة منطقية لتنظيم وعزل الـ Database Objects.

[Schema vs Database in PostgreSQL: What's the Difference? - When to Use Schemas vs Separate Databases - PostgreSQL Data Organization Guide 2026 | QueryGlow Blog](https://images.openai.com/static-rsc-4/9aSttFiozRpzGz9DFcG7iQ1_3Zlk4gCmAMz_Ch3kgtqbKkP2SMjD9vmsHaR2CIwTR1OpqVMV9wL6AVunIOzTIpnup_UVNPjJ-KtAJM_Pu2WHgSpbd12pqFT41fiUKKN6gXWbrSN2pqftMv3v1yI2CFd4m6t_VDW608bNllVclH8?purpose=inline)

[queryglow.com](https://queryglow.com/blog/schema-vs-database-postgresql?utm_source=chatgpt.com)

# 2. طيب إزاي Database واحدة يكون جواها أكتر من Schema؟

خلينا ناخد مثال عملي.

عندنا Database اسمها:

```
CREATE DATABASE learning_academy;
```

بعد ما تعمل Connect على الـ Database، تقدر تنشئ أكتر من Schema:

```
CREATE SCHEMA academy;
CREATE SCHEMA finance;
CREATE SCHEMA analytics;
```

كده عندك Database واحدة فيها 3 Schemas.

وكل Schema تقدر تحتوي على Tables مختلفة.

مثلاً:

Database: learning_academy

Schema: academy

Tables

- users
    
- courses
    
- lessons
    
- enrollments
    

Schema: finance

Tables

- payments
    
- invoices
    
- transactions
    

Schema: analytics

Tables

- reports
    
- user_activity
    
- course_statistics
    

لاحظ إن كل الـ Schemas دي موجودة داخل نفس الـ Database.

وده معناه إنك مش محتاج تعمل Database جديدة لكل مجموعة Tables.

# 3. إزاي أتحكم في الـ Schemas؟

هنا بقى PostgreSQL بتديلك تحكم كبير.

أولاً: إنشاء Schema

```
CREATE SCHEMA tenant_a;
CREATE SCHEMA tenant_b;
```

ثانياً: إنشاء Table داخل Schema معينة

```
CREATE TABLE tenant_a.users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150)
);
```

لاحظ الصيغة:

```
schema_name.table_name
```

يعني `tenant_a.users` معناها جدول users الموجود داخل Schema اسمها tenant_a.

تقدر تعمل نفس اسم الجدول في Schema تانية عادي جداً:

```
CREATE TABLE tenant_b.users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150)
);
```

دلوقتي عندك جدولين اسمهم users، لكن كل واحد في Schema مختلفة.

ثالثاً: قراءة البيانات من Schema معينة

```
SELECT * FROM tenant_a.users;

SELECT * FROM tenant_b.users;
```

PostgreSQL عارفة بالضبط أنت تقصد أنهي جدول.

رابعاً: حذف Schema

```
DROP SCHEMA tenant_a;
```

لكن لو فيها Tables، الأمر ده مش هيشتغل إلا لو الـ Schema فاضية.

ممكن تستخدم:

```
DROP SCHEMA tenant_a CASCADE;
```

خلي بالك: `CASCADE` بيحذف الـ Schema وكل الـ Objects التابعة ليها. خطير جداً في Production.

# 4. إيه هو Search Path؟

دي نقطة مهمة جداً.

تخيل إن عندك:

```
tenant_a.users
tenant_b.users
```

وأنت كتبت:

```
SELECT * FROM users;
```

PostgreSQL هتعرف منين تقصد أنهي جدول؟

هنا بييجي دور الـ Search Path.

الـ Search Path بيحدد ترتيب الـ Schemas اللي PostgreSQL تدور فيها لما تستخدم اسم Object من غير ما تحدد الـ Schema.

اعرضه:

```
SHOW search_path;
```

ممكن تلاقي:

```
"$user", public
```

تقدر تغيره:

```
SET search_path TO tenant_a, public;
```

دلوقتي لما تعمل:

```
SELECT * FROM users;
```

PostgreSQL هتدور في `tenant_a` الأول.

وتقدر تعمل:

```
SET search_path TO tenant_b, public;
```

كده هتدور في `tenant_b` الأول.

معلومة مهمة: الـ Search Path مش وسيلة Security أو Tenant Isolation لوحده. لازم تتحكم في الصلاحيات وتمنع المستخدم من الوصول لبيانات Tenant تانية.

# 5. إزاي أتحكم في مين يقدر يدخل على Schema معينة؟

هنا بنتكلم عن PostgreSQL Permissions.

مثلاً عندك User اسمه `app_user`.

تقدر تديله صلاحية الوصول لـ Schema:

```
GRANT USAGE ON SCHEMA tenant_a TO app_user;
```

وتديله صلاحية القراءة من Tables الموجودة فيها:

```
GRANT SELECT ON ALL TABLES
IN SCHEMA tenant_a
TO app_user;
```

ولو عايزه يقدر يعمل Insert وUpdate وDelete:

```
GRANT INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA tenant_a
TO app_user;
```

ولو عايز تمنعه من الوصول:

```
REVOKE USAGE ON SCHEMA tenant_a FROM app_user;
```

لاحظ الفرق:

- `USAGE`: تسمح له باستخدام Objects الموجودة في الـ Schema لو عنده صلاحيات عليها.
    
- `SELECT`: تسمح له بقراءة بيانات الـ Tables.
    
- `INSERT`: تسمح له بإضافة بيانات.
    

الـ Permissions بتتكون من أكتر من مستوى، ومجرد `USAGE` مش معناه إن عنده صلاحية قراءة كل البيانات.

# 6. نربط الكلام ده بالـ Multi-Tenancy

دي أهم نقطة بالنسبة للـ SaaS Architecture.

تخيل إنك بتعمل SaaS Learning Platform، وعندك 3 شركات مشتركة في النظام:

- Company A
    
- Company B
    
- Company C
    

وكل شركة عايزة بياناتها تكون معزولة عن الشركات التانية.

عندك 3 اختيارات مشهورة:

## Architecture 1: Shared Database + Shared Schema

[Multi-Tenant SaaS with Django: The Three Isolation Strategies and How to Pick the Right One | by Md Mojno Miya | Apr, 2026 | Medium](https://images.openai.com/static-rsc-4/U31_NKnYhlqMIkQ8uxLXu6GyKclYqMC9hINJOhL15EsyEPdSw2k8gZMHm3WE4ut2m2mxoLksNpfVx5kgzhsJxbbtuMi5mPZwIsSLGIhBdtg1UFDz_KPqui5r5_117LvMY2cIvr2S9tIVqeuI81jDafsKrhrmWsFcIfYWCzBNWsE?purpose=inline)

[medium.com](https://medium.com/%40mmoznu/multi-tenant-saas-with-django-the-three-isolation-strategies-and-how-to-pick-the-right-one-11baa44740b9?utm_source=chatgpt.com)

كل الشركات بتستخدم نفس الـ Database ونفس الـ Tables.

مثلاً:

users table

|id|name|tenant_id|
|---|---|---|
|1|Ali|101|
|2|Ahmed|102|
|3|Omar|101|
|4|Mohamed|103|

هنا كل Record عنده `tenant_id`.

لما Company A تطلب بيانات المستخدمين:

```
SELECT *
FROM users
WHERE tenant_id = 101;
```

كل الشركات مشتركة في نفس الـ Tables، والـ Isolation بيعتمد على تطبيق Tenant Filtering بشكل صحيح.

## Architecture 2: Shared Database + Separate Schemas

[Multi-Tenant Architecture](https://images.openai.com/static-rsc-4/nFeeI_1Enqt2WILtTDiQVsVi4DzAh7bvUj7Xwqsy5MFdLFlQ4rWaJTQ4xgJWH_UuRi-IMFpnm-s5BpLvip1pSazHgHe1_nAzMSYD_1TXPYcQY3uaqkjZGZ7L6Rnrg3dqDA65FOzlg2Z8yQSlQEZ5XF7lP8rVcUk4c5qjPOX3CcY?purpose=inline)

[linkedin.com](https://www.linkedin.com/pulse/multi-tenant-architecture-codecraft-technologies-private-lim?utm_source=chatgpt.com)

هنا Database واحدة، لكن كل Tenant ليه Schema منفصلة.

```
learning_academy
│
├── tenant_101
│   ├── users
│   ├── courses
│   └── enrollments
│
├── tenant_102
│   ├── users
│   ├── courses
│   └── enrollments
│
└── tenant_103
    ├── users
    ├── courses
    └── enrollments
```

لاحظ إن كل Tenant عنده نفس تصميم الجداول، لكن البيانات منفصلة.

مثلاً:

```
SELECT * FROM tenant_101.users;
```

مستحيل الجدول ده يرجع بيانات `tenant_102` لأنه جدول مختلف تماماً.

لكن لازم تفضل عامل حساب الـ Database Permissions والـ Application Security، لأن فصل الـ Schema مش معناه إن أي Application User ممنوع تلقائياً من باقي الـ Schemas.

## Architecture 3: Separate Database Per Tenant

[Multi-Tenant Architecture: The Complete Guide for SaaS | QuantumByte Success Story](https://images.openai.com/static-rsc-4/QIfSAsZejxZKLIeblwYE0HoqSAa7QWsU3L5UrOj7m8JKkaka9eCHc8uD1YAJpG_EbX0Vy2Sf_X-TePc6tgYyDRVtLi56paXtnVN4TT4eo_KXNRlxWD3VNS8pHyrUGLRh9_2ylwAtuqWCHsAms_xQhGVjiUG0uuI9uxKbGLVQ-rw?purpose=inline)

[quantumbyte.ai](https://quantumbyte.ai/articles/multi-tenant-architecture?utm_source=chatgpt.com)

هنا كل شركة ليها Database مستقلة:

```
PostgreSQL Server
│
├── tenant_101_db
├── tenant_102_db
└── tenant_103_db
```

ده بيدي Isolation أقوى على مستوى قواعد البيانات، لكنه بيزود تكلفة الإدارة والـ Connections والـ Migrations والـ Backups.

# 7. طيب في Spring Boot أتحكم في الـ Schema إزاي؟

خلينا نفترض إنك اخترت Separate Schema per Tenant.

السيناريو هيكون:

1. المستخدم يعمل Login.
    
2. Spring Security يتحقق من هويته.
    
3. تستخرج الـ Tenant ID المرتبط بالمستخدم من بيانات موثوقة.
    
4. تحدد الـ Schema الخاصة بالـ Tenant.
    
5. Hibernate ينفذ Queries على الـ Schema الصحيحة.
    

في Hibernate فيه مفهوم اسمه:

MultiTenantConnectionProvider

وده مسؤول عن توفير Connection مناسبة للـ Tenant المطلوب.

وفيه:

CurrentTenantIdentifierResolver

وده مسؤول عن تحديد الـ Tenant الحالي.

مثال مبسط للفكرة:

```
@Component
public class TenantIdentifierResolver
        implements CurrentTenantIdentifierResolver<String> {

    @Override
    public String resolveCurrentTenantIdentifier() {
        return TenantContext.getCurrentTenant();
    }

    @Override
    public boolean validateExistingCurrentSessions() {
        return true;
    }
}
```

وفي الـ Connection Provider تقدر تضبط الـ Schema للـ Connection:

```
connection.setSchema(tenantSchema);
```

طبعاً ده مثال توضيحي، والـ Production Implementation محتاجة تتعامل مع Connection Pooling وإرجاع الـ Connection لحالتها الأصلية، والتحقق من الـ Tenant Schema قبل استخدامها.

تحذير أمني مهم: متاخدش اسم الـ Schema مباشرة من Request Parameter أو Header وتنفذ بيه SQL. لازم تربطه بالـ Tenant المصرح للمستخدم بالوصول ليه، وتستخدم أسماء Schemas من Registry موثوق، مش User Input.

# 8. اختبر فهمك

خلينا نشوف هل فهمت الفرق بين Database وSchema وTable ولا لأ.

Schema Fundamentals Quiz

4/4 Answered

1. إيه الوصف الأدق للـ Schema في PostgreSQL؟

Physical Database منفصلة

Logical Namespace بتنظم Database Objects

مجرد Table كبيرة

Incorrect

الـ Schema عبارة عن Namespace بتجمع Tables وViews وFunctions وغيرها داخل Database واحدة.

2. عندك tenant_a.users و tenant_b.users، هل ممكن يكونوا بنفس الاسم؟

لا، ممنوع

آه، لأن كل واحد في Schema مختلفة

آه، لكن لازم Database مختلفة

Correct

الاسم الكامل للـ Object هو schema_name.table_name، فمفيش تعارض.

3. إيه وظيفة search_path؟

تحديد ترتيب البحث عن Objects غير المؤهلة باسم Schema

إنشاء Schema تلقائياً

منع المستخدم من الوصول لباقي Schemas

Correct

Search Path بيحدد ترتيب الـ Schemas اللي PostgreSQL تبحث فيها.

4. هل Shared Database + Separate Schemas معناها إن Tenant Isolation مضمون تلقائياً؟

نعم تماماً

لا، لازم Security وPermissions وضبط Tenant Resolution

لازم نضيف tenant_id لكل Table بالضرورة

Correct

الفصل بالـ Schema يساعد في العزل، لكن لازم تمنع الوصول غير المصرح به وتتحقق من اختيار الـ Tenant.

# 3 / 4

Try again

الخلاصة اللي عايزك تحفظها:

- Database: الحاوية الكبيرة.
    
- Schema: Logical Namespace بتنظم الـ Database Objects.
    
- Table: المكان اللي بيحتوي على Rows وColumns.
    
- Search Path: بيحدد PostgreSQL تدور في أنهي Schema.
    
- GRANT / REVOKE: للتحكم في الصلاحيات.
    

والأهم: في Multi-Tenancy، اختيار Shared Schema أو Separate Schemas مش مجرد تنظيم للداتا؛ ده قرار Architecture بيأثر على Security وScalability وMigrations وCaching وOperations.