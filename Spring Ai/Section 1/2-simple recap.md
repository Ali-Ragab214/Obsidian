```java 
List<Message> messages = List.of(
new SystemMessage("You are a java expert"),
new UserMessage("Explain OOP")
);

Ptompt propmt = new Prompt(message);
//ثم تبعته للـ ChatModel.
ChatResponse response = chatModel.call(prompt);
```


Defaults :  تقدر تحدد default configuration للـ ChatClient. 
مثلا : 
كل request يطلع من الـ ChatClient ده، اعتبر إن الـ system instruction بتاعته كذا.

Prompt is a set of messages 

              Spring AI
                  │
             ChatClient
                  │
               Prompt
                  │
        ┌─────────┴─────────┐
        │                   │
 SystemMessage        UserMessage
        │                   │
        └─────────┬─────────┘
                  │
              ChatModel
                  │
                  ▼
               LLM
                  │
                  ▼
             ChatResponse 


والـ `ChatModel` هو abstraction بيتعامل مع الـ underlying AI model. 

  يعني حاجه low level 
```java 
@Service 
public class AiService 
{
private final ChatClient chatclient;

public AiService(ChatClient chatClient) { this.chatClient = chatClient;
 }
public String ask(String question)

{
return chatclient.prompt()
        .user(question)
        .call()
        .content();
}

}
```

Message
   │
   ├── System → defines behavior/instructions
   │
   ├── User → user's request
   │
   └── Assistant → model's previous response


**Q1: What is a Message in an LLM application?**  
رسالة لها role ومحتوى، مثل System/User/Assistant.

**Q2: What's the difference between System and User messages?**  
System تحدد سلوك وتعليمات الـ model، بينما User تحتوي طلب المستخدم.

**Q3: What is an Assistant message?**  
تمثل response سابق من الـ model، ومهمة في conversational context.

**Q4: What is a Prompt?**  
مجموعة الـ messages/context التي يتم إرسالها إلى الـ model.

**Q5: What are defaults in Spring AI?**  
إعدادات مشتركة يتم تطبيقها على requests الخاصة بـ `ChatClient` بدل تكرارها.

---


# Prompt Template

> **A reusable prompt structure containing fixed instructions and dynamic variables.**