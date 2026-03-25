

هشرحلك **أول جزء (Foundations)** بشكل منظم كأنك داخل كورس احترافي 👇

---

# 🎓 المرحلة 1: أساسيات Spring Data JPA

## 🟢 1. أنت وصلت لفين دلوقتي؟

✔ عملت connection مع DB  
✔ ظهّرت الداتابيز في IntelliJ  
✔ استخدمت Lombok

💯 ممتاز… ده 30% من البداية

---

# 🧠 2. الصورة الكبيرة (Big Picture)

قبل ما نكتب كود، لازم تفهم الدنيا ماشية إزاي:

```
Controller → Service → Repository → Database
```

لكن في المرحلة دي هنركز على:

```
Entity → Repository → Database
```

---

# 🟡 3. Entity (أهم خطوة 🔥)

## ✅ يعني إيه Entity؟

> كلاس بيمثل جدول في الداتابيز

---

## 🔥 مثال عملي

```java
import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;

    private String email;
}
```

---

## 🎯 شرح مهم جدًا

### `@Entity`

👉 ده بيقول: الكلاس ده جدول في الداتابيز

---

### `@Table`

👉 اسم الجدول

---

### `@Id`

👉 Primary Key

---

### `@GeneratedValue`

👉 الداتابيز تولّد ID

---

### Lombok

بيعمل:

- Getter / Setter
    
- Constructor
    
- Builder
    

👉 بدل ما تكتبهم بإيدك

---

# 🟠 4. Repository (السحر هنا 🧠🔥)

## ✅ يعني إيه Repository؟

> Interface بيتعامل مع الداتابيز بدل ما تكتب SQL

---

## 🔥 مثال

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Integer> {
}
```

---

## 🎯 حصل إيه هنا؟

بمجرد ما تعمل كده 👇

```java
extends JpaRepository<User, Integer>
```

Spring بيديك جاهز:

|Method|الوظيفة|
|---|---|
|save()|insert / update|
|findById()|يجيب record|
|findAll()|يجيب كل البيانات|
|deleteById()|حذف|

💥 بدون ما تكتب SQL

---

# 🔵 5. نجرب عملي (Flow كامل)

## إنشاء User

```java
User user = User.builder()
        .name("Ali")
        .email("ali@gmail.com")
        .build();

userRepository.save(user);
```

---

## قراءة البيانات

```java
List<User> users = userRepository.findAll();
```

---

# 🧪 6. مهم جدًا (Debug & فهم)

لما تعمل save:

### بيحصل:

```sql
INSERT INTO users (name, email) VALUES (?, ?);
```

---

# ⚠️ أخطاء لازم تاخد بالك منها

### ❌ نسيان @Entity

→ مش هيتعمل table

### ❌ نوع ID غلط

→ لازم `Integer` مش `int`

### ❌ مشكلة Lombok

→ لازم Annotation Processing يكون شغال

---

# 🚀 أنت كده خلصت إيه؟

✔ Entity  
✔ Repository  
✔ Basic CRUD

💥 ده اسمه:

> **Spring Data JPA Core Basics**

---

# 🔥 الخطوة الجاية (مهم جدًا)

لو عايز تكمل صح:

## 🧭 Next Step:

1. Custom Queries
    
2. Query Methods (findByName)
    
3. Relationships (OneToMany 🔥)
    
4. DTOs (مهم جدًا)
    
5. Service Layer
    

---

# 💡 نصيحة مني ليك

بلاش تحفظ… افهم flow:

> "أنا بعمل class → يتحول table → أتعامل معاه بال repository"

---

