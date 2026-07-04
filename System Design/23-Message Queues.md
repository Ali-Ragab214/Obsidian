
## يعني إيه Message Queue؟

تخيل إن عندك مطعم:

- العميل بيطلب أوردر عند الكاشير.
    
- الكاشير بيكتب الطلب ويحطه في مكان مخصوص.
    
- الشيف بياخد الطلب لما يفضى ويبدأ يجهزه.
    

المكان اللي بيتحط فيه الطلب ده هو الـ **Queue**.

في البرمجة:

- خدمة أو نظام معين بيبعت رسالة (**Producer**).
    
- الرسالة بتتحط في الـ **Queue**.
    
- خدمة تانية بتستقبل الرسالة وتعالجها (**Consumer**).
    

```text
Producer  --->  Queue  ---> Consumer
```

مثال:

```text
User registers
      ↓
User Service
      ↓
Send Email Message
      ↓
Message Queue
      ↓
Email Service
      ↓
Send Welcome Email
```

بدل ما المستخدم يستنى إرسال الإيميل، التسجيل بيخلص فورًا والإيميل بيتبعت في الخلفية.

---

# ليه بنستخدم Message Queues ؟

## 1) Asynchronous Processing

بدل ما الـ Request يستنى كل العمليات تخلص.

بدل:

```text
Create Order
↓
Send Email
↓
Update Analytics
↓
Notify Warehouse
↓
Return Response
```

الـ Response ممكن ياخد 5 ثواني.

أما باستخدام Queue:

```text
Create Order
↓
Put tasks in queue
↓
Return Response immediately
```

والعمليات التانية تشتغل في الخلفية.

---

## 2) Decoupling

الخدمات متبقاش مرتبطة ببعض مباشرة.

بدون Queue:

```text
Order Service
   ↓
Email Service
   ↓
Notification Service
   ↓
Analytics Service
```

لو Email Service وقعت:

```text
Order Creation fails
```

مع Queue:

```text
Order Service
      ↓
 Message Broker
   ↙     ↓      ↘
Email  Analytics Notification
```

أي Service تقع، الباقي يشتغل عادي.

---

## 3) Load Leveling

لو عندك فجأة:

```text
100000 requests
```

مش هتقدر تعالجهم مرة واحدة.

الـ Queue تمسكهم وتوزعهم تدريجيًا.

مثال:

```text
100000 image uploads
↓
Queue stores them
↓
Workers process 100/sec
```

---

## 4) Reliability

لو الـ Consumer وقع:

```text
Message stays in queue
```

ولما الخدمة ترجع تكمل الشغل.

---

## المكونات الأساسية

### Producer

اللي بيبعت الرسائل.

مثال:

- Order Service
    
- User Service
    
- Payment Service
    

---

### Queue

مكان تخزين الرسائل مؤقتًا.

---

### Consumer

اللي بيقرأ الرسائل ويعالجها.

---

### Message Broker

هو المسؤول عن إدارة الرسائل والـ Queues.

أشهر الأنظمة:

- RabbitMQ
    
- Apache Kafka
    
- Apache ActiveMQ
    
- Amazon SQS
    

---

# Workflow

```text
Producer
   ↓
Broker
   ↓
Queue
   ↓
Consumer
```

مثال:

```text
User uploads image
        ↓
API Server
        ↓
ImageProcessingQueue
        ↓
Image Worker
        ↓
Resize Image
```

---

# Push vs Pull Model

## Pull

الـ Consumer يسأل:

```text
هل فيه رسالة؟
هل فيه رسالة؟
هل فيه رسالة؟
```

مثل:

- Apache Kafka
    

---

## Push

الـ Broker يبعت الرسالة بنفسه للـ Consumer.

مثل:

- RabbitMQ
    

---

# Queue vs Stack

Queue:

```text
FIFO
First In First Out
```

```text
A
B
C
```

الترتيب:

```text
A -> B -> C
```

---

Stack:

```text
LIFO
Last In First Out
```

```text
A
B
C
```

الترتيب:

```text
C -> B -> A
```

---

# أنواع الـ Messaging Patterns

---

## 1) Point To Point Queue

رسالة واحدة يستهلكها Consumer واحد فقط.

```text
Producer
   ↓
 Queue
 ↓  ↓  ↓
C1 C2 C3
```

لو C1 أخذ الرسالة:

```text
C2 and C3 won't receive it.
```

مثال:

- معالجة الطلبات
    
- إرسال الإيميلات
    
- معالجة الصور
    

---

## 2) Publish Subscribe

كل الـ Subscribers يستلموا نفس الرسالة.

```text
Producer
    ↓
 Topic
 ↙  ↓  ↘
A   B   C
```

