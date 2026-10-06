# 2.1 — ما هو الـ LLM؟ وكيف بيشتغل؟ 🤖

بص، الهدف من التوبيك ده **مش إنك تتعلم Deep Learning أو رياضيات**.  
الهدف إنك لو اتسألت:

> **How does an LLM generate a response?**

تعرف تشرح الرحلة كاملة من أول الـ text لحد الـ response بشكل مفهوم.

---

# 1. يعني إيه LLM؟

**LLM = Large Language Model**

هو Model اتدرّب على كمية ضخمة جدًا من الـ text عشان يتعلم **patterns والعلاقات بين الكلمات والـ tokens**، ويقدر بناءً عليها يتوقع الـ token التالي.

أشهر أمثلة:

- GPT
    
- Claude
    
- Gemini
    
- Llama
    

لكن مهم جدًا:

> الـ LLM مش بيشتغل كأنه Database حافظ الإجابة وبيطلعها.

هو بيعمل **prediction** للـ next token بناءً على الـ context الموجود قدامه.

---

# 2. الصورة الكبيرة

لو كتبت:

```text
Java is a programming
```

الـ LLM ممكن يتوقع:

```text
language
```

لكن داخليًا الرحلة تقريبًا:

```text
Your Text
   ↓
Tokenization
   ↓
Tokens
   ↓
Neural Network / Transformer
   ↓
Probabilities
   ↓
Choose Next Token
   ↓
Add Token to Context
   ↓
Predict Again
   ↓
...
   ↓
Final Response
```

ودي أهم صورة تحفظها في التوبيك كله.

---

# 3. أول خطوة: Text → Tokens

الـ LLM مش بيتعامل مع الـ text بالشكل اللي إحنا شايفينه.

بيستخدم حاجة اسمها:

> **Tokenization**

والـ **Token** مش بالضرورة يكون كلمة كاملة.

مثلاً:

```text
I love programming
```

ممكن تتحول بشكل مبسط إلى:

```text
["I", " love", " program", "ming"]
```

وده مجرد مثال توضيحي؛ الـ tokenizer الحقيقي ممكن يقسمها بشكل مختلف.

### ليه بنستخدم Tokens؟

لأن الـ model بيحتاج يحول اللغة الطبيعية إلى representation يقدر يعالجها computationally.

فبدل:

```text
I love programming
```

عنده IDs مثلًا:

```text
[40, 291, 8452, 123]
```

الـ IDs دي مجرد أرقام تمثل الـ tokens في الـ vocabulary.

---

# 4. يعني إيه Vocabulary؟

الـ LLM عنده **Vocabulary** تحتوي على عدد كبير من الـ tokens.

مثلاً بشكل مبسط:

```text
"Java"      → 1254
"Spring"    → 8231
"Boot"      → 4412
"program"   → 982
"ing"       → 731
```

لما يدخل له:

```text
I love Java
```

الـ tokenizer يحولها إلى IDs.

مثلاً:

```text
"I"      → 40
" love"  → 291
" Java"  → 1254
```

---

# 5. بعد كده إيه؟

هنا يدخل الجزء المهم:

## Neural Network

الـ tokens بتدخل إلى الـ **Neural Network**.

والـ architecture الأساسية المستخدمة في الـ LLMs الحديثة هي:

> **Transformer**

مش مطلوب منك حاليًا تدخل في تفاصيل الـ Transformer architecture.

لكن لازم تعرف إن الـ Transformer هو اللي بيساعد الـ model يفهم **العلاقات والـ context بين الـ tokens**.

---

# 6. يعني إيه Context؟

دي نقطة مهمة جدًا.

لو قلت:

```text
The bank is near the river.
```

كلمة:

```text
bank
```

معناها هنا غالبًا بنك.

لكن:

```text
I sat on the bank of the river.
```

هنا المقصود ضفة النهر.

الـ model بيستخدم الـ **context** عشان يحدد العلاقات والمعنى المحتمل للكلام.

---

# 7. الـ LLM مش بيتوقع الإجابة كلها مرة واحدة

دي من أهم الحاجات اللي لازم تفهمها.

لو سألت:

```text
What is Java?
```

الـ model مش بيعمل:

```text
Generate entire answer
        ↓
Return answer
```

لكن تقريبًا:

```text
What is Java?
       ↓
Predict next token
       ↓
"Java"
       ↓
Predict next token
       ↓
"is"
       ↓
Predict next token
       ↓
"a"
       ↓
Predict next token
       ↓
"programming"
       ↓
...
```

يعني **Token by Token**.

---

# 8. طيب إزاي بيختار الـ Next Token؟

هنا ندخل على:

## Probability

افترض إن الـ input:

```text
Java is a
```

الـ model ممكن ينتج احتمالات تقريبية:

```text
programming → 0.70
program       → 0.15
language      → 0.10
coffee        → 0.01
...
```

يعني الـ model بيقول:

> بناءً على الـ context الحالي، دي احتمالات الـ next token.

ثم يتم اختيار token وفقًا لآلية الـ decoding والإعدادات المستخدمة.

لو اختار:

```text
programming
```

يبقى الـ context أصبح:

```text
Java is a programming
```

وبعدين يعمل prediction مرة تانية.

---

# 9. العملية بتتكرر

مثلاً:

```text
Input:

Java is a
```

### Prediction #1

```text
programming
```

يبقى:

```text
Java is a programming
```

### Prediction #2

```text
language
```

يبقى:

```text
Java is a programming language
```

### Prediction #3

```text
used
```

يبقى:

```text
Java is a programming language used
```

وهكذا.

لحد ما الـ model يقرر إنه وصل لنهاية مناسبة للـ response.

---

# 10. هل أعلى Probability دائمًا هي اللي بتتختار؟

**مش بالضرورة.**

هنا ممكن تسمع عن:

### Temperature

الـ Temperature بتأثر على **مدى عشوائية الاختيار من الاحتمالات**.

بشكل مبسط:

### Low Temperature

الـ model يميل لاختيارات أكثر predictable.

مثلاً:

```text
Java is a programming ...
```

غالبًا:

```text
language
```

### Higher Temperature

ممكن يسمح باختيارات أقل احتمالًا، وبالتالي responses أكثر تنوعًا.

مثلاً في creative writing ده ممكن يكون مفيد.

> المهم حاليًا تفهم إن الـ model عنده probability distribution، والـ decoding settings بتأثر على طريقة اختيار الـ next token.

---

# 11. طيب الـ LLM "فاهم" الكلام؟

هنا لازم نفرق بين المعنى العملي والفلسفي.

من الناحية الهندسية:

الـ LLM اتعلم **patterns وعلاقات إحصائية معقدة جدًا** من بيانات التدريب.

فمش صح تتخيله كـ:

```text
Database of answers
```

ولا تتخيله ببساطة كـ:

```text
if question == "What is Java?"
    return stored_answer
```

هو model بيحسب representations واحتمالات بناءً على الـ context.

---

# 12. مثال كامل

لو كتبت:

```text
Explain dependency injection in Spring
```

الرحلة بشكل مبسط:

```text
"Explain dependency injection in Spring"
                ↓
          Tokenization
                ↓
      [tokens + token IDs]
                ↓
          Transformer
                ↓
    Understand relationships/context
                ↓
     Probability distribution
                ↓
      Next token prediction
                ↓
           "Dependency"
                ↓
     Predict next token again
                ↓
           "Injection"
                ↓
               ...
                ↓
         Complete response
```

---

# 13. طب فين الـ Training في الموضوع؟

دي نقطة مهمة جدًا.

الـ LLM الأول بيتم **Training** عليه كمية ضخمة من البيانات.

بشكل مبسط جدًا:

```text
Training Data
     ↓
Text
     ↓
Tokens
     ↓
Model tries to predict next token
     ↓
Compare prediction with actual token
     ↓
Calculate error
     ↓
Update model parameters
     ↓
Repeat... billions/millions of times
```

مثال:

الـ training text:

```text
Java is a programming language.
```

الـ model يشوف:

```text
Java is a programming
```

ويحاول يتوقع:

```text
language
```

لو توقع حاجة غلط، يحصل **error** ويتم تعديل الـ model parameters أثناء التدريب.

مع تكرار التدريب على بيانات ضخمة، يتعلم patterns معقدة جدًا.

---

# 14. Training vs Inference

دي نقطة **مهمة جدًا للـ AI Engineer**.

## Training

هنا بنعلم الـ model:

```text
Data
 ↓
Model
 ↓
Prediction
 ↓
Error
 ↓
Update Parameters
 ↓
Repeat
```

