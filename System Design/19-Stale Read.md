

```text
Replication
├── Synchronous Replication
├── Asynchronous Replication
│
├── Replication Lag
│
├── Stale Read
│
├── Read-After-Write Consistency
│
├── Replication Types
│   ├── Single Leader
│   ├── Multi Leader
│   └── Leaderless
│
└── Data Consistency Across Shards
```

---

# Stale Read

## ما هو Stale Read؟

هو قراءة بيانات قديمة (Outdated Data) من Replica لأن التحديث الأخير لم يصل إليها بعد.

---

## مثال

لدينا:

```text
Primary
Replica
```

في الساعة:

```text
12:00:00
```

قام المستخدم بتغيير اسمه:

```sql
UPDATE users
SET name = 'Ali'
WHERE id = 1;
```

تم التحديث على الـ Primary مباشرة.

لكن الـ Replica لم تستلم التحديث بعد.

---

في الساعة:

```text
12:00:01
```

قام المستخدم بعمل Read من الـ Replica:

```sql
SELECT * FROM users;
```

الـ Replica ما زالت تحتوي:

```text
name = Ahmed
```

رغم أن القيمة الصحيحة أصبحت:

```text
name = Ali
```

هذا يسمى:

```text
Stale Read
```

---

# لماذا يحدث؟

بسبب:

```text
Replication Lag
```

أي أن النسخة الاحتياطية لم تستقبل آخر التحديثات بعد.

---

# العلاقة بين Replication Lag و Stale Read

```text
Replication Lag
        ↓
Replica Behind
        ↓
Stale Read
```

أو بشكل أبسط:

```text
Lag = Cause
Stale Read = Result
```

---

# مشكلة حقيقية

تخيل أنك على Facebook أو Instagram.

قمت بنشر منشور جديد.

```text
Write Request
```

ثم عملت Refresh مباشرة.

```text
Read Request
```

إذا تمت القراءة من Replica متأخرة:

قد لا ترى المنشور الذي نشرته منذ ثانية واحدة.

هذا مثال عملي على:

```text
Stale Read
```

---

# كيف نمنع Stale Read؟

## 1. القراءة من Primary بعد الكتابة

بعد تنفيذ:

```sql
INSERT
UPDATE
DELETE
```

نجبر المستخدم لفترة قصيرة أن يقرأ من Primary.

يسمى ذلك:

```text
Read Your Writes
```

---

## 2. استخدام Synchronous Replication

```text
Write
 ↓
Primary
 ↓
Replica
 ↓
Success
```

لن يرجع النظام Success إلا بعد تحديث Replica.

وبالتالي:

```text
Lag ≈ 0
```

لكن الأداء يصبح أبطأ.

---

## 3. انتظار اكتمال المزامنة

بعض الأنظمة تؤخر القراءة لثوانٍ قليلة حتى تصل البيانات للـ Replica.

---

# Read-After-Write Consistency

## ما هي؟

ضمان أن المستخدم الذي كتب البيانات سيرى تحديثاته مباشرة عند أول قراءة.

---

### مثال

المستخدم غيّر صورته الشخصية:

```text
Upload New Profile Picture
```

بعدها مباشرة فتح صفحته.

يجب أن يرى الصورة الجديدة.

وليس القديمة.

---

إذا رأى الصورة الجديدة دائماً:

```text
Read-After-Write Consistency
```

إذا رأى القديمة أحياناً:

```text
Stale Read
```

---

### قاعدة مهمة للإنترفيو

لو سألك:

> What is the difference between Replication Lag and Stale Read?

الإجابة:

- **Replication Lag** = الزمن الذي تستغرقه البيانات للانتقال من Primary إلى Replica.
    
- **Stale Read** = قراءة بيانات قديمة بسبب هذا التأخير.
    

يعني:

```text
Replication Lag → Technical Delay

Stale Read → User Visible Problem
```

وده فعلاً الترتيب اللي أغلب مهندسي الـ Distributed Systems بيشرحوا بيه الموضوع. بعد ما تفهم Stale Read كويس، ساعتها ندخل على **Read-After-Write Consistency** و **Eventual Consistency** لأنهم مرتبطين مباشرة بنفس المشكلة.