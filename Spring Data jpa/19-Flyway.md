## Flyway & DB Migrations — الشرح الكامل

---

## 1. المشكلة — ليه مش بنستخدم `ddl-auto`؟

### معاني الـ `ddl-auto` options

|Option|بيعمل إيه|
|---|---|
|`create`|بيمسح كل الـ tables وبيعملها من الأول عند كل start ❌|
|`create-drop`|بيعمل tables وبيمسحها لما الـ app يقفل ❌|
|`update`|بيعدل الـ tables تلقائياً — خطر على data ❌|
|`validate`|بيتأكد إن الـ Entity بتطابق الـ DB بس ✅|
|`none`|مش بيعمل حاجة ✅|

### المشكلة الأساسية

```
ddl-auto = update
```

لو حذفت column من الـ Entity → الـ column بيفضل في الـ DB لو عندك data مهمة → ممكن يتمسح بالغلط مفيش history لأي تغيير اتعمل

> في الـ Production **بنفصل إدارة الـ DB عن الـ App تماماً**

---

## 2. الحل — DB Migration Tool

### الفكرة الأساسية

```
بدل ما الـ App يتحكم في الـ DB
انت بتكتب SQL scripts بترتيب
والـ Migration Tool بينفذهم بالترتيب
```

```
V1__create_users.sql      ← ينفذ أول مرة
V2__create_courses.sql    ← ينفذ تاني
V3__update_lessons.sql    ← ينفذ تالت
```

---

## 3. Flyway — ازاي بيشتغل؟

### خلف الكواليس

```
App يبدأ
    │
    ▼
Flyway يتفعل
    │
    ▼
بيبص على جدول flyway_schema_history في الـ DB
    │
    ├── مش موجود؟ → بيعمله
    │
    ▼
بيبص على db/migration folder
    │
    ▼
بيقارن الـ scripts الموجودة بالـ history
    │
    ├── script اتنفذ قبل؟ → skip
    │
    └── script جديد؟ → ينفذه
```

### جدول `flyway_schema_history`

|version|description|script|checksum|success|
|---|---|---|---|---|
|1|init database|V1__init.sql|123456|true|
|2|update lessons|V2__update_lessons.sql|789012|true|

---

## 4. الـ File Pattern

```
V{version}__{description}.sql
```

|جزء|القاعدة|مثال|
|---|---|---|
|`V`|حرف كبير دايماً|`V`|
|`{version}`|رقم أو أرقام|`1`, `2`, `3`|
|`__`|underscore مزدوج|`__`|
|`{description}`|وصف بـ underscore|`init_database`|
|`.sql`|امتداد الملف|`.sql`|

### أمثلة صح ✅

```
V1__init_database.sql
V2__create_courses_table.sql
V3__update_lessons_table.sql
V4__add_index_to_users.sql
```

### أمثلة غلط ❌

```
v1__init.sql          ← V صغيرة
V1_init.sql           ← underscore واحد
V1__init              ← من غير .sql
V01__init.sql         ← ممكن يسبب ترتيب غلط
```

---

## 5. الـ Setup خطوة بخطوة

### الـ Dependency

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

### الـ Folder Structure

```
src/
└── main/
    └── resources/
        └── db/
            └── migration/
                ├── V1__init_database.sql
                ├── V2__update_lessons_table.sql
                └── V3__....sql
```

### الـ application.properties

```properties
# وقف الـ ddl-auto
spring.jpa.hibernate.ddl-auto=validate

# Flyway settings
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true
```

### معنى `baseline-on-migrate`

```
لو عندك DB موجودة بالفعل وعايز تبدأ تستخدم Flyway
baseline-on-migrate=true بيقوله:
"اعتبر إن كل حاجة موجودة دلوقتي هي الـ baseline
وابدأ تتابع التغييرات من دلوقتي"
```

---

## 6. الـ Workflow في الـ Production

```
Developer يعدل Entity
        │
        ▼
يكتب SQL migration file
        │
        ▼
يرفعه على Git
        │
        ▼
CI/CD Pipeline
        │
        ▼
App يبدأ في الـ Production
        │
        ▼
Flyway ينفذ الـ migration
        │
        ▼
DB تتحدث بأمان ✅
```

---

## 7. قواعد مهمة جداً ⚠️

```
❌ لا تعدل على migration file اتنفذ قبل كده أبداً
   Flyway بيحسب checksum للـ file
   لو اتغير → هيرمي error عند الـ startup

✅ لو محتاج تعدل → اعمل migration file جديد
   V3__fix_lessons_column.sql
```

---

## 8. ليه Flyway أحسن من `ddl-auto`؟

||`ddl-auto`|Flyway|
|---|---|---|
|التحكم|Hibernate|انت ✅|
|History|❌|✅|
|Rollback|❌|✅|
|Team work|❌|✅|
|Production safe|❌|✅|
|Review قبل التنفيذ|❌|✅|

---

عايز نكمل الـ `LessonController` دلوقتي؟