وده مكلف جدًا computationally.

---

## Inference

ده لما أنت تستخدم Model جاهز.

مثلاً:

```text
User Prompt
     ↓
Trained LLM
     ↓
Next Token
     ↓
Next Token
     ↓
...
     ↓
Response
```

هنا **مش بنغير الـ model parameters** أثناء الاستخدام العادي.

---

# 15. أهم مصطلحين

## Parameters

دي القيم اللي الـ model اتعلمها أثناء الـ training.

ممكن تسمع:

```text
7B parameters
70B parameters
```

الـ B هنا = Billion.

يعني:

```text
7B = 7 billion parameters
```

لكن **عدد الـ parameters لوحده مش مقياس كافي للحكم على جودة model**.

---

# 16. Context Window

دي مهمة جدًا وهتقابلك كتير في الـ AI Engineering.

الـ model عنده حد أقصى لكمية الـ tokens اللي يقدر يعالجها في context معين.

مثلاً بشكل توضيحي:

```text
Context Window
┌─────────────────────────────┐
│ System Instructions         │
│ Conversation                │
│ User Prompt                 │
│ Retrieved Documents         │
│ Previous Messages           │
└─────────────────────────────┘
```

كل ده بيتحول إلى tokens.

لو كمية الـ tokens تعدت الحد المسموح به، مش هتقدر تحط كل المحتوى في نفس الـ context.

وده هيبقى مهم جدًا لما تدرس:

> **RAG**

لأننا بدل ما نحط كل الـ documents في الـ prompt، بنجيب فقط الـ relevant information.

---

# 17. علاقة الكلام ده بالـ RAG

هتحتاج الجزئية دي بعدين جدًا.

افترض عندك:

```text
Company Documents
     ↓
10,000 Documents
```

والـ user سأل:

```text
What is our vacation policy?
```

مش منطقي تحط الـ 10,000 documents في الـ prompt.

RAG بيعمل تقريبًا:

```text
User Question
      ↓
Retrieve Relevant Documents
      ↓
Relevant Context
      ↓
LLM
      ↓
Answer
```

فالـ LLM نفسه **مش لازم يكون عارف محتوى الـ company documents من التدريب**.

إحنا بنديله المعلومات relevant وقت الـ inference.

---

# 18. الخلاصة اللي تحفظها للانترفيو

لو اتسألت:

> **How does an LLM work?**

تقدر تقول:

> An LLM is a neural network, typically based on the Transformer architecture, that processes text as tokens. Given a sequence of tokens, it predicts a probability distribution over possible next tokens. A decoding strategy selects the next token, which is then added to the context, and the process is repeated token by token until the response is complete.

ودي إجابة ممتازة كبداية.

---

# 🧠 الـ Mental Model اللي عايزك تحفظه

```text
                USER PROMPT
                     ↓
                 Tokenizer
                     ↓
                   Tokens
                     ↓
                Transformer
                     ↓
          Probability Distribution
                     ↓
              Next Token
                     ↓
          Add to Existing Context
                     ↓
          Probability Distribution
                     ↓
              Next Token
                     ↓
                    ...
                     ↓
               Final Response
```

### والـ Training كان قبل كل ده:

```text
Huge Training Data
        ↓
      Tokens
        ↓
     Model
        ↓
 Predict Next Token
        ↓
 Calculate Error
        ↓
 Update Parameters
        ↓
       Repeat
```

## المطلوب منك في الأسبوع ده

مش مطلوب تحفظ رياضيات أو تفاصيل الـ Transformer الداخلية دلوقتي.

اتأكد إنك فاهم دول كويس:

-  LLM = Large Language Model
    
-  Text → Tokens
    
-  Tokens represented by IDs
    
-  Transformer processes the context
    
-  LLM predicts the next token
    
-  Prediction is probabilistic
    
-  Response generated token-by-token
    
-  Temperature affects sampling/decoding behavior
    
-  Training ≠ Inference
    
-  Parameters
    
-  Context Window
    
-  RAG later provides relevant external context to the LLM
    

**وده بالضبط المستوى المطلوب من 2.1.**  
بعده الطبيعي جدًا في الرودماب إننا نمسك **2.2 — Tokens & Tokenization** بشكل أعمق شوية، لأنه أساس مهم جدًا قبل ما ندخل على الـ Embeddings والـ RAG.