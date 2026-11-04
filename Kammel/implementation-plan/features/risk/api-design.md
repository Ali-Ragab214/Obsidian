# Risk — تصميم الـ API

> **Base Path:** `/api/v1/risk`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/risk/at-risk` | Owner, Manager | قائمة الأعضاء المعرضين للخطر |
| GET | `/risk/members/{memberId}` | Owner, Manager, Trainer (عميله) | آخر Score + Reasons |
| GET | `/risk/summary` | Owner, Manager | توزيع المستويات |
| POST | `/risk/recompute` | Owner | إعادة حساب عضو |

---

## 2. `GET /risk/at-risk`

**Query:** `level` (`HIGH`, `CRITICAL` — Default الاثنين), `branchId` (Owner فقط), `minScore`, `search`, `page`, `size`, `sort`.

**Response `200 OK`:** `PageResponse<AtRiskMemberResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
        "fullName": "Karim Adel",
        "phone": "01234567890",
        "memberCode": "M-000456",
        "branchId": "99999999-8888-7777-6666-555555555555",
        "score": 85,
        "riskLevel": "CRITICAL",
        "reasons": [
          "غياب 14 يوم متواصل",
          "انخفاض الحضور بنسبة 65% خلال آخر 14 يوم",
          "الاشتراك ينتهي خلال 4 يوم (2026-11-07)"
        ],
        "recommendedAction": "تواصل النهارده — عرض تجديد أو جلسة مع الكابتن.",
        "draftMessage": "يا كريم، وحشتنا في جولدز جيم! 🏋️‍♂️...",
        "subscriptionValueMinor": 70000,
        "subscriptionValueFormatted": "700.00 EGP",
        "computedAt": "2026-11-03T01:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1,
    "first": true,
    "last": true
  }
}
```

**Rules:**
- `level` Default = `HIGH,CRITICAL`.
- `INSUFFICIENT_DATA` مستبعد افتراضياً — يظهر لو `level=INSUFFICIENT_DATA`.
- Manager → فرعه فقط.
- الترتيب الافتراضي: `score DESC`.

**Errors:** `403` للـ Receptionist/Trainer.

---

## 3. `GET /risk/members/{memberId}`

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "memberId": "a0b1c2d3-...",
    "fullName": "Karim Adel",
    "memberCode": "M-000456",
    "score": 85,
    "riskLevel": "CRITICAL",
    "reasons": [
      "غياب 14 يوم متواصل",
      "انخفاض الحضور بنسبة 65% خلال آخر 14 يوم",
      "الاشتراك ينتهي خلال 4 يوم (2026-11-07)"
    ],
    "signals": {
      "daysSinceLastVisit": 14,
      "visitsLast14": 0,
      "visitsPrior14": 4,
      "attendanceDropPct": 100,
      "daysRemaining": 4,
      "delayedActivations": 1,
      "freezeCount": 2,
      "tenureDays": 45,
      "points": {
        "absence": 30,
        "attendanceDrop": 25,
        "expiry": 20,
        "paymentFriction": 10,
        "freeze": 10,
        "tenure": 5
      }
    },
    "recommendedAction": "تواصل النهارده — عرض تجديد أو جلسة مع الكابتن.",
    "draftMessage": "يا كريم، وحشتنا في جولدز جيم! 🏋️‍♂️...",
    "modelVersion": "rules-v1",
    "computedAt": "2026-11-03T01:00:00Z"
  }
}
```

**Rules:**
- Trainer → بس لو `member.assigned_trainer_id = currentUserId`، وإلا `403`.
- Manager → فرعه فقط.
- لو مفيش Score → `404 RISK_SCORE_NOT_FOUND`.

---

## 4. `GET /risk/summary`

**Query:** `branchId` (Owner فقط).

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "lowCount": 120,
    "mediumCount": 45,
    "highCount": 14,
    "criticalCount": 8,
    "insufficientDataCount": 23,
    "totalAtRiskCount": 22,
    "totalAtRiskValueMinor": 1540000,
    "totalAtRiskValueFormatted": "15,400.00 EGP",
    "computedAt": "2026-11-03T01:00:00Z"
  }
}
```

**Rules:**
- `totalAtRiskCount` = `high + critical`.
- `totalAtRiskValue` = مجموع `price_minor_snapshot` للاشتراكات الحالية.
- Manager → فرعه فقط.
- لو الـ Job لسه ما اشتغلش → كل القيم `0` + `computedAt: null`.

---

## 5. `POST /risk/recompute`

**Request:**
```json
{
  "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d"
}
```

**Validation:**
- `memberId`: `@NotNull`.
- العضو تابع للـ Org.
- العضو عنده اشتراك `ACTIVE` / `GRACE`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "Risk score recomputed",
  "data": {
    "memberId": "a0b1c2d3-...",
    "score": 85,
    "riskLevel": "CRITICAL",
    "reasons": ["..."],
    "recommendedAction": "...",
    "draftMessage": "...",
    "modelVersion": "rules-v1",
    "computedAt": "2026-11-03T09:30:00Z"
  }
}
```

**Rules:**
- Owner فقط.
- Synchronous — بياخد < 500ms لعضو واحد.
- ممنوع Recompute لكل الـ Org من الـ API — الـ Job هو المسؤول.
- الـ `computed_at` في الـ Recompute اليدوي = `now` (مش 03:00) — عشان يبقى أحدث Score.

**Errors:** `404 MEMBER_NOT_FOUND`, `422 NO_ACTIVE_SUBSCRIPTION`, `403`.

---

## 6. الـ DTOs

```java
public record AtRiskMemberResponse(
    UUID memberId,
    String fullName,
    String phone,
    String memberCode,
    UUID branchId,
    Integer score,
    RiskLevel riskLevel,
    List<String> reasons,
    String recommendedAction,
    String draftMessage,
    long subscriptionValueMinor,
    String subscriptionValueFormatted,
    Instant computedAt
) {}

public record RiskScoreResponse(
    UUID memberId,
    String fullName,
    String memberCode,
    Integer score,
    RiskLevel riskLevel,
    List<String> reasons,
    JsonNode signals,
    String recommendedAction,
    String draftMessage,
    String modelVersion,
    Instant computedAt
) {}

public record RiskSummaryResponse(
    long lowCount,
    long mediumCount,
    long highCount,
    long criticalCount,
    long insufficientDataCount,
    long totalAtRiskCount,
    long totalAtRiskValueMinor,
    String totalAtRiskValueFormatted,
    Instant computedAt
) {}

public record RecomputeRiskRequest(
    @NotNull UUID memberId
) {}
```

---

## 7. الصلاحيات

```java
@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
PageResponse<AtRiskMemberResponse> atRisk(AtRiskFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','TRAINER')")
RiskScoreResponse memberScore(UUID memberId);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
RiskSummaryResponse summary(UUID branchId);

@PreAuthorize("hasRole('GYM_OWNER')")
RiskScoreResponse recompute(RecomputeRiskRequest request);
```

- Trainer Scope جوه الـ Service: `assignedTrainerId == currentUserId`.
- Manager Branch Scope جوه الـ Service.
- Receptionist → `403` في كل الـ Endpoints.

---

## 8. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
