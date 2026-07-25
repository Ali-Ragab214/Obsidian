يعني بدل ما نكررر السيستم مسدج في كل  endpoint 
![[Pasted image 20260723165503.png]]
# Defaults في Spring AI — باختصار

الفكرة بسيطة: بدل ما تكتب نفس الـ `system` message أو نفس الإعدادات (زي الـ advisors) في **كل مرة** بتعمل فيها request للـ AI، تحطها **مرة واحدة** وقت ما بتبني الـ `ChatClient`، وهي تتطبق تلقائيًا على أي prompt هتبعته بعد كده — من غير ما تكررها.

## مثال عملي

بدل ما تكتب كده في كل endpoint:

```java
chatClient.prompt()
    .system("You are a helpful assistant that answers HR-related queries.")
    .user(userMessage)
    .call();
```

تحطها مرة واحدة في الـ `Bean` بتاع الـ `ChatClient`:

```java
@Bean
public ChatClient chatClient(ChatClient.Builder builder) {
    return builder
        .defaultSystem("""
            You are a helpful assistant that answers HR-related queries.
        """)
        .build();
}
```

وبعد كده في أي مكان في الكود:

```java
chatClient.prompt()
    .user(userMessage)  // مش محتاج تكتب system تاني، هي متطبقة تلقائي
    .call();
```

## ليه مفيدة؟

- علشان متكررش نفس الـ system prompt في كل controller/service.
- الـ tone أو الـ role بتاع الـ AI بيفضل ثابت في كل حتة في المشروع.
- لو عايز تغير سلوك الـ assistant كله، بتغير في مكان واحد بس مش كل endpoint.

## ملحوظة مهمة

لو محتاج تغيّر السلوك في request معين (استثناء)، تقدر تعمل override عادي:

```java
chatClient.prompt()
    .system("Override: answer only in bullet points.") // ده هيغلب الـ default
    .user(userMessage)
    .call();
```

يعني الـ Default مش قيد، هو بس **قيمة افتراضية** تقدر تغيرها وقت ما تحتاج.