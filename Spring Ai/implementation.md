بما إن مشروعك أصلاً بيستخدم `ApiResponse<T>` envelope pattern وbuilds على layered architecture (Controller → Service)، هعمللك الكود متوافق مع نفس الستايل ده بدل ما أخليه controller لوحده بمنطق جوّاه.

## 1. DTO — Request

**`application/dto/request/ChatRequest.java`**

```java
package com.example.Skill_Hub_Learning_Platform.application.dto.request;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record ChatRequest(
        @NotBlank(message = "Message must not be empty")
        @Size(max = 2000, message = "Message must not exceed 2000 characters")
        String message
) {}
```

## 2. Service Interface

**`application/services/chat/ChatService.java`**

```java
package com.example.Skill_Hub_Learning_Platform.application.services.chat;

public interface ChatService {
    String getResponse(String message);
}
```

## 3. Service Implementation

**`application/services/chat/ChatServiceImpl.java`**

```java
package com.example.Skill_Hub_Learning_Platform.application.services.chat;

import lombok.extern.slf4j.Slf4j;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.openai.OpenAiChatModel;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class ChatServiceImpl implements ChatService {

    private final ChatClient chatClient;

    public ChatServiceImpl(OpenAiChatModel chatModel) {
        this.chatClient = ChatClient.create(chatModel);
    }

    @Override
    public String getResponse(String message) {
        try {
            return chatClient.prompt(message)
                    .call()
                    .content();
        } catch (Exception e) {
            log.error("Chat model call failed: {}", e.getMessage());
            throw new ChatServiceException("Failed to get response from AI model", e);
        }
    }
}
```

## 4. Custom Exception

**`application/exceptions/ChatServiceException.java`**

```java
package com.example.Skill_Hub_Learning_Platform.application.exceptions;

public class ChatServiceException extends RuntimeException {
    public ChatServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

_(لو عندك `GlobalExceptionHandler` جاهز، ضيفله `@ExceptionHandler(ChatServiceException.class)` بيرجع HTTP 502/500 بنفس شكل الـ `ApiResponse`)_

## 5. Controller

**`api/controller/ChatController.java`**

```java
package com.example.Skill_Hub_Learning_Platform.api.controller;

import com.example.Skill_Hub_Learning_Platform.application.dto.request.ChatRequest;
import com.example.Skill_Hub_Learning_Platform.application.responses.ApiResponse;
import com.example.Skill_Hub_Learning_Platform.application.services.chat.ChatService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/chat")
@RequiredArgsConstructor
public class ChatController {

    private final ChatService chatService;

    @PostMapping
    public ResponseEntity<ApiResponse<String>> chat(@Valid @RequestBody ChatRequest request) {
        String response = chatService.getResponse(request.message());
        return ResponseEntity.status(HttpStatus.OK)
                .body(ApiResponse.success("Response generated successfully", response));
    }
}
```

_(عدّل `ApiResponse.success(...)` حسب الـ static factory method الموجود عندك بالفعل في الكلاس)_

---

### إيه اللي اتغيّر ولية best practice:

|التغيير|السبب|
|---|---|
|فصل الـ Service عن الـ Controller|الـ Controller المفروض يبقى thin — استقبال request، تفويض، رد. المنطق (استدعاء الـ AI model) مكانه الـ service layer، زي باقي مشروعك تمامًا|
|`@RequiredArgsConstructor` على الـ Controller بس|مفيش تعارض دلوقتي لأن الـ `ChatClient.create()` اتنقل جوه الـ `ChatServiceImpl` اللي محتاج constructor يدوي|
|`@Valid` + `@NotBlank/@Size`|يمنع إرسال رسائل فاضية أو طويلة جدًا تكلفك تكلفة API عبثًا — بنفس نمط الـ validation في باقي DTOs بتاعتك|
|`try/catch` + Custom Exception|لو الـ Groq API وقع أو رجّع rate limit، الغلطة تتلقط بشكل نظيف بدل ما تطلع كـ 500 عشوائي|
|`ApiResponse<T>` envelope|اتساق مع باقي الـ 40+ endpoint في مشروعك بدل ما يرجع `String` عاري|
|`POST /api/v1/chat`|متوافق مع الـ URL convention بتاعتك (`/api/v1/{resource}`)|

**متبقاش تنسى:** لسه الـ Groq API key اللي بعتهولي فات ده لازم تعمله revoke لو لسه ماعملتوش.