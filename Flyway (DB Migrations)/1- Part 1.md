**Part 1 (النهارده)**
    
    - يعني إيه Database Migration؟
        
    - المشكلة اللي Flyway بيحلها.
        
    - ليه منستخدمش Hibernate Update؟
        
    - فكرة Flyway.
        
    - إزاي Flyway بيشتغل داخليًا.
        
    - أول Setup في Spring Boot.
وده هيكون شرح احترافي مش مجرد Documentation.

---

# أولاً... يعنى إيه Database Migration؟

قبل ما نتكلم عن Flyway لازم نفهم المشكلة.

تخيل عندك مشروع Spring Boot.

أول يوم عملت جدول Users.

```sql
CREATE TABLE users(
    id BIGINT,
    name VARCHAR(100)
);
```

بعد أسبوع العميل قال:

> عايز Email.

فأنت عملت

```sql
ALTER TABLE users
ADD email VARCHAR(255);
```

بعد شهر

قال

> عايز Password.

```sql
ALTER TABLE users
ADD password VARCHAR(255);
```

بعد شهر

قال

> غير اسم العمود name إلى full_name

```sql
ALTER TABLE users
RENAME COLUMN name TO full_name;
```

بعد شهر

قال

> اعمل جدول Orders.

وهكذا...

السؤال...

**إيه اللي حصل للداتابيز؟**

هي اتغيرت.

والتغييرات دي لازم تكون محفوظة.

وده اسمه

> Database Migration : History of All Changes on Db

يعني

> تاريخ كل التغييرات اللي حصلت على قاعدة البيانات.

---

## تخيل السيناريو ده

أنت شغال لوحدك.

مفيش مشكلة.

لكن لو عندك فريق فيه

- علي
    
- أحمد
    
- محمد
    
- عمر
    

كل واحد عنده Database على جهازه.

---

أنت عملت

```sql
ALTER TABLE users
ADD email;
```

أحمد نسى يعملها.

محمد عملها بطريقة مختلفة.

عمر أصلاً غير اسم الجدول.

كل واحد بقى عنده Database مختلفة.

وده كارثة.

---

## الكارثة الأكبر

لو دخل معاكم عضو جديد.

هيعمل Clone للمشروع.

لكن Database فاضية.

هيعرف يبنيها إزاي؟

مين هيقعد يقوله:

- اعمل الجدول ده
    
- وبعدها ضيف العمود ده
    
- وبعدها غير الاسم ده
    
- وبعدها اعمل Index
    
- وبعدها Foreign Key
    

لو نسيت خطوة واحدة...

المشروع مش هيشتغل.

---

# إذن إحنا محتاجين حاجة تحفظ ال History ل الداتابيز

زى ما Git بيحفظ تاريخ الكود.

إحنا محتاجين Git للداتابيز.

وده بالظبط Flyway.

---

# Flyway هو إيه؟

ببساطة جدًا

هو Tool مسؤولة عن

> Version Control للDatabase.

زى Git للكود.

Flyway للداتابيز.

هو بيقول

كل تغيير يحصل للداتابيز...

يتكتب فى ملف SQL.

وأنا هطبقه بالترتيب.

---

## مثال

بدل ما تدخل PostgreSQL وتكتب

```sql
CREATE TABLE users
```

تعمل ملف اسمه

```text
V1__create_users_table.sql
```

وجواه

```sql
CREATE TABLE users(
   id BIGSERIAL PRIMARY KEY,
   full_name VARCHAR(100)
);
```

خلص.

Flyway هيشغله.

---

بعد أسبوع

ملف جديد

```text
V2__add_email.sql
```

```sql
ALTER TABLE users
ADD email VARCHAR(255);
```

بعد أسبوع

```text
V3__create_orders.sql
```

```sql
CREATE TABLE orders(
   id BIGSERIAL PRIMARY KEY
);
```

كل تغيير

يبقى ملف.

مش تعديل في القديم.

---

# ليه منعدلش V1؟

لأن التاريخ لا يتغير.

زى Git Commit.

لما تعمل Commit

بعدين المشروع يكبر

مش بترجع تعدل Commit قديم.

بتعمل Commit جديد.

Flyway نفس الفكرة.

---

# Flyway بيشتغل إزاى؟

أول ما Spring Boot يبدأ.

هيحصل الآتي.

```
Application Start
       │
       ▼
Connect Database
       │
       ▼
Flyway Starts
       │
       ▼
Search Migration Files
       │
       ▼
Check Executed Files
       │
       ▼
Run New Files
       │
       ▼
Start Spring
```

يعنى قبل ما Application تشتغل.

Flyway بيجهز الداتابيز.

---

# طب هو يعرف الملفات اتنفذت ولا لأ إزاى؟

أول مرة يشتغل

بيعمل جدول بنفسه.

اسمه

```text
flyway_schema_history
```

وده أهم جدول في Flyway.

هنشرحه بالتفصيل بعدين.

---

مثلاً

|Version|Description|Success|
|---|---|---|
|1|create users|✅|
|2|add email|✅|
|3|orders|✅|

