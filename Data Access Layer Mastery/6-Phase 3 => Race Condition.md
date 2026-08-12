تمام يا علي. بما إنك خلصت **المرحلة 1 و2**، فالمرحلة 3 هي فعلًا من أهم أجزاء الـ JPA/Database بالنسبة ليك، خصوصًا إنك شغال على **Stock Movement / Inventory / Ledger**؛ لأن أي خطأ هنا ممكن يخلي الـ stock نفسه يبقى غلط.

هنشرحها كأنها **Interview + Production** في نفس الوقت، وبالترتيب:

> **Lost Update → Optimistic Locking → Pessimistic Locking → Isolation Levels → Deadlocks → إزاي تختار الحل**

---

# المرحلة 3 — Race Conditions & Concurrency في JPA

## 0. الأول: يعني إيه Race Condition؟

ببساطة:

> **==Race Condition بتحصل لما أكتر من Thread / Transaction يحاولوا يتعاملوا مع نفس الـ data في نفس الوقت، والنتيجة تعتمد على مين سبق مين.**==

مثال عندك:

```text
SKU: Laptop
Current Stock = 10
```

وفي نفس اللحظة جالك طلبين:

```text
Order A → يخصم 7
Order B → يخصم 5
```

منطقيًا:

```text
10 - 7 - 5 = -2
```

يعني المفروض العملية التانية تترفض لو الـ business rule بيمنع stock سالب.

لكن لو مفيش concurrency control، ممكن يحصل شيء أخطر.

---

# 1. Lost Update Problem

دي أهم نقطة تفهمها قبل أي Locking.

## السيناريو

عندنا:

```text
stock = 10
```

Transaction A:

```text
Read stock → 10
```

Transaction B:

```text
Read stock → 10
```

بعد كده:

```text
A: 10 - 7 = 3
B: 10 - 5 = 5
```

وبعدين:

```text
A → UPDATE stock = 3
B → UPDATE stock = 5
```

النتيجة النهائية:

```text
stock = 5
```

لكن المفروض:

```text
stock = -2
```

أو على الأقل العملية التانية تترفض.

### إيه اللي حصل؟

Transaction B **دهست نتيجة Transaction A**.

وده اسمه:

> **Lost Update**

---

# 2. ليه Java synchronized مش كفاية؟

دي نقطة Interview مهمة جدًا.

ممكن حد يقول:

```java
public synchronized void decreaseStock(...) {
    ...
}
```

هل كده المشكلة اتحلت؟

**لا.**

لأن:

```java
synchronized
```

بيحمي الـ threads داخل **JVM واحدة**.

لكن تخيل عندك:

```text
Application Instance 1
        ↓
     Database
        ↑
Application Instance 2
```

كل Instance ليها JVM مختلفة.

فـ:

```text
Instance 1 → synchronized
Instance 2 → synchronized
```

الاتنين مش شايفين بعض.

عشان كده concurrency control في الـ database لازم يكون على مستوى الـ **Database/Transaction** لما البيانات مشتركة بين instances.

---

# 3. Transaction نفسها مهمة جدًا

قبل ما نتكلم عن locks، لازم نفهم الـ transaction.

مثلاً:

```java
@Transactional
public void decreaseStock(UUID skuId, int quantity) {

    Sku sku = repository.findById(skuId)
            .orElseThrow();

    sku.decrease(quantity);

    movementRepository.save(
        new StockMovement(...)
    );
}
```

العملية دي بالنسبة للـ database المفروض تبقى:

```text
BEGIN

SELECT sku...

UPDATE sku...

INSERT stock_movement...

COMMIT
```

لو حاجة حصلت:

```text
ROLLBACK
```

يعني يا إما العملية كلها تنجح، يا إما كلها تفشل.

---

# 4. Optimistic Locking

دلوقتي أول حل للـ Lost Update.

الفكرة:

> **أنا مش همنع transaction تانية من قراءة الـ record، لكن هتحقق قبل الـ UPDATE إن البيانات اللي قرأتها لسه هي نفسها.**

وده اسمه:

