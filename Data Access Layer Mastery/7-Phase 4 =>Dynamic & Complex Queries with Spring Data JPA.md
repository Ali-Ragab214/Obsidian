أيوه، بما إنك ذاكرت أول 3 مراحل، فالمرحلة الرابعة مهمة جدًا لأنها بتنقلك من إنك تعرف تعمل **queries ثابتة** إلى إنك تعرف تبني **queries ديناميكية ومعقدة** حسب الـ input اللي جاي من الـ client.

## عنوان المرحلة الرابعة

# Phase 4 — Dynamic & Complex Queries with Spring Data JPA

والـ roadmap بتاعها يكون:

1. **Why Dynamic Queries?**
    
2. **Specification Pattern**
    
3. **Spring Data JPA `Specification` API**
    
4. **Criteria API**
    
5. **Combining Specifications**
    
6. **Dynamic Joins**
    
7. **Dynamic Sorting & Pagination**
    
8. **Query Composition**
    
9. **QueryDSL — فهمه ومقارنته بالـ Specification**
    
10. **Best Practices & Common Mistakes**
    
11. **Practical Project**
    

خلينا نمسكها من الصفر.

---

# 1. الأول: يعني إيه Dynamic Query؟

افترض عندك API:

```http
GET /api/skus
```

والـ user ممكن يبعت:

```http
GET /api/skus?category=Electronics
```

أو:

```http
GET /api/skus?category=Electronics&minPrice=100
```

أو:

```http
GET /api/skus?category=Electronics&minPrice=100&maxPrice=1000
```

أو:

```http
GET /api/skus?category=Electronics&lowStock=true
```

أو كلهم:

```http
GET /api/skus?
    category=Electronics
    &minPrice=100
    &maxPrice=1000
    &lowStock=true
```

المشكلة هنا:

**أنت مش عارف مسبقًا الـ filters اللي المستخدم هيبعتها.**

---

# 2. الطريقة الساذجة

ممكن تقول:

```java
findByCategory(...)
findByCategoryAndMinPrice(...)
findByCategoryAndMaxPrice(...)
findByCategoryAndMinPriceAndMaxPrice(...)
findByCategoryAndLowStock(...)
findByCategoryAndMinPriceAndLowStock(...)
```

وهكذا...

وده طبعًا **كارثة**.

لو عندك 10 filters، عدد combinations ممكن يكبر بشكل ضخم جدًا.

وهنا بيظهر مفهوم:

> **Dynamic Query**

يعني query بتتكون بناءً على الـ parameters الموجودة في الـ request.

---

# 3. هنا يظهر Specification Pattern

الـ Specification ببساطة بتخليك تمثل **شرط واحد من شروط البحث** كـ object/function قابل لإعادة الاستخدام.

مثلًا:

```java
hasCategory("Electronics")
```

دي Specification.

و:

```java
hasMinPrice(100)
```

دي Specification تانية.

و:

```java
isLowStock()
```

دي Specification تالتة.

وبعدين تقدر تركبهم:

```text
hasCategory
      AND
hasMinPrice
      AND
isLowStock
```

وده جوهر الـ Specification Pattern.

---

# 4. شكل الـ Specification في Spring

Spring Data JPA عنده interface اسمها:

```java
Specification<T>
```

مثلًا:

```java
Specification<Sku>
```

معناها:

> Specification قادرة على بناء condition خاص بالـ `Sku`.

مثال:

```java
public static Specification<Sku> hasCategory(String category) {
    return (root, query, cb) ->
            cb.equal(root.get("category"), category);
}
```

خلينا نفكها واحدة واحدة.

---

# 5. نفهم الـ Lambda

عندك:

```java
(root, query, cb) -> ...
```

دول أهم 3 objects في الـ Specification.

## `root`

يمثل الـ Entity اللي بتعمل عليها query.

هنا:

```java
Specification<Sku>
```

إذن:

```java
root
```

يمثل:

```java
Sku
```

يعني:

```java
root.get("category")
```

معناها:

> هات field الـ `category` من الـ Sku.

---

# 6. `CriteriaQuery`

ده بيمثل الـ query نفسها.

```java
query
```

