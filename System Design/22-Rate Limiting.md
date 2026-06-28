# Rate Limiting

## ما هو الـ Rate Limiting ؟

الـ Rate Limiting هو عملية **تحديد عدد الطلبات (Requests) المسموح بها خلال فترة زمنية معينة**.

مثال:

```text
كل مستخدم مسموح له بـ 100 Request كل دقيقة
```

إذا حاول المستخدم إرسال الطلب رقم 101 خلال نفس الدقيقة:

```http
HTTP 429 Too Many Requests
```

---

# لماذا نحتاج Rate Limiting ؟

بدون Rate Limiting أي شخص يمكنه تنفيذ:

```text
10000 Request/sec
```

مما يؤدي إلى:

- استهلاك CPU بشكل ضخم.
    
- استهلاك Memory.
    
- الضغط على Database.
    
- استهلاك API quotas الخارجية.
    
- تعطيل الخدمة لباقي المستخدمين.
    

---

# مثال حقيقي

لنفترض أن لديك Endpoint:

```http
POST /api/auth/login
```

بدون Rate Limiting:

```text
Attacker
↓
1,000,000 login attempts
↓
Brute Force Attack
```

مع Rate Limiting:

```text
5 login attempts / minute / IP
```

بعد المحاولة السادسة:

```http
429 Too Many Requests
```

---

# مثال آخر مع الـ AI

لديك:

```http
POST /api/chat
```

وكل Request يكلفك:

```text
0.01$
```

مهاجم أرسل:

```text
50000 Request
```

التكلفة:

```text
500$
```

لهذا تعتبر Rate Limiting شبه إلزامية في تطبيقات الـ AI.

---

# أين نستخدم Rate Limiting ؟

## 1) Login APIs

مثال:

```text
5 requests/minute/IP
```

---

## 2) Register APIs

مثال:

```text
3 requests/hour/IP
```

---

## 3) Password Reset

مثال:

```text
2 requests/hour/email
```

---

## 4) AI Endpoints

مثال:

```text
20 requests/minute/user
```

---

## 5) OTP Verification

مثال:

```text
5 requests/10 minutes
```

---

## 6) Public APIs

مثال:

```text
100 requests/minute/API Key
```

---

## 7) Search Endpoint

مثال:

```text
60 requests/minute/user
```

---

# ماذا يمكن أن نحدد عليه الـ Rate Limit ؟

## By IP

مثال:

```text
192.168.1.100
```

---

## By User ID

مثال:

```text
user:15
```

---

## By API Key

مثال:

```text
api-key:sk-xxxx
```

---

## By Subscription Plan

مثال:

```text
Free User -> 100 requests/day
Pro User -> 10000 requests/day
```

---

# كيف يعمل داخلياً ؟

عندما يصل Request:

```text
Request arrives
        ↓
Identify User/IP
        ↓
Check Current Count
        ↓
Count < Limit ?
       / \
     Yes  No
      |    |
Allow 429 Error
```

---

# أين يتم تخزين العدادات ؟

غالباً في:

- Redis
    
- In Memory Cache
    
- Database (غير مفضل)
    

الأكثر استخداماً في المشاريع الحقيقية:

```text
Redis
```

لأن:

- سريع جداً.
    
- يدعم Expiration.
    
- مناسب للأنظمة الموزعة.
    

---

# لماذا لا نستخدم Database ؟

تخيل:

```text
1000 Request/sec
```

وكل Request ينفذ:

```sql
SELECT ...
UPDATE ...
```

سيؤدي ذلك إلى حمل ضخم على قاعدة البيانات.

---

# أشهر الخوارزميات

## 1) Fixed Window

مثال:

```text
100 Request / Minute
```

إذا أرسل المستخدم:

```text
100 Request عند 12:00:59
```

ثم:

```text
100 Request عند 12:01:00
```

فقد أرسل:

```text
200 Request خلال ثانية واحدة
```

رغم أنه لم يخالف القواعد.

عيبه:

```text
Burst Problem
```

---

## 2) Sliding Window

يقوم بحساب آخر:

```text
60 ثانية فعلية
```

وليس الدقيقة الحالية فقط.

الأكثر دقة.

---

## 3) Token Bucket

الأكثر استخداماً.

الفكرة:

يوجد Bucket يحتوي على Tokens.

مثال:

```text
Bucket Size = 10
Refill Rate = 1 token/sec
```

كل Request يستهلك Token.

---

### البداية

```text
[10 Tokens]
```

بعد 3 Requests:

```text
[7 Tokens]
```

بعد مرور 5 ثواني:

```text
[10 Tokens]
```

---

إذا أصبح:

```text
[0 Tokens]
```

فإن أي Request جديد يتم رفضه.

---

## لماذا Token Bucket مشهور جداً ؟

لأنه يسمح بـ:

```text
Small Bursts
```

ويمنع:

```text
Huge Traffic Spikes
```

ولهذا تستخدمه:

- Cloudflare
    
- AWS API Gateway
    
- Nginx
    
- Stripe
    

---

## 4) Leaky Bucket

يتعامل مع الـ Requests كالماء داخل دلو.

الطلبات تخرج بسرعة ثابتة.

إذا امتلأ الدلو:

```text
Drop Requests
```

---

# ماذا يعيد السيرفر عند تجاوز الحد ؟

```http
HTTP/1.1 429 Too Many Requests
```

وغالباً:

```http
Retry-After: 60
```

مثال:

```json
{
  "message": "Too many requests",
  "retryAfter": 60
}
```

---

# استخدام Rate Limiting في SkillHub

## Login

```text
5 attempts/minute/IP
```

---

## Register

```text
3 registrations/hour/IP
```

---

## Chatbot

```text
20 requests/minute/user
```

---

## Course Search

```text
100 requests/minute/user
```

---

## Review Creation

```text
10 reviews/hour/user
```

---

# أشهر المكتبات في Spring Boot

## Bucket4j

الأكثر انتشاراً مع Spring Boot.

يدعم:

- Redis
    
- Hazelcast
    
- In Memory
    
- JDBC
    

---

## Resilience4j

يحتوي على:

- Circuit Breaker
    
- Retry
    
- Rate Limiter
    
- Bulkhead
    

---

## Spring Cloud Gateway Rate Limiter

يستخدم غالباً مع:

```text
Microservices
```

---

# ماذا ستتعلم عملياً ؟

عندما تبدأ التطبيق العملي ستتعلم:

- Bucket4j
    
- Redis Integration
    
- Per User Limit
    
- Per IP Limit
    
- JWT Based Limits
    
- Dynamic Limits حسب الباقة
    
- Custom 429 Response
    

---

# الخلاصة

الـ Rate Limiting هو:

> آلية لحماية التطبيق عن طريق تحديد عدد الطلبات المسموح بها خلال فترة زمنية محددة.

ستحتاجه غالباً في:

- Authentication
    
- AI Endpoints
    
- Search APIs
    
- Public APIs
    
- SaaS Applications
    

وبالنسبة لمشروعك SkillHub خصوصاً إذا أضفت Chatbot أو AI Features فهو يعتبر من الأشياء المهمة جداً في الـ Production وليس مجرد إضافة اختيارية.


