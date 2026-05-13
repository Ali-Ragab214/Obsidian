
---

# أولاً: Builder بيتصنف تحت ايه؟

Builder واحد من **Creational Design Patterns**  
يعني مسؤول عن **إنشاء الـ Objects**.

---

# المشكلة اللي حلّها Builder

تخيل عندك Object كبير فيه اختيارات كتير.

مثال: **User / Car / Pizza / Order**

خلينا ناخد مثال User 👇

المستخدم ممكن يكون عنده:

- name (إجباري)
    
- email (إجباري)
    
- phone (اختياري)
    
- address (اختياري)
    
- age (اختياري)
    
- gender (اختياري)
    

---

## الحل الغلط رقم 1 ❌

Constructor فيه كل حاجة:

```java
User(String name, String email, String phone, String address, int age, String gender)
```

ولما نستخدمه:

```java
new User("Ali","ali@mail.com", null, null, 0, null);
```

😵‍💫 كارثة:

- مش واضح ايه null ده
    
- صعب القراءة
    
- كل ما تزود field تكسر الكود
    

---

## الحل الغلط رقم 2 ❌ (Setters)

```java
User user = new User();
user.setName("Ali");
user.setEmail("ali@mail.com");
user.setAge(25);
```

المشكلة:

- ممكن تنسى تحط name أو email
    
- الobject ممكن يبقى ناقص وغير صالح ❌
    

---

# هنا يظهر Builder 💡

Builder يسمح لك:

- تبني Object خطوة خطوة
    
- تحدد الاختياري براحتك
    
- وتطلع Object جاهز وصحيح في الآخر
    

زي طلب بيتزا 🍕:

- عجينة
    
- جبنة
    
- زيتون
    
- مشروم
    

مش لازم تحط كل حاجة.

---

# الفكرة ببساطة

بدل ما تبني Object مرة واحدة  
تبنيه **خطوة خطوة** ثم تقول له: build()

---

# نطبق المثال 👨‍💻

## 1️⃣ نعمل الكلاس الأساسي (User)

```java
public class User {

    private String name;
    private String email;
    private String phone;
    private String address;
    private int age;

    private User(UserBuilder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
        this.age = builder.age;
    }
```

لاحظ 👀  
الـ constructor بقى **private**  
محدش يقدر يعمل new User مباشرة.

---

## 2️⃣ نعمل Builder داخل الكلاس

```java
    public static class UserBuilder {

        private String name;
        private String email;
        private String phone;
        private String address;
        private int age;
```

---

## 3️⃣ constructor للـ Builder (للبيانات الإجباري)

```java
        public UserBuilder(String name, String email) {
            this.name = name;
            this.email = email;
        }
```

الـ name و email بقوا mandatory ✔️

---

## 4️⃣ Methods للـ fields الاختيارية

لاحظ الحركة الجميلة 😍  
كل method ترجع نفس الـ builder.

```java
        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
```

ده اسمه:  
👉 Method Chaining

---

## 5️⃣ أهم خطوة: build()2

```java
        public User build() {
            return new User(this);
        }
    }
}
```

هنا يتم إنشاء الـ Object الحقيقي.

---

# استخدام Builder 😍

هنا يظهر الجمال الحقيقي:

```java
User user = new User.UserBuilder("Ali", "ali@mail.com")
                    .phone("0100000")
                    .address("Cairo")
                    .age(25)
                    .build();
```

واضح جداً 👌  
مفيش null  
مفيش لخبطه  
Readable جداً.

---

# لو مش عايز تحط كل الحقول

عادي جداً:

```java
User user = new User.UserBuilder("Ali", "ali@mail.com")
                    .age(25)
                    .build();
```

---

# نشرح الفكرة مرة أخيرة ببساطة جداً

Builder يعني:

- Object كبير ومعقد
    
- نبنيه خطوة خطوة
    
- نحدد الإجباري والاختياري
    
- نمنع إنشاء Object ناقص
    

