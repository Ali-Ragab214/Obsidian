# Reporting — لوحة المالك والتقارير

> **الـ Phase:** 3 · **الأولوية:** P0 · **الترتيب التنفيذي:** 11 (آخر Feature)  
> **الـ Package:** `com.kammel.features.reporting`  
> **المرجع:** [`../../../Kammel_Project_Documentation/07_API_DESIGN.md`](../../../Kammel_Project_Documentation/07_API_DESIGN.md) · [`../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md`](../../../Kammel_Project_Documentation/11_BACKGROUND_JOBS_AND_NOTIFICATIONS.md)

---

## 1. الميزة دي بتعمل إيه؟

`reporting` هي **عيون صاحب الجيم**:

- ملخص تنفيذي: تجديدات الأسبوع، الإيراد، معدل التجديد، الـ Churn.
- قائمة التجديدات المتوقعة.
- التحصيل والمتأخرات (Collections) مع Aging Buckets.
- الحضور حسب الفترة والفرع والساعات.
- توزيع المخاطر.
- **Timeline موحّد** للعضو: اشتراكات + مدفوعات + حضور + رسائل + Risk.
- كل الأرقام **Org-scoped** و**Branch-scoped** حسب الدور.

**مش مسؤولة عن:**
- تعديل أي بيانات — **Read-Only**.
- حساب الـ Risk أو الـ Payment — بتقرأ النتائج.
- إرسال رسائل — `notification`.
- تحديث الحالة — `subscription` / `payment`.

---

## 2. المصطلحات

| المصطلح | المعنى |
|---|---|
| `Dashboard Summary` | مؤشرات المالك المجمعة. |
| `Renewals` | الاشتراكات اللي هتنتهي ومين جدّد. |
| `Collections` | الاشتراكات اللي عليها متبقي. |
| `Aging Bucket` | تقسيم المتأخرات حسب العمر. |
| `Churn` | عضو كان نشط وفقد الاشتراك. |
| `Renewal Rate` | نسبة التجديد من اللي كانوا هينتهوا. |
| `Timeline` | كل أحداث العضو في ترتيب زمني. |
| `ReportingQueryRepository` | استعلامات قراءة فقط — بدون Entities. |

---

## 3. النطاق

### داخل الـ MVP
- [x] `GET /dashboard/summary`.
- [x] `GET /dashboard/renewals`.
- [x] `GET /dashboard/collections`.
- [x] `GET /dashboard/attendance`.
- [x] `GET /dashboard/risk`.
- [x] `GET /dashboard/members/{id}/timeline`.
- [x] Branch Scope.
- [x] Date Range Validation.
- [x] Read-Only Projections.
- [x] Empty States.

### خارج الـ MVP
- [ ] Export Excel/PDF (P1).
- [ ] Scheduled Email Reports (P1).
- [ ] Cohort Analysis متقدم (P1).
- [ ] Holdout A/B Lift (P1).
- [ ] Custom Date Comparisons.
- [ ] Data Warehouse / BI.

---

## 4. الصلاحيات

| العملية | Owner | Manager | Receptionist | Trainer |
|---|:---:|:---:|:---:|:---:|
| Summary | ✅ | ❌ | ❌ | ❌ |
| Renewals | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Collections | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Attendance | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Risk Distribution | ✅ | ✅ (فرعه) | ❌ | ❌ |
| Member Timeline | ✅ | ✅ (فرعه) | ✅ (فرعه) | ❌ |

> **C-14:** Manager بيشوف Collections/Attendance لفرعه، لكن **مش** Org Revenue أو Summary التنفيذي.

---

## 5. القواعد الأساسية

| ID | القاعدة |
|---|---|
| BR-004 | Dashboard Org-scoped، والـ Owner يشوف كل الفروع. |
| BR-005 | At-Risk List فيها Score + Reason + Action + Draft. |
| BR-031 | Variance الورديات يظهر في Dashboard — **P1**. |
| C-14 | Manager ممنوع من Org Revenue. |
| REP-01 | كل استعلام لازم `org_id` — RLS + Explicit. |
| REP-02 | Manager → `branch_id` إلزامي من الـ Token. |
| REP-03 | Read-Only — ممنوع `INSERT`/`UPDATE`/`DELETE`. |
| REP-04 | ممنوع N+1 — Projections جاهزة. |
| REP-05 | Date Range أقصى 366 يوم. |
| REP-06 | كل المبالغ `Long` بالقروش. |
| REP-07 | التواريخ بـ `Africa/Cairo` والعرض UTC. |
| REP-08 | ممنوع Cache في MVP (C-07). |
| REP-09 | Empty State واضح — مفيش `null` بدل arrays. |
| REP-10 | ممنوع كشف بيانات عضو لمستخدم مش مخوّل. |

---

## 6. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `TenantContext`, `MoneyUtil`, `DateUtil` |
| `tenancy` | Read | `OrganizationQueryService`, `BranchQueryService` |
| `member` | Read | `MemberQueryService` (Timeline enrichment) |
| `plan` | Read | أسماء الباقات |
| `subscription` | Read | الاشتراكات والتجديدات |
| `payment` | Read | المدفوعات والتحصيل |
| `checkin` | Read | الحضور |
| `risk` | Read | توزيع المخاطر |
| `notification` | Read | الرسائل والـ Attribution |
| `audit` | Read | Timeline (اختياري) |

> **مفيش Feature تعتمد على `reporting`** — هو آخر واحد في الـ Build Order.
> `reporting` بيستخدم `ReportingQueryRepository` بـ SQL/JPQL Projections عبر الجداول — استثناء موثق لأنه Read-Only.

---

## 7. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`domain-design.md`](./domain-design.md) | الـ KPIs، الـ Formulas، الـ Queries. |
| [`implementation-plan.md`](./implementation-plan.md) | المهام. |
| [`api-design.md`](./api-design.md) | الـ Endpoints. |
| [`testing-plan.md`](./testing-plan.md) | الاختبارات. |

---

## 8. Definition of Done

- [ ] `ReportingQueryRepository` — Read-Only.
- [ ] Summary + Renewals + Collections.
- [ ] Attendance + Risk.
- [ ] Member Timeline.
- [ ] Branch Scope.
- [ ] Date Validation.
- [ ] مفيش N+1.
- [ ] P95 < 500ms لكل Endpoint.
- [ ] Empty States.
- [ ] Tests.

---

## 9. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| بداية الأسبوع إيه؟ | الاثنين 00:00 Cairo. |
| Churn محسوب إزاي؟ | Cohort بداية الشهر — تفاصيل في domain-design. |
| هل فيه Cache؟ | لا في MVP. |
| هل Export مطلوب؟ | P1. |
| هل Manager يشوف Revenue؟ | لا — Collections بس. |
| هل Timeline بيتقّص؟ | Pagination 20/صفحة. |
| هل نعرض أرقام سالبة؟ | لا — `max(0, ...)`. |