> Optimistic Locking

---

# 5. استخدام `@Version`

مثلاً:

```java
@Entity
public class Sku {

    @Id
    private UUID id;

    private String name;

    private int currentQuantity;

    @Version
    private Long version;
}
```

Hibernate هيضيف concept اسمه:

```text
version
```

مثلاً:

```text
id       stock    version
-------------------------
1        10       5
```

---

# 6. Optimistic Locking بيشتغل إزاي؟

Transaction A:

```text
SELECT stock, version
FROM sku
WHERE id = 1
```

ترجع:

```text
stock = 10
version = 5
```

Transaction B تعمل نفس الكلام:

```text
stock = 10
version = 5
```

A تعدل:

```text
stock = 3
```

Hibernate يعمل حاجة شبه:

```sql
UPDATE sku
SET current_quantity = 3,
    version = 6
WHERE id = 1
AND version = 5;
```

الـ update نجح.

دلوقتي:

```text
version = 6
```

---

Transaction B كانت عندها:

```text
version = 5
```

فلما تحاول تعمل:

```sql
UPDATE sku
SET current_quantity = 5,
    version = 6
WHERE id = 1
AND version = 5;
```

الـ database هتقول:

```text
0 rows updated
```

لأن:

```text
version = 6
```

مش:

```text
version = 5
```

Hibernate يعرف إن حد تاني عدل الـ entity، وبالتالي يرمي:

```java
OptimisticLockException
```

---

# 7. ليه اسمه Optimistic؟

لأنك **متفائل** إن الـ conflict مش هيحصل.

يعني بتقول:

> "غالبًا محدش هيعدل نفس الـ record وأنا شغال." فبنستخدمه في ال systems that hasnot a lot of concurrent updates 

فبدل ما أقفل الـ record، هسمح للناس تقرأ وتشتغل، ولو حصل conflict أكتشفه في الآخر.

---

# 8. إمتى Optimistic Locking مناسب؟

مناسب لما:

```text
conflicts are rare
```

مثلاً:

```text
User Profile
Product Description
Course Information
Employee Information
```

لو اتنين عدلوا اسم الـ course في نفس اللحظة، مش سيناريو بيحصل باستمرار.

---

# 9. مشكلة Optimistic Locking

لو عندك:

```text
1000 requests/sec
```

كلهم بيعدلوا نفس:

```text
SKU stock
```

هتلاقي conflicts كتير جدًا.

ساعتها Optimistic Locking ممكن يؤدي إلى:

```text
OptimisticLockException
OptimisticLockException
OptimisticLockException
...
```

وممكن تحتاج retry mechanism.

---

# 10. Pessimistic Locking

هنا الفكرة مختلفة.

بدل ما أقول:

> "خلينا نجرب وربنا يستر."

بقول:

> **"أنا هقفل الـ row، ومحدش يعدله لحد ما أخلص."**

وده:

> **Pessimistic Locking**

---

# 11. `PESSIMISTIC_WRITE`

في Spring Data JPA: 

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("""
    SELECT s
    FROM Sku s
    WHERE s.id = :id
""")
Optional<Sku> findByIdForUpdate(@Param("id") UUID id);
```

الفكرة إن Hibernate هيطلب من الـ database lock على الـ row.

في PostgreSQL غالبًا ده يترجم لمفهوم:

```sql
SELECT *
FROM skus
WHERE id = ?
FOR UPDATE;
```

---

# 12. السيناريو بتاع الـ Stock

عندنا:

```text
stock = 10
```

Transaction A:

```text
SELECT ... FOR UPDATE
```

تمسك الـ row.

Transaction B:

```text
SELECT ... FOR UPDATE
```

تحاول تمسك نفس الـ row.

لكن A ماسكاه.

فـ B تستنى.

---

## A

```text
Read stock = 10

Decrease 7

stock = 3

