# نظرة عامة على المشروع (Project Overview)

> **المرجع:** [`../../Kammel_Project_Documentation/00_PROJECT_OVERVIEW.md`](../../Kammel_Project_Documentation/00_PROJECT_OVERVIEW.md) · [`../../0-doc.md`](../../0-doc.md) · [`../../1-doc2.md`](../../1-doc2.md)  
> **الملفات المرتبطة:** [requirements-traceability](./requirements-traceability.md) · [decisions-and-assumptions](./decisions-and-assumptions.md) · [project-structure](./project-structure.md)

---

## 1. المنتج في سطر واحد

**كمّل (Kammel)** منصة SaaS متعددة المستأجرين (Multi-Tenant) لإدارة **الاحتفاظ بالأعضاء والتحصيل** في الجيمات المصرية.

> بيقول لصاحب الجيم مين هيسيب الشهر الجاي، وبيجهّز الرسالة اللي ترجّعه، وبيوريه فلوسه بتروح فين.

المنتج مش ERP ولا Gym Management System كامل. هو **Retention & Collections Engine** فوق نواة تشغيلية خفيفة:

```text
Core Operations (Members, Plans, Subscriptions, Payments, Check-ins)
        ↓ data
Intelligence (Risk Score, Renewal Queue, Collections, Dashboard)
        ↓ action
Outcome (Renewals, Less Leakage, Measurable EGP)
```

---

## 2. المشكلة اللي بنحلها

| المشكلة | الأثر على الجيم | حل كمّل |
|---|---|---|
| الأعضاء بيسيبوا بصمت | إيراد شهري بيضيع بعد ما يفوت الوقت | Risk Score + At-Risk List + Message Queue |
| التجديد والتحصيل يدوي | متابعة غير منتظمة ومفيش مسؤولية | Renewal Queue + Collections + Attribution |
| الكاش بيعدي على الموظفين | تسريب غير مكتشف | Payments + Audit + Shift Reconciliation (P1) |
| الليدز بتضيع | فلوس إعلانات بتضيع | Leads CRM (P1) |
| مفيش رؤية مالية | قرارات بالحدس | Owner Dashboard + KPIs |

> **مهم:** كل أرقام السوق والسلوكيات دي **فرضيات** لازم تتأكد في Phase 0. مفيش رقم سوق مثبت في الوثائق.

---

## 3. المستخدمون (Personas)

| الشخصية | الدور | احتياجه الأساسي | مقياس نجاحه |
|---|---|---|---|
| **صاحب الجيم (Buyer)** | قرار الشراء ومتابعة الإيراد | يعرف مين هيسيب، والفلوس رايحة فين، وكل فرع بيكسب كام | إيراد أعلى وتسريب أقل |
| **مدير الفرع** | إدارة تشغيلية للفرع | موافقات، تقارير فرعه، متابعة الكاش | فرع منضبط |
| **موظف الاستقبال (Daily User)** | العمليات اليومية | شاشة سريعة وبسيطة وقت الزحمة | مفيش تعطيل |
| **الكابتن (Trainer)** | متابعة عملائه | يشوف عملاءه وحضورهم | عملاء ملتزمين |
| **العضو (End User)** | عميل الجيم | يعرف اشتراكه ويدخل بسهولة | تجربة سهلة |

> **قاعدة تصميم:** المشتري (Owner) واليوزر اليومي (Receptionist) ناس مختلفة. النظام لازم يبيع للـ Owner (Dashboard) ويشتغل للـ Receptionist (Speed & Simplicity). لو الريسبشن صعّب عليه، النظام هيتساب.

---

## 4. أهداف البيزنس

1. تقليل الـ Churn من خلال اكتشاف المخاطر قبل ما العضو يمشي.
2. منع تسريب الإيرادات من خلال Payments + Audit + Reconciliation.
3. أتمتة متابعة التجديدات من خلال WhatsApp Queue.
4. إعطاء صاحب الجيم رؤية مالية حقيقية (Dashboard + KPIs).
5. إثبات ROI بالجنيه مش بالـ Features.
6. دعم الفروع المتعددة من أول يوم في الـ Schema.
7. بناء Data Asset تشغيلي (حضور + دفع + تجديد) يفتح باب الـ AI بعدين.

---

## 5. الـ Value Proposition

### معادلة الـ Break-Even (توضيحية — تتأكد في الـ Pilot)

جيم 300 عضو، متوسط اشتراك 700 جنيه، Churn شهري 8%:

