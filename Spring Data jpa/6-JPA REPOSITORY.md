
---

# أولاً: يعني ايه Repository أصلاً؟

في Spring احنا بنطبق Pattern اسمه **Repository Pattern**.

وظيفته:

> يعزل التعامل مع الداتا بيز عن باقي الكود.

بدل ما تكتب SQL و EntityManager بإيدك 😵‍💫  
Spring يعمل ده أوتوماتيك.

---

# ليه بنورث من JpaRepository ؟

انت كاتب:

```java
public interface AuthorRepository extends JpaRepository<Author,Integer> {
}
```

السطر ده لوحده Spring يولّدلك Implementation كامل وقت التشغيل 🤯

بدون ما تكتب ولا سطر كود.

---

# نفك السطر ده حرف حرف 🧠

```java
JpaRepository<Author, Integer>
```

فيه Generic parameters مهمين جداً:

|الجزء|معناه|
|---|---|
|Author|الـ Entity اللي هنشتغل عليها|
|Integer|نوع الـ Primary Key|

يعني Spring فهم:

> ده Repository خاص بجدول Authors والـ ID بتاعه Integer.

---

# طب ايه اللي Spring بيدهولي جاهز؟ 😍

JpaRepository بيديك CRUD كامل.
save(),findById(),findAll() ,

خلينا نشوف أهم الميثودز اللي لازم تحفظها.

---

# 🟢 CREATE & UPDATE

## save()

```java
authorRepository.save(author);
```

الميثود دي بتعمل:

- Insert لو ID = null
    
- Update لو ID موجود
    

💡 نفس الميثود للحالتين!

---

### مثال عملي

```java
Author author = Author.builder()
        .name("Ali")
        .age(25)
        .build();

authorRepository.save(author);
```

Spring يعمل:

```sql
insert into authors ...
```

---

## Update مثال


```java
Author author = authorRepository.findById(1).get();
author.setName("Ahmed");

authorRepository.save(author);

//NOT BEST PRACTISE


```

Spring يعمل:

```sql
update authors set name='Ahmed' where id=1
```

---

# 🔵 READ

## findById()

```java
Optional<Author> author = authorRepository.findById(1);
```

ليه Optional؟  
عشان ممكن ميكونش موجود.

الاستخدام الصح:

```java
Author author = authorRepository.findById(1)
        .orElseThrow(() -> new RuntimeException("Author not found"));
        
```

⚠️ ممنوع تستخدم get() مباشرة في الشغل الحقيقي.

---

## findAll()

```java
List<Author> authors = authorRepository.findAll();
```

يرجع كل البيانات.

---

## existsById()

```java
boolean exists = authorRepository.existsById(1);

```

مفيدة جداً قبل delete.

---

## count()

```java
long total = authorRepository.count();

```

عدد الصفوف في الجدول.

---

# 🔴 DELETE

## deleteById()
الحل الافضل : 


```java
if(authorRepository.existsById(1)){
    authorRepository.deleteById(1);
}
```

---

## delete(entity)

```java

Optional<Author> a = arepo.findById(1).
orElseWhat(()->RunTimeException(""Not Found))

authorRepository.delete(author); //will delete its Row

```

---

## deleteAll()
تمسح كل الـ rows في الجدول بالكامل.

```java
authorRepository.deleteAll();
```

⚠️ تمسح الجدول كله 😅

---

# 🧠 معلومة مهمة جداً (Interview Question)

في فرق بين:

deleteAll()   =>row by row 
deleteAllInBatch() =>one query 

### deleteAll()

يمسح row by row:

SELECT * FROM author;  
DELETE FROM author WHERE id=1;  
DELETE FROM author WHERE id=2;  
DELETE FROM author WHERE id=3;

❌ بطيء لو البيانات كتير

---

### deleteAllInBatch()

authorRepository.deleteAllInBatch();

ينفذ Query واحدة فقط:

DELETE FROM author;

🚀 أسرع بكتير.
---

# 🎯 نطبّق عملي عندك (CommandLineRunner)

ضيف ده في Main:

```java
@Bean
CommandLineRunner runner(AuthorRepository repo) {
    return args -> {

        // 1️⃣ INSERT
        Author a1 = Author.builder().name("Ali").age(25).build();
        Author a2 = Author.builder().name("Sara").age(30).build();

        repo.save(a1);
        repo.save(a2);

        // 2️⃣ READ ALL
        System.out.println("All authors:");
        repo.findAll().forEach(System.out::println);

        // 3️⃣ FIND BY ID
        Author author = repo.findById(1)
                .orElseThrow(() -> new RuntimeException("Not found"));
        System.out.println(author);

        // 4️⃣ UPDATE
        author.setName("Ali Updated");
        repo.save(author);

        // 5️⃣ COUNT
        System.out.println("Count = " + repo.count());

        // 6️⃣ DELETE
        repo.deleteById(2);
    };
}
```

لو شغلت المشروع هتشوف كل العمليات حصلت على DB 🔥

---

# 🧠 نقطة Interview مهمة

ليه JpaRepository مش CrudRepository؟

لأن:

```
CrudRepository
   ↑
PagingAndSortingRepository
   ↑
JpaRepository
```

JpaRepository = كل حاجة فوق + Features زيادة.

---

# الخلاصة

انت دلوقتي تعلمت:

- Repository Pattern
    
- JpaRepository generics
    
- CRUD كامل
    
- Optional best practice
    
- تطبيق عملي
    

---

الخطوة الطبيعية بعد كده مباشرة 👀  
**Derived Query Methods** (أجمد ميزة في Spring Data)

جاهز نخش عليها؟ 😎




أيوه كلامك صح 100% 👌  
السطر ده مش Best Practice خالص:

```java
Author author = authorRepository.findById(1).get();
authorRepository.delete(author);
```

ليه؟ لأن `.get()` خطر جدًا 😈

---

# ❌ مشكلة findById().get()

الميثود `findById()` بترجع:

```java
Optional<Author>
```

يعني ممكن يرجع Author أو يرجع فاضي.

لما تعمل:

```java
optional.get();
```

لو الـ Author مش موجود → يحصل Crash:

```
NoSuchElementException
```

وده Exception وحش لأنه:

- مش واضح للمستخدم
    
- مش مناسب لـ REST API
    
- مش Professional
    

---

# ✅ الحل الصح (Best Practice)

## 1️⃣ استخدام orElseThrow()

وده الحل الاحترافي 👑

```java
Author author = authorRepository.findById(1)
        .orElseThrow(() -> new RuntimeException("Author not found"));

authorRepository.delete(author);
```

لو مش موجود:  
هيطلع Exception واضح برسالة مفهومة.

---

## 2️⃣ الأفضل في Spring Boot APIs 🔥

نعمل Exception مخصص:

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

ونستخدمه:

```java
Author author = authorRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Author not found with id " + id));

authorRepository.delete(author);
```

ده Professional جدًا وبيستخدم في الشركات.

---

# 🤯 طب ليه مش نستخدم deleteById وخلاص؟

سؤال عبقري 👏

## استخدم deleteById لو:

- مفيش business logic قبل الحذف
    
- مجرد حذف مباشر
    

```java
authorRepository.deleteById(id);
```

---

## استخدم delete(entity) لو فيه Business Logic قبل الحذف

مثلاً:

- ممنوع حذف Author لو عنده Books
    
- أو لازم نتحقق من Ownership
    
- أو Logging
    

```java
Author author = authorRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Author not found"));

if(!author.getBooks().isEmpty()){
    throw new RuntimeException("Cannot delete author with books");
}

authorRepository.delete(author);
```

💥 هنا لازم تجيب الـ entity.

---

# 🎯 خلاصة الاحتراف

|الطريقة|تقييم|
|---|---|
|findById().get()|❌ ممنوعة|
|findById().orElseThrow()|✅ الأفضل|
|deleteById()|✅ لو حذف مباشر|
|delete(entity)|✅ لو فيه Business Logic|

---

لو حابب نكمل بقى الجزء الجامد 🔥  
**Cascade Delete + orphanRemoval**  
وده أهم جزء في الحذف في JPA 💪