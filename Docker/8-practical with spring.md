تمام يا علي، خلينا نمشيها كأنك أول مرة تسمع عن Maven و JAR و Docker.

# 1. أنت بتكتب إيه؟

أنت بتكتب:

```java
@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello Ali";
    }
}
```

ده مجرد **Source Code**.

```text
src/
 └── main/
      └── java/
```

السورس لوحده مش بيتشغل.

---

# 2. Java بتعمل Compile

لما تضغط Run في IntelliJ أو تعمل:

```bash
./mvnw compile
```

Java بتحول:

```text
HelloController.java
```

إلى:

```text
HelloController.class
```

وتحطهم هنا:

```text
target/classes
```

وده المجلد اللي أنت شايفه في الصورة.

---

# 3. يعني إيه Maven؟

Maven أداة Build.

مسؤوليتها:

- تنزيل Dependencies
    
- Compile
    
- Test
    
- Package
    

من خلال:

```xml
pom.xml
```

---

# 4. يعني إيه Package؟

Package يعني:

> اجمعلي كل حاجة في ملف واحد جاهز للتشغيل.

الأمر:

```bash
./mvnw package
```

أو:

```bash
./mvnw clean package
```

---

بعد التنفيذ هتلاقي:

```text
target/
 └── docker-test-0.0.1-SNAPSHOT.jar
```

---

# 5. يعني إيه JAR؟

JAR = Java Archive

ملف مضغوط يحتوي على:

```text
.class files
dependencies
resources
application.properties
```

يعني تطبيق كامل.

---

# 6. أجرب الـ JAR

قبل Docker لازم أتأكد إنه شغال.

```bash
java -jar target/docker-test-0.0.1-SNAPSHOT.jar
```

هيشغل:

```text
Spring Boot
Tomcat
Port 8080
```

وكأنك ضغطت Run من IntelliJ.
![[Pasted image 20260620114947.png]]





---
---
---
---
----
----
دي طريقه غير Docker file 
flow 
تمام، ده الفلو الكامل بالترتيب من الصفر:

## 1. بناء الـ jar من مشروع Spring Boot

```bash
mvn clean package
```

ده بيعمل الـ jar في `target/docker-test-0.0.1-SNAPSHOT.jar`

## 2. عمل container من image جاهزة (فيها JDK)

```bash
docker run -dit --name java-container openjdk:22-jdk
```

- `-d` → يشغل في الخلفية
- `-i -t` → يفضل تفاعلي وما يقفلش
- `--name` → تسمية الكونتينر عشان تشاور عليه بسهولة

تأكد إنه شغال:

```bash
docker ps
```

## 3. نقل الـ jar من جهازك للكونتينر

```bash
docker cp target/docker-test-0.0.1-SNAPSHOT.jar java-container:/tmp/docker-test.jar
```

لاحظ هنا غيرت الاسم لـ `docker-test.jar` وقت النقل عشان يبقى أبسط لاحقًا.

تأكد إنه دخل صحيح:

```bash
docker exec java-container ls /tmp
```

## 4. تحويل الكونتينر لـ image جديدة (commit)

```bash
docker commit --change='CMD ["java", "-jar", "/tmp/docker-test.jar"]' java-container alidocker:v1
```

- ده بياخد snapshot من الكونتينر (بكل اللي فيه دلوقتي، يعني الـ jar اللي نقلته)
- و بيضبط الأمر الافتراضي (`CMD`) عشان يشغل الـ jar أوتوماتيك

## 5. تشغيل كونتينر جديد من الـ image دي

```bash
docker run alidocker:v1
```

هيشغل تلقائيًا:

```
java -jar /tmp/docker-test.jar
```

## 6. (اختياري) تشوف الـ logs لو شغلته بـ `-d`

```bash
docker run -d --name app-final alidocker:v1
docker logs app-final
```

---

**خلاصة الفلو:**

```
mvn package → docker run (container فاضي) → docker cp (تنزيل jar) → docker commit (تحويل container لـ image + ضبط CMD) → docker run (تشغيل الـ image الجديدة)
```

