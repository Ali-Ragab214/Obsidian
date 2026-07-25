![[Pasted image 20260723232906.png]]
ChatClient  --بيستخدم جواه-->  ChatModel  --بيكلم-->  LLM Provider![[Pasted image 20260723233030.png]]
ممتاز جدًا، وأنا شايف إنك بدأت تدخل في **الجزء الحقيقي من Spring AI**. من دلوقتي بقى مش هنشرح الكود بس، هنشرح **إزاي الـ AI بيفكر** وإيه اللي بيحصل خلف الكواليس.

وأول حاجة هقولها:

> **الـ 500 Internal Server Error غالبًا سببه Bug في الكود، وليس في فكرة Prompt Templates نفسها.**

خليني أمشي معاك واحدة واحدة.

---

the code 
```java
package com.example.openai.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/prompt-template")
public class PromptTemplateController {
private final ChatClient chatClient;

    public PromptTemplateController(ChatClient chatClient) {
        this.chatClient = chatClient;
    }
//    String promptTemplate =
//            """
//                    A customer named {customerName} has sent the following message:
//                    "{customerMessage}
//
//                    write a polite and helpful email response addressing the issue .
//                    maintain a professional tone and provide clear instructions or solutions to resolve the customer's concern.
//
//                    Respond as if you are writing the email body only. dont include subject signature
//                    """;

    @Value("classpath:promptTemplates/userPromptTemplate.st")
    Resource userPromptTemplate;
@GetMapping("/email")
    public String emailResponse(@RequestParam("customerName") String customerName,
                                @RequestParam("customerMessage") String customerMessage)
{
        return chatClient
                .prompt()
                .system("""
                        You are a customer support representative.
                        which helps drafting email responses to customers based on their messages.
                        """)
                .user(promptUserSpec ->
                        promptUserSpec.text(userPromptTemplate)
                                .param("customerName", customerName)
                .param("customerMessage", customerMessage))
                .call()
                .content();
}

    // You can add methods here to handle requests related to prompt templates
// لو عندي اكتر من برومت تيمبليت هحط كل واحد في فايل لوحده
}

```
### لخلاصة (الـ Chain كامل في جملة واحدة)

> "ابدأ prompt جديد → حط system message مؤقت لهذا الـ request → خد الـ user message من ملف template خارجي واستبدل الـ placeholders بالقيم اللي جايه من الـ request → نفّذ الـ call للـ LLM → استخرج الرد كـ نص بسيط بس
> 
# أولًا: ما هو Prompt Template؟

قبل ما نتكلم عن Spring AI، تخيل إن عندك Email Support.

بدل ما كل مرة تكتب Prompt بالشكل ده:

```text
A customer named Ali has sent:

"My order didn't arrive"

Write a professional email...
```

وبعد شوية:

```text
A customer named Mohamed has sent:

"I want a refund"

Write a professional email...
```

وبعدها:

```text
A customer named Sara...
```

أنت كل مرة بتكرر نفس الكلام.

وده غلط.

---

## الحل

نكتب Template.

زي HTML Template.

أو Thymeleaf.

أو Mustache.

أو Freemarker.

لكن هنا للـ AI.

---

يعني نكتب:

```text
A customer named {customerName}
has sent:

{customerMessage}

Write a professional email...
```

شايف؟

بدل

```text
Ali
```

كتبنا

```text
{customerName}
```

وبدل

```text
My order...
```

كتبنا

```text
{customerMessage}
```

وده اسمه

> Placeholder

---

بعد كده Spring AI يعمل Replace.

يعني

```text
customerName = Ali
```

↓

```text
{customerName}
```

↓

```text
Ali
```

---

يعني الـ AI أصلاً مش بيشوف

```text
{customerName}
```

هو بيشوف

```text
Ali
```

---

# ليه Prompt Templates؟

بدل

```java
String prompt =
"""
Customer:
""" + customerName +
"""
Message:
""" + customerMessage +
"""
....
""";
```

وده شكله وحش.

نعمل

```text
A customer named {customerName}
```

وبعدين

```java
.param("customerName",customerName)
```

---

# طيب الـ .st دي إيه؟

دى مجرد

```
String Template
```

امتداد ملف فقط.

يعني

```
userPromptTemplate.st
```

جواه Text.

بس.

مش Java.

مش Spring.

ولا AI.

---

مثلاً

```
src/main/resources/promptTemplates/userPromptTemplate.st
```

جواه

```text
Hello {name}

Your balance is {balance}
```

---

# Spring AI بيعمل إيه؟

