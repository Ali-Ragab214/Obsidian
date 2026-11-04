# القرارات والافتراضات والتعارضات (Decisions, Assumptions & Conflicts)

> **المرجع:** [`../../Kammel_Project_Documentation/README.md`](../../Kammel_Project_Documentation/README.md) · [`../../0-doc.md`](../../0-doc.md) · [`../../1-doc2.md`](../../1-doc2.md)  
> **الهدف:** تسجيل كل قرار معماري، وكل تعارض بين الوثائق، وكل افتراض محتاج تأكيد — عشان التنفيذ يمشي بدون تخمين.

---

## 1. ترتيب المصادر (Source Priority)

لو حصل تعارض، الترتيب ده هو المرجع:

1. **`Kammel_Project_Documentation/`** — الوثائق التقنية المرجعية (الأحدث والأكثر تفصيلاً).
2. **`0-doc.md`** — وثيقة المنتج الأصلية (الرؤية والنطاق).
3. **`1-doc2.md`** — التقييم الهندسي (قيود وملاحظات على النطاق).
4. **`implementation-blueprint/`** — Blueprint سابق (مرجع مساعد، مش مصدر حقيقة).
5. **`implementation-plan/`** — الخطة دي (تفسير وتنظيم وتنفيذ).

> لو الخطة دي خالفت الوثائق، **الوثائق تكسب** إلا لو فيه قرار موثق هنا بسبب واضح.

---

## 2. التعارضات اللي تم حلها (Resolved Conflicts)

| # | الموضوع | التعارض | القرار | السبب |
|---|---|---|---|---|
| C-01 | مدة الـ MVP | `0-doc` قال 8 أسابيع، `1-doc2` قال غير واقعي، الوثائق التقنية قالت 12 | **12 أسبوع** | مطور واحد + Testing + Pilot حقيقي |
| C-02 | Cash Reconciliation | `0-doc` P0، الوثائق التقنية P1 | **P1 افتراضياً** | يترقى لـ P0 لو المقابلات أثبتت إنه الألم الأول |
| C-03 | Refunds | API Design وRoadmap فيها Refund، لكن MVP Exclusions قال P1 | **P1** | مش ضروري لأول 14 يوم Pilot |
| C-04 | `EXPIRING_SOON` | بعض الوثائق تعتبره Status، والتانية Computed | **Computed — مش متخزن** | يقلل حالات غير متسقة |
| C-05 | تمديد التجميد | `0-doc` قال يمتد تلقائياً، `08` قال عند فك التجميد بعدد الأيام الفعلية | **عند فك التجميد بعدد الأيام الفعلية** | أدق لو العضو رجع بدري |
| C-06 | `PLATFORM_ADMIN` | Security Matrix فيه Role، Domain Model مفيهوش | **مش موجود في MVP** | الـ `users` كلهم `org_id NOT NULL`؛ إنشاء الجيمات يتم بـ Seed/Admin CLI |
| C-07 | Redis | "ممكن بعدين" vs "ممنوع في MVP" | **ممنوع في MVP** | مش Needed لـ 500 عضو |
| C-08 | WhatsApp | Automation من أول يوم vs Manual | **Manual `wa.me` في P0** | تكلفة/موافقات/حظر |
| C-09 | RLS Migration | V14 واحدة في الآخر vs كل جدول من أول يوم | **RLS مع كل Migration من أول يوم** | مفيش فترة بدون حماية |
| C-10 | Audit Timing | Roadmap قال Week 7، لكن Subscription/Payment محتاجينه | **Audit Core في Phase 1، API في Phase 3** | مفيش عملية حساسة بدون Audit |
| C-11 | Renewal بعد الانتهاء | State Machine قال `EXPIRED → ACTIVE` ممنوع، لكن فيه Renewal بعد الانتهاء | **إنشاء Subscription جديد** بنفس الـ Member، والـ `/renew` بتعمل ده داخلياً | يحافظ على تاريخ الاشتراكات |
| C-12 | RLS و`risk_scores`/`subscription_events` | الجدولين في RLS list لكن مفيهم `org_id` | **إضافة `org_id` للجدولين** | Policy محتاجة العمود |
| C-13 | Idempotency Unique | Global Unique vs Per Org | **Unique `(org_id, idempotency_key)`** | عزل المستأجرين |
| C-14 | Manager Financial Access | Matrix قال ممنوع Financial Dashboards، لكن مسموح Cash Reconciliation | **Manager يشوف Collections/Cash لفرعه فقط، مش Org Revenue** | فصل واضح |
| C-15 | API Envelope | بعض الأمثلة Raw، والبعض `ApiResponse` | **`ApiResponse<T>` لكل النجاح، `ErrorResponse` للأخطاء** | توحيد العقود |
| C-16 | Audit Immutability vs Payment Reversal | Payments لازم Immutable، لكن Status بيتغير لـ `REVERSED` | **منع DELETE وكل الحقول المالية، والسماح بتغيير Status فقط عبر Service** | يحافظ على الـ Ledger |
| C-17 | Trainer Clients | مطلوب Trainer يشوف عملاءه، لكن مفيش علاقة في الـ Schema | **`member.assigned_trainer_id`** | أبسط حل للـ MVP |
| C-18 | Receipt Numbering | مطلوب Sequential، لكن مفيش Table للـ Sequence | **`document_sequences` + Row Lock** | يمنع التكرار تحت التزامن |
| C-19 | Login Identifier | الـ API بيعرض `username` فقط، لكن `(org_id, phone)` مش Global Unique | **`organizationSlug` + `username` + `password` في Login** | الـ Tenant لازم يتحدد قبل الـ JWT، ومنع Ambiguity بين جيمات |
| C-20 | Renewal Record | State Machine قال تمديد نفس الاشتراك، لكن ده بيكسر `price_minor_snapshot` لو السعر اتغير | **كل تجديد بينشئ `Subscription` جديد** — قبل الانتهاء يبدأ من `end_date`، وبعد الانتهاء من تاريخ الدفع | الحفاظ على Price Snapshot لكل فترة + Ledger نظيف |