واضح كده؟ لو في خطوة لسه مش واضحة قولي عليها بالتحديد.

![[Pasted image 20260620130106.png]]
---
![[Pasted image 20260620130748.png]]
---
---
---
---
---
---
----
---
---
---

**يعني خلاصة اللي عملته:** عملت image فيه الـ jar، وضبطت الـ image عشان يشغل الـ jar تلقائيًا أول ما يتعمله run، وشغلت كونتينر منه.

تتأكد إن الـ jar شغال صح بـ:

```bash
docker logs 06132a17cb6a
```

عايز تشوف الـ output؟

---

# 7. طب ليه Docker؟

افترض إن السيرفر مفيهوش IntelliJ.

ولا فيه Source Code.

ولا فيه Maven.

أنا عايز أبعتله حاجة جاهزة.

يبقى أبعت:

```text
docker-test.jar
```

---

  هنحط بقا ال jar دا في ال  container 
# 8. Docker Image

بنكتب:

```dockerfile
FROM eclipse-temurin:21-jre

COPY target/docker-test-0.0.1-SNAPSHOT.jar app.jar

ENTRYPOINT ["java","-jar","app.jar"]
```

---

معنى السطور:

### نزل Java

```dockerfile
FROM eclipse-temurin:21-jre
```

---

### انسخ الـ JAR

```dockerfile
COPY target/docker-test-0.0.1-SNAPSHOT.jar app.jar
```

---

### شغله

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```

---

# 9. ابني Image

```bash
docker build -t docker-test .
```

هيعمل:

```text
Docker Image
```

---

شوفها:

```bash
docker images
```

---

# 10. شغل Container

```bash
docker run -p 8080:8080 docker-test
```

معناها:

```text
جهازك
8080
  |
  V
Container
8080
```

---

# 11. افتح المتصفح

```text
http://localhost:8080
```

هتشوف:

```text
Hello Ali
```

---

# التسلسل الكامل

```text
كتبت Java Code
       |
       V
./mvnw compile
       |
       V
.class files
       |
       V
./mvnw package
       |
       V
docker-test.jar
       |
       V
docker build
       |
       V
Docker Image
       |
       V
docker run
       |
       V
Container
       |
       V
