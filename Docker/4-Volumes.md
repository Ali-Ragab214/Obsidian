## Docker Volume — الشرح الكامل

---

## 1. المشكلة الأول

لما تشغل container وتقفله:

```
docker-compose down
```

```
┌─────────────────┐
│   Container     │  ← اتمسح
│                 │
│   DB Data  💾   │  ← اتمسح معاه ❌
└─────────────────┘
```

يعني كل الـ data اللي في الـ PostgreSQL **راحت!**

---

## 2. الحل — Volume

```
┌─────────────────┐         ┌──────────────┐
│   Container     │ ──────► │    Volume    │
│                 │         │              │
│   DB Data  💾   │         │  data 💾 💾  │  ← على الـ machine الحقيقية
└─────────────────┘         └──────────────┘
     اتمسح ❌                  فاضل ✅
```

الـ Volume بيخزن الـ data **بره الـ container** على الـ machine الحقيقية.

---

## 3. أنواع الـ Storage في Docker

### النوع الأول — Container Layer (بدون volume)

```
┌─────────────────────┐
│     Container       │
│  ┌───────────────┐  │
│  │   Data  💾    │  │  ← جوه الـ container
│  └───────────────┘  │    لما يتمسح → Data راحت ❌
└─────────────────────┘
```

### النوع التاني — Volume ✅

```
┌─────────────────────┐      ┌─────────────────┐
│     Container       │      │  Docker Volume  │
│                     │ ───► │                 │
│   يكتب/يقرأ Data   │       │   Data  💾 💾   │ 
└─────────────────────┘      └─────────────────┘
   لو اتمسح → Volume فاضل ✅
```

### النوع التالت — Bind Mount

```
┌─────────────────────┐      ┌─────────────────┐
│     Container       │      │  Folder على     │
│                     │ ───► │  جهازك          │
│                     │      │  /home/ali/data  │
└─────────────────────┘      └─────────────────┘
```

---

## 4. الفرق بينهم

# 📊 مقارنة بين Container Layer vs Volume vs Bind Mount

| الخاصية                              | Container Layer            | Volume                                    | Bind Mount                          |
| ------------------------------------ | -------------------------- | ----------------------------------------- | ----------------------------------- |
| هل البيانات بتتمسح مع الـ container؟ | ❌ نعم                      | ❌ لا                                      | ❌ لا                                |
| مين اللي بيدير البيانات؟             | Docker مؤقتًا              | Docker                                    | أنت (Host Machine)                  |
| تحكمك في البيانات                    | ❌ لا                       | محدود                                     | ✅ كامل                              |
| الاستخدام الأساسي                    | بيانات مؤقتة أثناء التشغيل | تخزين بيانات قواعد البيانات و persistence | تطوير (Dev) ومزامنة الكود مع الجهاز |
| الأفضل في                            | تجارب سريعة / caching      | Databases (MySQL, Postgres)               | Live development / source code      |

---


## 5. ازاي تنشئ Volume؟

### الطريقة الأولى — في `docker-compose.yml` (الأشيع)

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      #   ↑                ↑
      #   اسم الـ Volume   المكان جوه الـ container

# هنا بتعرّف الـ volumes
volumes:
  postgres_data:    # Docker بيعمله تلقائياً 
```

```YAML
services:
  db:
     image:postgres:16-alpine
     volumes:
       - postgres_data:/var/lib/postgresql/data
         
volumes:
  postgres_data:
```
### الطريقة التانية — Command Line

```bash
# عمل volume
docker volume create postgres_data

# شوف كل الـ volumes
docker volume ls

# تفاصيل volume معين
docker volume inspect postgres_data

# مسح volume
docker volume rm postgres_data

# مسح كل volumes مش بتستخدمها
docker volume prune
```


```
docker volume create postgres_data 
docker volume inspect postgres_data 
docker volume rm postgres_data 
docker volume prune 
```
---

## 6. خلف الكواليس

```
docker-compose up
        │
        ▼
Docker بيدور على volume اسمه postgres_data
        │
        ├── موجود؟ → يستخدمه ✅
        │
        └── مش موجود؟ → يعمله تلقائياً ✅
        │
        ▼
الـ container بيكتب data في /var/lib/postgresql/data
        │
        ▼
Docker بيعمل sync تلقائي مع الـ volume
        │
        ▼
docker-compose down
        │
        ▼
الـ container اتمسح
الـ volume فاضل على جهازك ✅
        │
        ▼
docker-compose up تاني
        │
        ▼
نفس الـ data موجودة ✅
```

---

## 7. في مشروعك بالظبط

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    container_name: skillhub_db
    environment:
      POSTGRES_DB: learning_academy
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      # Volume للـ Data عشان ما تتمسحش
      - postgres_data:/var/lib/postgresql/data

  app:
    build: .
    container_name: skillhub_app
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/learning_academy
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
    depends_on:
      - db

volumes:
  postgres_data:  # ✅ Data هتفضل حتى لو الـ container اتمسح
```

---

## 8. امتى تستخدم Volume؟

|الحالة|Volume؟|
|---|---|
|PostgreSQL / MySQL data|✅ دايماً|
|Redis cache|✅ لو عايز تحتفظ بالـ cache|
|Uploaded files|✅ دايماً|
|Logs|✅ لو محتاج تراجعها|
|Source code في dev|Bind Mount أحسن|

---

## 9. الأوامر المهمة

```bash
# شوف الـ volumes الموجودة
docker volume ls

# شوف تفاصيل volume ومكانه على جهازك
docker volume inspect postgres_data

# امسح الـ containers بس — Volume بيفضل ✅
docker-compose down

# امسح الـ containers والـ volumes ❌ بتمسح الـ data
docker-compose down -v
```

---

## الخلاصة بجملة واحدة

> الـ Volume هو **USB flash drive** بتحطه في الـ container — لما تنزع الـ container الـ data بتفضل على الـ USB ✅