COMMIT
```

الـ lock يتحرر.

---

## B

دلوقتي B تكمل.

تقرأ:

```text
stock = 3
```

تحاول تخصم:

```text
3 - 5
```

الـ business rule يقول:

```text
Not enough stock
```

فتترفض العملية.

وده بالضبط اللي أنت محتاجه في الـ inventory.

---

# 13. الفرق الجوهري بين Optimistic و Pessimistic

|Optimistic|Pessimistic|
|---|---|
|لا يقفل الـ row غالبًا|يقفل الـ row|
|يسمح بالـ concurrency|يقلل الـ concurrency|
|يكتشف الـ conflict|يمنع conflict أثناء الانتظار|
|يحتاج handling للـ exception|ممكن transaction تستنى|
|مناسب للـ conflicts النادرة|مناسب للـ conflicts المتكررة|
|`@Version`|`PESSIMISTIC_WRITE`|

احفظ الجملة دي للـ Interview:

> **Optimistic locking detects conflicts, while pessimistic locking prevents concurrent modifications by acquiring a database lock.**

---

# 14. مهم جدًا: Lock لوحده مش كفاية

بص على الكود ده:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Sku> findByIdForUpdate(UUID id);
```

هل كده خلاص؟

**لا.**

لازم يكون داخل Transaction.

مثلاً:

```java
@Transactional
public void decreaseStock(UUID skuId, int quantity) {

    Sku sku = repository.findByIdForUpdate(skuId)
            .orElseThrow();

    sku.decrease(quantity);

    movementRepository.save(...);
}
```

ليه؟

لأن الـ lock بيكون مرتبط بالـ database transaction.

والـ transaction هي اللي في الآخر تعمل:

```text
COMMIT
```

فيتحرر الـ lock.

---

# 15. ليه `@Transactional` مهمة جدًا هنا؟

تخيل:

```java
@Transactional
public void decreaseStock(...) {

    Sku sku = findByIdForUpdate();

    sku.decrease(...);

    saveMovement();

}
```

الـ lifecycle:

```text
BEGIN
  ↓
SELECT ... FOR UPDATE
  ↓
lock acquired
  ↓
business logic
  ↓
UPDATE
  ↓
INSERT movement
  ↓
COMMIT
  ↓
lock released
```

دي العملية الصح.

---

# 16. دلوقتي ندخل على Isolation Levels

دي من أهم الحاجات في المرحلة دي.

الـ database ممكن يكون عندها أكتر من transaction شغالين في نفس الوقت.

السؤال:

> Transaction تشوف إيه من Transaction تانية؟

ده اللي الـ **Isolation Level** بيتحكم فيه.

---

# 17. أشهر مشاكل الـ Concurrency

لازم تعرف الأربع مصطلحات دول:

### 1. Dirty Read

Transaction A تعدل data لكن **لسه ماعملتش commit**.

Transaction B تقرأ التعديل ده.

بعد كده A تعمل:

```text
ROLLBACK
```

B تكون قرأت data عمرها ما اتثبتت.

---

### 2. Non-Repeatable Read

Transaction A تقرأ:

```text
stock = 10
```

Transaction B تعدل:

```text
stock = 5
```

وتعمل:

```text
COMMIT
```

Transaction A تقرأ تاني:

```text
stock = 5
```

نفس transaction قرأت نفس الـ row مرتين وطلعلها قيمتين مختلفتين.

---

### 3. Phantom Read

Transaction A تعمل:

```sql
SELECT *
FROM orders
WHERE status = 'PENDING';
```

طلعلها:

```text
10 orders
```

Transaction B تعمل:

```sql
INSERT INTO orders ...
```

وتعمل:

```text
COMMIT
```

Transaction A تعمل نفس query تاني:

```text
11 orders
```

ظهر record جديد داخل نفس transaction.

ده Phantom Read.

---

### 4. Lost Update

اتكلمنا عنه:

```text
A reads 10
B reads 10

A writes 3
B writes 5

A's update is lost
```

---

# 18. Isolation Levels

عندك أربعة مستويات مشهورة:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

كل ما تطلع:

```text
Isolation ↑
```

عادةً:

```text
Consistency ↑
Concurrency ↓
```

لكن التفاصيل تعتمد على database implementation.

---

# 19. READ_UNCOMMITTED