---

## 3. القرارات المعمارية (Architecture Decisions)

### D-01: Feature-Based Modular Monolith
- **القرار:** تطبيق واحد Spring Boot، منظم حول Features، وكل Feature فيها `api / application / domain / infrastructure`.
- **السبب:** مطور واحد + Transactions بسيطة + حدود واضحة + إمكانية استخراج Service لاحقاً.
- **البديل المرفوض:** Microservices (تعقيد شبكة وDistributed Transactions) — Layered Packages (تشتيت التغيير).

### D-02: Shared Database + Shared Schema + RLS
- **القرار:** قاعدة واحدة، `org_id` في كل جدول، PostgreSQL RLS إجباري، و`SET LOCAL app.current_org`.
- **السبب:** أرخص وأبسط تشغيلياً، وRLS بيدي ضمان على مستوى الـ DB.
- **شرط:** التطبيق يتصل بـ `kammel_app` Non-Superuser، وممنوع `BYPASSRLS`.

### D-03: المبالغ `Long` بالقروش
- **القرار:** كل مبلغ `BIGINT`/`Long` بوحدة القرش. ممنوع `double`/`float`.
- **السبب:** دقة مالية. `BigDecimal` غير لازم لأن مفيش كسور أقل من قرش.

### D-04: Subscription State Machine بالأحداث
- **القرار:** `status` بيتغير فقط من خلال State Machine، وكل تغيير بيسجل `SubscriptionEvent` Immutable.
- **السبب:** Audit + منع التلاعب + تاريخ كامل.

### D-05: Price Snapshot
- **القرار:** `price_minor_snapshot` وقت إنشاء الاشتراك، ومايتغيرش.
- **السبب:** تغيير سعر الباقة مايأثرش على العقود القديمة.

### D-06: WhatsApp يدوي في P0
- **القرار:** `wa.me` Links + تأكيد إرسال يدوي. الـ Cloud API في P1.
- **السبب:** بدون رسوم/موافقات/حظر، ومعظم القيمة متحققة.

### D-07: Audit Core مبكر
- **القرار:** Audit Entity + Service + Migration في Phase 1، والاستخدام في كل Feature، والـ Query API في Phase 3.
- **السبب:** Freeze/Refund/Override محتاجين Audit من أول عملية.

### D-08: RLS مع كل Table
- **القرار:** دالة `enable_tenant_rls(table)` تتسمى في نفس Migration اللي بتنشئ الجدول.
- **السبب:** مفيش Window بدون حماية، وأسهل في الـ Review.

### D-09: الأحداث بين الـ Features
- **القرار:** Integration Events في `shared/contracts/events` كـ Records.
- **السبب:** فك الارتباط مع الحفاظ على Transactions. الأحداث الحرجة تتناول Synchronously داخل نفس الـ Transaction، والـ Side Effects بعد الـ Commit.

