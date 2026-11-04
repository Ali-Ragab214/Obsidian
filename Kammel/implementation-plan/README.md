# خطة تنفيذ كمّل (Kammel Implementation Plan)

> **المشروع:** Kammel — Multi-Tenant Gym Retention & Revenue Management SaaS  
> **السوق:** الجيمات المستقلة والسلاسل الصغيرة في مصر  
> **الـ Stack:** Java 21 + Spring Boot 3 + PostgreSQL 16 + Flyway + Spring Security + JWT + Docker  
> **الـ Architecture:** Feature-Based Modular Monolith (Vertical Slices)  
> **حالة الخطة:** جاهزة للتنفيذ — الإصدار 1.0

---

## 1. الخطة دي بتعمل إيه؟

الملفات دي هي **خطة التنفيذ العملية** لمشروع كمّل، مبنية على الوثائق الأصلية الموجودة في:

- [`../Kammel_Project_Documentation/`](../Kammel_Project_Documentation/README.md) — الوثائق التقنية المرجعية (Primary Source of Truth).
- [`../0-doc.md`](../0-doc.md) — وثيقة المنتج الأصلية.
- [`../1-doc2.md`](../1-doc2.md) — التقييم الهندسي والملاحظات على النطاق.
- [`../implementation-blueprint/`](../implementation-blueprint/00-project-overview.md) — Blueprint سابق بنفس فلسفة الـ Feature-Based.

الخطة دي **مش كود**، ومش بديل عن الوثائق الأصلية. هي بتجاوب على 4 أسئلة:

1. **هنبني إيه بالظبط؟** — الـ Features ونطاق كل واحدة.
2. **هنبني إزاي؟** — الـ Architecture والـ Shared Components والـ Contracts.
3. **هنبني بالترتيب إيه؟** — Feature-by-Feature مع Dependencies وCheckpoints.
4. **إزاي نتأكد إن اللي اتبنى صح؟** — Testing وDefinition of Done وMaster Checklist.

---

## 2. إزاي تستخدم الخطة دي؟

### لو هتبدأ من الصفر
1. اقرأ [project-overview](./architecture/project-overview.md) و[decisions-and-assumptions](./architecture/decisions-and-assumptions.md).
2. اقرأ [project-structure](./architecture/project-structure.md) و[shared-foundation](./architecture/shared-foundation.md).
3. افتح [implementation-phases](./development-roadmap/implementation-phases.md) واشتغل Phase by Phase.
4. استخدم [master-checklist](./development-roadmap/master-checklist.md) كـ Progress Tracker يومي.

### لو بتشتغل على Feature معينة
1. افتح فولدر الـ Feature من [`features/`](./features/).
2. اقرأ `README.md` → `domain-design.md` → `implementation-plan.md` → `api-design.md` → `testing-plan.md`.
3. راجع الـ Dependencies في [feature-dependencies](./architecture/feature-dependencies.md) قبل ما تبدأ.
4. علّم على المهام في [master-checklist](./development-roadmap/master-checklist.md) بعد كل Task.

### لو بتعمل Integration بين ميزتين
- اقرأ [integration-roadmap](./development-roadmap/integration-roadmap.md).
- راجع الـ Events والـ Contracts في [feature-dependencies](./architecture/feature-dependencies.md).

---

## 3. المشروع في سطر واحد

> كمّل بيقول لصاحب الجيم **مين هيسيب الشهر الجاي**، وبيجهّز **الرسالة اللي ترجّعه**، وبيوريه **فلوسه بتروح فين**.

المنتج مش Gym Management System تقليدي. هو **Retention & Collections Engine** فوق نواة تشغيلية خفيفة:

- النواة: Members + Plans + Subscriptions + Payments + Check-ins.
- القيمة: Risk Score + Renewal Queue + Collections + Dashboard + Audit.
- القنوات: WhatsApp يدوي عبر `wa.me` في الـ MVP، والـ Cloud API في P1.
- الـ AI: مؤجل لمرحلة ما بعد الـ MVP — الـ Risk Score في الـ MVP **Rule-Based ومفسّر**، مش ML.

---

## 4. الـ Features ونطاقها

