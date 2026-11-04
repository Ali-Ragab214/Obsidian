# Reporting — تصميم الـ API

> **Base Path:** `/api/v1/dashboard`  
> **المرتبط:** [`README.md`](./README.md) · [api-design العام](../../api-design.md)

---

## 1. الـ Endpoints

| Method | Path | Roles | الوصف |
|---|---|---|---|
| GET | `/dashboard/summary` | Owner | KPIs تنفيذية |
| GET | `/dashboard/renewals` | Owner, Manager | تجديدات الفترة |
| GET | `/dashboard/collections` | Owner, Manager | المتأخرات |
| GET | `/dashboard/attendance` | Owner, Manager | الحضور |
| GET | `/dashboard/risk` | Owner, Manager | توزيع المخاطر |
| GET | `/dashboard/members/{id}/timeline` | Owner, Manager, Receptionist | Timeline العضو |

---

## 2. `GET /dashboard/summary`

**Query:** `branchId` (Owner فقط).

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "renewalsThisWeek": {
      "expiringCount": 18,
      "renewedCount": 7,
      "pendingCount": 11,
      "totalValueMinor": 1260000,
      "totalValueFormatted": "12,600.00 EGP"
    },
    "atRiskSummary": {
      "criticalCount": 8,
      "highCount": 14,
      "totalAtRiskValueMinor": 1540000,
      "totalAtRiskValueFormatted": "15,400.00 EGP"
    },
    "revenueMonthToDateMinor": 8400000,
    "revenueMonthToDateFormatted": "84,000.00 EGP",
    "renewalRatePercentage": 38.9,
    "monthlyChurnPercentage": 7.2,
    "generatedAt": "2026-11-03T10:00:00Z"
  }
}
```

**Rules:**
- Owner فقط — Manager → `403`.
- `weekStart` = الاثنين Cairo.
- `renewalRate` = `renewed / expiring * 100` — `0` لو مفيش.
- `churn` = Cohort بداية الشهر.
- Empty → أصفار.

---

## 3. `GET /dashboard/renewals`

**Query:** `from`, `to`, `branchId` (Owner فقط), `status`, `renewed`, `page`, `size`, `sort`.

**Response `200 OK`:** `PageResponse<RenewalItemResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "subscriptionId": "77777777-...",
        "memberId": "a0b1c2d3-...",
        "memberName": "Ahmed Tarek",
        "memberPhone": "01012345678",
        "planName": "Gold 3-Month",
        "startDate": "2026-09-05",
        "endDate": "2026-12-03",
        "priceMinorSnapshot": 180000,
        "priceFormatted": "1,800.00 EGP",
        "status": "ACTIVE",
        "daysRemaining": 30,
        "renewed": false,
        "renewalSubscriptionId": null
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 18,
    "totalPages": 1,
    "first": true,
    "last": true
  }
}
```

**Rules:**
- Default `from` / `to` = الأسبوع الحالي.
- المدى الأقصى 366 يوم.
- `renewed` = فيه `renewal_of` يشاور على الاشتراك.
- Manager → فرعه فقط.

---

## 4. `GET /dashboard/collections`

**Query:** `branchId` (Owner فقط), `agingBucket`, `search`, `page`, `size`, `sort`.

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "totalOutstandingMinor": 2700000,
    "totalOutstandingFormatted": "27,000.00 EGP",
    "count": 14,
    "buckets": [
      { "label": "0-7",  "count": 5, "outstandingMinor": 900000,  "outstandingFormatted": "9,000.00 EGP" },
      { "label": "8-14", "count": 4, "outstandingMinor": 800000,  "outstandingFormatted": "8,000.00 EGP" },
      { "label": "15-30","count": 3, "outstandingMinor": 600000,  "outstandingFormatted": "6,000.00 EGP" },
      { "label": "31+",  "count": 2, "outstandingMinor": 400000,  "outstandingFormatted": "4,000.00 EGP" }
    ],
    "items": {
      "content": [
        {
          "subscriptionId": "77777777-...",
          "memberId": "a0b1c2d3-...",
          "memberName": "Ahmed Tarek",
          "memberPhone": "01012345678",
          "planName": "Gold 3-Month",
          "priceMinorSnapshot": 180000,
          "paidTotalMinor": 90000,
          "outstandingMinor": 90000,
          "outstandingFormatted": "900.00 EGP",
          "daysOutstanding": 12,
          "agingBucket": "8-14"
        }
      ],
      "page": 0,
      "size": 20,
      "totalElements": 14,
      "totalPages": 1,
      "first": true,
      "last": true
    },
    "generatedAt": "2026-11-03T10:00:00Z"
  }
}
```

**Rules:**
- الـ Buckets محسوبة على كل النتائج، مش الصفحة.
- Manager → فرعه.
- الترتيب الافتراضي: `daysOutstanding DESC`.

---

## 5. `GET /dashboard/attendance`

**Query:** `from` (Default `today-6`), `to` (Default `today`), `branchId` (Owner فقط).

**Response `200 OK`:**
```json
{
  "success": true,
  "message": "OK",
  "data": {
    "from": "2026-10-28",
    "to": "2026-11-03",
    "totalCheckIns": 412,
    "uniqueMembers": 168,
    "averagePerDay": 58.9,
    "peakHour": 19,
    "overrideCount": 6,
    "daily": [
      { "date": "2026-10-28", "count": 55, "uniqueMembers": 48 },
      { "date": "2026-10-29", "count": 61, "uniqueMembers": 52 }
    ],
    "branches": [
      { "branchId": "99999999-...", "branchName": "Maadi", "count": 300 },
      { "branchId": "88888888-...", "branchName": "Nasr City", "count": 112 }
    ],
    "generatedAt": "2026-11-03T10:00:00Z"
  }
}
```

