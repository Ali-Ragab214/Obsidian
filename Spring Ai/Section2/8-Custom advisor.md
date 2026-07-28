حلو جدًا. السكشن ده من أهم السكاشن في Spring AI، لأنه بيخليك تفهم إن **Advisor مش مجرد Logging**، لا، ده يعتبر **Middleware / Interceptor** للـ AI Request Lifecycle.

هشرحلك الموضوع من أول الصفر، وهقولك الفرق بين الـ Built-in Advisors والـ Custom Advisor وإمتى تستخدم كل واحد، وإيه اللي بيحصل جوه Spring AI.

---

# أولاً... ليه أصلاً في Advisors ؟

بص على الفلو ده:

```
Controller
      │
      ▼
ChatClient
      │
      ▼
Advisor 1
      │
      ▼
Advisor 2
      │
      ▼
Advisor 3
      │
      ▼
OpenAI
      │
      ▼
Advisor 3
      │
      ▼
Advisor 2
      │
      ▼
Advisor 1
      │
      ▼
Controller
```

يعني قبل ما الريكوست يروح للـ LLM

كل Advisor ياخد فرصة يعمل حاجة.

ولما الريسبونس يرجع

كل Advisor ياخد فرصة يعدل عليه.

وده تقريباً نفس فكرة

- Spring MVC Interceptor
    
- Servlet Filter
    
- Spring Security Filter Chain
    
- Aspect (AOP)
    

لكن مخصوص للـ AI.

---

# مثال

المستخدم كتب

```
Summarize this PDF
```

قبل ما يروح لـ OpenAI

Advisor ممكن

- يضيف System Prompt
    
- يعمل Logging
    
- يحسب Cost
    
- يعمل Audit
    
- يعمل Rate Limit
    
- يعمل Authentication
    
- يعمل RAG
    
- يضيف Conversation History
    

بعد ما الريسبونس يرجع

Advisor يقدر

- يحسب Tokens
    
- يحسب Cost
    
- يخزن Conversation
    
- يعمل Metrics
    
- يعمل Caching
    

كل ده قبل ما الريسبونس يوصل للـ Controller.

---

# الـ Built-in Advisors

Spring AI جايبلك شوية Advisors جاهزين.

مثلاً

## 1) SimpleLoggerAdvisor

وده اللي استخدمناه.

بيعمل

```
Incoming Prompt

Outgoing Response
```

فقط.

يعني Logging.

---

## 2) MessageChatMemoryAdvisor

ده بيضيف

Conversation Memory.

يعني

```
User:
Hi

AI:
Hello

User:
Who am I?
```

هو بيبعت المحادثة كلها للـ LLM.

---

## 3) QuestionAnswerAdvisor

ده أشهر Advisor.

وده مسئول عن

RAG

هو اللي

- يبحث في Vector DB
    
- يجيب Documents
    
- يحطهم في الـ Prompt
    

---

## 4) PromptChatMemoryAdvisor

بيعمل Memory باستخدام Prompt.

---

## 5) VectorStoreChatMemoryAdvisor

بيخزن History داخل Vector Database.

---

يعني Spring AI وفرلك شوية Advisors جاهزين.

لكن ساعات المشروع يحتاج Logic خاص بيك.

هنا بييجي دور

Custom Advisor.

---

# هو بيبني Custom Advisor ليه؟

هو قال

عايز أعرف

كل Request

استهلك كام Token.

ليه؟

لأن OpenAI

بتحاسبك على

Tokens

مش

Requests.

---

مثلاً

```
100 Requests
```

مش مهم.

المهم

```
كل Request كام Token
```

---

OpenAI Pricing

مثلاً

```
Prompt Tokens

100

Completion Tokens

50

Total

150
```

لو مليون Token سعره

```
$0.15
```

يبقى كل مرة تعرف

كلفتك كام.

وده مهم جداً للشركات.

---

# ليه الشركات تهتم بالـ Tokens؟

تخيل عندك SaaS.

فيه

10000 User

كل واحد بيستخدم AI.

كل User هيكلفك.

لازم تعرف

```
Ali

100 Tokens

Ahmed

3000 Tokens

Omar

500 Tokens
```

علشان تعرف تحاسب العملاء.

وده اسمه

Usage Tracking

أو

Billing.

---

# إنشاء Advisor

هو عمل Package

```
advisors
```

ثم

```
TokenUsageAuditAdvisor
```

---

# Implement CallAdvisor

```
implements CallAdvisor
```

ليه؟

