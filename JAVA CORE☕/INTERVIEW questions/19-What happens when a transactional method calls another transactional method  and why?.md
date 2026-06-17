
## السؤال

> **What happens when a transactional method calls another transactional method, and why?**

بالعربي:

> **ماذا يحدث عندما تقوم Method عليها `@Transactional` باستدعاء Method أخرى عليها أيضًا `@Transactional`؟ ولماذا؟**

---

# الإجابة المختصرة للإنترفيو

> **If a transactional method calls another transactional method within the same Spring bean, the second method usually does not create a new transaction because the call bypasses Spring's proxy mechanism. Therefore, both methods execute within the same transaction.**

---

# شرح الفكرة

لكي نفهم الإجابة، يجب أن نعرف أن Spring تطبق `@Transactional` باستخدام شيء اسمه:

> **Proxy**

الـ Proxy عبارة عن object وسيط (Wrapper) ينفذ المنطق الخاص بالـ Transaction.

مثلاً:

```text
Client
   ↓
Spring Proxy
   ↓
Your Service
```

عندما تستدعي Method من خارج الـ Bean:

```java
userService.createUser();
```

فالاستدعاء يمر على الـ Proxy:

```text
Client
 ↓
Proxy
 ↓
createUser()
```

فيقوم الـ Proxy بـ:

- فتح Transaction.
    
- تنفيذ الميثود.
    
- عمل Commit أو Rollback.
    

---

# المشكلة

إذا كانت لديك Method داخل نفس الـ Service تستدعي Method أخرى:

```java
@Service
public class UserService {

    @Transactional
    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
        // logic
    }
}
```

فماذا يحدث؟

---

## ماذا يحدث فعليًا؟

عند استدعاء:

```java
userService.methodA();
```

يحدث الآتي:

```text
Client
 ↓
Proxy
 ↓
methodA()
```

يتم فتح Transaction.

لكن داخل:

```java
methodA() {
    methodB();
}
```

فإن الاستدعاء يكون:

```text
this.methodB()
```

أي:

```text
UserService
 ↓
methodB()
```

وليس:

```text
Proxy
 ↓
methodB()
```

وبالتالي:

> **يتم تجاوز الـ Proxy (Bypass Proxy).**

---

# النتيجة

## إذا كانت الـ Propagation الافتراضية (`REQUIRED`)

فإن:

> **methodB لن تنشئ Transaction جديدة، بل ستعمل داخل نفس Transaction الخاصة بـ methodA.**

مثال:

```java
@Service
public class UserService {

    @Transactional
    public void methodA() {

        System.out.println("A");

        methodB();

        System.out.println("End A");
    }

    @Transactional
    public void methodB() {

        System.out.println("B");
    }
}
```

النتيجة:

```text
Transaction Started
A
B
End A
Transaction Commit
```

يوجد Transaction واحدة فقط.

---

# لماذا؟

لأن:

> **Spring AOP works through proxies, and self-invocation bypasses the proxy.**

بالعربي:

> لأن Spring تعتمد على الـ Proxy لتطبيق `@Transactional`، وعندما تستدعي Method داخل نفس الكلاس فإن الاستدعاء لا يمر على الـ Proxy.

---

# سؤال مهم جدًا

## ماذا لو كانت methodB تستخدم:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

مثال:

```java
@Transactional
public void methodA() {

    methodB();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {

}
```

قد تتوقع أن:

```text
Transaction A
 ↓
Transaction B (New)
```

لكن الحقيقة:

> **لن يتم إنشاء Transaction جديدة إذا كان الاستدعاء داخل نفس الـ Bean.**

لأن:

```text
methodA()
 ↓
this.methodB()
```

وليس:

```text
Proxy.methodB()
```

وبالتالي لن يتم تطبيق `REQUIRES_NEW`.

---

# كيف نجعل `REQUIRES_NEW` تعمل؟

هناك أكثر من طريقة، أشهرها:

## 1) فصل الميثود في Service أخرى

مثال:

```java
@Service
public class OrderService {

    private final AuditService auditService;

    public OrderService(AuditService auditService) {
        this.auditService = auditService;
    }

    @Transactional
    public void placeOrder() {

        // save order

        auditService.saveAudit();
    }
}
```

```java
@Service
public class AuditService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveAudit() {

        // save audit
    }
}
```

هنا يحدث:

```text
Client
 ↓
Proxy(OrderService)
 ↓
placeOrder()
 ↓
Proxy(AuditService)
 ↓
saveAudit()
```

فتعمل:

```text
Transaction A
    ↓
Suspend A
    ↓
Transaction B
    ↓
Commit B
    ↓
Resume A
```

---

# مثال عملي

تخيل أنك تقوم بإنشاء طلب شراء:

```text
Place Order
```

وفي نفس الوقت تريد تسجيل Log أو Audit:

```text
Save Audit
```

حتى لو فشل الطلب الأساسي، تريد حفظ الـ Audit.

هنا تستخدم:

```java
REQUIRES_NEW
```

لكن يجب أن يكون الاستدعاء عبر Proxy (Service أخرى).

---

# أسئلة إنترفيو متوقعة

### Q: What happens when a transactional method calls another transactional method in the same class?

> The second method does not go through Spring's proxy, so no new transactional behavior is applied.

---

### Q: Why does this happen?

> Because Spring implements `@Transactional` using proxies, and self-invocation bypasses the proxy.

---

### Q: Will `REQUIRES_NEW` work during self-invocation?

> No. It will be ignored because the call does not pass through the proxy.

---

### Q: How can you make `REQUIRES_NEW` work?

> Move the second method to another Spring-managed bean and call it through that bean.

---

# ملخص سريع للحفظ

> عندما تستدعي Method عليها `@Transactional` Method أخرى عليها `@Transactional` داخل نفس الـ Service، فإن الاستدعاء لا يمر على Spring Proxy، لذلك لا يتم تطبيق Transaction جديدة، وتعمل الميثود الثانية داخل نفس الـ Transaction الموجودة بالفعل. ولهذا فإن `REQUIRES_NEW` لن تعمل في حالة الـ Self Invocation إلا إذا تم الاستدعاء من Bean أخرى تمر عبر الـ Proxy.