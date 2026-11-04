# Audit — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.audit.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. `AuditLog`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL` | Tenant |
| `actorType` | `ActorType` | `NOT NULL` | `USER` / `SYSTEM` |
| `actorId` | `UUID` | `NULL` | `NULL` للـ System |
| `action` | `String` | `NOT NULL`, ≤ 50 | من الكتالوج |
| `entityType` | `String` | `NOT NULL`, ≤ 50 | `MEMBER`, `SUBSCRIPTION` ... |
| `entityId` | `UUID` | `NOT NULL` | — |
| `beforeState` | `JsonNode` | `NULL` | JSONB |
| `afterState` | `JsonNode` | `NULL` | JSONB |
| `reason` | `String` | `NULL`, ≤ 500 | إلزامي لعمليات معينة |
| `ipAddress` | `String` | `NULL`, ≤ 45 | IPv4/IPv6 |
| `userAgent` | `String` | `NULL`, ≤ 255 | — |
| `traceId` | `String` | `NULL`, ≤ 64 | ربط بالـ Logs |
| `createdAt` | `Instant` | `NOT NULL` | — |

```java
@Entity
@Table(name = "audit_log")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class AuditLog {

    @Id
    private UUID id;

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Enumerated(EnumType.STRING)
    @Column(name = "actor_type", nullable = false, length = 10)
    private ActorType actorType;

    @Column(name = "actor_id")
    private UUID actorId;

    @Column(nullable = false, length = 50)
    private String action;

    @Column(name = "entity_type", nullable = false, length = 50)
    private String entityType;

    @Column(name = "entity_id", nullable = false)
    private UUID entityId;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "before_state", columnDefinition = "jsonb")
    private JsonNode beforeState;

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "after_state", columnDefinition = "jsonb")
    private JsonNode afterState;

    @Column(length = 500)
    private String reason;

    @Column(name = "ip_address", length = 45)
    private String ipAddress;

    @Column(name = "user_agent", length = 255)
    private String userAgent;

    @Column(name = "trace_id", length = 64)
    private String traceId;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt;

    public static AuditLog create(AuditCommand cmd) { ... }
}
```

> **مفيش Setters** — الكيان Append-Only.

---

## 2. الـ Enums

```java
public enum ActorType {
    USER,
    SYSTEM
}

public enum AuditEntityType {
    ORGANIZATION, BRANCH, USER, MEMBER, PLAN,
    SUBSCRIPTION, PAYMENT, CHECK_IN, MESSAGE, RISK_SCORE
}
```

> `action` مخزّن كـ `String` مش Enum في الـ DB — عشان نضيف Actions جديدة بدون Migration. الـ Enum في الكود للـ Type Safety.

---

## 3. `AuditCommand`

```java
public record AuditCommand(
    String action,
    String entityType,
    UUID entityId,
    Object beforeState,
    Object afterState,
    String reason
) {}
```

- `orgId` / `actorId` / IP / TraceId بتتاخد من الـ Context — **مش من الـ Caller**.
- `beforeState` / `afterState` بيتحولوا لـ JSON بـ Jackson.
- ممنوع تمرير Entity كاملة — استخدم Snapshot مختصر.

---

## 4. `AuditService`

```java
public interface AuditService {

    void record(AuditCommand command);

    void recordSystem(AuditCommand command);

    void recordBestEffort(AuditCommand command);
}
```

| Method | السلوك | الاستخدام |
|---|---|---|
| `record` | نفس Transaction — فشله يفشل العملية | Freeze, Payment, Cancel, Override |
| `recordSystem` | `actorType = SYSTEM`, `actorId = null` | Scheduler Expiration |
| `recordBestEffort` | `REQUIRES_NEW` + Catch + Log | `LOGIN_FAILED` |

### 4.1 قواعد التسجيل

1. **قبل** تعديل الـ Entity → خد `beforeState`.
2. **بعد** التعديل → خد `afterState`.
3. نادي `record` **جوه** نفس الـ Transaction.
4. لو الـ Audit فشل → Rollback العملية.
5. ممنوع تسجيل Password/Token/Request Body كامل.

### 4.2 مثال

