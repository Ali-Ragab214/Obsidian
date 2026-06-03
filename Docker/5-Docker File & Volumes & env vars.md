# Docker Deep Dive — Production Onboarding Session

---

## قبل ما نبدأ — فلسفة Docker في Production

لازم تفهم حاجة مهمة قبل أي أمر:

> Docker مش بس "بتشغل app في container" — هو بيحل مشكلة **"بيشتغل على جهازي بس"** على مستوى الـ production.

---

# 📦 Part 1: Docker File System

---

## ما هو الـ Container Filesystem؟

لما بتشغل container، Docker بيعمله filesystem معزول تماماً عن الـ host machine. ده بيتعمل عن طريق حاجة اسمها **Union File System (UnionFS)**.

```
┌─────────────────────────────────────┐
│           Container Layer           │  ← Read/Write (بيتحذف مع الـ container)
├─────────────────────────────────────┤
│           Image Layer 3             │  ← Read Only
├─────────────────────────────────────┤
│           Image Layer 2             │  ← Read Only
├─────────────────────────────────────┤
│           Image Layer 1             │  ← Read Only (Base OS)
└─────────────────────────────────────┘
```

يعني الـ Image هي layers فوق بعض كلها Read Only، والـ container بيضيف layer فوقيهم هو بس اللي بيتكتب فيه.

---

## Container Filesystem vs Host Filesystem

```
HOST MACHINE                    CONTAINER
─────────────────               ─────────────────
/home/ali/                      /app/
/home/ali/projects/             /app/myapp.jar
/etc/                           /etc/
/var/                           /var/
/opt/                           /tmp/

   ❌ الـ container مش شايف الـ host
   ❌ الـ host مش شايف الـ container
   ✅ إلا لو عملت mount صريح
```

---

## Ephemeral vs Persistent Data

```
┌─────────────────────────────────────────────┐
│  docker stop mycontainer                    │
│  docker rm mycontainer                      │
│                                             │
│  ❌ كل اللي كان جوه الـ container اتحذف     │
│  ✅ الـ Volumes لسه موجودة                  │
└─────────────────────────────────────────────┘
```

**Ephemeral** = الـ container layer — بيتحذف مع الـ container **Persistent** = الـ Volumes — بتفضل موجودة حتى لو الـ container اتحذف

---

## Bind Mounts vs Volumes

```
BIND MOUNT                          VOLUME
──────────────────                  ──────────────────
Host Path → Container Path          Docker manages it
/home/ali/logs → /app/logs          myvolume → /app/data

✅ انت بتتحكم في المكان            ✅ Docker بيتحكم في المكان
✅ كويس للـ development             ✅ كويس للـ production
❌ path مرتبط بالـ host             ❌ مش بتشوف الـ data بسهولة
```

---

## أوامر عملية

```bash
# شوف الـ filesystem جوه الـ container
docker exec -it mycontainer sh
ls /app

# انسخ ملف من الـ container للـ host
docker cp mycontainer:/app/logs/app.log ./logs/

# انسخ ملف من الـ host للـ container
docker cp ./config.yml mycontainer:/app/config/
```

---

## ⚠️ Common Mistakes

```bash
# ❌ غلط — بتكتب data جوه الـ container وهتتحذف
docker run myapp
# كل الـ logs والـ data هتتحذف مع الـ container

# ✅ صح — استخدم volume للـ data المهمة
docker run -v myvolume:/app/logs myapp
```

---

## 📝 Summary — File System

- الـ Container عنده filesystem معزول بالكامل
- الـ Image عبارة عن Read-Only layers
- الـ Container layer بيتحذف مع الـ container
- Bind Mount = انت بتحدد الـ path على الـ host
- Volume = Docker بيتحكم في الـ storage

## 🎤 Interview Questions

1. إيه الفرق بين Bind Mount وVolume؟
2. لو اتحذف الـ container، الـ data بتروح فين؟
3. إيه هو الـ Union File System؟

## 🏭 Production Insight

> في الـ production، **لا تستخدم Bind Mounts أبداً** للـ data المهمة. الـ Volumes أكثر أماناً وأسهل في الـ backup والـ migration.

