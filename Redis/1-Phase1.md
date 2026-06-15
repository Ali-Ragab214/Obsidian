## Phase 1 — Redis Fundamentals
Remote Dictionary Server
---

## 1. ما هو Redis؟

```
Traditional Database (PostgreSQL)
─────────────────────────────────
Request → App → Disk 💿 → Response
                 ↑
            بطيء (milliseconds)

Redis
─────────────────────────────────
Request → App → RAM ⚡ → Response
                ↑
           سريع جداً (microseconds)
```

Redis بيخزن كل حاجة في الـ **RAM** مش الـ Disk — ده اللي بيخليه أسرع بـ 100x.

---

## 2. Redis كـ Key-Value Store

الفكرة الأساسية بسيطة جداً:

```
┌─────────────────────────────────────┐
│           Redis Database            │
│                                     │
│   Key          →      Value         │
│   ─────────────────────────         │
│   "user:1"     →      "Ali"         │
│   "course:5"   →      "Spring Boot" │
│   "token:abc"  →      "valid"       │
│                                     │
└─────────────────────────────────────┘
```

زي الـ HashMap بالظبط — بتحط key وبتجيب value.

---

## 3. Data Structures

### String — أبسط نوع

```
Key          Value
──────────── ──────────────
"name"    →  "Ali"
"age"     →  "25"
"score"   →  "100"
```

```bash
# حفظ
SET name "Ali"
SET age 25

# جيب
GET name        # "Ali"
GET age         # "25"

# حذف
DEL name

# هل موجود؟
EXISTS name     # 1 (آه) أو 0 (لأ)

# زود بـ 1
INCR age        # 26
INCRBY age 5    # 31
```

**بتستخدمه إمتى؟**

- حفظ session tokens
- حفظ OTP codes
- Counters (عدد الـ views)
- Cache نتيجة query بسيطة

---

### List — قائمة مرتبة

```
Key: "notifications:user:1"

Index:  0          1           2
      ┌──────────┬───────────┬──────────┐
      │ "New msg"│"New course"│"Payment" │
      └──────────┴───────────┴──────────┘
         ↑ LPUSH                  ↑ RPUSH
      (من الشمال)             (من اليمين)
```

```bash
# أضف من الشمال
LPUSH notifications "New message"
LPUSH notifications "New course"

# أضف من اليمين
RPUSH notifications "Payment done"

# جيب من index 0 لـ آخر عنصر
LRANGE notifications 0 -1
# ["New course", "New message", "Payment done"]

# جيب وامسح من الشمال
LPOP notifications    # "New course"

# كام عنصر؟
LLEN notifications    # 2
```

**بتستخدمه إمتى؟**

- Notifications queue
- Recent activity feed
- Chat messages

---

### Set — مجموعة بدون تكرار

```
Key: "course:1:enrolled_users"

┌─────────────────────────────┐
│  { "user:1", "user:2",      │
│    "user:3", "user:5" }     │
│                             │
│  ← مفيش تكرار               │
│  ← مفيش ترتيب               │
└─────────────────────────────┘
```

```bash
# أضف
SADD course:1:students "user:1"
SADD course:1:students "user:2"
SADD course:1:students "user:1"  # مش هيتضاف تاني

# جيب كل العناصر
SMEMBERS course:1:students
# {"user:1", "user:2"}

# هل موجود؟
SISMEMBER course:1:students "user:1"  # 1 (آه)

# كام عنصر؟
SCARD course:1:students   # 2

# امسح
SREM course:1:students "user:1"
```

**بتستخدمه إمتى؟**

- الـ users اللي عملوا enroll في course
- Tags بتاعة article
- Unique visitors

---

### Hash — زي الـ Object

```
Key: "user:1"

Field        Value
──────────── ──────────────
"name"    →  "Ali"
"email"   →  "ali@gmail.com"
"role"    →  "INSTRUCTOR"
"courses" →  "5"
```

```bash
# حفظ
HSET user:1 name "Ali"
HSET user:1 email "ali@gmail.com"
HSET user:1 role "INSTRUCTOR"

# أو كلهم مرة واحدة
HMSET user:1 name "Ali" email "ali@gmail.com" role "INSTRUCTOR"

# جيب field واحد
HGET user:1 name        # "Ali"

# جيب كل حاجة
HGETALL user:1
# {name: "Ali", email: "ali@gmail.com", role: "INSTRUCTOR"}

# جيب fields معينة
HMGET user:1 name email
# ["Ali", "ali@gmail.com"]

# حذف field
HDEL user:1 role

# هل الـ field موجود؟
HEXISTS user:1 name     # 1
```

