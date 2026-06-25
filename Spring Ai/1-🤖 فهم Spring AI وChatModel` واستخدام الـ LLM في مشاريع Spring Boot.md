

أنت دلوقتي دخلت عالم الـ AI Integration مع Spring Boot، وخليني أشرح لك الموضوع من الصفر لحد ما تعرف تستخدمه في مشروع زي SkillHub.

---

# 1. إيه هو Spring AI؟

[Spring AI Documentation](https://docs.spring.io/spring-ai/reference/?utm_source=chatgpt.com)

`Spring AI` هو Framework من Spring بيخلّيك تتعامل مع نماذج الذكاء الاصطناعي (LLMs) بنفس طريقة التعامل مع:

- `JdbcTemplate`
    
- `RestTemplate`
    
- `Spring Data JPA`
    

يعني بدل ما تبعت HTTP Requests يدويًا لـ OpenAI أو Groq، Spring AI بيديك abstraction جاهز.

---

# 2. يعني إيه LLM أصلاً؟

`LLM = Large Language Model`

أمثلة:

- OpenAI Models
    
- Anthropic Models
    
- Google Models
    
- [Groq](https://groq.com/?utm_source=chatgpt.com) models hosting
    

الـ LLM هو المخ اللي بيفهم النصوص ويولد ردود.

---

# 3. شرح الـ Configuration بتاعك

## اسم التطبيق

```properties
spring.application.name=SpringAiCode
```

اسم الـ Spring Application.

---

## API Key

```properties
spring.ai.openai.api-key=YOUR_KEY
```

ده الـ API Key اللي بيخلي تطبيقك يكلم Groq.

---

## Base URL

```properties
spring.ai.openai.base-url=https://api.groq.com/openai
```

هنا أنت بتقول لـ Spring AI:

> بدل ما أكلم OpenAI الرسمي، كلم Groq.

لأن Groq عامل API متوافق مع OpenAI.

---

## الموديل

```properties
spring.ai.openai.chat.options.model=openai/gpt-oss-120b
```

ده الـ Model اللي هيجاوب على الأسئلة.

---

# 4. إيه هو `ChatModel` ؟

`ChatModel` هو `Interface` في Spring AI يمثل أي LLM قادر على إجراء محادثة.

تقريبًا كده:

```java
public interface ChatModel {
    ChatResponse call(Prompt prompt);
}
```

هو abstraction فوق:

- OpenAI
    
- Groq
    
- Gemini
    
- Claude
    

---

# ليه استخدم `ChatModel`؟

بدل:

```java
HttpClient
↓
JSON
↓
API Parsing
```

تعمل:

```java
chatModel.call(...)
```

وخلاص.

---

# 5. حقن الـ ChatModel

```java
@RestController
public class OpenAiController {

    private final ChatModel chatModel;

    public OpenAiController(ChatModel chatModel) {
        this.chatModel = chatModel;
    }
}
```

Spring بيعمل Auto Configuration.

وبيعمل Bean من نوع:

```java
OpenAiChatModel
```

ويحقنهولك.

---

# ليه بنعمله `final` ؟

```java
private final ChatModel chatModel;
```

لأن:

- Dependency ثابتة.
    
- Immutable.
    
- Best Practice في Spring.
    

---

# أول Request

```java
@GetMapping("/{message}")
public String chat(@PathVariable String message) {
    return chatModel.call(message);
}
```

أنت تبعت:

```text
/hello
```

الـ Controller:

```text
Controller
     ↓
ChatModel
     ↓
Groq API
     ↓
LLM
     ↓
Response
```

---

# 6. هل ده يعتبر LLM في SkillHub؟

الإجابة:

## نعم 100%

لو حطيت:

```java
ChatModel
```

في SkillHub فإنت كده ضفت LLM للسيستم.

لكن الـ LLM نفسه موجود على Groq.

أنت فقط بتستخدمه.

---

# مثال في SkillHub

### AI Course Assistant

```text
Student:
"Explain Dependency Injection"

↓

LLM

↓

"Dependency Injection is..."
```

---

### Quiz Generator

```text
Generate 10 Java Questions
```

---

### Roadmap Generator

```text
I want to become Backend Developer
```

---

### Course Recommendation System

```text
I know Java and SQL
What should I learn next?
```

---

# 7. أهم Methods في `ChatModel`

## 1.

```java
String response = chatModel.call("Hello");
```

ترجع Text مباشرة.

---

## 2.

```java
ChatResponse response =
        chatModel.call(new Prompt("Hello"));
```

ترجع Response كامل.

---

## 3.

```java
Prompt prompt = new Prompt("Explain Spring");
ChatResponse response =
        chatModel.call(prompt);
```

---

## استخراج النص

```java
response.getResult()
        .getOutput()
        .getText();
```

---

# 8. Prompt Object

```java
Prompt prompt =
        new Prompt("Explain Java");
```

أو:

```java
Prompt prompt =
        new Prompt(List.of(
            new SystemMessage(
                "You are a Java teacher"
            ),
            new UserMessage(
                "Explain Streams"
            )
        ));
```

---

# ليه الـ Prompt مهم؟

لأن الـ Prompt بيسمحلك:

- System Messages
    
- User Messages
    
- Memory
    
- RAG
    
- Tools
    
- Structured Output
    

---

# 9. إمتى أستخدم `ChatModel` ؟

لما تحتاج:

✅ Chatbot

✅ Q&A

✅ Summarization

✅ Recommendations

✅ Generate Content

✅ Generate Quizzes

✅ Explain Code

✅ AI Tutor

---

# إمتى مش مناسب؟

لو محتاج:

- Embeddings
    
- Vector Search
    
- RAG Pipelines
    
- AI Agents
    

هتستخدم Components تانية في Spring AI.

---

# 10. Architecture

```text
Controller
      ↓
Service
      ↓
ChatModel
      ↓
Spring AI
      ↓
Groq API
      ↓
LLM
```

---

# في SkillHub أنا كنت هستخدمه في إيه؟

أعمل Service:

```java
AIService
```

جواه:

```java
ChatModel
```

وأضيف Features زي:

- AI Tutor
    
- Quiz Generator
    
- Course Recommendation
    
- Roadmap Generator
    
- Resume Reviewer
    
- Interview Simulator
    

---

# Roadmap بعد `ChatModel`

بعد ما تفهم `ChatModel` كويس ذاكر:

1. Prompt
    
2. SystemMessage
    
3. ChatClient
    
4. Structured Output
    
5. Embeddings
    
6. Vector Store
    
7. RAG
    
8. Tools Function Calling
    
9. AI Agents
    
10. Memory
    

وده تقريبًا هو الطريق الحقيقي عشان تبقى جامد في Spring AI وتستخدم الـ AI داخل مشاريعك بشكل احترافي.