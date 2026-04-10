جميل جدًا 👏 الكود ده معمول مخصوص عشان تتعلم CSRF عمليًا.

خلينا نمشي خطوة خطوة ونفهم:

- إنت عملت ايه
    
- ليه محتاج CSRF
    
- إزاي تستخدمه فعليًا في Postman أو Frontend
    

---

# أولًا: ليه الكود ده محتاج CSRF أصلاً؟

أنت شغال بالـ **Session (stateful)**  
يعني لما تعمل Login، Spring Security بيستخدم:

```
JSESSIONID Cookie
```

وأي Request بعد كده المتصفح هيبعت الكوكي تلقائي.

وده بالضبط السيناريو اللي يحصل فيه هجوم CSRF 😈  
عشان كده Spring Security مفعل CSRF تلقائيًا.

---

# ثانيًا: فهم Endpoint مهم جدًا 👇

## 1) Endpoint جلب التوكن

```java
@GetMapping("/csrf")
public CsrfToken getCsrfToken(HttpServletRequest request) {
    return (CsrfToken) request.getAttribute("_csrf");
}
```

ده Endpoint مهم جدًا 💥  
وظيفته يطلعلك الـ CSRF Token.

Spring بيحط التوكن داخل الـ request تلقائيًا في attribute اسمه:

```
_csrf
```

إنت هنا بتطلعه وترجعه للـ client.

---

# شكل الـ Response هيكون عامل كده

لما تعمل GET على:

```
GET /csrf
```

هترجع حاجة شبه:

```json
{
  "token": "e8c3b2b7-9a3d",
  "parameterName": "_csrf",
  "headerName": "X-CSRF-TOKEN"
}
```

ركز في:

```
headerName = X-CSRF-TOKEN
```

ده أهم سطر 👈

---

# ثالثًا: امتى لازم أستخدم CSRF ؟

CSRF مطلوب مع أي Request **يغير بيانات**:

|Method|يحتاج CSRF|
|---|---|
|GET|❌ لا|
|POST|✔️ نعم|
|PUT|✔️ نعم|
|DELETE|✔️ نعم|

عشان كده:

- `/students` شغال عادي ✔️
    
- `/students/add` هيطلع 403 ❌ بدون CSRF
    

---

# رابعًا: ازاي تستخدم CSRF فعليًا (أهم جزء) 🔥

هنعمل سيناريو Postman خطوة خطوة.

---

## Step 1️⃣ اعمل Login

بعد login المتصفح أو Postman هيخزن:

```
JSESSIONID Cookie
```

بدونها مش هينفع تكمل.

---

## Step 2️⃣ هات CSRF Token

اعمل Request:

```
GET /csrf
```

لازم تبعته معاه الكوكي:

```
Cookie: JSESSIONID=xxxx
```

هيجيلك Response فيه:

```
token = ABC123
headerName = X-CSRF-TOKEN
```

---

## Step 3️⃣ استخدم التوكن في POST

دلوقتي تقدر تعمل:

```
POST /students/add
```

لازم تضيف Header:

```
X-CSRF-TOKEN: ABC123
```

ومعاه الكوكي:

```
JSESSIONID=xxxx
```

كده الطلب يعدي ✔️

لو نسيت التوكن:

```
403 Forbidden ❌
```

---

# ليه لازم الكوكي + التوكن مع بعض؟

السيرفر بيتأكد من حاجتين:

1️⃣ هل ده نفس المستخدم؟ → من الكوكي  
2️⃣ هل الطلب جاي من الموقع الحقيقي؟ → من CSRF Token

الاتنين لازم يكونوا موجودين.

---

# خامسًا: مثال Request كامل

```
POST /students/add

Cookie: JSESSIONID=ABC123
X-CSRF-TOKEN: f9s8df7sdf

Body:
name=Omar&id=4&grade=95
```

لو التوكن غلط → 403  
لو الكوكي مش موجود → 401

---

# سادسًا: ليه عملنا Endpoint `/csrf` ؟

في التطبيقات الحقيقية:

- Frontend (Angular/React) لازم يجيب التوكن الأول
    
- وبعدين يستخدمه في أي POST
    

الـ flow الطبيعي:

```
Frontend → GET /csrf → يخزن token
Frontend → POST/PUT/DELETE + token
```

---

# الخلاصة السهلة جدًا 🎯

إنت دلوقتي بتستخدم CSRF صح لما:

1️⃣ تعمل Login → تاخد Session Cookie  
2️⃣ تجيب CSRF Token من `/csrf`  
3️⃣ تبعته في Header مع أي POST/PUT/DELETE

لو نسيت التوكن → Spring يرفض الطلب.

---

لو حابب أشرحلك ليه Spring بيخزن التوكن في Session وازاي بيقارنه داخليًا قولّي 😄