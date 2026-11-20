# **JPA Auditing vs Advanced Auditing (Tracking Current User)**

خلينا نعتبر الموضوع ده جزء من System Design وDatabase Management، لأنك هتقابله كتير في الـ Enterprise Applications والأنظمة اللي فيها أكتر من User.

# **1. What is JPA Auditing?**

**JPA Auditing** هي Feature موجودة في Spring Data JPA بتسمحلك تتابع التغييرات اللي بتحصل على الـ Database Entities بشكل تلقائي.
track the changes on spring data jpa  automatically


بمعنى إنك تعرف:

- إمتى الـ Entity اتعملت؟
    
- إمتى آخر تعديل حصل عليها؟
    

بدل ما تكتب الكود ده بنفسك في كل Service:

```
entity.setCreatedAt(LocalDateTime.now());
entity.setUpdatedAt(LocalDateTime.now());
```

Spring بيتولى الموضوع تلقائيًا.

## **2. Basic JPA Auditing (Date Tracking)**

ده أبسط شكل من الـ Auditing، وبيتابع التوقيت فقط.

**Step 1: Create BaseEntity**

```
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
@Setter
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

شرح الـ Annotations:

|Annotation|Responsibility|
|---|---|
|`@MappedSuperclass`|بتخلي الـ Entities اللي بتورث منها تستخدم الـ Fields الموجودة فيها|
|`@EntityListeners`|بتربط الـ Entity بـ Auditing Listener|
|`@CreatedDate`|بتسجل وقت إنشاء الـ Entity|
|`@LastModifiedDate`|بتسجل وقت آخر تعديل|

**Step 2: Enable JPA Auditing**

نعمل Configuration Class:

```
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

الـ `@EnableJpaAuditing` بتفعّل نظام الـ Auditing في Spring Data JPA.

كده خلصنا!

أي Entity بتعمل `extends BaseEntity` هتستفيد من الـ Auditing تلقائيًا.

# **3. Advanced JPA Auditing (User Tracking)**

طيب عندنا مشكلة دلوقتي.

تخيل عندك نظام لإدارة المخازن، وفيه 100 موظف.

موظف عمل Stock Movement، وبعدها موظف تاني عدّل عليه.

أنت عارف إن التعديل حصل الساعة كام، لكن مش عارف مين اللي عمله!

هنا بنحتاج نوع تاني من الـ Auditing اسمه **User Tracking**.

بدل ما نتابع الوقت فقط، هنتابع كمان هوية المستخدم اللي نفذ العملية.

هنضيف Fieldين:

```
@CreatedBy
private String createdBy;

@LastModifiedBy
private String updatedBy;
```

| Annotation        | Responsibility                      |
| ----------------- | ----------------------------------- |
| `@CreatedBy`      | بتسجل المستخدم اللي أنشأ الـ Entity |
| `@LastModifiedBy` | بتسجل آخر مستخدم عدّل الـ Entity    |
|                   |                                     |

لكن فيه سؤال مهم:

**Spring هيعرف منين مين المستخدم الحالي؟**

هنا بييجي دور `AuditorAware`.

# **4. What is AuditorAware?**

`AuditorAware` عبارة عن Interface من Spring Data JPA.

مسؤوليته إنه يحدد هوية المستخدم الحالي اللي نفذ العملية.

في مشروعنا هنجيب المستخدم الحالي من Spring Security باستخدام:

`SecurityContextHolder`.

## **Understanding SecurityContextHolder**

لما المستخدم يعمل Login ويبعت Request، Spring Security بيحتفظ بمعلومات الـ Authentication داخل الـ SecurityContext.

تقدر توصل للمستخدم الحالي عن طريق:

```
Authentication authentication =
    SecurityContextHolder.getContext().getAuthentication();
```

الـ Authentication بيحتوي على معلومات زي:

- Principal: هوية المستخدم.
    
- Authorities: الصلاحيات والـ Roles.
    
- Authenticated: هل المستخدم تم التحقق منه؟
    

## **5. Implementing AuditorAware**

نعمل Configuration Class:

```
@Configuration
public class JpaAuditingConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }

    public static class AuditorAwareImpl
            implements AuditorAware<String> {

        @Override
        public Optional<String> getCurrentAuditor() {

            Authentication authentication =
                SecurityContextHolder.getContext()
                    .getAuthentication();

            if (authentication == null ||
                !authentication.isAuthenticated() ||
                authentication.getPrincipal().equals("anonymousUser")) {

                return Optional.empty();
            }

            if (authentication.getPrincipal() != null) {
                return Optional.of(
                    (String) authentication.getPrincipal()
                );
            }

            return Optional.empty();
        }
    }
}
```

خلينا نفهم الـ Flow:

**User sends HTTP Request**

**Spring Security**

Authenticates the user

**SecurityContextHolder**

Stores current Authentication

**AuditorAware**

Extracts current user ID

**JPA Auditing Listener**

Sets createdBy / updatedBy automatically

**Database**

Persists auditing information

**مهم تفهم نقطة:** `AuditorAware` مش بيعمل Authentication بنفسه، هو بس بيقرأ المستخدم اللي Spring Security سبق وعمل له Authentication.

# **6. Final BaseEntity**

بعد ما فهمنا الجزئين، نقدر نجمعهم في BaseEntity واحدة:

```
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
@Setter
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}
```

وتأكد إن عندك `@EnableJpaAuditing`، وإن Spring شايف الـ `AuditorAware` Bean.

# **7. Basic vs Advanced JPA Auditing**

ده أهم فرق لازم تكون فاهمه:

|Comparison|Basic Auditing|Advanced Auditing|
|---|---|---|
|Created Date|Yes|Yes|
|Updated Date|Yes|Yes|
|Created By|No|Yes|
|Updated By|No|Yes|
|AuditorAware|Not required|Required|
|Spring Security|Not required|Commonly used|

**ملحوظة:** الـ Advanced Auditing مش Feature مختلفة عن JPA Auditing؛ هو نفس نظام Spring Data JPA Auditing، لكن بنستخدم إمكانيات إضافية فيه لتتبع المستخدمين.

# **8. Important Notes for Real-world Applications**

فيه شوية نقاط مهمة:

1. **Principal Type:** مش شرط `authentication.getPrincipal()` يرجع `String`. لو بتستخدم CustomUserDetails، اعمل Cast للـ CustomUserDetails واستخرج الـ User ID.

2. **Anonymous Requests:** لو مفيش مستخدم authenticated، ممكن `AuditorAware` يرجع `Optional.empty()`.

3. **Background Jobs:** لو عندك Scheduled Jobs أو Async Tasks، مش بالضرورة يكون فيه SecurityContext. ممكن تحتاج System Auditor بدل المستخدم العادي.

4. **Multi-Tenancy:** خلي بالك إن `createdBy` و`updatedBy` بيسجلوا هوية المستخدم، لكن مش بديل عن `tenant_id` أو آليات عزل بيانات الـ Tenants.

5. **ThreadLocal:** الـ SecurityContext بيكون مرتبط بالـ Thread في الإعدادات المعتادة، فلا تفترض إنه هينتقل تلقائيًا لأي Async Thread.

# **9. Interview Questions**

اختبر فهمك بالأسئلة دي:

**JPA Auditing Quiz**

0/4 answered

**1****.** **What is the main purpose of @CreatedDate?**

Track the current user

Track entity creation time

Track entity deletion

**2****.** **What is the responsibility of AuditorAware?**

Authenticate users

Connect to the database

Identify the current user performing an operation

**3****.** **Do you need Spring Security to use basic JPA Auditing?**

Yes, always

No

Only with PostgreSQL

**4****.** **What happens if getCurrentAuditor() returns Optional.empty()?**

Spring automatically uses admin

No auditor value is supplied

The application must crash

Check answers

**أهم حاجة تخرج بيها من الموضوع:**

JPA Auditing بيجاوب على سؤالين أساسيين:

- When? باستخدام `@CreatedDate` و`@LastModifiedDate`.
    
- Who? باستخدام `@CreatedBy` و`@LastModifiedBy` مع `AuditorAware`.
    

وده بيديك أساس قوي للـ Auditing في أي Enterprise Application.