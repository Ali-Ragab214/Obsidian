ممتاز. **Dirty Checking** من أكتر الأسئلة اللي بتيجي في مقابلات Spring Boot/Hibernate، ومش هتفهمه كويس إلا لو فاهم **Persistence Context** و **First-Level Cache**. هنبدأ من الصفر.

---

# أولاً: المشكلة اللي Dirty Checking بيحلها

تخيل إنك جبت User من قاعدة البيانات.

```java
User user = entityManager.find(User.class, 1L);
```

بعدها غيرت الاسم.

```java
user.setName("Ali");
```

السؤال هنا:

**إنت مقولتش لـ Hibernate يعمل Update، يبقى إزاي البيانات اتحدثت؟**

الإجابة:

**Dirty Checking**

---

# يعني إيه Dirty؟

Dirty معناها

> الكائن (Object) اتغير بعد ما Hibernate قرأه من قاعدة البيانات.

مثال

قبل

```text
User

id = 1
name = Ahmed
```

بعد

```java
user.setName("Ali");
```

بقى

```text
User

id = 1
name = Ali
```

يبقى الـ Entity بقت

```text
Dirty
```

يعني Modified.

---

# Dirty Checking

هو ميكانيزم داخل Hibernate

بيراقب الـ Entities الموجودة داخل

```text
Persistence Context
```

ولو لاحظ إن أي Entity اتغيرت،

Hibernate يعمل

```sql
UPDATE
```

أوتوماتيك.

---

# السؤال المهم

## هو Hibernate عرف إنها اتغيرت إزاي؟

لما يعمل

```java
entityManager.find(User.class,1L);
```

يحصل الآتي

```text
Database

↓

User(id=1,name="Ahmed")

↓

Persistence Context
```

لكن Hibernate بيعمل حاجة ذكية.

هو بيخزن

نسخة من الـ Entity.

اسمها

Snapshot

---

يبقى الشكل

```text
Persistence Context

----------------------------

Entity

name = Ahmed

----------------------------

Snapshot

name = Ahmed

----------------------------
```

---

بعد شوية

أنت كتبت

```java
user.setName("Ali");
```

بقى

```text
Entity

Ali
```

لكن

Snapshot

لسه

```text
Ahmed
```

---

عند نهاية الـ Transaction

Hibernate يقارن

```text
Entity

Ali

VS

Snapshot

Ahmed
```

لاحظ الفرق.

إذن

```text
Dirty = true
```

فينفذ

```sql
UPDATE users
SET name='Ali'
WHERE id=1;
```

---

# Diagram

```text
Database

↓

User(Ahmed)

↓

Persistence Context

↓

Entity(Ali)

Snapshot(Ahmed)

↓

Compare

↓

UPDATE
```

---

# مثال كامل

لدينا

```text
users

----------------

1 Ahmed

----------------
```

---

كتبنا

```java
@Transactional
public void updateUser(){

    User user =
        entityManager.find(User.class,1L);

    user.setName("Ali");
}
```

لاحظ

مفيش

```java
entityManager.update()
```

ولا

```java
repository.save()
```

ولا أي حاجة.

ومع ذلك

هيتنفذ

```sql
SELECT * FROM users WHERE id=1;
```

ثم

```sql
UPDATE users
SET name='Ali'
WHERE id=1;
```

---

# إمتى Hibernate يعمل المقارنة؟

مش كل ثانية.

بيعملها غالباً عند

## Flush

أو

## Commit

يعني

```text
Transaction

↓

find()

↓

modify()

↓

Commit

↓

Dirty Checking

↓

UPDATE
```

---

# يعني إيه Flush؟

Flush

معناها

Hibernate يقول

> "كل التغييرات اللي في الذاكرة ابعتها للداتابيز."

يعني

```text
Persistence Context

↓

Database
```

---

# العلاقة بين Dirty Checking و Flush

Dirty Checking

هو اكتشاف التغيير.

Flush

هو تنفيذ SQL.

---

يعني

```text
Dirty Checking

↓

وجد تغيير

↓

Flush

↓

UPDATE
```

---

# مثال

```java
@Transactional
public void test(){

    User user =
      entityManager.find(User.class,1);

    user.setName("Ali");

}
```

عند نهاية الميثود

```text
Commit

↓

Flush

↓

Dirty Checking

↓

UPDATE
```

---

# طب لو مغيرتش حاجة؟

```java
User user =
entityManager.find(User.class,1);
```

بس.

