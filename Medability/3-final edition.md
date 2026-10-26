# منصة توافر الأدوية — Project Blueprint

_أُعدّ لـ علي رجب · 24 أكتوبر 2026 · خلاصة وتحليل ملفين: `1-Details.pdf` (الفكرة والمقترح التقني الأول) و`2-claude.pdf` (مراجعة السوق والتعديلات والـ schema)_

## 1. الفكرة في سطرين

منصة بتخلّي المريض يلاقي الدواء الناقص أو المقطوع في أقرب صيدلية بتملكه فعلًا، أو بديله العلمي المطابق، في ثواني بدل ما يلف على 10–15 صيدلية أو يكلّمهم واحدة واحدة. المنصة بتربط المرضى بالصيدليات (C2B) والصيدليات ببعضها (B2B)، وهتتبني بـ Spring Boot 3 كـ modular monolith.

- **الاسم:** لسه مفيش اسم متفق عليه (مؤقتًا: Pharmacy Availability Platform).
- **نوع المشروع:** Portfolio مؤثر (impact-driven)، مش startup جاهز للسوق.

## 2. المشكلة اللي بيحلها

1. **نقص الأدوية وتعب البحث:** لما دواء الضغط أو السكر أو الأمراض المزمنة أو الأدوية الحيوية بيبقى ناقص، المريض أو قريبه بيلف أو بيتصل بصيدلية ورا التانية. ده بياكل وقت وأعصاب، وممكن يعرّض حياة المريض للخطر لو الوقت متأخر. السبب الجذري حسب الملف التاني: أزمة العملة الأجنبية.
2. **مفيش قاعدة موحدة للبدائل:** الدواء له مادة فعالة واحدة (Active Ingredient) وأكتر من اسم تجاري. المريض بيرفض البديل لأنه مايعرفوش، والصيدلي مش دايمًا فاضي يشرح.
3. **الصيدليات تقليدية:** مش كل صيدلية عندها API أو سيستم POS، فمفيش مصدر موحد لحالة المخزون.
4. **قيمة للصيدلية كمان:** بتصرّف الأدوية الراكنة عندها وبتوصل لمرضى جداد.

> **ملاحظة جوهرية:** مؤسسة شفاء بتقول إن المشكلة مش عدد الصيدليات، المشكلة إن مخزونها مش متساوي. يعني المشروع هيعيش أو يموت على **جودة بيانات المخزون**، مش على Elasticsearch ولا على الـ AI.

## 3. السوق والمنافسين

_(الأرقام دي من الملف التاني، راجعها قبل ما تذكرها في إنترفيو.)_

- هيئة الدواء عندها خدمة على موقعها وخط ساخن **15301** للاستعلام عن توافر الأدوية، يعني المشكلة كبيرة لدرجة إن الدولة اتدخلت.
- **شفاء (Chefaa)** أكبر صيدلية إلكترونية في مصر: المستخدم بيكتب اسم الدواء أو يرفع الروشتة وبتوصله بصيدليات قريبة عندها الدواء.
- في 2018 كان فيه 19 تطبيق توصيل أدوية تانيين.
- **الخلاصة:** المشكلة حقيقية لكن الفكرة مش جديدة والسوق زحمة. كمشروع تعلّم و CV ممتاز، وكـ startup صعب. في الإنترفيو قول إنك عارف المنافسين.

## 4. الفرق بين الملفين وإيه اللي اتعتمد

الملف الأول متحمس ومقترحاته التقنية أثقل (Elasticsearch وAI من البداية)، والملف التاني بيراجعها بشكل أكثر واقعية. الجدول بيلخّص التعارضات والقرار المعتمد في الـ Blueprint (قابل للتعديل):