---

---

# 💾 Part 2: Docker Volumes

---

## ما هو الـ Volume؟

الـ Volume هو storage موجود على الـ host machine بس Docker هو اللي بيديره، ومش مرتبط بـ container معين.

```
HOST MACHINE
┌──────────────────────────────────────────┐
│  /var/lib/docker/volumes/                │
│  ├── myvolume/                           │
│  │   └── _data/                         │
│  │       ├── postgresql/                 │
│  │       └── app.db                     │
│  └── logs_volume/                        │
│      └── _data/                         │
│          └── app.log                    │
└──────────────────────────────────────────┘
          ↕ mount
┌──────────────────────────────────────────┐
│  CONTAINER                               │
│  /var/lib/postgresql/data  ←── myvolume  │
│  /app/logs                 ←── logs_vol  │
└──────────────────────────────────────────┘
```

---

## أوامر الـ Volumes

```bash
# إنشاء volume
docker volume create myvolume

# قائمة الـ volumes
docker volume ls

# تفاصيل الـ volume
docker volume inspect myvolume

# حذف volume
docker volume rm myvolume

# حذف كل الـ volumes غير المستخدمة
docker volume prune
```

---

## مثال 1: Spring Boot App في Docker

### الـ Dockerfile

```dockerfile
# Base Image
FROM eclipse-temurin:17-jre-alpine

# مجلد الشغل جوه الـ container
WORKDIR /app

# انسخ الـ jar
COPY target/myapp.jar app.jar

# مجلد للـ logs
RUN mkdir -p /app/logs /app/config

# الـ port
EXPOSE 8080

# تشغيل الـ app
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### بناء وتشغيل

```bash
# Build الـ image
docker build -t myspringapp:1.0 .

# تشغيل مع volume للـ logs
docker volume create spring_logs

docker run -d \
  --name myapp \
  -p 8080:8080 \
  -v spring_logs:/app/logs \
  myspringapp:1.0
```

### هيكل الـ folders

```
CONTAINER                    HOST (Volume)
/app/                        /var/lib/docker/volumes/spring_logs/_data/
├── app.jar                  └── app.log
├── logs/          ←──────────────┘
│   └── app.log
└── config/
```

---

## مثال 2: PostgreSQL مع Volume

```bash
# إنشاء volume للـ database
docker volume create postgres_data

# تشغيل PostgreSQL
docker run -d \
  --name mypostgres \
  -e POSTGRES_DB=mydb \
  -e POSTGRES_USER=ali \
  -e POSTGRES_PASSWORD=secret123 \
  -v postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# تأكد إن الـ data اتحفظت
docker volume inspect postgres_data
```

```
Volume Mapping:
──────────────────────────────────────────────
postgres_data (Docker Volume)
        ↕
/var/lib/postgresql/data (inside container)
        ↕
/var/lib/docker/volumes/postgres_data/_data (on host)
──────────────────────────────────────────────
لو الـ container اتحذف → الـ data لسه موجودة في الـ volume
```

---

## مثال 3: Docker Compose — Spring Boot + PostgreSQL

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: mypostgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: ali
      POSTGRES_PASSWORD: secret123
    ports:
      - "5432:5432"

  springapp:
    build: .
    container_name: myapp
    depends_on:
      - postgres
    volumes:
      - spring_logs:/app/logs
      - ./config:/app/config   # bind mount للـ config
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/mydb
      SPRING_DATASOURCE_USERNAME: ali
      SPRING_DATASOURCE_PASSWORD: secret123

volumes:
  postgres_data:
  spring_logs:
```

---

## Volume Permissions — مشكلة شائعة في Production

```bash
# ❌ مشكلة — الـ container بيشتغل بـ root لكن الـ volume مش بتاعه
docker run -v myvolume:/app/data myapp
# Error: Permission denied

# ✅ حل 1 — حدد الـ user في الـ Dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser

# ✅ حل 2 — حدد الـ permissions في الـ run
docker run -v myvolume:/app/data --user 1000:1000 myapp
```

---

## 📝 Summary — Volumes

