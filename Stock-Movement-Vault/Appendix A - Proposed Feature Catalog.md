---
title: "Appendix A: Proposed Feature Catalog"
cssclasses: [rtl]
tags: [appendix, features, priorities, proposed]
section: A
---

# Appendix A: Proposed Feature Catalog
⬅️ [[17 - Future Enhancements]] | ➡️ [[Appendix B - Engineering Risk Register]]

الـ features دي مختارة **للقيمة التجارية**، مش للجدّة التقنية. كل feature فيها: المشكلة، قيمة المستخدم، طريقة التنفيذ، الـ entities المطلوبة، الـ APIs، الـ dependencies، التعقيد، والأولوية.

> [!info] معنى الأولويات
> - 🔴 **Must Have** — مطلوب لـ MVP موثوق.
> - 🟠 **Should Have** — قيمة عالية بعد الـ MVP.
> - 🟢 **Nice to Have** — قيّم لقطاعات معينة.

---

## F1. End-to-end tenant isolation — 🔴 Must Have

- **المشكلة:** الـ SaaS vendor **مايقدرش يخاطر** إن شركة تشوف مخزون شركة تانية؛ ده **متطلب ثقة وقانوني**.
- **قيمة المستخدم:** تشغيل multi-tenant آمن؛ مشتري الـ enterprise يقدر يعدّي **security review**.
- **الطريقة:** اختار **schema-per-tenant** أو **shared-schema + RLS**؛ نفّذ resolver/provider أو predicates إجبارية؛ **fail closed**؛ ضيف isolation tests؛ سجّل وصول الـ platform في audit.
- **Entities:** `tenants`/`users` الموجودة؛ مفيش business entity جديدة، لكن `tenant_id` لو اتختار الـ shared model.
- **APIs:** مفيش API جديد للمستخدم؛ بيأثر على **كل endpoint**.
- **Dependencies:** common migration، authentication.
- **التعقيد:** High.
- **الأولوية:** 🔴 **Must Have — بيمنع كل حاجة تانية.**

## F2. Immutable stock ledger and balance projection — 🔴 Must Have

- **المشكلة:** كميات المخزون **غير موثوقة وغير قابلة للـ audit**؛ التصحيحات بتكتب فوق التاريخ.
- **قيمة المستخدم:** balances موثوقة، تاريخ قابل للشرح، تصحيحات آمنة.
- **الطريقة:** **append-only** movement + lines؛ balance row لكل warehouse/product؛ posting transactional؛ **reversals بدل التعديل**؛ reconciliation job.
- **Entities:** `stock_movements`, `stock_movement_lines`, `stock_balances`.
- **APIs:** `POST/GET /stock-movements`, `POST /stock-movements/{id}/reverse`, `GET /stock-balances`.
- **Dependencies:** warehouses, products, tenant isolation.
- **التعقيد:** High.
- **الأولوية:** 🔴 Must Have.

## F3. Warehouses and warehouse-to-warehouse transfers — 🔴 Must Have

- **المشكلة:** المخزون متخزّن في أماكن متعددة، والـ transfers بتتتبّع بشكل غير رسمي.
- **قيمة المستخدم:** مخزون دقيق لكل موقع وـ replenishment مُتحكَّم فيه بين المواقع.
- **الطريقة:** **warehouse entity**؛ ساقين ledger مربوطين في transaction واحدة؛ **ترتيب lock حتمي**؛ approval/in-transit اختياري.
- **Entities:** `warehouses`، وحقول transfer على الحركات أو `stock_transfers`.
- **APIs:** `GET/POST/PUT /warehouses`, `POST /stock-transfers`.
- **Dependencies:** F2، tenant isolation.
- **التعقيد:** Medium–High.
- **الأولوية:** 🔴 Must Have للشركات متعددة المواقع؛ 🟠 Should Have للموقع الواحد.

## F4. Authentication, RBAC, and team management — 🔴 Must Have