|   |   |   |   |
|---|---|---|---|
|الموضوع|الملف 1 (Details)|الملف 2 (مراجعة)|المعتمد|
|البحث|Elasticsearch من البداية (Fuzzy + Edge NGram)|Postgres بس في الـ MVP (full-text + pg_trgm + unaccent)، وES بعد benchmark|**Postgres أولًا**|
|البدائل|Spring AI Agent + LLM|SQL حتمي بنفس المادة والتركيز والشكل الدوائي، لأن الـ LLM ممكن يهلوس|**SQL حتمي**|
|دور الـ AI|في قلب النظام|اختياري وآخر مرحلة (OCR) مع guardrails|**اختياري**|
|Redis|Cache لنتائج البحث والقوائم|للكتالوج والبدائل بس، وممنوع للتوافر|**كتالوج + rate limiting**|
|تحديث المخزون|CSV/Excel + API للـ POS|3 مصادر: رفع مخزون + Request broadcast + بلاغات مستخدمين|**المصادر التلاتة + TTL**|
|المعمارية|موديولات منفصلة|Monolith modular (Spring Modulith)|**Modular monolith**|
|الجغرافيا|PostGIS + ST_DWithin أو Haversine|geography(Point,4326) + GiST|**geography + ST_DWithin**|

> **سبب الترجيح:** كتالوج الأدوية صغير نسبيًا فمش محتاج Elasticsearch من اليوم الأول، والغلط في الدواء مكلف، والثقة بتضيع لو قلنا «متوفر» والدواء خلص من ساعة.

## 5. النطاق (Scope)

**داخل الـ MVP:** كتالوج أدوية + بدائل، بحث عربي/إنجليزي بيتحمّل الأخطاء الإملائية، بحث جغرافي عن الصيدليات، تسجيل وتوثيق الصيدليات، تحديث المخزون (CSV + يدوي) مع TTL، Availability Request للصيدليات القريبة، بلاغات المستخدمين، JWT + أدوار.

**بعد الـ MVP (اختياري):** Elasticsearch بعد benchmark، AI/OCR للروشتة أو العلبة، تكامل POS لحظي (أصعب جزء، وأغلب الصيدليات مش هتلتزم بيه).

**خارج النطاق:** التوصيل والدفع، أي قرار أو نصيحة طبية، عرض الأدوية المخدرة والمضبوطة.

## 6. الأدوار (Roles)

الملف الأول ذكر «صيدلي، مسؤول صيدلية، إدمن»، والتاني اختصرها في 3 أدوار:

|   |   |
|---|---|
|الدور|بيعمل إيه|
|`PATIENT`|يبحث عن دواء، يشوف البدائل والصيدليات القريبة، يبعت Availability Request، يبلّغ «لقيته هنا»|
|`PHARMACY_STAFF`|يسجّل الصيدلية، يرفع المخزون (CSV أو يدوي)، يرد على الطلبات (متوفر / لأ / بديل)|
|`ADMIN`|يوثّق الصيدليات، يستورد ويراجع الكتالوج، يتابع الـ import jobs|

## 7. المعمارية والـ Tech Stack

**Modular monolith (Spring Modulith)** بدل microservices: أبسط في الـ MVP وأسهل في الشرح. الموديولات:

- `identity`: تسجيل/دخول، JWT، الأدوار.
- `catalog`: الأدوية، المواد الفعالة، الشركات، البدائل.
- `search`: بحث الأدوية (Postgres أولًا، وadapter لـ ES اختياري).
- `pharmacy`: بيانات الصيدلية، التوثيق، البحث الجغرافي.
- `availability`: المخزون، import jobs، requests/responses، reports، الـ TTL.
- `notification`: إشعار الصيدليات بالطلبات (القناة لسه مش محددة).
- `ai` (اختياري): OCR + guardrails.

