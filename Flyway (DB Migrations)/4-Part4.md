أكيد. لو قصدك **أشرح Part 3 من الصفر وبهدوء وبأمثلة عملية أكتر**، تعال نمسكه واحدة واحدة، وهنربطه بمشروع Spring Boot بتاعك.

# Part 3 — Flyway Advanced

إحنا لحد دلوقتي عارفين إن عندنا:

```text
src/main/resources/db/migration

V1__create_users.sql
V2__create_courses.sql
V3__add_email.sql
```

وFlyway بينفذهم بالترتيب.

لكن السؤال المهم:

> Flyway بيعرف منين إن V1 و V2 اتنفذوا؟  
> ولو الملف اتغير، هيعرف إزاي؟  
> ولو عندي Database قديمة أعمل إيه؟  
> ولو Migration وصلت متأخرة أعمل إيه؟

هنا Part 3.

---

# 1. Flyway Schema History

دي أهم حاجة في الجزء كله.

لما تشغل Flyway لأول مرة، Flyway بيعمل Table في الـ Database اسمه:

```text
flyway_schema_history
```

اعتبره **دفتر Flyway**.

يعني Flyway بيكتب فيه:

> أنا نفذت Migration رقم كام؟  
> اسمها إيه؟  
> إمتى نفذتها؟  
> نجحت ولا فشلت؟  
> والـ checksum بتاعها إيه؟

---

## مثال

عندك:

```text
V1__create_users.sql
V2__create_courses.sql
V3__add_email_to_users.sql
```

بعد ما تشغل التطبيق، ممكن الـ history يبقى بشكل مبسط:

```text
version | description              | success
------------------------------------------------
1       | create users             | true
2       | create courses           | true
3       | add email to users       | true
```

فلو قفلت التطبيق وشغلته تاني:

Flyway هيقول:

```text
V1 → موجودة في history
V2 → موجودة في history
V3 → موجودة في history
```

إذن:

```text
مش هشغلهم تاني.
```

---

## لو أضفت V4؟

```text
V4__create_reviews.sql
```

تشغل التطبيق.

Flyway يشوف:

```text
V1 → موجودة
V2 → موجودة
V3 → موجودة
V4 → مش موجودة
```

فيعمل:

```text
V4
 ↓
Execute SQL
 ↓
Success
 ↓
Add V4 to flyway_schema_history
```

وده سبب إن Migration بتتنفذ **مرة واحدة**.

---

# 2. Checksums

دي من أهم الحاجات اللي لازم تفهمها.

خلينا نقول عندنا:

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

Flyway لما ينفذ الملف بيحسب له **Checksum**.

فكر فيها كأنها:

> بصمة للـ Migration file.

مثلاً للتوضيح:

```text
V1
Checksum = 12345
```

ويخزنها في:

```text
flyway_schema_history
```

---

## طيب ليه؟

علشان يحميك من إنك تغير Migration قديمة.

مثلاً بعد أسبوع فتحت:

```text
V1__create_users.sql
```

وغيرتها:

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

دلوقتي محتوى الملف اتغير.

وبالتالي:

```text
Old Checksum = 12345

New Checksum = 98765
```

Flyway يقول:

> استنى... أنا فاكر إن V1 كانت بالبصمة 12345، إزاي بقت 98765؟

فيطلع:

```text
Validate failed
Checksum mismatch
```

---

# ليه Flyway بيمنعني أغير V1؟

لأن V1 ممكن تكون اتنفذت بالفعل في:

```text
Developer A
Developer B
Testing
Staging
Production
```

فلو أنت غيرت الملف، هيبقى عندنا:

```text
Production
   ↓
V1 القديمة

Your Git Repository
   ↓
V1 الجديدة
```

وده خراب.

عشان كده القاعدة:

> **Once a Versioned Migration has been applied, don't modify it.**

---

# طب أنا محتاج أضيف email!

اعمل Migration جديدة:

```text
V2__add_email_to_users.sql
```

وجواها:

```sql
ALTER TABLE users
ADD COLUMN email VARCHAR(255);
```

فتاريخك يبقى:

```text
V1 → Create users

V2 → Add email
```

وده هو الأسلوب الصح.

---

# 3. Validation

دلوقتي عندنا:

```text
Migration Files
        ↓
       Flyway
        ↓
flyway_schema_history
```

Flyway بيقارن الـ Migration files الحالية باللي موجود في الـ history.