ومن خلاله ممكن تعمل حاجات متقدمة زي:

- ordering
    
- distinct
    
- joins
    
- subqueries
    

مش هنحتاج نستخدمه كتير في البداية.

---

# 7. `CriteriaBuilder`

وده أهم واحد.

```java
cb
```

ده object بيساعدك تبني الـ conditions.

مثلًا:

```java
cb.equal(...)
```

تعني:

```sql
=
```

---

مثلاً:

```java
cb.equal(root.get("category"), "Electronics")
```

تقريبًا معناها:

```sql
WHERE category = 'Electronics'
```

---

## أمثلة مهمة جدًا

### Equal

```java
cb.equal(root.get("category"), category)
```

SQL:

```sql
category = ?
```

---

### Not Equal

```java
cb.notEqual(root.get("status"), status)
```

SQL:

```sql
status <> ?
```

---

### Greater Than

```java
cb.greaterThan(root.get("price"), price)
```

SQL:

```sql
price > ?
```

---

### Less Than

```java
cb.lessThan(root.get("price"), price)
```

SQL:

```sql
price < ?
```

---

### Greater Than or Equal

```java
cb.greaterThanOrEqualTo(root.get("price"), price)
```

---

### Less Than or Equal

```java
cb.lessThanOrEqualTo(root.get("price"), price)
```

---

### Like

```java
cb.like(root.get("name"), "%" + name + "%")
```

SQL:

```sql
name LIKE '%phone%'
```

---

### AND

```java
cb.and(condition1, condition2)
```

---

### OR

```java
cb.or(condition1, condition2)
```

---

### NOT

```java
cb.not(condition)
```

---

# 8. نعمل أول Specification حقيقية

افترض:

```java
@Entity
public class Sku {

    private String name;

    private String category;

    private BigDecimal price;

    private Integer currentQuantity;

    private Integer reorderThreshold;
}
```

نقدر نعمل:

```java
public class SkuSpecifications {

    public static Specification<Sku> hasCategory(String category) {

        return (root, query, cb) ->
                cb.equal(root.get("category"), category);
    }
}
```

دي بتقول:

> لو المستخدم بعت category، أضف شرط category.

---

# 9. Specification تانية

عايزين minimum price:

```java
public static Specification<Sku> hasMinPrice(BigDecimal minPrice) {

    return (root, query, cb) ->
            cb.greaterThanOrEqualTo(
                    root.get("price"),
                    minPrice
            );
}
```

يعني:

```sql
price >= ?
```

---

# 10. Specification للـ maximum price

```java
public static Specification<Sku> hasMaxPrice(BigDecimal maxPrice) {

    return (root, query, cb) ->
            cb.lessThanOrEqualTo(
                    root.get("price"),
                    maxPrice
            );
}
```

---

# 11. Specification للـ Low Stock

دي مهمة جدًا بالنسبة لمشروع الـ inventory بتاعك.

```java
public static Specification<Sku> isLowStock() {

    return (root, query, cb) ->
            cb.lessThan(
                    root.get("currentQuantity"),
                    root.get("reorderThreshold")
            );
}
```

هنا interesting point:

إحنا مش بنقارن field بقيمة ثابتة.

إحنا بنقارن:

```text
currentQuantity
```

مع:

```text
reorderThreshold
```

يعني:

```sql
current_quantity < reorder_threshold
```

---

# 12. طب أستخدمهم إزاي؟

الـ Repository لازم يدعم Specification.

```java
public interface SkuRepository
        extends JpaRepository<Sku, UUID>,
                JpaSpecificationExecutor<Sku> {
}
```

الجزء المهم:

```java
JpaSpecificationExecutor<Sku>
```

ده بيدي repository capability إنها تستقبل:

```java
Specification<Sku>
```

وبعدين:

```java
repository.findAll(specification);
```

---

# 13. Composition — أهم جزء

دلوقتي عندنا:

```java
hasCategory(...)
hasMinPrice(...)
hasMaxPrice(...)
isLowStock()
```

نقدر نركبهم.

مثلاً:

```java
Specification<Sku> spec =
        Specification
                .where(hasCategory("Electronics"))
                .and(hasMinPrice(BigDecimal.valueOf(100)))
                .and(hasMaxPrice(BigDecimal.valueOf(1000)))
                .and(isLowStock());
```