|                 |                                            |                         |
| --------------- | ------------------------------------------ | ----------------------- |
| الطبقة          | التقنية                                    | ملاحظة                  |
| Backend         | Spring Boot 3 + Spring Modulith            |                         |
| Security        | Spring Security + JWT                      | Role-based access       |
| Database        | PostgreSQL + PostGIS (+ pg_trgm, unaccent) | `geography(Point,4326)` |
| Migrations      | Flyway                                     |                         |
| Batch           | Spring Batch                               | استيراد CSV/Excel       |
| Cache           | Redis                                      | كتالوج + rate limiting  |
| Search (لاحقًا) | Elasticsearch                              | بعد benchmark           |
| AI (اختياري)    | Spring AI                                  | OCR بس                  |
| Infra/Testing   | Docker Compose, Testcontainers, CI         |                         |

## 8. تفاصيل كل موديول

### 8.1 الكتالوج والبدائل

- `formulation_key` هو hash لمجموعة مرتبة من (المواد الفعالة + التركيزات + الشكل الدوائي). **كل الأدوية اللي ليها نفس المفتاح بدائل لبعض.**
- بيتحسب وقت الاستيراد ويتفهرس بـ B-tree، فالبدائل بقت query بسيط من غير LLM.
- `drug_ingredient` جدول ربط لأن فيه أدوية بأكتر من مادة فعالة.
- نفس المادة بتركيز مختلف أو شكل دوائي مختلف (أقراص مقابل شراب) مش بديل. وطبّع الوحدات (mg/g) قبل حساب الـ hash.
- استبعد `is_controlled = true` من النتائج.

```sql
SELECT d2.*
FROM drug d1
JOIN drug d2 ON d2.formulation_key = d1.formulation_key AND d2.id <> d1.id
WHERE d1.id = :drugId AND d2.is_controlled = false;
```

### 8.2 البحث عن الأدوية

- المرحلة الأولى من غير ES: Postgres full-text + `pg_trgm` (GIN index) + `unaccent` على الاسم العربي والإنجليزي والمادة الفعالة والشركة.
- **تطبيع عربي** (من تحدي «همزات اللغة العربية» في الملف الأول): أ/إ/آ ← ا، ى ← ي، ة ← ه، وشيل التشكيل والتطويل. طبّق نفس التطبيع على اللي بيتخزن وعلى كلمة البحث.
- لازم يتحمّل غلطة حرف أو حرفين، ويقبل إدخال عربي ويرجّع الاسم الإنجليزي (كل دواء مخزّن بالاسمين).
- لاحقًا لو الـ benchmark برّر: Index في Elasticsearch فيه الاسم العربي والإنجليزي والمادة الفعالة والشركة، مع Fuzzy matching وEdge NGram tokenizer.

```sql
SELECT id, trade_name_en, trade_name_ar,
       GREATEST(similarity(trade_name_en, :q), similarity(trade_name_ar, :q)) AS score
FROM drug
WHERE trade_name_en % :q OR trade_name_ar % :q
ORDER BY score DESC
LIMIT 10;
```

### 8.3 البحث الجغرافي

- `pharmacy.location` نوعه `geography(Point,4326)` بدل geometry عشان المسافات تطلع بالمتر، مع GiST index.
- انتبه: `ST_MakePoint` بياخد (lng, lat) بالترتيب ده.
- نطاق البحث الافتراضي 3 أو 5 كم، والنتايج مرتبة بالأقرب.
- **ترتيب العرض:** الأقرب اللي عنده الدواء الأصلي الأول، وبعده الأقرب اللي عنده البديل مع توضيح إنه بديل.

```sql
SELECT p.id, p.name,
       ST_Distance(p.location, pt.g) AS distance_m,
       s.status, s.updated_at,
       (s.expires_at > now()) AS is_confirmed
FROM pharmacy p
JOIN pharmacy_stock s ON s.pharmacy_id = p.id
CROSS JOIN (SELECT ST_SetSRID(ST_MakePoint(:lng, :lat), 4326)::geography AS g) pt
WHERE s.drug_id = :drugId
  AND s.status <> 'OUT'
  AND p.verification_status = 'VERIFIED'
  AND ST_DWithin(p.location, pt.g, :radiusM)
ORDER BY distance_m;
```