وده اسمه:

# Validation

يعني:

> هل الـ Migrations اللي عندي دلوقتي هي نفس الـ Migrations اللي اتنفذت قبل كده؟

---

## مثال

History:

```text
V1
Checksum = 100
```

والـ file الحالي:

```text
V1
Checksum = 100
```

تمام:

```text
Validation PASSED
```

لكن لو:

```text
History:
V1 → 100

File:
V1 → 200
```

يبقى:

```text
Validation FAILED
```

---

## ليه ده مفيد؟

تخيل Team.

Developer عمل:

```text
V5__add_status.sql
```

وبعدين حد تاني عدل الملف بالغلط.

Flyway يكتشف المشكلة بدل ما يسكت.

يعني Validation عبارة عن:

> **Safety check**

بتتأكد إن تاريخ الـ Database متوافق مع ملفات الـ Migration.

---

# 4. Repair

دي بقى نقطة ناس كتير بتفهمها غلط.

**Repair مش معناها إن Flyway يصلح الـ Database schema.**

يعني لو عندك:

```text
users table broken
```

مش هتعمل:

```text
flyway repair
```

وتتوقع إن الجدول يتصلح.

لا.

Repair مرتبطة بـ:

```text
flyway_schema_history
```

يعني **metadata/history الخاصة بـ Flyway**.

---

## مثال بسيط

حصل Migration failure أو مشكلة في الـ history metadata.

ممكن تلاقي Migration معينة مسجلة كـ failed.

في بعض السيناريوهات تستخدم:

```text
flyway repair
```

عشان تنظف/تصلح الـ metadata التي يعتمد عليها Flyway.

---

## أهم حاجة تحفظها

```text
Repair
   ↓
Flyway History Metadata
```

مش:

```text
Repair
   ↓
Business Database
```

---

# 5. Baseline

دي مختلفة تمامًا.

افترض إن عندك مشروع قديم جدًا.

Database موجودة بالفعل:

```text
users
courses
lessons
reviews
enrollments
```

لكن المشروع **لم يكن يستخدم Flyway أصلًا**.

دلوقتي قلت:

> من النهارده هنستخدم Flyway.

المشكلة:

Flyway أول ما يشتغل مش عارف إن الـ Tables دي موجودة بالفعل.

هو عايز يعرف:

> أبدأ منين؟

هنا Baseline.

---

## Baseline يعني إيه؟

ببساطة:

> **اعتبر الـ Database الحالية نقطة بداية معروفة.**

مثلاً تقول:

```text
Baseline Version = 5
```

يعني:

> اعتبر إن Database دي بالفعل وصلت للـ version 5.

وبعدها تبدأ تعمل:

```text
V6__add_phone.sql
V7__create_reviews.sql
V8__add_indexes.sql
```

---

## مثال

Database قديمة:

```text
users
courses
lessons
```

أنت قررت تبدأ Flyway.

تقول:

```text
Baseline = 3
```

يبقى Flyway يتعامل معها كأن:

```text
V1 ✅
V2 ✅
V3 ✅
```

موجودين بالفعل.

وبعدها يبدأ يتابع:

```text
V4
V5
V6
...
```

---

## إمتى أحتاج Baseline؟

لما يكون عندك:

```text
Existing Database
+
No Flyway History
```

أما لو بتبدأ مشروع جديد:

```text
Empty Database
+
Flyway
```

غالبًا:

**مش محتاج Baseline.**

تبدأ:

```text
V1
V2
V3
```

---

# 6. Out Of Order

دي بتحصل كتير جدًا مع الـ Teams.

افترض إنك أنت Developer A.

عملت:

```text
V5__add_reviews.sql
```

Developer B عمل:

```text
V6__add_categories.sql
```

لكن الـ V6 وصلت للـ Database الأول.

فبقى عندنا:

```text
V1
V2
V3
V4
V6
```

وبعدين V5 وصلت.

يعني:

```text
V5
```

جاءت **بعد V6**.

ده اسمه:

# Out Of Order Migration

---

## ليه حصل؟

مش لأن Flyway غلط.

غالبًا بسبب:

```text
Git branches
Parallel development
Different deployments
```

---

## Flyway يقول إيه؟

هو شايف:

```text
Latest applied = V6
```

وفجأة لقى:

```text
V5
```

فيقول:

> V5 أقدم من آخر Migration عندي، ليه ظهرت دلوقتي؟

