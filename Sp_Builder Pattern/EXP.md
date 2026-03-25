
---

# 🧠 أولًا: يعني إيه Design Pattern؟

الـ Design Pattern هو:

> حل جاهز لمشكلة متكررة في البرمجة

---

# 🏗️ Builder Pattern (الفكرة الأساسية)

الـ **Builder Pattern** بيستخدم لما يكون عندك Object:

- فيه Fields كتير
    
- أو فيه Optional Fields
    
- أو Construction معقد
    

👉 فبدل ما تعمل Constructor كبير ومقرف  
بنقسم عملية البناء step-by-step

---

# 😵 المشكلة (بدون Builder)

```java
public class User {
    private String name;
    private String email;
    private String phone;
    private int age;
    private String address;

    public User(String name, String email, String phone, int age, String address) {
        this.name = name;
        this.email = email;
        this.phone = phone;
        this.age = age;
        this.address = address;
    }
}
```

### الاستخدام:

```java
User user = new User("Ali", "ali@mail.com", null, 22, null);
```

❌ مشاكل:

- مش واضح ايه null ده
    
- ترتيب غلط = bug
    
- صعب جدًا تقرأ
    

---

# 💡 الحل: Builder Pattern

بنخلي إنشاء الـ object يتم خطوة خطوة

---

# 🧱 Implementation (Manual Builder)

```java
public class User {

    private String name;
    private String email;
    private String phone;
    private int age;
    private String address;

    // Constructor خاص
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.phone = builder.phone;
        this.age = builder.age;
        this.address = builder.address;
    }

    // Builder Class
    public static class Builder {
        private String name;
        private String email;
        private String phone;
        private int age;
        private String address;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

---

# 🚀 الاستخدام

```java
User user = new User.Builder()
        .name("Ali")
        .email("ali@mail.com")
        .age(22)
        .build();
```

---

# 🔥 ليه ده أحسن؟

✅ readable جدًا  
✅ مفيش ترتيب إجباري  
✅ تقدر تضيف optional fields بسهولة  
✅ immutable design (لو عايز)

---

# ⚡ استخدام Lombok (اللي بيحصل في الحقيقة)

بدل كل الكود ده 😅  
بنستخدم:

👉 Project Lombok

```java
import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class User {
    private String name;
    private String email;
    private int age;
}
```

---

### الاستخدام:

```java
User user = User.builder()
        .name("Ali")
        .email("ali@mail.com")
        .age(22)
        .build();
```

🔥 نفس الفكرة بدون وجع دماغ

---

# 🧠 امتى تستخدم Builder؟

استخدمه لما:

✔ عندك Object كبير  
✔ فيه Optional Fields كتير  
✔ عايز readability  
✔ عايز تمنع Constructor hell

---

# ❌ امتى متستخدموش؟

❌ Object صغير (2-3 fields)  
❌ مش محتاج flexibility

---

# 🆚 مقارنة سريعة

|الطريقة|العيوب|
|---|---|
|Constructor|صعب + غير واضح|
|Setters|mutable + unsafe|
|Builder|الأفضل 👍|

---

# 💣 حتة جامدة (Interview Level)

## Immutable Object باستخدام Builder

```java
public class User {
    private final String name;
    private final String email;

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
    }
}
```

✔ كده الobject مش بيتغير بعد ما يتبني  
✔ ده design قوي جدًا في backend

---

# 🧩 بيستخدم فين في الواقع؟

في شغلك كـ Backend Developer:

- DTOs
    
- API Responses
    
- Entities (أحيانًا)
    
- Config Objects
    

---

# 🧠 الخلاصة

- Builder Pattern = طريقة محترفة لإنشاء Objects
    
- بيحل مشكلة Constructors الكبيرة
    
- بيخلي الكود readable و flexible
    
- غالبًا بنستخدمه مع Lombok
    

---

لو عايز بقى المستوى اللي بعده 🔥  
أشرحلك:

- الفرق بين Builder و Factory Pattern
    
- أو نطبقه في مشروع Spring (DTO + Entity + API)
    

قولّي وهنخش على Level أعلى 👌