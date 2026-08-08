 **Part 2**
    
    - Naming Convention
        
    - SQL Scripts
        
    - Versioning
        
    - Repeatable Migrations
        
    - Folder Structure
        
    - أول Migration عملي.
- ---




# Part 2 - Flyway Deep Dive

هنشرح:

1. Naming Convention
    
2. SQL Scripts
    
3. Versioning
    
4. Repeatable Migrations
    
5. Folder Structure
    
6. أول Migration عملي
    

---

# أولاً: Naming Convention

دي أهم نقطة في Flyway.

Flyway مش بيقرأ أي SQL file وخلاص.

هو بيقرأ الملفات بناءً على اسمها.

لازم يكون الاسم متوافق مع Pattern معين.

الـ Pattern الأساسي:

```text
V<Version>__<Description>.sql
```

لاحظ:

```
V
```

كابيتال.

بعدها رقم النسخة.

بعدها

```
__
```

اتنين underscore.

بعدها اسم يوضح الملف.

---

مثال

```text
V1__create_users_table.sql
```

أو

```text
V2__create_courses_table.sql
```

أو

```text
V3__add_email_to_users.sql
```

أو

```text
V4__create_indexes.sql
```

---

## لماذا لازم كده؟

لأن Flyway بيعمل Parsing للاسم.

يعني لما يشوف

```
V5__add_phone.sql

```

يفهم إن:

Version = 5

Description = add phone

---

لو كتبت

```text
create_users.sql
```

مش هيشغله.

لو كتبت

```text
Migration1.sql
```

مش هيشغله.

لو كتبت

```text
v1_create.sql
```

مش هيشغله.

---

## مثال

عندك الملفات دي:

```
V1__create_users.sql
V2__create_courses.sql
V3__create_lessons.sql
```

Flyway هيشغلهم بالترتيب.

---

# ماذا لو كتبت؟

```
V10__create_roles.sql
V2__create_course.sql
```

هيشتغلوا كده:

```
V2

ثم

V10
```

مش حسب ترتيب الملفات.

حسب رقم الـ Version.

---

## الوصف Description

الوصف مجرد اسم.

مثلاً

```
V5__add_phone_number.sql
```

أو

```
V5__phone.sql
```

الاتنين شغالين.

لكن الأفضل يكون واضح.

---

أفضل تسمية

```
V8__create_enrollments_table.sql
```

أسوأ تسمية

```
V8__test.sql
```

---

# ثانياً: SQL Scripts

كل Migration عبارة عن SQL File.

مثلاً:

```
V1__create_users.sql
```

جواه

```sql
CREATE TABLE users(
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(255)
);
```

خلص.

---

Migration تاني

```
V2__create_courses.sql
```

```sql
CREATE TABLE courses(
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(255)
);
```

---

Migration تالت

```
V3__add_price_to_course.sql
```

```sql
ALTER TABLE courses
ADD COLUMN price NUMERIC(10,2);
```

---

لاحظ إن كل File بيعمل Task واحدة فقط.

وده Best Practice.

---

غلط

```sql
CREATE TABLE users

CREATE TABLE courses

CREATE TABLE lessons

CREATE TABLE reviews

CREATE TABLE enrollments
```

كلهم في File واحد.

---

صح

```
V1 users

V2 courses

V3 lessons

V4 reviews

V5 enrollments
```

كل واحدة لوحدها.

---

# ثالثاً: Versioning

دي أهم فكرة في Flyway.

كل Migration ليها Version.

مثلاً

```
V1

V2

V3

V4

V5
```

يعني تاريخ الداتا بيز.

---

تخيل عندك المشروع بدأ من الصفر.

اليوم الأول

```
V1
```

أنشأت users.

---

بعد أسبوع

```
V2
```

أنشأت courses.

---

بعد شهر

```
V3
```

أضفت email verification.

---

بعد شهرين

```
V4
```

عملت index.

---

كل ده محفوظ.

---

أي Developer جديد يدخل الفريق.

يشغل المشروع.

Flyway ينفذ:

```
V1

↓

V2

↓

V3

↓

V4
```

والداتا بيز تبقى نفس الموجودة عند باقي الفريق.

---

# هل أقدر أغير V2؟

لا.

ودي قاعدة ذهبية.

بعد ما Migration تتنفذ...

❌ ممنوع تعدلها.

---

بدل

```
V2
```

اعمل

```
V5
```

فيه التعديل الجديد.

---

ليه؟

لأن V2 اتنفذت بالفعل عند باقي الفريق.

لو غيرتها:

- الـ Checksum هيتغير.
    
- Flyway هيعتبر إن في حد لعب في التاريخ.
    

---

# رابعاً: Repeatable Migration

دي مختلفة.

بدل

```
V
```

بتستخدم

```
R
```

مثال

```
R__create_views.sql
```

أو

```
R__create_functions.sql
```

---

Repeatable معناها:

كل مرة الملف يتغير...

Flyway يعيد تشغيله.

---

ليه؟

علشان في حاجات بتتغير باستمرار.

زي

```
VIEW
```

أو

```
FUNCTION
```

أو

```
Stored Procedure
```

---

مثلاً

```sql
CREATE OR REPLACE VIEW active_users AS
SELECT *
FROM users
WHERE active=true;
```

بعد أسبوع عدلتها.

بدل

```
WHERE active=true
```

خليتها

```sql
WHERE enabled=true
```

هنا Flyway يشوف إن الملف اتغير.

فيعيد تشغيله.

---

لكن

Version Migration

```
V5
```

لا.

يتنفذ مرة واحدة فقط.

---

### الفرق

|Version Migration|Repeatable Migration|
|---|---|
|V|R|
|مرة واحدة|كلما الملف يتغير|
|CREATE TABLE|VIEW|
|ALTER TABLE|FUNCTION|
|تاريخ الداتا بيز|Objects قابلة لإعادة الإنشاء|

