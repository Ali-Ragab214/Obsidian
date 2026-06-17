# ما هو Spring Proxy؟

بص يا علي، لو فهمت الـ **Proxy** كويس، هتفهم بعدها:

- `@Transactional`
    
- `@Async`
    
- `@Cacheable`
    
- `@Secured`
    
- جزء كبير من Spring AOP
    

لأن كلهم تقريبًا بيعتمدوا على نفس الفكرة.

---

# تخيل المثال ده

عندك Service:

```java
@Service
public class PaymentService {

    public void processPayment() {
        System.out.println("Processing payment...");
    }
}
```

ولو عملت:

```java
paymentService.processPayment();
```

هيتنفذ الكود مباشرة.

---

## لكن ماذا لو Spring عايزة تضيف حاجة قبل وبعد تنفيذ الميثود؟

مثلاً:

- فتح Transaction.
    
- التحقق من الصلاحيات.
    
- تسجيل Logs.
    
- قياس وقت التنفيذ.
    
- عمل Caching.
    

هل هتدخل تعدل في `PaymentService` نفسه؟

لا.

بدل ذلك Spring تعمل كائن وسيط اسمه:

> **Proxy**

---

# ما هو الـ Proxy؟

> **A Proxy is an object that sits between the client and the real object to add extra behavior before or after method execution.**

بالعربي:

> الـ Proxy هو كائن وسيط يقف بين المستخدم والكائن الحقيقي، ويضيف سلوكًا إضافيًا قبل أو بعد تنفيذ الميثود.

---

# الشكل الطبيعي

بدون Proxy:

```text
Client
   ↓
PaymentService
   ↓
processPayment()
```

---

# مع Proxy

```text
Client
   ↓
Spring Proxy
   ↓
PaymentService
   ↓
processPayment()
```

---

# مثال Transaction

لو كتبت:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {

        // save payment

    }
}
```

أنت لم تكتب:

```java
beginTransaction();
commit();
rollback();
```

صح؟

طيب مين كتبهم؟

الإجابة:

> **Spring Proxy**

تقريبًا بيعمل كده:

```java
proxy.processPayment() {

    beginTransaction();

    try {

        realObject.processPayment();

        commit();

    } catch(Exception e) {

        rollback();

        throw e;
    }
}
```

---

# يعني هل الـ Proxy هو الكلاس الأصلي؟

لا.

عندك فعليًا كائنين:

## الكائن الحقيقي

```java
PaymentService
```

وده فيه الـ Business Logic.

---

## الكائن الوسيط

```java
PaymentServiceProxy
```

وده مسؤول عن:

- فتح الـ Transaction.
    
- عمل Commit.
    
- عمل Rollback.
    

ثم يستدعي الكائن الحقيقي.

---

# Spring بتنشئ الـ Proxy إمتى؟

أثناء تشغيل التطبيق.

يعني لما Spring تشوف Annotation زي:

```java
@Transactional
```

تقول:

> "بدل ما أرجع الـ Bean الحقيقية، هلفها داخل Proxy."

---

# مثال عملي

أنت تكتب:

```java
paymentService.processPayment();
```

لكن اللي بيحصل فعليًا:

```text
Proxy.processPayment()
        ↓
Open Transaction
        ↓
PaymentService.processPayment()
        ↓
Commit/Rollback
```

---

# لماذا نحتاج Proxy؟

عشان تضيف وظائف عامة بدون ما تلوث الـ Business Logic.

بدل:

```java
public void transfer() {

    openTransaction();

    checkSecurity();

    writeLogs();

    // business logic

    commit();
}
```

تخلي الكود:

```java
@Transactional
public void transfer() {

    // business logic only
}
```

وSpring تتولى الباقي.

---

# أشهر الأشياء التي تستخدم Proxy

## 1. Transactions

```java
@Transactional
```

---

## 2. Async

```java
@Async
```

تشغل الميثود في Thread آخر.

---

## 3. Caching

```java
@Cacheable
```

---

## 4. Security

```java
@PreAuthorize
```

---

## 5. Spring AOP

مثل:

- Logging
    
- Performance Monitoring
    
- Auditing
    

---

# أنواع الـ Proxy في Spring

## 1) JDK Dynamic Proxy

تستخدم إذا كان الكلاس يطبق Interface.

مثال:

```java
public interface PaymentService {
    void pay();
}
```

```java
@Service
public class PaymentServiceImpl implements PaymentService {

    public void pay() {

    }
}
```

Spring تنشئ Proxy للإInterface.

---

## 2) CGLIB Proxy

إذا لم يوجد Interface.

مثال:

```java
@Service
public class PaymentService {

    public void pay() {

    }
}
```

Spring تنشئ Class جديد يرث من:

```java
PaymentService
```

تقريبًا:

```java
class PaymentServiceProxy extends PaymentService {

    @Override
    public void pay() {

        // extra behavior

        super.pay();
    }
}
```

---

# لماذا لا تعمل @Transactional أحيانًا؟

لأن الاستدعاء يجب أن يمر عبر الـ Proxy.

مثال:

```java
@Service
public class UserService {

    @Transactional
    public void methodA() {

        methodB();
    }

    @Transactional
    public void methodB() {

    }
}
```

هنا:

```text
methodA()
 ↓
this.methodB()
```

وليس:

```text
Proxy.methodB()
```

لذلك يتم تجاوز الـ Proxy.

وهذا يسمى:

> **Self Invocation**

وهو أشهر سؤال إنترفيو متعلق بالـ Proxy.

---

# هل الـ Proxy Design Pattern؟

نعم.

Spring تستخدم:

> **Proxy Design Pattern**

وهو أحد أنماط التصميم (Structural Design Patterns).

فكرته:

> إعطاء كائن بديل يتحكم في الوصول للكائن الحقيقي ويضيف سلوكًا إضافيًا.

---

# تعريف بسيط للإنترفيو

### What is Spring Proxy?

> **A Spring Proxy is an object created by Spring that wraps the target bean and intercepts method calls to provide additional behaviors such as transaction management, security, caching, and asynchronous execution.**

بالعربي:

> هو كائن تنشئه Spring ليلتف حول الـ Bean الحقيقي ويعترض استدعاءات الميثود لإضافة وظائف مثل الـ Transactions والـ Security والـ Caching دون تعديل الـ Business Logic.

---

# احفظ الجملة دي لأنها مفتاح الموضوع كله

> **`@Transactional` does not work by magic. It works because Spring creates a Proxy around the bean and intercepts method calls.`**

لو فهمت الجملة دي، هتفهم بسهولة لماذا أحيانًا `@Transactional` تعمل، ولماذا أحيانًا لا تعمل، ولماذا `REQUIRES_NEW` قد لا يُطبق في حالة الـ Self Invocation.