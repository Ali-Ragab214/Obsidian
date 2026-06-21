ممتاز يا علي. ده من أهم المواضيع في الـ System Design، وخصوصًا لو ناوي تشتغل Backend على مستوى شركات كبيرة.

قبل ما ندخل في Sharding، لازم تفهم Replication كويس جدًا لأنها أبسط وهي الأساس.

---

# أولًا: ما هي Replication؟

تخيل عندك Database واحدة:

```text
        Users
           |
           v
      +---------+
      | DB Main |
      +---------+
```

كل الـ Users بيقروا ويكتبوا عليها.

---

المشكلة؟

لو الـ Database وقعت:

```text
        Users
           |
           X
      +---------+
      | DB Main |
      +---------+
```

💥 الموقع كله وقع.

---

كمان لو عندك مليون User:

```text
1,000,000 Read Requests
100,000 Write Requests
```

Database واحدة هتبدأ تتعب.

---

هنا بنستخدم Replication.

---

# فكرة Replication

بدل Database واحدة:

```text
           Primary
         +---------+
         |   DB1   |
         +---------+
```

نعمل نسخ منها:

```text
                Primary
              +---------+
              |   DB1   |
              +---------+
                  |
          ------------------
          |                |
          v                v
      +---------+      +---------+
      |   DB2   |      |   DB3   |
      +---------+      +---------+

       Replica          Replica
```

DB2 و DB3 نسخ من DB1.

---

# ليه بنعمل Replication؟

## 1. High Availability

لو DB1 وقعت:

```text
DB1 ❌
DB2 ✅
DB3 ✅
```

لسه عندك قواعد بيانات شغالة.

---

## 2. Fault Tolerance

النظام يفضل شغال حتى لو جزء منه وقع.

---

## 3. Read Scaling

بدل كل القراءات تروح لـ DB1:

```text
Read -> DB2
Read -> DB3
Read -> DB2
Read -> DB3
```

فنخفف الحمل.

---

# Primary-Replica Pattern

أشهر Pattern.

```text
                WRITE
                  |
                  v

             +--------+
             |Primary |
             +--------+
                  |
        -------------------
        |                 |
        v                 v

    +--------+       +--------+
    |Replica1|       |Replica2|
    +--------+       +--------+
```

---

## الكتابة

لازم تروح للـ Primary

```sql
INSERT INTO users ...
UPDATE users ...
DELETE FROM users ...
```

---

## القراءة

ممكن تروح للـ Replicas

```sql
SELECT * FROM users
```

---

مثال Amazon:

```text
1000 Writes / second
100000 Reads / second
```

القراءات أكثر بكتير.

فنوزعها على الـ Replicas.

---

![[Pasted image 20260621013036.png]]
 بص لما اليوزر يبعت ال request in synchronous  هيدخل الداتا بيز ويتخزن في db1 فالريسبونس مش هيرجع الا لما النسخه تتخزن في db2 and db3   الريكوست هيرجع بعد كدا بقا بريسبونس 


لكن في ال async اليوزر مثلا هيبعت الريكوست والسيرفر هيرد بريسبونس بقا وهو بيخزن الداتا في db2 and db3 


---

# Synchronous Replication

## الشكل

```text
Client
   |
WRITE
   |
   v

Primary
   |
   |
   +----> Replica 1
   |
   +----> Replica 2

Wait...
```

---

الـ Primary مش بيرد للمستخدم إلا بعد ما كل Replica تكتب.

---

مثال

```sql
INSERT User
```

الخطوات:

```text
Primary writes
Replica1 writes
Replica2 writes

then

Success
```

---

الميزة:

```text
كل الداتا متطابقة 100%
```

---

العيب:

```text
أبطأ
```

لأنه مستني الجميع.

---

# Asynchronous Replication

الأشهر في الواقع.

```text
Client
   |
WRITE
   |
   v

Primary
   |
Success immediately
   |
   v

Replicas later...
```

---

مثال

```sql
INSERT User
```

```text
Primary writes

Success

Replica1 updated later
Replica2 updated later
```

---

الميزة:

```text
سريع جدًا
```