مثال:

عند إنشاء Order:

- Email Service
    
- Inventory Service
    
- Analytics Service
    
- Recommendation Service
    

كلهم يستلموا نفس الـ Event.

---

## 3) Fanout

نوع من الـ Pub/Sub بيبعت لكل الـ Consumers بدون Filtering.

غالبًا موجود في:

- RabbitMQ
    

---

## 4) Topic Exchange

الـ Consumer يختار رسائل معينة فقط.

مثال:

```text
order.created
order.updated
order.cancelled
```

خدمة الشحن ممكن تهتم بـ:

```text
order.created
```

فقط.

---

## 5) Routing

إرسال الرسائل حسب Routing Key.

```text
payment.success
payment.failed
payment.pending
```

---

# Delivery Guarantees

دي من أشهر أسئلة الانترفيوهات.

---

## At Most Once

```text
0 or 1 delivery
```

ممكن الرسالة تضيع لكن مستحيل تتكرر.

---

## At Least Once

```text
1 or more deliveries
```

الرسالة لن تضيع ولكن ممكن تتكرر.

الأكثر انتشارًا.

---

## Exactly Once

```text
Exactly one delivery
```

أصعب نوع.

غالبًا يحتاج:

- Transactions
    
- Idempotency
    

---

# What is ACK ؟

بعد ما الـ Consumer يخلص:

```text
I processed the message successfully.
```

يبعت:

```text
ACK
```

للـ Broker.

لو الـ Consumer وقع قبل الـ ACK:

```text
Broker re-delivers message.
```

---

# What is Retry ؟

لو حصل خطأ:

```text
Try again after 5 seconds
Try again after 30 seconds
Try again after 1 minute
```

يسمى:

```text
Exponential Backoff
```

---

# Dead Letter Queue (DLQ)

لو الرسالة فشلت 5 مرات مثلًا:

```text
Main Queue
    ↓
Failed
    ↓
DLQ
```

يتم نقلها لـ:

```text
Dead Letter Queue
```

عشان يتم تحليل المشكلة.

---

# Ordering

هل الرسائل تحافظ على ترتيبها؟

بعض الأنظمة:

تحافظ على الترتيب داخل Partition فقط.

مثل:

- Apache Kafka
    

بينما أنظمة أخرى قد لا تضمن الترتيب الكامل مع وجود عدة Consumers.

---

# Partitioning

لو عندك:

```text
1 Million messages/sec
```

Queue واحدة قد لا تكفي.

فنقسم الرسائل:

```text
Partition 1
Partition 2
Partition 3
Partition 4
```

كل Consumer يعالج جزءًا.

---

# أشهر Message Brokers ومتى تستخدمهم

|النظام|أفضل استخدام|
|---|---|
|RabbitMQ|Tasks, Background Jobs, Emails|
|Apache Kafka|Event Streaming, Analytics, Logs|
|Amazon SQS|Managed Cloud Queue|
|Apache ActiveMQ|Enterprise Messaging|

---

# RabbitMQ vs Kafka

|Feature|RabbitMQ|Kafka|
|---|---|---|
|Architecture|Queue Based|Log Based|
|Typical Use|Background Jobs|Event Streaming|
|Ordering|Queue Ordering|Partition Ordering|
|Retention|Delete after consume|Keep for configurable time|
|Throughput|High|Extremely High|
|Replay Messages|No|Yes|

---

# مثال من الواقع في E-commerce

```text
User creates order
        ↓
Order Service
        ↓
OrderCreated Event
        ↓
Kafka Topic
    ↙    ↓      ↘
Email Inventory Analytics
```

كل Service مستقلة تمامًا.

---

# أسئلة انترفيو مشهورة

1. ما الفرق بين Queue و Topic؟
    
2. ما الفرق بين RabbitMQ و Kafka؟
    
3. ما هو الـ ACK؟
    
4. ما هو الـ DLQ؟
    
5. ما الفرق بين At Least Once و Exactly Once؟
    
6. لماذا نستخدم Message Queue بدل REST Calls؟
    
7. ما معنى Idempotency؟
    
8. كيف تمنع Duplicate Messages؟
    
9. ماذا يحدث إذا تعطل الـ Consumer؟
    
10. ما هو Consumer Group؟
    

بعد ما تستوعب المقدمة دي، الخطوة الطبيعية بعد كده هي دراسة:

1. Queue Data Structure basics.
    
2. Architecture الخاصة بالـ Message Broker.
    
3. RabbitMQ بالتفصيل.
    
4. Apache Kafka بالتفصيل.
    
5. Event Driven Architecture و Event Sourcing.
    
6. CQRS وعلاقتهما بالـ Messaging Systems.