تمام يا علي. بما إن **Part 1 و Part 2 خلصوا**، فـ Part 3 بقى هو الجزء اللي هيخليك تفهم **Flyway من جوه بجد**، خصوصًا المشاكل اللي بتحصل لما المشروع يكبر أو تشتغل مع Team.

هنمشي بنفس الترتيب:

1. Flyway Schema History
    
2. Checksums
    
3. Validation
    
4. Repair
    
5. Baseline
    
6. Out Of Order
    
7. Ignore Migrations
    

---

# Part 3 — Flyway Advanced Concepts

## 1. Flyway Schema History

دي أهم نقطة في Part 3.

إحنا قلنا قبل كده إن Flyway بيعمل Table اسمه:

```text
flyway_schema_history
```

الجدول ده هو **ذاكرة Flyway**.

يعني Flyway من غيره مش هيعرف بسهولة:

> "أنا نفذت إيه قبل كده؟"

---

## شكله تقريبًا

ممكن تلاقي فيه Columns زي:

```text
installed_rank
version
description
type
script
checksum
installed_by
installed_on
execution_time
success
```

مثلاً:

|version|description|script|checksum|success|
|---|---|---|---|---|
|1|create users|V1__create_users.sql|12345|true|
|2|create courses|V2__create_courses.sql|98231|true|
|3|add email|V3__add_email.sql|55121|true|

---

## Flyway بيستخدم الجدول ده إزاي؟

عندك:

```text
V1__create_users.sql
V2__create_courses.sql
V3__create_courses.sql
```

أول مرة تشغل التطبيق:

```text
Flyway
   ↓
يشوف الملفات
   ↓
يشوف flyway_schema_history
   ↓
V1 مش موجودة
   ↓
ينفذ V1
   ↓
يسجل V1
   ↓
V2 مش موجودة
   ↓
ينفذ V2
   ↓
يسجل V2
```

بعد كده شغلت التطبيق تاني.

Flyway يشوف:

```text
V1 → موجودة
V2 → موجودة
V3 → مش موجودة
```

فيعمل:

```text
Skip V1
Skip V2
Run V3
```

---

# 2. Checksums

دي نقطة مهمة جدًا.

افترض عندك:

```text
V1__create_users.sql
```

ومحتواه:

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

Flyway لما ينفذ الـ Migration بيحسب لها **Checksum**.

اعتبره fingerprint للـ SQL file.

مثلاً بشكل مبسط:

```text
V1
Checksum = 123456
```

ويخزن الرقم في:

```text
flyway_schema_history
```

---

## المشكلة

بعد ما V1 اتنفذت، جيت أنت فتحت الملف:

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

أنت غيرت الـ file.

الـ Checksum بقى مثلًا:

```text
Old Checksum = 123456
New Checksum = 987654
```

Flyway يقول:

> لحظة... الملف ده اتغير!

فيطلع Error.

---

## ليه Flyway بيعمل كده؟

علشان يحميك من إنك تغير التاريخ.

فاكر القاعدة اللي قلناها؟

> **Migration اتنفذت = متعدلهاش.**

الـ Checksum هو اللي بيساعد Flyway يكتشف إنك خالفت القاعدة دي.

---

# مثال عملي

عندك:

```text
V1__create_users.sql
```

اتنفذت.

بعدها اكتشفت إنك محتاج:

```sql
email VARCHAR(255)
```

❌ متعملش:

```text
V1__create_users.sql
```

وتضيفها.

اعمل:

```text
V2__add_email_to_users.sql
```

```sql
ALTER TABLE users
ADD COLUMN email VARCHAR(255);
```

كده التاريخ أصبح:

```text
V1 → create users

V2 → add email
```

وده الصح.

---

# 3. Validation

دلوقتي ندخل على:

**Flyway Validation**

Validation معناها:

> Flyway بيتأكد إن الـ Migrations الموجودة حاليًا في المشروع متوافقة مع اللي اتسجل قبل كده في Database.

يعني بيقارن حاجات زي:

```text
Migration files
        VS
flyway_schema_history
```

---

## مثال

عندك:

```text
V1__create_users.sql
```

اتنفذت.

وفي `flyway_schema_history`:

```text
V1
Checksum = 12345
```

بعدها حد عدل الملف.

بقى:

```text
V1
Checksum = 99999
```

لما Flyway يعمل Validation:

```text
12345 != 99999
```

فيقول:

```text
Validation failed
```

---

# Validation بيحميك من إيه؟

بيحميك من مشكلة خطيرة جدًا:

```text
Database
   ↓
حالتها مبنية على Version قديمة

Code
   ↓
الـ Migration files اتغيرت
```

وده ممكن يعمل:

- اختلاف بين Developers.
    
- اختلاف بين Development وProduction.
    
- مشاكل في Deployment.
    
- Database schema مش متوافق مع الكود.
    