---

العيب:

قد يحدث:

```text
Replication Lag
```

---

مثال

كتبت:

```text
Ali
```

في Primary.

بعد ثانية:

```text
Replica1
```

لسه موصلهاش التحديث.

فتقرأ بيانات قديمة.

---

# Read Replicas

الصورة الحقيقية غالبًا:

```text
                Load Balancer
                       |
         --------------------------------
         |              |              |
         v              v              v

      Replica1      Replica2      Replica3
```

كل SELECT يتوزع.

---

بدل:

```text
100000 Reads
     |
     v
     DB
```

تبقى:

```text
33000 -> Replica1
33000 -> Replica2
33000 -> Replica3
```

---

# كيف تحسن Performance؟

نفترض:

Database واحدة تتحمل:

```text
10000 Read/sec
```

عملت 3 Replicas:

```text
30000 Read/sec
```

تقريبًا.

وده اسمه:

```text
Horizontal Read Scaling
```

---

# Failover

دي أهم نقطة.

---

بدون Replication

```text
App
 |
 v

DB
```

لو DB وقعت:

```text
App
 |
 X

DB
```

💥 الموقع انتهى.

---

مع Replication

```text
          Primary
             |
      ----------------
      |              |
      v              v

   Replica1      Replica2
```

لو الـ Primary وقع:

```text
Primary ❌
Replica1 ✅
Replica2 ✅
```

نختار Replica:

```text
Replica1
```

وتتحول إلى:

```text
New Primary
```

---

فيصبح:

```text
            Primary
           Replica1
                |
                v
            Replica2
```

---

وده اسمه:

```text
Failover
```

---

# مثال مصر وآسيا وأوروبا

نفترض:

```text
Egypt   -> DB1
Europe  -> DB2
Asia    -> DB3
```

---

بدون Replication

مستخدم في اليابان:

```text
Japan
  |
  |
  v

Egypt DB
```

كل Request تعدي آلاف الكيلومترات.

Latency عالية.

---

تقريبًا:

```text
200ms
300ms
400ms
```

---

# مع Replication

```text
Egypt DB
   |
   +------> Europe Replica
   |
   +------> Asia Replica
```

---

مستخدم من اليابان:

```text
Japan User
     |
     v

Asia Replica
```

---

بدل:

```text
300ms
```

تبقى:

```text
20ms
30ms
```

---

# لو أوروبا وقعت

بدون Replication

```text
Europe Users
     |
     X
```

المستخدمين متضررين.

---

مع Replication

```text
Europe Replica ❌

Redirect

Europe Users
      |
      v

Egypt Replica
```

الخدمة تستمر.

---

# العلاقة بين Replication والمفاهيم المهمة

## Fault Tolerance

قبل:

```text
1 DB
```

نقطة فشل واحدة.

بعد:

```text
3 DBs
```

لو واحدة وقعت النظام يكمل.

---

## Throughput

قبل:

```text
10000 Reads/sec
```

بعد:

```text
30000 Reads/sec
```

لأن القراءات موزعة.

---

## Latency
الـ **Latency** ببساطة هي:

> **الوقت اللي بيستغرقه الطلب عشان يروح للسيرفر ويرجع الرد للمستخدم.**

يعني لو فتحت موقع:

```
www.google.com
```

وضغطت Enter.

المدة بين:

```
إرسال الطلب        ↓استلام الرد
```

دي اسمها **Latency**.
قبل:

```text
Japan -> Egypt
```

Latency كبيرة.

بعد:

```text
Japan -> Asia Replica
```

Latency صغيرة.

---

# الخلاصة المهمة للانترفيو

Replication تحقق 3 أشياء رئيسية:

```text
1. High Availability
2. Fault Tolerance
3. Read Scalability
```

لكنها **لا تحل مشكلة الكتابة (Write Scaling)** لأن كل الـ Writes ما زالت تذهب إلى الـ Primary.

وهنا يأتي دور **Sharding**، لأنه يحل مشكلة الـ Writes والـ Storage Limits، وده الجزء اللي بعد Replication مباشرة في الـ System Design.