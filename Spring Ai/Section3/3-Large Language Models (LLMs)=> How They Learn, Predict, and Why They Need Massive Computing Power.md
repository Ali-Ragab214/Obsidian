# 🧠 عنوان الجزء

**Large Language Models (LLMs): How They Learn, Predict, and Why They Need Massive Computing Power**

الجزء ده مهم جدًا لأنه بيديك أول تصور عن **إزاي الـLLM بيتعلم وإيه اللي بيحصل أثناء الـtraining**. هنركز على المفيد ونسيب أرقام وتفاصيل الـbusiness اللي مش هتفرق معاك كمطور.

---

## 1. يعني إيه LLM؟

> **A Large Language Model (LLM) is an AI model trained on massive amounts of text to understand and generate human-like language.**

الـLLMs تعتبر **text experts**؛ اتدربت على كميات ضخمة من:

- Books
    
- Websites
    
- Articles
    
- Conversations
    

وعشان كده تقدر تسألها وتطلب منها:

```text
Answer questions
Generate text
Generate code
Explain concepts
```

---

# 2. ليه اسمها "Large"؟

الـLarge هنا بسبب حاجتين أساسيتين:

### ① Massive Training Data

اتدربت على **billions of words/tokens**.

### ② Billions of Parameters

> **Parameters are learned values inside the model that are adjusted during training to capture patterns in the data.**

ودي هنفهمها بالتفصيل بعدين.

فببساطة:

```text
LLM = Massive Data + Huge Number of Parameters
```

---

# 3. أهم نقطة: Unsupervised Learning

دي من أهم الحاجات في السكريبت.

في **traditional supervised learning**، الإنسان بيجهز البيانات ويعمل لها **labels**.

مثلاً:

```text
"I am happy with the service"
        ↓
     Positive
```

أو:

```text
Image → 🐱
Label → Cat
```

فالـmodel بيتعلم من:

```text
Input + Label
```

---

## لكن LLMs؟

السكريبت بيشرح إن الـLLMs بتستخدم **unsupervised learning** أثناء الـtraining، يعني مفيش حد محتاج يعمل labeling يدوي لكل النصوص.

بدل:

```text
Sentence → Human label
```

الـLLM بيتعلم من **raw text** وبيكتشف الـpatterns بنفسه.

وده أحد أسباب إننا نقدر ندربها على كميات هائلة من النصوص بدون ما البشر يقعدوا يعملوا labeling لكل كلمة وجملة.

> **The model learns patterns by observing large amounts of language.**

---

# 4. طيب الـLLM بيتعلم إزاي؟

دي أهم فكرة في الفيديو كله.

الـLLM أثناء الـtraining بيلعب بشكل مبسط **next-word prediction game**.

مثلاً:

```text
The cat sat on the ...
```

الـmodel يحاول يتوقع:

```text
mat   → 40%
chair → 20%
roof  → 10%
...
```

لو التوقع غلط، الـmodel بيعمل **update للـlearned patterns**.

ويكرر العملية **ملايين/مليارات المرات** أثناء التدريب.

مثال تاني:

```text
I'm going to the store to buy some ...
```

ممكن يتوقع:

```text
milk
bread
apples
groceries
```

وبناءً على الـcontext والبيانات اللي اتدرب عليها، يتعلم إيه الكلمات والأنماط الأكثر احتمالًا.

---

# 5. هل معنى كده إن ChatGPT مجرد Next Word Predictor؟

**على مستوى الـtraining الأساسي، prediction of the next token هو فكرة محورية جدًا.**

لكن متفهمش الموضوع على إنه:

> "هو بيبحث عن إجابة محفوظة ويطلعها."

الفكرة إن الـmodel أثناء التدريب بيتعلم **patterns and relationships in language**.

مثلاً يتعلم إن:

```text
store
 ↙ ↓ ↘
shopping  money  items
```

و:

```text
rain
 ↙ ↓ ↘
clouds  umbrella  wet
```

السكريبت بيسمي ده بشكل مبسط **giant web of connected ideas**.

---

# 6. ليه محتاجين GPUs؟

أثناء الـtraining، الـmodel بيعمل **massive mathematical computations** باستمرار.

والـGPU ممتاز في:

> **Parallel Processing**

الفرق المبسط:

```text
CPU
Few powerful cores
→ General-purpose tasks

GPU
Many smaller cores
→ Massive parallel mathematical operations
```

لذلك تدريب LLMs على scale ضخم بيعتمد بشكل كبير على GPUs.

---

# 7. CPU vs GPU — اللي يهمك كمطور

مش محتاج تحفظ أرقام الـGPUs اللي اتقالت في السكريبت.

احفظ الفكرة:

```text
LLM Training
     ↓
Huge amount of mathematical operations
     ↓
Need massive parallel processing
     ↓
GPUs are highly suitable
```

وده سبب أساسي في إن تدريب الـlarge models مكلف جدًا computationally.

---

# 8. الـTraining بيعمل إيه في النهاية؟

بعد التدريب، الـmodel بيكون اتعلم **patterns and relationships** من كمية ضخمة من النصوص.

مثلاً بشكل مبسط:

```text
"How are you?"
        ↓
learned relationship
        ↓
"Good / I'm fine / ..."
```

أو:

```text
"Java Spring Boot"
        ↓
related concepts
        ↓
REST APIs
JPA
Dependency Injection
Security
...
```

لكن مهم جدًا:

**هو مش بيخزن Database بالشكل التقليدي.**

هو بيخزن **learned patterns داخل parameters**.

---

# 9. تشبيه الطفل 👶

السكريبت عمل analogy لطيف:

الطفل بيسمع:

```text
Good morning
Good morning
Good morning
```

مع الوقت يبدأ يلاحظ:

```text
Good → Morning
```

وكذلك يسمع:

```text
How are you?
```

مرارًا وتكرارًا.

فيتعلم العلاقات والـpatterns الموجودة في اللغة.

الـLLM بيعمل حاجة مشابهة **على scale ضخم جدًا**:

```text
Massive Text
     ↓
Observe Patterns
     ↓
Predict
     ↓
Compare with actual data
     ↓
Update Parameters
     ↓
Repeat
```

---

# 🔥 الخلاصة المهمة جدًا

احفظ الـflow ده:

```text
Massive Text Data
        ↓
     Training
        ↓
Next Token Prediction
        ↓
Calculate Probabilities
        ↓
Update Parameters
        ↓
Learn Language Patterns
        ↓
Trained LLM
```

وبالتالي:

> **An LLM learns patterns from massive amounts of text by repeatedly predicting the next token and adjusting its parameters based on the prediction error.**

ودي تعتبر **أهم جملة في السكريبت كله**.

---

## 🎯 إيه اللي مش محتاج تحفظه؟

السكريبت دخل في أرقام زي:

- GPT-4 اتدرب على كام token.
    
- عدد الـGPUs المحتمل.
    
- تكلفة التدريب.
    
- كام سنة الإنسان يحتاج يقرأ البيانات.
    
- NVIDIA وأسعار الأسهم.
    

**دي أمثلة لإظهار حجم الـLLMs، مش concepts محتاج تحفظها.**

الأهم ليك كمطور Spring AI هو:

**LLM → Training → Parameters → Tokens → Next-token prediction → Probabilities → Patterns → GPU**

والـ**Tokens** والـ**Parameters** بالذات هنقف عليهم بالتفصيل في الأجزاء الجاية، لأنهم هيفسروا حاجات مهمة جدًا بعد كده في **Spring AI، Token Usage، Pricing، Context Window، وLLM behavior**.