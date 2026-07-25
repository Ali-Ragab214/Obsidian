![[Pasted image 20260725131004.png]]
package com.example.openai.controller;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

public class PromptStuffingController {
    private  final ChatClient chatClient;
    public PromptStuffingController(ChatClient chatClient)
    {
        this.chatClient=chatClient;
    }

    @Value("classpath:promptTemplates/systemPromptTemplate.st")
    Resource systemPromptTemplate;

    @GetMapping("/prompt-stuffing")
    public String promptStuffing(
            @RequestParam("message") String message
    )
    {
        return  chatClient
                .prompt()
                .system(systemPromptTemplate)
                .user(message)
                .call()
                .content();

    }

}
# نستخدم هذا لما يكون الداتا حجمها صغير مش كبير وبنروح للراج لما تكون الداتا كبيره 

# exp part 1 
تمام، خليني أشرحهولك بالطريقة اللي هتثبت في دماغك سواء للمذاكرة أو للـ Interview.

---

# Prompt Stuffing

## الفكرة في سطر واحد

**Prompt Stuffing هو إنك تدّي الـ LLM كل المعلومات اللي هيحتاجها قبل ما تجاوبه على سؤال.**

يعني بدل ما الـ AI يعتمد على المعلومات اللي اتدرب عليها، أنت بتبعتله المعلومات مع الـ Prompt.

فكر فيها كأنك داخل امتحان Open Book.

بدل ما المدرس يقولك:

> جاوب من ذاكرتك.

بيقولك:

> خد الكتاب معاك، واقرأ منه وبعدها جاوب.

وده بالظبط Prompt Stuffing.

---

# ليه بنستخدمه؟

لأن الـ LLM مش عارف بيانات شركتك.

مثلاً:

شركة عندها HR Policy

وفيها:

- 18 Leave
    
- 30 Days Notice
    
- 9-6 Working Hours
    

الحاجات دي OpenAI عمره ما اتدرب عليها.

فلو سألت:

> How many annual leaves do employees have?

هيقول معلومات عامة أو ممكن يخمن.

لكن لو أنت بعتله الـ Policy الأول

هيجاوب بالإجابة الصح.

---

# إيه اللي بيحصل؟

بدل ما يحصل كده

```text
User
   │
   ▼
LLM
```

بيحصل كده

```text
System Prompt (HR Policy)
        │
        ▼
User Question
        │
        ▼
LLM
```

يعني الـ LLM بيقرأ الأول الـ HR Policy.

بعدها يقرأ سؤال المستخدم.

بعدها يجاوب.

---

# مثال

بدل

```text
User:
How many leave days?
```

تبعت

```text
System:

18 leave days
30 days notice
9-6 working hours

User:

How many leave days?
```

الـ LLM هيقرأ الاتنين مع بعض.

---

# هو بيحفظ المعلومات؟

لا.

دي نقطة مهمة.

الـ Model **مش بيتعلم** منها.

هو بيستخدمها في الـ Request الحالي فقط.

بعد ما الـ Request يخلص...

كل المعلومات دي بتختفي.

---

# إمتى أستخدم Prompt Stuffing؟

لما تكون البيانات:

✅ صغيرة

زي

- HR Policy
    
- Company Rules
    
- Restaurant Menu
    
- FAQ
    
- شروط البنك
    
- تعليمات معينة
    

يعني Text صغير.

---

# إمتى مينفعش؟

لو عندك

- PDF 300 صفحة
    
- 10000 Product
    
- مليون Customer
    
- Documentation كبيرة
    

مينفعش تحطهم في Prompt.

لأن:

- Token Limit
    
- Cost أعلى
    
- Response أبطأ
    

وساعتها بنستخدم **RAG**.

---

# الكود بتاعك بيعمل إيه؟

```java
@Value("classpath:promptTemplates/systemPromptTemplate.st")
Resource systemPromptTemplate;
```

هنا Spring AI بيقرأ الملف.

مثلاً الملف اسمه

```text
systemPromptTemplate.st
```

وجواه

```text
You are an HR Assistant.

18 leave days.

Notice period 30 days.

Working hours 9-6.

Only answer HR questions.
```

---

بعدها

```java
chatClient
    .prompt()
```

بدأت تبني الـ Prompt.

---

```java
.system(systemPromptTemplate)
```

هنا حصل Prompt Stuffing.

يعني كل الكلام الموجود في الملف اتحط داخل الـ **System Prompt**.

---

بعدها

```java
.user(message)
```

أخد سؤال المستخدم.

مثلاً

```text
How many leave days?
```

---

فالـ Prompt النهائي اللي راح لـ OpenAI تقريبًا هيكون بالشكل ده:

```text
System:

You are an HR assistant.

18 leave days.

Notice period 30 days.

Working hours 9-6.

Only answer HR questions.

------------------------

User:

How many leave days?
```

وده اسمه Prompt Stuffing.

---

بعدها

```java
.call()
```

يبعت الـ Prompt للـ LLM.

---

```java
.content();
```

يرجع النص فقط.

مثلاً

```text
Employees receive 18 paid leave days annually.
```

---

# ليه استخدمنا System Prompt؟

لأن الـ System Prompt بيحدد:

- شخصية الـ AI.
    
- القواعد اللي يمشي عليها.
    
- المعلومات اللي يعتمد عليها.
    
- حدوده (Scope).
    

يعني هنا أنت بتقوله:

> أنت HR Assistant فقط.

ولو حد سأله:

```text
Who won the World Cup?
```

هيقول

```text
Sorry, I only answer HR policy questions.
```

وده لأن الـ Prompt كان محشي بالمعلومات والتعليمات.

---

# ملخص سريع للانترفيو

**What is Prompt Stuffing?**

> Prompt Stuffing is a technique where additional context or reference information is included directly in the prompt before sending it to the language model. The model uses this context to generate a more accurate response. It is suitable for small, static datasets such as company policies or business rules, but it is not suitable for large datasets due to token limits and higher cost.

## احفظ الجملة دي

> **Prompt Stuffing = Put the required context inside the prompt before asking the LLM to answer.**

ولو سألك بعدها: **"طيب لو البيانات كبيرة؟"** تبقى الإجابة مباشرة:

> **We don't use Prompt Stuffing for large knowledge bases. Instead, we use RAG, which retrieves only the relevant information and injects it into the prompt before sending it to the LLM.**

دي تعتبر الصورة الكاملة للمفهوم، وهي اللي غالبًا هتحتاجها في الشرح أو في أي Interview عن Spring AI.