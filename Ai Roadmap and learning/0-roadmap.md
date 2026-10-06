
---

# 🧠 AI Engineering Roadmap — Java/Spring Boot Developer

## 🎯 الهدف النهائي

في نهاية الـ 12 أسبوع تكون قادر تعمل:

```text
User
  ↓
Spring Boot API
  ↓
LLM
  ↓
Prompt / Structured Output
  ↓
RAG
  ↓
Tools / Function Calling
  ↓
Agents
  ↓
Multi-Agent
  ↓
Evaluation + Observability
  ↓
Docker / Production
```

وتقدر تعمل مشروع Portfolio محترم باستخدام:

- Java
    
- Spring Boot
    
- Spring AI
    
- PostgreSQL
    
- pgvector
    
- Redis
    
- Docker
    
- Ollama
    
- LLM APIs
    
- RAG
    
- Tool Calling
    
- Agents
    
- MCP
    

---

# 🗓️ المرحلة الأولى — LLM Fundamentals

## الأسبوع 1 — Python for AI + HTTP APIs

### 🎯 الهدف

مش مطلوب تبقى Python Developer.

مطلوب فقط تكون قادر تقرأ وتكتب Python عشان تفهم أغلب محتوى الـ AI.

---

### Day 1 — Python Basics

ذاكر:

- Variables
    
- Data Types
    
- Strings
    
- Numbers
    
- Boolean
    
- Lists
    
- Tuples
    
- Sets
    
- Dictionaries
    
- if / elif / else
    

مثال:

```python
user = {
    "name": "Ali",
    "age": 23,
    "skills": ["Java", "Spring Boot"]
}
```

### Day 2 — Control Flow

ذاكر:

- for
    
- while
    
- break
    
- continue
    
- range
    

### Day 3 — Functions

ذاكر:

```python
def calculate_price(price, tax):
    return price + price * tax
```

وافهم:

- Parameters
    
- Return
    
- Default parameters
    
- `*args`
    
- `**kwargs`
    
- Lambda بشكل بسيط
    

### Day 4 — OOP

مش محتاج تتعمق.

افهم:

- Class
    
- Object
    
- Constructor
    
- Inheritance
    
- Encapsulation
    
- Method
    

### Day 5 — Python للـ Data

ذاكر:

- List comprehension
    
- Dictionary comprehension
    
- JSON
    
- Reading files
    
- Environment variables
    

### Day 6 — HTTP + REST APIs

راجع اللي أنت أصلًا عارفه من Spring:

```text
GET
POST
PUT
PATCH
DELETE
```

وافهم:

```text
Request
 ↓
API
 ↓
Response
```

### Day 7 — تطبيق

اعمل Python script:

```text
User enters question
        ↓
Python
        ↓
HTTP Request
        ↓
LLM API
        ↓
Response
```

### ✅ أنت خلصت الأسبوع لما تقدر:

- تكتب Python script صغير بدون مساعدة كبيرة.
    
- تقرأ JSON.
    
- تعمل HTTP request.
    
- تتعامل مع API response.
    
- تفهم API key / environment variable.
    

---

# الأسبوع 2 — كيف الـ LLM بيشتغل؟

هنا تبدأ **AI Fundamentals**.

## 2.1 ما هو LLM؟

افهم:

```text
Text
 ↓
Tokens
 ↓
Neural Network
 ↓
Probability
 ↓
Next Token
 ↓
Next Token
 ↓
...
```

لا تدخل Deep Learning رياضي.

المطلوب:

> تفهم ماذا يحدث Conceptually.

---

## 2.2 Tokens

ذاكر:

- What is a token?
    
- Tokenization
    
- Token ≠ word
    
- Input tokens
    
- Output tokens
    
- Token cost
    

افهم مثلًا:

```text
"Hello Ali"
```

ممكن تتقسم إلى tokens مختلفة حسب الـ tokenizer.

---

## 2.3 Context Window

افهم:

```text
System Prompt
+
Conversation
+
User Message
+
Retrieved Documents
+
Tools
        ↓
Context Window
        ↓
LLM
```

ذاكر:

- Context window
    
- Input context
    
- Output limits
    
- Why long conversations become expensive
    
- Context management
    

---

# 2.4 Temperature

افهم:

```text
Temperature = randomness
```

مش تحفظ رقم معين.

افهم:

```text
Low temperature
→ predictable

High temperature
→ more diverse
```

---

# 2.5 Embeddings — Concept فقط

لسه مش هتطبق.

افهم:

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

مثال conceptual:

```text
"Java Spring Boot"
        ↓
[0.12, -0.44, 0.87, ...]
```

وافهم ليه بنعمل ده.

---

# 2.6 Transformer — Conceptual Understanding

هنا تشوف:

### لازم تفهم:

- Neural Network بشكل بسيط
    
- Transformer
    
- Attention
    
- Self-Attention
    
- Encoder
    
- Decoder
    
- Transformer-based models
    

### مش مطلوب:

❌ Calculus

❌ Matrix derivations

❌ بناء Transformer من الصفر

---

## 📚 مذاكرة الأسبوع

بالترتيب:

1. Karpathy — Intro to Large Language Models
    
2. 3Blue1Brown — Transformers
    
3. اقرأ documentation بسيطة عن tokens/context/temperature.
    

---

## 🧪 تطبيق الأسبوع

استخدم:

**Ollama**

وشغل model محلي.

جرب:

```text
same prompt
temperature = low

same prompt
temperature = high
```

وشوف الفرق.

### ✅ خلصت الأسبوع لما تقدر تشرح:

> إيه هو LLM؟

> إيه هو Token؟

> إيه هو Context Window؟

> إيه هو Embedding؟

> إيه هو Transformer؟

> إيه هو Temperature؟

---

# الأسبوع 3 — Prompt Engineering

هنا تبدأ تتعامل مع الـ LLM بشكل حقيقي.

---

# 3.1 Prompt Structure

افهم:

```text
System
User
Assistant
```

مثال:

```text
System:
You are a Java interviewer.

User:
Explain dependency injection.
```

---

# 3.2 Zero-shot Prompting

```text
Explain Java Generics.
```

---

# 3.3 Few-shot Prompting

تدي examples:

```text
Example:
Input → ...
Output → ...

Example:
Input → ...
Output → ...

Now:
Input → ...
```

---

# 3.4 Role Prompting

مثال:

```text
You are a senior Java interviewer.
```

افهم إمتى ده مفيد وإمتى مش لازم.

---

# 3.5 Structured Output

دي **مهمة جدًا** للـ backend developers.

بدل:

```text
Ali is 23 and knows Java.
```

عايز:

```json
{
  "name": "Ali",
  "age": 23,
  "skills": ["Java"]
}
```

افهم:

- JSON output
    
- Schema
    
- Validation
    
- Parsing
    
- Failure handling
    

---

# 3.6 Prompt Injection

مهم جدًا.

افهم:

```text
User input
≠
Trusted instruction
```

وتعلم:

- Prompt injection
    
- System prompt leakage
    
- Untrusted content
    
- Basic defenses
    

---

# 3.7 Evaluation of Prompts

بدل:

> "الـ prompt ده شكله حلو."

اعمل:

```text
Prompt A
→ 20 test cases

Prompt B
→ 20 test cases
```

وقارن النتائج.

دي أول خطوة ناحية **AI Evaluation**.

---

# 🧪 Project

اعمل:

## AI Java Interviewer

```text
User
 ↓
Spring Boot
 ↓
LLM
 ↓
Question Generator
 ↓
Structured JSON
 ↓
Database
```

يحفظ:

- Question
    
- Difficulty
    
- Topic
    
- Expected Answer
    

---

# الأسبوع 4 — LLM Integration with Spring Boot

هنا نرجع للـ stack بتاعك.

## المطلوب