---

# 4. Repair

هنا لازم نفرق جدًا.

**Repair مش معناها: "صلح الـ Database".**

دي غلطة شائعة.

Repair وظيفتها الأساسية إنها **تصلح metadata الخاصة بـ Flyway** في `flyway_schema_history`.

---

## مثال

افترض عندك:

```text
V1
V2
V3
```

وحصل Migration فشلت.

ممكن تلاقي سجل في history فيه:

```text
V3
success = false
```

أو حصل مشكلة في الـ metadata/checksum.

هنا ممكن تستخدم:

```text
flyway repair
```

عشان Flyway يصلح الـ history metadata حسب الحالة.

---

## مهم جدًا

Repair **مش بترجع SQL file القديم**.

ومش معناها:

```text
Repair Database
```

هي أقرب لـ:

```text
Repair Flyway's bookkeeping
```

يعني تصلح **سجل Flyway**.

---

# طيب أستخدم Repair إمتى؟

مثلاً لو عندك:

```text
Validation failed
```

والـ Migration نفسها اتغيرت بشكل مقصود في سيناريو معين، أو عندك metadata تحتاج تنظيف/إصلاح.

لكن:

⚠️ **Repair مش الحل الطبيعي لتعديل Migration قديمة.**

لو أنت غيرت:

```text
V2
```

بالغلط...

مش معنى كده:

```text
flyway repair
```

وخلاص.

الأصل:

```text
V2 لا تتعدل
↓
اعمل V3 جديدة
```

---

# 5. Baseline

دي مهمة جدًا لما تدخل Flyway على **Database موجودة أصلًا**.

تخيل عندك Database Production من سنتين.

وفيها:

```text
users
courses
lessons
enrollments
...
```

لكن المشروع مكنش بيستخدم Flyway.

دلوقتي قررت:

> من النهارده هنستخدم Flyway.

المشكلة:

Flyway هيلاقي Database فيها Tables بالفعل.

لكن:

```text
flyway_schema_history
```

مش موجود.

فـ Flyway ممكن يعتبر إن الـ Database فاضية من ناحية الـ migrations.

---

## هنا Baseline

Baseline معناها تقريبًا:

> "اعتبر الـ Database الحالية نقطة بداية."

مثلاً Database الحالية تعتبر:

```text
Version 5
```

فتعمل Baseline:

```text
Baseline Version = 5
```

Flyway يقول:

> تمام، أنا هعتبر إن كل حاجة لحد Version 5 موجودة بالفعل.

وبعدها تبدأ تعمل:

```text
V6__add_phone.sql
V7__create_reviews.sql
V8__add_indexes.sql
```

---

## مثال

عندك Database موجودة:

```text
users
courses
lessons
```

وأنت بدأت Flyway.

تقول:

```text
Baseline = 3
```

معناها:

```text
V1
V2
V3
```

اعتبرهم موجودين بالفعل.

وبعدها Flyway يبدأ من:

```text
V4
```

---

## إمتى تستخدم Baseline؟

غالبًا لما:

- تدخل Flyway على مشروع قديم.
    
- Database موجودة بالفعل.
    
- مفيش Migration history قديمة.
    

لكن في مشروع جديد من الصفر:

❌ غالبًا مش محتاج Baseline.

لأنك تبدأ عادي:

```text
V1
V2
V3
```

---

# 6. Out Of Order

دي من أهم المشاكل اللي هتقابلك لما تشتغل Team.

تعال نشوف سيناريو حقيقي.

أنت وDeveloper تاني شغالين.

أنت عملت:

```text
V5__add_reviews.sql
```

والـ Developer التاني عمل:

```text
V6__add_categories.sql
```

لكن حصل إن:

```text
V6
```

وصلت للـ Database الأول.

فالـ Database بقت:

```text
V1
V2
V3
V4
V6
```

وبعدين أنت جيت وحاولت تضيف:

```text
V5
```

هنا عندنا Migration قديمة نسبيًا ظهرت بعد Migration أحدث.

ده اسمه:

**Out Of Order Migration**

---

## المشكلة

Flyway الطبيعي بيحب التاريخ يكون:

```text
V1
↓
V2
↓
V3
↓
V4
↓
V5
↓
V6
```

لكن عندنا:

```text
V1
↓
V2
↓
V3
↓
V4
↓
V6
↓
V5
```

---

## Flyway يعمل إيه؟

افتراضيًا Flyway ممكن يرفض تشغيل الـ V5 لأنها جاءت بعد V6.

لكن ممكن تسمح له باستخدام:

```properties
spring.flyway.out-of-order=true
```

ساعتها يقدر يقول:

> آه، V5 متأخرة، لكن أنا هطبقها.

فتصبح:

```text
V1
V2
V3
V4
V6
V5
```