_(مثال توضيحي، بيتضبط في التنفيذ.)_

### 8.4 التوافر والمخزون (قلب المشروع)

القاعدة: **ماتعتمدش على مصدر واحد.** فيه 3 مصادر:

1. **مخزون مرفوع من الصيدلية** (CSV أو API) مع `updated_at` و`expires_at` (TTL). بعد المدة بيتحول لحالة «غير مؤكد».
2. **Request broadcast:** المريض يطلب دواء، فالصيدليات القريبة يوصلها إشعار وترد «متوفر / لأ» (أو ببديل) في دقايق. بيشتغل من غير ما الصيدلية تحافظ على مخزون دقيق طول الوقت.
3. **بلاغات المستخدمين** («لقيته هنا»): بوزن أقل وصلاحية قصيرة.

قواعد العرض: اعرض دايمًا «آخر تأكيد من كذا دقيقة»، و**ماتكاشّيش حالة التوافر**. الحالات: `AVAILABLE / LOW / OUT`، والمصادر: `CSV / API / MANUAL / REQUEST_REPLY`.

### 8.5 استيراد المخزون (Spring Batch)

- الصيدلية ترفع CSV/Excel يوميًا أو أسبوعيًا.
- الـ Job: reader ← validation ← upsert في `pharmacy_stock` (الـ PK هو pharmacy_id + drug_id).
- كل تشغيل يتسجل في `stock_import_job` (rows_total, rows_failed, status, started/finished) عشان الصيدلية تشوف الصفوف الفاشلة.

### 8.6 الأمان

- JWT + role-based access، وموظف الصيدلية يعدّل مخزون صيدليته بس (بالتحقق من `pharmacy_staff`).
- الصيدلية ماتظهرش في النتايج إلا بعد `verification_status = VERIFIED` (اقتراح).
- Rate limiting على الـ endpoints العامة.
- موقع المريض وتليفونه بيانات حساسة (قانون حماية البيانات الشخصية).

### 8.7 الـ Caching (Redis)

للكتالوج والبدائل بس (بيانات ثابتة) لتخفيف الضغط على الـ DB، وللـ rate limiting. **ممنوع** cache لحالة التوافر.

### 8.8 الـ AI (اختياري، آخر مرحلة)

- الاستخدام الوحيد المقبول: OCR لصورة الروشتة أو علبة الدواء لاستخراج اسم الدواء، وبعدها البحث والبدائل يكملوا بشكل حتمي.
- الـ LLM ماياخدش أي قرار طبي ولا يختار بدائل.
- لو الـ AI دخل، الـ System Prompt الصارم المقترح في الملف الأول:

> «أنت مساعد ذكي لتوفير معلومات الأدوية. دورك فقط عرض البدائل المسجلة رسميًا التي لها نفس المادة الفعالة، ويجب إرفاق تنبيه طبي واضح بأن أي تبديل للدواء يجب أن يكون بمراجعة الطبيب المعالج أو الصيدلي.»

## 9. الـ Database Schema (PostgreSQL + PostGIS)

المسودة من الملف التاني. الأنواع والـ constraints هتتكتب في Flyway migration.

**الكتالوج**

```
active_ingredient (id, name_en, name_ar)
manufacturer (id, name_en, name_ar)
drug (id, trade_name_en, trade_name_ar, manufacturer_id, dosage_form, pack_size,
      official_price, is_controlled, formulation_key)
drug_ingredient (drug_id, ingredient_id, strength_value, strength_unit)
```

**المستخدمين والصيدليات**

```
app_user (id, phone/email, password_hash, role, created_at)
pharmacy (id, name, license_no, phone, address, governorate, area,
          location geography(Point,4326), opening_hours jsonb, is_24h, verification_status)
pharmacy_staff (user_id, pharmacy_id)
```

**التوافر**

