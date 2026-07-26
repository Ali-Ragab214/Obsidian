ممتاز، دي من أهم المواضيع في Spring Data JPA، وبتتسأل عنها في كل انترفيو Backend تقريبًا، خصوصًا لو الشركة شغالة على Systems كبيرة.

**الهدف من الشرح ده:**

- تفهم المشكلة من جوه مش تحفظها.
    
- تعرف ليه Hibernate بيعمل كده.
    
- تعرف إمتى تستخدم كل حل.
    
- تعرف أفضل حل في كل Scenario.
    
- تعرف إجابة الانترفيو المثالية.
    

---

# المرحلة الأولى

# الأول نفهم JPA و Hibernate بيشتغلوا إزاي

نفترض عندنا الـ Entities دي.

```java
@Entity
public class PurchaseOrder {

    @Id
    private Long id;

    private String status;

    @ManyToOne(fetch = FetchType.LAZY)
    private Vendor vendor;
}
```

والـ Vendor

```java
@Entity
public class Vendor {

    @Id
    private Long id;

    private String name;
}
```

العلاقة هنا

```
PurchaseOrder -----> Vendor

Many Orders
      |
      |
      V
 One Vendor
```

يعني كل PurchaseOrder ليه Vendor.

---

# السؤال المهم

لما أعمل

```java
purchaseOrderRepository.findAll();
```

هل Hibernate هيجيب الـ Vendor؟

الإجابة...

**لا**

ليه؟

لأن

```java
@ManyToOne(fetch = FetchType.LAZY)
```

يعني

> متجيبش الـ Vendor دلوقتي...
> 
> استنى لحد ما حد يطلبه.

وده اسمه

# Lazy Loading

---

# Lazy Loading بيعمل إيه؟

بدل ما Hibernate يحط Vendor الحقيقي

يحط Proxy

يعني Object مزيف.

مثلاً

```
PurchaseOrder

id = 10

vendor = Vendor$HibernateProxy
```

مش الـ Vendor الحقيقي.

أول ما تعمل

```java
po.getVendor()
```

Hibernate يقول

> آه...
> 
> هو عايز الـ Vendor الحقيقي.

فيروح Database.

---

# تعالى نشوف مثال

Database

PurchaseOrder

|id|vendor_id|
|---|---|
|1|5|
|2|6|
|3|8|

Vendor

|id|name|
|---|---|
|5|Dell|
|6|HP|
|8|Apple|

---

الكود

```java
List<PurchaseOrder> orders =
        purchaseOrderRepository.findAll();
```

Hibernate يعمل

```sql
SELECT *
FROM purchase_order;
```

ويرجع

```
Order1

Order2

Order3
```

لكن

```
Vendor

لسه متجابش.
```

---

بعدها

```java
for(PurchaseOrder po : orders){

    System.out.println(
        po.getVendor().getName()
    );

}
```

Hibernate يبدأ.

أول Order

```
Vendor = 5
```

ينفذ

```sql
SELECT *
FROM vendor
WHERE id = 5;
```

---

ثاني Order

```
Vendor = 6
```

```sql
SELECT *
FROM vendor
WHERE id = 6;
```

---

ثالث Order

```sql
SELECT *
FROM vendor
WHERE id = 8;
```

---

إجمالي الـ Queries

```
1

+

3

=

4 Queries
```

---

لو عندك

```
100 Orders
```

يبقى

```
1

+

100

=

101 Query
```

وده اسمه

# N+1 Problem

يعني

```
واحدة

+

N
```

---

# ليه اسمها N+1؟

لو عندك

```
N Orders
```

يبقى

```
Query

لجيب الـ Orders

+

N Queries

لجيب Vendors
```

---

# المشكلة الحقيقية

المشكلة مش إن فيه Queries كتير بس.

المشكلة إن

كل Query معناها

```
Connection

↓

Network

↓

Database Parsing

↓

Execution

↓

Return Data
```

كل ده بيتكرر.

لو كل Query

5 ms

وعندك

1000 Query

```
5000 ms

=

5 Seconds
```

بس علشان تجيب بيانات!

---

# ازاي أعرف إن عندي N+1؟

أول علامة

```
spring.jpa.show-sql=true
```

تشوف Console.

بدل ما تشوف

```sql
select ...
```

مرة

تلاقي

```sql
select ...

select ...

select ...

select ...

select ...

select ...

select ...

select ...

select ...
```

مئات المرات.

---

# ليه Hibernate عمل كده؟

لأنه Lazy.

هو بينفذ الفلسفة دي

> Don't Load Until Needed

وده صح جدًا.

لكن

لو احتجت البيانات كلها

يبقى Lazy بقى ضدك.

---

# الحل الأول

# JOIN FETCH

وده أهم وأشهر حل.

JPQL