أقل مستوى.

يسمح إن transaction تقرأ data لسه مش committed.

يعني ممكن يحصل:

```text
Dirty Read
```

مثال:

```text
A:
stock = 10
UPDATE → 5
(no commit)

B:
SELECT stock
→ 5

A:
ROLLBACK

Actual stock = 10
```

B قرأت قيمة غير حقيقية.

---

# 20. READ_COMMITTED

ده مهم جدًا.

معناه:

> Transaction تشوف فقط البيانات اللي اتعملها commit.

في PostgreSQL ده هو الـ default isolation level.

مثلاً:

```text
A:
UPDATE stock = 5
(no commit)

B:
SELECT stock
→ 10
```

بعد:

```text
A → COMMIT
```

لو B عملت SELECT تاني:

```text
→ 5
```

إذن Dirty Read ممنوع.

لكن:

> Non-repeatable reads ممكنة.

---

# 21. REPEATABLE_READ

هنا بنقول:

> لو قرأت row مرة، نفس transaction هتفضل شايفة نفس القيمة المنطقية اللي كانت متاحة لها وفقًا للـ isolation semantics.

مثلاً:

```text
Transaction A

SELECT stock
→ 10
```

Transaction B:

```text
UPDATE stock = 5
COMMIT
```

A تقرأ تاني:

```text
→ 10
```

بدل ما تشوف:

```text
5
```

---

# 22. SERIALIZABLE

أعلى isolation level.

الفكرة:

> transactions تتصرف وكأنها اتنفذت واحدة وراء التانية.

يعني:

```text
Transaction A
      ↓
Transaction B
```

مش:

```text
A + B simultaneously
```

وده أقوى consistency، لكن ممكن يقلل performance ويزود:

```text
blocking
serialization failures
deadlocks/retries
```

عشان كده مش بتستخدمه في كل حاجة.

---

# 23. جدول مهم جدًا للـ Interview

|Isolation|Dirty Read|Non-repeatable Read|Phantom Read|
|---|---|---|---|
|READ UNCOMMITTED|ممكن|ممكن|ممكن|
|READ COMMITTED|❌|ممكن|ممكن|
|REPEATABLE READ|❌|❌|حسب الـ DB|
|SERIALIZABLE|❌|❌|❌|

**مهم:** تفاصيل `REPEATABLE_READ` خصوصًا الـ phantom reads تختلف حسب الـ database engine. PostgreSQL له MVCC behavior خاص به، فلا تحفظ الجدول كأنه قانون عالمي لكل databases.

---

# 24. PostgreSQL + Spring

في Spring:

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void process() {
}
```

أو:

```java
@Transactional(
    isolation = Isolation.REPEATABLE_READ
)
```

أو:

```java
@Transactional(
    isolation = Isolation.SERIALIZABLE
)
```

لكن:

> **متغيرش isolation level لمجرد إنك تقدر.**

استخدم أقل مستوى يحقق الـ business requirements.

---

# 25. طب إيه الفرق بين Locking و Isolation؟

دي نقطة Interview ممتازة.

### Isolation

بتحدد:

> **Transaction تشوف إيه من transaction تانية؟**

### Locking

بيحدد:

> **مين يقدر يعدل/يقرأ resource معين وإمتى؟**

ممكن تستخدم الاتنين مع بعض.

مثلاً:

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void decreaseStock(...) {

    skuRepository.findByIdForUpdate(...);

}
```

هنا عندك:

```text
Isolation Level
+
Pessimistic Lock
```

---

# 26. Deadlocks

دلوقتي دخلنا في آخر جزء مهم.

الـ Deadlock يحصل لما Transaction A تستنى B، وB تستنى A.

مثال:

### Transaction A

تمسك:

```text
SKU 1
```

وبعدين تحاول تمسك:

```text
SKU 2
```

### Transaction B

تمسك:

```text
SKU 2
```

وبعدين تحاول تمسك:

```text
SKU 1
```

فتبقى:

```text
A → waiting for SKU 2
B → waiting for SKU 1
```

