# `@Primary` vs `@Qualifier` in Spring Boot

بص يا علي، الاتنين بنستخدمهم لما يكون عندنا أكتر من Bean من نفس النوع (Type)، وSpring مش عارفة تختار أنهي واحدة تحقنها في الـ Dependency Injection.

## 1. نفهم المشكلة الأول

تخيّل عندك `NotificationService` interface، وعندك implementation بتبعت Email وimplementation تانية بتبعت SMS.

```

public interface NotificationService {
    void send(String message);
}
```

عندنا implementation للـ Email:

```

@Service
public class EmailNotificationService
        implements NotificationService {

    public void send(String message) {
        System.out.println("Sending Email");
    }
}
```

وimplementation للـ SMS:

```

@Service
public class SmsNotificationService
        implements NotificationService {

    public void send(String message) {
        System.out.println("Sending SMS");
    }
}
```

دلوقتي عندنا Service محتاجة تستخدم `NotificationService`:

```

@Service
@RequiredArgsConstructor
public class OrderService {

    private final NotificationService notificationService;
}
```

المشكلة فين؟

Spring لقت اتنين Beans من نفس النوع:

- `EmailNotificationService`
    
- `SmsNotificationService`
    

فمش عارفة تختار أنهي واحدة تحقنها في `OrderService`، وبالتالي هتحصل مشكلة وقت إنشاء الـ Bean بسبب وجود أكتر من مرشح مناسب للـ Dependency.

هنا بييجي دور `@Primary` و`@Qualifier`.

## 2. إيه هي `@Primary`؟

معناها: لو فيه أكتر من Bean مناسبة لنفس الـ Dependency، اعتبر الـ Bean دي هي الاختيار الافتراضي.

مثلاً:

```

@Service
@Primary
public class EmailNotificationService
        implements NotificationService {

    public void send(String message) {
        System.out.println("Sending Email");
    }
}
```

دلوقتي لو عندنا:

```
private final NotificationService notificationService;
```

Spring هتختار `EmailNotificationService` لأنها متعلّمة بـ `@Primary`.

`NotificationService`

`@Primary` → `EmailNotificationService`

Default choice

`SmsNotificationService`

مش الاختيار الافتراضي

ملحوظة: `@Primary` مش معناها إن Spring هتستخدم الـ Bean دي في كل الحالات، لكنها الاختيار الافتراضي لما يكون فيه أكتر من مرشح مناسب.

## 3. إيه هي `@Qualifier`؟

معناها: أنا عايز الـ Bean دي بالاسم تحديدًا، مش الاختيار الافتراضي.

مثلاً، حتى لو `EmailNotificationService` عليها `@Primary`، تقدر تطلب SMS بالشكل ده:

```

@Service
@RequiredArgsConstructor
public class OrderService {

    @Qualifier("smsNotificationService")
    private final NotificationService notificationService;
}
```

لكن خد بالك! الكود ده مش هو الاستخدام المعتاد مع `@RequiredArgsConstructor`، لأن `@Qualifier` هنا موجودة على الـ field، وLombok مش بالضرورة تنقلها للـ generated constructor parameter من غير إعدادات إضافية.

الطريقة الواضحة والآمنة:

```

@Service
public class OrderService {

    private final NotificationService notificationService;

    public OrderService(
            @Qualifier("smsNotificationService")
            NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

هنا Spring هتحقن `SmsNotificationService` تحديدًا، حتى لو الـ Email هي الـ `@Primary`.

الاسم الافتراضي للـ Bean عادةً بيكون اسم الكلاس بحرف صغير في البداية، زي `smsNotificationService`.

## 4. الفرق بين `@Primary` و`@Qualifier`

|المقارنة|`@Primary`|`@Qualifier`|
|---|---|---|
|الوظيفة|تحدد الاختيار الافتراضي|تحدد الـ Bean المطلوبة تحديدًا|
|مكان الاستخدام|على تعريف الـ Bean|غالبًا عند نقطة الحقن، ويمكن كمان استخدامها على تعريف الـ Bean كـ qualifier|
|لو فيه أكتر من Bean|تفضّل الـ Bean المعلّمة بها|بتحدد المرشح المطلوب|
|مين له الأولوية؟|اختيار افتراضي|عند استخدامها لتحديد Bean مناسبة، بتتغلب على الاختيار الافتراضي|

## 5. لو استخدمت الاتنين مع بعض؟

افترض إن Email عليها `@Primary`، لكن كتبت:

```
public OrderService(
        @Qualifier("smsNotificationService")
        NotificationService notificationService) {
    this.notificationService = notificationService;
}
```

النتيجة: SMS هي اللي هتتحقن؛ لأنك حددتها صراحةً باستخدام `@Qualifier`.

خد بالك كمان إن `@Primary` مش بتحل المشكلة لو فيه أكتر من Bean معلّمة بـ `@Primary` لنفس النوع، أو لو مفيش Bean مناسبة أصلًا.

## 6. تحفظ إيه للانترفيو؟

- `@Primary`: عندي أكتر من implementation، وعايز واحدة تكون الـ default.
    
- `@Qualifier`: عندي أكتر من implementation، وعايز أختار واحدة بعينها عند الـ injection point.
    

مثال سريع: عندك Email وSMS وPush Notification. لو معظم النظام بيستخدم Email، ممكن تخليها `@Primary`. لكن في `OrderService` لو عايز تبعت SMS تحديدًا، استخدم `@Qualifier`.