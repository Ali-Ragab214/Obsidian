# Interview Question: لماذا تعتبر Constructor Injection أفضل لأن الـ Dependencies تصبح Immutable؟

من أكثر الأسئلة التي تُسأل في مقابلات الـ Java وSpring Boot.

---

# السؤال

> **Why is Constructor Injection preferred over Field Injection? Explain how dependencies become immutable.**

---



عند استخدام **Constructor Injection**، فإن جميع الـ Dependencies يتم تمريرها مرة واحدة فقط أثناء إنشاء الـ Object (Object Creation).

بعد إنشاء الـ Object لا يمكن تغيير هذه الـ Dependencies مرة أخرى، لذلك نقول إنها أصبحت **Immutable**.

وهذا يجعل الـ Object أكثر أمانًا وأسهل في الصيانة والاختبار.

---

# مثال

بدون Constructor Injection

```java
@Service
public class UserService {

    @Autowired //=>for field injection
    private EmailService emailService;

}
```

هنا الـ Dependency ليست Immutable.

لأن Spring يخلق الـ Object أولًا، ثم بعد ذلك يحقن الـ Dependency.

يعني نظريًا يمكن تغييرها فيما بعد.

---

أما باستخدام Constructor Injection

```java
@Service
public class UserService {

    private final EmailService emailService;

    public UserService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

لاحظ وجود

```java
final
```

بمجرد إنشاء الـ Object...

يتم إعطاء الـ EmailService مرة واحدة فقط.

بعدها لا يمكن تغييره.

---

# لماذا final تجعل الـ Dependency Immutable؟

في Java

```java
final EmailService emailService;
```

تعني:

> يمكن إسناد قيمة واحدة فقط لهذا المتغير طوال عمر الـ Object.

مثال

```java
emailService = new EmailService();
```

مسموح مرة واحدة.

لكن

```java
emailService = new AnotherEmailService();
```

سيعطي Compile Error.

---

# ماذا يحدث أثناء إنشاء الـ Object؟

لنفترض لدينا

```java
public UserService(EmailService emailService) {
    this.emailService = emailService;
}
```

Spring يقوم بالخطوات التالية:

```
Spring Container
        │
        ▼
ينشئ EmailService
        │
        ▼
ينشئ UserService
        │
        ▼
يمرر EmailService في الـ Constructor
        │
        ▼
يحفظه داخل final field
```

بعد هذه اللحظة...

انتهى الأمر.

لا يوجد أي Setter.

ولا يمكن تغيير المرجع (Reference).

---

# ماذا لو استخدمنا Setter Injection؟

```java
@Service
public class UserService {

    private EmailService emailService;

    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

يمكن استدعاء

```java
setEmailService(...)
```

مرة ثانية.

وممكن تغيير الـ Dependency.

إذن ليست Immutable.

---

# ماذا عن Field Injection؟

```java
@Autowired
private EmailService emailService;
```

Spring يستخدم Reflection ليحقن القيمة بعد إنشاء الـ Object.

أي أن الـ Object يولد أولًا وهو غير مكتمل.

ثم يتم تعديل حالته (State) لاحقًا.

لهذا السبب لا نستطيع كتابة

```java
private final EmailService emailService;
```

مع Field Injection.

---

# لماذا هذا أفضل؟

## 1. يمنع تغيير الـ Dependency

لن يستطيع أي شخص لاحقًا تغييرها.

---

## 2. يجعل الـ Object دائمًا في حالة صحيحة (Valid State)

لا يمكن إنشاء

```java
UserService
```

بدون

```java
EmailService
```

أي أن أي Object يتم إنشاؤه سيكون جاهزًا للعمل مباشرة.

---

## 3. يزيد الأمان (Thread Safety)

طالما أن المرجع لا يتغير، تقل احتمالية حدوث مشاكل عند استخدام الـ Object من أكثر من Thread.

> ملاحظة: جعل المرجع `final` لا يجعل الكائن نفسه Immutable؛ بل يمنع تغيير المرجع إلى كائن آخر.

---

## 4. يسهل الاختبار (Testing)

```java
EmailService email = mock(EmailService.class);

UserService service = new UserService(email);
```

لا تحتاج إلى Reflection أو Spring Container.

---

## 5. تظهر الـ Dependencies بوضوح

بمجرد النظر إلى الـ Constructor تعرف كل ما يحتاجه الـ Class.

```java
public UserService(
        EmailService emailService,
        UserRepository repository,
        JwtService jwtService,
        NotificationService notificationService
)
```

أما مع Field Injection فقد تكون الاعتمادات موزعة داخل الكلاس ويصعب ملاحظتها بسرعة.

---

# هل الـ Object نفسه أصبح Immutable؟

لا.

هناك فرق بين:

```java
final EmailService emailService;
```

و

```java
Immutable Object
```

`final` يمنع تغيير المرجع فقط.

لكن إذا كان

```java
EmailService
```

يحتوي على متغيرات داخلية قابلة للتغيير، فيمكن تغيير حالته.

مثال:

```java
emailService.setHost("gmail");
```

هذا مسموح.

لكن

```java
emailService = new EmailService();
```

غير مسموح.

---

# إجابة مختصرة في المقابلة

> **Constructor Injection is preferred because dependencies are provided when the object is created and stored in `final` fields. This makes the references immutable, prevents reassignment, ensures the object is always fully initialized, improves readability, makes unit testing easier, and avoids the drawbacks of reflection-based field injection.**