- الـ Volume بيفضل موجود حتى لو الـ container اتحذف
- Docker بيحفظه في `/var/lib/docker/volumes/`
- استخدم Volumes للـ database data والـ logs
- استخدم Bind Mounts للـ config files في الـ development

## 🎤 Interview Questions

1. إيه الفرق بين Volume وBind Mount في الـ production؟
2. لو عندك PostgreSQL في container وعايز تعمل backup، إزاي؟
3. إزاي تحل مشكلة الـ permissions في الـ volumes؟

## 🏭 Production Insight

> في الـ production دايماً اعمل **named volumes** مش anonymous volumes. الـ anonymous volumes بتبقى صعبة تتعرف عليها وتعمل لها backup.

```bash
# ❌ Anonymous Volume — صعب تتابعه
docker run -v /app/data myapp

# ✅ Named Volume — واضح وسهل
docker run -v myapp_data:/app/data myapp
```

---

---

# 🔐 Part 3: Environment Variables

---

## ليه بنستخدم Environment Variables؟

```
❌ المشكلة بدونها:
──────────────────
# الـ credentials hardcoded في الكود
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.password=secret123
# لو commit الكود ده على GitHub = كارثة أمنية

✅ الحل:
──────────
# الـ credentials في environment variables
spring.datasource.url=${DB_URL}
spring.datasource.password=${DB_PASSWORD}
# الـ values بتيجي من الـ environment مش من الكود
```

---

## طرق تمرير الـ Environment Variables

### 1. في الـ `docker run`

```bash
docker run -d \
  --name myapp \
  -e DB_URL=jdbc:postgresql://postgres:5432/mydb \
  -e DB_USERNAME=ali \
  -e DB_PASSWORD=secret123 \
  -e APP_ENV=production \
  -e SERVER_PORT=8080 \
  myspringapp:1.0
```

### 2. عن طريق ملف `.env`

```bash
# ملف .env
DB_URL=jdbc:postgresql://postgres:5432/mydb
DB_USERNAME=ali
DB_PASSWORD=secret123
APP_ENV=production
SERVER_PORT=8080
```

```bash
# استخدام الـ .env file
docker run --env-file .env myspringapp:1.0
```

### 3. في الـ Docker Compose

```yaml
version: '3.8'

services:
  springapp:
    build: .
    environment:
      # طريقة 1: مباشرة
      APP_ENV: production
      SERVER_PORT: 8080

      # طريقة 2: من الـ .env file
      DB_URL: ${DB_URL}
      DB_PASSWORD: ${DB_PASSWORD}

    env_file:
      - .env  # طريقة 3: ملف كامل
```

---

## كيف Spring Boot بيقرأها؟

```
Environment Variable          application.properties
────────────────────          ──────────────────────
DB_URL              ────→     ${DB_URL}
DB_USERNAME         ────→     ${DB_USERNAME}
DB_PASSWORD         ────→     ${DB_PASSWORD}
SERVER_PORT         ────→     ${SERVER_PORT:8080}
                               ↑ default value لو مش موجود
```

```properties
# application.properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
server.port=${SERVER_PORT:8080}

# أو بنمط Spring Boot المباشر
# SPRING_DATASOURCE_URL بتتحول تلقائياً لـ spring.datasource.url
```

---

## مثال كامل: Spring Boot + PostgreSQL + Env Variables

### هيكل المشروع

```
myproject/
├── src/
├── target/
│   └── myapp.jar
├── Dockerfile
├── docker-compose.yml
├── .env                  ← هنا الـ secrets
├── .env.example          ← template بدون values حقيقية
└── .gitignore            ← لازم يشمل .env
```

### ملف `.env`

```bash
# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=mydb
DB_USERNAME=ali
DB_PASSWORD=SuperSecret123!

# App
APP_ENV=production
APP_PORT=8080
JWT_SECRET=myJwtSecretKey
```

### ملف `.env.example`

```bash
# Database
DB_HOST=
DB_PORT=5432
DB_NAME=
DB_USERNAME=
DB_PASSWORD=

# App
APP_ENV=development
APP_PORT=8080
JWT_SECRET=
```