لأن الريكوست عادي.

مش Streaming.

---

في Spring AI فيه نوعين

```
CallAdvisor

StreamAdvisor
```

---

## CallAdvisor

بيستخدم مع

```
chatClient.prompt().call()
```

يعني

Request

↓

Response

مرة واحدة.

---

## StreamAdvisor

بيستخدم مع

```
stream()
```

يعني

```
ChatGPT Typing...
```

```
Hello
```

بعدها

```
How
```

بعدها

```
Are
```

بعدها

```
You
```

كل كلمة بتيجي لوحدها.

---

# methods المطلوبة

هو عمل Override لـ

```
adviceCall()

getName()

getOrder()
```

---

## getName()

```
TokenUsageAuditAdvisor
```

اسم الـ Advisor.

للتتبع والـ Logging.

---

## getOrder()

دي مهمة جداً.

---

Spring بينفذ Advisors بالترتيب.

مثلاً

```
Order 0

Logger
```

```
Order 1

Token
```

```
Order 2

RAG
```

كل واحد يتنفذ حسب الرقم.

---

لكن خد بالك من نقطة مهمة جدًا.

هو قال

> order = 1 means highest preference

وده **مش دقيق** إذا كان عنده Advisor تاني بـ `order = 0`.

في Spring عمومًا:

**الأرقام الأصغر لها أولوية أعلى**.

يعني:

```
0
```

يتنفذ قبل

```
1
```

ثم

```
2
```

فلو الـ `SimpleLoggerAdvisor` فعلًا `order = 0` والـ `TokenUsageAuditAdvisor` `order = 1`، فالـ Logger هيتنفذ أولًا، ثم الـ Token Advisor، إلا إذا كان يقصد ترتيب مختلف داخل سلسلة الاستدعاء الخاصة بالرد. لذلك خلي المرجع الأساسي هو قاعدة Spring: **Lower order = Higher precedence**.

---

# adviceCall()

ده قلب الـ Advisor.

تقريباً بيبقى بالشكل ده:

```java
public ChatClientResponse adviceCall(
    ChatClientRequest request,
    CallAdvisorChain chain
)
```

---

الـ request

هو

```
Prompt

Options

Messages

Context
```

---

الـ chain

هو

السلسلة اللي هتكمل لباقي الـ Advisors ثم الـ LLM.

---

# أهم سطر

```
chain.nextCall(request)
```

ده أهم سطر.

ليه؟

لأنه بيقول

```
كمّل
```

وروّح للـ Advisor اللي بعدي.

---

لو شلته

ولا Advisor بعدك هيتنفذ.

ولا OpenAI هيتنادى.

يعني انتهى كل حاجة.

وده نفس فكرة

```
filterChain.doFilter()
```

في Servlet Filter.

---

# ليه هو معملش حاجة قبل الـ nextCall ؟

لأن

قبل ما OpenAI يشتغل

لسه

معندناش Tokens.

---

الـ Prompt

لسه متبعتش.

فمستحيل تعرف

```
Prompt Tokens

Completion Tokens
```

---

فهو عمل

```
response = chain.nextCall(request)
```

وبعدها بدأ يقرأ

```
response
```

---

# ChatClientResponse

بعد الـ nextCall

بقى عندك

```
ChatClientResponse
```

جواه

```
ChatResponse
```

---

وجواها

```
metadata
```

---

وجواها

```
usage
```

---

يعني

```
ChatClientResponse

↓

ChatResponse

↓

Metadata

↓

Usage
```

---

# Usage

ده أهم Object.

جواه

```
Prompt Tokens

Completion Tokens

Total Tokens
```

---

مثال

```
Prompt

120
```

```
Completion

40
```

```
Total

160
```

---

# Prompt Tokens

هي

كل اللي انت بعته.

مثلاً

```
Question

+

System Prompt

+

Memory

+

RAG Documents
```

كل ده بيتحسب.

---

# Completion Tokens

كل اللي الـ AI رجعه.

---

# Total

```
Prompt

+

Completion
```

---

# ليه قال إن الكود ده خاص بـ OpenAI؟

لأن كل مزود LLM ممكن يرجع بيانات الاستخدام بطريقة مختلفة.

قد تجد نفس المفهوم (Usage / Token Counts)، لكن مكانه أو تفاصيله تختلف حسب الـ Provider.

لذلك عند تغيير المزود (مثل Anthropic أو Gemini أو Ollama...) لازم تتأكد من شكل الـ Metadata الذي يرجعه.

---

