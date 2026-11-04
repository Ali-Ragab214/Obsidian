# 04 - المكونات العامة المشتركة (Shared Components & Infrastructure)

> **مسار الملف:** `implementation-blueprint/04-shared-components.md`  
> **الحالة:** معتمد للتنفيذ (Approved for Implementation)  
> **المرجع:** [01-target-project-structure.md](./01-target-project-structure.md) · [03-entity-relationships.md](./03-entity-relationships.md) · [06-security-and-multi-tenancy.md](../Kammel_Project_Documentation/06_SECURITY_AND_MULTI_TENANCY.md)

---

## 1. فلسفة المكونات المشتركة (Shared Philosophy & Rules)

الحزمة العامة `com.kammel.shared` مخصصة فقط للأدوات العابرة للطبقات (Cross-Cutting Concerns) والبنية التحتية البرمجية التي تخدم جميع الميزات.

### 1.1 القواعد الصارمة لما يدخل وما لا يدخل في `shared`:
- ✅ **مسموح داخل `shared`:**
  - الكيانات الأب المشتركة (`BaseEntity`, `BaseAuditableEntity`).
  - أطر الاستجابة العامة للـ HTTP (`ApiResponse`, `PageResponse`, `ErrorResponse`).
  - معالجة الاستثناءات المركزية (`GlobalExceptionHandler`, `BusinessException`).
  - أدوات أمان وسياق المستأجر (`TenantContext`, `TenantFilter`, `JwtService`).
  - كلاسات الأدوات النقية الخالية من الحالات (Stateless Pure Utilities: `MoneyUtil`, `PhoneUtil`, `DateUtil`).
  - الـ Validators العامة المشتركة (مثل فحص أرقام الهواتف المصرية).
- ❌ **ممنوع منعاً باتاً وضعه داخل `shared`:**
  - أي كيان بيزنس خاص بميزة معينة (مثل `Member`, `Subscription`, `Payment`).
  - أي استثناء مالي أو إجرائي يخص نطاق عمل محدد (مثل `SubscriptionFrozenException`).
  - أي DTO خاص بعملية تسجيل أو تجديد أو دفع.
  - أي منطق أعمال (Business Logic) أو حسابات أرباح أو اشتراكات.

---

## 2. الكيانات الأساسية المشتركة (Shared Domain Foundations)

### 2.1 الكيان الأب المشترك: `BaseEntity.java`
يوفر المفتاح الأساسي العشوائي UUID مع تواريخ الإنشاء والتعديل التلقائية لجميع الكيانات.

```java
package com.kammel.shared.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.time.Instant;
import java.util.Objects;
import java.util.UUID;

@Getter
@Setter
@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id", updatable = false, nullable = false)
    private UUID id;

    @CreationTimestamp
    @Column(name = "created_at", updatable = false, nullable = false)
    private Instant createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at", nullable = false)
    private Instant updatedAt;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof BaseEntity that)) return false;
        return id != null && Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

### 2.2 واجهة المستأجر الإلزامية: `TenantScoped.java`
تفرض على الكيانات التابعة للجيم وجود الـ `org_id` وضمان استخدامه تلقائياً في فلاتر الـ JPA وسياسات الـ RLS.

```java
package com.kammel.shared.domain;

import java.util.UUID;

public interface TenantScoped {
    UUID getOrgId();
    void setOrgId(UUID orgId);
}
```

---

## 3. الهياكل المعيارية للاستجابات (API Standard Envelopes)

### 3.1 غلاف الاستجابة الموحد: `ApiResponse<T>`
يضمن أن كل استجابة JSON تخرج للـ Frontend بشكل موحد وواضح.

```java
package com.kammel.shared.api;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.Builder;

import java.time.Instant;

@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public record ApiResponse<T>(
    boolean success,
    String message,
    T data,
    Instant timestamp
) {
    public static <T> ApiResponse<T> success(T data, String message) {
        return ApiResponse.<T>builder()
                .success(true)
                .message(message)
                .data(data)
                .timestamp(Instant.now())
                .build();
    }

    public static <T> ApiResponse<T> success(T data) {
        return success(data, "Operation completed successfully");
    }
}
```

### 3.2 غلاف التصفح والترقيم: `PageResponse<T>`
يغلف مصفوفة النتائج مع مؤشرات الصفحات بشكل موحد.

```java
package com.kammel.shared.api;

import org.springframework.data.domain.Page;
import java.util.List;

public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean first,
    boolean last
) {
    public static <T> PageResponse<T> from(Page<T> springPage) {
        return new PageResponse<>(
                springPage.getContent(),
                springPage.getNumber(),
                springPage.getSize(),
                springPage.getTotalElements(),
                springPage.getTotalPages(),
                springPage.isFirst(),
                springPage.isLast()
        );
    }
}
```

### 3.3 هيكل تفاصيل الأخطاء: `ErrorResponse.java`

```java
package com.kammel.shared.api;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.Builder;

import java.time.Instant;
import java.util.List;