- 24 عضو بيسيبوا كل شهر = **16,800 جنيه** إيراد شهري بيضيع.
- لو الـ Churn نزل لـ 6% = **6 أعضاء** زيادة بيفضلوا.
- اشتراك كمّل ~1,000 جنيه/شهر → إنقاذ **عضوين** (1,400 جنيه) يغطي التكلفة.

> الرقم ده **مش وعد**، هو فرضية تتأكد بأرقام جيم حقيقي في الـ Pilot.

### الركائز الثلاثة

1. **Retention Intelligence** — Risk Score مفسّر + رسائل جاهزة.
2. **Revenue Leakage Detection** — Audit + Collections + Cash Variance.
3. **Operational Efficiency** — Check-in سريع + إيصالات + تقارير.

---

## 6. نطاق المنتج (P0 / P1 / P2)

| Module | P0 (MVP) | P1 (Post-MVP) | P2 (Advanced) |
|---|---|---|---|
| Members & Plans | CRUD + Excel Import + باقات أساسية | Coupons / Discounts | Family / Corporate |
| Subscriptions | Lifecycle + Renewal + Freeze | Smart Alerts | Auto-renewal |
| Payments | Cash/Transfer + Receipt + Partial | Refunds + Paymob + Shifts | Installments |
| Attendance | Search + Static QR | Dynamic QR | Hardware |
| Roles | 4 Roles + Audit | Custom Permissions | SSO |
| Dashboard | Renewals + Collections + Risk | Trainer Performance + Peak Hours | Branch Comparison |
| WhatsApp | Manual Queue (`wa.me`) | Cloud API | Campaigns |
| Leads | — | Simple CRM | Instagram Integration |
| Member App | — | PWA | Challenges |
| AI | Rule-based Risk Score | — | ML + Assistant |

> التفاصيل الكاملة في [decisions-and-assumptions](./decisions-and-assumptions.md) و[development roadmap](../development-roadmap/implementation-phases.md).

---

## 7. الـ MVP — تعريف النجاح

### جملة النجاح

> **جيم حقيقي بيستخدم كمّل يومياً 14 يوم متواصلين من غير ما يرجع للدفتر أو الـ Excel.**

### معايير قابلة للقياس

| المعيار | الهدف |
|---|---|
| أيام الاستخدام المتواصل | 14 يوم |
| نسبة الـ Check-ins المسجلة في النظام | ≥ 70% |
| Time to First Value (من الاستيراد لأول تذكير) | < 1 يوم |
| تجديد واحد على الأقل مرتبط برسالة | ≥ 1 |
| صاحب الجيم يقدر يشوف قيمة بالجنيه | نعم |

### الـ North Star Metric

> **قيمة التجديدات المنسوبة للنظام بالجنيه** — رسالة اتبعتت → العضو جدد خلال 7 أيام → القيمة تتسجل.

---

## 8. الـ KPIs — تعريفات ثابتة من أول يوم

| KPI | المعادلة |
|---|---|
| **Renewal Rate** | اللي جددوا ÷ اللي اشتراكهم انتهى في الفترة |
| **Monthly Churn** | اللي مجددوش بعد Grace ÷ الأعضاء النشطين أول الشهر |
| **ARPM** | إيراد الاشتراكات ÷ الأعضاء النشطين |
| **Weekly Visits / Member** | إجمالي الـ Check-ins ÷ الأعضاء النشطين ÷ عدد الأسابيع |
| **Average Lifetime** | متوسط عدد الشهور من أول اشتراك لآخر Churn |
| **Attributed Revenue** | قيمة التجديدات اللي حصلت خلال 7 أيام من رسالة مؤكدة |

> **قاعدة:** التعريفات دي متتغيرش بعد بداية الـ Pilot، عشان الأرقام تبقى قابلة للمقارنة.

---

## 9. خارج النطاق (Explicitly Out of Scope)

| الحاجة | السبب |
|---|---|
| ERP / محاسبة / ضرائب | كمّل مش نظام محاسبة |
| Hardware (Turnstiles) | تعقيد دعم وهاردوير |
| Payment Aggregation (نمسك فلوس) | تعقيد قانوني — كمّل بيسجل مش بيعالج |
| Marketplace | موديل بيزنس مختلف |
| Native Mobile App | PWA كافية |
| AI/ML في الـ MVP | بيانات غير كافية |
| Workout Programming | مش core للـ Retention |
| Medical / Health Data | مسؤولية قانونية |

---

## 10. الـ Non-Functional Requirements