ولا واحدة تقدر تكمل.

---

# 27. شكلها

```text
Transaction A
    |
    | locks SKU 1
    ↓
 waits for SKU 2
    ↑
    |
Transaction B
    |
    | locks SKU 2
    ↓
 waits for SKU 1
```

Database تكتشف الـ deadlock وتعمل rollback لإحدى الـ transactions.

---

# 28. إزاي تمنع Deadlocks؟

## 1. Consistent Lock Ordering

دي أهم واحدة.

لو عندك:

```text
SKU 1
SKU 2
```

خلي كل transactions تاخد locks بالترتيب:

```text
SKU 1 → SKU 2
```

ومتعملش:

```text
A: SKU 1 → SKU 2

B: SKU 2 → SKU 1
```

خلي:

```text
A: SKU 1 → SKU 2
B: SKU 1 → SKU 2
```

كده B هتستنى A بدل ما يحصل cycle.

---

# 29. خلي Transactions قصيرة

غلط:

```java
@Transactional
public void process() {

    lockSku();

    callExternalAPI();

    sendEmail();

    generateReport();

    updateDatabase();
}
```

أنت ماسك lock أثناء:

```text
HTTP call
Email
Report generation
```

ده سيئ جدًا.

الأفضل:

```text
Prepare data
   ↓
Transaction
   ↓
Lock
   ↓
Update
   ↓
Commit
```

وخلاص.

---

# 30. متعملش External API Calls جوه Transaction مقفولة

مثلاً:

```java
@Transactional
public void updateStock() {

    Sku sku = findForUpdate();

    paymentService.callExternalAPI(); // ❌

    sku.decrease();

}
```

ليه سيء؟

لأن الـ external API ممكن ياخد:

```text
5 seconds
```

وأنت خلال الخمس ثواني:

```text
row locked
```

فكل requests اللي محتاجة نفس الـ SKU ممكن تستنى.

---

# 31. Retry

حتى لو عملت كل حاجة صح، في systems كبيرة ممكن يحصل:

```text
Deadlock
OptimisticLockException
Serialization failure
```

فممكن تستخدم retry strategy في العمليات المناسبة.

مثلاً:

```text
Try
 ↓
Transaction
 ↓
Conflict
 ↓
Rollback
 ↓
Retry
```

لكن خلي بالك:

> **Retry مش حل سحري.**

لازم العملية تكون مناسبة للـ retry، ويفضل يكون عندك:

```text
limited retries
+
backoff
```

---

# 32. دلوقتي نطبق على Inventory System بتاعك

افترض:

```java
@Entity
public class Sku {

    @Id
    private UUID id;

    private int currentQuantity;

    private int reorderThreshold;
}
```

وعندنا:

```text
currentQuantity = 100
```

جالك:

```text
Stock Movement
type = OUT
quantity = 30
```

---

## الطريقة الغلط

```java
@Transactional
public void decrease(UUID id, int quantity) {

    Sku sku = repository.findById(id)
            .orElseThrow();

    if (sku.getCurrentQuantity() < quantity) {
        throw new InsufficientStockException();
    }

    sku.setCurrentQuantity(
        sku.getCurrentQuantity() - quantity
    );
}
```

لو requests كتير وصلت في نفس الوقت:

```text
A reads 100
B reads 100

A → 70
B → 50
```

آخر update يكسب.

**Lost Update.**

---

# 33. الحل بـ Pessimistic Lock

Repository:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("""
    SELECT s
    FROM Sku s
    WHERE s.id = :id
""")
Optional<Sku> findByIdForUpdate(UUID id);
```

Service:

```java
@Transactional
public void decrease(UUID id, int quantity) {

    Sku sku = repository.findByIdForUpdate(id)
            .orElseThrow();

    if (sku.getCurrentQuantity() < quantity) {
        throw new InsufficientStockException();
    }

    sku.setCurrentQuantity(
        sku.getCurrentQuantity() - quantity
    );
}
```

دلوقتي:

```text
Request A
   ↓
LOCK SKU
   ↓
read 100
   ↓
70
   ↓