```
pharmacy_stock (pharmacy_id, drug_id, status [AVAILABLE/LOW/OUT], quantity?,
                source [CSV/API/MANUAL/REQUEST_REPLY], updated_at, expires_at)  -- PK (pharmacy_id, drug_id)
stock_import_job (id, pharmacy_id, file_name, status, rows_total, rows_failed, started_at, finished_at)
availability_request (id, patient_id, drug_id, location, radius_m, status, created_at, expires_at)
availability_response (id, request_id, pharmacy_id, drug_id, status, responded_at)  -- drug_id ممكن يبقى بديل
availability_report (id, user_id, pharmacy_id, drug_id, found boolean, created_at)
```

**Indexes الأساسية**

- GiST على `pharmacy.location`.
- GIN trigram على أسماء الأدوية.
- B-tree على `drug.formulation_key` وعلى `active_ingredient`.
- Partial index على `pharmacy_stock(drug_id) WHERE status <> 'OUT'`.

## 10. الـ API Endpoints (مقترح)

_مش مذكورة في الملفين، اتشتقت من الـ features وهتتظبط في التنفيذ. علامة (_) يعني متوقف على قرار إن البحث متاح للزوار.*

|   |   |   |   |
|---|---|---|---|
|Method|Path|الدور|الوظيفة|
|POST|`/api/auth/register`, `/api/auth/login`|عام|تسجيل ودخول (JWT)|
|GET|`/api/drugs/search?q=`|عام*|بحث الأدوية|
|GET|`/api/drugs/{id}/alternatives`|عام*|البدائل المطابقة|
|GET|`/api/availability?drugId&lat&lng&radius`|PATIENT|الصيدليات القريبة اللي عندها الدواء|
|POST|`/api/availability/requests`|PATIENT|طلب توافر (broadcast)|
|GET|`/api/availability/requests/{id}`|PATIENT|ردود الصيدليات|
|POST|`/api/availability/reports`|PATIENT|بلاغ «لقيته هنا»|
|POST|`/api/pharmacies`|PHARMACY_STAFF|تسجيل صيدلية|
|POST|`/api/pharmacies/{id}/stock/import`|PHARMACY_STAFF|رفع CSV/Excel|
|GET|`/api/pharmacies/{id}/stock/import-jobs/{jobId}`|PHARMACY_STAFF|حالة الـ import والصفوف الفاشلة|
|PUT|`/api/pharmacies/{id}/stock/{drugId}`|PHARMACY_STAFF|تحديث يدوي|
|GET|`/api/pharmacy/requests`|PHARMACY_STAFF|الطلبات الواردة|
|POST|`/api/pharmacy/requests/{id}/respond`|PHARMACY_STAFF|رد: متوفر / لأ / بديل|
|PATCH|`/api/admin/pharmacies/{id}/verify`|ADMIN|توثيق صيدلية|
|POST|`/api/admin/catalog/import`|ADMIN|استيراد كتالوج الأدوية|

## 11. خطة التنفيذ

|   |   |
|---|---|
|المرحلة|الشغل|
|**0. التحضير** (أيام)|تحديد حدود الـ MVP، إيجاد dataset للأدوية المصرية (راجع ترخيصه)، رسم الـ ERD|
|**1. الأساس**|Spring Boot، Flyway، Docker Compose (Postgres+PostGIS)، Spring Security + JWT بالأدوار التلاتة|
|**2. الكتالوج**|استيراد الأدوية، البحث (full-text + trigram عربي/إنجليزي)، endpoint البدائل|
|**3. الصيدليات**|تسجيل وتوثيق الصيدلية، البحث الجغرافي بالمسافة|
|**4. التوافر**|استيراد المخزون بـ Spring Batch، availability requests، الـ TTL وحالة «غير مؤكد»|
|**5. الجودة**|Redis للكتالوج، rate limiting، Testcontainers، CI، logging وmetrics|
|**6. اختياري**|Elasticsearch لو الـ benchmark برّره، AI/OCR بـ guardrails صارمة|