Spring Boot Running
```

# أهم أوامر تحفظها

### Build JAR

```bash
./mvnw clean package
```

### تشغيل JAR

```bash
java -jar target/*.jar
```

### بناء Docker Image

```bash
docker build -t docker-test .
```

### تشغيل Container

```bash
docker run -p 8080:8080 docker-test
```

### مشاهدة الصور

```bash
docker images
```

### مشاهدة الـ Containers

```bash
docker ps
```

لو فهمت السلسلة:

**Source Code → JAR → Docker Image → Container**

يبقى فهمت 80% من Docker مع Spring Boot. 🚀


----
---
---
# 🐳 Dockerfile

طريقة أسهل وأنضف من اللي عملناه يدوي بالأوامر (run + cp + commit) — كل الخطوات دي بقت مكتوبة في ملف واحد، والـ Docker بينفذها أوتوماتيك.

## الملف نفسه

```dockerfile
FROM openjdk:22-jdk
COPY target/docker-test-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

## شرح كل سطر

### 1. `FROM openjdk:22-jdk`

ده بيحدد **الـ base image** اللي هنبني عليها — يعني نظام فيه JDK 22 متثبت جاهز، عشان نقدر نشغل تطبيقات Java. كل Dockerfile لازم يبدأ بـ `FROM`.

### 2. `COPY target/docker-test-0.0.1-SNAPSHOT.jar app.jar`

بينسخ الـ jar بتاعك (اللي عمله Maven في `target/`) من جهازك لجوه الـ image، وبيسميه `app.jar`. ده بديل أنضف لـ `docker cp` اللي كنا بنعمله يدوي بعد ما الكونتينر يشتغل.

### 3. `ENTRYPOINT ["java","-jar","/app.jar"]`

ده بيحدد **الأمر اللي هيشتغل تلقائيًا** أول ما حد يعمل `docker run` من الـ image دي. هنا بنقوله: شغل الجافا وشغل الـ jar. ده بديل الـ `--change='CMD [...]'` اللي كنا بنعمله مع `docker commit`.

## خطوات التنفيذ

**1. بناء الـ jar أولًا (لازم يكون موجود قبل ما تبني الـ image):**

```bash
mvn clean package
```

**2. بناء الـ image من الـ Dockerfile:**

```bash
docker build -t alidocker:v3 .
```

- `-t alidocker:v3` → اسم وتاج الـ image
- `.` → معنى إن الـ Dockerfile في المجلد الحالي

**3. تشغيل container من الـ image:**

```bash
docker run alidocker:v3
```

**4. (اختياري) تشغيله في الخلفية وتشوف الـ logs:**

```bash
docker run -d --name app-container alidocker:v3
docker logs app-container
```

---

**الفرق عن الطريقة اليدوية اللي عملناها قبل كده:**

|اليدوي (الأول)|Dockerfile (دلوقتي)|
|---|---|
|`docker run` → container فاضي|`FROM`|
|`docker cp` لنقل الـ jar|`COPY`|
|`docker commit --change`|`ENTRYPOINT/CMD`|
|كل خطوة بأمر لوحدها|كل حاجة في ملف واحد، تتكرر بسهولة|

عايز أضيفلك `.dockerignore` كمان عشان الـ build ما يبقاش بطيء بسبب ملفات زيادة زي `target/classes`؟


# 🐳 Docker Compose — الفلو الكامل

## 1. الفكرة الأساسية

بدل ما تشغل كل container بأمر `docker run` لوحده وتظبط الشبكة والـ ports يدوي، بتكتب **ملف واحد** (`docker-compose.yml`) يوصف كل السيرفيسات اللي محتاجها، وأمر واحد يشغلهم كلهم مع بعض:

```bash
docker compose up --build
```

## 2. مثال كامل (التطبيق بتاعك + postgres)

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    networks:
      - s-network
    depends_on:
      - postgres
    volumes:
      - ./logs:/app/logs

  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: Sara
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: sara_db
    ports:
      - "5433:5432"
    networks:
      - s-network
    volumes:
      - pg-data:/var/lib/postgresql/data

networks:
  s-network:
    driver: bridge

volumes:
  pg-data:
```

## 3. شرح كل جزء

### `services:`

كل service هنا = container واحد هيشتغل. عندك اتنين: `app` و `postgres`.

### `build: .`

يقول لـ Docker Compose يبني الـ image من الـ Dockerfile الموجود في المجلد الحالي.

### `image: postgres:13`

بدل `build`، هنا بنقول استخدم image جاهزة من Docker Hub مباشرة (مش محتاجين نبنيها، هي جاهزة).

### `ports:`

ربط بورت جهازك بالبورت جوه الكونتينر:

```
"8000:8000"  →  جهازك:الكونتينر
```

### `environment:`

متغيرات بيئة بتتبعت للكونتينر وقت التشغيل — هنا بتستخدمها postgres عشان تعرف تعمل user وpassword وقاعدة بيانات افتراضية.

### `depends_on:`

يقول لـ Compose: "شغل postgres الأول، وبعدين شغل app". **ملحوظة مهمة:** ده بيتحكم في **ترتيب التشغيل** بس، مش بيتأكد إن postgres "جاهزة فعليًا" تستقبل اتصالات (ممكن الكونتينر يشتغل بس الداتابيز نفسها لسه بتعمل setup). لو حصلت مشكلة كده، نضيف healthcheck (أقولك تحت لو عايز).

---

## 4. الشبكة (Network) — إزاي الكونتينرات بتشوف بعض

دي أهم نقطة سألت عليها 👇

لما تحدد:

```yaml
networks:
  - s-network
```

لكل السيرفيسين، Docker بيعمل **شبكة افتراضية داخلية** بينهم، والاتنين بقوا على نفس الشبكة دي.

**السحر هنا:** Docker Compose بيعمل **DNS داخلي تلقائي** — يعني كل container يقدر يوصل للتاني **باستخدام اسم السيرفيس نفسه كـ hostname**، من غير ما تحتاج تعرف IP خالص.

يعني في كود الـ Spring Boot بتاعك، الـ connection string للداتابيز يكون:

```properties
spring.datasource.url=jdbc:postgresql://postgres:5432/sara_db
```

لاحظ: **`postgres`** هنا هو اسم الـ service في الـ yml، مش `localhost` ومش IP. Docker بيحل الاسم ده لـ IP الكونتينر تلقائيًا.

**ليه محتاجين الشبكة دي أصلاً؟**

- لو سيبت الكونتينرات بدون `networks` محدد، Compose بيعمل default network برضو وبتقدر تتكلم، بس تحديد شبكة مخصصة بيديك تحكم أكتر (مثلاً تعزل سيرفيسات معينة عن بعضها لو عندك أكتر من شبكة).
- من غير شبكة مشتركة، الكونتينرات معزولة عن بعض تمامًا وما يشوفوا بعضهم خالص.

---

## 5. الـ Volume — هو إيه ولماذا؟

### المشكلة بدون volume:

الكونتينر لما يقف أو يتمسح، **كل البيانات اللي جواه بتضيع** (لأن الـ writable layer بتاعته بيروح). يعني لو الـ postgres اتقفل أو اتعمله rebuild، **كل بيانات قاعدة البيانات هتمسح!**

### الحل: Volume

الـ Volume هو **مساحة تخزين بره الكونتينر** (على جهازك الحقيقي، بيديره Docker)، بتتربط بمجلد جوه الكونتينر. يعني البيانات بتتخزن فعليًا على جهازك مش جوه الكونتينر، فلو الكونتينر اتمسح، البيانات تفضل موجودة.

```yaml
volumes:
  - pg-data:/var/lib/postgresql/data
```

ده معناه: "اللي بيتخزن جوه الكونتينر في `/var/lib/postgresql/data` (ده المكان اللي postgres بتحفظ فيه بياناتها)، اربطه بـ volume اسمه `pg-data`".

وتحت لازم تعرفه في الآخر:

```yaml
volumes:
  pg-data:
```

**يعني عمليًا:**

- تعمل `docker compose down` → الكونتينرات بتقف وتتمسح
- تعمل `docker compose up` تاني → كونتينر postgres جديد، **بس البيانات القديمة لسه موجودة** لأنها متخزنة في الـ volume مش جوه الكونتينر القديم

---

## 6. الأوامر الأساسية اللي هتستخدمها

```bash
docker compose up --build      # يبني ويشغل كل السيرفيسات
docker compose up -d           # يشغلهم في الخلفية
docker compose down            # يوقف ويمسح الكونتينرات (الـ volumes تفضل، إلا لو -v)
docker compose down -v         # يوقف ويمسح الكونتينرات + الـ volumes (هتفضي الداتابيز!)
docker compose ps              # يعرضلك الكونتينرات الشغالة
docker compose logs -f app     # تتابع الـ logs بتاعة سيرفيس معين
docker compose exec app bash   # تدخل جوه الكونتينر بتاع app
```

---

## 7. خلاصة الفلو كامل

```
1. تكتب docker-compose.yml يوصف السيرفيسات (app + postgres)
        ↓
2. تحدد network مشتركة بينهم (s-network)
        ↓
3. تحدد volume لقاعدة البيانات عشان بياناتها ما تضيعش (pg-data)
        ↓
4. docker compose up --build
        ↓
5. Docker بيبني image الـ app، وينزل image الـ postgres
        ↓
6. بيشغل الاتنين على الشبكة المشتركة
        ↓
7. التطبيق بيوصل للداتابيز عن طريق الاسم "postgres" كـ hostname
        ↓
8. لو الكونتينرات اتقفلت، الداتا محفوظة في الـ volume
```

عايزني أضيفلك healthcheck عشان الـ app ما يحاولش يتصل بالداتابيز قبل ما تكون جاهزة فعليًا؟