### D-10: لا Redis / لا Cache في MVP
- **القرار:** PostgreSQL + Indexes فقط. Rate Limiting في الذاكرة (Caffeine).
- **السبب:** تعقيد غير مبرر. Redis لما P95 > 500ms أو نحتاج Distributed Rate Limit.

### D-11: لا AI في MVP
- **القرار:** Risk Score قواعد فقط. ML/Spring AI في P2/P3.
- **السبب:** بيانات غير كافية + مخاطر Hallucination.

### D-12: Member ≠ User
- **القرار:** `User` موظف بيسجل دخول، `Member` عميل الجيم. Member ملهوش Login في MVP.
- **السبب:** Auth models مختلفة، وPWA في P1.

### D-13: Soft Delete للمembers فقط
- **القرار:** `deleted_at` للأعضاء. المدفوعات/الحضور/الأحداث ممنوع يتعملها Delete.
- **السبب:** الحفاظ على التاريخ المالي والتشغيلي.

### D-14: Member Code + QR Token
- **القرار:** إضافة `member_code` (Sequential per Org) و`qr_token` (UUID) للـ Member.
- **السبب:** البحث بالكود وQR الثابت — مذكورين في المتطلبات لكن مش في الـ Schema.

### D-15: Plan `grants_access`
- **القرار:** إضافة `grants_access BOOLEAN` للـ Plan. الـ PT Packages لا تمنح دخول.
- **السبب:** حل غموض تعدد الاشتراكات (Gym + PT).