COMMIT
   ↓
UNLOCK
```

بعدها:

```text
Request B
   ↓
LOCK SKU
   ↓
read 70
   ↓
...
```

فأنت بقيت بتتعامل مع **الحالة الحالية الحقيقية** للـ stock.

---

# 34. طب Optimistic Version في نفس السيناريو؟

ممكن:

```java
@Entity
public class Sku {

    @Version
    private Long version;
}
```

وتعمل:

```java
@Transactional
public void decrease(...) {

    Sku sku = repository.findById(id)
            .orElseThrow();

    sku.decrease(quantity);
}
```

لو حصل conflict:

```text
OptimisticLockException
```

ممكن تعمل:

```text
Retry
```

وتعيد العملية بناءً على أحدث stock.

---

# 35. أختار إيه في الـ Inventory؟

في حالة زي:

```text
Stock
Balance
Bank Account
Wallet
Inventory Quantity
Counters
```

غالبًا الـ contention ممكن يكون عالي، والـ correctness critical.

فـ:

> **Pessimistic locking is often a strong choice when many concurrent transactions update the same row.**

لكن مش معنى كده إن:

```text
Inventory = Pessimistic always
```

لا.

اختيارك يعتمد على:

```text
contention
+
transaction duration
+
business requirements
+
database behavior
+
acceptable retry rate
```

---

# 36. نقطة مهمة جدًا: Atomic SQL Update

في بعض حالات الـ inventory فيه حل أقوى وأبسط من إنك:

```text
SELECT
↓
check
↓
UPDATE
```

ممكن تعمل العملية كلها في SQL بشكل atomic.

مثلاً:

```sql
UPDATE skus
SET current_quantity = current_quantity - :quantity
WHERE id = :id
AND current_quantity >= :quantity;
```

وبعدين تشوف:

```text
rows affected
```

لو:

```text
1
```

العملية نجحت.

لو:

```text
0
```

يبقى مفيش stock كفاية أو الـ SKU مش موجود.

ده بيقلل الـ race condition لأن الـ check والـ update بقوا **عملية database واحدة**.

ودي نقطة Advanced مهمة جدًا.

---

# 37. طب والـ Ledger؟

لو عندك:

```text
StockMovement
```

مثلاً:

```text
Movement 1:
IN +100

Movement 2:
OUT -20

Movement 3:
OUT -30
```

فأنت ممكن يكون عندك:

```text
Sku.currentQuantity = 50
```

والـ ledger يحتفظ بتاريخ كل movements.

الـ concurrency problem هنا إن:

```text
Current Quantity
```

لازم يفضل consistent مع العمليات اللي بتتحفظ.

فعملية الـ movement نفسها لازم تكون atomic:

```text
BEGIN

lock SKU

validate stock

update quantity

insert movement

COMMIT
```

لو أي خطوة فشلت:

```text
ROLLBACK
```

فلا يحصل:

```text
quantity updated
+
movement missing
```

ولا العكس.

---

# 38. الخلاصة الكبيرة للمرحلة كلها

احفظ الـ mental model ده:

```text
              CONCURRENCY
                   │
                   ↓
             Race Conditions
                   │
                   ↓
             Lost Updates
                   │
          ┌────────┴────────┐
          ↓                 ↓
   Optimistic          Pessimistic
      Locking             Locking
          │                 │
       @Version       FOR UPDATE
          │                 │
      conflict          blocking
      detection          writers
          │                 │
          └────────┬────────┘
                   ↓
              Transactions
                   │
                   ↓
            Isolation Levels
                   │
                   ↓
              Deadlocks
                   │
                   ↓
        Ordering + Short Tx +
             Retry
