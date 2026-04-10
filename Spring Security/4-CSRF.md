Cross Site Request Forgery


---

# يعني ايه CSRF ؟

CSRF =  
**Cross Site Request Forgery**

بالعربي:  
موقع يخدع المتصفح بتاعك يعمل Request لموقع تاني **وانت مسجل دخول فيه**.

يعني:

> الهاكر يخلي المتصفح بتاعك يعمل عملية وانت مش واخد بالك.

---

# الفكرة الأساسية للهجوم 😈

المشكلة إن المتصفح:

- بيخزن Cookies تلقائيًا
    
- وبيبعتها تلقائيًا مع أي Request
    

حتى لو الطلب جاي من موقع خبيث!

ودي نقطة خطيرة جدًا.

---

# مثال واقعي قوي 💣

## عندك بنك أونلاين 🏦

وانت مسجل دخول فيه.

وعندهم endpoint:

```
POST /transfer
```

بيعمل تحويل فلوس.

---

## وانت فاتح موقع البنك

المتصفح عنده Cookie:

```
JSESSIONID = abc123
```

---

## تدخل موقع هاكر 😈

الموقع فيه كود مخفي:

```html
<form action="https://bank.com/transfer" method="POST">
   <input type="hidden" name="amount" value="10000">
   <input type="hidden" name="toAccount" value="HACKER">
</form>

<script>
   document.forms[0].submit();
</script>
```

أول ما تفتح الصفحة 😱  
المتصفح يبعت Request للبنك تلقائيًا!

والمصيبة 💥  
هيبعت Cookie البنك معاه:

```
POST /transfer
Cookie: JSESSIONID=abc123
```

البنك يفكر:

> ده المستخدم الحقيقي!

ويحول الفلوس 💸

وانت مش عارف حاجة!

ده هو CSRF.

---

# ليه الهجوم ده بيحصل؟

لأن السيرفر بيعتمد على:

```
Session Cookie فقط
```

والمتصفح بيبعتها تلقائيًا لأي Request.

السيرفر مش قادر يعرف:

- هل الطلب جاي من الموقع الحقيقي؟
    
- ولا من موقع خبيث؟
    

---

# الحل: CSRF Token 🛡️

الفكرة:  
نضيف حاجة **الهاكر مش يقدر يعرفها**.

اسمها:

```
CSRF Token
```

Token سري يولده السيرفر لكل Session.

---

## لما تفتح صفحة البنك

السيرفر يبعت:

- Session cookie
    
- CSRF Token
    

مثلاً:

```
CSRF Token = X9K2P4
```

---

## لما تعمل عملية حساسة

لازم تبعت التوكن مع الطلب:

```
POST /transfer
Cookie: JSESSIONID=abc123
X-CSRF-TOKEN: X9K2P4
```

السيرفر يقارن:

- التوكن اللي في السيشن
    
- التوكن اللي في الطلب
    

لو مش موجود أو غلط → ❌ يرفض الطلب.

---

# ليه الهاكر ما يقدرش يسرق التوكن؟

لأن:

- المتصفح يمنع المواقع من قراءة بيانات مواقع أخرى
    
- الهاكر يقدر يبعث Request
    
- لكن **ما يقدرش يقرأ التوكن**
    

وده الفرق المهم جدًا.

---

# امتى CSRF بيكون خطر؟

فقط في:

```
Session-based authentication (Cookies)
```

يعني:

- Spring Security الافتراضي → محتاج CSRF ✔️
    

---

# طب مع JWT؟

في REST APIs:

- التوكن بيتبعت في Header
    

```
Authorization: Bearer token
```

المتصفح **لا يرسله تلقائيًا**.

يعني موقع الهاكر مش قادر يبعته.

لذلك:

```
JWT APIs → غالبًا CSRF مش مطلوب
```

عشان كده بنعمل:

```java
http.csrf().disable();
```

في تطبيقات REST.

---

# مثال في Spring Security

Spring يفعل CSRF افتراضيًا.

لو حاولت تبعت POST بدون توكن:

```
403 Forbidden
```

ولما تستخدم Thymeleaf مثلاً:  
Spring يضيف التوكن تلقائيًا للفورم.

---

# خلاصة سهلة جدًا 🎯

CSRF =  
موقع يخدع متصفحك يعمل Request وانت logged in.

الحل =  
CSRF Token لازم يتبعت مع أي Request حساس.

- Session + Cookies → لازم CSRF
    
- JWT REST API → غالبًا نوقف CSRF
    

---

لو حابب نفهم الفرق عملي بين 401 و 403 في Spring Security بعدها قولّي 😄