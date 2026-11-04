# Member — تصميم الـ Domain

> **الـ Package:** `com.kammel.features.member.domain`  
> **المرتبط:** [`README.md`](./README.md) · [entity-relationships](../../architecture/entity-relationships.md) · [database-strategy](../../architecture/database-strategy.md)

---

## 1. `Member`

| الحقل | النوع | القيود | ملاحظات |
|---|---|---|---|
| `id` | `UUID` | PK | — |
| `orgId` | `UUID` | `NOT NULL` | Tenant |
| `branchId` | `UUID` | `NOT NULL` | الفرع الأساسي |
| `memberCode` | `String` | `NOT NULL`, Unique `(org_id, code)` | `M-000123` |
| `qrToken` | `UUID` | `NOT NULL`, Unique | QR ثابت |
| `fullName` | `String` | `NOT NULL`, 2–100 | — |
| `phone` | `String` | `NOT NULL` | Unique بين غير المحذوفين |
| `gender` | `Gender` | `NULL` | `MALE` / `FEMALE` |
| `birthDate` | `LocalDate` | `NULL` | — |
| `photoUrl` | `String` | `NULL`, ≤ 500 | رابط خارجي فقط |
| `source` | `MemberSource` | `NULL` | `INSTAGRAM`, `FRIEND`, `WALK_IN`, `AD`, `REFERRAL`, `OTHER` |
| `notes` | `String` | `NULL`, ≤ 1000 | — |
| `whatsappOptIn` | `boolean` | `NOT NULL` | `false` |
| `whatsappOptInAt` | `Instant` | `NULL` | وقت الموافقة |
| `assignedTrainerId` | `UUID` | `NULL` | FK → users |
| `status` | `MemberStatus` | `NOT NULL` | `ACTIVE` |
| `createdAt` / `updatedAt` | `Instant` | `NOT NULL` | — |
| `deletedAt` | `Instant` | `NULL` | Soft Delete |

```java
@Entity
@Table(name = "members")
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member extends BaseEntity implements TenantScoped {

    @Column(name = "org_id", nullable = false)
    private UUID orgId;

    @Column(name = "branch_id", nullable = false)
    private UUID branchId;

    @Column(name = "member_code", nullable = false, length = 20)
    private String memberCode;

    @Column(name = "qr_token", nullable = false, unique = true)
    private UUID qrToken;

    @Column(name = "full_name", nullable = false, length = 100)
    private String fullName;

    @Column(nullable = false, length = 20)
    private String phone;

    @Enumerated(EnumType.STRING)
    @Column(length = 10)
    private Gender gender;

    @Column(name = "birth_date")
    private LocalDate birthDate;

    @Column(name = "photo_url", length = 500)
    private String photoUrl;

    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private MemberSource source;

    @Column(length = 1000)
    private String notes;

    @Column(name = "whatsapp_opt_in", nullable = false)
    private boolean whatsappOptIn = false;

    @Column(name = "whatsapp_opt_in_at")
    private Instant whatsappOptInAt;

    @Column(name = "assigned_trainer_id")
    private UUID assignedTrainerId;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private MemberStatus status = MemberStatus.ACTIVE;

    @Column(name = "deleted_at")
    private Instant deletedAt;

    public void softDelete(Instant now) {
        this.deletedAt = now;
        this.status = MemberStatus.INACTIVE;
    }

    public void activate()   { this.status = MemberStatus.ACTIVE; }
    public void deactivate() { this.status = MemberStatus.INACTIVE; }

    public boolean isDeleted() { return deletedAt != null; }
}
```

---

## 2. الـ Enums

```java
public enum MemberStatus { ACTIVE, INACTIVE }

public enum Gender { MALE, FEMALE }

public enum MemberSource {
    INSTAGRAM, FRIEND, WALK_IN, AD, REFERRAL, OTHER
}
```

---

## 3. توليد `member_code`

**القاعدة:** تسلسل لكل Org، بدون Gaps مسموحة بعد Soft Delete.

```sql
-- داخل Transaction
SELECT pg_advisory_xact_lock(hashtext(:orgId));

SELECT COALESCE(
    MAX(CAST(SUBSTRING(member_code FROM 3) AS BIGINT)), 0
) + 1
FROM members
WHERE org_id = :orgId;
```