يعني Flyway بيقول:

أنا نفذت V1.

نفذت V2.

نفذت V3.

---

المرة الجاية.

يشوف الملفات.

```
V1

V2

V3

V4
```

هيقول

V1 موجود.

V2 موجود.

V3 موجود.

يبقى هنفذ V4 فقط.

---

# ليه Flyway أحسن من

```properties
spring.jpa.hibernate.ddl-auto=update
```

دي نقطة مهمة جدًا في الإنترفيو.

## Hibernate Update

هو بيشوف الـ Entity.

ويحاول يعدل الداتابيز.

بس.

---

المشكلة

هل بيعرف

مين عمل التعديل؟

❌ لا.

---

هل فيه History؟

❌ لا.

---

هل يقدر يرجع Version قديم؟

❌ لا.

---

هل يشتغل كويس مع Team؟

❌ لا.

---

هل ينفع Production؟

❌ لا.

---

هل يقدر يعمل Data Migration؟

❌ لا.

---

طيب Flyway؟

✔ Versioned

✔ Team Friendly

✔ Git Friendly

✔ Production Ready

✔ History

✔ Validation

✔ Checksums

✔ Rollback Strategies (بطرق مختلفة)

✔ Repeatable Scripts

---

# مقارنة بسيطة

|Hibernate Update|Flyway|
|---|---|
|يعتمد على الـ Entities|يعتمد على ملفات SQL|
|لا يوجد History|كل تغيير محفوظ|
|غير مناسب للإنتاج غالبًا|مناسب للإنتاج|
|لا يوجد Versioning|يوجد Versioning|
|لا يوجد Validation|يوجد Validation|
|قد يُجري تغييرات غير متوقعة|التغييرات صريحة وواضحة|

> ملاحظة: `ddl-auto=update` قد يكون مناسبًا في مشاريع تجريبية أو أثناء التطوير السريع، لكنه ليس الخيار المعتاد لإدارة تغييرات قاعدة البيانات في بيئات الإنتاج.

---

# دلوقتى هنبدأ الـ Setup

نفترض عندنا مشروع Spring Boot.

## Maven

أضف Dependency

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

لو PostgreSQL

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

---

بعدها

Spring Boot هيكتشف Flyway تلقائيًا.

مش محتاج تعمل Bean.

---

# بعد كده هنضيف Folder

داخل

```
src
 └── main
      └── resources
            └── db
                 └── migration
```

وده مكان مهم جدًا.

Flyway بيدور عليه تلقائيًا.

يعني لازم يكون:

```text
src/main/resources/db/migration
```

وليس مثلًا:

```text
resources/sql
```

ولا

```text
resources/scripts
```

إلا إذا غيّرت إعدادات Flyway يدويًا، وهو ما سنتعلمه لاحقًا.

---

# أول ملف

هيكون بالشكل ده

```text
V1__create_users_table.sql
```

وده ليس اسمًا عشوائيًا، بل جزء من الـ **naming convention** الذي يعتمد عليه Flyway لمعرفة ترتيب التنفيذ.

في الجزء القادم سنفكك هذا الاسم حرفًا حرفًا:

- لماذا يبدأ بـ `V`؟
    
- لماذا الرقم `1`؟
    
- لماذا يوجد شرطتان سفليتان `__`؟
    
- ماذا يحدث إذا كتبت شرطـة واحدة فقط؟
    
- هل يمكن استخدام `V1.1` أو `V2_0`؟
    
- كيف يتعامل Flyway مع الترتيب إذا كان لدينا `V2` و`V10`؟
    
- ما الفرق بين `V` و`R` (Repeatable Migrations)؟
    

وسننشئ أول قاعدة بيانات كاملة خطوة بخطوة باستخدام عدة ملفات Migration ونرى كيف يسجل Flyway كل عملية داخل جدول `flyway_schema_history`.




spring.jpa.hibernate.ddl-auto=validate/none =>validate is better 
عشان لو ضيفت فيلد في الانتيتي ومعملتلوش ميجرشن هيديك ايرور لكن لو العكس وعملت ميجريشن ومضيفتش الفيلد هيشتغل عادي بس hibernate مش هيستخدم الكولمن دا 
V1__create_courses_table.sql 


# في المشاريع الاحترافية

لو سألت أي Senior:

> "مين بينشئ الـ Database؟"

هيقولك:

**Flyway أو Liquibase.**

ولو سألته:

> "مين بيتعامل مع البيانات؟"

هيقولك:

**Hibernate/JPA.** 

## القاعدة الذهبية اللي عايزك تحفظها

| المسؤول           | وظيفته                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Flyway**        | إنشاء الجداول، تعديلها، إضافة الأعمدة، إنشاء الـ Indexes والـ Foreign Keys، وإدارة تاريخ تغييرات قاعدة البيانات (Schema). |
| **JPA/Hibernate** | ربط الـ Entities بالجداول الموجودة بالفعل، وتنفيذ عمليات `SELECT` و`INSERT` و`UPDATE` و`DELETE` عليها.                    |
flyway create tables , jpa deal with data 