 طبعًا. **API Versioning** من التوبيكس المهمة جدًا في الـ Backend والـ System Design، وبيتسأل عنها كتير في Interviews، خاصة لو Java أو .NET أو NestJS.

هشرحها من الصفر لحد مستوى الشركات.

---

# أولًا... يعني إيه API Versioning؟

ببساطة:

> **طريقة تسمحلك تطور الـ API بدون ما تكسر التطبيقات القديمة اللي بتستخدمه.**

يعني تقدر تعمل تحديثات على الـ API، وفي نفس الوقت العملاء القدامى يفضلوا شغالين.

---

# المشكلة

تخيل إن عندك API:

```http
GET /users/1
```

وكان بيرجع:

```json
{
  "id": 1,
  "name": "Ali"
}
```

فيه 10,000 Mobile App بيستخدموا الـ API ده.

---

بعد سنة

أنت قررت تغير الـ Response إلى:

```json
{
  "id": 1,
  "firstName": "Ali",
  "lastName": "Ragab"
}
```

الـ Mobile App لسه مستني

```json
name
```

مش

```json
firstName
```

فالـ App هيكسر.

وده اسمه

> **Breaking Change**

---

# يبقى ليه API Versioning؟

علشان

- الناس القديمة تكمل شغالة.
    
- الناس الجديدة تستخدم النسخة الجديدة.
    
- متكسرش Clients موجودين بالفعل.
    

---

# مثال من الحياة

تخيل عندك تطبيق بنك.

الإصدار الأول

```http
POST /transfer
```

بياخد

```json
{
  "amount":100
}
```

بعد سنة

البنك قرر يضيف

```json
currency
```

بقت

```json
{
  "amount":100,
  "currency":"USD"
}
```

لو غيرت الـ API مباشرة

كل التطبيقات القديمة هتقع.

---

الحل؟

تسيب القديم

وتعمل Version جديد.

---

# أشهر طرق الـ API Versioning

فيه أربع طرق مشهورة.

---

# 1) URL Versioning ⭐⭐⭐⭐⭐

وده الأشهر.

```http
/api/v1/products
```

بعدها

```http
/api/v2/products
```

مثال

```http
GET /api/v1/users
```

يرجع

```json
{
 "name":"Ali"
}
```

---

```http
GET /api/v2/users
```

يرجع

```json
{
 "firstName":"Ali",
 "lastName":"Ragab"
}
```

---

وده أكثر حاجة هتشوفها في Spring Boot وNestJS.

---

## فى Spring Boot

```java
@RestController
@RequestMapping("/api/v1/products")
```

---

والنسخة الجديدة

```java
@RestController
@RequestMapping("/api/v2/products")
```

---

فى NestJS

```typescript
@Controller({
  path: 'products',
  version: '1'
})
```

---

# 2) Header Versioning

بدل الـ URL

يبقى

```http
GET /products
```

لكن تبعت Header

```http
API-Version:2
```

أو

```http
X-API-Version:2
```

السيرفر يشوف

النسخة = 2

ويرجع Response المناسب.

---

المميزات

الـ URL يفضل ثابت.

---

العيوب

صعب تجربه من Browser.

---

# 3) Query Parameter Versioning

```http
/products?version=2
```

---

وده سهل.

بس قليل الاستخدام.

---

# 4) Accept Header Versioning

ودي الشركات الكبيرة بتحبها.

مثلاً

```http
Accept:
application/vnd.company.v2+json
```

السيرفر يفهم

إن العميل عايز

V2.

---

# إمتى أعمل Version جديد؟

مش كل تعديل.

---

مثلاً

أضفت

```json
phone
```

للـ Response.

```json
{
 "name":"Ali",
 "phone":"010..."
}
```

دى غالبًا

مش Breaking.

لأن التطبيقات القديمة هتتجاهل الـ Field الجديد.

---

لكن

غيرت

```json
name
```

إلى

```json
firstName
```

دى Breaking.

لازم Version جديد.

---

غيرت

```http
POST
```

إلى

```http
PUT
```

Version جديد.

---

غيرت

شكل الـ JSON.

Version جديد.

---

# ليه منمسحش V1؟

لأن

يمكن فيه

Android App

قديم.

أو

POS System

قديم.

أو

Partner

لسه بيستخدم V1.

---

# إذن الشركة بتعمل

```text
V1

Supported
```

و

```text
V2

Latest
```

وبعد فترة

تعلن

```text
V1 Deprecated
```

وبعدها

تشيله.

---

# فى الـ System Design

تخيل عندك

```
            Clients

        Android V1

        Android V2

        Web

        Partner APIs

              │

              ▼

          API Gateway

      ┌───────────────┐
      │               │
   Route V1        Route V2
      │               │
      ▼               ▼

  Product V1      Product V2
```

الـ API Gateway يشوف

النسخة.

ويروح للسيرفر المناسب.

---

# هل V1 و V2 يبقوا مشروعين؟

مش شرط.

---

ممكن

نفس المشروع.

```text
controller

    v1

       UserController

    v2

       UserController
```

---

أو

Microservices مختلفة.

---

# أفضل Practice

```
Controllers

    v1

    v2

Services

Repositories

Entities
```

يعني غالبًا

Service

واحدة.

Repository

واحدة.

الاختلاف يكون

فى

Controller

و

DTO.

---

# مثال Interview

سألك:

> عندك API عليها مليون مستخدم، وعايز تغير الـ Response، تعمل إيه؟

الإجابة:

> **I wouldn't modify the existing API because that would introduce a breaking change. Instead, I'd introduce a new API version (for example `/api/v2`), keep the old version operational for backward compatibility, announce its deprecation, and eventually remove it after clients have migrated.**

---

# في NestJS

NestJS بيدعم الـ Versioning Built-in.

في `main.ts`:

```typescript
app.enableVersioning({
  type: VersioningType.URI,
});
```

وبعدين:

```typescript
@Controller({
  path: 'products',
  version: '1',
})
export class ProductV1Controller {}
```

والنسخة الجديدة:

```typescript
@Controller({
  path: 'products',
  version: '2',
})
export class ProductV2Controller {}
```

هيبقى عندك:

```http
GET /v1/products
GET /v2/products
```

---

# ملخص تحفظه

**API Versioning** هو أسلوب لإدارة تطور الـ APIs مع الحفاظ على **Backward Compatibility**. الهدف منه هو السماح بإضافة أو تغيير خصائص الـ API دون كسر التطبيقات الحالية. نلجأ إليه عند وجود **Breaking Changes**، وأكثر طريقة استخدامًا هي **URI Versioning** مثل `/api/v1` و`/api/v2`. أما التعديلات غير الكاسرة (مثل إضافة حقول جديدة اختيارية) فغالبًا لا تحتاج إلى إصدار جديد. في الأنظمة الكبيرة، قد يتولى **API Gateway** توجيه الطلبات إلى النسخة المناسبة بناءً على رقم الإصدار.