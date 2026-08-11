## 🎯 عنوان الجزء

**Types of Generative AI Models: LLMs vs Diffusion Models**

الجزء ده بيعمل حاجة أساسية جدًا: بيقسم **Generative AI Models** حسب نوع الـcontent اللي بتولده، وبعدها بيركز على الفرق بين **LLMs** و **Diffusion Models**.

---

## 1. Generative AI مش Model واحد

**Generative AI** هو اسم عام لمجموعة من الـmodels، وكل model عنده **different capabilities and strengths**.

ممكن model يولد:

- Text
    
- Images
    
- Videos
    
- Audio
    
- Code
    

يعني:

```text
Generative AI
│
├── LLMs
│   └── Mainly Text
│
└── Diffusion Models
    ├── Images
    ├── Videos
    ├── Audio
    └── ...
```

---

# 2. Large Language Models — LLMs

> **Large Language Models (LLMs) are Generative AI models primarily used to understand and generate text.**

أمثلة ذكرها السكريبت:

- ChatGPT
    
- LLaMA
    

وبنستخدمهم في حاجات زي:

```text
Question Answering
Code Generation
Text Generation
```

مثلاً:

```text
You:
Write a Java REST API.

LLM:
@RestController
public class CourseController {
    ...
}
```

---

# 3. Diffusion Models

> **Diffusion Models are Generative AI models that can generate content such as images, videos, and audio.**

أمثلة موجودة في السكريبت:

- DALL-E 2
    
- Midjourney
    
- Sora
    

والـcourse هيشرحها بشكل بسيط، لكن **التركيز الأساسي هيكون على LLMs** لأن استخدام الـLLMs أكبر في الـenterprise applications حسب السكريبت.

يعني بالنسبة لنا كمطورين Spring AI:

```text
Generative AI
      │
      ├── Diffusion Models
      │      └── Images / Video / Audio
      │
      └── LLMs ⭐
             └── Text / Code / Q&A
```

**إذن هنركز على LLMs لأنها الأهم في الجزء الأكبر من الكورس.**

---

# 4. طيب Diffusion Model بيولد صورة إزاي؟

دي أهم فكرة في الجزء كله.

الـDiffusion Model بيتعلم من **الكثير من الصور**.

أثناء الـtraining بيعمل حاجة اسمها:

> **Adding Noise**

يعني بياخد صورة أصلية:

```text
🐶
```

ويبدأ يضيف عليها noise تدريجيًا:

```text
Original Image
      ↓
  Add Noise
      ↓
 More Noise
      ↓
 More Noise
      ↓
Almost Complete Noise
```

في النهاية الصورة الأصلية بتتحول تقريبًا إلى **random noise**.

الفكرة إن الـmodel أثناء التدريب يتعلم:

> "إزاي الصورة النظيفة ممكن تكون اتكونت من الـnoise ده؟"

---

# 5. وقت الـGeneration بيعمل العكس

لما تقول للـmodel:

```text
Generate a picture of a dog.
```

يبدأ من:

```text
Random Noise
     ↓
Remove Noise
     ↓
Remove More Noise
     ↓
Remove More Noise
     ↓
🐶 Dog
```

العملية دي اسمها:

> **Denoising**

يعني بدل ما كان:

```text
Image → Noise
```

أثناء التدريب، وقت التوليد بيعمل تقريبًا:

```text
Noise → Denoising → Image
```

وبعد عدة iterations، الصورة بتظهر تدريجيًا.

---

# 🧠 الخلاصة اللي تحفظها

### Generative AI

> **A broad category of AI models capable of generating new content.**

### LLM

> **A Generative AI model primarily focused on understanding and generating text.**

### Diffusion Model

> **A Generative AI model that generates content through a process of gradually removing noise.**

والـflow المهم:

```text
             Generative AI
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
      LLMs             Diffusion Models
        │                   │
      Text             Image/Video/Audio
        │                   │
 ChatGPT, LLaMA     DALL-E, Midjourney, Sora
```

### ⭐ بالنسبة للكورس بتاعنا

إحنا **مش محتاجين نتعمق في Diffusion Models** دلوقتي.

المهم تفهم إن:

**Generative AI ≠ LLM**

بل:

**LLM is one type of Generative AI model.**

والجزء الجاي هو الأهم بالنسبة لك: **هنبدأ ندخل جوه الـLLM ونفهم بيشتغل إزاي internally**، وده اللي هيفيدك جدًا لما نبدأ **Spring AI + Tokens + Embeddings + RAG + Advisors**.