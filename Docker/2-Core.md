
# 🐳 Docker — الدليل الشامل

---

## 1. معمارية Docker Engine — بيكلموا بعض إزاي؟

لما بتكتب `docker run nginx` في الـ terminal، الأمر ده بيعدي على 4 مكونات قبل ما الـ container يشتغل فعلاً.

```
Docker CLI  →  dockerd  →  containerd  →  runc  →  Linux Kernel  →  Container
```

### المكونات بالتفصيل

**Docker CLI (العميل)** ده اللي بتكتب فيه الأوامر. بيترجم الأمر لـ HTTP Request وبيبعته للـ Daemon عن طريق Unix Socket موجود على `/var/run/docker.sock`. ممكن يكون على نفس الجهاز أو remote.

**dockerd (الـ Daemon)** القلب. بيستنى requests على الـ Unix Socket. بيتحكم في كل حاجة — Images, Networks, Volumes. هو مش بيشغّل الـ containers نفسه، بيفوض ده لـ containerd عن طريق gRPC.

**containerd** مسؤول عن الـ container lifecycle كامل (create → start → stop → delete). بيسحب الـ images من الـ registry. بيستخدم runc عشان يعمل الـ container الفعلي.

**runc** أصغر جزء وأهمه. بيتكلم مع الـ Linux Kernel مباشرةً. بيعمل الـ namespaces (عشان العزل) والـ cgroups (عشان يحدد الموارد زي CPU وMemory). بيشغّل الـ process الفعلي جوا الـ container.

> **ملاحظة:** لو الـ Daemon مش شغال وكتبت أي أمر هتلاقي: `Cannot connect to the Docker daemon`

---

## 2. Images — الـ Layers والـ Read-Only

### الـ Image مش ملف واحد

الـ Image عبارة عن مجموعة layers مرتبة فوق بعض، كل layer بتمثل تغيير. كل layers الـ image هي **Read-Only** — مش ممكن تتغير.

```
┌─────────────────────────────────┐  ← Container Layer (Read-Write) ← بس الـ container اللي بيضيفها
├─────────────────────────────────┤
│  Layer 3 — COPY app.py /app     │
├─────────────────────────────────┤
│  Layer 2 — RUN pip install      │  ← كل دول Read-Only (الـ Image)
├─────────────────────────────────┤
│  Layer 1 — RUN apt-get update   │
├─────────────────────────────────┤
│  Base  — FROM python:3.11       │
└─────────────────────────────────┘
```

لو عملت 10 containers من نفس الـ image، كل واحد بياخد نفس الـ layers بس بيعمل layer R/W خاصة بيه. الـ Image نفسها ما بتتأثرش.

### repo:tag

الاسم الكامل لأي image بيبقى على الشكل ده: `nginx:1.25` أو `myapp:latest`

- **repo** = اسم الـ image
- **tag** = الـ version
- لو مكتبتش tag بيبقى `latest` أوتوماتيك

---

## 3. Image Commands

### عرض الـ Images المحلية

```bash
docker image ls
```

بيعرض كل الـ images الموجودة على جهازك مع الـ repository والـ tag والـ ID والـ size.

```bash
docker image ls -q
```

`-q` يعني quiet — بيرجع الـ IDs بس من غير تفاصيل. مفيدة لما تيجي تمرر الـ output لأمر تاني.

---

### سحب Image من Docker Hub

```bash
docker pull nginx
```

بيسحب آخر version (latest).

```bash
docker pull nginx:1.25
```

بيسحب version محددة.

```bash
docker pull ubuntu:22.04
```

لو الـ image موجودة محلياً مش بيسحبها تاني — بيتحقق الأول.

---

### فحص Image بالتفصيل

```bash
docker image inspect nginx
```

بيرجع JSON كامل عن الـ image فيه:

- **Id** — الـ full SHA256 hash
- **Created** — امتى اتبنيت
- **Config.Cmd** — الـ default command
- **Config.Env** — الـ environment variables
- **RootFS.Layers** — كل الـ layers في الآخر
- **History** — كل أمر اتعمل في الـ Dockerfile

```bash
docker image inspect nginx:latest
docker image inspect 45c          # ممكن تستخدم أول 3 حروف من الـ ID
```

لو عايز field بعينها:

```bash
docker image inspect nginx --format '{{.Id}}'
docker image inspect nginx --format '{{.Config.Cmd}}'
```

---

### شوف الـ Layers

```bash
docker history nginx
```

بيعرض كل layer وحجمها والأمر اللي عملها.

---

### مسح Images

```bash
docker image rm nginx
```

بيمسح image واحدة. لازم مفيش container (حتى لو stopped) بيستخدمها.

```bash
docker image rm nginx ubuntu:22.04
```

مسح أكتر من image دفعة واحدة.

```bash
docker image rm $(docker image ls -q)
```

مسح كل الـ images دفعة واحدة. الـ `$()` بيشغّل الأمر اللي جواه الأول وبيحط النتيجة مكانه.

```bash
docker image prune
```

بيمسح الـ dangling images فقط — دي الـ images اللي ملهاش tag (بتظهر كـ `<none>`).

```bash
docker image prune -a
```

بيمسح كل الـ images اللي مفيش container بيستخدمها حالياً.

> **تحذير:** لازم تتأكد إن مفيش container شغال أو موقوف من الـ image اللي هتمسحها. لو في container، امسحه الأول أو استخدم `-f` للـ force.

---

### Alias للتسهيل

```bash
alias dimg-clean='docker image rm $(docker image ls -q)'
```

حطه في `~/.bashrc` أو `~/.zshrc` عشان يتحفظ.

---

## 4. Containers — الـ Runtime للـ Image

الـ Container هو الـ Image وهي شغالة. بياخد كل layers الـ image (Read-Only) ويضيف فوقيهم layer واحدة Read-Write.

```
Image (ثابتة، Read-Only)  →  docker run  →  Container (Image + RW Layer)
```

---

## 5. Container Lifecycle — دورة الحياة

```
CREATED  →  RUNNING  ⇄  PAUSED
              ↓
           STOPPED  →  REMOVED
```

|الحالة|بتحصل إزاي|
|---|---|
|CREATED|`docker create` — بيعمل الـ container بس مش بيشغله|
|RUNNING|`docker start` أو `docker run`|
|PAUSED|`docker pause` — بيوقف مؤقتاً بدون مسح|
|STOPPED|`docker stop` أو `docker kill`|
|REMOVED|`docker rm` — بعد كده مش ممكن ترجعه|

---

## 6. Signals — SIGTERM vs SIGKILL

### SIGTERM — الوقف الناعم

```bash
docker stop mycontainer
```

بيبعت SIGTERM للـ process — ده بيقوله "وقّف نفسك بشكل نضيف". الـ app ممكن تعمل cleanup وتقفل connections وتحفظ data قبل ما توقف. Docker بيستنى 10 ثواني، لو مش ردّ بيبعت SIGKILL.

```bash
docker stop --time 20 mycontainer    # زود وقت الانتظار لـ 20 ثانية
```

### SIGKILL — الوقف القسري

```bash
docker kill mycontainer
```

بيقتل الـ process فوراً من غير ما يديه فرصة يعمل حاجة. زي إنك تسحب الكهرباء. استخدمه بس لو الـ container مش بيستجوب لـ stop.

---

## 7. Restart Policies — سياسات إعادة التشغيل

بتحددها في `docker run` بـ `--restart`

### no (الـ default)

```bash
docker run -d nginx
```

لو الـ container وقف — وقف. مش بيتشغل تاني.

### always

```bash
docker run -d --restart always nginx
```

بيعيد التشغيل دايماً حتى لو exit code = 0. وبيشتغل أوتوماتيك مع الـ Docker Daemon لو الجهاز اترستارت. مفيد للـ production servers.

### on-failure

```bash
docker run -d --restart on-failure myapp
docker run -d --restart on-failure:5 myapp    # أقصى 5 محاولات
```

بيعيد التشغيل بس لو الـ container خرج بـ error (exit code ≠ 0). لو الـ app خرجت بشكل طبيعي (exit 0) مش بيعيدها.

### unless-stopped

```bash
docker run -d --restart unless-stopped nginx
```

زي always بالظبط، بس لو أنت وقفته يدوياً بـ `docker stop` — مش بيشتغل تاني لو الجهاز اترستارت. بيحترم قرارك.

---

## 8. Container Commands — كل أمر بالتفصيل

### docker run — الأمر الأهم

```bash
docker container run nginx
```

بيعمل 3 حاجات دفعة واحدة:

1. **Pull** — لو الـ image مش موجودة locally بيسحبها
2. **Create** — بيعمل container من الـ image
3. **Start** — بيشغل الـ container

---

### -it vs -d

**-it (Interactive Mode)**

```bash
docker run -it ubuntu bash
```

- `-i` = interactive — بيخليك تبعت input للـ container
- `-t` = tty — بيعمل terminal جوا الـ container

الـ terminal بتاعتك بتتحول لجوا الـ container. بتستخدمه لما عايز تشتغل جوا الـ container يدوياً أو تعمل debugging.

**-d (Detached Mode)**

```bash
docker run -d nginx
```

الـ container بيشتغل في الخلفية وبيرجع لك الـ terminal على طول. بتستخدمه للـ web servers والـ databases — أي حاجة المفروض تفضل شغالة من غير ما تمسك الـ terminal.

---

### الـ Flags المهمة في docker run

```bash
docker run -d \
  --name webserver \        # اسم للـ container بدل الـ random name
  -p 8080:80 \              # port mapping: host:container
  -e MY_VAR=hello \         # environment variable
  -v /mydata:/app/data \    # volume: host_path:container_path
  --restart always \        # restart policy
  --rm \                    # امسح الـ container أوتوماتيك لما يوقف
  nginx
```

**Port mapping `-p 8080:80`** الرقم الأول (8080) هو البورت على جهازك. الرقم التاني (80) هو البورت جوا الـ container. يعني لو فتحت `localhost:8080` هيوصّلك لبورت 80 جوا الـ container.

**Volume `-v /mydata:/app/data`** بيربط folder على جهازك بـ folder جوا الـ container. الـ data بتتحفظ على جهازك حتى لو الـ container اتمسح.

---

### عرض الـ Containers

```bash
docker container ls
docker ps                   # نفس الأمر، shortcut
```

بيعرض الـ running containers بس.

```bash
docker container ls -a
docker ps -a
```

بيعرض كل الـ containers — running وstopped.

```bash
docker ps -q                # IDs بس للـ running
docker ps -aq               # IDs بس للكل
```

---

### فحص Container

```bash
docker container inspect mycontainer
```

بيرجع JSON كامل فيه كل حاجة عن الـ container — الـ IP address, الـ ports, الـ volumes, الـ environment variables, الـ restart policy.

---

### التحكم في الـ Container

```bash
docker container stop mycontainer       # SIGTERM ثم SIGKILL بعد 10 ثواني
docker container start mycontainer      # بيشغل container موقوف
docker container restart mycontainer    # stop + start
docker container pause mycontainer      # إيقاف مؤقت
docker container unpause mycontainer    # استكمال
```

---

### الدخول لـ Container شغال

```bash
docker exec -it mycontainer bash
docker exec -it mycontainer sh          # لو bash مش موجود
```

مختلف عن `docker run` — ده بيشغل command جوا container **شغال بالفعل** مش بيعمل container جديد. مفيد جداً للـ debugging.

---

### الـ Logs

```bash
docker logs mycontainer                 # بيعرض كل الـ logs
docker logs -f mycontainer             # -f = follow, زي tail -f
docker logs --tail 50 mycontainer      # آخر 50 سطر بس
docker logs --since 1h mycontainer     # logs من ساعة بس
```

---

### مسح الـ Containers

```bash
docker container rm mycontainer             # لازم يكون stopped
docker container rm -f mycontainer          # -f = force, حتى لو شغال
docker container rm $(docker ps -aq)        # مسح الكل
docker container prune                      # بيمسح الـ stopped containers بس
```

---

### مراقبة الـ Resources

```bash
docker stats                    # CPU/Memory/Network/IO لكل الـ containers live
docker stats mycontainer        # container معين بس
docker top mycontainer          # الـ processes الشغالة جوا الـ container
```

---

## 9. Dockerfile — ابني Image بتاعتك

### مثال كامل — Flask App

```dockerfile
# الـ Base Image — من فين هنبدأ
FROM python:3.11-slim

# الـ working directory جوا الـ container
WORKDIR /app

# نسخ requirements الأول (عشان نستفيد من الـ cache)
COPY requirements.txt .

# تثبيت الـ dependencies
RUN pip install -r requirements.txt

# نسخ باقي الكود
COPY . .

# فتح البورت (documentation بس، مش بيفتحه فعلاً)
EXPOSE 5000

# الأمر اللي بيشتغل لما الـ container يبدأ
CMD ["python", "app.py"]
```