```java
@Query("""
SELECT po
FROM PurchaseOrder po
JOIN FETCH po.vendor
WHERE po.status=:status
""")
List<PurchaseOrder> findAllWithVendor(String status);
```

---

Hibernate يحولها تقريبًا إلى

```sql
SELECT

po.*,

vendor.*

FROM purchase_order po

JOIN vendor v

ON po.vendor_id=v.id
```

يعني

بدل

```
101 Query
```

تبقى

```
1 Query
```

---

النتيجة

```
Order1

Vendor جاهز

Order2

Vendor جاهز

Order3

Vendor جاهز
```

ولما تعمل

```java
po.getVendor()
```

مش هيروح Database.

---

# ليه اسمه Fetch Join؟

الـ JOIN العادي

بيعمل Join

لكن

مش شرط يخزن الـ Vendor داخل Entity.

إنما

```
JOIN FETCH
```

بيقول لـ Hibernate

> اعمل Join...

وكمان

املالي الـ Entity.

---

# مثال

بدون Fetch

```
SELECT po
FROM PurchaseOrder po

JOIN po.vendor
```

ده Join

للفلترة غالبًا.

لكن

الـ Vendor

لسه Lazy.

---

أما

```java
JOIN FETCH
```

يبقى

```
Join

+

Load
```

في نفس الوقت.

---

# المميزات

✅ أسرع حل

✅ واضح جدًا

✅ Query واحدة

✅ أشهر حل

---

# العيوب

لو عندك

```
Vendor

+

Items

+

Payments

+

Invoices

+

Attachments
```

وبدأت تعمل

```
JOIN FETCH

JOIN FETCH

JOIN FETCH

JOIN FETCH
```

الـ Query تبقى ضخمة جدًا.

---

ولو عندك Collection

مثلاً

```
Order

↓

LineItems
```

كل Order فيه

50 Item

يبقى

```
100 Order

×

50 Item

=

5000 Row
```

رغم إن عندك

100 Order فقط.

وده اسمه

# Cartesian Explosion

مثال:

```
Order 1
 ├── Item A
 ├── Item B
 └── Item C
```

الـ SQL الناتج هيكرر بيانات الـ Order مع كل Item:

|Order ID|Vendor|Item|
|---|---|---|
|1|Dell|Mouse|
|1|Dell|Keyboard|
|1|Dell|Monitor|

Hibernate هيجمعهم مرة أخرى إلى `Order` واحد داخليًا، لكن الشبكة وقاعدة البيانات نقلوا 3 صفوف بدل صف واحد. تخيل بقى لو فيه أكتر من Collection معمول لها `JOIN FETCH`.

---

# الحل الثاني

# EntityGraph

ناس كتير بتفتكره Annotation.

هو في الحقيقة

Fetch Plan.

يعني

بتقول لـ Hibernate

> وأنا بنفذ Query دي...

اعمل Fetch للحاجات دي.

مثال

```java
@EntityGraph(
    attributePaths = {
        "vendor",
        "lineItems"
    }
)
List<PurchaseOrder> findByStatus(String status);
```

الجميل هنا إنك محتفظ باسم الـ Repository Method المشتق (`findByStatus`) لكن غيرت فقط طريقة الـ Fetch.

Hibernate هيولد SQL مناسب (غالبًا باستخدام `JOIN`) لجلب العلاقات المطلوبة في نفس الاستعلام.

---

## إمتى أفضل من JOIN FETCH؟

لو عندك Repository فيه Derived Queries كتير:

```java
findByStatus()

findByVendorId()

findByCreatedAtBetween()

findByStatusAndType()
```

مش هتكتب `@Query` لكل واحدة.

تحط

```java
@EntityGraph(attributePaths = {
    "vendor",
    "lineItems"
})
```

وخلاص.

---

## المميزات

✅ أنظف

✅ يحافظ على Derived Query

✅ سهل الصيانة

✅ مناسب للمشاريع الكبيرة

---

## العيوب

- أقل مرونة من كتابة JPQL معقد.
    
- لو احتجت شروط معقدة أو Aggregation أو DTO، هترجع لـ JPQL.
    

---

# الحل الثالث

# Batch Fetching

وده ناس كتير مبتفهموش.

نفترض عندك

100 Order

وكلهم Lazy.

بدل

```
101 Query
```

Hibernate يقول

> هجيب Vendors بالعشرات.

لو

```java
@BatchSize(size = 20)
```

يبقى

```
SELECT * FROM vendor

WHERE id IN
(
1,2,3,4,5...

20
)
```

بعدها

```
21-40
```

بعدها

```
41-60
```

بدل 100 Query، هيبقى تقريبًا:

- Query واحدة للـ Orders.
    
- 5 Queries للـ Vendors (20 في كل مرة).
    

يعني 6 Queries بدل 101.

---

## إمتى يستخدم؟

