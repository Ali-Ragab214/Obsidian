

---

# 1) Token

## Analogy

تخيل إنك بتبعت ملف كبير على الشبكة.

قبل ما الملف يتبعت بيتقسم لـ Packets.

الـ LLM بيعمل نفس الفكرة مع النص، بس بدل Packets اسمها Tokens.

---

## التعريف

الـ Token هو أصغر وحدة بيفهمها الموديل.

مش شرط تكون كلمة كاملة.

ممكن تكون:

- كلمة
    
- جزء من كلمة
    
- حرف
    
- علامة ترقيم
    

---

## مثال

الجملة:

```text
I love programming
```

ممكن تتقسم:

```text
["I", " love", " program", "ming"]
```

لاحظ إن كلمة programming اتقسمت جزئين.

---

الجملة العربية:

```text
انا بحب البرمجة
```

ممكن تبقى:

```text
["انا", " بحب", " البر", "مجة"]
```

أو تقسيمة مختلفة حسب الـ tokenizer.

---

## ليه ده مهم؟

لأن كل موديل عنده تكلفة وحدود مبنية على عدد الـ Tokens مش عدد الكلمات.

مثال:

```text
1000 كلمة ≠ 1000 Token
```

ممكن يبقوا:

```text
1300 Token
```

أو أكتر.

---

## Prompt للتجربة

```text
Count approximately how many tokens are in this text and explain why:

"Spring Boot is amazing for backend development"
```

---

## غلطة شائعة

مطورين كتير فاكرين:

```text
1 word = 1 token
```

وده غلط.

---

# 2) Context Window

## Analogy

تخيل عندك Whiteboard في مكتبك.

الموديل بيشوف اللي مكتوب على السبورة بس.

لو السبورة اتمليت:

لازم تمسح جزء قديم عشان تكتب جديد.

---

## التعريف

الـ Context Window هو كمية الـ Tokens اللي الموديل يقدر يشوفها في نفس الوقت.

---

مثال:

لو الموديل بيدعم:

```text
128K Tokens
```

فهو يقدر يشوف:

- System Prompt
    
- User Messages
    
- Previous Messages
    
- Current Message
    

كلهم مع بعض داخل الـ 128K.

---

## لو تخطيت الحد؟

الأجزاء القديمة غالباً بتتقص.

فيبدأ ينسى.

فتلاقيه يقول:

```text
معلش مش فاكر المشروع اللي اتكلمنا عنه
```

---

## Prompt للتجربة

```text
Remember this:

Project Name = CoursePilot

I will ask you later.
```

وبعدين بعد شات طويل اسأله:

```text
What was the project name?
```

هتفهم فكرة الـ Context عملي.

---

## غلطة شائعة

ناس كتير فاكرة إن الموديل عنده Memory دائمة.

الحقيقة:

```text
Memory != Context
```

---

# 3) Temperature

Temperature ≈ "مدى التزام الموديل بأكتر إجابة متوقعة".  
كل ما تعليها → تنوع وإبداع أكتر، لكن دقة وثبات أقل.  
كل ما توطيها → ثبات ودقة أكتر، لكن إبداع أقل.
كل ما تعلي ال temperature   هيديك اجابات برا الصندوق 

## Analogy

تخيل مطور Junior بيحل مشكلة.

### Temperature = 0

هيمشي بالحرف على الحل المعروف.

### Temperature = 1.5

هيبدأ يبدع ويخترع حلول جديدة.

---

## التعريف

Temperature بتتحكم في العشوائية أثناء اختيار الكلمات.

---

### Low Temperature

مثلاً:

```text
0.1
```

نتائج:

- ثابتة
    
- متوقعة
    
- دقيقة
    

---

### High Temperature

مثلاً:

```text
1.2
```

نتائج:

- إبداعية
    
- متنوعة
    
- أحياناً أقل دقة
    

---

## مثال

Prompt:

```text
Give me a startup idea for education.
```

### Temperature = 0.1

```text
Online course management platform.
```

---

### Temperature = 1.2

```text
AI mentor that simulates famous historical scientists
and teaches students through role-playing.
```

---

## إمتى أستخدم إيه؟

### كود

```text
0 - 0.3
```

### Documentation

```text
0.3 - 0.7
```

### Brainstorming

```text
0.8 - 1.5
```

---

## Prompt للتجربة

```text
Give me 5 startup ideas in healthcare.
```

جرّبه بدرجات مختلفة.

---

## غلطة شائعة

رفع الـ Temperature مش معناه:

```text
Smartness++
```

هو فقط:

```text
Randomness++
```

---

# 4) System / User / Assistant Messages

## Analogy

تخيل Spring Application.

---

### System Message

زي:

```java
@Configuration
```

قواعد عامة للتطبيق.

---

### User Message

زي:

```java
REST Request
```

الطلب الحالي.

---

### Assistant Message

زي:

```java
Response
```

الرد الناتج.

---

## التعريف

### System

يحدد شخصية وسلوك الموديل.

مثال:

```text
You are a senior Java architect.
Always answer in Arabic.
```

---

### User

طلب المستخدم.

```text
Explain Redis.
```

---

### Assistant

الرد السابق.

```text
Redis is an in-memory datastore...
```

---

## شكل المحادثة

```text
System
↓
User
↓
Assistant
↓
User
↓
Assistant
```

---

## Prompt للتجربة

System:

```text
You are a strict Java interviewer.
```

User:

```text
Explain Dependency Injection.
```

---

## غلطة شائعة

