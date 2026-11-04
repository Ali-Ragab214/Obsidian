# 07 - توثيق واجهات برمجة التطبيقات (REST API Design & Contracts)

> **مسار الملف:** `implementation-blueprint/07-api-design.md`  
> **الحالة:** معتمد للتنفيذ (Approved for Implementation)  
> **المرجع:** [01-target-project-structure.md](./01-target-project-structure.md) · [02-features-breakdown.md](./02-features-breakdown.md) · [04-shared-components.md](./04-shared-components.md)

---

## 1. المعايير العامة للـ REST APIs

- **مسار الأساس (Base URL):** `/api/v1`
- **تنسيق البيانات (Payload Format):** JSON بنمط `camelCase`.
- **التواريخ والأوقات (Timestamps):** معيار ISO-8601 UTC (مثال: `2026-11-03T10:15:30Z`).
- **المبالغ المالية (Currency):** حقل حسابي بالقروش `amountMinor` كـ `Long` وحقل منسق للعرض `amountFormatted` (مثال: `"700.00 EGP"`).
- **الاستجابات:** مغلفة دائماً في غلاف `ApiResponse<T>` أو `PageResponse<T>`.
- **أكواد الحالة القياسية:**
  - `200 OK`: نجاح العملية أو جلب البيانات.
  - `201 Created`: نجاح إنشاء مورد جديد في النظام.
  - `400 Bad Request`: خطأ في صيغة أو مدخلات الطلب (Validation Error).
  - `401 Unauthorized`: غياب أو انتهاء صلاحية توكن الـ JWT.
  - `403 Forbidden`: محاولة الوصول لمورد غير مصرح به للرتبة الحالية.
  - `404 Not Found`: المورد غير موجود (أو يتبع مستأجر آخر لمنع كشف الوجود).
  - `409 Conflict`: تعارض (مثل تكرار رقم الموبايل أو تعارض مفتاح الـ Idempotency).
  - `422 Unprocessable Entity`: مخالفة لقواعد الأعمال وآلة الحالة (Invalid State Transition).
  - `500 Internal Server Error`: خطأ خادم غير متوقع.

---

## 2. ميزة الهوية والمصادقة (`features/identity`)

### 2.1 تسجيل الدخول واستخراج التوكن
- **Endpoint:** `POST /api/v1/auth/login`
- **الصلاحية (Auth):** عام (Public)
- **Request (`LoginRequest`):**
```json
{
  "username": "01012345678",
  "password": "SecurePassword123!"
}
```
- **Response `200 OK` (`AuthResponse`):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsIn...",
    "refreshToken": "4d7a8e9f-01b2-3c4d-5e6f-7a8b9c0d1e2f",
    "expiresInSeconds": 900,
    "user": {
      "id": "e2f3a4b5-c6d7-8e9f-0a1b-2c3d4e5f6a7b",
      "name": "Ali Hassan",
      "role": "RECEPTIONIST",
      "orgId": "11111111-2222-3333-4444-555555555555",
      "branchId": "99999999-8888-7777-6666-555555555555"
    }
  },
  "timestamp": "2026-11-03T10:00:00Z"
}
```

### 2.2 تجديد التوكن (Refresh Token Rotation)
- **Endpoint:** `POST /api/v1/auth/refresh`
- **الصلاحية:** عام (Public)
- **Request:** `{"refreshToken": "4d7a8e9f-01b2-3c4d-5e6f-7a8b9c0d1e2f"}`
- **Response `200 OK`:** نفس هيكل `AuthResponse` مع إصدار Access Token جديد وتدوير الـ Refresh Token.

---

## 3. ميزة إدارة الأعضاء (`features/member`)

### 3.1 تسجيل عضو جديد
- **Endpoint:** `POST /api/v1/members`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')")`
- **Request (`CreateMemberRequest`):**
```json
{
  "fullName": "Ahmed Tarek",
  "phone": "01012345678",
  "gender": "MALE",
  "birthDate": "1995-04-15",
  "branchId": "99999999-8888-7777-6666-555555555555",
  "source": "INSTAGRAM",
  "notes": "Interested in strength training",
  "whatsappOptIn": true
}
```
- **Validation Rules:**
  - `fullName`: `@NotBlank`, `@Size(min = 2, max = 100)`
  - `phone`: `@NotBlank`, `@ValidPhone` (يجب أن يبدأ بـ 010 أو 011 أو 012 أو 015 ويكون 11 رقماً)
  - `branchId`: `@NotNull`
  - `whatsappOptIn`: `@NotNull`
- **Response `201 Created` (`MemberResponse`):**
```json
{
  "success": true,
  "message": "Member registered successfully",
  "data": {
    "id": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "fullName": "Ahmed Tarek",
    "phone": "01012345678",
    "gender": "MALE",
    "birthDate": "1995-04-15",
    "branchId": "99999999-8888-7777-6666-555555555555",
    "source": "INSTAGRAM",
    "status": "ACTIVE",
    "whatsappOptIn": true,
    "hasActiveSubscription": false,
    "createdAt": "2026-11-03T10:15:00Z"
  },
  "timestamp": "2026-11-03T10:15:00Z"
}
```