```

---

# 39. أهم أسئلة Interview لازم تعرف تجاوبها

### Q1: What is a race condition?

> A race condition occurs when multiple threads or transactions concurrently access and modify shared data, and the result depends on the timing or ordering of their execution.

---

### Q2: What is Lost Update?

> A lost update occurs when two transactions read the same data, modify it independently, and one transaction overwrites the changes made by the other.

---

### Q3: How can you prevent Lost Updates in JPA?

أهم الإجابات:

```text
Optimistic Locking
Pessimistic Locking
Atomic database updates
Proper transaction design
```

---

### Q4: What is Optimistic Locking?

> Optimistic locking assumes conflicts are rare and detects concurrent modifications using a version field, such as JPA's `@Version`.

---

### Q5: What is Pessimistic Locking?

> Pessimistic locking acquires a database lock on the selected rows so that conflicting transactions have to wait until the lock is released.

---

### Q6: Difference between Optimistic and Pessimistic?

احفظ:

> **Optimistic locking detects conflicts after concurrent work occurs, while pessimistic locking prevents conflicting modifications by locking the data before the work.**

---

### Q7: What does `PESSIMISTIC_WRITE` do?

> It requests an exclusive database lock on the selected entity/row, preventing other transactions from concurrently acquiring conflicting write locks until the current transaction completes.

---

### Q8: Why does pessimistic locking require a transaction?

لأن الـ lock مرتبط بعمر الـ database transaction:

```text
BEGIN
 ↓
LOCK
 ↓
WORK
 ↓
COMMIT / ROLLBACK
 ↓
UNLOCK
```

---

### Q9: What's the default isolation level in PostgreSQL?

**READ COMMITTED.**

---

### Q10: What is Dirty Read?

> Reading data written by another transaction before that transaction commits.

---

### Q11: What is Non-repeatable Read?

> Reading the same row twice within one transaction and getting different committed values because another transaction modified it between the reads.

---

### Q12: What is Phantom Read?

> Re-executing a query and finding a different set of rows because another transaction inserted, deleted, or otherwise changed rows matching the query predicate.

---

### Q13: What is Deadlock?

> A deadlock occurs when two or more transactions wait for each other to release resources, creating a circular dependency.

---

### Q14: How do you prevent Deadlocks?

أهم 4:

```text
1. Consistent lock ordering
2. Keep transactions short
3. Avoid unnecessary locks
4. Retry transient failures when appropriate
```

---

# 40. وأهم حاجة تربط كل ده ببعضه

لو سألك في Interview:

> **"You have an inventory system where multiple requests can update the same SKU simultaneously. How would you prevent incorrect stock updates?"**

ماتقولش بس:

> "I will use `@Transactional`."

دي مش إجابة كافية.

الإجابة الاحترافية:

> "First, I would make the stock update atomic and transactional. If concurrent updates on the same SKU are frequent, I would consider pessimistic locking using `PESSIMISTIC_WRITE` so that only one transaction can modify the stock row at a time. Another option is optimistic locking using `@Version` when conflicts are relatively rare. I would also keep the transaction short, avoid external calls inside it, and use consistent lock ordering to reduce deadlock risk. For high-contention operations, an atomic conditional SQL update can also be a very efficient approach."

دي إجابة تبين إنك **فاهم المشكلة مش حافظ annotation**.

---

## 🎯 ترتيب مذاكرتك بعد المرحلة دي

أنت كده المفروض تكون فاهم:

```text
✅ Transactions
      ↓
✅ Race Conditions
      ↓
✅ Lost Update
      ↓
✅ @Version / Optimistic Locking
      ↓
✅ Pessimistic Locking
      ↓
✅ @Lock(PESSIMISTIC_WRITE)
      ↓
✅ Isolation Levels
      ↓
✅ Dirty Read
✅ Non-repeatable Read
✅ Phantom Read
      ↓
✅ Deadlocks
      ↓
✅ Atomic Updates
      ↓
✅ Inventory / Ledger Concurrency
```

**والخطوة اللي بعدها منطقيًا جدًا** إننا ندخل على **Transaction Management في Spring بالتفصيل**: `@Transactional`، الـ propagation، `REQUIRED / REQUIRES_NEW / NESTED`، rollback rules، self-invocation، وtransaction boundaries؛ لأن ده هو الجزء اللي هيخليك تفهم _فعليًا_ ليه الـ locking والـ isolation اللي فوق بيشتغلوا بالطريقة دي.


```