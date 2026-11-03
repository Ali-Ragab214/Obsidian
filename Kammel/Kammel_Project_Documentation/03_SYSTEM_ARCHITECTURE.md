# Kammel — System Architecture

> **Document Version:** 1.0  
> **Last Updated:** November 2026  
> **Related:** [04_DOMAIN_MODEL.md](./04_DOMAIN_MODEL.md) · [05_DATABASE_DESIGN.md](./05_DATABASE_DESIGN.md) · [06_SECURITY_AND_MULTI_TENANCY.md](./06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. Architectural Style: Modular Monolith

**Decision:** Kammel is built as a **Modular Monolith** — a single deployable application with clearly defined internal module boundaries.

| Factor | Modular Monolith | Microservices |
|--------|-----------------|---------------|
| Developer count | ✅ One developer | ❌ Requires team |
| Deployment complexity | ✅ Single artifact | ❌ Multiple services, orchestration |
| Transactional consistency | ✅ Single DB transaction | ❌ Distributed transactions |
| Debugging | ✅ Single process | ❌ Distributed tracing |
| Operational cost | ✅ One VPS | ❌ Multiple containers/services |
| Future extraction | ✅ Clean boundaries enable later extraction | — |

**Trade-off:** We lose independent deployment per module but gain simplicity, consistency, and development speed — all critical for a single-developer project.

**Boundary enforcement:** Use **Spring Modulith** in CI to verify that modules respect their declared dependencies. This prevents the monolith from degrading into a "big ball of mud."

---

## 2. Feature-Based Package Structure

### 2.1 Top-Level Structure

```
com.kammel/
├── KammelApplication.java
├── common/                    # Shared infrastructure
│   ├── config/                # Security, Redis, Flyway, OpenAPI, async configs
│   ├── exception/             # Global exception handler, error DTOs
│   ├── security/              # JWT filter, tenant context, UserPrincipal
│   ├── audit/                 # Audit interceptor, AuditLog entity & repository
│   ├── validation/            # Custom validators (@ValidPhone, etc.)
│   ├── dto/                   # Shared DTOs: PageResponse, ErrorResponse
│   └── util/                  # MoneyUtil, PhoneUtil, DateUtil
│
├── identity/                  # Authentication, users, roles
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── dto/request/
│   ├── dto/response/
│   ├── entity/
│   ├── enums/
│   └── mapper/
│
├── tenancy/                   # Organizations, branches
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── dto/
│   ├── entity/
│   └── mapper/
│
├── members/                   # Member management, Excel import
├── plans/                     # Membership plan templates
├── subscriptions/             # Subscription lifecycle, events
├── payments/                  # Payments, refunds, shifts
├── checkin/                   # Attendance check-in
├── notifications/             # Message queue, channels
├── risk/                      # Risk score computation
├── reporting/                 # Dashboard queries, KPIs
└── leads/                     # P1: Lead management
```

### 2.2 Within-Module Structure (Example: subscriptions)

```
subscriptions/
├── controller/
│   └── SubscriptionController.java
├── service/
│   ├── SubscriptionService.java          # Public interface
│   └── SubscriptionServiceImpl.java      # Implementation
├── repository/
│   └── SubscriptionRepository.java
│   └── SubscriptionEventRepository.java
├── dto/
│   ├── request/
│   │   ├── CreateSubscriptionRequest.java
│   │   ├── RenewSubscriptionRequest.java
│   │   └── FreezeSubscriptionRequest.java
│   └── response/
│       ├── SubscriptionResponse.java
│       └── SubscriptionEventResponse.java
├── entity/
│   ├── Subscription.java
│   └── SubscriptionEvent.java
├── enums/
│   ├── SubscriptionStatus.java
│   └── SubscriptionEventType.java
├── mapper/
│   └── SubscriptionMapper.java
└── exception/
    └── InvalidStateTransitionException.java
```

### 2.3 Module Responsibilities

| Module | Owns | Public API |
|--------|------|-----------|
| **common** | Shared config, security infra, audit, utilities | Utility classes, base DTOs |
| **identity** | Users, authentication, JWT, password hashing | `AuthService`, `UserService` |
| **tenancy** | Organizations, branches, tenant lifecycle | `OrganizationService`, `BranchService` |
| **members** | Member records, Excel import | `MemberService`, `ImportService` |
| **plans** | Membership plan definitions | `PlanService` |
| **subscriptions** | Subscription lifecycle, state machine, events | `SubscriptionService` |
| **payments** | Payment records, refunds, cash shifts | `PaymentService`, `ShiftService` |
| **checkin** | Check-in records, QR validation | `CheckInService` |
| **notifications** | Message queue, templates, channel adapters | `NotificationService` |
| **risk** | Risk score computation, signals | `RiskScoreService` |
| **reporting** | Dashboard aggregations, KPIs | `DashboardService` |
| **leads** | Lead funnel (P1) | `LeadService` |

### 2.4 Module Communication Rules

1. **Modules communicate ONLY through public Service Interfaces** — never by accessing another module's Repository directly.
2. **Dependency direction** flows downward. Upper modules depend on lower modules, never the reverse.
3. **Spring Application Events** for decoupled cross-module communication (e.g., `PaymentReceivedEvent` triggers subscription activation).
4. **No circular dependencies** between modules.

### 2.5 Dependency Graph

```mermaid
graph TD
    common["common"]
    identity["identity"]
    tenancy["tenancy"]
    members["members"]
    plans["plans"]
    subscriptions["subscriptions"]
    payments["payments"]
    checkin["checkin"]
    notifications["notifications"]
    risk["risk"]
    reporting["reporting"]

    identity --> common
    tenancy --> common
    tenancy --> identity
    members --> common
    members --> tenancy
    plans --> common
    plans --> tenancy
    subscriptions --> common
    subscriptions --> members
    subscriptions --> plans
    payments --> common
    payments --> subscriptions
    payments --> members
    checkin --> common
    checkin --> members
    checkin --> subscriptions
    notifications --> common
    notifications --> members
    notifications --> subscriptions
    risk --> common
    risk --> members
    risk --> subscriptions
    risk --> checkin
    reporting --> common
    reporting --> members
    reporting --> subscriptions
    reporting --> payments
    reporting --> checkin
    reporting --> risk
```

The `reporting` module depends on many others because it aggregates data for dashboards. It only reads through service interfaces — never writes.

### 2.6 Shared Components (`common`)

The `common` module contains infrastructure that all modules use:

- **Security config:** JWT filter, CORS, method security
- **Tenant context:** `TenantContext` (ThreadLocal), tenant-setting interceptor
- **Audit:** `AuditLog` entity and logging interceptor
- **Exception handling:** `GlobalExceptionHandler` with standard error format
- **Validation:** Custom constraint annotations
- **DTOs:** `PageResponse<T>`, `ErrorResponse`
- **Utilities:** `MoneyUtil` (piasters ↔ display), `PhoneUtil` (format validation), `DateUtil` (timezone conversion)

---

## 3. Layer Responsibilities

| Layer | Responsibility | Rules |
|-------|---------------|-------|
| **Controller** | Accept HTTP requests, validate input, delegate to service, return DTOs | No business logic. No direct repository calls. Thin layer. |
| **Service Interface** | Define public contract for the module | Java interface. Only input/output DTOs — never entities. |
| **Service Implementation** | Business logic, orchestration, transaction management, authorization | @Transactional. @PreAuthorize. Domain rule enforcement. |
| **Repository** | Data access via Spring Data JPA | Interface extending JpaRepository. Custom @Query where needed. No business logic. |
| **Entity** | JPA entity mapping to database table | @Entity. Constraints. Relationships. Audit columns. org_id. |
| **DTO** | Request validation + response shaping | Bean Validation annotations on request DTOs. Response DTOs with formatted data. |
| **Mapper** | Convert between Entity and DTO | Stateless. Can be manual methods or MapStruct. |
| **Enums** | Business enumerations | Status values, types, methods. |
| **Exception** | Module-specific business exceptions | Extend RuntimeException. Caught by GlobalExceptionHandler. |

---

## 4. Request Processing Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant SF as Security Filter Chain
    participant JWT as JWT Filter
    participant TC as Tenant Context
    participant RLS as PostgreSQL RLS
    participant Ctrl as Controller
    participant Svc as Service
    participant Repo as Repository
    participant DB as PostgreSQL

    C->>SF: HTTP Request + Bearer Token
    SF->>JWT: Validate JWT
    JWT->>JWT: Check signature, expiry, claims
    JWT->>TC: Set TenantContext(org_id, branch_id, role)
    TC->>RLS: SET LOCAL app.current_org = org_id
    SF->>Ctrl: Authenticated request
    Ctrl->>Ctrl: Validate request DTO (@Valid)
    Ctrl->>Svc: Call service method
    Svc->>Svc: @PreAuthorize check
    Svc->>Svc: Business logic + validation
    Svc->>Repo: Database operation
    Repo->>DB: SQL query (RLS applied automatically)
    DB-->>Repo: Filtered results (tenant-scoped)
    Repo-->>Svc: Entity/data
    Svc-->>Ctrl: Response DTO
    Ctrl-->>C: HTTP Response
    Note over TC: TenantContext.clear() after request
```

Every request passes through: authentication → tenant context setup → authorization → business logic → tenant-filtered data access.

---

## 5. Authentication & Authorization Flow

```mermaid
sequenceDiagram
    participant U as User
    participant Auth as AuthController
    participant AS as AuthService
    participant DB as Database
    participant JWT as JWT Utility

    U->>Auth: POST /auth/login {phone, password}
    Auth->>AS: authenticate(phone, password)
    AS->>DB: Find user by phone
    DB-->>AS: User entity
    AS->>AS: Verify password (bcrypt)
    AS->>AS: Check user.active && org.status == ACTIVE
    AS->>JWT: Generate access token (15 min)
    AS->>JWT: Generate refresh token (7 days)
    AS->>DB: Store refresh token hash
    AS-->>Auth: AuthResponse{accessToken, refreshToken, user}
    Auth-->>U: 200 OK

    Note over U: Later, access token expires...

    U->>Auth: POST /auth/refresh {refreshToken}
    Auth->>AS: refresh(token)
    AS->>DB: Validate refresh token (not revoked, not expired)
    AS->>JWT: Generate new access token
    AS->>JWT: Rotate refresh token (old revoked)
    AS-->>Auth: New AuthResponse
    Auth-->>U: 200 OK
```

---

## 6. Subscription Workflow

```mermaid
stateDiagram-v2
    [*] --> PENDING_PAYMENT : Create Subscription
    PENDING_PAYMENT --> ACTIVE : Full Payment Received
    PENDING_PAYMENT --> CANCELLED : Cancel Before Payment

    ACTIVE --> FROZEN : Freeze (reason required)
    ACTIVE --> GRACE : End Date Reached (grace > 0)
    ACTIVE --> EXPIRED : End Date Reached (grace = 0)
    ACTIVE --> ACTIVE : Renew Before Expiry
    ACTIVE --> CANCELLED : Cancel by Manager

    FROZEN --> ACTIVE : Unfreeze (end date extended)
    FROZEN --> CANCELLED : Cancel While Frozen

    GRACE --> ACTIVE : Renew During Grace
    GRACE --> EXPIRED : Grace Period Ended

    EXPIRED --> [*]
    CANCELLED --> [*]

    note right of ACTIVE : EXPIRING_SOON is computed\n(≤7 days remaining)\nnot a stored state
```

Each transition creates an immutable `SubscriptionEvent` record with actor, timestamp, reason, and before/after metadata.

---

## 7. Payment Workflow

```mermaid
sequenceDiagram
    participant R as Receptionist
    participant PC as PaymentController
    participant PS as PaymentService
    participant SS as SubscriptionService
    participant DB as Database

    R->>PC: POST /payments {subscriptionId, amount, method, idempotencyKey}
    PC->>PS: recordPayment(request)
    PS->>DB: Check idempotency key exists
    alt Key exists
        DB-->>PS: Existing payment
        PS-->>PC: Return existing (idempotent)
    else New payment
        PS->>DB: Find subscription (with lock)
        PS->>PS: Validate: tenant, amount > 0, subscription status
        PS->>DB: Save Payment record
        PS->>PS: Calculate total paid vs price snapshot
        alt Total >= Price
            PS->>SS: activateOrRenew(subscription)
            SS->>DB: Update subscription status
            SS->>DB: Create SubscriptionEvent
        else Partial
            PS->>PS: Track remaining balance
        end
        PS-->>PC: PaymentResponse
    end
    PC-->>R: 201 Created
```

---

## 8. Notification Workflow

```mermaid
sequenceDiagram
    participant Job as Scheduled Job
    participant NS as NotificationService
    participant DB as Database
    participant R as Receptionist
    participant WA as WhatsApp (wa.me)

    Job->>NS: generateRenewalReminders()
    NS->>DB: Find expiring subscriptions (7 days)
    NS->>NS: Check duplicate prevention (no recent message)
    NS->>NS: Render template with member data
    NS->>DB: Save MessageQueueItem (PENDING)

    Note over R: Later, receptionist opens queue...

    R->>NS: GET /notifications/queue
    NS->>DB: Find PENDING messages
    DB-->>NS: Messages list
    NS-->>R: Message queue with wa.me links

    R->>WA: Click wa.me link (opens WhatsApp)
    R->>NS: PATCH /notifications/{id}/sent
    NS->>DB: Update status to SENT_CONFIRMED

    Note over NS: 7 days later, if member renews...
    NS->>DB: Set attributed_subscription_id on message
```

---

## 9. Background Job Workflow

```mermaid
flowchart TD
    A[Scheduler Trigger - 2 AM Daily] --> B{ShedLock: Acquire Lock?}
    B -->|No - Another instance holds lock| Z[Skip execution]
    B -->|Yes - Lock acquired| C[Load active organizations]
    C --> D[For each organization]
    D --> E[Set TenantContext org_id]
    E --> F[Set RLS context: SET LOCAL app.current_org]
    F --> G[Execute job logic for this org]
    G --> H{Success?}
    H -->|Yes| I[Log summary for org]
    H -->|No| J[Log error, continue to next org]
    I --> K{More organizations?}
    J --> K
    K -->|Yes| D
    K -->|No| L[Clear TenantContext]
    L --> M[Release ShedLock]
    M --> N[Job complete]
```

**Critical:** Each organization processed in its own transaction with explicit tenant context. One org failing does not block others. `TenantContext.clear()` in finally block.

---

## 10. Transaction Management

| Rule | Description |
|------|-------------|
| `@Transactional` on service methods | Not on controllers, not on repositories |
| Read-only for queries | `@Transactional(readOnly = true)` for GET operations |
| Atomic financial operations | Payment + subscription status update in single transaction |
| Tenant context per transaction | `SET LOCAL app.current_org` at transaction start for RLS |
| Optimistic locking | `@Version` on entities modified concurrently |
| Pessimistic locking | `SELECT FOR UPDATE` for subscription payment operations |

---

## 11. Exception Handling

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    // ResourceNotFoundException → 404
    // DuplicateResourceException → 409
    // InvalidStateTransitionException → 422
    // AccessDeniedException → 403
    // TenantMismatchException → 404 (not 403, prevents ID enumeration)
    // MethodArgumentNotValidException → 400 with field details
    // Exception → 500 (generic, no stack trace in production)
}
```

### Standard Error Response

```json
{
  "timestamp": "2026-11-03T12:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "path": "/api/v1/members",
  "details": [
    { "field": "phone", "message": "Phone number is required" }
  ]
}
```

---

## 12. Validation Strategy

| Level | Mechanism | Purpose |
|-------|-----------|---------|
| **DTO** | Bean Validation (`@NotBlank`, `@Size`, `@Pattern`) | Input format validation |
| **Custom Validators** | `@ValidPhone`, `@ValidDateRange` | Business format rules |
| **Service** | Programmatic checks | Cross-entity business rules (e.g., freeze days remaining) |
| **Database** | CHECK constraints, UNIQUE, FK | Final safety net |

---

## 13. Caching Strategy

**Not included in MVP.** When introduced, Redis caches plan lists, org config, and dashboard aggregates with tenant-aware cache keys (`kammel:{org_id}:{entity}:{id}`). See [09_REDIS_CACHING.md](./09_REDIS_CACHING.md).

---

## 14. Deployment Architecture

```mermaid
graph LR
    Client[Browser / PWA] --> Nginx[Nginx Reverse Proxy]
    Nginx --> App[Spring Boot App - Docker]
    App --> PG[(PostgreSQL - Docker)]
    App -.-> Redis[(Redis - Docker, when added)]
    GHA[GitHub Actions] --> |Build + Test + Deploy| VPS[VPS Server]
    VPS --> Nginx
    Backup[Daily Backup Job] --> ExtStorage[External Storage]
    PG --> Backup
    App --> Sentry[Sentry / Monitoring]
```

### Docker Compose Services

| Service | Image | Purpose |
|---------|-------|---------|
| `app` | Custom Spring Boot image | Application |
| `postgres` | postgres:16 | Primary database |
| `redis` | redis:7 (when added) | Cache |

### Environment Profiles

| Profile | Purpose | Configuration |
|---------|---------|--------------|
| `dev` | Local development | H2 or local PostgreSQL, debug logging |
| `test` | CI testing | Testcontainers, test data |
| `staging` | Pre-production | Production-like, test data |
| `prod` | Production | Production database, info logging, HTTPS |