### 3.2 البحث والتصفح في قائمة الأعضاء
- **Endpoint:** `GET /api/v1/members`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST', 'TRAINER')")`
- **Query Params:**
  - `search` (اختياري: بحث جزئي في الاسم أو رقم الموبايل)
  - `status` (اختياري: `ACTIVE`, `INACTIVE`)
  - `branchId` (اختياري)
  - `page` (افتراضي: 0), `size` (افتراضي: 20, حد أقصى 100), `sort` (افتراضي: `createdAt,desc`)
- **Response `200 OK`:** غلاف `PageResponse<MemberSummaryResponse>`.

### 3.3 استيراد شيت إكسيل جماعي للأعضاء
- **Endpoint:** `POST /api/v1/members/import`
- **الصلاحية:** `@PreAuthorize("hasRole('GYM_OWNER')")`
- **Content-Type:** `multipart/form-data` مع بارامتر `file`
- **Response `200 OK` (`ImportResultResponse`):**
```json
{
  "success": true,
  "message": "Excel import processed",
  "data": {
    "totalRows": 250,
    "successCount": 248,
    "failureCount": 2,
    "errors": [
      { "rowNumber": 14, "phone": "01099", "error": "Invalid phone format" },
      { "rowNumber": 89, "phone": "01122334455", "error": "Duplicate phone number in file" }
    ]
  },
  "timestamp": "2026-11-03T10:20:00Z"
}
```

---

## 4. ميزة باقات الاشتراك (`features/plan`)

### 4.1 إضافة باقة اشتراك جديدة
- **Endpoint:** `POST /api/v1/plans`
- **الصلاحية:** `@PreAuthorize("hasRole('GYM_OWNER')")`
- **Request (`CreatePlanRequest`):**
```json
{
  "name": "Gold 3-Month Plan",
  "type": "QUARTERLY",
  "durationDays": 90,
  "sessionCount": null,
  "priceMinor": 180000,
  "freezeMaxDays": 14,
  "graceDays": 5,
  "timeRestriction": null
}
```
- **Response `201 Created` (`PlanResponse`).**

---

## 5. ميزة عقود الاشتراكات (`features/subscription`)

### 5.1 إنشاء عقد اشتراك جديد
- **Endpoint:** `POST /api/v1/subscriptions`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')")`
- **Request (`CreateSubscriptionRequest`):**
```json
{
  "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
  "planId": "33333333-4444-5555-6666-777777777777",
  "startDate": "2026-11-05"
}
```
- **Response `201 Created` (`SubscriptionResponse`):**
```json
{
  "success": true,
  "message": "Subscription created in pending payment state",
  "data": {
    "id": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
    "planName": "Gold 3-Month Plan",
    "priceMinorSnapshot": 180000,
    "priceFormatted": "1,800.00 EGP",
    "startDate": "2026-11-05",
    "endDate": "2027-02-03",
    "status": "PENDING_PAYMENT",
    "sessionsRemaining": null,
    "frozenDaysUsed": 0
  },
  "timestamp": "2026-11-03T10:30:00Z"
}
```

### 5.2 تجديد اشتراك (Renew)
- **Endpoint:** `POST /api/v1/subscriptions/{id}/renew`
- **Request (`RenewSubscriptionRequest`):** `{"planId": "33333333-4444-5555-6666-777777777777"}`
- **Response `200 OK`:** عقد جديد ممدد التواريخ بحالة `PENDING_PAYMENT` لحين السداد.

### 5.3 تجميد اشتراك (Freeze)
- **Endpoint:** `POST /api/v1/subscriptions/{id}/freeze`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')")`
- **Request (`FreezeSubscriptionRequest`):**
```json
{
  "days": 7,
  "reason": "Traveling for work"
}
```
- **Response `200 OK`:** تحديث حالة العقد إلى `FROZEN` وتوثيق السجل في `SubscriptionEvent`.

---

## 6. ميزة المدفوعات والتحصيل (`features/payment`)

### 6.1 تسجيل سند سداد (مع منع التكرار)
- **Endpoint:** `POST /api/v1/payments`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')")`
- **Header:** `Idempotency-Key: c9d8e7f6-5a4b-3c2d-1e0f-9a8b7c6d5e4f` (إلزامي)
- **Request (`RecordPaymentRequest`):**
```json
{
  "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
  "amountMinor": 180000,
  "method": "INSTAPAY",
  "reference": "TXN-987654321",
  "notes": "Full payment received via InstaPay"
}
```
- **Response `201 Created` (`PaymentResponse`):**
```json
{
  "success": true,
  "message": "Payment recorded and subscription activated",
  "data": {
    "id": "55555555-6666-7777-8888-999999999999",
    "subscriptionId": "77777777-8888-9999-aaaa-bbbbbbbbbbbb",
    "amountMinor": 180000,
    "amountFormatted": "1,800.00 EGP",
    "method": "INSTAPAY",
    "reference": "TXN-987654321",
    "receiptNumber": "KM-2026-00421",
    "status": "COMPLETED",
    "subscriptionStatus": "ACTIVE",
    "createdAt": "2026-11-03T10:45:00Z"
  },
  "timestamp": "2026-11-03T10:45:00Z"
}
```

