# Part 1: Why Do Relational Databases Exist?

قبل ما نتكلم عن SQL و ACID لازم نفهم المشكلة.

تخيل أنك تبني:

- Amazon
    
- Uber
    
- Talabat
    
- Booking.com
    
- Banking System
    

كل هذه الأنظمة تحتوي على بيانات مرتبطة ببعضها:

```text
User
 ├── Orders
 │      ├── Order Items
 │      └── Payments
 └── Addresses
```

هنا يوجد:

- علاقات (Relationships)
    
- قيود (Constraints)
    
- قواعد عمل (Business Rules)
    

مثال:

```text
Order must belong to a valid user
```

لا يمكن إنشاء Order بدون User.

لهذا ظهرت Relational Databases.

---

# Why Not Just Store Everything in Files?

تخيل لديك:

```json
{
  "user":"Ali",
  "orders":[...]
}
```

بعد مليون مستخدم:

- البحث يصبح بطيئًا
    
- التعديل صعب
    
- العلاقات معقدة
    
- التزامن (Concurrency) كارثي
    

لذلك نحتاج Database Engine.

---

# Core Components

## Table

يشبه Excel Sheet.

```text
Users
+----+-------+---------+
| Id | Name  | Email   |
+----+-------+---------+
| 1  | Ali   | x@y.com |
| 2  | Ahmed | z@y.com |
+----+-------+---------+
```

---

## Row

سجل واحد.

```text
1 | Ali | x@y.com
```

---

## Column

خاصية معينة.

```text
Name
Email
Id
```

---

# Primary Key

## Problem

كيف أميز كل مستخدم؟

اسم المستخدم ليس فريدًا.

```text
Ali
Ali
Ali
```

مشكلة.

---

الحل:

```sql
UserId
```

مثال:

```text
1
2
3
```

فريد دائمًا.

---

خصائص الـ Primary Key:

```text
Unique
Not Null
Indexed
```

---

# Foreign Key

## Problem

كيف أعرف صاحب الطلب؟

Orders Table

```text
OrderId | UserId
```

هنا:

```text
UserId
```

يشير إلى:

```text
Users.Id
```

---

مثال:

Users

```text
1 Ali
2 Ahmed
```

Orders

```text
100 | 1
101 | 1
102 | 2
```

---

ASCII

```text
Users
  |
  |
  +---- Orders
```

---

# Relationships

## One To One

مثال:

```text
User
Passport
```

```text
User ---- Passport
```

مستخدم واحد يمتلك جوازًا واحدًا.

---

## One To Many

الأكثر شيوعًا.

```text
User
  |
  +---- Orders
```

User واحد لديه Orders كثيرة.

---

## Many To Many

مثال:

Learning Platform

```text
Student
Course
```

طالب يمكنه التسجيل في عدة كورسات.

وكورس واحد يحتوي عدة طلاب.

---

الحل:

```text
Student
Course
Enrollment
```

---

# Normalization

## المشكلة

نفترض:

```text
Orders

OrderId
CustomerName
CustomerEmail
CustomerPhone
```

كل Order يكرر البيانات.

---

مثال:

```text
Ali
Ali
Ali
Ali
Ali
```

تكرار ضخم.

---

## الحل

نفصل الجداول.

```text
Users
Orders
```

---

الفوائد:

✅ أقل تكرار

✅ أقل Storage

✅ أقل Bugs

---

Normal Forms

عادة في الصناعة:

```text
1NF
2NF
3NF
```

تكفي في 90% من الأنظمة.

---

# Denormalization

بعدما يكبر النظام تظهر مشكلة جديدة.

---

لنفترض:

Amazon

كل مرة نريد عرض Order.

نحتاج:

```sql
User
Order
Product
Category
Payment
```

خمسة Joins.

---

مكلف.

---

نكرر بعض البيانات عمدًا:

```text
Order

CustomerName
CustomerEmail
```

