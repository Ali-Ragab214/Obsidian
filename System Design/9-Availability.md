
## ما هي الـ Availability؟

**التعريف الرسمي:** هي النسبة المئوية للوقت الذي يكون فيه نظامك شغال ومتاح للمستخدم.

> _"The fraction of time that our service is operationally functional and accessible to the user"_

---

## ليه مهمة؟ — الأهمية

**من ناحية المستخدم العادي:**

- بيحاول يشتري من online store والصفحة ما بتفتحش
- بيجيله error وقت الـ checkout
- بيفقد access للإيميل لفترة طويلة

**من ناحية الـ B2B (شركة بتخدم شركات تانية):**

- لو نظامك وقع، مش بس عملاءك اللي بيتأثروا — كل عملاء عملاءك بيتأثروا
- مثال حقيقي: AWS S3 وقع في فبراير 2017 وكاد يوقف الإنترنت كله لأن آلاف الشركات كانت بتعتمد عليه

**من ناحية الـ Mission Critical:**

- Air traffic control في المطارات
- أنظمة المستشفيات
- لو النظام وقع، حياة بشر بتكون على المحك

---

## تأثير الـ Downtime على البيزنس

**لو النظام وقع:**

- المستخدم مش قادر يستخدم النظام → **الربح = صفر**
- المستخدم بيفقد ثقته → **يروح للمنافس**

المعادلة بسيطة: Downtime = Loss of Revenue + Loss of Customers

---

## الصيغة الرياضية**شرح المصطلحات:**

- **Uptime:** الوقت اللي النظام فيه شغال ومتاح
- **Downtime:** الوقت اللي النظام فيه واقع
- **MTBF (Mean Time Between Failures):** متوسط الوقت بين عطلين — كلما زاد، أحسن
- **MTTR (Mean Time To Recovery):** متوسط الوقت اللي بياخده الـ recovery — كلما قل، أحسن
- في الواقع: الـ MTTR مستحيل يكون صفر، دايمًا بياخد وقت للإصلاح

---

## جدول الـ Availability vs Downtime---

## الـ "Nines" — ازاي الناس بتتكلم عن الـ Availability

في الـ industry، مش بيقولوا "99.9%" — بيقولوا "**three nines**" — يعني عدد الـ 9 في الرقم:

- 99% → two nines
- 99.9% → three nines ✅ (معيار الـ cloud vendors)
- 99.99% → four nines
- 99.999% → five nines (خاص بأنظمة زي شبكات الاتصالات)

---

## ليه مش ممكن نوصل لـ 100%؟

من ناحية هندسية، الـ 100% availability:

- **صعبة جدًا** تحقيقها عمليًا
- **مش بتسيبلك وقت للـ maintenance والـ upgrades**
- الـ MTTR مستحيل يبقى صفر

عشان كدا المعيار المقبول في الـ cloud هو **99.9% (three nines)** — ودا اللي AWS وGoogle Cloud وAzure بيضمنوه في الـ SLA بتاعتهم.

---

## مثال عملي بـ Spring Boot

يا علي، لما بتبني Spring Boot application، فكّر في الـ Availability من الأول. مثلاً، Spring Boot Actuator بيديك endpoint جاهز تقيس بيه الـ health:

```java
// في application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, metrics
  endpoint:
    health:
      show-details: always
```

لو فتحت `/actuator/health` هيرجعلك:

```json
{
  "status": "UP",
  "uptime": "PT3H25M"
}
```

وبتقدر تحسب الـ Availability بناءً على الـ uptime دي مقارنةً بالوقت الكلي.

---

**الـ key takeaways:**

- Availability = Uptime / (Uptime + Downtime)
- أو = MTBF / (MTBF + MTTR)
- المعيار في الصناعة = 99.9% (three nines)
- الـ downtime بيكلف revenue وعملاء
- الـ MTTR مستحيل يكون صفر في الواقع

عايز أكمل وأشرحلك ازاي تحقق الـ High Availability عمليًا (Redundancy, Load Balancing, Failover)؟
--

# System Outage 


**System Outage** = النظام وقع تمامًا وبقى مش متاح للمستخدمين.

يعني لو عندك website أو API أو تطبيق — المستخدم بيحاول يدخل عليه ومش بيقدر، بيجيله error أو الصفحة مش بتفتح خالص.

---

**مثال بسيط:** تخيل إنك بتشتري من Noon أو Amazon — وفجأة الموقع وقع. دا **system outage**. مش بطيء — واقع خالص.

---

**الفرق بين المصطلحات:**

|المصطلح|المعنى|
|---|---|
|**Outage**|النظام واقع تمامًا ❌|
|**Downtime**|الفترة الزمنية اللي النظام فيها واقع|
|**Degradation**|النظام شغال بس بطيء أو بيعمل errors جزئية|

---

يعني الـ **downtime** هو الوقت اللي بيحصل فيه الـ **outage**.