### 6.2 استرجاع مبلغ دفعة (Refund)
- **Endpoint:** `POST /api/v1/payments/{id}/refund`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')")`
- **Request (`RefundRequest`):**
```json
{
  "amountMinor": 180000,
  "reason": "Member relocated to another city"
}
```
- **Response `200 OK`:** تسجيل سند الاسترداد العكسي وتحويل حالة السند الأصلي إلى `REVERSED`.

---

## 7. ميزة تسجيل الحضور السريع (`features/checkin`)

### 7.1 فحص وتسجيل الحضور
- **Endpoint:** `POST /api/v1/checkins`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER', 'RECEPTIONIST')")`
- **Request (`CheckInRequest`):**
```json
{
  "identifier": "01012345678",
  "method": "PHONE_SEARCH"
}
```
- **Response `200 OK` (حالة النجاح):**
```json
{
  "success": true,
  "message": "Check-in successful",
  "data": {
    "checkInId": "44444444-5555-6666-7777-888888888888",
    "status": "SUCCESS",
    "memberName": "Ahmed Tarek",
    "planName": "Gold 3-Month Plan",
    "daysRemaining": 4,
    "alert": "Subscription expires in 4 days. Remind member to renew!"
  },
  "timestamp": "2026-11-03T11:00:00Z"
}
```
- **Response `422 Unprocessable Entity` (في حال التجميد):**
```json
{
  "timestamp": "2026-11-03T11:00:00Z",
  "status": 422,
  "error": "INVALID_STATE_TRANSITION",
  "message": "Subscription is frozen until 2026-11-10. Reason: Traveling for work",
  "path": "/api/v1/checkins"
}
```

---

## 8. ميزة طابور الواتساب (`features/notification`)

### 8.1 جلب طابور الرسائل اليومي
- **Endpoint:** `GET /api/v1/notifications/queue`
- **Query Params:** `template`, `dueDate`
- **Response `200 OK` (`List<MessageQueueItemResponse>`):**
```json
{
  "success": true,
  "data": [
    {
      "id": "11112222-3333-4444-5555-666677778888",
      "memberName": "Mohamed Samir",
      "memberPhone": "01099887766",
      "template": "RENEWAL_REMINDER",
      "renderedText": "أهلاً يا محمد، اشتراكك في الجيم بيخلص يوم 10 نوفمبر. مجهزينلك عرض تجديد مميز لما تشرفنا الأسبوع ده!",
      "waMeLink": "https://wa.me/201099887766?text=%D8%A3%D9%87%D9%84%D8%A7%D9%8B...",
      "dueDate": "2026-11-03",
      "status": "PENDING"
    }
  ]
}
```

### 8.2 تأكيد إرسال الرسالة عبر الاستقبال
- **Endpoint:** `PATCH /api/v1/notifications/{id}/sent`
- **Response `200 OK`:** تحديث حالة الرسالة إلى `SENT_CONFIRMED`.

---

## 9. ميزة تقييم المخاطر الذكي (`features/risk`)

### 9.1 استعراض الأعضاء المعرضين للتسرب
- **Endpoint:** `GET /api/v1/risk/at-risk`
- **الصلاحية:** `@PreAuthorize("hasAnyRole('GYM_OWNER', 'BRANCH_MANAGER')")`
- **Response `200 OK` (`List<AtRiskMemberResponse>`):**
```json
{
  "success": true,
  "data": [
    {
      "memberId": "a0b1c2d3-e4f5-6a7b-8c9d-0e1f2a3b4c5d",
      "fullName": "Karim Adel",
      "phone": "01234567890",
      "score": 85,
      "riskLevel": "CRITICAL",
      "reasons": [
        "Absent for 14 consecutive days",
        "Attendance dropped by 65% over past 14 days",
        "Subscription expires in 4 days"
      ],
      "recommendedAction": "Contact member today with personalized renewal incentive",
      "subscriptionValueFormatted": "700.00 EGP"
    }
  ]
}
```

---

## 10. ميزة لوحة التحكم التنفيذية (`features/reporting`)

### 10.1 مؤشرات الأداء الرئيسية لصاحب الجيم
- **Endpoint:** `GET /api/v1/dashboard/summary`
- **الصلاحية:** `@PreAuthorize("hasRole('GYM_OWNER')")`
- **Response `200 OK` (`DashboardSummaryResponse`):**
```json
{
  "success": true,
  "data": {
    "renewalsThisWeek": {
      "expiringCount": 18,
      "renewedCount": 7,
      "pendingCount": 11,
      "totalValueFormatted": "12,600.00 EGP"
    },
    "atRiskSummary": {
      "criticalCount": 8,
      "highCount": 14,
      "totalAtRiskValueFormatted": "15,400.00 EGP"
    },
    "revenueMonthToDateFormatted": "84,000.00 EGP",
    "renewalRatePercentage": 64.5,
    "monthlyChurnPercentage": 7.2,
    "attributedRevenueFormatted": "9,800.00 EGP"
  }
}
```