لما مش عايز تغير كل Repository وتكتب `JOIN FETCH` أو `EntityGraph`، ولسه عايز تحتفظ بـ Lazy Loading، لكن تقلل عدد الاستعلامات.

---

## هل هو أحسن؟

لا.

هو

Optimization

مش Solution كامل.

---

# الحل الرابع

# DTO Projection

وده أقوى حل للأداء في كثير من حالات القراءة.

لو الشاشة محتاجة

```
Order Id

Vendor Name
```

بس.

ليه أجيب

```
Entity كاملة

Persistence Context

Proxy

Dirty Checking

Caching
```

كل ده؟

اعمل DTO.

```java
public record PoSummaryDto(
    Long id,
    String vendorName
){}
```

JPQL

```java
@Query("""
SELECT new com.app.dto.PoSummaryDto(
    po.id,
    v.name
)
FROM PurchaseOrder po
JOIN po.vendor v
""")
```

Hibernate هيجيب

```
ID

Vendor Name
```

فقط.

---

## ليه ده أسرع؟

لأنه:

- لا ينشئ Entities كاملة.
    
- لا يضيفها إلى Persistence Context.
    
- لا يوجد Lazy Loading.
    
- لا يوجد Dirty Checking.
    

وده بيقلل استهلاك الذاكرة والمعالجة بشكل واضح.

---

# مقارنة الحلول

|الحل|الأداء|سهولة الاستخدام|أفضل استخدام|
|---|---|---|---|
|JOIN FETCH|⭐⭐⭐⭐⭐|⭐⭐⭐⭐|لما تحتاج Entity كاملة مع علاقات محددة|
|EntityGraph|⭐⭐⭐⭐⭐|⭐⭐⭐⭐⭐|مع Derived Queries أو Fetch Plans قابلة لإعادة الاستخدام|
|BatchSize|⭐⭐⭐|⭐⭐⭐⭐|تحسين Lazy Loading بدون تغيير الاستعلامات|
|DTO Projection|⭐⭐⭐⭐⭐⭐|⭐⭐⭐|شاشات القراءة (Read-only APIs)|

---

# أيهم أفضل؟

الإجابة الصحيحة في الانترفيو هي:

> **لا يوجد حل واحد هو الأفضل دائمًا؛ الاختيار يعتمد على الـ Use Case.**

لكن عمليًا:

### لو محتاج Entity كاملة

استخدم:

> **`JOIN FETCH` أو `@EntityGraph`**

وأنا أميل إلى **`@EntityGraph`** في المشاريع الكبيرة لأنه:

- يحافظ على نظافة الـ Repository.
    
- لا يكرر استعلامات JPQL.
    
- يسهل تغيير خطة الـ Fetch بدون تعديل منطق الاستعلام.
    

### لو API للعرض فقط (Dashboard، Table، List)

استخدم:

> **DTO Projection**

وده غالبًا أفضل أداء لأنه لا يحمل كائنات Entity أصلًا.

### لو عندك Lazy Loading في أماكن كثيرة ولا تريد تعديل كل Query

استخدم:

> **`@BatchSize`**

كتحسين إضافي، وليس كبديل أساسي.

---

# أداة لازم تتعلمها

أثناء التطوير فعّل:

```properties
spring.jpa.show-sql=true

logging.level.org.hibernate.SQL=DEBUG

logging.level.org.hibernate.orm.jdbc.bind=TRACE
```

هتشوف:

- الـ SQL الحقيقي.
    
- قيم الـ Parameters.
    
- عدد الاستعلامات المنفذة.
    

وللمشاريع الاحترافية استخدم:

- **Hibernate Statistics**: يعطيك إحصائيات مثل عدد الـ Queries، وعدد الـ Entities التي تم تحميلها، ونسبة الـ Cache Hits.
    
- **P6Spy**: يعرض كل SQL المنفذ مع الزمن المستغرق، وهو ممتاز لاكتشاف مشاكل الأداء.
    
- **datasource-proxy**: مناسب لعمل Logging وقياس أداء استعلامات قاعدة البيانات داخل التطبيق.
    

---

# سؤال انترفيو شائع

**Q: What is the N+1 Select Problem?**

**Answer:**

> The N+1 Select Problem happens when JPA loads a parent collection with one query, then executes an additional query for each related entity because of lazy loading. For example, loading 100 orders and then accessing each order's vendor may execute 101 SQL queries. The common solutions are `JOIN FETCH`, `@EntityGraph`, `DTO Projection`, or `Batch Fetching`, depending on the use case.

---

دي تعتبر من أهم مشاكل الأداء في JPA، ولو فهمتها بعمق هتكون مستعد لأسئلة متقدمة زي **FetchType.LAZY vs EAGER**، و**Persistence Context**، و**Open Session in View (OSIV)**، و**Multiple Bag Fetch Exception**، و**Pagination مع JOIN FETCH**، وهي كلها مرتبطة مباشرة بموضوع الـ N+1.