### D-16: Member Portal Scope منفصل
- **القرار:** البوابة بتستخدم `scope = MEMBER` + `app.current_member` + Restrictive RLS، مش Role من أدوار الموظفين.
- **السبب:** `Member ≠ User` ([D-12](#d-12-member--user))، والفصل بيمنع أي تسريب بين الموظفين والأعضاء.
- **البديل المرفوض:** إضافة Role جديد `MEMBER` — كان هيدخل في RBAC الموظفين ويصعّب الـ Branch Scope.

### D-17: Magic Link Transactional
- **القرار:** `PORTAL_MAGIC_LINK` رسالة Transactional بتتجاوز Cooldown/Window.
- **السبب:** الرابط صالح 15 دقيقة — التأجيل بيلغيه.
- **البديل المرفوض:** Queue عادية — كانت هتأجل الرابط لليوم التالي.

---

## 4. الافتراضات (Assumptions)

> كل Assumption ليها قرار مؤقت، وطريقة تأكيد، وتأثير لو اتغيرت.

### ASM-01: مدة الـ MVP 12 أسبوع
- **الافتراض:** مطور واحد بدوام شبه كامل.
- **التأكيد:** تتبع الساعات أسبوعياً.
- **لو اتغير:** 16 أسبوع لو الواجهة الأمامية جديدة.

### ASM-02: Refunds وCash Shifts في P1
- **الافتراض:** أول 14 يوم مش محتاجين Refund/Shift.
- **التأكيد:** Phase 0 Interviews.
- **لو اتغير:** يتعملوا P0 ويقدموا في Phase 2.

### ASM-03: التجميد بيتحسب عند فك التجميد
- **الافتراض:** `frozen_at` + `scheduled_unfreeze_date`، والتمديد بعدد الأيام الفعلية.
- **التأكيد:** سيناريوهات Business في الـ Pilot.
- **لو اتغير:** التمديد يتعمل وقت التجميد بعدد الأيام المطلوبة.

### ASM-04: التجديد بعد الانتهاء = Subscription جديد
- **الافتراض:** الـ API `/renew` بتنشئ Record جديد لو الحالة `EXPIRED`.
- **التأكيد:** مراجعة محاسبية للـ Pilot.
- **لو اتغير:** يتعمل In-place Renewal مع الحفاظ على History.

### ASM-05: Member Code + QR Token
- **الافتراض:** `M-000123` وUUID للـ QR.
- **التأكيد:** احتياج الاستقبال الفعلي.
- **لو اتغير:** QR Dynamic في P1.

### ASM-06: Receipt Numbering
- **الافتراض:** `KM-{YYYY}-{000001}` لكل Org، باستخدام `document_sequences`.
- **التأكيد:** مراجعة صاحب الجيم.
- **لو اتغير:** Prefix مختلف أو Global Sequence.

### ASM-07: مفيش `PLATFORM_ADMIN` في MVP
- **الافتراض:** Provisioning بـ Seed Script / CLI.
- **التأكيد:** هل نحتاج Admin UI قبل أول 10 جيمات؟
- **لو اتغير:** يتضاف `platform` module في P1.

### ASM-08: `org_id` في `risk_scores` و`subscription_events`
- **الافتراض:** إضافة العمود للاتنين.
- **التأكيد:** مراجعة RLS Coverage Test.
- **لو اتغير:** Policy تعتمد على Join — أبطأ وأعقد.

### ASM-09: Audit Core في Phase 1
- **الافتراض:** Entity + Service + Migration فقط، بدون API.
- **التأكيد:** مراجعة الـ Phase 1 Checkpoint.
- **لو اتغير:** Audit يتأخر ويبقى فيه Gap أمني.

### ASM-10: API Envelope موحد
- **الافتراض:** `ApiResponse<T>` لكل النجاح.
- **التأكيد:** مراجعة الـ Frontend Contract.
- **لو اتغير:** Raw DTOs — مش مستحسن.

### ASM-11: Excel Import على مرحلتين
- **الافتراض:** `dryRun=true` للـ Preview، و`dryRun=false` للـ Commit في Transaction واحدة.
- **التأكيد:** تجربة على شيت حقيقي 400 عضو.
- **لو اتغير:** Import Batch Table + Resume.

### ASM-12: Member Status محسوب
- **الافتراض:** `INACTIVE` لما مفيش `ACTIVE`/`GRACE` subscription، بيتحدث بالـ Job أو Event.
- **التأكيد:** سلوك الـ Receptionist.
- **لو اتغير:** Status يدوي.

### ASM-13: Trainer Assignment
- **الافتراض:** `assigned_trainer_id` واحد لكل Member.
- **التأكيد:** هل العضو ليه أكتر من كابتن؟
- **لو اتغير:** Join Table `member_trainers`.

### ASM-14: Time-Restricted Plans — Data Only
- **الافتراض:** الحقل موجود، لكن الـ Enforcement (صباحي/مسائي/سيدات) مش في MVP.
- **التأكيد:** Phase 0 Interviews.
- **لو اتغير:** Check-in Time Validation في P1.

### ASM-15: Refresh Tokens في DB
- **الافتراض:** Table `refresh_tokens` بـ Hash + Rotation + Revocation.
- **التأكيد:** Security Review.
- **لو اتغير:** Stateless Refresh — أقل أماناً.

### ASM-16: `grants_access` للـ Plan
- **الافتراض:** PT Package لا يمنح دخول الجيم.
- **التأكيد:** سياسة الجيم في الـ Pilot.
- **لو اتغير:** كل الاشتراكات تمنح دخول.

### ASM-17: Risk Score لأعضاء عندهم ≥ 14 يوم بيانات
- **الافتراض:** أقل من كده → `INSUFFICIENT_DATA`.
- **التأكيد:** تحليل أول شهر Pilot.
- **لو اتغير:** Score بقواعد مبسطة.

### ASM-18: مفيش Modeling لإجازات/غلق الجيم
- **الافتراض:** الـ Risk مش هيعرف إن الجيم كان مقفول.
- **التأكيد:** ملاحظات الـ Pilot.
- **لو اتغير:** `gym_closures` Table + Signal.

### ASM-19: تعدد الاشتراكات
- **الافتراض:** مسموح أكتر من Subscription، لكن Check-in يختار واحد `grants_access = true` و`ACTIVE/GRACE`.
- **التأكيد:** Phase 0 Interviews.
- **لو اتغير:** قيد Unique على `(member_id)` للحالة النشطة.

### ASM-20: Timezone
- **الافتراض:** كل التخزين UTC، والعرض/الحسابات اليومية `Africa/Cairo`.
- **التأكيد:** إعداد الـ Branch Timezone.
- **لو اتغير:** دعم Timezones متعددة.

### ASM-21: Rate Limiting في الذاكرة
- **الافتراض:** Caffeine per-instance كافي في MVP.
- **التأكيد:** عدد الـ Instances.
- **لو اتغير:** Redis في P1.

### ASM-22: WhatsApp Opt-in
- **الافتراض:** يتسجل وقت التسجيل، ويتعدل يدوياً.
- **التأكيد:** مراجعة قانونية.
- **لو اتغير:** Opt-in Flow منفصل.

### ASM-23: مفيش SMS/OTP في MVP
- **الافتراض:** Staff Login بالـ Password فقط.
- **التأكيد:** أمان الحسابات.
- **لو اتغير:** 2FA في P1.

### ASM-24: Hosting
- **الافتراض:** VPS خارج مصر (مثلاً Hetzner) في البداية.
- **التأكيد:** مراجعة قانونية Law #151/2020.
- **لو اتغير:** استضافة محلية.

### ASM-25: Frontend
- **الافتراض:** React + RTL + Admin Template.
- **التأكيد:** مستوى الـ Frontend.
- **لو اتغير:** Thymeleaf للـ MVP.

### ASM-26: Organization Slug
- **الافتراض:** كل Org ليها `slug` فريد (مثال: `gold-gym-maadi`) وبيتستخدم في شاشة الدخول.
- **التأكيد:** تجربة الاستقبال في الـ Pilot — هل الـ Slug سهل يتقال؟
- **لو اتغير:** Login بـ Email Global Unique أو Subdomain لكل جيم.

### ASM-27: WhatsApp Delivery للـ Magic Link
- **الافتراض:** في P1، رابط الدخول بيتسلم عبر `wa.me` من طابور الاستقبال، أو Cloud API لو متاح.
- **التأكيد:** تجربة العضو في الـ Pilot — هل بيستنى الموظف يبعتله؟
- **لو اتغير:** تفعيل Cloud API إجباري في P1، أو OTP SMS كبديل.

---

## 5. الأسئلة المفتوحة (Open Questions)

| # | السؤال | التأثير | القرار المؤقت | متى يتأكد |
|---|---|---|---|---|
| Q-01 | هل الكاش أهم من الـ Churn؟ | ترتيب P0/P1 | Churn أولاً، Shift P1 | Phase 0 |
| Q-02 | هل فيه باقات صباحي/مسائي/سيدات؟ | Check-in Validation | Data only | Phase 0 |
| Q-03 | حد الخصم المسموح للـ Receptionist؟ | Discount Feature | مفيش Discount في MVP | Phase 0 |
| Q-04 | هل العضو يقدر يجمد أكتر من مرة؟ | Freeze Rules | حسب `freeze_max_days` | Phase 0 |
| Q-05 | هل العضو ليه أكتر من اشتراك نشط؟ | Check-in Selection | `grants_access` | Phase 0 |
| Q-06 | مدة الـ Cooldown بين الرسائل؟ | Notification Rules | 3 أيام | Pilot |
| Q-07 | الاستضافة داخل/خارج مصر؟ | Law/Latency | خارج مصر مؤقتاً | قبل الإطلاق |
| Q-08 | رسوم WhatsApp على مين؟ | Pricing | Pass-through | P1 |
| Q-09 | مين بيوافق على الـ Refund؟ | Permissions | Manager/Owner | P1 |
| Q-10 | هل نحتاج فاتورة ضريبية؟ | Out of Scope | لا | مراجعة قانونية |
| Q-11 | هل العضو يقدر يعمل Login لو رقمه مشترك؟ | Portal Identity | Unique `(org_id, phone)` — عضو واحد | P1 |
| Q-12 | هل نسمح للاستقبال يشوف Magic Link؟ | Privacy | أيوه — رابط Single-use 15 دقيقة + Audit | P1 |
| Q-13 | هل البوابة عربي فقط؟ | Scope | أيوه P1 | P1 |
| Q-14 | هل العضو يقدر يعدل بياناته؟ | Scope | لا — طلب تجديد فقط | P1 |

---

## 6. قواعد ثابتة لا تتفاوض (Non-Negotiables)

1. مفيش `double`/`float` في أي حساب مالي.
2. مفيش `DELETE` على `payments`, `subscription_events`, `audit_log`, `check_ins`.
3. مفيش Query على Tenant Data بدون `org_id` أو RLS Context.
4. مفيش `status` يتعدل يدوي — لازم State Machine.
5. مفيش `orgId` يتاخد من الـ Request Body — دايماً من الـ JWT/TenantContext.
6. مفيش PII في الـ Logs.
7. مفيش AI في الـ MVP.
8. مفيش Feature تخرج للـ Production بدون Tenant Isolation Test.

---

## 7. إزاي نحدّث الملف ده

- أي قرار جديد → أضف `D-xx` بالسبب والبديل المرفوض.
- أي تعارض جديد → أضف `C-xx` بالقرار.
- أي افتراض جديد → أضف `ASM-xx` بطريقة التأكيد.
- أي سؤال يتجاوب → انقله من Open Questions لقرار موثق.
- حدّث [requirements-traceability](./requirements-traceability.md) و[master-checklist](../development-roadmap/master-checklist.md) لو القرار أثر على النطاق.