### شرح كل instruction

|Instruction|بتعمل إيه|
|---|---|
|`FROM`|الـ Base Image، كل image لازم تبدأ منها، بتعمل أول layer|
|`WORKDIR`|بيحدد الـ working directory جوا الـ container، زي `cd`|
|`COPY`|بينسخ ملفات من جهازك لجوا الـ image|
|`ADD`|زي COPY بس بيقدر يفك ضغط `.tar` وبيدعم URLs|
|`RUN`|بينفذ command وقت الـ build وبيعمل layer جديدة|
|`EXPOSE`|بيوثّق البورت (مش بيفتحه فعلاً)|
|`ENV`|بيحدد environment variable تتحفظ في الـ image|
|`ARG`|متغير وقت الـ build بس، مش بيتحفظ في الـ image|
|`CMD`|الأمر الـ default لما الـ container يشتغل، ممكن تعداه في docker run|
|`ENTRYPOINT`|زي CMD بس صعب تعداه، بيتحكم في الـ executable الأساسي|

### Build الـ Image

```bash
docker build -t myapp:1.0 .
```

- `-t myapp:1.0` — اسم وتاغ الـ image
- `.` — الـ build context (المجلد الحالي، المكان اللي فيه الـ Dockerfile)

```bash
docker build -t myapp:latest -f Dockerfile.prod .    # Dockerfile بـ اسم مختلف
docker build -t myapp:1.0 --no-cache .               # من غير cache
```

---

## 10. Docker Hub — ارفع التطبيق

### الـ Flow الكامل

**الخطوة 1 — إنشاء حساب** اعمل account على `hub.docker.com`. الـ username مهم لأنه هيكون جزء من اسم الـ image.

**الخطوة 2 — Login**

```bash
docker login
```

هيطلب username وpassword.

**الخطوة 3 — Tag الـ Image**

```bash
docker tag myapp:1.0 yourusername/myapp:1.0
docker tag myapp:1.0 yourusername/myapp:latest
```

الـ format دايماً: `username/repo-name:tag`

**الخطوة 4 — Push**

```bash
docker push yourusername/myapp:1.0
docker push yourusername/myapp:latest
```

**الخطوة 5 — Pull من أي جهاز**

```bash
docker pull yourusername/myapp:1.0
docker run -d -p 5000:5000 yourusername/myapp:1.0
```

---

### الـ Workflow الكامل من الصفر

```bash
# 1. ابني الـ Image
docker build -t myflaskapp:1.0 .

# 2. اختبر محلياً
docker run -d -p 5000:5000 --name flask-test myflaskapp:1.0
curl http://localhost:5000

# 3. لو اشتغل صح، login
docker login

# 4. Tag
docker tag myflaskapp:1.0 ahmed123/myflaskapp:1.0
docker tag myflaskapp:1.0 ahmed123/myflaskapp:latest

# 5. Push
docker push ahmed123/myflaskapp:1.0
docker push ahmed123/myflaskapp:latest

# 6. على أي سيرفر تاني
docker pull ahmed123/myflaskapp:1.0
docker run -d -p 5000:5000 --restart always ahmed123/myflaskapp:1.0
```

---

### Search في Hub

```bash
docker search nginx
docker search --filter is-official=true python
```

---

## Quick Reference

```bash
# ═══ IMAGES ═══
docker pull nginx
docker images
docker image ls -q
docker image rm nginx
docker image rm $(docker image ls -q)
docker image prune -a
docker build -t myapp:1.0 .
docker image inspect nginx
docker history nginx

# ═══ CONTAINERS ═══
docker run -d -p 8080:80 --name web nginx
docker run -it ubuntu bash
docker ps -a
docker start web
docker stop web
docker restart web
docker exec -it web bash
docker logs -f web
docker stats
docker rm -f web
docker container prune

# ═══ HUB ═══
docker login
docker tag myapp:1.0 user/myapp:1.0
docker push user/myapp:1.0
docker pull user/myapp:1.0

# ═══ CLEANUP ═══
docker system prune        # بيمسح كل حاجة مش بتستخدمها
docker system prune -a     # كل حاجة خالص
```