تتعامل مع LLM من:

```text
Spring Boot
        ↓
Spring AI
        ↓
Ollama / OpenAI-compatible API
```

---

# ذاكر:

### 4.1 Spring AI Basics

افهم:

- ChatClient
    
- ChatModel
    
- Prompt
    
- Message
    
- UserMessage
    
- SystemMessage
    

---

### 4.2 Calling LLM

اعمل endpoint:

```http
POST /api/chat
```

```json
{
  "message": "Explain dependency injection"
}
```

---

### 4.3 Conversation

اعمل:

```text
Conversation
 ├── User message
 ├── AI response
 ├── User message
 └── AI response
```

وخزنها في PostgreSQL.

---

### 4.4 Streaming

بدل:

```text
Request
   ↓
WAIT
   ↓
Complete response
```

اعمل:

```text
Request
 ↓
Token
 ↓
Token
 ↓
Token
 ↓
Token
```

وافهم:

- Streaming
    
- SSE
    
- Reactive response concept
    

---

### 4.5 Error Handling

لازم تتعامل مع:

- API timeout
    
- Rate limit
    
- Invalid response
    
- Model unavailable
    
- Token limit
    

---

# 🧪 مشروع الشهر الأول

## AI Chat Application

### Backend

```text
Spring Boot
Spring AI
PostgreSQL
Redis
Docker
```

### Features

- Authentication
    
- Conversations
    
- Messages
    
- LLM chat
    
- Streaming
    
- Conversation history
    
- Redis caching
    
- Error handling
    

### Architecture

```text
Controller
    ↓
Application Service
    ↓
AI Service
    ↓
Spring AI
    ↓
LLM
```

---

# 🧠 المرحلة الثانية — RAG

دي أهم مرحلة في الرود ماب بالنسبة لك.

# الأسبوع 5 — Embeddings

الآن نرجع للـ Embeddings بجد.

---

## ذاكر:

### 5.1 What is Embedding?

```text
Document
   ↓
Embedding Model
   ↓
Vector
```

---

### 5.2 Semantic Similarity

افهم الفرق بين:

```text
Keyword Search
```

و:

```text
Semantic Search
```

مثال:

```text
"How can I secure my Spring application?"
```

ممكن تلاقي:

```text
"Spring Security authentication"
```

حتى لو الكلمات مش متطابقة.

---

# 5.3 Cosine Similarity

مش محتاج رياضيات عميقة.

افهم فقط:

```text
Vector A
Vector B

        ↓

Similarity Score
```

مثال conceptual:

```text
0.95 → very similar
0.70 → somewhat similar
0.10 → unrelated
```

الأرقام هنا مجرد توضيح للمفهوم، مش thresholds ثابتة.

---

# 5.4 Chunking

دي مهمة جدًا.

لو عندك PDF:

```text
10,000 words
```

مش هتحطه كله مرة واحدة.

تقسم:

```text
Document
 ↓
Chunk 1
Chunk 2
Chunk 3
Chunk 4
...
```

ذاكر:

- Chunk size
    
- Chunk overlap
    
- Semantic chunking
    
- Fixed-size chunking
    

---

# الأسبوع 6 — Vector Database

بما إنك Java + PostgreSQL:

## ابدأ بـ pgvector

Architecture:

```text
PostgreSQL
    +
pgvector
```

ذاكر:

- Vector column
    
- Vector similarity search
    
- Indexing
    
- Metadata
    

---

## Metadata

مثال:

```json
{
  "document": "spring-security.pdf",
  "page": 12,
  "section": "JWT"
}
```

ودي مهمة جدًا في الـ RAG.

---

# 🧪 تطبيق الأسبوع

اعمل:

```text
PDF
 ↓
Text extraction
 ↓
Chunking
 ↓
Embedding
 ↓
PostgreSQL + pgvector
```

وبعدين:

```text
User Question
 ↓
Embedding
 ↓
Vector Search
 ↓
Top K chunks
```

---