- **المشكلة:** محدش يقدر يعمل login النهاردة؛ الـ roles غير متفرضة؛ مستخدمو الـ tenant ممكن يصعّدوا صلاحياتهم.
- **قيمة المستخدم:** وصول آمن وعمليات بأقل صلاحيات (**least-privilege**).
- **الطريقة:** login/refresh/logout؛ access tokens قصيرة؛ فحص حالة tenant/user؛ permission matrix؛ invitations؛ audit.
- **Entities:** users, roles/permissions, invitations, refresh tokens/token versions.
- **APIs:** `/auth/*`, `/users/*`, `/invitations`.
- **Dependencies:** common migration، tenant isolation.
- **التعقيد:** High.
- **الأولوية:** 🔴 Must Have.

## F5. Reorder points and low-stock alerts — 🟠 Should Have

- **المشكلة:** الـ stockouts بتتكتشف متأخر؛ المديرين بيراقبوا spreadsheets يدويًا.
- **قيمة المستخدم:** replenishment استباقي ومبيعات ضايعة أقل.
- **الطريقة:** reorder policy لكل warehouse/product؛ تقييم مجدول؛ تنبيهات داخل التطبيق/email؛ reorder point مبني على forecast اختياريًا.
- **Entities:** `reorder_policies`, alerts/notifications.
- **APIs:** `GET/PUT /reorder-policies`, `GET /alerts/low-stock`.
- **Dependencies:** F2، F3، آلية إشعارات.
- **التعقيد:** Medium.
- **الأولوية:** 🟠 Should Have (قيمة عالية).

## F6. Stock reservations — 🟠 Should Have

- **المشكلة:** المخزون الملتزم بيتباع مرتين أو بيتخصص للأوردر الغلط.
- **قيمة المستخدم:** **available-to-promise** موثوق وبيع زيادة أقل.
- **الطريقة:** صفوف reservation بـ expiry/status؛ `available = on-hand − active reservations`؛ release عند التنفيذ/الإلغاء؛ expiry job.
- **Entities:** `reservations`.
- **APIs:** `POST/GET/DELETE /reservations`.
- **Dependencies:** F2، F3، تكامل order/customer (أو reference عام).
- **التعقيد:** Medium–High.
- **الأولوية:** 🟠 Should Have للشركات المبنية على الأوردرات.

## F7. Immutable audit trail and activity log — 🟠 Should Have

- **المشكلة:** التحقيقات والامتثال محتاجين معرفة **مين غيّر إيه ومتى وليه**.
- **قيمة المستخدم:** مساءلة، استجابة أسرع للحوادث، جاهزية للـ audit.
- **الطريقة:** جدول audit **append-only**؛ سجّل actor و tenant و action و entity و before/after و correlation id؛ قيّد الوصول؛ retention policy.
- **Entities:** `audit_log` (والـ ledger كـ inventory audit).
- **APIs:** `GET /audit-events` (بصلاحيات مقيّدة).
- **Dependencies:** auth، F2.
- **التعقيد:** Medium.
- **الأولوية:** 🟠 Should Have.

## F8. Inventory reporting and exports — 🟠 Should Have

- **المشكلة:** المديرين مايقدروش يجاوبوا أسئلة أساسية من غير شغل spreadsheets يدوي.
- **قيمة المستخدم:** قرارات أسرع، تقارير قابلة للمشاركة، مطابقة مع المالية.
- **الطريقة:** reporting views منسّقة؛ تعريفات metrics؛ APIs بفلترة وpagination؛ export CSV/Excel/PDF **async**؛ read-only datasource.
- **Entities:** reporting views/marts؛ snapshots اختيارية.
- **APIs:** `/reports/stock-summary`, `/reports/movements`, `/reports/turnover`, `/reports/valuation`, `/reports/export`.
- **Dependencies:** F2، F3، بيانات التكلفة للـ valuation.
- **التعقيد:** Medium–High.
- **الأولوية:** 🟠 Should Have.

