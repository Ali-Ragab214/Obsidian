بص يا علي، تخيل إن الـ LLM ده **موظف جديد**، وإنت كل مرة بتكلمه لازم تديه كل المعلومات من أول وجديد لأنه **بينسى** بعد كل Request.

يعني كل Request بيكون عبارة عن **شات كامل** بيتبعت للـ LLM.

مثلاً:

```
System : أنت مدرس Java محترف.

User : اشرح Spring AI.

Assistant : Spring AI هو Framework ...

User : اديني مثال.
```

كل ده بيتبعت مرة واحدة للـ LLM.

وده معناه إن كل Message ليها **Role**.

---

# 1) UserMessage

ده أسهل واحد.

هو الرسالة اللي المستخدم كتبها.

مثال:

```
أنا:
اشرح Spring AI
```

في Spring AI:

```java
UserMessage user =
    new UserMessage("اشرح Spring AI");
```

---

# 2) SystemMessage

ده مش بيكتبه المستخدم.

**ده إنت كمبرمج اللي بتكتبه.**

وده بيقول للـ AI:

> "أنت مين؟"

مثال

```
أنت مدرس Java.
جاوب بالعربي.
استخدم أمثلة.
```

يعني بدل ما كل مستخدم يقول:

> جاوب بالعربي

إنت بتحطها مرة واحدة في الـ System Message.

في Spring AI:

```java
SystemMessage system =
    new SystemMessage("""
You are a Java instructor.
Answer in Arabic.
Use examples.
""");
```

يبقى أول حاجة الـ AI بيقراها هي دي.

---

## مثال

لو عندك

```
System:
أنت مدرس أطفال.
```

ثم المستخدم قال

```
اشرح الـ JVM.
```

هيشرحها بطريقة سهلة.

لكن لو

```
System:
أنت مهندس Senior.
```

هيشرحها بمستوى أعلى.

إذن

> **System Message = شخصية وتعليمات الـ AI.**

---

# 3) AssistantMessage

ده رد الـ AI القديم.

هتقولي

> "هو ليه أبعتهوله تاني؟"

عشان الـ LLM **بينسى**.

بص للمثال.

أنت قلت

```
اسمي علي.
```

الـ AI رد

```
تشرفت يا علي.
```

بعد دقيقة قلت

```
اسمي إيه؟
```

لو بعت بس

```
اسمي إيه؟
```

هيقول

```
لا أعرف.
```

لكن لو بعت

```
User:
اسمي علي.

Assistant:
تشرفت يا علي.

User:
اسمي إيه؟
```

هيقول

```
اسمك علي.
```

إذن

**AssistantMessage هو ردود الـ AI القديمة علشان يحافظ على سياق المحادثة.**

في Spring AI

```java
AssistantMessage assistant =
    new AssistantMessage("تشرفت يا علي.");
```

---

# 4) Tool / Function Message

ده أصعب واحد.

خلينا نفترض إن المستخدم قال

```
كام درجة الحرارة في القاهرة؟
```

الـ AI ميعرفش.

فيقول

```
لازم أستخدم Weather API.
```

هنا التطبيق بتاعك هو اللي يروح يسأل الـ API.

الـ API رجع

```
38°
Sunny
```

بعدها التطبيق يبعت النتيجة للـ AI.

```
Tool:
درجة الحرارة 38
الجو Sunny
```

الـ AI يرد

```
النهارده في القاهرة 38 درجة والجو مشمس.
```

إذن

Tool Message هو **نتيجة أداة خارجية**.

زي

- Database
    
- REST API
    
- Search
    
- Calculator
    
- File
    

---

# دلوقتي Spring AI Classes

تقريبًا هتستخدم دول كتير.

## UserMessage

```java
UserMessage user =
    new UserMessage("Hello");
```

يمثل كلام المستخدم.

---

## SystemMessage

```java
SystemMessage system =
    new SystemMessage("You are a Java instructor.");
```

يمثل تعليمات الـ AI.

---

## AssistantMessage

```java
AssistantMessage assistant =
    new AssistantMessage("Hello Ali");
```

يمثل الردود السابقة للـ AI.

---

## ToolResponseMessage

```java
ToolResponseMessage
```

يمثل ناتج Tool أو Function.

---

## Prompt

ده أهم Class.

هو عبارة عن **Container** بيجمع كل الـ Messages.

يعني

```
Prompt
│
├── SystemMessage
├── UserMessage
├── AssistantMessage
└── ToolResponseMessage
```

مثال

```java
Prompt prompt = new Prompt(List.of(
    new SystemMessage("You are a Java teacher."),
    new UserMessage("Explain Spring AI.")
));
```

بعد كده تبعته

```java
ChatResponse response =
        chatModel.call(prompt);
```

---

# ChatModel

ده الـ Interface اللي بيتكلم مع الـ LLM.

```java
ChatResponse response =
        chatModel.call(prompt);
```

كأنك بتقول

```
يا ChatGPT

خد الـ Prompt ده

ورد عليا.
```

---

# ChatResponse

ده الرد اللي رجع من الـ LLM.

```java
ChatResponse response =
        chatModel.call(prompt);
```

الرد هيكون

```
Spring AI is ...
```

---

# احفظها بالطريقة دي

```
أنا (User)
        │
        ▼
UserMessage
        │

المبرمج
        │
        ▼
SystemMessage
        │

كلهم يتحطوا داخل
        │
        ▼
Prompt
        │
        ▼
ChatModel.call(prompt)
        │
        ▼
LLM (OpenAI / Claude / Gemini)
        │
        ▼
ChatResponse
        │
        ▼
AssistantMessage (الرد)
```

## الخلاصة اللي لازم تفتكرها

- **UserMessage** → كلام المستخدم.
    
- **SystemMessage** → تعليمات المبرمج للـ AI (شخصيته، أسلوبه، القواعد).
    
- **AssistantMessage** → ردود الـ AI السابقة للحفاظ على سياق المحادثة.
    
- **ToolResponseMessage** → نتائج أدوات خارجية (API، قاعدة بيانات، بحث...).
    
- **Prompt** → يجمع كل الـ Messages في Request واحد.
    
- **ChatModel** → يرسل الـ Prompt إلى الـ LLM.
    
- **ChatResponse** → يستقبل الرد من الـ LLM.
    

لو هتكمل Spring AI، ففهم **Prompt + Messages + ChatModel** كويس جدًا؛ لأنهم الأساس اللي هتبني عليه بعد كده الـ Prompt Engineering، والـ Chat Memory، والـ RAG، والـ AI Agents.