رغم وجودها في Users.

---

هذا يسمى:

```text
Denormalization
```

Trade-Off:

```text
Read Speed ↑
Storage ↑
Complexity ↑
```

---

# Indexes

أهم موضوع في قواعد البيانات.

---

بدون Index:

```sql
SELECT *
FROM Users
WHERE Email='a@b.com'
```

Database تعمل:

```text
Full Table Scan
```

تفحص كل الصفوف.

---

لو لدينا:

```text
100 Million Users
```

كارثة.

---

مع Index:

```sql
CREATE INDEX idx_email
ON Users(email);
```

تصبح العملية:

```text
O(log n)
```

بدل:

```text
O(n)
```

---

تشبيه:

فهرس الكتاب.

---

# Joins

الـ Database لا تخزن كل شيء في جدول واحد.

---

Users

```text
1 Ali
```

Orders

```text
100 1
```

---

لجلب البيانات:

```sql
SELECT *
FROM Users U
JOIN Orders O
ON U.Id = O.UserId
```

---

أنواع مهمة:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL JOIN
```

في المقابلات:

ركز على

```text
INNER
LEFT
```

---

# Transactions

وصلنا لأهم نقطة.

---

تخيل Banking System.

علي يحول:

```text
100$
```

إلى أحمد.

---

يجب تنفيذ خطوتين:

```sql
Deduct 100
Add 100
```

---

ماذا لو:

```text
Deduct succeeded
Add failed
```

اختفت الأموال.

---

هنا ظهر مفهوم:

```text
Transaction
```

---

# What Is A Transaction?



مجموعة عمليات تنجح كلها أو تفشل كلها.

```sql
BEGIN TRANSACTION

UPDATE Accounts
SET Balance=Balance-100
WHERE Id=1;

UPDATE Accounts
SET Balance=Balance+100
WHERE Id=2;

COMMIT;
```

أو:

```sql
ROLLBACK;
```

---

# ACID

كل Relational Database محترمة تطبق ACID.

# A = Atomicity

الفكرة:

```text
All or Nothing
```

إما كل العملية أو لا شيء.

---

Bank Transfer

```text
Deduct
Add
```

لو الثانية فشلت:

```sql
ROLLBACK
```

---

بدون Atomicity:

```text
Money Lost
```

---

# C = Consistency

يحافظ على قواعد النظام.

مثال:

الرصيد لا يمكن أن يصبح:

```text
-100000
```

إذا كانت القاعدة تمنع ذلك.

---

قبل Transaction:

```text
Valid State
```

بعد Transaction:

```text
Valid State
```

أيضًا.

---

# I = Isolation

الموضوع متعلق بالـ Concurrency.

تخيل:

```text
1000 users
```

يشترون نفس المنتج.

---

يجب ألا تؤثر المعاملات على بعضها بطريقة خاطئة.

---

# D = Durability

بعد:

```sql
COMMIT
```

انتهى الأمر.

حتى لو:

```text
Power Failure
Server Crash
```

البيانات محفوظة.

---

Database تستخدم:

```text
Transaction Logs
Write Ahead Log (WAL)
```

لحماية البيانات.

---

# Isolation Levels

هنا يدخل System Design الحقيقي.

---

# Read Uncommitted

الأسرع.

يسمح بقراءة بيانات لم يتم Commit لها.

مشكلة:

```text
Dirty Reads
```

---

# Read Committed

الأكثر استخدامًا.

لا يقرأ البيانات غير المعتمدة.

---

# Repeatable Read

إذا قرأت صفًا مرتين داخل نفس Transaction.

ستحصل على نفس النتيجة.

---

# Serializable

الأكثر أمانًا.

يتصرف وكأن جميع المعاملات تنفذ واحدة تلو الأخرى.

---

Trade-Off

```text
Safety ↑
Concurrency ↓
Performance ↓
```

---

# Common Concurrency Problems

## Dirty Read

Transaction A:

```text
Balance = 500
```

تغيره إلى:

```text
1000
```

ولم تعمل Commit.

---

Transaction B قرأت:

```text
1000
```

ثم A عملت Rollback.

---

B قرأت بيانات لم تكن حقيقية.

---

## Non Repeatable Read

قرأت:

```text
500
```

ثم Transaction أخرى عدلت القيمة.

---

قرأت مرة أخرى:

```text
700
```

داخل نفس Transaction.

---

# Phantom Read

أول Query:

```sql
SELECT *
FROM Orders
WHERE Status='Pending'
```

وجدت:

```text
10 rows
```

---

عملية أخرى أضافت Order جديد.

---

نفس Query:

```text
11 rows
```

ظهر "شبح" جديد.

---

# Lost Update

أخطر مشكلة في الأنظمة.

---

علي وأحمد يقرآن:

```text
Balance=100
```

في نفس اللحظة.

---

علي:

```text
100 -> 80
```

---

أحمد:

```text
100 -> 50
```

---

النتيجة النهائية:

```text
50
```

بدل:

```text
30
```

---

هذا يسمى:

```text
Lost Update
```

---

# Scaling Relational Databases

عندما يكبر النظام.

---

# Vertical Scaling

```text
More CPU
More RAM
```

سهل.

لكن له حد.

---

# Read Replicas

```text
          Master
             |
      ----------------
      |              |
 Replica1      Replica2