افتراضيًا ممكن يرفض تشغيلها.

---

## ممكن تسمح بده

في Spring Boot:

```properties
spring.flyway.out-of-order=true
```

ساعتها Flyway يسمح بتنفيذ Migration متأخرة.

---

## هل أستخدمها دائمًا؟

لا.

هي feature للحالات اللي فعلًا محتاجها.

لأن الأفضل أصلًا إن الـ Migration history تكون:

```text
V1
V2
V3
V4
V5
V6
```

مش:

```text
V1
V2
V3
V4
V6
V5
```

---

# 7. Ignore Migrations

آخر واحدة.

Flyway عنده configuration اسمها:

```properties
spring.flyway.ignore-migration-patterns=...
```

الفكرة:

> في حالات معينة، بدل ما Flyway يعتبر Migration معينة مشكلة، تقول له تجاهلها في العمليات التي ينطبق عليها الـ pattern.

---

## مثال

ممكن تستخدم Pattern مثل:

```properties
spring.flyway.ignore-migration-patterns=*:future
```

الفكرة هنا إنك بتقول لـ Flyway:

> لو فيه Migration تعتبر Future بالنسبة للـ Database الحالية، متعتبرهاش مشكلة في السياق اللي الـ pattern بيطبّق عليه.

---

## بس خد بالك

دي **مش طريقة لإخفاء المشاكل**.

يعني لو عندك:

```text
V5 failed
```

مينفعش تقول:

> خلاص Ignore.

لأنك كده ممكن تخلي الـ Application يشتغل فوق Database ناقصة.

---

# خلينا نعمل Scenario كامل

ودي أهم حاجة عشان كل الكلام يثبت.

عندك:

```text
V1__create_users.sql
V2__create_courses.sql
V3__create_lessons.sql
```

شغلت المشروع.

Flyway عمل:

```text
flyway_schema_history
```

وسجل:

```text
V1 ✓
V2 ✓
V3 ✓
```

---

## بعدها غيرت V2

الـ Checksum اتغير.

Flyway يعمل:

```text
Validation
     ↓
Checksum mismatch
     ↓
❌ Error
```

الحل الطبيعي:

```text
❌ متعدلش V2

✅ اعمل V4
```

مثلاً:

```text
V4__add_duration_to_courses.sql
```

---

## Database قديمة

لو بدأت Flyway على Database موجودة أصلًا:

```text
Existing DB
```

تستخدم:

```text
Baseline
```

وتقول:

```text
Current DB = Version 3
```

ثم تبدأ:

```text
V4
V5
V6
```

---

## Team مشكلة

Developer B نشر:

```text
V6
```

قبل:

```text
V5
```

وصلنا:

```text
V1
V2
V3
V4
V6
```

وبعدين V5 وصلت.

دي:

```text
Out Of Order
```

ولو السيناريو محتاج:

```properties
spring.flyway.out-of-order=true
```

---

# الخريطة اللي عايزك تحفظها

```text
                  Flyway
                     │
                     ▼
          flyway_schema_history
                     │
        ┌────────────┼─────────────┐
        │            │             │
        ▼            ▼             ▼
    Version       Checksum      Success
    Migration     بصمة الملف    نجحت؟
```

والـ Advanced Concepts:

```text
Schema History
      │
      ├── Checksums
      │      ↓
      │   هل الملف اتغير؟
      │
      ├── Validation
      │      ↓
      │   هل الـ files متوافقة
      │   مع الـ history؟
      │
      ├── Repair
      │      ↓
      │   إصلاح Flyway metadata
      │
      ├── Baseline
      │      ↓
      │   Database قديمة
      │
      ├── Out Of Order
      │      ↓
      │   Migration وصلت متأخرة
      │
      └── Ignore
             ↓
          تجاهل حالات
          محددة بالـ pattern
```

## وأهم 4 قواعد في Part 3

**1. Migration اتنفذت → متعدلهاش.**

```text
V3 ❌ تعديل
V4 ✅ Migration جديدة
```

**2. Checksum = بصمة الـ Migration.**

**3. `flyway_schema_history` = ذاكرة Flyway.**

**4. Repair لا يصلح الـ Database نفسها؛ هو متعلق بـ Flyway metadata/history.**

لو فهمت الأربع نقط دول، فأنت فاهم الجزء الأكبر من الـ **Flyway internals** اللي محتاجه في Spring Boot.