# الأسبوع 7 — Build RAG

دلوقتي نركب كل حاجة.

# RAG =

```text
Retrieval
+
Augmented Generation
```

Architecture:

```text
User Question
      ↓
Embedding
      ↓
Vector Search
      ↓
Relevant Chunks
      ↓
Prompt
      ↓
LLM
      ↓
Answer
```

---

## ذاكر مكونات RAG بالترتيب

### 1. Document ingestion

```text
PDF
 ↓
Text
```

### 2. Chunking

```text
Text
 ↓
Chunks
```

### 3. Embedding

```text
Chunks
 ↓
Vectors
```

### 4. Storage

```text
PostgreSQL + pgvector
```

### 5. Retrieval

```text
Question
 ↓
Vector Search
 ↓
Top K
```

### 6. Prompt Augmentation

```text
Question
+
Retrieved Context
```

### 7. Generation

```text
LLM
 ↓
Answer
```

---

# الأسبوع 8 — RAG Advanced

هنا تبدأ تنتقل من:

> RAG شغال

إلى:

> RAG كويس.

---

## 8.1 Metadata Filtering

مثال:

```text
department = IT
document_type = policy
year = 2026
```

---

## 8.2 Top-K

افهم:

```text
Top 3
Top 5
Top 10
```

وإزاي الاختيار يؤثر على الـ context.

---

# 8.3 Hybrid Search

بدل:

```text
Vector Search
```

فقط:

```text
Keyword Search
+
Vector Search
```

افهم لماذا قد تحتاج الاثنين.

---

# 8.4 Reranking

```text
Retrieve 20
      ↓
Reranker
      ↓
Best 5
      ↓
LLM
```

---

# 8.5 RAG Evaluation

دي **مهمة جدًا**.

ذاكر:

- Retrieval quality
    
- Answer correctness
    
- Groundedness
    
- Relevance
    
- Faithfulness
    
- Recall / Precision بشكل مفاهيمي
    

---

# 🧪 مشروع الشهر الثاني

## "Ask Your Documents"

Features:

```text
User
 ↓
Upload PDF
 ↓
Process
 ↓
Chunk
 ↓
Embedding
 ↓
pgvector
```

وبعدين:

```text
Question
 ↓
Retrieve
 ↓
LLM
 ↓
Answer
```

والأهم:

```text
Answer
+
Sources
```

مثال:

> According to the document...

```text
Sources:
- Spring Security Guide
- Page 14
- Section: Authentication
```

---

# 🤖 المرحلة الثالثة — AI Agents

# الأسبوع 9 — Tool / Function Calling

دي النقلة من:

```text
LLM = يجاوب
```

إلى:

```text
LLM = يقرر يستخدم Tool
```

مثال:

```text
User:
What is my current balance?
```

LLM:

```text
I need account_balance tool
```

Backend:

```java
getAccountBalance(userId)
```

ثم:

```text
Tool Result
 ↓
LLM
 ↓
Final Answer
```

---

# ذاكر:

- Tool
    
- Function calling
    
- Tool schema
    
- Tool arguments
    
- Tool result
    
- Tool errors
    
- Multiple tools
    

---

# 🧪 أهم تمرين

**اعمل Tool Calling من غير Agent framework.**

يعني بنفسك:

```text
User
 ↓
LLM
 ↓
Does it request tool?
 ↓
YES
 ↓
Execute Java method
 ↓
Return result to LLM
 ↓
Final answer
```

لو فهمت ده كويس، هتفهم الـ Agents بسهولة جدًا.

---

# الأسبوع 10 — Agents

دلوقتي:

## What is an Agent?

مش مجرد:

```text
LLM + Tool
```

افهم الـ loop:

```text
Goal
 ↓
Reason / Decide
 ↓
Action
 ↓
Observation
 ↓
Decide again
 ↓
...
 ↓
Final Answer
```

---

# ذاكر Agent Patterns

## 1. ReAct