```java
@Transactional
public SubscriptionResponse freeze(UUID subscriptionId, FreezeSubscriptionRequest request) {
    Subscription sub = subscriptionRepository.findByIdForUpdate(subscriptionId)
        .orElseThrow(ResourceNotFoundException::new);

    Map<String, Object> before = Map.of(
        "status", sub.getStatus(),
        "endDate", sub.getEndDate()
    );

    stateMachine.freeze(sub, request.days(), request.reason(), SecurityUtils.getCurrentUserId());
    subscriptionEventRepository.save(SubscriptionEvent.frozen(sub, ...));

    auditService.record(new AuditCommand(
        "SUBSCRIPTION_FROZEN",
        "SUBSCRIPTION",
        sub.getId(),
        before,
        Map.of("status", sub.getStatus(), "endDate", sub.getEndDate()),
        request.reason()
    ));

    return mapper.toResponse(sub);
}
```

---

## 5. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-AUD-01 | ممنوع UPDATE/DELETE | Trigger + Grants |
| INV-AUD-02 | `orgId` من الـ Context | Service |
| INV-AUD-03 | `actorId` من الـ Context | Service |
| INV-AUD-04 | `SYSTEM` → `actorId = NULL` | DB CHECK |
| INV-AUD-05 | `USER` → `actorId != NULL` | DB CHECK |
| INV-AUD-06 | `reason` مطلوب لـ Freeze/Cancel/Override/Reversal | Service |
| INV-AUD-07 | ممنوع PII في `action`/`reason` | Code Review + Test |
| INV-AUD-08 | `beforeState`/`afterState` JSON Valid | Jackson |
| INV-AUD-09 | ممنوع تسجيل Tokens/Passwords | Code Review |
| INV-AUD-10 | كل عملية حساسة لها Action من الكتالوج | Arch/Test |

---

## 6. شكل الـ JSON

### Freeze
```json
{
  "beforeState": {
    "status": "ACTIVE",
    "endDate": "2026-12-10",
    "frozenDaysUsed": 0
  },
  "afterState": {
    "status": "FROZEN",
    "endDate": "2026-12-17",
    "frozenDaysUsed": 7
  }
}
```

### Payment Reversal
```json
{
  "beforeState": { "status": "COMPLETED" },
  "afterState": { "status": "REVERSED", "reversedAt": "2026-11-03T12:00:00Z" }
}
```

### Member Soft Delete
```json
{
  "beforeState": { "deletedAt": null, "status": "ACTIVE" },
  "afterState": { "deletedAt": "2026-11-03T12:00:00Z", "status": "INACTIVE" }
}
```

---

## 7. الـ Context

```java
@Component
public class AuditContext {

    public String currentIp() {
        ServletRequestAttributes attrs =
            (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attrs == null) return null;
        HttpServletRequest req = attrs.getRequest();
        String forwarded = req.getHeader("X-Forwarded-For");
        return forwarded != null ? forwarded.split(",")[0].trim() : req.getRemoteAddr();
    }

    public String currentUserAgent() { ... }

    public String currentTraceId() {
        return MDC.get("traceId");
    }
}
```

> **مهم:** `X-Forwarded-For` بيتاخد من Nginx فقط — لازم Nginx يضبطه ويشيل القيمة الجاية من العميل.

---

## 8. الـ Repository

```java
public interface AuditLogRepository extends JpaRepository<AuditLog, UUID>,
        JpaSpecificationExecutor<AuditLog> {

    List<AuditLog> findAllByOrgIdAndEntityTypeAndEntityIdOrderByCreatedAtDesc(
        UUID orgId, String entityType, UUID entityId);
}
```

- كل Query لازم تتفلتر بـ `orgId` (RLS + Explicit).
- الفلاتر الديناميكية بـ `Specification`.

---

## 9. الاحتفاظ والأرشفة

| البند | القرار |
|---|---|
| Retention | 5 سنوات |
| Archival | P1 — جدول `audit_log_archive` أو Cold Storage |
| Purge | ممنوع قبل المراجعة القانونية |
| Backup | ضمن الـ DB Backup اليومي |

---

## 10. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [database-strategy](../../architecture/database-strategy.md)
