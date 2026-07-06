بص يا علي، أول حاجة أحب أقولك إن **Idempotency من أهم التوبيكس في الـ Backend والـ System Design**، ولو فهمتها صح هتفهم ليه شركات زي Stripe وPayPal وAmazon مهتمين بيها جدًا.

أنا هشرحها لك من الصفر، وبعدين نشرح المثال اللي في الصور سطر سطر، وبعدها هقولك ليه المثال ده فيه مشكلة أصلاً، وإزاي يتعمل Production Ready.

---

# أولاً: يعني إيه Idempotency؟

الكلمة جاية من الرياضيات.

يعني:

> **لو نفذت نفس العملية مرة أو ألف مرة، فالنتيجة النهائية هتكون واحدة.**

يعني العملية مش بتعمل Side Effects جديدة كل مرة.

مثال بسيط

عندك لمبة.

```
Turn Off
```

لو هي مطفية بالفعل.

```
Off
```

ولو قلت

```
Turn Off
```

تاني

لسه

```
Off
```

ولو كررت مليون مرة

```
Turn Off
```

لسه

```
Off
```

إذن العملية دي

```
Idempotent
```

---

أما لو قلت

```
Increase Volume
```

كل مرة الصوت هيزيد

يعني

مرة

```
20
```

مرة

```
30
```

مرة

```
40
```

يبقى دي

```
Not Idempotent
```

---

# في الـ Backend

تخيل عندك API

```
POST /payments
```

وده بيسحب فلوس.

لو العميل بعت الريكوست مرتين

يبقى الفلوس اتخصمت مرتين.

وده كارثة.

---

# ليه العميل هيبعت مرتين؟

مش شرط يكون غبي.

ممكن يحصل

- النت وقع
    
- Timeout
    
- Response ضاع
    
- ضغط الزر مرتين
    
- الموبايل عمل Retry
    

كل دي حاجات بتحصل كل يوم.

عشان كده أنت لازم تحمي السيرفر.

---

# ودي أهم نقطة

المشكلة

**مش إن اليوزر ضغط مرتين**

المشكلة

**إن السيرفر نفذ العملية مرتين.**

---

# Safe Methods

في HTTP عندنا حاجة اسمها

Safe Methods

يعني

الميثود دي

**لا تغير أي بيانات على السيرفر.**

زي

```
GET
```

مثلاً

```
GET /users
```

هيجيب المستخدمين فقط.

مهما عملت

```
GET
```

1000 مرة

ولا صف هيتغير.

---

وكمان

```
HEAD
```

بيرجع Headers فقط.

---

```
OPTIONS
```

بيعرفك السيرفر بيدعم إيه.

---

```
TRACE
```

لأغراض الـ debugging.

---

كل دول

Safe

---

# Idempotent Methods

مش معناها Safe.

في فرق.

تعالى نشوف.

---

## GET

يرجع البيانات فقط.

كل مرة

نفس النتيجة.

يبقى

Idempotent

---

## PUT

```
PUT /user/1
```

```
{
"name":"Ali"
}
```

أول مرة

هيغير الاسم.

ثاني مرة

الاسم أصلاً Ali.

مفيش تغيير.

إذن

Idempotent.

---

## DELETE

```
DELETE /user/5
```

أول مرة

يحذفه.

ثاني مرة

هو أصلاً اتحذف.

الحالة النهائية هي

"مش موجود"

إذن

Idempotent.

---

## POST

هنا المشكلة.

```
POST /orders
```

كل مرة

Order جديد.

يبقى

Not Idempotent.

---

# مثال الدفع

نفترض العميل اشترى حاجة.

ضغط

```
Pay
```

الـ Request وصل.

السيرفر خصم الفلوس.

لكن

الـ Response ضاع.

العميل شاف Loading.

قال

واضح إنه محصلش حاجة.

ضغط

Pay

تاني.

السيرفر خصم الفلوس مرة تانية.

كارثة.

---

# الحل؟

نعرف إن الريكوست ده

هو نفس الريكوست القديم.

إزاي؟

---

# UUID

UUID

اختصار

Universally Unique Identifier

مثال

```
f47ac10b-58cc-4372-a567-0e02b2c3d479
```

رقم يكاد يكون مستحيل يتكرر.

---

الـ Client يولد UUID.

مثلاً

```
123-abc
```

ويبعت

```
POST /payments
```

Header

```
Idempotency-Key:123-abc
```

---

السيرفر يعمل إيه؟

يشوف

```
123-abc
```

هل موجود؟

لا.

يبقى

نفذ الدفع.

بعد التنفيذ

يحفظ

```
123-abc
```

---

لو العميل بعت نفس الريكوست

بنفس

```
123-abc
```

السيرفر يقول

```
آه

أنا نفذت ده قبل كده.
```

إذن

مش هننفذه تاني.

---

# إذن Idempotency Key وظيفته

إدارة

