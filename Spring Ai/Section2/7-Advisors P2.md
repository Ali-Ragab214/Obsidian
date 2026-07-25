السكريبت ده بيتكلم عن **Advisors** في Spring AI، وخصوصًا اتنين: `SimpleLoggerAdvisor` و `SafeGuardAdvisor`. خليني ألخصهولك بدون لف ودوران.

---

# الأول: يعني إيه Advisor؟

الـ **Advisor** هو كود بيتنفذ **قبل أو بعد** ما الـ Request يروح للـ LLM.

فكر فيه كأنه **Interceptor أو Filter** في Spring.

يعني:

```text
User
   │
Advisor
   │
LLM
   │
Advisor
   │
Response
```

---

# أول حاجة اتكلم عنها: SimpleLoggerAdvisor

ده Advisor جاهز موجود في Spring AI.

وظيفته:

- يطبع الـ Request في الـ Console.
    
- يطبع الـ Response في الـ Console.
    

يعني لو المستخدم بعت:

```text
How to reset my password?
```

هيطبع:

```text
Request:
System: You are an IT assistant...
User: How to reset my password
```

وبعد ما الـ LLM يرد:

```text
Response:
To reset your password...
```

وده بيساعدك في الـ Debugging.

---

# هو بيعمل ده إزاي؟

قبل ما يبعت للـ LLM

```text
Log Request
```

يبعت للـ LLM

```text
Call LLM
```

بعد ما الـ LLM يرد

```text
Log Response
```

---

# ليه فعل Debug؟

لأن الـ Logger بيستخدم:

```java
logger.debug(...)
```

والـ Debug مقفول افتراضياً.

فلازم تضيف:

```properties
logging.level.org.springframework.ai.chat.client.advisor=DEBUG
```

علشان تشوف الـ Logs.

---

# بعد كده اتكلم عن Order

كل Advisor ليه رقم.

مثلاً

```text
Advisor A -> Order 10

Advisor B -> Order 5
```

اللي رقمه أعلى بيتنفذ الأول.

يعني

```
10

ثم

5
```

---

# بعد كده اتكلم عن Streaming

فيه طريقتين للرد من الـ LLM.

## Normal

```text
User

...

استنى

...

كل الرد ييجي مرة واحدة.
```

---

## Streaming

```text
User

Hello

↓

H

↓

He

↓

Hel

↓

Hello

↓

Hello Ali
```

الرد بينزل جزء جزء.

زي ChatGPT.

---

علشان كده الـ Advisor فيه ميثودين.

```java
adviceCall()
```

للـ Normal.

و

```java
adviceStream()
```

للـ Streaming.

---

# بعد كده اتكلم عن SafeGuardAdvisor

ده Advisor جاهز برضه.

بدل ما يطبع Logs

هو بيمنع كلمات معينة.

مثلاً

أنت حاطط:

```text
hack

password

credit card
```

لو المستخدم كتب

```text
How to hack Facebook?
```

الـ Advisor هيوقف الطلب.

ومش هيبعته أصلاً للـ LLM.

ويرجع رسالة جاهزة:

```text
I'm unable to respond due to sensitive content.
```

وده مفيد في تطبيقات الشركات لحماية النظام.

---

# آخر حاجة... هو غير إيه في الكود؟

قبل الفيديو غالبًا كان الـ `ChatClient` معمول كده:

```java
@Bean
ChatClient chatClient(ChatClient.Builder builder) {
    return builder.build();
}
```

بعد الفيديو بقى:

```java
@Bean
ChatClient chatClient(ChatClient.Builder builder) {
    return builder
            .defaultAdvisors(new SimpleLoggerAdvisor())
            .build();
}
```

**الفرق الوحيد** إنه أضاف:

```java
.defaultAdvisors(new SimpleLoggerAdvisor())
```

علشان أي Request يطلع منه يطبع الـ Request والـ Response تلقائيًا.

وكمان أضاف في `application.properties`:

```properties
logging.level.org.springframework.ai.chat.client.advisor=DEBUG
```

علشان الـ Logs تظهر.

---

# الخلاصة

- **Advisor** = كود بيتنفذ قبل أو بعد استدعاء الـ LLM.
    
- **SimpleLoggerAdvisor** = يسجل الـ Request والـ Response في الـ Console.
    
- **SafeGuardAdvisor** = يمنع الطلبات اللي فيها كلمات حساسة.
    
- **`adviceCall()`** للطلبات العادية، و**`adviceStream()`** للـ Streaming.
    
- أضاف `defaultAdvisors()` في `ChatClient` وفعّل الـ `DEBUG` Logging، وبكده كل الـ Requests والـ Responses بقت تتسجل تلقائيًا.