| Requirement | الهدف |
|---|---|
| **Tenant Isolation** | صفر تسريب بين الجيمات — RLS + Tests |
| **Financial Integrity** | مفيش أخطاء حسابية — `Long` بالقروش |
| **Auditability** | كل عملية حساسة متسجلة Immutable |
| **Check-in Latency** | < 1 ثانية (هدف عملي < 500ms) |
| **Availability** | 99.5%+ |
| **Arabic RTL** | دعم كامل |
| **Mobile Responsive** | Tablet + Phone |
| **Security** | HTTPS + BCrypt + JWT + Rate Limiting |
| **Privacy** | Egyptian Data Protection Law #151/2020 |
| **Backup** | يومي مشفر + Restore Tested |
| **Concurrent Users** | 5–15 لكل جيم في الذروة |

---

## 11. الـ Personas → الـ Features Mapping

| الشخصية | الـ Features الأساسية |
|---|---|
| Owner | Reporting, Risk, Notification, Audit, Subscription, Payment |
| Branch Manager | Check-in, Subscription (Freeze), Payment, Shift (P1), Reporting (Branch) |
| Receptionist | Member, Check-in, Payment, Notification Queue |
| Trainer | Member (Read), Check-in (Read) — P1 Client List |
| Member | Member Portal (P1) |

---

## 12. الافتراضات الحرجة (تحتاج تأكيد قبل الكود)

| # | الافتراض | يتأكد إزاي |
|---|---|---|
| A1 | أصحاب الجيمات شايفين الـ Churn مشكلة تستاهل دفع | 10 مقابلات |
| A2 | الاستقبال هيتقبل النظام لو سريع | Pilot Observation |
| A3 | WhatsApp Reminders بترفع التجديد | Holdout Group |
| A4 | الكاش مشكلة كبيرة تستاهل Shift Reconciliation | مقابلات |
| A5 | أغلب الجيمات على Excel/دفتر | Market Research |
| A6 | أول 60 يوم هي أخطر فترة | تحليل بيانات الـ Pilot |
| A7 | مطور واحد يقدر يخلص MVP في 12 أسبوع | Effort Tracking |

> أي افتراض يتأكد أو يتغير، يتحدث في [decisions-and-assumptions](./decisions-and-assumptions.md) وفي [master-checklist](../development-roadmap/master-checklist.md).

---

## 13. الـ Success Metrics للمنتج

| Metric | Definition | Target |
|---|---|---|
| Check-in Registration Rate | % الحضور المسجل في النظام | ≥ 70% في الـ Pilot |
| Time to First Value | من Excel Import لأول Reminder | < 1 يوم |
| Message-to-Renewal Rate | % الرسائل اللي اتبعتت وجابت تجديد خلال 7 أيام | يتقاس ويتحسن |
| Daily Active Usage | استخدام يومي بدون رجوع للورق | 14 يوم |
| Paying Gyms (6 شهور) | عدد الجيمات الدافعة | 5–10 |
| Gym Churn | % الجيمات اللي بتوقف الدفع | < 10% |

---

## 14. المخاطر الأساسية وخطط التعامل

| الخطر | التأثير | الإجراء |
|---|---|---|
| فريق الجيم مش هيستخدم النظام | عالي | Champion على الاستقبال + UI بسيط + تدريب |
| تسريب بيانات بين الجيمات | عالي جداً | RLS + Tenant Tests + Pen Test |
| أخطاء مالية | عالي | Idempotency + State Machine + Tests |
| WhatsApp Ban | متوسط | Manual `wa.me` في MVP |
| Scope Creep | متوسط | الالتزام بـ P0 + Master Checklist |
| مطور واحد / Burnout | عالي | 12 أسبوع + Checkpoints أسبوعية |
| مشاكل قانونية | متوسط | مراجعة قانونية قبل الإطلاق |

---

## 15. القرارات المعمارية الكبرى (ملخص)

| القرار | الاختيار | السبب |
|---|---|---|
| Style | Modular Monolith | مطور واحد + Transactions بسيطة |
| Packaging | Feature-Based Vertical Slices | High Cohesion / Low Coupling |
| DB | PostgreSQL 16 | RLS + JSONB + Indexes |
| Tenancy | Shared Schema + `org_id` + RLS | أبسط تشغيلاً وأأمن |
| Money | `Long` Minor Units (Piasters) | دقة مالية |
| State | Event-Driven Subscription State Machine | Audit + منع التعديل اليدوي |
| Messaging | Spring Events | فصل بدون Broker |
| Jobs | Spring Scheduler + ShedLock | منع التشغيل المكرر |
| WhatsApp | Manual `wa.me` (P0) | بدون تكلفة/حظر |
| Cache | بدون Redis في MVP | مش Needed |
| AI | مؤجل — Rules أولاً | بيانات غير كافية |

التفاصيل في [decisions-and-assumptions](./decisions-and-assumptions.md).