### `application.properties`

```properties
spring.datasource.url=jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
server.port=${APP_PORT:8080}
app.jwt.secret=${JWT_SECRET}
spring.profiles.active=${APP_ENV:development}
```

### `docker-compose.yml` الكامل

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_db
    env_file: .env
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME}"]
      interval: 10s
      timeout: 5s
      retries: 5

  springapp:
    build: .
    container_name: spring_app
    env_file: .env
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - spring_logs:/app/logs
    ports:
      - "${APP_PORT}:8080"

volumes:
  postgres_data:
  spring_logs:
```

---

## تأكد من الـ Environment Variables

```bash
# شوف الـ env variables جوه الـ container
docker exec -it spring_app env

# ابحث عن variable معين
docker exec -it spring_app env | grep DB

# inspect الـ container وشوف الـ env
docker inspect spring_app | grep -A 20 "Env"
```

---

## 🔒 Production Security Rules

```bash
# ❌ كارثة — لا تعمل كده أبداً
docker run -e DB_PASSWORD=secret123 myapp
# الـ password ظاهر في الـ shell history

# ✅ استخدم .env file
docker run --env-file .env myapp

# ✅ في الـ CI/CD استخدم secrets
# GitHub Actions:
# ${{ secrets.DB_PASSWORD }}

# ✅ في الـ production الحقيقي استخدم
# - Docker Secrets
# - Kubernetes Secrets
# - AWS Secrets Manager
# - HashiCorp Vault
```

---

# 🔧 Troubleshooting Section

---

## مشكلة 1: Data بتتحذف بعد الـ restart

```bash
# السبب: مفيش volume
docker run mypostgres  # ❌

# الحل: استخدم volume
docker run -v postgres_data:/var/lib/postgresql/data mypostgres  # ✅
```

## مشكلة 2: Env Variables مش شغالة

```bash
# تأكد إنها وصلت للـ container
docker exec -it myapp env | grep MY_VAR

# لو مش موجودة، تأكد من الـ docker-compose.yml
# أو إن الـ .env file موجود في نفس directory الـ compose
```

## مشكلة 3: Volume Permission Denied

```bash
# شوف الـ permissions
docker exec -it myapp ls -la /app/data

# الحل في الـ Dockerfile
RUN mkdir -p /app/data && chmod 755 /app/data
USER 1000:1000
```

## مشكلة 4: Spring Boot مش بتقرأ الـ Variables

```bash
# تأكد من الـ naming convention
# Docker: DB_URL
# Spring: spring.datasource.url=${DB_URL}

# أو استخدم Spring Boot naming مباشرة
# SPRING_DATASOURCE_URL=jdbc:postgresql://...
# Spring Boot بيتعرف عليها تلقائياً
```

---

# 📋 الملخص العام

```
┌─────────────────────────────────────────────────┐
│              Docker Storage Strategy             │
├─────────────────┬───────────────────────────────┤
│ Database Data   │ Named Volume                  │
│ App Logs        │ Named Volume                  │
│ Config Files    │ Bind Mount (dev only)          │
│ Secrets         │ Docker Secrets / Vault         │
│ Temp Files      │ Container Layer (ephemeral)    │
└─────────────────┴───────────────────────────────┘
```

---

# 🎤 Interview Questions الشاملة

1. إيه الفرق بين Volume وBind Mount؟
2. إزاي بتحمي الـ secrets في الـ production؟
3. لو الـ database container اتحذف، الـ data بتروح فين؟
4. إزاي Spring Boot بتقرأ الـ environment variables؟
5. إيه الفرق بين `COPY` و`ADD` في الـ Dockerfile؟
6. إزاي تعمل healthcheck للـ database قبل ما الـ app تشتغل؟

---

# 🏭 Final Production Insights

> **القاعدة الذهبية:** الـ container لازم يكون **stateless** — كل الـ state يروح في volumes أو external services.

> **Security:** الـ `.env` file لازم يكون في الـ `.gitignore` دايماً بدون استثناء.

> **Naming:** دايماً سمّي الـ volumes بأسماء واضحة — `projectname_postgres_data` مش بس `data`.