## F9. Inventory valuation — 🟢 Nice to Have (حسب القطاع)

- **المشكلة:** المالية محتاجة **قيمة** المخزون مش الكميات بس؛ الهوامش ورأس المال الزايد مش ظاهرين.
- **قيمة المستخدم:** valuation دقيقة، تحكّم في التكلفة، قرارات شراء أحسن.
- **الطريقة:** سجّل unit cost عند الاستلام؛ **FIFO layers** أو weighted average؛ snapshots دورية؛ مطابقة مع الـ ledger.
- **Entities:** cost layers أو `inventory_valuations`، وحقول cost على movement lines.
- **APIs:** `GET /reports/valuation`, `GET /products/{id}/valuation`.
- **Dependencies:** F2، بيانات التكلفة، سياسة المحاسبة.
- **التعقيد:** High.
- **الأولوية:** 🟢 Nice to Have مبدئيًا؛ جدوله في Phase 3 **بعد** التحقق من use cases المالية.

## F10. Tenant-safe AI assistant and forecasting — 🟢 Nice to Have (Phase 4)

- **المشكلة:** المستخدمين بيتعبوا في الاستعلام عن البيانات؛ الـ replenishment reactive.
- **قيمة المستخدم:** إجابات سريعة، توصيات استباقية، تحليل يدوي أقل.
- **الطريقة:** **read-only tool calling** فوق services حتمية؛ NL reporting عبر semantic layer؛ forecasting إحصائي؛ anomalies بقواعد؛ **LLM للشرح فقط**.
- **Entities:** `ai_conversations`, `ai_audit`, `demand_forecasts`, `anomaly_flags`, `recommendations`.
- **APIs:** `/ai/*` (شوف [[11 - AI Capabilities]] §11.12).
- **Dependencies:** F1، F2، F7، F8، تاريخ طلب نضيف.
- **التعقيد:** High.
- **الأولوية:** 🟢 Nice to Have دلوقتي؛ 🟠 Should Have لما النواة تستقر وتتميّز.

---

## ➕ Features إضافية (ملخص)

| Feature | المشكلة التجارية | الطريقة | التعقيد | الأولوية |
|---|---|---|---|---|
| **Batch/lot و serial tracking** | Traceability والـ recalls | جداول lot/serial مربوطة بالحركات؛ FEFO | High | 🟠 Should Have للقطاعات المنظّمة |
| **Expiry management** | الفساد والامتثال | Expiry على الـ lots؛ تنبيهات؛ FEFO picking | Medium–High | 🟠 Should Have للأغذية/الأدوية |
| **Cycle counting** | مخزون النظام بيبعد عن الواقع | جلسات جرد، موافقة على الفروقات، adjustment postings | Medium | 🟠 Should Have |
| **Subscription plans و usage limits** | تحقيق الدخل وضبط التكلفة | Plans و entitlements و metering و billing webhooks | High | 🟠 Should Have للإطلاق التجاري |
| **Outbox و notifications** | Side effects غير موثوقة | Transactional outbox + relay + email/webhook | Medium–High | 🟠 Should Have |
| **Supplier / purchase orders** | تنفيذ الـ replenishment | Suppliers و POs و goods receipt و matching | High | 🟢 Nice to Have |
| **Barcode / mobile workflows** | إدخال بطيء وعرضة للأخطاء | Barcode scanning وـ mobile UI/API | Medium | 🟢 Nice to Have |
| **SSO / SAML / SCIM** | هوية الـ enterprise | تكامل identity provider | High | 🟢 Nice to Have |
| **Multi-currency** | العمليات العالمية | Currency على التكاليف/الأسعار، FX rates | High | 🟢 Nice to Have |
| **Data warehouse** | Product analytics عبر الـ tenants | ETL/ELT بضوابط خصوصية | High | 🟢 Nice to Have |

---
⬅️ [[17 - Future Enhancements]] | ➡️ [[Appendix B - Engineering Risk Register]]