# هو عمل

```
usage.toString()
```

بدل

```
usage.getPromptTokens()

usage.getCompletionTokens()

usage.getTotalTokens()
```

ليه؟

لأن

toString()

بيطبع كله مرة واحدة.

لكن في Production

الأفضل تعمل

```
logger.info(
"Prompt={}",
usage.getPromptTokens()
);
```

وهكذا.

---

# ليه الشركات تخزن الكلام ده؟

مثلاً

```
Table

AI_USAGE
```

```
id

user_id

prompt_tokens

completion_tokens

total_tokens

cost

created_at
```

ومنها تعمل

Dashboard

وتعرف

```
مين صرف أكتر؟

مين أقل؟

مين لازم يتقفل؟

مين تجاوز الحد؟
```

---

# تسجيل الـ Advisor

عنده طريقتين.

## الأولى

داخل Controller

```java
.advisors(
new TokenUsageAuditAdvisor()
)
```

وده ينفع لو عايز الـ Advisor يشتغل على Endpoint معين فقط.

---

## الثانية (الأفضل)

داخل

```
ChatClientConfig
```

```java
.defaultAdvisors(...)
```

وده يخليه يشتغل مع كل طلب يخرج من الـ `ChatClient`.

وده الأنسب لمعظم الـ Cross-Cutting Concerns زي Logging وMetrics وAuditing.

---

# List.of()

بدل

```java
.defaultAdvisors(
logger,
token
)
```

عمل

```java
List.of(

logger,

token
)
```

وده أسهل لما العدد يكبر.

---

# نقطة مهمة قالها عن Prompt Stuffing

لاحظ لما استخدم

Prompt Stuffing

عدد الـ Tokens زاد.

ليه؟

لأن

```
Question

+

كتاب

+

PDF

+

100 صفحة

+

Instructions
```

كل ده بيتحسب Prompt Tokens.

فلو عندك

```
50000 كلمة
```

هتدفع عليها.

---

وده سبب إن

RAG

أفضل.

بدل ما تبعت

100 صفحة

تبعت

3 صفحات

بس.

---

# مقارنة بين الـ Built-in Advisor والـ Custom Advisor

|Built-in Advisor|Custom Advisor|
|---|---|
|جاهز من Spring AI|أنت تكتبه بنفسك|
|وظائف عامة مثل Logging وMemory وRAG|أي منطق خاص بمشروعك|
|لا يحتاج تنفيذ يدوي|تنفذ `CallAdvisor` أو `StreamAdvisor`|
|مناسب لمعظم الاستخدامات الشائعة|مناسب للـ Audit، Billing، Rate Limiting، Analytics، Security، وغيرها|

---

# أفكار ممتازة لـ Custom Advisors في مشروعك

بما إنك شغال على مشروع AI، فدي Advisors لها قيمة حقيقية:

1. **CostAdvisor**: يحسب تكلفة كل Request بالدولار أو بالجنيه بناءً على أسعار النموذج.
    
2. **UserQuotaAdvisor**: يمنع المستخدم من تجاوز حد معين من الـ Tokens يوميًا أو شهريًا.
    
3. **AuditAdvisor**: يسجل المستخدم، الـ Prompt، والوقت، والنتيجة لأغراض المراجعة.
    
4. **PIIRedactionAdvisor**: يخفي أو يشفر البيانات الحساسة (مثل أرقام البطاقات أو أرقام الهوية) قبل إرسالها للـ LLM.
    
5. **LatencyAdvisor**: يقيس الزمن من لحظة إرسال الطلب حتى وصول الرد.
    
6. **PromptValidationAdvisor**: يرفض أو يعدل الـ Prompt إذا خالف سياسات التطبيق.
    
7. **ResponseModerationAdvisor**: يفحص الرد قبل إرجاعه للمستخدم.
    

---

## الخلاصة التي أريدك أن تتذكرها في الانترفيو

لو سألك أحد: **What is an Advisor in Spring AI?**

الإجابة القوية هي:

> **An Advisor in Spring AI is a middleware/interceptor that participates in the lifecycle of an AI request and response. It can inspect or modify the prompt before it reaches the LLM, invoke the next advisor through the advisor chain, and process the response after it returns. Advisors are commonly used for logging, RAG, chat memory, token auditing, security, caching, rate limiting, and other cross-cutting concerns. Spring AI provides built-in advisors, and developers can create custom advisors by implementing `CallAdvisor` or `StreamAdvisor` depending on whether they use synchronous or streaming interactions.**