---
title: Performance and Scalability
cssclasses: [rtl]
tags: [performance, scalability, hikari, caching]
section: 13
---

# 13. Performance and Scalability
⬅️ [[12 - Security Considerations]] | ➡️ [[14 - Testing]]

## 13.1 الـ Configuration الحالية

| Setting | القيمة | التقييم |
|---|---|---|
| **Hikari maximum pool size** | 10 | بداية معقولة؛ حدّد الحجم حسب الـ instance والـ workload |
| **Minimum idle** | 5 | مناسب لـ deployment صغير |
| **Connection timeout** | 20 s | مناسب |
| **Idle timeout** | 30 s | 🟡 **عدواني**؛ ممكن يعمل churn للـ connections |
| **Max lifetime** | 20 min | معقول |
| **JDBC batch size** | 20 | ✅ كويس للـ bulk inserts |
| **`order_inserts` / `order_updates`** | `true` | ✅ كويس |
| **`open-in-view`** | `false` | ✅ كويس؛ بيمنع lazy loading في الـ view layer |
| **`show-sql`** | `true` | 🔴 **وحش للـ production**؛ مزعج وممكن يكشف بيانات |
| **`format_sql` / `use_sql_comments`** | `true` | Development فقط |
| **DDL auto** | `none` | ✅ صح؛ Flyway هو مالك الـ schema |
| **Pagination** | `PageRequest.of(page, size)` | مفيش cap ولا sorting |

## 13.2 مشاكل الـ Queries والـ Indexes

- Indexes ناقصة على `stock_mvts.product_id` و `stock_mvts.date_mvt` و `products.category_id` و `users.tenant_id`.
- أي حساب balance مستقبلي بـ `SUM` على الـ ledger كله **هيتدهور خطيًا** مع زيادة التاريخ.
- `Product.category` بتتجلب **eagerly** وبتتحوّل لكل صف؛ استخدم **projections** في الـ list endpoints.
- مفيش **N+1 كلاسيكي مثبت وقت التشغيل** لأن مفيش product endpoint والـ build مابيشتغلش. **لكن** الـ eager association + الـ per-row DTO mapping = **خطر latent** لـ N+1/lazy-loading؛ تحقق بـ SQL logging وـ query-count test لما الـ endpoint يتعمل.
- `PageResponse` بيحوّل entities كاملة لـ DTOs في الذاكرة؛ للقوائم الكبيرة استخدم **query projection** والأعمدة المطلوبة بس.
- مفيش `@EntityGraph` / join fetch strategy.
- مفيش **query timeout** ولا slow-query logging.

## 13.3 أداء الـ Transactions والـ Locking

- الـ transactions الطويلة اللي ماسكة row locks على **SKUs ساخنة** هتعمل **serialize** للعمليات الخارجة. خلّي الجزء المقفول أقل ما يمكن.
- **ماتنادّيش external services** (email, AI, webhooks) جوه transaction.
- استخدم **outbox pattern** للـ events عشان النشر يبقى موثوق من غير distributed transactions.
- فكّر في **atomic conditional updates** لعمليات `OUT` عالية الـ contention، و **pessimistic locks** للـ transfers/adjustments.
- راقب **lock waits و deadlocks**؛ اظبط `lock_timeout` للـ interactive requests.

## 13.4 الـ Caching

مفيش cache متظبط. `TenantSchemaResolver` عليه `@Cacheable` لكن التطبيق **ما فعّلش الـ caching**، فالـ annotation **ملهاش تأثير**.

### التوصيات
- اعمل cache لـ **tenant metadata** (id → schema/status) في cache صغير محدود الحجم مع **eviction صريح** عند تغيير الـ tenant.
- اعمل cache لـ **reference data** (categories, قوايم المخازن) بـ TTL قصير ومفاتيح tenant-scoped.
- **ماتعملش cache للـ balances** من غير استراتيجية invalidation مربوطة بالـ postings.
- ضيف **Redis** بس لما يبقى فيه احتياج multi-instance لـ shared cache أو rate limiting أو distributed locks؛ **Caffeine محلي كفاية في الأول**.
- **عمرك ما تستخدم cache كمصدر حقيقة للمخزون.**

## 13.5 Pagination والـ Filtering

- فرض **حد أقصى لحجم الصفحة** (مثلًا 100) و **default** (مثلًا 20).
- **Allowlist** للحقول القابلة للترتيب لمنع الـ injection والترتيبات الغالية.
- ضيف filters بـ product, warehouse, movement type, status, date range, actor.
- استخدم **keyset pagination** لتاريخ الحركات الكبير بدل الـ offsets العميقة.
- رجّع `totalElements` **بس لما تحتاجه**؛ العدّ على جداول كبيرة غالي.

## 13.6 المعالجة الـ Async والـ Background

مفيش background processing حاليًا. المنتج هيحتاج:

- **Tenant provisioning** (DDL/Flyway طويلة).
- توليد التقارير والـ exports.
- إشعارات low-stock والـ anomalies.
- Jobs للـ forecast والـ reconciliation.
- **Outbox relay** وتسليم webhooks.

ابدأ بـ `@Scheduled` / `TaskExecutor` وجدول **job/outbox** على الـ database. **ضيف broker بس لما الـ throughput أو تنسيق الـ multi-service يبرر.** ماتدخلش **Kafka** لـ monolith واحد.

## 13.7 مسار الـ Scalability

1. **Stateless application instances** ورا load balancer. الـ JWT بيتفادى server-side sessions.
2. **Connection budgeting:** كل instance بيفتح لحد 10 connections؛ N instances بيضاعفوا الرقم. استخدم **pooler (PgBouncer)** أو قلّل pools لكل instance مع التوسع.
3. **Read replicas** للـ reporting مع routing صريح وانتباه لـ replica lag.
4. **Partitioning** لجداول الحركات/الـ audit بالـ tenant أو الوقت لما تكبر.
5. **Archival** للحركات القديمة لـ cold storage مع الحفاظ على الوصول للـ audit.
6. **Per-tenant rate limits و quotas** لمنع الـ noisy neighbours.
7. **Tenant-aware observability** لتحديد الـ tenants والـ queries الغالية.
8. **Operational tooling للـ schema-per-tenant** (migrations, backups, restore) لو الموديل ده اتحافظ عليه.

## 13.8 القياس

> [!warning] مفيش performance metrics النهاردة، فمينفعش نذكر أي أرقام throughput أو latency.

قبل أي optimisation:
- ضيف **Actuator/Micrometer** و database metrics.
- Instrument لـ endpoint الـ posting واستعلامات الـ balance والتقارير.
- اعمل **baselines** بـ dataset واقعي (منتجات، مخازن، سنين من الحركات).
- **Load-test** لـ `OUT` متزامنة على نفس الـ SKU، و transfer deadlocks، واستعلامات التقارير الكبيرة.
- تتبّع **p50 / p95 / p99** و error rate و lock wait time و pool saturation.

الأرقام الأساسية (baselines) **TBD** لحد ما القياسات تتوفر.

---
⬅️ [[12 - Security Considerations]] | ➡️ [[14 - Testing]]
