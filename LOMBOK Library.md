

# 🧠 يعني ايه Lombok؟

**Project Lombok** هو:

> ✅ مكتبة (Library) في Java  
> وظيفتها تقلل الكود المكرر (Boilerplate Code)

---

# 🔥 المشكلة اللي بيحلها

في Java الطبيعي 👇

```java
public class Author {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

😑 كود كتير على حاجة بسيطة

---

# ⚡ الحل مع Lombok

```java
@Getter
@Setter
public class Author {
    private String name;
}
```

🔥 نفس النتيجة… بكود أقل بكتير

---

# 💡 Lombok بيشتغل إزاي؟ (مهم جدًا)

👉 هو بيستخدم حاجة اسمها:  
**Annotation Processing**

يعني:

- انت تكتب `@Getter`
    
- وقت الـ Compile
    
- Lombok يولد الكود الحقيقي (getter method)
    

📌 يعني الكود موجود… بس **مش ظاهرلك**

---

# 🧩 أشهر استخداماته

## ✅ 1. Entities (مع JPA)

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Author {
}
```

---

## ✅ 2. DTOs

```java
@Getter
@Setter
@AllArgsConstructor
public class AuthorRequest {
    private String name;
}
```

---

## ✅ 3. Constructors

```java
@RequiredArgsConstructor
```

→ يعمل constructor للـ fields المهمة

---

## ✅ 4. Builder Pattern 🔥

```java
@Builder
public class Author {
    private String name;
}
```

تستخدمه كده:

```java
Author a = Author.builder()
                 .name("Ali")
                 .build();
```

---

# ⚠️ عيوب Lombok (لازم تعرفها)

## ❌ 1. الكود مش ظاهر

- ممكن يلغبطك في الأول
    
- صعب debugging أحيانًا
    

---

## ❌ 2. مشاكل مع JPA لو استخدمت غلط

- زي `@Data` مع العلاقات
    

---

## ❌ 3. بيعتمد على IDE

- لازم Plugin + settings مظبوطة
    

---

# 🆚 تستخدمه ولا لا؟

## 🟢 استخدمه لو:

- شغال Spring Boot
    
- عايز تقلل الكود
    
- فاهم هو بيعمل ايه
    

## 🔴 متستخدموش لو:

- لسه مبتدئ ومش فاهم getters/setters
    
- أو في سيستم strict جدًا
    

---

# 🎯 الخلاصة (احفظها كده)

|السؤال|الإجابة|
|---|---|
|Lombok ايه؟|Library|
|بيعمل ايه؟|يولد كود أوتوماتيك|
|بيشتغل ازاي؟|Annotation Processing|
|بنستخدمه فين؟|Entities + DTOs + Services|

---

# 💪 نصيحة قوية ليك

بما إنك Backend Advanced 👇

👉 لازم تبقى فاهم الاتنين:

- تكتب الكود بإيدك
    
- تستخدم Lombok
    

عشان:

> في الإنترفيو ممكن يقولك "اكتب getter بإيدك"

---

لو عايز 🔥  
أديك **تمرين عملي + mini project** تستخدم فيه Lombok مع Spring + JPA + Hibernate قولي وهنطبقه سوا 💪