Hibernate يقارن

Snapshot

مع

Entity

يلاقيهم متساويين.

يبقى

```text
Dirty = false
```

ولا ينفذ

أي

```sql
UPDATE
```

---

# مثال آخر

```java
User user =
entityManager.find(User.class,1);

user.setAge(20);

user.setEmail("a@gmail.com");
```

Hibernate هيلاحظ

إن

Age

اتغير

و

Email

اتغير.

فينفذ

```sql
UPDATE users

SET age=?,
    email=?

WHERE id=1;
```

---

# طب لو غيرت ورجعت القيمة القديمة؟

مثال

```java
user.setName("Ali");

user.setName("Ahmed");
```

الـ Snapshot

```text
Ahmed
```

والـ Entity

```text
Ahmed
```

يبقى

```text
Dirty = false
```

ولا يحصل Update.

---

# Dirty Checking بيشتغل على إيه؟

بيشتغل فقط على

Managed Entities.

---

## Entity States

وده موضوع مهم جداً.

Hibernate عنده أربع حالات للـ Entity.

---

### 1- Transient

لسه Object جديد.

```java
User user = new User();
```

لسه مش في الداتابيز.

---

### 2- Managed

أول ما Hibernate يعمل

```java
find()
```

أو

```java
persist()
```

يبقى

Managed.

وده الوحيد

اللي Dirty Checking يراقبه.

---

### 3- Detached

كان Managed.

لكن الـ Session قفلت.

```text
Session Closed

↓

Detached
```

لو غيرته

```java
user.setName("Ali");
```

Hibernate

مش شايفه أصلاً.

ولا هيعمل

Update.

---

### 4- Removed

هيتشال من الداتابيز.

---

# Diagram

```text
new User()

↓

Transient

↓

persist()

↓

Managed

↓

Session Close

↓

Detached

↓

remove()

↓

Removed
```

---

# طب ليه أوقات بنستخدم save()؟

في Spring Data JPA

```java
repository.save(user);
```

لها حالتين:

لو

```text
Entity جديدة
```

تعمل

```sql
INSERT
```

ولو

```text
Detached Entity
```

تعمل

```text
merge()
```

وترجعها Managed.

بعدها Dirty Checking يشتغل عليها.

---

# Dirty Checking و First-Level Cache

الاتنين مرتبطين جداً.

لأن

First-Level Cache

هو اللي بيخزن

```text
Entity

+

Snapshot
```

يعني

```text
Persistence Context

-------------------

Entity

Snapshot

-------------------
```

بدون

Persistence Context

مفيش

Dirty Checking.

---

# هل Dirty Checking ليه تكلفة؟

أيوه.

عند الـ Flush

Hibernate

يقارن

كل Managed Entities

بالـ Snapshots.

لو عندك

```text
100000 Entities
```

هيعمل

100000 مقارنة.

عشان كده

لو بتقرأ فقط

يفضل تستخدم

```java
@Transactional(readOnly = true)
```

أو

تفصل الـ Entity عن الـ Persistence Context باستخدام `detach()` لو مش محتاج تتابع تغييراتها.

---

# أشهر سؤال Interview

### هل لازم أستدعي `save()` بعد تعديل Entity داخل `@Transactional`؟

الإجابة:

**لا، إذا كانت الـ Entity ما زالت Managed داخل نفس الـ Transaction.**

مثال:

```java
@Transactional
public void updateName(Long id) {
    User user = repository.findById(id).orElseThrow();
    user.setName("Ali");
}
```

لا يوجد `save()`، ومع ذلك عند الـ Commit سيقوم Hibernate بعمل Dirty Checking ثم ينفذ:

```sql
UPDATE users
SET name = 'Ali'
WHERE id = ?;
```

---

# ملخص المقابلة (Interview Answer)

> **Dirty Checking** هو آلية داخل Hibernate تراقب الـ **Managed Entities** الموجودة داخل الـ **Persistence Context**. عند تحميل الـ Entity، يحتفظ Hibernate بنسخة أصلية منها (Snapshot). عند تنفيذ `flush()` أو `commit()` يقارن الحالة الحالية للـ Entity بالـ Snapshot. إذا اكتشف أي اختلاف، يعتبر الـ Entity **Dirty** ويولد تلقائيًا جملة `UPDATE` المناسبة دون الحاجة لاستدعاء `update()` أو `save()`، بشرط أن تكون الـ Entity ما زالت **Managed** داخل نفس الـ Session أو الـ Transaction.