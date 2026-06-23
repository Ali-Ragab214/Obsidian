آه كده فهمت قصدك.L Enineer.

---

# ذاكر إيه؟

## المرحلة 1: فهم الـ LLMs

لازم تفهم:

### LLM يعني إيه؟

- Token
    
- Context Window
    
- Temperature
    
- System Prompt
    
- User Prompt
    
- Assistant Message
    

ليه النموذج بيهبد؟

ليه بينسى؟

ليه بيرجع إجابات مختلفة؟

---

## المرحلة 2: Prompt Engineering

دي أول حاجة فعلاً.

تعلم:

### Zero Shot

```text
Translate this text to Arabic
```

---

### Few Shot

```text
Example 1:
...

Example 2:
...

Now solve:
...
```

---

### Chain Of Thought

---

### Structured Output

JSON

مثال:

```text
Return response as JSON

{
   "title":"",
   "price":""
}
```

---

## المرحلة 3: APIs

اتعلم ازاي تتعامل مع:

[OpenAI Platform](https://platform.openai.com/?utm_source=chatgpt.com)

---

في Java:

```java
POST /chat/completions
```

وتفهم:

- Messages
    
- Model
    
- Temperature
    
- Max Tokens
    

---

# المرحلة 4: AI داخل مشاريع Spring

هنا تبدأ المتعة.

مثال:

## E-commerce

- وصف منتجات تلقائي
    
- Chatbot
    

---

## LMS

- تلخيص كورسات
    
- توليد Quiz
    
- توليد أسئلة
    

---

## Hospital System

- تلخيص تقارير
    
- استخراج بيانات
    

---

# المرحلة 5: RAG

دي أهم حاجة في السوق حاليًا.

لازم تفهم:

```text
PDF
 ↓
Chunking
 ↓
Embedding
 ↓
Vector DB
 ↓
Search
 ↓
LLM
```

---

ذاكر:

### Embeddings

يعني إيه؟

---

### Vector Database

مثل:

[Qdrant](https://qdrant.tech/?utm_source=chatgpt.com)

أو

[ChromaDB](https://www.trychroma.com/?utm_source=chatgpt.com)

---

## مشروع

PDF Chat

زي ChatGPT على ملفاتك.

---

# المرحلة 6: AI Agents

دي النقلة الكبيرة.

افهم إن Agent =

```text
LLM
+
Memory
+
Tools
+
Reasoning
```

---

مثال:

المستخدم يقول:

```text
احجزلي فندق في القاهرة
```

الـ Agent:

1. يفهم الطلب
    
2. يبحث
    
3. يقارن
    
4. يرد
    

---

## لازم تتعلم

### Tool Calling

النموذج يقرر ينادي Function.

مثال:

```java
getWeather()
```

أو

```java
sendEmail()
```

---

### Function Calling

---

### MCP

Model Context Protocol

وده بقى منتشر جدًا.

---

# المرحلة 7: Frameworks

ركز على:

### LangChain

[LangChain Docs](https://python.langchain.com/?utm_source=chatgpt.com)

---

### LangGraph

[LangGraph Docs](https://langchain-ai.github.io/langgraph/?utm_source=chatgpt.com)

---

لو هتكمل Java:

### Spring AI

[Spring AI](https://spring.io/projects/spring-ai?utm_source=chatgpt.com)

دي أنصحك بيها جدًا.

---

# المرحلة 8: Automation

وده الجزء اللي سألت عنه.

الـ AI + Automation = فلوس كتير حاليًا.

أمثلة:

### Email Agent

يقرأ الإيميلات ويرد.

---

### CV Screening Agent

يفرز CVs.

---

### Customer Support Agent

يرد على العملاء.

---

### Sales Agent

يكلم العملاء.

---

### Content Agent

يكتب المحتوى.

---

### Research Agent

يبحث ويلخص.

---

# لو أنا مكانك

بما إنك Java/Spring Boot

هعمل الترتيب ده:

### شهر 1

- Prompt Engineering
    
- OpenAI APIs
    
- Spring AI
    

---

### شهر 2

- Function Calling
    
- Structured Output
    
- JSON Responses
    

---

### شهر 3

- RAG
    
- Embeddings
    
- Vector Databases
    

---

### شهر 4

- Agents
    
- LangGraph
    
- MCP
    

---

### شهر 5

- Automation Workflows
    
- Multi-Agent Systems
    

---

### شهر 6

ابنِ مشروع قوي مثل:

**AI Study Assistant**

- Spring Boot
    
- PostgreSQL
    
- Redis
    
- Spring AI
    
- RAG
    
- PDF Chat
    
- Quiz Generator
    
- Agent System
    
- Docker
    

لو مشيت في الطريق ده 6 شهور بتركيز، هتبقى قادر تضيف AI Features لأي مشروع Spring Boot تقريبًا، وتبني Agents وAutomations من غير ما تدخل في التعقيد الأكاديمي للـ Machine Learning.