# Member Portal — تطبيق العضو PWA (P1)

> **الـ Phase:** Post-MVP · **الأولوية:** P1 · **الـ Package:** `com.kammel.features.memberportal`  
> **المرجع:** [`../../../0-doc.md`](../../../0-doc.md) (القسم 10) · [`../../../Kammel_Project_Documentation/01_BUSINESS_REQUIREMENTS.md`](../../../Kammel_Project_Documentation/01_BUSINESS_REQUIREMENTS.md)

---

## 1. الميزة دي بتعمل إيه؟

`member-portal` هي **واجهة العضو** (PWA) — مش تطبيق موظفين:

- العضو بيدخل برقم موبايله + **Magic Link** بيوصله على WhatsApp.
- يشوف **اشتراكه الحالي** (الحالة، البداية، النهاية، الأيام المتبقية، التجميد).
- يشوف **كود الدخول (QR)** بتاعه.
- يشوف **سجل حضوره** وسلسلة الأيام (Streak).
- يشوف **مدفوعاته** ويحمّل الإيصالات.
- يبعت **طلب تجديد** للاستقبال بضغطة.
- الواجهة **White-label**: اسم الجيم، اللوجو، واللون الأساسي.

**مش مسؤولة عن:**
- تسجيل دخول الموظفين → `identity`.
- إنشاء الأعضاء أو تعديل بياناتهم → `member`.
- إنشاء/تجديد الاشتراك فعليًا → `subscription`.
- تسجيل الدفع → `payment`.
- تسجيل الحضور → `checkin` (البوابة بتعرض بس).
- إرسال الرسائل → `notification` (البوابة بتطلب الإرسال).

---

## 2. ليه P1؟

- `0-doc` حددها P1 صريح: «تطبيق العضو (PWA) — P1».
- الـ MVP بيركّز على **الاستقبال والمالك**، والعضو لسه passive (رسائل + حضور من الاستقبال).
- الاعتماد على `subscription` + `payment` + `checkin` مكتملين الأول — البوابة **قارئة** لبياناتهم.
- الـ Schema من أول يوم فيها `members.qr_token` عشان الـ QR يشتغل من غير ترحيل مؤلم.

---

## 3. المصطلحات

| المصطلح | المعنى |
|---|---|
| `PWA` | Progressive Web App — بيتثبّت من رابط، بدون متجر. |
| `Magic Link` | رابط دخول مؤقت يُرسل على WhatsApp، بدل كلمة مرور أو OTP SMS. |
| `Portal Token` | توكن عشوائي 32-byte، بيتخزن كـ `SHA-256` فقط. |
| `Portal Session` | Access JWT قصير + Refresh Token طويل، منفصلين تمامًا عن جلسات الموظفين. |
| `Member Scope` | الـ JWT فيه `scope = MEMBER` و`member_id` — مفيش `role`. |
| `White-label` | البوابة بتظهر بهوية الجيم (`name`, `logo_url`, `primary_color`). |
| `Renewal Request` | طلب تجديد من العضو، بيظهر للاستقبال — **مش** تجديد فعلي. |
| `Streak` | عدد الأيام المتتالية اللي فيها حضور. |

---

## 4. النطاق

### داخل P1
- [x] طلب Magic Link بالـ `organizationSlug` + رقم الموبايل.
- [x] استلام الرابط على WhatsApp (عبر `notification`).
- [x] Verify Link → Portal Session.
- [x] Refresh + Logout.
- [x] `GET /portal/me` — بيانات العضو + هوية الجيم.
- [x] `GET /portal/subscription` — الحالي + السابق.
- [x] `GET /portal/qr` — كود ثابت + QR Image.
- [x] `GET /portal/checkins` — سجل الحضور + Streak.
- [x] `GET /portal/payments` + تحميل إيصال.
- [x] `POST /portal/renewal-requests` — طلب تجديد.
- [x] PWA Manifest + Service Worker (Cache Shell فقط).
- [x] Rate Limiting + Tenant/Member Isolation.

### خارج P1
- [ ] الدفع الأونلاين (Paymob) → P2.
- [ ] تعديل بيانات العضو من البوابة.
- [ ] تجميد/فك تجميد ذاتي.
- [ ] برنامج التمرين وتسجيل التمارين → P2.
- [ ] التحديات والشارات → P2.
- [ ] إشعارات Push.
- [ ] Multi-language (عربي فقط في P1).

---

## 5. القواعد