| # | Feature | Folder | Phase | Scope | المسؤولية في سطر |
|---|---|---|---|---|---|
| 1 | Tenancy | [`features/tenancy`](./features/tenancy/README.md) | Phase 1 | **P0** | المنظمات والفروع وعزل البيانات. |
| 2 | Identity | [`features/identity`](./features/identity/README.md) | Phase 1 | **P0** | تسجيل دخول الموظفين، JWT، Refresh Tokens، الأدوار. |
| 3 | Audit | [`features/audit`](./features/audit/README.md) | Phase 1 (Core) / Phase 3 (API) | **P0** | سجل غير قابل للتعديل للعمليات الحساسة. |
| 4 | Member | [`features/member`](./features/member/README.md) | Phase 2 | **P0** | تسجيل الأعضاء، البحث، الـ Soft Delete، استيراد Excel. |
| 5 | Plan | [`features/plan`](./features/plan/README.md) | Phase 2 | **P0** | باقات الاشتراك والأسعار والمدد وقواعد التجميد. |
| 6 | Subscription | [`features/subscription`](./features/subscription/README.md) | Phase 2 | **P0** | دورة حياة الاشتراك، التجديد، التجميد، الأحداث. |
| 7 | Payment | [`features/payment`](./features/payment/README.md) | Phase 2 | **P0** | تسجيل الدفع، الـ Idempotency، الإيصالات، المدفوعات الجزئية. |
| 8 | Check-in | [`features/checkin`](./features/checkin/README.md) | Phase 3 | **P0** | الحضور بالبحث/QR، فحص الاشتراك، خصم الحصص. |
| 9 | Risk | [`features/risk`](./features/risk/README.md) | Phase 3 | **P0** | Risk Score بقواعد واضحة + أسباب + Draft Message. |
| 10 | Notification | [`features/notification`](./features/notification/README.md) | Phase 3 | **P0** | طابور رسائل WhatsApp عبر `wa.me` + Attribution. |
| 11 | Reporting | [`features/reporting`](./features/reporting/README.md) | Phase 3 | **P0** | Dashboard المالك وKPIs وCollections. |
| 12 | Shift | [`features/shift`](./features/shift/README.md) | Post-MVP | P1 | جرد الكاش ومطابقة الورديات. |
| 13 | Lead | [`features/lead`](./features/lead/README.md) | Post-MVP | P1 | قمع المبيعات والـ Leads. |
| 14 | Member Portal | [`features/member-portal`](./features/member-portal/README.md) | Post-MVP | P1 | PWA للعضو: اشتراك، QR، حضور. |

> **قاعدة:** الـ P0 Features هي الـ MVP. الـ P1 Features لها ملفاتها، لكنها **مش جزء من الـ 12-week MVP** ومتتنفذش غير بعد نجاح الـ Pilot.

---

## 5. خريطة الملفات

```text
implementation-plan/
├── README.md                          ← أنت هنا
│
├── architecture/
│   ├── project-overview.md            ← الرؤية، المستخدمين، الأهداف، نطاق المنتج
│   ├── requirements-traceability.md   ← ربط BR-001..BR-041 بالـ Features
│   ├── decisions-and-assumptions.md   ← القرارات، التعارضات، الافتراضات، الأسئلة المفتوحة
│   ├── project-structure.md           ← هيكل الحزم والـ Vertical Slices
│   ├── shared-foundation.md           ← الـ Shared Components بالتفصيل
│   ├── entity-relationships.md        ← الـ ERD وملكية الكيانات
│   ├── feature-dependencies.md        ← Dependency Matrix + Mermaid + Order
│   ├── database-strategy.md           ← الجداول، الـ Migrations، الفهارس، RLS
│   └── security-strategy.md           ← Auth، RBAC، Multi-Tenancy، Privacy
│
├── features/
│   ├── tenancy/                       ← README + implementation + domain + api + testing
│   ├── identity/
│   ├── member/
│   ├── plan/
│   ├── subscription/
│   ├── payment/
│   ├── checkin/
│   ├── risk/
│   ├── notification/
│   ├── reporting/
│   ├── audit/
│   ├── shift/                         ← P1 (README + implementation-plan)
│   ├── lead/                          ← P1 (README + implementation-plan)
│   └── member-portal/                 ← P1 (README + implementation-plan)
│
├── api-design.md                      ← قواعد الـ API والـ Endpoint Catalog
├── testing-strategy.md                ← استراتيجية الاختبارات العامة
│
├── development-roadmap/
│   ├── implementation-phases.md       ← المراحل الأسبوعية بالتفصيل
│   ├── integration-roadmap.md         ← Integration Checkpoints بين الـ Features
│   └── master-checklist.md            ← الـ Progress Tracker الرئيسي
│
└── ai/
    ├── ai-overview.md                 ← لماذا الـ AI مؤجل ومتى ندخله
    └── ai-integration-roadmap.md      ← ML + Spring AI + Tools + Testing
```