**Rules:**
- المدى الأقصى 90 يوم.
- `peakHour` بتوقيت Cairo (0–23).
- `averagePerDay` = `total / days` — `0` لو مفيش.
- Manager → فرعه.

---

## 6. `GET /dashboard/risk`

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
    "averageScore": 34.6,
    "highCriticalValueMinor": 1540000,
    "highCriticalValueFormatted": "15,400.00 EGP",
    "computedAt": "2026-11-03T01:00:00Z",
    "topRisks": [
      {
        "memberId": "a0b1c2d3-...",
        "memberName": "Karim Adel",
        "score": 85,
        "riskLevel": "CRITICAL",
        "recommendedAction": "تواصل النهارده — عرض تجديد أو جلسة مع الكابتن."
      }
    ]
  }
}
```

**Rules:**
- Latest Score لكل عضو.
- `averageScore` بيستبعد `INSUFFICIENT_DATA`.
- لو الـ Job لسه ما اشتغلش → أصفار + `computedAt: null`.
- Manager → فرعه.

---

## 7. `GET /dashboard/members/{id}/timeline`

**Query:** `type` (`SUBSCRIPTION`, `PAYMENT`, `CHECK_IN`, `MESSAGE`, `RISK`), `from`, `to`, `page`, `size`.

**Response `200 OK`:** `PageResponse<TimelineItemResponse>`.

```json
{
  "success": true,
  "message": "OK",
  "data": {
    "content": [
      {
        "type": "PAYMENT",
        "entityId": "55555555-...",
        "occurredAt": "2026-11-03T11:00:00Z",
        "title": "دفعة جديدة",
        "description": "InstaPay — إيصال KM-2026-000421",
        "amountMinor": 180000,
        "amountFormatted": "1,800.00 EGP",
        "metadata": { "method": "INSTAPAY", "status": "COMPLETED" }
      },
      {
        "type": "CHECK_IN",
        "entityId": "44444444-...",
        "occurredAt": "2026-11-03T18:15:00Z",
        "title": "حضور",
        "description": "فرع المعادي — Phone Search",
        "amountMinor": null,
        "amountFormatted": null,
        "metadata": { "branchId": "99999999-...", "overridden": false }
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 42,
    "totalPages": 3,
    "first": true,
    "last": false
  }
}
```

**Rules:**
- المدى الأقصى 366 يوم.
- `type` اختياري.
- Receptionist → فرعه فقط.
- لو العضو مش موجود/فرع تاني → `404`.
- الترتيب `occurredAt DESC`.

---

## 8. الـ DTOs

```java
public record DashboardSummaryResponse(
    RenewalsSummaryResponse renewalsThisWeek,
    AtRiskSummaryResponse atRiskSummary,
    long revenueMonthToDateMinor,
    String revenueMonthToDateFormatted,
    double renewalRatePercentage,
    double monthlyChurnPercentage,
    Instant generatedAt
) {}

public record RenewalsSummaryResponse(
    long expiringCount,
    long renewedCount,
    long pendingCount,
    long totalValueMinor,
    String totalValueFormatted
) {}

public record AtRiskSummaryResponse(
    long criticalCount,
    long highCount,
    long totalAtRiskValueMinor,
    String totalAtRiskValueFormatted
) {}

public record RenewalItemResponse(
    UUID subscriptionId,
    UUID memberId,
    String memberName,
    String memberPhone,
    String planName,
    LocalDate startDate,
    LocalDate endDate,
    long priceMinorSnapshot,
    String priceFormatted,
    String status,
    Integer daysRemaining,
    boolean renewed,
    UUID renewalSubscriptionId
) {}

public record CollectionsReportResponse(
    long totalOutstandingMinor,
    String totalOutstandingFormatted,
    long count,
    List<AgingBucketResponse> buckets,
    PageResponse<CollectionRowResponse> items,
    Instant generatedAt
) {}

public record AttendanceReportResponse(
    LocalDate from,
    LocalDate to,
    long totalCheckIns,
    long uniqueMembers,
    double averagePerDay,
    int peakHour,
    long overrideCount,
    List<DailyAttendanceResponse> daily,
    List<BranchAttendanceResponse> branches,
    Instant generatedAt
) {}

public record RiskReportResponse(
    long lowCount,
    long mediumCount,
    long highCount,
    long criticalCount,
    long insufficientDataCount,
    Double averageScore,
    long highCriticalValueMinor,
    String highCriticalValueFormatted,
    Instant computedAt,
    List<TopRiskMemberResponse> topRisks
) {}

public record TimelineItemResponse(
    String type,
    UUID entityId,
    Instant occurredAt,
    String title,
    String description,
    Long amountMinor,
    String amountFormatted,
    JsonNode metadata
) {}
```

---

## 9. الصلاحيات

```java
@PreAuthorize("hasRole('GYM_OWNER')")
DashboardSummaryResponse summary(UUID branchId);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
PageResponse<RenewalItemResponse> renewals(RenewalFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
CollectionsReportResponse collections(CollectionFilter filter, Pageable pageable);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
AttendanceReportResponse attendance(LocalDate from, LocalDate to, UUID branchId);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER')")
RiskReportResponse risk(UUID branchId);

@PreAuthorize("hasAnyRole('GYM_OWNER','BRANCH_MANAGER','RECEPTIONIST')")
PageResponse<TimelineItemResponse> timeline(UUID memberId, TimelineFilter filter, Pageable pageable);
```

- Manager Branch Scope جوه الـ Service.
- Trainer → `403`.
- Receptionist → Timeline فقط.

---

## 10. مرتبط بـ

- [`domain-design.md`](./domain-design.md)
- [`testing-plan.md`](./testing-plan.md)
- [api-design العام](../../api-design.md)