حسب الملف التاني، المراحل 2 و3 و4 مستقلة نسبيًا بعد المرحلة 1 فلو في فريق تقدروا توزعوها (مع إن المرحلة 4 محتاجة جداول الكتالوج والصيدليات).

## 12. السلامة والقانون والجودة

- **ماتعرضش الأدوية المخدرة والمضبوطة** (`is_controlled`).
- **تنبيه طبي واضح** في أي شاشة فيها بدائل: أي تبديل بمراجعة الطبيب أو الصيدلي.
- **السعر:** على حد علم الملف التاني أسعار الأدوية في مصر مسعّرة رسميًا، فخلي السعر مرجعي من `official_price` في الكتالوج مش من الصيدلية.
- **حماية البيانات الشخصية** لموقع المريض وتليفونه.
- **ترخيص الـ dataset** لازم يتراجع قبل الاستخدام.
- **Concurrency:** آلاف المستخدمين ممكن يبحثوا في نفس الوقت، فالـ indexes والـ connection pooling والـ cache للكتالوج مهمين.
- **الجودة:** Testcontainers، CI، logging وmetrics.

## 13. المخاطر

|   |   |
|---|---|
|الخطر|التخفيف|
|الصيدليات مش هتلتزم بتحديث المخزون لحظيًا|TTL + حالة «غير مؤكد» + Request broadcast + بلاغات المستخدمين|
|بيانات توافر قديمة تضيّع ثقة المريض|اعرض وقت آخر تأكيد دايمًا وماتكاشّيش التوافر|
|هلوسة الـ LLM في دواء|البدائل بـ SQL حتمي، والـ AI اختياري ومقيّد|
|ربط أسماء الأدوية في ملف الصيدلية بالكتالوج _(نقطة مضافة من عندي)_|Template CSV بكود/ID الدواء + تقرير بالصفوف الفاشلة|
|ترخيص الـ dataset|مراجعته في المرحلة 0|
|منافسة قوية (شفاء وغيرها)|اعتبره portfolio وركّز على جودة التصميم|

## 14. أسئلة لازم نحسمها قبل الكود

الملفين مش بيجاوبوا عليها:

1. مصدر الـ dataset بتاع الأدوية المصرية وترخيصه إيه؟
2. إشعارات الصيدليات هتبقى عن طريق إيه (Push / SMS / WhatsApp)؟
3. المريض لازم يسجّل ولا البحث متاح للزوار؟
4. الواجهة موبايل ولا ويب ولا API بس؟
5. هنخزّن `quantity` ولا الـ status بس؟
6. قيمة الـ TTL كام (ساعات؟ يوم؟)
7. توثيق الصيدلية هيبقى يدوي من الـ ADMIN بناءً على `license_no`؟
8. هنبدأ بأنهي محافظات؟

## 15. إزاي تقدّمه في الـ CV والإنترفيو

- مشروع **impact-driven** بيحل مشكلة مجتمعية حقيقية.
- Stack تقيل في مكان واحد: Spring Boot 3 + PostGIS + Redis (+ Elasticsearch وSpring AI لو دخلوا).
- تحديات تصميم حقيقية: البحث العربي وهمزاته، Geolocation queries، concurrency عالية.
- قرارات مبنية على قياس: «بدأت بـ Postgres ومادخلتش ES غير بعد benchmark» أقوى من إنك تحط أحدث تقنية من الأول.
- قول إنك عارف المنافسين، وإن نجاح المشروع بيعتمد على جودة بيانات التوافر.

## 16. الخطوة الجاية

1. كتابة **Flyway migration (V1)** كاملة بالأنواع والـ constraints والـ indexes، مع أمثلة الـ queries الأساسية.
2. المرحلة 1: مشروع Spring Boot + Docker Compose (Postgres+PostGIS) + Security.