المعنى:

```text
category = Electronics
AND
price >= 100
AND
price <= 1000
AND
currentQuantity < reorderThreshold
```

---

# 14. ودي النقطة اللي تخلي Specification قوية جدًا

ممكن تعمل:

```java
Specification<Sku> spec = Specification.where(null);
```

وبعدين تضيف conditions حسب الـ input.

مثلاً:

```java
if (category != null) {
    spec = spec.and(hasCategory(category));
}

if (minPrice != null) {
    spec = spec.and(hasMinPrice(minPrice));
}

if (maxPrice != null) {
    spec = spec.and(hasMaxPrice(maxPrice));
}

if (lowStock != null && lowStock) {
    spec = spec.and(isLowStock());
}
```

وبعدين:

```java
repository.findAll(spec);
```

فأنت عندك **query dynamic بالكامل**.

---

# 15. مثال كامل

افترض عندك:

```java
public record SkuSearchCriteria(
        String category,
        BigDecimal minPrice,
        BigDecimal maxPrice,
        Boolean lowStock
) {}
```

وفي service:

```java
public List<Sku> search(SkuSearchCriteria criteria) {

    Specification<Sku> spec = Specification.where(null);

    if (criteria.category() != null) {
        spec = spec.and(
                SkuSpecifications.hasCategory(
                        criteria.category()
                )
        );
    }

    if (criteria.minPrice() != null) {
        spec = spec.and(
                SkuSpecifications.hasMinPrice(
                        criteria.minPrice()
                )
        );
    }

    if (criteria.maxPrice() != null) {
        spec = spec.and(
                SkuSpecifications.hasMaxPrice(
                        criteria.maxPrice()
                )
        );
    }

    if (Boolean.TRUE.equals(criteria.lowStock())) {
        spec = spec.and(
                SkuSpecifications.isLowStock()
        );
    }

    return repository.findAll(spec);
}
```

ده implementation حقيقي ومناسب جدًا.

---

# 16. ليه ده أفضل من JPQL؟

افترض إنك كتبت:

```java
@Query("""
    SELECT s FROM Sku s
    WHERE (:category IS NULL OR s.category = :category)
    AND (:minPrice IS NULL OR s.price >= :minPrice)
    AND (:maxPrice IS NULL OR s.price <= :maxPrice)
""")
```

ده ممكن يشتغل.

لكن لما الـ filters تزيد:

```text
category
minPrice
maxPrice
status
warehouse
supplier
createdAfter
createdBefore
lowStock
search
...
```

الـ query هتبدأ تبقى:

**وحشة جدًا في الـ maintenance.**

Specification بتخليك تفصل كل business condition عن التانية.

---

# 17. نقطة مهمة جدًا: Specification مش هي Criteria API نفسها

دي نقطة لازم تثبت في دماغك.

العلاقة:

```text
Spring Data JPA
       │
       ▼
Specification
       │
       ▼
Criteria API
       │
       ▼
Hibernate
       │
       ▼
SQL
       │
       ▼
PostgreSQL
```

يعني:

**Specification abstraction مبنية باستخدام Criteria API.**

أنت غالبًا مش هتتعامل مع Criteria API مباشرة في المشاريع العادية.

---

# 18. طيب Criteria API بتعمل إيه؟

لو معندكش Specification، ممكن تبني query بنفسك:

```java
CriteriaBuilder cb =
        entityManager.getCriteriaBuilder();

CriteriaQuery<Sku> query =
        cb.createQuery(Sku.class);

Root<Sku> root =
        query.from(Sku.class);
```

وبعدين:

```java
Predicate predicate =
        cb.equal(
                root.get("category"),
                "Electronics"
        );

query.where(predicate);
```

وبعدين execute:

```java
entityManager
        .createQuery(query)
        .getResultList();
```

شوف كمية الـ boilerplate.

عشان كده Specification أسهل.

---

# 19. Predicate

مفهوم مهم جدًا في Criteria API.

لما تعمل:

```java
cb.equal(...)
```

الناتج بيكون:

```java
Predicate
```

يعني condition.