| ID | القاعدة |
|---|---|
| BR-039 | العضو بيشوف بياناته هو فقط — ممنوع أي `memberId` من الـ Request. |
| BR-040 | مفيش كلمة مرور للعضو — Magic Link فقط. |
| BR-041 | البوابة Read-only ما عدا طلب التجديد. |
| MP-01 | `POST /portal/auth/request-link` بيرجع `202` **دايمًا** — حتى لو الرقم مش موجود (منع Enumeration). |
| MP-02 | الـ Magic Link صالح **15 دقيقة**، Single-use، ويتخزن Hashed فقط. |
| MP-03 | الـ Refresh Token صالح **30 يوم** مع Rotation، والـ Access **30 دقيقة**. |
| MP-04 | Rate Limit: 3 طلبات / 15 دقيقة لكل رقم، و10 / ساعة لكل IP. |
| MP-05 | الإرسال بيحترم `whatsappOptIn` — لو `false` مفيش إرسال، ويتسجل `PORTAL_LINK_SUPPRESSED`. |
| MP-06 | Org `SUSPENDED/CLOSED` أو Member `INACTIVE`/محذوف → ممنوع الدخول وإلغاء الجلسات. |
| MP-07 | الـ Portal JWT مالوش أي صلاحية على `/api/v1/**` الخاصة بالموظفين. |
| MP-08 | الـ QR **ثابت** (`members.qr_token`) — البوابة بتعرضه ومبتغيّرهوش. |
| MP-09 | مفيش دفع أونلاين — طلب التجديد بيدخل Queue للاستقبال. |
| MP-10 | كل استعلامات البوابة `org_id` + `member_id` إجباريين (RLS + Repository Contract). |
| MP-11 | ممنوع تسجيل الـ Token أو الـ Magic Link في الـ Logs. |
| MP-12 | الطلب بيتم في `TenantContext` — الـ `orgId` بيتحدد من الـ Slug أو الـ Token، مش من الـ Body. |

---

## 6. الصلاحيات

البوابة **مش** جزء من RBAC الموظفين:

| الجهة | الوصول |
|---|---|
| Anonymous | `request-link` + `verify` + `refresh` فقط. Rate-limited. |
| Member (scope `MEMBER`) | `/portal/**` الخاصة به فقط. |
| Staff (أي Role) | ممنوع يستخدم Portal Token. |
| Member | ممنوع يستخدم أي Endpoint موظفين. |

---

## 7. الاعتماديات

| يعتمد على | النوع | الاستخدام |
|---|---|---|
| `shared` | Compile | `ApiResponse`, `TenantContext`, `JwtService`, `Clock` |
| `tenancy` | Compile | `OrganizationQueryService.findBySlug` + Branding |
| `identity` | Compile | `JwtService` + Security Filter Chain (Scope) |
| `member` | Compile | `MemberQueryService` (بيانات العضو + QR) |
| `subscription` | Read | `SubscriptionQueryService` (الحالي + التاريخ) |
| `payment` | Read | `PaymentQueryService` (المدفوعات + الإيصال) |
| `checkin` | Read | `CheckInQueryService` (السجل + Streak) |
| `notification` | Compile | `NotificationService.enqueueMagicLink` |
| `audit` | Compile | `AuditService.record` |

**مين بيعتمد على `member-portal`؟**
`reporting` (P1) — عدد طلبات التجديد وPortal Adoption. مفيش Feature تانية بتعتمد على الـ Domain.

---

## 8. خريطة الملفات

| الملف | المحتوى |
|---|---|
| [`README.md`](./README.md) | أنت هنا. |
| [`implementation-plan.md`](./implementation-plan.md) | Domain + API + Security + Testing + Tasks. |

> **ملاحظة:** لأن `member-portal` P1، الوثائق consolidated في ملفين بدل خمسة — زي `shift` و`lead`.

---

## 9. Definition of Done

- [ ] `V23__create_member_portal_tokens.sql` + RLS.
- [ ] `V24__create_portal_renewal_requests.sql` + RLS.
- [ ] `V25__add_organization_branding.sql` (SC-28).
- [ ] `MemberPortalToken` + `PortalRenewalRequest` Entities.
- [ ] Magic Link Flow (Request / Verify / Refresh / Logout).
- [ ] Portal JWT Scope + Filter Chain منفصلة.
- [ ] Member Isolation (Repository Contract + GUC + Tests).
- [ ] `/portal/me`, `/portal/subscription`, `/portal/qr`.
- [ ] `/portal/checkins` + Streak.
- [ ] `/portal/payments` + Receipt.
- [ ] `POST /portal/renewal-requests`.
- [ ] Branding + PWA Manifest.
- [ ] Rate Limit + Enumeration Protection.
- [ ] Audit للطلبات الحساسة.
- [ ] Tests (Unit + Integration + Isolation + E2E).

---

## 10. أسئلة مفتوحة

| السؤال | القرار المؤقت |
|---|---|
| Magic Link على WhatsApp إزاي؟ | عبر `notification` كـ `PORTAL_MAGIC_LINK` — `wa.me` مش مناسب للرابط الحساس. |
| لو `wa.me` مش بيسمح بإرسال رابط تلقائي؟ | P1: Receptionist يبعت الرابط من شاشة العضو بعد ما يظهر مرة واحدة (Owner/Manager). |
| هل الـ QR ثابت ولا متغير؟ | ثابت — `members.qr_token` (0-doc). |
| هل العضو يقدر يلغي الاشتراك؟ | لا — طلب تجديد فقط. |
| هل نسمح بتسجيل الدخول بـ OTP SMS؟ | لا في P1 — Magic Link أرخص وأسهل. |
| هل البوابة ليها دفع أونلاين؟ | لا — P2 مع Paymob. |
| هل نعرض سعر التجديد؟ | نعم — سعر الباقة الحالية كـ «ابتدائي» مع ملاحظة أنه يتأكد عند التجديد. |
