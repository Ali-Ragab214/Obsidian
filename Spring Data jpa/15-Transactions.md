
# 🧾 Transaction في Spring

---

## 🧠 أولاً: يعني إيه Transaction؟

> **Transaction =  مجوعه عمليات علي الداتا بيز بتتنفذ كوحده واحده**
> 
> يا إمّا كلها تنجح ✅  
> يا إمّا كلها تفشل ❌

---

## ⚙️ قاعدة ACID

|الحرف|المعنى|
|---|---|
|**A - Atomicity**|كله أو لا شيء|
|**C - Consistency**|الـ DB تفضل صحيحة|
|**I - Isolation**|كل Transaction مستقلة|
|**D - Durability**|البيانات بتثبت بعد النجاح|

---


# 🧠 تخيل السيناريو الحقيقي

عندك API بيعمل:

```
Create Author
+ يضيفه في جدول authors
+ يربطه بكورسات في جدول courses
+ يسجل log في جدول logs
```

يعني 3 عمليات DB ورا بعض.

لو واحدة فشلت؟ 💥  
يبقى الداتا باز باظت.

هنا يظهر دور **Transaction**.

---

# 🎯 الفكرة ببساطة جداً

Transaction =  
"يا كل العمليات تنجح مع بعض  
يا كلهم يتلغوا كأنهم محصلوش"

زي زرار Undo.

---

# 💥 من غير Transaction يحصل ايه؟

خلينا نشوف الكود بدون Transaction:

```java
public void createAuthor() {
    repository.save(author);        // 1️⃣ نجحت
    courseRepo.linkAuthor();        // 2️⃣ نجحت
    logRepo.save(log);              // 3️⃣ السيرفر وقع ❌
}
```

النتيجة في الداتابيز:

|authors|courses|logs|
|---|---|---|
|✔️|✔️|❌|

الداتا بقت **مش consistent**  
وده أسوأ كابوس لأي Backend Dev 😅

---

# ✅ مع Transaction يحصل ايه؟

```java
@Transactional
public void createAuthor() {
    repository.save(author);
    courseRepo.linkAuthor();
    logRepo.save(log); // حصل error هنا
}
```

Spring يعمل التالي داخلياً 👇

```
BEGIN TRANSACTION
   save author
   link course
   error حصل
ROLLBACK
```

النتيجة في DB:

|authors|courses|logs|
|---|---|---|
|❌|❌|❌|

كأن ولا حاجة حصلت ❤️

---

# ⚙️ طيب Spring بيعمل ده ازاي؟

لما تحط:

```java
@Transactional
```

Spring يعمل حاجة اسمها:

```
Transaction Proxy
```

يعني يلف حوالين الميثود ويعمل:

```
open transaction
run method
لو نجحت → commit
لو حصل exception → rollback
```

حرفياً Spring بيضيف كود مخفي حول الميثود.

---

# 🏗️ فين نحط Transaction؟

السؤال المهم 👌

## ❌ غلط

- Controller → مفيش business logic
    
- Repository → مجرد SQL
    

## ✅ الصح

Service Layer لأنها:

- فيها Business Logic
    
- فيها أكتر من Query
    

```
Controller → Service(@Transactional) → Repository → DB
```

---

# 🟢 امتى استخدم Transaction؟

## أي ميثود بتغير الداتا

|عملية|نحتاج Transaction؟|
|---|---|
|create|✅|
|update|✅|
|delete|✅|
|get|❌|

---

# ✨ تطبيق على مشروعك

## قبل

```java
@Service
public class AuthorServiceImpl {}
```

## بعد

```java
@Service
@Transactional(readOnly = true) // كل الكلاس read
public class AuthorServiceImpl {}
```

ليه readOnly؟  
عشان معظم الميثودز read.

---

## نفتح Transaction للـ write فقط

### create

```java
@Transactional
public AuthorDTO createAuthor(AuthorRequest request)
```

### update

```java
@Transactional
public AuthorDTO updateAuthor(Long id , AuthorRequest request)
```

### delete

```java
@Transactional
public void deleteAuthor(Long id)
```

بس كده 👌

---

# 🤯 سؤال مهم: ليه update مش محتاج save ؟

دي أهم نقطة تفهمها 🔥

لما تعمل:

```java
Author a = repository.findById(id).get();
a.setFirstname("Ali");
```

Hibernate يعمل:

```
Entity Managed
↓
Hibernate يراقب التغييرات
↓
قبل انتهاء transaction يعمل UPDATE تلقائي
```

اسمها:

# Dirty Checking

يعني مجرد ما تعدل الـ object → يتكتب في DB تلقائي عند نهاية Transaction.

علشان كده save() مش لازم في update.

---

# 🟡 readOnly = true بيعمل ايه؟

```java
@Transactional(readOnly = true)
```

Spring يقول لـ Hibernate:

- متراقبش التغييرات
    
- متعملش update
    
- اشتغل أسرع
    

يعني performance boost 🚀

---

# 🔄 امتى يحصل Rollback؟

افتراضياً يحصل لما يحصل:

```
RuntimeException
```

زي:

```java
throw new RuntimeException()
```

ساعتها Spring يعمل rollback لكل اللي حصل.

---

# 🧠 تخيل اللي بيحصل داخلياً

لما API يتضرب:

```
Controller call Service
↓
Spring يفتح Transaction
↓
Hibernate ينفذ Queries
↓
لو مفيش Exception → COMMIT
لو Exception → ROLLBACK
↓
يقفل Transaction
```

---

# 🎉 الخلاصة البسيطة جداً

- Transaction = حماية الداتا
    
- تتحط في Service فقط
    
- write methods لازم @Transactional
    
- read methods readOnly = true
    
- rollback يحصل مع RuntimeException
    
- Hibernate يعمل update تلقائي (dirty checking)
    

---

لو حابب أحول الموضوع لرسمة flow أو cheat sheet سريعة للمراجعة قبل الانترفيو أعملهالك 👍  
والخطوة الجاية ندخل Global Exception Handling.