**مهم:** ترتيب التنفيذ الفعلي هنا لا يعني أن رقم `V5` اتنفذ قبل `V6`؛ الـ migration history يحتفظ بمعلومة الترتيب/النسخة، لكن التنفيذ حصل متأخرًا.

---

## هل Out Of Order شيء كويس؟

مش حاجة تستخدمها بشكل عشوائي.

هي مفيدة في بعض سيناريوهات:

- Teams كبيرة.
    
- Branches كثيرة.
    
- Releases متوازية.
    
- Migrations اتأخرت بسبب الـ deployment flow.
    

لكن في مشروع صغير:

```text
V1
V2
V3
V4
```

حاول تحافظ على ترتيب طبيعي.

---

# 7. Ignore Migrations

آخر نقطة:

**Ignore Migrations**

دي معناها:

> قول لـ Flyway يتجاهل بعض الـ migrations بدل ما يعتبر وجودها مشكلة.

لكن لازم نفرق بين:

```text
Ignore
```

و

```text
Skip
```

الفكرة العامة إن Flyway عنده إعدادات للتعامل مع migrations الموجودة في الـ database أو الملفات والتي لا تتوافق مع النطاق المتوقع للتنفيذ، حسب نوع الحالة وإصدار Flyway وإعدادات الـ configuration.

---

## مثال بسيط للفكرة

عندك Database وصلت:

```text
V1
V2
V3
V4
V5
```

لكن الـ application الحالي عنده مجموعة migrations مختلفة أو migration خارج النطاق اللي Flyway بيتوقعه.

بدل ما Flyway يفشل مباشرة، بعض إعدادات **ignoreMigrationPatterns** تسمح بتحديد migrations يتم تجاهلها في عمليات معينة مثل validation أو pending/out-of-order handling.

مثلاً في الإصدارات الحديثة ممكن تشوف configuration بالشكل:

```properties
spring.flyway.ignore-migration-patterns=*:future
```

معنى `*:future` بشكل مبسط:

> تجاهل migrations التي تعتبر مستقبلية بالنسبة للـ database الحالية في السياق الذي ينطبق عليه الـ pattern.

---

# أهم حاجة هنا

متتعاملش مع:

```text
ignore
```

على إنه:

> "أنا عندي مشكلة في Migration، أخلي Flyway يتجاهلها."

لا.

دي Configuration متقدمة لها Use Cases محددة.

لو استخدمتها غلط ممكن تخفي مشكلة حقيقية في الـ Database.

---

# نربط Part 3 كله ببعض

خلينا نعمل سيناريو كامل.

عندك:

```text
V1__create_users.sql
V2__create_courses.sql
V3__add_email.sql
```

Flyway نفذهم.

وفي:

```text
flyway_schema_history
```

عندك:

```text
V1 → checksum A
V2 → checksum B
V3 → checksum C
```

---

## حد عدل V2

بقى:

```text
V2 → checksum X
```

Flyway يعمل:

**Validation**

ويقول:

```text
Expected: B
Found: X
```

❌ Validation Failed.

---

لو التغيير مقصود ومحتاج تصلح الـ metadata في سيناريو مناسب:

```text
Repair
```

لكن في الـ workflow الطبيعي:

```text
Don't modify V2
        ↓
Create V4
```

---

## Database قديمة

لو المشروع بدأ من غير Flyway:

```text
Existing Database
       ↓
Baseline
       ↓
Flyway starts tracking
       ↓
V4
V5
V6
```

---

## Team عندها Migration متأخرة

عندك:

```text
V6
```

وصلت قبل:

```text
V5
```

ده:

```text
Out Of Order
```

وممكن تسمح به حسب احتياج المشروع.

---

# الخلاصة اللي عايزك تحفظها

|Concept|معناه|
|---|---|
|`flyway_schema_history`|ذاكرة Flyway وتاريخ الـ migrations|
|Checksum|Fingerprint للـ migration file|
|Validation|يتأكد إن الـ files متوافقة مع history|
|Repair|يصلح metadata/history الخاصة بـ Flyway|
|Baseline|نقطة بداية لـ Database موجودة بالفعل|
|Out Of Order|Migration قديمة اتنفذت بعد Migration أحدث|
|Ignore|تخلي Flyway يتجاهل migrations معينة وفق patterns/configuration|

وأهم **Rule** في Flyway كله:

```text
Migration اتنفذت
       ↓
❌ متعدلهاش
       ↓
لو عايز تغيير
       ↓
اعمل Migration جديدة
```

يعني لو عندك:

```text
V1__create_users.sql
```

وبعد شهر اكتشفت إنك محتاج تضيف:

```text
phone_number
```

مش:

```text
V1 ← تعدلها
```

لكن:

```text
V2__add_phone_number_to_users.sql
```

وده هو جوهر Flyway: **Database schema بتتطور عن طريق سلسلة تغييرات immutable ومُتتبعة، مش عن طريق تعديل الماضي.**