```

الكتابة على:

```text
Master
```

القراءة من:

```text
Replicas
```

---

ميزة:

```text
Read Throughput ↑
```

---

عيب:

```text
Replication Lag
```

---

# Partitioning

تقسيم الجدول داخليًا.

```text
Orders_2024
Orders_2025
Orders_2026
```

---

# Sharding

تقسيم قاعدة البيانات نفسها.

```text
Shard1
Users 1-1M

Shard2
Users 1M-2M
```

---

Trade-Off:

```text
Scalability ↑
Complexity ↑↑↑
```

---

# متى لا تستخدم Relational Databases؟

عندما تحتاج:

- مليارات الأحداث (Logs)
    
- بيانات غير منظمة
    
- Schema متغير باستمرار
    
- Social Media Feed
    
- Real-time Analytics
    

حينها نستخدم أحيانًا قواعد NoSQL مثل:

- MongoDB
    
- Cassandra
    
- Redis
    

---

# أسئلة مقابلات مهمة

### ما الفرق بين Primary Key و Foreign Key؟

- Primary Key يميز الصف بشكل فريد.
    
- Foreign Key يربط جدولًا بآخر.
    

---

### لماذا نستخدم Index؟

لتسريع عمليات البحث وتقليل Full Table Scan.

---

### ما هي ACID؟

- Atomicity
    
- Consistency
    
- Isolation
    
- Durability
    

---

### ما الفرق بين Read Replica و Sharding؟

Read Replica:

- لحل مشكلة القراءة.
    

Sharding:

- لحل مشكلة الحجم والكتابة والتوسع الأفقي.
    

---

### لماذا Serializable ليس المستوى الافتراضي؟

لأنه يوفر أعلى أمان لكنه يقلل الـ Throughput والـ Concurrency بشكل كبير.

---

# تمرين System Design

صمم قاعدة بيانات لـ Food Delivery App تحتوي على:

```text
Users
Restaurants
MenuItems
Orders
OrderItems
Payments
Drivers
```

ثم حدد:

1. Primary Keys.
    
2. Foreign Keys.
    
3. العلاقات.
    
4. أين ستحتاج Transaction؟
    
5. أين ستضيف Indexes؟
    
6. لو وصل النظام إلى 100 مليون Order، هل ستستخدم Read Replicas أم Sharding أم الاثنين؟
    

جاوب على التمرين، وأنا سأراجعه معك كأنها جلسة System Design Interview حقيقية.