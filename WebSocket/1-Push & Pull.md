
---

# الأول: يعني إيه Push و Pull؟

تخيل عندك تطبيق شات.

## Pull

العميل (Frontend) بيسأل السيرفر كل شوية:

```text
هل في رسالة جديدة؟
هل في رسالة جديدة؟
هل في رسالة جديدة؟
```

مثلا كل 3 ثواني:

```javascript
setInterval(() => {
   fetch("/messages")
}, 3000)
```

الطريقة دي اسمها:

**Polling (Pull)**

لأن:

```text
Client ---> Server
```

العميل هو اللي بيروح يجيب الداتا.

---

### المشكلة؟

لو مفيش رسالة جديدة:

```text
Request
Response

Request
Response

Request
Response
```

كل ده استهلاك شبكة وموارد بدون فائدة.

---

# Push

بدل ما العميل يسأل كل شوية:

السيرفر هو اللي يبعت لما يحصل تغيير.

```text
Server ---> Client
```

مثلا:

- وصلت رسالة جديدة
    
- اتبعت إشعار
    
- تغير سعر سهم
    

السيرفر فورًا يبعت للعميل.

---

# طب WebSocket فين هنا؟

websocket بروتوكول بيعمل   
connection 
دائم بين 

```text
Client <=====> Server
```

الاتصال يفضل مفتوح.

مش زي HTTP.

---

# HTTP

كل مرة:

```text
Request
Response

Connection Closed
```

---

# WebSocket

مرة واحدة بس:

```text
Handshake

Connection Open
```

بعدها:

```text
Client ---> Server

Server ---> Client

Client ---> Server

Server ---> Client
```

بدون إنشاء Connection جديد كل مرة.

---

# ليه اتعمل؟

عشان التطبيقات الـ Real-Time.

زي:

- WhatsApp
    
- Messenger
    
- Discord
    
- Telegram
    
- Online Games
    
- Trading Systems
    
- Notifications
    

---

# مثال عملي

## بدون WebSocket

أنت فاتح شات.

كل ثانية:

```javascript
GET /messages
```

السيرفر يرد:

```json
[]
```

كل ثانية.

---

## مع WebSocket

أول ما حد يبعت رسالة:

السيرفر يقول:

```json
{
   "message":"Hello"
}
```

فتظهر فورًا.

---

# الفرق بين REST API و WebSocket

|REST|WebSocket|
|---|---|
|Request/Response|اتصال دائم|
|Client يبدأ الاتصال|أي طرف يقدر يرسل|
|مناسب CRUD|مناسب Real-Time|
|Stateless|Stateful|
|أسهل|أعقد شوية|

---

# إمتى أستخدم REST؟

لو:

```text
Login
Register
Products
Orders
Users
Categories
```

يعني عمليات عادية.

---

# إمتى أستخدم WebSocket؟

لو:

```text
Chat
Notifications
Live Tracking
Live Score
Stock Market
Collaborative Editing
```

---

# في Spring Boot بنستخدمه إزاي؟

Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

---

# WebSocket Config

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig
        implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(
            StompEndpointRegistry registry) {

        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(
            MessageBrokerRegistry registry) {

        registry.enableSimpleBroker("/topic");

        registry.setApplicationDestinationPrefixes("/app");
    }
}
```

---

# Controller

```java
@Controller
public class ChatController {

    @MessageMapping("/chat")
    @SendTo("/topic/messages")
    public String sendMessage(String message) {

        return message;
    }
}
```

---

# التدفق

Client يرسل:

```text
/app/chat
```

↓

Spring يستقبل

↓

Controller

↓

يرسل إلى:

```text
/topic/messages
```

↓

كل العملاء المتصلين يستقبلوا الرسالة.

---

# في React

تستخدم مكتبة مثل:

```bash
npm install sockjs-client
npm install stompjs
```

أو

```bash
npm install @stomp/stompjs
```

ثم:

```javascript
client.subscribe(
   "/topic/messages",
   callback
);
```

---

# أشهر استخدام هتقابله كشغل Backend

## Notifications

مثلا عندك منصة تعليمية.

طالب اشترى كورس.

الأدمن يشوف إشعار فورًا.

---

## Chat

زي دعم العملاء:

```text
Customer
    ↔
Support Agent
```

---

## Live Dashboard

عدد المستخدمين الحاليين.

مثلا:

```text
1250 users online
```

يتحدث لحظيًا.

---

## Tracking

زي أوبر.

```text
Driver Location
```

تتحدث كل ثانية.

---

# هل لازم أتعلمه دلوقتي؟

بصراحة بالنسبة لمستواك الحالي:

أولوية أعلى:

1. Spring Security
    
2. JWT
    
3. OAuth2
    
4. JPA/Hibernate
    
5. Redis
    
6. Docker
    
7. Testing
    
8. Microservices
    

ثم بعدها:

9. WebSocket
    

لكن بما إنك بدأت الكورس بالفعل، فافهم:

- Push vs Pull
    
- HTTP vs WebSocket
    
- STOMP
    
- Topics
    
- Subscriptions
    
- Real-Time Notifications
    
- Chat Application
    

دول يمثلوا تقريبًا **80% من الاستخدام الحقيقي لـ WebSocket في شغل Spring Boot**.

الخلاصة:

**WebSocket = قناة اتصال دائمة بين العميل والسيرفر تسمح للطرفين بإرسال واستقبال البيانات لحظيًا (Real-Time).**  
ستحتاجه في الشات، الإشعارات الفورية، تتبع المواقع، لوحات البيانات الحية، وأي نظام يحتاج تحديثات مباشرة بدون أن يظل العميل يطلب البيانات كل ثانية. في Spring Boot غالبًا ستستخدمه مع STOMP وSockJS وتدمجه مع React أو أي Frontend آخر.

فتح  connection  دايم بين ال client and server يعني لو في اي حاجه جديده السيرفر بيبعتها تلقائي بدل ما يبقا في كل شويه ريكوست رايح وريسبونص راجع بين الكلاينت والسيرفر ودا مهلك
> "لو في حاجة جديدة السيرفر بيبعتها تلقائي بدل ما يبقا في كل شوية request"

✔️ ده بالظبط اسمه:

### 👉 Push Model

السيرفر هو اللي “يدفع” الداتا للعميل أول ما يحصل Event.
# ⚡ الفرق المهم اللي لازم تثبته

## ❌ HTTP (REST)

- العميل هو اللي يسأل كل مرة
- كل Request مستقل
- connection بتتقفل بعد الرد

## ✅ WebSocket

- Connection واحدة مستمرة
- أي طرف يقدر يبعث في أي وقت
- السيرفر يقدر يبعت لوحده بدون طلب
- ---

- # 🧩 ليه ده مهم؟

لأنه يقلل:

- Requests زيادة بدون فايدة ❌
- Load على السيرفر ❌
- Delay في التحديث ❌

ويخلي:

- Real-time apps شغالة صح ✅
- ---
- WebSocket هو بروتوكول بيعمل اتصال دائم بين العميل والسيرفر، وبيسمح بتبادل البيانات في الاتجاهين بشكل لحظي بدون الحاجة لعمل requests متكررة، وده بيخليه مناسب لتطبيقات الـ Real-time.