```text
Reason
 ↓
Act
 ↓
Observe
 ↓
Reason
```

افهمه Conceptually، ومش لازم تدخل في تفاصيل hidden chain-of-thought.

---

## 2. Planning

```text
Goal
 ↓
Plan
 ↓
Task 1
Task 2
Task 3
```

---

## 3. Reflection / Review

```text
Generate
 ↓
Review
 ↓
Improve
```

---

## 4. Memory

افهم الفرق بين:

### Short-term memory

```text
Current conversation
```

### Long-term memory

```text
Stored user information
```

وإمتى تستخدم كل واحد.

---

# الأسبوع 11 — Multi-Agent + MCP

هنا تبدأ المستوى المتقدم.

## Multi-Agent

بدل Agent واحد:

```text
                    ┌── RAG Agent
User → Router Agent ├── Action Agent
                    └── Reviewer Agent
```

---

# افهم:

### Router

يحدد:

```text
Which agent should handle this?
```

### Specialist Agent

متخصص في مهمة.

### Reviewer

يراجع output.

---

# MCP

ذاكر:

## Model Context Protocol

وافهم:

```text
AI Model
   ↓
MCP
   ↓
Tools / Resources
```

ذاكر:

- MCP Client
    
- MCP Server
    
- Tools
    
- Resources
    
- Prompts
    
- Why MCP exists
    
- MCP vs normal function calling
    

**مش مطلوب تحفظ implementation بالكامل في البداية.**

---

# الأسبوع 12 — Production AI

دي المرحلة اللي هتفرق مشروعك عن Demo.

---

# 12.1 Guardrails

ذاكر:

- Input validation
    
- Output validation
    
- Prompt injection protection
    
- Tool permissions
    
- Rate limiting
    
- Sensitive data handling
    

---

# 12.2 Observability

لازم تعرف:

```text
What did the user ask?
Which model?
How many tokens?
Which tools?
How long?
What failed?
```

اعمل logging لـ:

```text
Request
Model
Latency
Token usage
Tool calls
Errors
```

---

# 12.3 Evaluation

اعمل Dataset:

```text
Question
Expected Answer
Actual Answer
```

وبعدين قيس:

```text
Correctness
Relevance
Groundedness
Retrieval quality
```

---

# 12.4 Cost Management

افهم:

```text
Input tokens
+
Output tokens
=
Cost
```

وإزاي تقلل:

- unnecessary context
    
- huge prompts
    
- unnecessary calls
    
- repeated embeddings
    

---

# 12.5 Docker

اعمل:

```text
Spring Boot
PostgreSQL
pgvector
Redis
Ollama
```

داخل:

```text
docker-compose.yml
```

---

# 🏗️ المشروع النهائي

أنا شايف إن المشروع النهائي المناسب جدًا للـ stack بتاعك يكون بالشكل ده:

# AI Knowledge & Operations Assistant

```text
                    ┌── RAG Agent
                    │
User → Router Agent ├── Action Agent
                    │
                    └── Reviewer Agent
```

---

## RAG Agent

يقرأ:

```text
Company Documents
Policies
Technical Docs
FAQs
```

ويجاوب منها.

---

## Action Agent

يقدر يعمل Tools:

```text
getUser()
getTask()
createTask()
updateTask()
searchDatabase()
```

---

## Reviewer Agent

يعمل:

```text
Answer
 ↓
Review
 ↓
Check:
- grounded?
- correct?
- safe?
 ↓
Final Answer
```

---

# 🧱 Architecture

استخدم الـ architecture اللي أنت بتحبه:

```text
src/main/java

├── feature
│   ├── chat
│   ├── document
│   ├── rag
│   ├── agent
│   ├── tool
│   └── user
│
├── shared
│   ├── security
│   ├── exception
│   ├── config
│   └── infrastructure
```

وممكن تطبق:

```text
Clean Architecture
+
Vertical Slice / Feature-based
```

---

# 🧭 ترتيب المذاكرة اليومي