كتير من الناس بتحط كل حاجة في User Message.

بينما التعليمات الدائمة الأفضل تبقى System Message.

---

# 5) Zero-shot vs Few-shot Prompting

## Analogy

### Zero-shot

مدير قالك:

```text
اعمل Task دي
```

وخلاص.

---

### Few-shot

مدير ورّاك مثالين الأول.

بعدين قالك:

```text
اعمل زيهم
```

---

# Zero-shot

Prompt:

```text
Classify sentiment:

"I love this phone"
```

---

Output:

```text
Positive
```

---

# Few-shot

Prompt:

```text
Text: I love this laptop
Sentiment: Positive

Text: This product is terrible
Sentiment: Negative

Text: The service was excellent
Sentiment:
```

---

Output:

```text
Positive
```

---

## إمتى أستخدم Few-shot؟

لما:

- الفورمات مهم
    
- المهمة متخصصة
    
- النتائج مش ثابتة
    

---

## Prompt للتجربة

```text
Convert user requests into JSON.

Example:
Book flight to Cairo

Output:
{
 "intent":"book_flight",
 "city":"Cairo"
}

Now:
Book flight to Dubai
```

---

## غلطة شائعة

ناس كتير تدي 20 مثال.

غالباً:

```text
2-5 examples
```

كفاية.

---

# 6) Chain of Thought (CoT)

## Analogy

تخيل Senior Developer.

بدل ما يقول:

```text
الحل هو X
```

يشرح:

```text
1- المشكلة
2- التحليل
3- القرار
4- الحل
```

---

## التعريف

إنك تطلب من الموديل يفكر خطوة بخطوة.

---

## قبل

```text
What's 17 × 24?
```

---

## بعد

```text
Solve step by step.
```

---

هيبقى:

```text
17 × 20 = 340

17 × 4 = 68

340 + 68 = 408
```

---

## مثال برمجي

بدل:

```text
Optimize this query.
```

اكتب:

```text
Analyze this SQL query step by step.
Identify bottlenecks.
Then suggest optimizations.
```

---

## Prompt للتجربة

```text
Act as a senior software architect.

Think step by step:

1. Analyze requirements
2. Identify entities
3. Suggest architecture
4. Explain tradeoffs

Build a hospital management system.
```

---

## غلطة شائعة

ناس كتير تفتكر إن CoT بيحسن كل حاجة.

الحقيقة:

- ممتاز للتحليل
    
- ممتاز للتخطيط
    
- ممتاز للمسائل المعقدة
    

لكن مش لازم لكل Prompt.

---

# 7) Structured Output (JSON)

## Analogy

تخيل API Response.

بدل ما يرجع Text حر:

```text
The user wants to buy a laptop...
```

يرجع DTO ثابت.

---

## التعريف

إنك تجبر الموديل يطلع Output منظم.

غالباً:

```json
{
  "intent": "",
  "entities": []
}
```

---

## مثال Prompt

```text
Extract information from the text.

Return ONLY valid JSON.

Schema:

{
  "name": "string",
  "age": "number",
  "skills": ["string"]
}

Text:

Ali is 24 years old and knows Java and React.
```

---

## Output

```json
{
  "name": "Ali",
  "age": 24,
  "skills": ["Java", "React"]
}
```

---

## لتحسين الالتزام

اكتب:

```text
Return ONLY valid JSON.
Do not add explanations.
Do not use markdown.
```

---

## استخدام مهم جداً ليك

لو عامل Agent أو Spring Backend:

```java
ChatGPT
   ↓
JSON
   ↓
Jackson
   ↓
DTO
```

بدل Parsing لنص عشوائي.

---

## Prompt للتجربة

```text
Extract task information.

Return ONLY valid JSON.

{
  "title":"",
  "priority":"",
  "dueDate":""
}

Create a task to finish Redis integration tomorrow with high priority.
```

---

## غلطة شائعة

المطورين أحياناً يطلبوا:

```text
Give me JSON
```

وبعدين يسيبوا الـ Schema.

الموديل ساعتها يخترع Structure من عنده.

لازم تحدد الـ Schema بوضوح.

---

# ملخص سريع

|Concept|تعريف من سطر واحد|مثال سريع|
|---|---|---|
|Token|أصغر وحدة بيفهمها الموديل|"programming" → "program" + "ming"|
|Context Window|حجم المعلومات اللي الموديل يقدر يشوفها مرة واحدة|128K Tokens|
|Temperature|درجة العشوائية في الرد|0.1 دقيق، 1.2 إبداعي|
|System Message|قواعد وسلوك الموديل|"You are a Java architect"|
|User Message|طلب المستخدم الحالي|"Explain Redis"|
|Assistant Message|ردود الموديل السابقة|الرد اللي طلع قبل كده|
|Zero-shot|طلب بدون أمثلة|"Classify sentiment"|
|Few-shot|طلب مع أمثلة|مثالين ثم المهمة|
|Chain of Thought|تحليل خطوة بخطوة|"Think step by step"|
|Structured Output|إخراج منظم (JSON)|DTO-like JSON response|

لو هدفك الحقيقي هو بناء Agents واستخدام الـ AI داخل مشاريع Spring Boot، فبعد فهم السبع مفاهيم دول الخطوة اللي بعدها مباشرة هي: **Embeddings → Vector Database → RAG → Function Calling/Tools → AI Agents → MCP**. دول أهم 5 مواضيع هتنقلك من "بكتب Prompt" إلى "ببني أنظمة AI فعلية".