**بتستخدمه إمتى؟**

- Cache بيانات User
- Cache بيانات Course
- Session data

---

### Sorted Set — Set مع ترتيب

```
Key: "leaderboard"

Score    Member
──────── ──────────
100   →  "Ali"
85    →  "Ahmed"
70    →  "Mohamed"

← مرتبين تلقائياً بالـ Score
```

```bash
# أضف مع score
ZADD leaderboard 100 "Ali"
ZADD leaderboard 85 "Ahmed"
ZADD leaderboard 70 "Mohamed"

# جيب الترتيب (تصاعدي)
ZRANGE leaderboard 0 -1 WITHSCORES

# جيب الترتيب (تنازلي)
ZREVRANGE leaderboard 0 -1 WITHSCORES
# Ali(100), Ahmed(85), Mohamed(70)

# مركز شخص معين
ZRANK leaderboard "Ali"      # 2 (من الأقل)
ZREVRANK leaderboard "Ali"   # 0 (الأول)
```

**بتستخدمه إمتى؟**

- Leaderboard
- Top courses by rating
- Trending content

---

## 4. TTL — Time To Live

### الفكرة

```
بدون TTL:
┌─────────────┐
│  Key: token │  ← هيفضل للأبد
│  Val: abc123│
└─────────────┘

مع TTL:
┌─────────────┐
│  Key: token │  ← هيتمسح تلقائياً بعد الوقت
│  Val: abc123│
│  TTL: 3600s │
└─────────────┘
     ↓ بعد ساعة
   اتمسح ✅
```

```bash
# حفظ مع TTL من الأول
SET otp "1234" EX 300        # بيتمسح بعد 5 دقايق

# أو حفظ وبعدين حط TTL
SET token "abc123"
EXPIRE token 3600            # بيتمسح بعد ساعة

# كام ثانية فاضلة؟
TTL token                    # 3599

# إزالة الـ TTL (يفضل للأبد)
PERSIST token

# حفظ بـ milliseconds
SET key "value" PX 5000      # بيتمسح بعد 5 ثواني
```

**بتستخدمه إمتى؟**

|الحالة|TTL|
|---|---|
|OTP Code|5 دقايق|
|JWT Token|نفس مدة الـ token|
|Cache بيانات|ساعة أو أكتر|
|Session|يوم أو أسبوع|

---

## 5. Caching Basics

### الفكرة

```
Flow بدون Cache:
────────────────
User → Request → App → DB Query 🐢 → Response

                 ↑ كل request بيروح للـ DB
                 ↑ حتى لو نفس البيانات
```

```
Flow مع Cache:
──────────────
              ┌─── Redis ───┐
              │   موجود؟    │
User → App →  │     ↓ آه   │→ Response ⚡
              │   ↓ لأ      │
              └─────────────┘
                    ↓
                   DB 🐢
                    ↓
              حفظ في Redis
                    ↓
                Response
```

### Cache Miss vs Cache Hit

```
Cache Miss (أول مرة):
─────────────────────
1. Request جه
2. دور في Redis → مش موجود ❌
3. روح للـ DB
4. جيب البيانات
5. احفظها في Redis
6. ارجع الـ Response

Cache Hit (بعد كده):
────────────────────
1. Request جه
2. دور في Redis → موجود ✅
3. ارجع الـ Response مباشرة ⚡
   (DB ما اتلمستش خالص)
```

### Cache Invalidation — امتى تمسح الـ Cache؟

```
┌─────────────────────────────────────┐
│         متى تمسح الـ Cache؟         │
│                                     │
│  1. TTL انتهى تلقائياً              │
│                                     │
│  2. البيانات اتحدثت                 │
│     update course → امسح cache      │
│                                     │
│  3. البيانات اتمسحت                 │
│     delete course → امسح cache      │
└─────────────────────────────────────┘
```

---

## 6. ملخص Data Structures

```
Redis Data Structures
│
├── String  → قيمة واحدة (token, counter, cache)
│
├── List    → قائمة مرتبة (notifications, queue)
│
├── Set     → مجموعة بدون تكرار (tags, members)
│
├── Hash    → object بـ fields (user, course)
│
└── Sorted Set → مجموعة مرتبة بـ score (leaderboard)
```

---

جاهز للـ **Phase 2 — Redis مع Spring Boot**؟ 🚀