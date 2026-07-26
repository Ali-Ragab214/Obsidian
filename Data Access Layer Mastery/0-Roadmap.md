# اسم المجال اللي بتسأل عليه

اللي بتوصفه اسمه **"Data Access Layer Mastery"** أو بشكل أدق: **JPA/Hibernate Query Optimization & Persistence Layer Design**. ده مش موضوع واحد، هو مجموعة مهارات مترابطة، وهقسملك بالظبط إيه هي ده وترتيب مذاكرتها من الأساس للاحتراف.

---

## الخريطة الكاملة (Roadmap) بالترتيب

### المرحلة 1: الأساس اللي لازم يكون صلب قبل أي حاجة تانية

#### 1.1 فهم الفرق بين الطرق التلاتة لكتابة queries في Spring Data JPA

لازم تفهم إمتى تستخدم كل واحدة وليه:

| الطريقة                               | الاستخدام                                                                         | مثال                                                                         |
| ------------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Derived Query Methods**             | queries بسيطة، Spring بيولدها من اسم الميثود                                      | `findByStatusAndCreatedAtAfter(...)`                                         |
| **JPQL** (`@Query`)                   | queries أعقد شوية، لسه object-oriented (بتتعامل مع الـ Entity مش الـ table)       | `@Query("SELECT s FROM Sku s WHERE s.currentQuantity < s.reorderThreshold")` |
| **Native SQL**                        | لما محتاج feature خاص بـ PostgreSQL نفسه (window functions, CTEs, JSON operators) | `@Query(value = "SELECT * FROM skus WHERE ...", nativeQuery = true)`         |
| **Criteria API / QuerySpecification** | queries ديناميكية (فلاتر بتتغير حسب الـ input) بدون string concatenation          | هنشرحه تحت                                                                   |

**ذاكر الأول:** JPQL syntax كامل (SELECT, JOIN, WHERE, GROUP BY, subqueries) — ده الأساس لأي حاجة جاية بعد كده.

---

### المرحلة 2: N+1 Problem — أشهر مشكلة أداء في أي مشروع JPA

#### 2.1 إيه هي بالظبط

```java
List<PurchaseOrder> orders = purchaseOrderRepository.findAll(); // Query 1
for (PurchaseOrder po : orders) {
    po.getVendor().getName(); // Query جديدة لكل PO! (Lazy Loading)
}
```

لو عندك 100 PO، هتعمل **1 query لجيب الـ orders + 100 query تانية** لجيب الـ vendor بتاع كل واحد = 101 query بدل واحدة أو اتنين. ده اسمه **N+1 Select Problem**.

#### 2.2 الحلول (ذاكرهم بالترتيب ده)

1. **`JOIN FETCH` في JPQL** — الحل الأساسي والأكتر وضوحًا:

```java
@Query("SELECT po FROM PurchaseOrder po JOIN FETCH po.vendor WHERE po.status = :status")
List<PurchaseOrder> findAllWithVendor(@Param("status") String status);
```

2. **`@EntityGraph`** — بديل أنظف للـ JOIN FETCH لما عندك علاقات كتير:

```java
@EntityGraph(attributePaths = {"vendor", "lineItems"})
List<PurchaseOrder> findByStatus(String status);
```

3. **Batch Fetching** (`@BatchSize` في Hibernate) — لو مش عايز تكتب fetch join لكل query، بيخلي Hibernate يجيب الـ related entities على دفعات بدل واحد واحد.
4. **DTO Projections** — أقوى حل لو أصلاً مش محتاج الـ Entity الكامل:

```java
@Query("SELECT new com.app.dto.PoSummaryDto(po.id, v.name) FROM PurchaseOrder po JOIN po.vendor v")
List<PoSummaryDto> findSummaries();
```

**أداة أساسية تتعلمها هنا:** فعّل `spring.jpa.show-sql=true` و `logging.level.org.hibernate.SQL=DEBUG` وشوف بعينك عدد الـ queries اللي بتتنفذ فعليًا. الأداة الاحترافية للمرحلة دي اسمها **Hibernate Statistics** أو مكتبة زي **datasource-proxy** أو **p6spy** بتوريك كل query بيتنفذ إمتى.

---

### المرحلة 3: Race Conditions — أهم حاجة اتكلمنا فيها في الـ Ledger بتاعك

انت فعليًا اتعلمت الجزء ده عملي في الـ StockMovement، بس خليني أرتبلك المفاهيم بالترتيب النظري:

1. **Lost Update Problem** — الأساس اللي لازم تفهمه الأول (شرحناه قبل كده).
2. **Optimistic Locking** (`@Version`):

```java
@Version
private Long version;
```

Hibernate بيتأكد إن الـ version متغيرتش قبل ما يعمل update، لو اتغيرت بيرمي `OptimisticLockException`. **مناسب لما التعارض نادر الحدوث.**