---

## 6. قواعد القراءة والكتابة

### اللغة
- الشرح بالعامية المصرية.
- المصطلحات التقنية وأسماء الـ Classes والـ Methods والـ Packages بالإنجليزي.
- ممنوع ترجمة أسماء الـ Enums أو الـ Endpoints أو الـ Tables.

### المصادر
- الوثائق الأصلية هي المرجع الأول.
- لو حصل تعارض بين وثيقتين، القرار والسبب متسجلين في [decisions-and-assumptions](./architecture/decisions-and-assumptions.md).
- أي حاجة مش موجودة في الوثائق ومحتاجة قرار، متسجلة كـ **Assumption (ASM-xx)** وبتتأكد في Phase 0 أو مع أول Pilot.

### الـ IDs المستخدمة
| Prefix | المعنى | الملف |
|---|---|---|
| `BR-xxx` | Business Rule من الوثائق الأصلية | [requirements-traceability](./architecture/requirements-traceability.md) |
| `ASM-xx` | Assumption محتاجة تأكيد | [decisions-and-assumptions](./architecture/decisions-and-assumptions.md) |
| `TSK-XXX-xxx` | Task تنفيذي | Feature `implementation-plan.md` |
| `CPx` | Checkpoint | [implementation-phases](./development-roadmap/implementation-phases.md) |
| `D-x` | Architecture Decision | [decisions-and-assumptions](./architecture/decisions-and-assumptions.md) |

---

## 7. الـ MVP في سطر واحد

> جيم حقيقي بيستخدم كمّل **14 يوم متواصلين** من غير ما يرجع للدفتر أو الـ Excel، و**70% على الأقل** من الـ Check-ins متسجلة في النظام، وصاحب الجيم شايف **تجديد واحد على الأقل** اتربط برسالة من النظام.

---

## 8. أهم 5 قرارات هتشتغل عليها من أول يوم

1. **Feature-Based Architecture** — كل Feature مالكة الـ Entity والـ Repository والـ Service والـ API والـ Tests بتاعتها.
2. **PostgreSQL RLS + `org_id` + TenantContext** — عزل البيانات مش مجرد `WHERE` في الكود.
3. **المبالغ كلها `Long` بالقروش** — ممنوع `double` أو `float`.
4. **الاشتراك State Machine بالأحداث** — ممنوع تعديل `status` يدوي.
5. **الـ WhatsApp يدوي في الـ MVP** — `wa.me` + تأكيد إرسال، والـ Cloud API في P1.

---

## 9. الـ Definition of Done العام

أي Feature متعتبرش خلصت غير لما:

- [ ] الـ Flyway Migration موجودة وبتشتغل من قاعدة بيانات نضيفة.
- [ ] الـ Entity فيه `org_id` (لو Tenant-owned) وRLS مفعّلة.
- [ ] الـ Repository والـ Service والـ Controller موجودين.
- [ ] كل الـ Service methods محمية بـ `@PreAuthorize`.
- [ ] الـ DTOs والـ Validation موجودين، والـ Entity مش بتخرج للـ API.
- [ ] الـ Unit Tests + Integration Tests + Tenant Isolation Test موجودين.
- [ ] الـ API موثقة في `api-design.md` بتاع الـ Feature.
- [ ] الـ Master Checklist محدّثة.
- [ ] الـ Feature بتشتغل End-to-End مع الـ Features اللي قبلها.

التفاصيل الكاملة في [implementation-phases](./development-roadmap/implementation-phases.md#2-definition-of-done).