أول ما يقرأ

```java
.text(userPromptTemplate)
```

يروح يقرأ الملف.

بعدها

```java
.param("name","Ali")
```

↓

يحط

Ali

مكان

```
{name}
```

---

# يعنى الـ AI بيستقبل إيه؟

هو لا يعرف

```
.st
```

ولا يعرف

```
param()
```

ولا يعرف

```
Resource
```

كل ده بيحصل داخل Spring.

في الآخر الـ AI يستقبل String واحدة.

مثلاً

```text
A customer named Ali
has sent

My internet isn't working

Write a polite email...
```

بس.

---

# خلينا نمشى على الكود

## أول حاجة

```java
@Resource
```

لا.

أنت عندك

```java
@Value(...)
Resource userPromptTemplate;
```

يعني Spring هيعمل Injection.

---

## دي معناها

```java
@Value("classpath:...")
```

يعني

روح افتح الملف الموجود هنا.

---

وده لازم يكون موجود.

```
resources
    promptTemplates
         userPromptTemplate.st
```

---

# أول Bug عندك

بص هنا

```java
@Value("classpath:promptTeemplates/userPromptTemplate.st")
```

شايف؟

كتبت

```
promptTeemplates
```

فيها

```
ee
```

😂

المفروض

```
promptTemplates
```

وده لوحده يعمل

500

لأن Spring مش هيلاقى الملف.

---

# ثاني Bug

بص على الـ Template

كتبت

```text
"{customerMessage}
```

نسيت تقفل

"

المفروض

```text
"{customerMessage}"
```

---

# ثالث Bug

أنت عامل

```java
String promptTemplate =
"""
...
""";
```

ومش مستخدمه.

أنت بتستخدم

```java
.text(userPromptTemplate)
```

يبقى الـ String ده ملوش لازمة.

---

# طيب دى

```java
.text(userPromptTemplate)
```

بتعمل إيه؟

دى بتقول

اقرأ النص من الملف.

مش من String.

---

ولو كنت عايز تستخدم String

يبقى

```java
.text(promptTemplate)
```

---

# يعنى عندك اختيارين

## الأول

من String

```java
.text(promptTemplate)
```

---

الثانى

من File

```java
.text(userPromptTemplate)
```

---

# بعد كده

```java
.param("customerName", customerName)
```

يعنى

Replace

```
{customerName}
```

↓

```
Ali
```

---

وده

```java
.param("customerMessage", customerMessage)
```

↓

```
Internet isn't working
```

---

# النتيجة

الـ AI يستقبل

```text
A customer named Ali has sent the following message:

"My internet isn't working"

Write a polite email...
```

---

# ليه Spring عمل Prompt Templates؟

تخيل عندك

40 Prompt.

لو كلهم

```java
"""
.....
"""
```

هيبقوا داخل Controller.

الكود هيبقى بشع.

الأفضل

```
resources

    promptTemplates

        summarize.st

        email.st

        translate.st

        review.st

        support.st
```

كل Prompt فى File لوحده.

زى SQL Scripts.

---

# طيب لو غيرت

```
email.st
```

هل أغير Java؟

لا.

تغير الـ Prompt فقط.

وده Separation of Concerns.

---

# عايزك تعمل حاجة قبل أي تعديل

بدل ما تشوف 500 فقط، ابعتلي **الـ Stack Trace بالكامل** اللي ظاهر في الـ Console (أول Exception وآخر `Caused by`)، لأن الـ 500 مجرد نتيجة، أما السبب الحقيقي فسيكون واضحًا في الـ Logs.

لكن من الكود اللي أرسلته، **أول حاجتين هراجعهم فورًا هما:**

1. ✅ صحح المسار:
    
    ```java
    @Value("classpath:promptTemplates/userPromptTemplate.st")
    ```
    
    بدل `promptTeemplates`.
    
2. ✅ اقفل علامة الاقتباس في ملف `userPromptTemplate.st`:
    
    ```text
    "{customerMessage}"
    ```
    

---

## اقتراح

بما إنك ناوي تتعمق في Spring AI، أنا أقدر أشرحلك كل Topic بنفس الأسلوب ده (إيه اللي بيحصل داخل Spring AI، وإيه اللي بيحصل داخل الـ LLM، وإيه اللي بيحصل في الخلفية)، بحيث مش تحفظ API بس، لكن تفهم الـ Architecture كمان. ده هيفرق جدًا لما نوصل لـ **Advisors، RAG، Memory، Structured Output، وAI Agents**.