@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public record ErrorResponse(
    Instant timestamp,
    int status,
    String error,
    String message,
    String path,
    List<ValidationErrorDetail> details
) {
    public record ValidationErrorDetail(
        String field,
        Object rejectedValue,
        String message
    ) {}
}
```

---

## 4. منظومة معالجة الاستثناءات (Global Exception Handling)

يتم ضبط كلاس مركزي `@RestControllerAdvice` لترجمة كل استثناء إلى كود HTTP قياسي ورسالة مفهومة:

| الاستثناء (Exception Class) | كود الـ HTTP | مفتاح الخطأ في الـ JSON |
|---|---|---|
| `MethodArgumentNotValidException` | `400 Bad Request` | `VALIDATION_FAILED` |
| `IllegalArgumentException` | `400 Bad Request` | `INVALID_ARGUMENT` |
| `BadCredentialsException` | `401 Unauthorized` | `AUTHENTICATION_FAILED` |
| `AccessDeniedException` | `403 Forbidden` | `ACCESS_DENIED` |
| `ResourceNotFoundException` | `404 Not Found` | `RESOURCE_NOT_FOUND` |
| `TenantMismatchException` | `404 Not Found` (ليس 403 لمنع كشف الوجود) | `RESOURCE_NOT_FOUND` |
| `DuplicateResourceException` | `409 Conflict` | `DUPLICATE_RESOURCE` |
| `IdempotencyConflictException`| `409 Conflict` | `IDEMPOTENCY_CONFLICT` |
| `InvalidStateTransitionException`| `422 Unprocessable Entity`| `INVALID_STATE_TRANSITION` |
| `Exception` (أي خطأ غير متوقع) | `500 Internal Server Error`| `INTERNAL_SERVER_ERROR` |

---

## 5. سياق المستأجر والأمان (Multi-Tenancy & Security Context)

### 5.1 سياق المستأجر: `TenantContext.java`
يستخدم `ThreadLocal` للحفاظ على هوية الجيم والفرع أثناء تنفيذ الـ HTTP Request الحالي، مع مسحه حتماً في الـ `finally block`.

```java
package com.kammel.shared.security;

import java.util.UUID;

public final class TenantContext {

    private static final ThreadLocal<UUID> currentOrg = new ThreadLocal<>();
    private static final ThreadLocal<UUID> currentBranch = new ThreadLocal<>();

    private TenantContext() {}

    public static void setOrgId(UUID orgId) {
        currentOrg.set(orgId);
    }

    public static UUID getOrgId() {
        return currentOrg.get();
    }

    public static void setBranchId(UUID branchId) {
        currentBranch.set(branchId);
    }

    public static UUID getBranchId() {
        return currentBranch.get();
    }

    public static void clear() {
        currentOrg.remove();
        currentBranch.remove();
    }
}
```

### 5.2 فلتر استخراج المستأجر وضبط قاعدة البيانات: `TenantFilter.java`
يقوم هذا الفلتر بالتقاط الـ JWT، والتحقق من صحته، ثم استخراج الـ `org_id` وضبطه في الـ `TenantContext` وتشغيل استعلام `SET LOCAL app.current_org = ?` على مستوى اتصال قاعدة البيانات لتفعيل سياسات الـ RLS فوراً.

---

## 6. كلاسات الأدوات الحسابية والزمنية (Pure Utilities)

### 6.1 أداة التعاملات المالية بالقروش: `MoneyUtil.java`
تمنع نهائياً التعامل مع الكسور العشرية الخاطئة.

```java
package com.kammel.shared.util;

import java.text.DecimalFormat;

public final class MoneyUtil {

    private static final DecimalFormat FORMATTER = new DecimalFormat("#,##0.00");

    private MoneyUtil() {}

    /**
     * تحويل الجنيهات إلى قروش (مثال: 700.50 EGP -> 70050 piasters)
     */
    public static long toMinor(double amountInEgp) {
        return Math.round(amountInEgp * 100.0);
    }

    /**
     * تحويل القروش إلى جنيهات كقيمة عشرية
     */
    public static double toMajor(long amountInMinor) {
        return amountInMinor / 100.0;
    }

    /**
     * تنسيق المبلغ للعرض بالجنيه المصري (مثال: 70000 -> "700.00 EGP")
     */
    public static String formatEgp(long amountInMinor) {
        return FORMATTER.format(toMajor(amountInMinor)) + " EGP";
    }
}
```

### 6.2 أداة فحص وتنسيق أرقام الهواتف: `PhoneUtil.java`
تطبيع رقم الهاتف المحمول المصري إلى الصيغة المعيارية `01xxxxxxxxx` أو صيغة الرابط الدولي `201xxxxxxxxx`.

```java
package com.kammel.shared.util;

import java.util.regex.Pattern;

public final class PhoneUtil {

    private static final Pattern EGYPT_PHONE_PATTERN = Pattern.compile("^(010|011|012|015)[0-9]{8}$");

    private PhoneUtil() {}

    public static boolean isValidEgyptianPhone(String phone) {
        if (phone == null) return false;
        String clean = phone.trim().replaceAll("[\\s\\-+]", "");
        if (clean.startsWith("201")) {
            clean = clean.substring(1); // إرجاعها لصيغة 01xxxxxxxxx
        }
        return EGYPT_PHONE_PATTERN.matcher(clean).matches();
    }

    public static String toWaMeFormat(String phone) {
        String clean = phone.trim().replaceAll("[\\s\\-+]", "");
        if (clean.startsWith("01")) {
            return "2" + clean; // مثال: 201012345678
        }
        return clean;
    }
}
```

### 6.3 أداة التوقيت المصري الموحد: `DateUtil.java`
تضمن حفظ جميع التواريخ كـ `UTC Instant` في قاعدة البيانات مع العرض والفلترة اليومية وفقاً لمنطقة `Africa/Cairo`.

---

## 7. المدققات المخصصة (Custom Bean Validators)

### أنوتيشن `@ValidPhone`:
يتم وضعه على حقول أرقام الهواتف في الـ Request DTOs لضمان الرفض المبكر لأي رقم موبايل غير مصري سليم:

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneValidator.class)
public @interface ValidPhone {
    String message() default "Invalid Egyptian mobile number format (must start with 010, 011, 012, or 015 and be 11 digits)";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```