وده مهم جدًا.

ما تعملش:

```text
3 ساعات فيديو
0 كود
```

اعمل:

### لو عندك ساعتين:

```text
30 min → Learn
30 min → Notes
60 min → Code
```

### لو عندك 3 ساعات:

```text
45 min → Theory
30 min → Notes
90 min → Implementation
15 min → Review
```

---

# 📝 طريقة مذاكرة كل Topic

لكل موضوع امشي بالنظام ده:

```text
1. What is it?
        ↓
2. Why do we need it?
        ↓
3. How does it work?
        ↓
4. Simple example
        ↓
5. Java/Spring implementation
        ↓
6. Common problems
        ↓
7. Interview questions
        ↓
8. Mini project
```

مثلاً **RAG**:

```text
What is RAG?
      ↓
Why RAG?
      ↓
How Retrieval works?
      ↓
Embeddings
      ↓
Vector DB
      ↓
Spring AI implementation
      ↓
Build RAG
      ↓
Evaluate RAG
      ↓
Explain it in interview
```

---

# 📚 ترتيب المصادر

مش عايزك تجمع 50 Course.

لكل Topic استخدم:

### 1️⃣ Understand

فيديو/شرح concept.

### 2️⃣ Official Documentation

خصوصًا:

- Spring AI
    
- Ollama
    
- PostgreSQL/pgvector
    
- OpenAI/Anthropic docs
    
- MCP documentation
    

### 3️⃣ Build

اكتب الكود بنفسك.

### 4️⃣ Explain

اقفل كل حاجة وحاول تشرح الموضوع من دماغك.

---

# 🚫 حاجات متذاكرهاش دلوقتي

متدخلش في:

```text
❌ Training LLM from scratch
❌ Deep Mathematics
❌ PyTorch deeply
❌ TensorFlow deeply
❌ Fine-tuning deeply
❌ CUDA
❌ GPU optimization
❌ Building neural networks from scratch
```

دي مش الأولوية لهدفك كـ **Java/Spring AI Engineer**.

---

# ⭐ الأولويات

لو وقتك ضيق، الترتيب ده:

```text
⭐⭐⭐⭐⭐ LLM Fundamentals
⭐⭐⭐⭐⭐ Prompt Engineering
⭐⭐⭐⭐⭐ Spring AI
⭐⭐⭐⭐⭐ Embeddings
⭐⭐⭐⭐⭐ RAG
⭐⭐⭐⭐⭐ Tool Calling
⭐⭐⭐⭐⭐ Agents
⭐⭐⭐⭐ Evaluation
⭐⭐⭐⭐ Observability
⭐⭐⭐ MCP
⭐⭐⭐ Fine-tuning
⭐ Deep Learning Math
```

---

# 🗺️ الخريطة النهائية في سطر واحد

```text
Python Basics
      ↓
LLM Fundamentals
      ↓
Prompt Engineering
      ↓
Spring AI
      ↓
Embeddings
      ↓
Vector DB
      ↓
RAG
      ↓
Advanced RAG
      ↓
Tool Calling
      ↓
Agents
      ↓
Multi-Agent
      ↓
MCP
      ↓
Evaluation
      ↓
Observability
      ↓
Guardrails
      ↓
Docker
      ↓
Production AI
```

**وأهم نقطة:** ما تبدأش المشروع النهائي من أول شهر. اعمل **Mini Project صغير في كل مرحلة**، وفي الآخر اجمع المفاهيم كلها في مشروع واحد. كده الـ AI عندك هيبقى مبني فوق خبرتك في Spring Boot، مش مسار منفصل عن الـ backend.

ولو عايز، أقدر أحول الـ12 أسبوع دول كمان إلى **خطة يوم بيوم (Day 1 → Day 84)**، وكل يوم أحدد لك _تذاكر إيه + تشوف إيه + تعمل إيه بالكود + إيه المطلوب تكون فاهمه قبل ما تنتقل لليوم اللي بعده_.