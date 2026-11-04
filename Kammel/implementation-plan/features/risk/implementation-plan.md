# Risk — خطة التنفيذ

> **الـ Phase:** 3 · **المدة:** 5–6 أيام · **الترتيب:** بعد `checkin`  
> **المرتبط:** [`README.md`](./README.md) · [`domain-design.md`](./domain-design.md)

---

## 1. المهام

### TSK-RSK-001 — Migration + Entity

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-CHK-001, TSK-SUB-001 |
| **المخرجات** | `V14__create_risk_scores.sql`, `RiskScore`, `RiskScoreId` |
| **معيار القبول** | RLS + Composite PK + CHECKs + Indexes |

**خطوات:**
1. `risk_scores` بكل الأعمدة.
2. `org_id UUID NOT NULL` — SC-01.
3. `branch_id UUID NOT NULL` — SC-23.
4. `PRIMARY KEY (member_id, computed_at)`.
5. `CHECK (score IS NULL OR score BETWEEN 0 AND 100)`.
6. `CHECK (risk_level IN ('LOW','MEDIUM','HIGH','CRITICAL','INSUFFICIENT_DATA'))`.
7. `SELECT app.enable_tenant_rls('risk_scores')`.
8. Indexes:
   - `(org_id, member_id, computed_at DESC)`
   - `(org_id, risk_level, score DESC) WHERE risk_level IS NOT NULL`
   - `(org_id, branch_id, computed_at DESC)`

---

### TSK-RSK-002 — Scoring Engine

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-001, TSK-CHK-006 |
| **المخرجات** | `RiskScoringService`, `RiskSignalCalculator` |
| **معيار القبول** | 6 إشارات + Score + Level + Reasons |

**خطوات:**
1. `daysSinceLastVisit` — `CheckInQueryService.getLastVisit`.
2. `attendanceDrop` — `countVisits` مرتين.
3. `daysRemaining` — `SubscriptionQueryService.findAccessSubscription`.
4. `paymentFriction` — `countDelayedActivations`.
5. `freezeCount` — `countFreezes`.
6. `tenureDays` — `MemberQueryService`.
7. `min(100, sum)`.
8. `RiskLevel.fromScore`.
9. `reasons` + `signals`.
10. `INSUFFICIENT_DATA` لو < 14 يوم.

---

### TSK-RSK-003 — Recommendation + Draft Message

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-002 |
| **المخرجات** | `RiskRecommendationService`, `DraftMessageRenderer` |
| **معيار القبول** | Action لكل Level + Draft لـ HIGH/CRITICAL |

**خطوات:**
1. Mapping الـ Action.
2. Draft Template.
3. `daysAbsent` + `endDateClause`.
4. `gymName` من `OrganizationQueryService`.
5. ممنوع Draft لـ LOW/MEDIUM/INSUFFICIENT.

---

### TSK-RSK-004 — Nightly Job

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-002, TSK-RSK-003 |
| **المخرجات** | `RiskScoringJob` + ShedLock |
| **معيار القبول** | 3:00 AM + Tenant Scope + Idempotent |

**خطوات:**
1. `@Scheduled(cron = "0 0 3 * * *", zone = "Africa/Cairo")`.
2. `@SchedulerLock(name = "riskScoringJob", lockAtMostFor = "45m")`.
3. `OrganizationQueryService.listActiveOrgIds`.
4. `TenantTransactionRunner.runInTenant`.
5. `listMembersWithAccess`.
6. لكل عضو — `try/catch` + Log + Continue.
7. Upsert على `(member_id, computed_at)`.
8. Test إعادة التشغيل.

---

### TSK-RSK-005 — Query Service

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-004 |
| **المخرجات** | `RiskQueryService` |
| **معيار القبول** | Latest + At-Risk + Summary |

**خطوات:**
1. `getLatestScore` — `ORDER BY computed_at DESC LIMIT 1`.
2. `listAtRisk` — `DISTINCT ON (member_id)` + Filter.
3. Enrich بـ Member + Subscription Value.
4. `getSummary` — Counts + Total Value.
5. `recompute(memberId)` — نفس الـ Engine.
6. Branch Scope.

---

### TSK-RSK-006 — API

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-005 |
| **المخرجات** | `RiskController` + 4 Endpoints |
| **معيار القبول** | [`api-design.md`](./api-design.md) |

**خطوات:**
1. `GET /risk/at-risk`.
2. `GET /risk/members/{memberId}`.
3. `GET /risk/summary`.
4. `POST /risk/recompute`.
5. `@PreAuthorize` + Trainer Check.

---

### TSK-RSK-007 — Tests

| البند | التفصيل |
|---|---|
| **الاعتماديات** | TSK-RSK-006 |
| **المخرجات** | Signal Boundaries + Job + Isolation + Performance |
| **معيار القبول** | [`testing-plan.md`](./testing-plan.md) |

---

## 2. الترتيب

```text
TSK-RSK-001 Migration + Entity
      ↓
TSK-RSK-002 Scoring Engine
      ↓
TSK-RSK-003 Recommendation + Draft
      ↓
TSK-RSK-004 Nightly Job
      ↓
TSK-RSK-005 Query Service
      ↓
TSK-RSK-006 API
      ↓
TSK-RSK-007 Tests
```

---

## 3. معايير القبول النهائية

- [ ] `risk_scores` + RLS + `branch_id`.
- [ ] 6 إشارات بأوزان صحيحة.
- [ ] Score 0..100.
- [ ] Level Boundaries صحيحة.
- [ ] `INSUFFICIENT_DATA` لـ < 14 يوم.
- [ ] Reasons عربية مفهومة.
- [ ] Draft لـ HIGH/CRITICAL فقط.
- [ ] Job 3:00 AM + ShedLock.
- [ ] Job Idempotent.
- [ ] At-Risk + Summary + Recompute.
- [ ] Branch Scope + Trainer Scope.
- [ ] Tests خضراء.

---

## 4. المخاطر

| المخاطرة | التخفيف |
|---|---|
| Job بطيء على 500 عضو | N+1 محسوب + Index + قياس |
| Score مش مفهوم لصاحب الجيم | Reasons عربية + Action |
| Payment Proxy مش دقيق | موثق + P1 يحسن |
| أعضاء جدد Score مضلل | `INSUFFICIENT_DATA` |
| Job يشتغل على Org غلط | `TenantTransactionRunner` |
| إعادة التشغيل تكرر البيانات | Upsert |
| Manager يشوف فروع تانية | `branch_id` + Branch Scope |
| Trainer يشوف كل الأعضاء | `assigned_trainer_id` Check |

---

## 5. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`api-design.md`](./api-design.md)
- [`testing-plan.md`](./testing-plan.md)
