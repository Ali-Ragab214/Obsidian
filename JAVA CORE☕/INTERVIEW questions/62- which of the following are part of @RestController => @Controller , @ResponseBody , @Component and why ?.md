السؤال ده شكله بيسأل: مين من الـ annotations دي بيحتوي (أو معمول عليه meta-annotation بـ) `@Component`؟ خليني أوضحلك العلاقة بينهم بالتفصيل.

## الفكرة الأساسية: Meta-Annotations

في Spring، بعض الـ annotations بتبقى معمول عليها annotations تانية (اسمها **meta-annotation**)، يعني الـ annotation نفسه بيورث خصائص annotation تاني.

## 1. `@Component` (الأساس)

ده الـ annotation الأساسي اللي بيقول لـ Spring: "الكلاس ده اعتبره **bean** وسجّله في الـ ApplicationContext (component scanning)."

```java
@Component
public class MyService { }
```

## 2. `@Controller`

لو فتحت الـ source code بتاعه هتلاقيه:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component  // <-- هنا
public @interface Controller {
}
```

يعني **`@Controller` هو `@Component`** بس معمول عليه meta-annotation، فبيورث كل خصائصه. فبمجرد ما تحط `@Controller` على الكلاس، Spring هيعامله كـ bean عادي زي أي حاجة عليها `@Component`، بس بمعنى إضافي إنه "controller" (يعني هيستقبل HTTP requests).

## 3. `@RestController`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller       // <-- هنا
@ResponseBody     // <-- وهنا
public @interface RestController {
}
```

يعني **`@RestController` = `@Controller` + `@ResponseBody`** مجمّعين مع بعض. وبما إن `@Controller` نفسه معمول عليه `@Component`، فبالـ transitive inheritance، `@RestController` كمان بيبقى **`@Component`** (بالوراثة من `@Controller`).

يعني لما بتحط `@RestController` على الكلاس، انت فعليًا بتحصل على تلاتة حاجات مع بعض:

1. تسجيل كـ bean (من `@Component`)
2. يشتغل كـ web controller (من `@Controller`)
3. كل الـ methods جواه بترجع البيانات مباشرة في الـ response body مش كـ view name (من `@ResponseBody`)

## 4. `@ResponseBody`

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseBody {
}
```

هنا **مفيش `@Component`** خالص. `@ResponseBody` مش stereotype annotation، هو بس **marker annotation** بيقول لـ Spring: "الـ return value بتاع الـ method ده متعامل معاه كـ HTTP response body مباشرة (JSON/XML)، مش هيتفسر كـ اسم view (زي في MVC التقليدي مع Thymeleaf مثلاً)."

يعني `@ResponseBody` ميعملش الكلاس/الميثود bean، هو بس بيغير طريقة التعامل مع الـ **return value**.

---

## الخلاصة (الإجابة المباشرة على السؤال)

|Annotation|معمول عليه `@Component`؟|ليه|
|---|---|---|
|`@Component`|(هو نفسه)|ده الأصل|
|`@Controller`|✅ أيوه|معمول عليه `@Component` مباشرة (meta-annotation)|
|`@RestController`|✅ أيوه|بيورثها transitively عن طريق `@Controller`|
|`@ResponseBody`|❌ لأ|مش stereotype annotation، هو بس بيتحكم في شكل الـ response|

**يعني الجزء اللي "بيبقى `@Component`" فعليًا هو: `@Controller` و `@RestController` بس. أما `@ResponseBody` مش جزء من الـ component scanning system خالص.**

هل السؤال ده من كويز أو من إنترفيو معين وعايز تتأكد من صيغة تانية ليه؟