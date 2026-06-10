

---

# ما هي Transaction أصلاً؟

الـ Transaction هي مجموعة عمليات على قاعدة البيانات تعتبر **وحدة واحدة (Unit of Work)**.

مثال:

```sql
BEGIN TRANSACTION

UPDATE Accounts
SET Balance = Balance - 100
WHERE Id = 1

UPDATE Accounts
SET Balance = Balance + 100
WHERE Id = 2

COMMIT
```

إحنا هنا بنحول 100 جنيه من حساب لحساب.

لو أول Update نجح والثاني فشل:

```sql
ROLLBACK
```

ويرجع كل شيء كما كان.

هنا يأتي دور ACID.

---

# A = Atomicity

> Either everything succeeds or everything fails.

كل العملية تنجح أو كلها تفشل.

---

## مثال بنكي

عندك:

```text
Ahmed = 1000$
Ali = 500$
```

تحويل:

```text
Ahmed -> Ali = 100$
```

الخطوات:

```sql
UPDATE Accounts
SET Balance = Balance - 100
WHERE Id = Ahmed

UPDATE Accounts
SET Balance = Balance + 100
WHERE Id = Ali
```

تخيل الكهرباء فصلت بعد أول Update.

سيصبح:

```text
Ahmed = 900
Ali = 500
```

الـ 100 اختفوا!

Atomicity تمنع هذا.

---

## في Spring

```java
@Transactional
public void transferMoney() {

    withdraw();

    deposit();
}
```

إذا فشلت:

```java
deposit();
```

سيتم:

```java
ROLLBACK
```

للعملية كلها.
لو عمليه واحده فشلت كله هيفشل 

---

# C = Consistency

> Database always remains valid.

البيانات تظل صحيحة.

---

## مثال

ممنوع أن يصبح الرصيد بالسالب. 

```java
if(balance < amount)
    throw new Exception();
```

قبل الـ Transaction:

```text
Balance = 100
```

بعد الـ Transaction:

```text
Balance = 50
```

أو

```text
Balance = 100
```

لكن ليس:

```text
Balance = -500
```

---

## مثال E-commerce

عندك:

```text
Stock = 5
```

جاء طلب 10 منتجات.

المفروض النظام يرفض.

Consistency تمنع:

```text
Stock = -5
```

---
# ISOLATION
# تخيل معايا

عندك حساب بنكي فيه:

```text
Balance = 1000
```

وفي نفس اللحظة وصل Requestين:

### Request A

عايز يسحب 800 جنيه.

### Request B

عايز يسحب 500 جنيه.

---

لو مفيش Isolation:

### Transaction A

تقرأ:

```text
Balance = 1000
```

---

### Transaction B

تقرأ أيضًا:

```text
Balance = 1000
```

(لأن A لم تنتهِ بعد)

---

A تخصم 800:

```text
Balance = 200
```

---

B تخصم 500 بناءً على القراءة القديمة:

```text
Balance = 500
```

أو أحيانًا ينتهي الأمر برصيد:

```text
Balance = -300
```

حسب طريقة التنفيذ.

وهنا حصلت مشكلة اسمها:

> Race Condition

---

# إذن ما هو Isolation؟

ببساطة:

> كل Transaction تتصرف كأنها وحدها في قاعدة البيانات.

يعني لما A شغالة، لا تجعل B ترى بيانات نصف مكتملة أو تتدخل بطريقة تفسد البيانات.

---

# مثال من حياتك

تخيل إنك داخل البنك.

الموظف بيحدث حسابك.

وأنت واقف.

هل يسمح لموظف آخر أن يعدل نفس الحساب في نفس اللحظة؟

طبعًا لا.

ينتظر حتى ينتهي الأول.

هذا تقريبًا هو مفهوم Isolation.

---

# لماذا نحتاجه؟

لأن قواعد البيانات الحقيقية فيها آلاف المستخدمين في نفس الوقت.

مثال:

- شخصان يحجزان آخر تذكرة.
    
- شخصان يشترون آخر منتج.
    
- شخصان يسحبان من نفس الحساب.
    

بدون Isolation ستحدث كوارث.

---

# مثال E-Commerce

المخزون:

```text
iPhone Stock = 1
```

أنت وأنا ضغطنا:

```text
Buy Now
```

في نفس الثانية.

---

كل واحد قرأ:

```text
Stock = 1
```

---

كل واحد أكمل الطلب.

الناتج:

```text
Stock = -1
```

لقد بعنا هاتفين بينما لدينا واحد فقط.

---

Isolation يمنع هذا.

---

# كيف تمنعه قاعدة البيانات؟

غالبًا باستخدام:

## Locks

عندما تبدأ Transaction:

```sql
SELECT * 
FROM Products
WHERE Id = 1
FOR UPDATE
```

قاعدة البيانات تقول:

> هذا الصف محجوز الآن.

أي Transaction أخرى تنتظر.

---

مثال:

```text
Transaction A
    |
    |---- Lock Product
    |
    |---- Update Stock
    |
    |---- Commit
    |
Transaction B يبدأ الآن
```

فلا يحدث تضارب.

---

# ماذا يحدث لو لم نستخدم Isolation جيدًا؟

تظهر مشاكل مشهورة جدًا.

## 1. Dirty Read

أنا أرى بيانات لم يتم حفظها بعد.