3. **Pessimistic Locking** (`LockModeType.PESSIMISTIC_WRITE`) — اللي استخدمناه في الـ Ledger:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT s FROM Sku s WHERE s.id = :id")
Optional<Sku> findByIdForUpdate(@Param("id") UUID id);
```

**مناسب لما التعارض متوقع يحصل كتير** (زي الـ stock updates).

4. **Isolation Levels** — لازم تفهم الفرق بين `READ_COMMITTED` و`REPEATABLE_READ` و`SERIALIZABLE`، وإمتى Spring/Postgres بيستخدم إيه افتراضيًا.

**ذاكر بالترتيب ده:** Lost Update → Optimistic vs Pessimistic Locking → Transaction Isolation Levels → Deadlocks وإزاي تتجنبها.

---

### المرحلة 4: Dynamic/Complex Queries — الجزء اللي فيه "Criteria API" و"QueryDSL"

دلوقتي نجي للمصطلحات اللي سمعتها:

#### 4.1 Specification Pattern (الأشهر في Spring Data JPA)

```java
public class SkuSpecifications {
    public static Specification<Sku> hasCategory(String category) {
        return (root, query, cb) -> cb.equal(root.get("category"), category);
    }
    public static Specification<Sku> lowStock() {
        return (root, query, cb) -> cb.lessThan(root.get("currentQuantity"), root.get("reorderThreshold"));
    }
}

// استخدامها
Specification<Sku> spec = Specification.where(hasCategory("Electronics")).and(lowStock());
repository.findAll(spec);
```

ده أنضف حل لما عندك **فلاتر ديناميكية** (يعني مش عارف مقدمًا الـ user هيبعت أنهي فلاتر).

#### 4.2 Criteria API (اللي `Specification` مبني فوقيه فعليًا)

```java
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<Sku> query = cb.createQuery(Sku.class);
Root<Sku> root = query.from(Sku.class);
query.where(cb.equal(root.get("category"), "Electronics"));
```

type-safe (compile-time checked)، لكن verbose جدًا. **مفيد تفهمه عشان تفهم الـ Specification من تحته، لكن مش هتكتبه يدويًا كتير في الشغل الحقيقي.**

#### 4.3 QueryDSL (المصطلح التاني اللي سمعته)

مكتبة خارجية (مش جزء من Spring نفسه) بتديك syntax أنضف من الـ Criteria API، باستخدام Q-classes بتتولد تلقائيًا:

```java
QSku sku = QSku.sku;
queryFactory.selectFrom(sku)
    .where(sku.category.eq("Electronics").and(sku.currentQuantity.lt(sku.reorderThreshold)))
    .fetch();
```

**ملحوظة مهمة:** QueryDSL بقت أقل استخدامًا في المشاريع الجديدة (المشروع نفسه توقف الصيانة الرسمية شوية)، فمعظم الناس دلوقتي بيفضلوا **Specification API** (built-in في Spring) بدل ما يجيبوا dependency خارجي. اتعلمها عشان تفهمها لو لقيتها في مشروع legacy، بس ركز أكتر على Specification.

---

## الترتيب الكامل للمذاكرة (الخلاصة)

```
1. JPQL syntax كامل (SELECT, JOIN, subqueries, aggregate functions)
2. Fetch strategies: LAZY vs EAGER, وليه LAZY هي الافتراضي الصح
3. N+1 Problem + الحلول (JOIN FETCH, @EntityGraph, @BatchSize, DTO projections)
4. Transactions: @Transactional, propagation, isolation levels
5. Locking: Optimistic (@Version) → Pessimistic (PESSIMISTIC_WRITE/READ)
6. Specification API (dynamic queries) — الأولوية هنا
7. Criteria API (فهم نظري، مش لازم تتقنه كتابة)
8. Native Queries + Postgres-specific features (window functions, JSONB queries)
9. Query Performance: EXPLAIN ANALYZE في Postgres نفسه (مش جزء من Spring، لكن ضروري تفهمه عشان تعرف query بطيء ليه)
10. Pagination الصحيح مع Sorting (Pageable) وتأثيره على الأداء مع joins
```

---

## إزاي تقوي نفسك فيه عمليًا (مش بس نظري)

1. **رجع لمشروع StockSavvy بتاعك بعد ما تخلص الـ Ledger والـ relations، واعمل تجربة عملية:** فعّل `show-sql` واستخدم endpoint بيجيب POs مع الـ vendor والـ line items، وشوف بنفسك هل بتقع في N+1 ولا لأ.
2. **حط breakpoint أو log قبل وبعد كل query معقدة، واتأكد من عدد الـ queries الفعلي.**
3. **اعمل benchmark بسيط:** query بدون `JOIN FETCH` مقابل نفسها بيه، وشوف الفرق في عدد الاستدعاءات بعينك.
4. **جرب تعمل نفس الـ Reorder Agent query** (اللي بيدور على أرخص vendor لـ SKU معين) بـ Specification API، عشان تتمرن على الفلاتر الديناميكية.

الجزء ده كله هو أكتر حاجة بتفرق بين مطور Junior وSenior في أي إنترفيو Spring — لو حسيت إنك فاهمه كويس، هيبان في أي سؤال تقني ليك.