Duplicate Requests

وليس

Authentication

ولا Authorization.

---

# المثال اللي في الفيديو

هو عمل

```
config
```

ليه؟

لأن

Interceptor

بيعتبر جزء من

Infrastructure

أو

Web Configuration

مش Business Logic.

---

عمل

```
IdempotencyInterceptor
```

```
implements HandlerInterceptor
```

يعني

عايز Spring ينادي عليه

قبل ما يدخل للـ Controller.

---

# يعني إيه HandlerInterceptor؟

Spring عنده Pipeline.

```
Request

↓

Filters

↓

DispatcherServlet

↓

Interceptors

↓

Controller

↓

Service

↓

Repository

↓

Database
```

الـ Interceptor

واقف هنا

قبل الكنترولر.

يعني يقدر يقول

```
الريكوست ده يكمل
```

أو

```
يقف هنا.
```

---

# ليه Override لـ preHandle؟

لأن

```
preHandle()
```

بتتنفذ

قبل الكنترولر.

يعني

لسه ولا Service اشتغلت

ولا Database.

وده المطلوب.

---

# الصورة الأولى

هو عامل

```java
private static Set<String> cache =
new HashSet<>();
```

السؤال

ليه؟

هو محتاج يخزن

UUID

اللي اتنفذت.

اختار

```
HashSet
```

ليه؟

لأن

```
contains()
```

سريعة جدًا.

تقريبًا

O(1)

---

# ليه اسمها cache؟

لأنه بيستخدمها كذاكرة مؤقتة.

بس دي

مش Redis.

ولا Database.

مجرد

RAM.

---

# بعد كده

```java
String idUuid =
request.getHeader("ID_UUID");
```

يعني

روح هات

Header

اسمه

```
ID_UUID
```

مثلاً

```
ID_UUID

abc-111
```

---

بعد كده

```java
if(cache.contains(idUuid))
```

يعني

هل UUID موجود؟

---

لو موجود

```
return false;
```

يعني

وقف الريكوست.

متكملوش.

---

لو مش موجود

```java
cache.add(idUuid);
```

يعني

احفظه.

---

ثم

```java
return HandlerInterceptor.super.preHandle(...)
```

يعني

اسمح للريكوست يكمل.

---

# الصورة الثانية

هو عمل

```
WebMvcConfigurer
```

ليه؟

علشان يسجل الـ Interceptor.

Spring لوحده مش هيعرفه.

---

جواها

```java
registry.addInterceptor(
new IdempotencyInterceptor())
```

يعني

أي Request

مر عليه الأول.

---

بعدها

```java
.addPathPatterns("/api/v1/payments")
```

يعني

الـ Interceptor

هيشتغل

على

```
payments
```

بس.

مش كل APIs.

---

# الصورة الثالثة

```
POST

/api/v1/payments
```

أول ما الريكوست يدخل

مش هيروح هنا.

هيروح

للـ Interceptor.

لو الـ UUID موجود

الكنترولر

مش هيتنفذ أصلاً.

وده معنى كلامك:

> "مش هنخلي الريكوست يوصل للداتا بيز"

لأننا وقفناه قبل ما يدخل الكنترولر، وبالتالي لا Service ولا Repository ولا Database هيتنفذوا.

---

# لكن...

المثال ده فيه مشاكل كبيرة.

أول مشكلة

```
HashSet
```

لو السيرفر Restart

كل البيانات راحت.

---

ثاني مشكلة

لو عندك

Server A

Server B

Server C

كل واحد عنده HashSet مختلفة.

يعني

Request يروح A

يحفظ UUID.

بعدها Retry يروح B.

B هيقول

"أول مرة أشوفه."

وهيخصم الفلوس تاني.

---

ثالث مشكلة

مفيش Expiration.

الـ HashSet هتفضل تكبر.

Memory Leak.

---

رابع مشكلة

هو بيحفظ الـ UUID **قبل** نجاح عملية الدفع.

لو الدفع فشل بعد كده، هيعتبر الـ UUID مستخدم، ومش هتقدر تعيد المحاولة بنفس المفتاح.

---

# الحل الحقيقي في الشركات

بدل

```
HashSet
```

بيستخدموا:

- Redis (مع TTL)
    
- أو Database Table فيها `idempotency_key`
    
- أحيانًا بيخزنوا كمان الـ Response نفسه، بحيث لو وصل نفس المفتاح مرة تانية يرجعوا نفس الرد بدل تنفيذ العملية مرة أخرى.
    

---

**نصيحتي ليك:** بما إنك بتذاكر Backend وSpring Boot وبتستعد للـ Interviews، اتعلم Idempotency بالطريقة الـ Production، باستخدام **Redis أو Database** مع **Idempotency-Key** في الـ Header. ده هو الأسلوب اللي هتسمعه في مقابلات الشغل، وهو اللي بيتستخدم في أنظمة الدفع الحقيقية.