مثال:

A تغير الرصيد إلى:

```text
5000
```

لكن لم تعمل Commit.

B يقرأ:

```text
5000
```

ثم A تعمل Rollback.

فيكتشف B أنه قرأ قيمة وهمية.

---

## 2. Non-Repeatable Read

أقرأ نفس السجل مرتين فأجد قيمتين مختلفتين.

مثال:

```text
Balance = 1000
```

بعد ثانية:

```text
Balance = 1200
```

رغم أنني داخل نفس Transaction.

---

## 3. Phantom Read

أقرأ عدد الطلبات:

```text
10 Orders
```

ثم أقرأ مرة أخرى:

```text
11 Orders
```

لأن أحدهم أضاف Order أثناء عملي.

---

# ما الذي يهمك كمطور Backend؟

في معظم المشاريع:

```java
@Transactional
public void createOrder()
{
   ...
}
```

ستعتمد على Isolation الافتراضي لقاعدة البيانات.

في PostgreSQL الافتراضي:

```text
READ COMMITTED
```

وفي أغلب الأنظمة يكون كافيًا.

لكن عندما تتعامل مع:

- Wallets
    
- Payments
    
- Banking
    
- Inventory
    

يجب أن تفكر في:

- Locks
    
- Isolation Levels
    
- Optimistic Locking
    
- Pessimistic Locking
    

---

# جملة مقابلات مهمة جدًا

لو سألك المحاور:

> What problem does Isolation solve?

جاوب:

> Isolation prevents concurrent transactions from interfering with each other. It ensures that multiple users can access and modify data at the same time without causing inconsistencies such as dirty reads, lost updates, or overselling inventory.

ولو عايز أفهمك بعد كده **Optimistic Locking vs Pessimistic Locking** (وهي أكثر حاجة تُسأل بعد Isolation في مقابلات الـ Backend)، أشرحها لك برسومات وأمثلة من الـ E-commerce والـ Banking.
---

# D = Durability

> Once committed, data is permanent.

---

مثال:

```sql
COMMIT
```

تم التنفيذ.

بعدها مباشرة:

```text
Server crashed
Power failure
```

عندما تعود قاعدة البيانات:

```text
Data still exists
```

كيف؟

---

## Write Ahead Log (WAL)

PostgreSQL مثلاً يكتب أولاً في:

```text
WAL Log
```

ثم يكتب للـ Data Files.

لذلك يستطيع استرجاع البيانات بعد الكراش.

---

# سؤال Interview

## لماذا SQL وليس NoSQL؟

---

لو نظام:

```text
Bank
Payments
Wallet
Inventory
Orders
```

أختار:

```text
SQL
```

لأنها تدعم:

- ACID
    
- Transactions
    
- Consistency
    

---

أما لو:

```text
Instagram Feed
Comments
Likes
Analytics
Logs
```

غالباً:

```text
NoSQL
```

أفضل.

---

# سؤال Interview

## كيف تضمن أن الرصيد لا يخصم مرتين؟

---

المشكلة:

ضغط المستخدم مرتين.

```text
Pay
Pay
Pay
```

وصلت 3 Requests.

---

الحل الأول:

Idempotency Key

```text
payment_123
```

إذا جاء نفس المفتاح:

```text
Ignore Duplicate Request
```

---

الحل الثاني:

Database Transaction

```java
@Transactional
public void pay()
{
   deductBalance();
   createPayment();
}
```

---

# سؤال Interview

## ماذا لو السيرفر وقع أثناء الدفع؟

---

بدون Transaction:

```text
Balance deducted
Payment not created
```

كارثة.

---

مع Transaction:

```text
ROLLBACK
```

ولا يحدث أي تغيير.

---

# سؤال Interview

## كيف تضمن إنشاء Order وخصم المخزون معاً؟

---

```java
@Transactional
public void createOrder()
{
    orderRepository.save(order);

    inventoryRepository.decreaseStock();

    paymentRepository.createPayment();
}
```

إذا فشل أي جزء:

```text
ROLLBACK
```

لكل شيء.

---

# المشكلة في Microservices

لو عندك:

```text
Order Service
Payment Service
Inventory Service
```

كل Service لها Database مستقلة.

هنا Transaction عادية لا تكفي.

---

نستخدم:

## Saga Pattern

مثال:

```text
Create Order
↓
Deduct Inventory
↓
Take Payment
```

إذا فشل الدفع:

```text
Refund Inventory
Cancel Order
```

وهذه تسمى:

```text
Compensating Transactions
```

---

# ما يجب أن تحفظه للمقابلة

إذا سألك:

> Explain ACID.

جاوب:

**Atomicity**

- All or Nothing.
    

**Consistency**

- Database remains valid.
    

**Isolation**

- Concurrent transactions don't interfere.
    

**Durability**

- Committed data survives crashes.
    

ثم أعطِ مثال تحويل أموال بين حسابين، لأنه يشرح الأربع خصائص دفعة واحدة.

هذه بالضبط هي النقطة التي ينتقل منها أغلب مهندسي الـ Backend من مجرد معرفة قواعد البيانات إلى فهم System Design الحقيقي، لأن معظم مشاكل الأنظمة الكبيرة تدور حول: **Consistency, Concurrency, Reliability, and Distributed Transactions**.