---

# إمتى أستخدم Builder؟

لو عندك:

- Object فيه optional fields كتير
    
- Constructor كبير ومزعج
    
- عايز object immutable (أفضل practice)
    

---

# الفرق السريع بين التلاتة

|Pattern|وظيفته|
|---|---|
|Singleton|Object واحد فقط|
|Factory|إنشاء Object بدون معرفة نوعه|
|Abstract Factory|إنشاء عائلة Objects|
|Builder|بناء Object كبير خطوة خطوة|

---

لو حابب بعد كده نطبق Builder على مثال E-commerce Order أو API Request DTO قولّي 👍

سؤال جامد 👌 انت دلوقتي بتتكلم عن **Lombok @Builder** صح؟  
يعني بدل ما نكتب Builder بإيدينا.

الإجابة المختصرة:  
👉 أيوه فيه طريقة أسهل باستخدام Lombok  
👉 ودي غالباً **أفضل في المشاريع العملية**

خلينا نقارن الاثنين ونفهم الصح فين.

---

# أولاً: الطريقة اللي كتبناها بإيدينا (Manual Builder)

كتبنا:

- inner static class
    
- methods chaining
    
- build()
    

الميزة:  
✔️ فاهم الـ pattern صح  
✔️ مناسب للتعلم والانترفيو

العيب:  
❌ كود كتير جداً boilerplate

في الشغل الحقيقي محدش بيكتب ده بإيده 😄

---

# الحل العملي في الشركات 🔥

نستخدم **Project Lombok**

Lombok بيكتب الكود بدلنا وقت الـ compile.

---

# الطريقة الأفضل: Lombok @Builder

## 1️⃣ ضيف Lombok للمشروع

لو Maven:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
```

---

## 2️⃣ نكتب الكلاس بشكل نظيف جداً

بدل 100 سطر… هيبقى 5 أسطر 😍

```java
import lombok.Builder;
import lombok.ToString;

@Builder
@ToString
public class User {

    private String name;
    private String email;
    private String phone;
    private String address;
    private int age;
}
```

خلصنا 😄  
Lombok أنشأ Builder كامل في الخلفية.

---

## 3️⃣ إنشاء Object خارج الكلاس (زي ما قلت)

```java
User user = User.builder()
        .name("Ali")
        .email("ali@mail.com")
        .age(25)
        .build();

System.out.println(user);
```

سهل جداً وواضح جداً.

---

# طيب الفرق الحقيقي بين الطريقتين؟

|Manual Builder|Lombok Builder|
|---|---|
|تتعلم منه الpattern|تستخدمه في الشغل|
|كود كتير|كود نظيف|
|مجهود عالي|سطر واحد annotation|
|صعب الصيانة|سهل التعديل|

👉 في المشاريع الحقيقية: Lombok هو الأفضل.

---

# سؤال مهم جداً (Interview Trap) ⚠️

هل Lombok بديل للـ Builder Pattern؟

الإجابة:  
❌ لا  
✔️ Lombok مجرد أداة **تولّد الكود**  
لكن الفكرة مازالت Builder Pattern.

---

# أفضل Practice في Backend

في مشاريع Spring / Clean Architecture:

نستخدم @Builder في:

- DTOs
    
- Entities
    
- Response Models
    
- Request Models
    

مثال DTO:

```java
@Builder
public record CreateUserRequest(
        String name,
        String email,
        String phone
) {}
```

واستخدامه:

```java
CreateUserRequest req = CreateUserRequest.builder()
        .name("Ali")
        .email("ali@mail.com")
        .phone("010000")
        .build();
        
```

---

# الخلاصة المهمة 💪

لو بتتعلم → اكتب Builder بإيدك  
لو بتشتغل → استخدم Lombok @Builder

وده فعلاً اللي بيحصل في الشركات.

---

لو حابب أقولك امتى ماينفعش تستخدم Lombok Builder (في حالات معينة) قولّي 👍