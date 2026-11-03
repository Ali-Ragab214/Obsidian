# Kammel — Redis Caching Strategy

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [03_SYSTEM_ARCHITECTURE.md](./03_SYSTEM_ARCHITECTURE.md) · [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. Architectural Strategy & Phasing

### 1.1 The Post-MVP Decision
In alignment with the Kammel Evaluation & Technical Recommendations document, **Redis is explicitly excluded from the initial P0 MVP**. 

During early pilot deployments (serving 1 to 5 gyms with 100 to 500 members each), PostgreSQL with proper indexes, connection pooling, and sub-second query execution comfortably handles the entire read/write load. Introducing Redis prematurely increases infrastructure cost, deployment moving parts, and risk of cache invalidation bugs.

Redis is introduced in **Phase 2 (P1/P2)** when:
1. P95 latency on dashboard aggregation queries begins to exceed 500ms under multi-tenant load.
2. Read-heavy public endpoints (e.g., plan listings, member search at reception) create repetitive database roundtrips.
3. Rapid rate limiting or JWT distributed blacklisting requires high-speed in-memory state tracking.

---

## 2. What to Cache vs. What NOT to Cache

| Data Category | Cached? | Target TTL | Justification & Cache Key |
|---|:---:|:---:|---|
| **Active Membership Plans** | ✅ YES | 15–30 min | Read frequently during subscription creation; rarely modified.<br>`kammel:{org_id}:plans:active` |
| **Organization & Branch Config** | ✅ YES | 60 min | Near-static metadata read on almost every request.<br>`kammel:{org_id}:config` |
| **Dashboard Executive Summaries** | ✅ YES | 5–10 min | Expensive multi-table aggregations; 5-minute staleness is business-acceptable.<br>`kammel:{org_id}:dashboard:summary` |
| **Nightly Risk Scores** | ✅ YES | 12–24 hrs | Computed once nightly; read repeatedly throughout the day.<br>`kammel:{org_id}:risk:summary` |
| **JWT Revocation Blacklist** | ✅ YES | Matches Token Expiry | Fast distributed lookup for emergency logout.<br>`kammel:global:jwt-blacklist:{jti}` |
| **Rate Limiter Counters** | ✅ YES | 1 min | Rolling sliding-window rate tracking.<br>`kammel:global:ratelimit:{ip}` |
| **Payment Records & Invoices** | ❌ **NEVER** | None | **Strict Invariant:** Financial transactions must always reflect PostgreSQL ground truth. |
| **Subscription Current Status** | ❌ **NEVER** | None | Live state transitions govern check-in access; stale cache could admit expired members. |
| **Cash Drawer Balances** | ❌ **NEVER** | None | Real-time shift reconciliation requires exact database accuracy. |
| **Member Personal Details (PII)** | ❌ **NEVER** | None | Minimizes plaintext PII surface area outside the primary encrypted DB. |

---

## 3. Tenant-Aware Cache Key Naming Convention

A critical vulnerability in multi-tenant caching is cross-tenant data leakage: if Gym A caches plans under `plans:active`, Gym B could read Gym A's pricing. 

In Kammel, **every cache key for tenant-owned data is strictly namespaced with `{org_id}`**:

$$\text{Key Pattern} = \texttt{kammel:\{org\_id\}:\{module\}:\{resource\}:[identifier]}$$

### Examples:
- Active plans for Organization `a1b2c3d4`:  
  `kammel:a1b2c3d4:plans:active`
- Branch configuration for Branch `e5f6a7b8`:  
  `kammel:a1b2c3d4:branches:e5f6a7b8:config`
- Member risk score for Member `99887766`:  
  `kammel:a1b2c3d4:risk:member:99887766`
- Global system rate limit (not tenant-scoped):  
  `kammel:global:ratelimit:192.168.1.50`

---

## 4. Spring Cache Annotations & Implementation

### 4.1 `@Cacheable` (Read-Through Cache)
```java
@Service
public class PlanServiceImpl implements PlanService {

    @Override
    @Cacheable(
        value = "plans",
        key = "T(com.kammel.common.security.TenantContext).getOrgId() + ':active'",
        unless = "#result.isEmpty()"
    )
    @Transactional(readOnly = true)
    public List<PlanResponse> getActivePlans() {
        UUID orgId = TenantContext.getOrgId();
        return planRepository.findByOrgIdAndActiveTrue(orgId)
                .stream()
                .map(planMapper::toResponse)
                .toList();
    }
}
```

### 4.2 `@CacheEvict` (Write-Invalidate Cache)
When an owner creates, updates, or deactivates a plan, the cache for that specific organization is immediately cleared:
```java
@Override
@CacheEvict(
    value = "plans",
    key = "T(com.kammel.common.security.TenantContext).getOrgId() + ':active'"
)
@Transactional
public PlanResponse createPlan(CreatePlanRequest request) {
    // Persist new plan to PostgreSQL
    Plan plan = planMapper.toEntity(request);
    plan.setOrgId(TenantContext.getOrgId());
    return planMapper.toResponse(planRepository.save(plan));
}
```

---

## 5. Cache Consistency & Stampede Protection

### 5.1 Write-Through vs. Cache-Aside
Kammel uses the **Cache-Aside (Lazy-Loading)** pattern with proactive cache eviction on mutation:
1. Application queries cache.
2. If hit $\to$ return payload immediately.
3. If miss $\to$ query PostgreSQL, populate cache, return payload.
4. On mutation (insert/update/delete) $\to$ execute database transaction first, then evict cache key.

### 5.2 Cache Stampede Prevention
For expensive dashboard summaries that expire every 5 minutes:
1. **Background Refresh Job:** A Spring `@Scheduled` background worker queries the database and pre-warms the cache 30 seconds before TTL expiration.
2. **Distributed Mutex (SETNX Lock):** If a cache miss occurs under high concurrent traffic, the first request acquires a short-lived Redis lock (`SET lock_key uuid NX EX 5`) to compute and populate the cache. Competing threads wait 100ms and read from the freshly populated cache rather than overwhelming PostgreSQL with identical expensive queries.

---

## 6. Resilience & Graceful Degradation

Redis is an ephemeral performance accelerator, **never an authoritative data store**. If the Redis container crashes or becomes unresponsive:
1. Spring Boot's `RedisCacheErrorHandler` intercepts the connection timeout.
2. The error is logged as a warning (`WARN: Redis unavailable, falling back to PostgreSQL`).
3. The query bypasses the cache and executes directly against PostgreSQL.
4. The application continues running without user-visible HTTP 500 errors.

---

## 7. Spring Boot Configuration (`application-prod.yml`)

```yaml
spring:
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 16
          max-idle: 8
          min-idle: 2
          max-wait: 1000ms
  cache:
    type: redis
    redis:
      time-to-live: 600000 # Default TTL: 10 minutes
      key-prefix: "kammel:"
      use-key-prefix: true
      cache-null-values: false
```