- `pg_advisory_xact_lock` بيمنع التزامن على نفس الـ Org.
- النتيجة تتنسق `M-%06d` → `M-000001`.
- لو الرقم > 999999 → `M-1000000` (6 أرقام مش حد أقصى).
- **ممنوع** `COUNT(*) + 1` — بيسبب تكرار بعد الحذف.
- **ممنوع** Sequence عالمية — بتكسر Per-Org.

---

## 4. `qr_token`

- `UUID.randomUUID()` عند الإنشاء.
- فريد عالمياً.
- ثابت — مش بيتغير.
- الـ QR Card بيحتوي `qr_token` فقط.
- الـ Check-in بيدور عليه بـ `qr_token`.

---

## 5. حالة العضو (`status`)

**`status` مش Source of Truth** — هي Cached Projection.

| الحدث | الحالة |
|---|---|
| إنشاء عضو | `ACTIVE` |
| `MemberAccessChangedEvent(hasActiveMembership = true)` | `ACTIVE` |
| `MemberAccessChangedEvent(hasActiveMembership = false)` | `INACTIVE` |
| Soft Delete | `INACTIVE` |
| Import مع اشتراك ساري | `ACTIVE` |

- `hasActiveMembership` معناها: فيه اشتراك بحالة `ACTIVE` / `FROZEN` / `GRACE` — مش `EXPIRED` / `CANCELLED`.
- الـ Event بييجي من `subscription` — لأنها المالكة لكل اشتراكات العضو.
- ممنوع `member` تسأل `SubscriptionQueryService` مباشرة — عشان مفيش Cycle ([feature-dependencies](../../architecture/feature-dependencies.md#7-الاعتماديات-الممنوعة-forbidden-dependencies)).

> **ملاحظة:** `FROZEN` يعني عضو نشط لكن مش قادر يدخل. الفصل بين `member.status` و`check-in access` مقصود.

---

## 6. قواعد التسجيل

### 6.1 Validation
- `fullName`: 2–100، Trim.
- `phone`: `@ValidPhone` + تطبيع لـ `01xxxxxxxxx`.
- `branchId`: موجود + تابع للـ Org + نشط.
- `email` مش موجود في Member (مفيش Login).
- `whatsappOptIn`: إلزامي (true/false) — لو true → `whatsappOptInAt = now()`.
- `assignedTrainerId`: لو موجود → User بدور `TRAINER` في نفس الـ Org والفرع.

### 6.2 التكرار
- `(org_id, phone)` بين غير المحذوفين.
- لو موجود → `409 DUPLICATE_RESOURCE` مع `memberId` الموجود.
- لو العضو محذوف بنفس الموبايل → مسموح تسجيل جديد.

### 6.3 الإنشاء
```text
1. Validate Branch + Trainer.
2. خد advisory lock → member_code.
3. qr_token = UUID.randomUUID().
4. status = ACTIVE.
5. Save.
6. Audit MEMBER_CREATED.
```

---

## 7. تعديل العضو

- كل الحقول قابلة للتعديل ما عدا:
  - `memberCode` — ثابت.
  - `qrToken` — ثابت.
  - `orgId` — من الـ Context.
  - `createdAt` — ثابت.
- تغيير الموبايل → نفس فحص التكرار.
- تغيير `whatsappOptIn` من false → true → `whatsappOptInAt = now()`.
- من false → false → `whatsappOptInAt` يفضل.
- Audit `MEMBER_UPDATED` مع قبل/بعد.

---

## 8. Soft Delete

```text
1. تحقق إن العضو تابع للـ Org.
2. deletedAt = now().
3. status = INACTIVE.
4. Audit MEMBER_DELETED.
5. ممنوع حذف بياناته المالية أو الاشتراكات.
```

- العضو المحذوف مش بيظهر في البحث أو الـ Check-in.
- `phone` بيبقى متاح للتسجيل من جديد.
- `member_code` و`qr_token` بيفضلوا محجوزين (Unique على كل الصفوف).

---

## 9. استيراد Excel

### 9.1 الأعمدة

| العمود | إلزامي | الصيغة | ملاحظات |
|---|:---:|---|---|
| `full_name` | ✅ | نص | — |
| `phone` | ✅ | `01xxxxxxxxx` | — |
| `gender` | ❌ | `MALE`/`FEMALE` | — |
| `birth_date` | ❌ | `yyyy-MM-dd` أو `dd/MM/yyyy` | — |
| `branch` | ❌ | اسم الفرع | Default: فرع المستخدم |
| `source` | ❌ | Enum | — |
| `notes` | ❌ | نص | ≤ 1000 |
| `whatsapp_opt_in` | ✅ | `TRUE`/`FALSE` | — |
| `plan_name` | ❌ | اسم باقة موجودة | لاستيراد الاشتراك الحالي |
| `start_date` | ❌ | تاريخ | مطلوب لو `plan_name` |
| `end_date` | ❌ | تاريخ | مطلوب لو `plan_name` |

### 9.2 المرحلتين

```text
POST /members/import?dryRun=true
  → Parse + Validate + Report (بدون كتابة)

POST /members/import?dryRun=false
  → نفس الـ Validation
  → إدخال الصفوف السليمة فقط
  → تقرير بالصفوف الفاشلة
```

### 9.3 Validation Rules
- `full_name` مش فاضي.
- `phone` صالح.
- تكرار `phone` داخل الملف → خطأ للصفوف المكررة (الأول ينجح).
- تكرار `phone` في الـ DB → خطأ.
- `branch` موجود ونشط.
- `plan_name` موجود ونشط.
- `end_date >= start_date`.
- `whatsapp_opt_in` Boolean واضح.

### 9.4 Import Result

```json
{
  "totalRows": 350,
  "successCount": 348,
  "failureCount": 2,
  "dryRun": true,
  "errors": [
    { "rowNumber": 45, "phone": "01000", "error": "INVALID_PHONE" },
    { "rowNumber": 112, "phone": "01122334455", "error": "DUPLICATE_IN_DB" }
  ]
}
```

### 9.5 استيراد الاشتراك
- لو `plan_name` موجود → إنشاء `Subscription` بحالة `ACTIVE`.
- `price_minor_snapshot = plan.price_minor`.
- `start_date` / `end_date` من الملف.
- `sessions_remaining = plan.session_count` لو Session-based.
- **مفيش Payment Record** — دي بيانات تشغيلية مش مالية.
- `SubscriptionEvent(type = CREATED, metadata.imported = true)`.
- Audit `MEMBER_IMPORTED` مع عدد الصفوف.

---

## 10. البحث

| النوع | الاستعلام |
|---|---|
| Phone | `findByOrgIdAndPhoneAndDeletedAtIsNull` |
| Code | `findByOrgIdAndMemberCode` |
| Name | `findByOrgIdAndFullNameContainingIgnoreCaseAndDeletedAtIsNull` |
| QR | `findByQrToken` |
| Trainer | `findAllByOrgIdAndAssignedTrainerIdAndDeletedAtIsNull` |

- البحث بالاسم بـ `LIKE` — مقبول لـ 500 عضو.
- لو البيانات كبرت → `pg_trgm` + GIN Index في P1.

---

## 11. الـ Invariants

| # | الـ Invariant | مكان التحقق |
|:---:|---|---|
| INV-MEM-01 | `phone` فريد بين غير المحذوفين | Partial Unique |
| INV-MEM-02 | `memberCode` فريد لكل Org | Unique |
| INV-MEM-03 | `qrToken` فريد | Unique |
| INV-MEM-04 | `branchId` تابع للـ Org | Service |
| INV-MEM-05 | `assignedTrainerId` بدور TRAINER | Service |
| INV-MEM-06 | `deletedAt != null` → مش في البحث | Query |
| INV-MEM-07 | Soft Delete لا يحذف مالياً | Service |
| INV-MEM-08 | `status` من الأحداث | Listener |
| INV-MEM-09 | `whatsappOptInAt` عند الموافقة | Service |
| INV-MEM-10 | Import ≤ 5000 صف | Controller |

---

## 12. الأخطاء

| Exception | HTTP | متى |
|---|:---:|---|
| `DuplicateMemberException` | 409 | Phone مكرر |
| `MemberNotFoundException` | 404 | مش موجود / Org تانية |
| `InvalidTrainerException` | 422 | User مش Trainer |
| `ImportValidationException` | 422 | Excel فيه أخطاء |
| `ImportTooLargeException` | 413 | > 5000 صف / 10MB |
| `MemberDeletedException` | 404 | عضو محذوف |

---

## 13. مرتبط بـ

- [`implementation-plan.md`](./implementation-plan.md)
- [`api-design.md`](./api-design.md)
- [database-strategy](../../architecture/database-strategy.md)