---

# خامساً: Folder Structure

داخل Spring Boot

```
src
 └── main
      └── resources
            └── db
                 └── migration
```

دا المكان الافتراضي.

---

يبقى

```
resources
    db
      migration
```

جواه

```
V1__create_users.sql

V2__create_courses.sql

V3__create_lessons.sql

R__create_views.sql
```

---

لما Spring يبدأ.

Flyway يروح للمجلد ده.

ويقرأ كل الملفات.

---

ممكن تغير المكان؟

أكيد.

في

```properties
spring.flyway.locations=classpath:database/migrations
```

لكن 99% من الناس بتستخدم

```
db/migration
```

---

# Flyway History Table

دي أهم حاجة بتحصل خلف الكواليس.

أول مرة Flyway يشتغل...

هيعمل Table اسمه

```
flyway_schema_history
```

جوه قاعدة البيانات.

ودي عبارة عن دفتر تاريخ.

مثال:

|Version|Description|Success|
|---|---|---|
|1|create users|✅|
|2|create courses|✅|
|3|add email|✅|

كل Migration بتتسجل هنا.

---

لما المشروع يشتغل مرة تانية...

Flyway يبص للجدول.

يشوف إن

```
V1
```

اتنفذت.

```
V2
```

اتنفذت.

```
V3
```

اتنفذت.

يبقى مش هيعيد تنفيذهم.

---

لو لقى

```
V4
```

جديدة.

ينفذها فقط.

---

# إزاي Flyway بيشتغل داخليًا؟ (Workflow)

خلينا نمشي خطوة بخطوة من لحظة تشغيل تطبيق Spring Boot:

```text
تشغيل التطبيق
        │
        ▼
Spring Boot يبدأ
        │
        ▼
Flyway يبحث في:
resources/db/migration
        │
        ▼
يقرأ أسماء الملفات (V1, V2, V3...)
        │
        ▼
يرتبها حسب الـ Version
        │
        ▼
يفتح جدول flyway_schema_history
        │
        ▼
هل V1 منفذة؟
      │
   نعم ─────────► تخطاها
      │
      لا
      ▼
نفذ V1
      │
      ▼
سجلها في flyway_schema_history
      │
      ▼
انتقل إلى V2 ثم V3... حتى آخر Migration
```

**النتيجة:** في كل مرة تشغل التطبيق، Flyway ينفذ فقط ما هو جديد، ويحافظ على سجل كامل لكل التغييرات.

---

# أول Migration عملي (SkillHub مثال)

خلينا نفترض إننا بدأنا مشروع جديد.

### الخطوة 1: إنشاء الملف

```
src/main/resources/db/migration/

V1__create_users_table.sql
```

### الخطوة 2: كتابة الـ SQL

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### الخطوة 3: تشغيل التطبيق

عند تشغيل Spring Boot:

- Flyway يكتشف `V1__create_users_table.sql`.
    
- ينفذ أمر `CREATE TABLE`.
    
- ينشئ جدول `flyway_schema_history`.
    
- يسجل أن `V1` تم تنفيذها بنجاح.
    

### الخطوة 4: إضافة Feature جديدة

بعد فترة احتجت تضيف رقم الهاتف للمستخدم.

❌ لا تفتح `V1` وتضيف العمود.

بدلًا من ذلك أنشئ ملفًا جديدًا:

```
V2__add_phone_number_to_users.sql
```

ومحتواه:

```sql
ALTER TABLE users
ADD COLUMN phone_number VARCHAR(20);
```

عند تشغيل التطبيق مرة أخرى:

- `V1` لن تُنفذ مرة أخرى.
    
- `V2` فقط هي التي ستُنفذ.
    
- سيُضاف سجل جديد في `flyway_schema_history`.
    

---

# أفضل الممارسات (Best Practices)

- استخدم **Migration واحدة لكل تغيير منطقي**.
    
- لا تعدّل أي `V` Migration بعد تنفيذها على أي بيئة مشتركة.
    
- اختر أسماء واضحة مثل `add_price_to_courses` بدلًا من `update_table`.
    
- استخدم `ALTER TABLE` في Migration جديدة بدل تعديل القديمة.
    
- استخدم `R__` فقط للأشياء التي يُعاد إنشاؤها مثل **Views** و**Functions** و**Stored Procedures**.
    
- راقب دائمًا جدول `flyway_schema_history` لأنه المرجع الذي يعتمد عليه Flyway لمعرفة ما تم تنفيذه.
    

---

## ربط Part 1 بـ Part 2

في **Part 1** فهمنا **لماذا** نحتاج Flyway، وما مشكلة `hibernate.ddl-auto=update`، وكيف يحافظ Flyway على تاريخ قاعدة البيانات.

في **Part 2** تعلمنا **كيف** ننظم هذا التاريخ عمليًا:

- نسمّي ملفات الـ Migration بطريقة يفهمها Flyway.
    
- نقسم كل تغيير إلى SQL Script مستقل.
    
- نعتمد على Versioning لبناء تاريخ واضح للداتا بيز.
    
- نستخدم Repeatable Migrations للحالات الخاصة.
    
- نفهم دور مجلد `db/migration` وجدول `flyway_schema_history`.
    
- وننشئ أول Migration بطريقة صحيحة.
    

بعد الجزء ده، تقدر تبدأ تستخدم Flyway في مشروع Spring Boot حقيقي بثقة، والجزء القادم هيكون عن السيناريوهات العملية المتقدمة مثل الـ Checksum، والتعامل مع الأخطاء، وإضافة Migrations أثناء العمل ضمن فريق، ودمج Flyway مع بيئات التطوير والإنتاج.