مثلاً:

```java
Predicate categoryPredicate =
        cb.equal(
                root.get("category"),
                "Electronics"
        );
```

و:

```java
Predicate pricePredicate =
        cb.greaterThan(
                root.get("price"),
                BigDecimal.valueOf(100)
        );
```

وتقدر تعمل:

```java
Predicate finalPredicate =
        cb.and(
                categoryPredicate,
                pricePredicate
        );
```

يعني:

```sql
WHERE category = 'Electronics'
AND price > 100
```

وده بالضبط تقريبًا اللي Specification بتسهله عليك.

---

# 20. Dynamic Joins

دي من أهم الحاجات اللي لازم تتعلمها بعد الأساسيات.

افترض:

```text
Sku
 |
 └── Warehouse
```

وعايز تبحث عن الـ SKUs الموجودة في warehouse معين.

ممكن تعمل:

```java
public static Specification<Sku> inWarehouse(UUID warehouseId) {

    return (root, query, cb) -> {

        Join<Sku, Warehouse> warehouse =
                root.join("warehouse");

        return cb.equal(
                warehouse.get("id"),
                warehouseId
        );
    };
}
```

هنا:

```java
root.join("warehouse")
```

بتعمل JOIN.

تقريبًا:

```sql
JOIN warehouses w
    ON ...
WHERE w.id = ?
```

---

# 21. `Join` مهم جدًا

الفرق:

```java
root.get("category")
```

لـ field داخل نفس entity.

بينما:

```java
root.join("warehouse")
```

لما محتاج تدخل relationship.

مثلاً:

```text
Sku
 ├── category
 ├── price
 └── warehouse
        ├── id
        └── name
```

لو عايز:

```text
warehouse.name = "Cairo Warehouse"
```

تعمل:

```java
Join<Sku, Warehouse> warehouse =
        root.join("warehouse");

return cb.equal(
        warehouse.get("name"),
        "Cairo Warehouse"
);
```

---

# 22. OR Conditions

مش كل حاجة AND.

مثلاً:

```text
name contains "phone"
OR
skuCode contains "phone"
```

ممكن:

```java
public static Specification<Sku> search(String keyword) {

    return (root, query, cb) -> {

        Predicate name =
                cb.like(
                        cb.lower(root.get("name")),
                        "%" + keyword.toLowerCase() + "%"
                );

        Predicate skuCode =
                cb.like(
                        cb.lower(root.get("skuCode")),
                        "%" + keyword.toLowerCase() + "%"
                );

        return cb.or(name, skuCode);
    };
}
```

وده هيطلع تقريبًا:

```sql
WHERE
    LOWER(name) LIKE '%phone%'
    OR
    LOWER(sku_code) LIKE '%phone%'
```

---

# 23. AND + OR مع بعض

دي نقطة لازم تفهمها كويس.

مثلاً عايزين:

```text
category = Electronics
AND
(
    name contains phone
    OR
    skuCode contains phone
)
```

الـ Specification:

```java
return Specification
        .where(hasCategory("Electronics"))
        .and(search("phone"));
```

والـ `search` نفسها فيها:

```java
cb.or(...)
```

فبتطلع structure زي:

```text
category = Electronics
AND
(
    name LIKE ...
    OR skuCode LIKE ...
)
```

وده بيسمى **Predicate Composition**.

---

# 24. Dynamic Sorting

مش لازم الـ dynamic query تكون filters بس.

ممكن الـ user يقول:

```http
GET /skus?sort=price&direction=desc
```

Spring Data يقدر يتعامل مع:

```java
Sort.by(
    Sort.Direction.DESC,
    "price"
);
```

وتقدر تجمعها مع Specification:

```java
repository.findAll(
    spec,
    Sort.by(Sort.Direction.DESC, "price")
);
```

---

# 25. Dynamic Pagination

وده غالبًا هيكون مع الـ search API الحقيقي.

مثلاً:

```java
Pageable pageable =
        PageRequest.of(
                page,
                size,
                Sort.by("price")
        );
```

وبعدين:

```java
Page<Sku> result =
        repository.findAll(spec, pageable);
```

وده مهم جدًا في production.

لأنك **ممنوع** تعمل:

```java
findAll()
```

وتجيب مليون record للـ application memory.

---

# 26. Specification + Pagination

دي تركيبة هتقابلها كتير جدًا:

```java
Page<Sku> result =
        skuRepository.findAll(
                specification,
                pageable
        );
```

يعني:

```text
Dynamic Filters
       +
Sorting
       +
Pagination
```

وكل ده في query واحدة مناسبة للـ use case.

---

# 27. QueryDSL

دلوقتي نروح للمصطلح التاني.

QueryDSL مكتبة external بتخليك تكتب queries بطريقة أكثر type-safe وأقرب للـ Java code.

بدل:

```java
root.get("category")
```

تقدر تستخدم:

```java
sku.category.eq("Electronics")
```

مثلاً:

```java
QSku sku = QSku.sku;

queryFactory
    .selectFrom(sku)
    .where(
        sku.category.eq("Electronics")
            .and(
                sku.currentQuantity
                    .lt(sku.reorderThreshold)
            )
    )
    .fetch();
```

وده شكله فعلًا أنضف.

---

# 28. ليه QueryDSL قوية؟

بص للفرق:

### Criteria

```java
root.get("currentQuantity")
```

لو كتبت:

```java
root.get("currentQuantitty")
```

ممكن المشكلة تظهر runtime.

بينما QueryDSL:

```java
sku.currentQuantity
```

لأن الـ Q-class generated من الـ entity.

---

# 29. لكن QueryDSL مش أول حاجة تركز عليها

بالنسبة لك حاليًا:

```text
★★★★★ Specification
★★★★☆ Criteria API
★★★☆☆ QueryDSL
```

ركز على:

### 1. Specification

دي الأهم.

### 2. Criteria API

افهمها عشان تعرف الـ Specification شغالة إزاي.

### 3. QueryDSL

اعرفها كويس بما يكفي إنك:

- تقرأ code مكتوب بيها
    
- تعرف مميزاتها
    
- تعرف الفرق بينها وبين Specification
    
- تستخدمها لو project بيستخدمها
    

لكن متخليش وقتك كله فيها.

---

# 30. أهم حاجة: متعملش Specification لكل حاجة

دي غلطة شائعة.

مش معنى إن عندك Specification إنك تستخدمها في كل query.

لو query بسيطة:

```java
findByStatus(status)
```

خلاص.

استخدم:

**Derived Query.**

لو query ثابتة ومعقدة:

```java
@Query(...)
```

استخدم JPQL.

لو dynamic:

```text
category?
price?
status?
warehouse?
supplier?
date?
```

استخدم Specification.

---

# 31. إمتى أستخدم كل طريقة؟

احفظ الجدول ده:

|الحالة|الحل|
|---|---|
|Query بسيطة وثابتة|Derived Query|
|Query متوسطة/معقدة وثابتة|JPQL|
|PostgreSQL-specific|Native SQL|
|Filters ديناميكية|Specification|
|Complex dynamic query + type safety|QueryDSL|
|فهم الـ Specification internals|Criteria API|

---

# 32. أهم Best Practice

متعملش:

```java
if (category != null) {
    ...
}

if (price != null) {
    ...
}

if (status != null) {
    ...
}
```

جوا repository.

خلي الـ specifications نفسها مسؤولة عن الـ conditions.

مثلاً:

```java
SkuSpecifications.hasCategory(category)
```

```java
SkuSpecifications.hasPriceBetween(min, max)
```

```java
SkuSpecifications.hasStatus(status)
```

```java
SkuSpecifications.isLowStock()
```

وبعدين تعمل composition.

---

# 33. والأفضل كمان تعمل method للـ search specification

بدل service ضخمة:

```java
public Specification<Sku> buildSpecification(
        SkuSearchCriteria criteria
) {
    Specification<Sku> spec =
            Specification.where(null);

    if (criteria.category() != null) {
        spec = spec.and(
                hasCategory(criteria.category())
        );
    }

    if (criteria.minPrice() != null) {
        spec = spec.and(
                hasMinPrice(criteria.minPrice())
        );
    }

    return spec;
}
```

وبعدين service:

```java
Specification<Sku> spec =
        skuSpecificationBuilder.build(criteria);

return repository.findAll(spec, pageable);
```

ده أنضف في المشاريع الكبيرة.

---

# 34. خلي بالك من نقطة مهمة جدًا

لما تعمل:

```java
root.get("category")
```

أنت بتستخدم **String**:

```java
"category"
```

وده مش type-safe بالكامل.

لو غيرت الـ entity:

```java
private String category;
```

إلى:

```java
private String productCategory;
```

لازم تفتكر تغير الـ string.

وده أحد الأسباب اللي تخلي QueryDSL جذابة في بعض المشاريع.

---

# 35. المرحلة دي بالنسبة للـ Interview

لازم تقدر تجاوب على الأسئلة دي:

### 1.

**Why do we need dynamic queries?**

لأن الـ filters ممكن تكون optional ومتغيرة حسب input المستخدم.

---

### 2.

**What is Specification in Spring Data JPA?**

طريقة لبناء reusable, composable dynamic query predicates باستخدام Criteria API.

---

### 3.

**What is `JpaSpecificationExecutor`?**

Interface بتضيف support لتنفيذ queries باستخدام:

```java
Specification<T>
```

---

### 4.

**What is Criteria API?**

API في JPA لبناء queries programmatically بدل كتابة JPQL strings.

---

### 5.

**What is the difference between Specification and Criteria API?**

Criteria API هي الـ low-level query-building API.

Specification abstraction مبنية عليها وبتخلي conditions reusable وcomposable.

---

### 6.

**What is QueryDSL?**

External library لبناء type-safe queries باستخدام generated Q-classes.

---

### 7.

**Why use Specification instead of a huge JPQL query?**

لأن الـ Specification بتقسم conditions إلى أجزاء reusable وتسمح بتركيبها dynamically.

---

# 36. التطبيق العملي اللي أنصحك تعمله

بما إنك شغال على inventory system، اعمل endpoint:

```http
GET /api/skus
```

يدعم:

```text
category
minPrice
maxPrice
status
warehouseId
supplierId
lowStock
keyword
createdAfter
createdBefore
```

مثلاً:

```http
GET /api/skus?
category=Electronics
&minPrice=100
&maxPrice=5000
&lowStock=true
&keyword=phone
&page=0
&size=20
&sort=price,desc
```

والـ architecture تكون:

```text
Controller
    ↓
SearchSkuQuery
    ↓
SearchSkuHandler / Service
    ↓
SkuSpecificationBuilder
    ↓
SkuRepository
    ↓
JpaSpecificationExecutor
    ↓
Hibernate
    ↓
PostgreSQL
```

وده exercise ممتاز جدًا لأنه هيجمع لك:

```text
Specification
      +
Criteria API
      +
Dynamic Filters
      +
Joins
      +
Pagination
      +
Sorting
      +
Query Composition
```

---

# الخلاصة اللي تثبتها في دماغك

الـ **Specification** فكر فيها بالشكل ده:

```text
User Request
     │
     │ optional filters
     ▼
Search Criteria
     │
     ▼
Build Specifications
     │
     ├── hasCategory()
     ├── hasMinPrice()
     ├── hasMaxPrice()
     ├── isLowStock()
     ├── search()
     └── inWarehouse()
     │
     ▼
Combine with AND / OR
     │
     ▼
Specification<Sku>
     │
     ▼
JpaSpecificationExecutor
     │
     ▼
Hibernate / Criteria API
     │
     ▼
SQL
```

**والأهم:** متحفظش الكود. افهم الـ mental model:

> **Specification = condition reusable**

> **Criteria API = mechanism used to build the query programmatically**

> **JpaSpecificationExecutor = اللي بيخلي Repository تنفذ الـ Specification**

> **QueryDSL = alternative أكثر type-safe لبناء dynamic queries**

ولو خلصت المرحلة دي كويس، تبقى بعد كده جاهز جدًا تدخل على **Phase 5: Fetching Strategies & Performance Optimization**، ودي هتربط لك الـ N+1 اللي ذاكرتَه مع `LAZY/EAGER`, `JOIN FETCH`, `EntityGraph`, projections, pagination، ومشاكل الـ Hibernate performance بشكل أعمق.