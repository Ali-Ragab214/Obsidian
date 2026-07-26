ده من أشهر أسئلة الانترفيوهات في Spring، لكن أغلب الناس بتحفظ الإجابة بدون ما تفهم **ليه اسمها Stereotype Annotation أصلاً**.

---

# أولاً: إجابة السؤال

**Question:**

> Which of the following are stereotype annotations in Spring Boot?

```java
@Component
@Service
@Repository
```

**Answer:**

✅ **All of them are stereotype annotations.**

بل كمان فيه واحدة رابعة مهمة جدًا:

```java
@Controller
```

ولو شغال REST APIs

```java
@RestController
```

هي كمان تعتبر Controller Stereotype لأنها مبنية على `@Controller`.

---

# طيب يعني إيه Stereotype؟

الكلمة الإنجليزية

> **Stereotype**

معناها

> **تصنيف**
> 
> أو
> 
> **Label**
> 
> أو
> 
> **Role**

يعني أنا بدي للكلاس "هوية".

مثلاً

لو عندك شركة

فيها

```
Ali
Mohamed
Ahmed
Sara
```

كلهم موظفين.

لكن كل واحد ليه وظيفة.

```
Ali ---------> Backend Developer

Mohamed -----> HR

Sara --------> Manager

Ahmed -------> Accountant
```

كلهم موظفين...

لكن كل واحد ليه **Role**.

نفس الفكرة في Spring.

---

عندك Class

```java
public class UserService {

}
```

Spring شايفه إيه؟

ولا حاجة.

مجرد Java Class.

لكن لما أكتب

```java
@Service
public class UserService {

}
```

أنا بقول لـ Spring

> الكلاس ده Service.

وده اسمه

**Stereotype**

يعني

> أنا بحدد دوره داخل التطبيق.

---

# ليه Spring محتاج يعرف الدور؟

علشان يعمل

Component Scanning

لما التطبيق يبدأ

Spring يعمل

```
Scan

↓

يشوف Classes

↓

يشوف Annotations

↓

ينشئ Beans
```

مثلاً

```
@Service
public class UserService
```

Spring يقول

```
ده Service

↓

اعمله Bean
```

---

# يعني إيه Bean؟

دي نقطة مهمة جدًا.

```
@Component

↓

Spring Container

↓

Bean
```

يعني أي Stereotype Annotation في النهاية

بتخلي Spring ينشئ Object.

---

# طيب إيه الفرق بينهم؟

وده السؤال اللي بيجي بعده في الانترفيو.

---

# 1) @Component

ده

الأب.

```
@Component
```

أي Class.

أي حاجة.

مثال

```java
@Component
public class EmailUtil {

}
```

أو

```java
@Component
public class JwtUtil {

}
```

Spring يقول

```
تمام

Bean
```

---

# 2) @Service

ده

Specialized Component.

يعني

```
@Service

=

@Component
```

لكن معناه

> الكلاس ده Business Logic.

مثال

```java
@Service
public class UserService {

}
```

جواه

```
Create User

Delete User

Update User
```

---

# 3) @Repository

برضو

Specialized Component

```
@Repository

=

@Component
```

لكن بيقول

> الكلاس ده مسؤول عن Database.

مثال

```java
@Repository
public interface UserRepository
extends JpaRepository<User,Long>{

}
```

---

# هل @Repository مجرد اسم؟

لا.

ودي نقطة انترفيو.

لها ميزة زيادة.

لو حصل

```
SQLException
```

Spring يعمل

Exception Translation

ويحولها إلى

```
DataAccessException
```

بدل Exceptions الخاصة بكل Database.

وده بيسهل التعامل مع أخطاء قواعد البيانات.

---

# 4) @Controller

```
@Controller

=

@Component
```

لكن

خاص بالـ MVC.

مثال

```java
@Controller
public class HomeController {

}
```

---

# 5) @RestController

دي

مش مباشرة من Component.

هي

```java
@RestController
```

=

```java
@Controller
+
@ResponseBody
```

يعني

Controller

لكن يرجع JSON.

---

# طيب لو كلهم Component...

ليه موجودين أصلاً؟

وده أهم سؤال.

لو

```java
@Component
public class UserRepository
```

هيشتغل؟

الإجابة

✅ نعم.

لكن

هل ده Best Practice؟

❌ لا.

---

ليه؟

بص على المشروع.

```
UserService

UserRepository

UserController

JwtFilter

EmailUtil
```

لو كله

```
@Component
```

يبقى

```
@Component
@Component
@Component
@Component
@Component
```

محدش فاهم حاجة.

لكن

```
@Controller

↓

بيعالج Requests

@Service

↓

Business Logic

@Repository

↓

Database

@Component

↓

Utility Classes
```

بصيت للكود...

عرفت كل Class بيعمل إيه.

---

# العلاقة بينهم

```
                 @Component
                     |
      ---------------------------------
      |               |               |
  @Service      @Repository     @Controller
                                      |
                              @RestController
```

كلهم في النهاية

```
Component
```

وكلهم

```
Bean
```

---

# مثال عملي

```
Request

↓

Controller

↓

Service

↓

Repository

↓

Database
```

الكود

```java
@RestController
public class UserController {

    @Autowired
    UserService service;
}
```

↓

```java
@Service
public class UserService {

    @Autowired
    UserRepository repository;
}
```

↓

```java
@Repository
public interface UserRepository
extends JpaRepository<User,Long>{

}
```

كل واحد عليه الـ Annotation المناسبة لدوره.

---

# سؤال انترفيو مشهور جدًا

**Interviewer:**

> What's the difference between `@Component`, `@Service`, and `@Repository`?

**الإجابة المثالية:**

> All of them are stereotype annotations and are detected during component scanning, so Spring registers them as beans in the Application Context. The difference is mainly semantic:
> 
> - `@Component` is a generic stereotype for any Spring-managed bean.
>     
> - `@Service` is used for the service layer that contains business logic.
>     
> - `@Repository` is used for the persistence layer and provides automatic exception translation from database-specific exceptions to Spring's `DataAccessException`.
>     
> - `@Controller` is used for handling web requests, while `@RestController` is a specialized controller that returns JSON responses.
>     

---

# سؤال أصعب شوية

### هل `@Service` يرث من `@Component`؟

الإجابة:

مش وراثة (Inheritance) بين Classes، لكنها **Meta-Annotation**.

لو فتحت تعريف `@Service` هتلاقي:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
}
```

يعني `@Service` نفسها **مكتوب فوقها `@Component`**، ولذلك Spring يعاملها كأنها `@Component` مع دلالة إضافية (Semantic Meaning).

---

## ملخص سريع تحفظه

|Annotation|تعتبر Stereotype؟|الغرض|
|---|---|---|
|`@Component`|✅|Generic Bean|
|`@Service`|✅|Business Logic|
|`@Repository`|✅|Data Access + Exception Translation|
|`@Controller`|✅|Spring MVC Controller|
|`@RestController`|✅ (بشكل غير مباشر)|REST API Controller (`@Controller` + `@ResponseBody`)|

### قاعدة ذهبية للانترفيو

> **Every `@Service`, `@Repository`, and `@Controller` is a `@Component`, but not every `@Component` is a `@Service`, `@Repository`, or `@Controller`.**

دي جملة قوية جدًا، ولو قلتها في الانترفيو هتوضح إنك فاهم العلاقة بينهم مش مجرد حافظ أسماء الـ annotations.