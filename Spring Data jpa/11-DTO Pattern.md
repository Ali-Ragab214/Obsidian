

---

# 🔹 أولاً: يعني إيه DTO؟

DTO = **Data Transfer Object**

👉 ببساطة:  
هو كلاس بنستخدمه **علشان ننقل الداتا بين layers** (خصوصًا بين الـ Controller و الـ Client) بدل ما نستخدم الـ Entity مباشرة.

---

# ❌ ليه ما نستخدمش الـ Entity مباشرة؟

لو رجعت الـ Entity زي ما هي:

- ممكن تبعت بيانات حساسة (زي password ❌)
    
- تعمل مشاكل في الـ Lazy Loading
    
- تربط الـ API بالـ DB structure (وده غلط في Clean Architecture)
    

---

# ✅ ليه نستخدم DTO؟

- نتحكم في البيانات اللي بتطلع
    
- نفصل بين الـ DB والـ API
    
- أمان أعلى 🔐
    
- كود أنضف وقابل للتطوير
    

---

# 💡 مثال كامل (User System)

هنعمل:

- Entity
    
- DTO
    
- Repository
    
- Service
    
- Controller
    

---

# 🧱 1. Entity

```java
package com.example.demo.entity;

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
    private Long id;

    private String name;

    private String email;

    private String password; // ⚠️ مش عايزين نطلعه للـ API
}
```

---

# 📦 2. DTO

```java
package com.example.demo.dto;

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserDTO {

    private Long id;
    private String name;
    private String email;

    // ❌ مفيش password هنا
}
```

---

# 🗄️ 3. Repository

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

# 🔄 4. Mapper (احترافي 🔥)

بدل ما نكتب التحويل كل مرة، نعمل Mapper

```java
package com.example.demo.mapper;

import com.example.demo.dto.UserDTO;
import com.example.demo.entity.User;

public class UserMapper {

    public static UserDTO toDTO(User user) {
        return UserDTO.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .build();
    }

    public static User toEntity(UserDTO dto) {
        return User.builder()
                .id(dto.getId())
                .name(dto.getName())
                .email(dto.getEmail())
                .build();
    }
}
```

---

# ⚙️ 5. Service

```java
package com.example.demo.service;

import com.example.demo.dto.UserDTO;
import com.example.demo.entity.User;
import com.example.demo.mapper.UserMapper;
import com.example.demo.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    // GET ALL
    public List<UserDTO> getAllUsers() {
        return userRepository.findAll()
                .stream()
                .map(UserMapper::toDTO)
                .collect(Collectors.toList());
    }

    // CREATE
    public UserDTO createUser(UserDTO dto) {
        User user = UserMapper.toEntity(dto);

        // مثال: نضيف password داخليًا
        user.setPassword("default123");

        User savedUser = userRepository.save(user);

        return UserMapper.toDTO(savedUser);
    }
}
```

---

# 🌐 6. Controller

```java
package com.example.demo.controller;

import com.example.demo.dto.UserDTO;
import com.example.demo.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public List<UserDTO> getAllUsers() {
        return userService.getAllUsers();
    }

    @PostMapping
    public UserDTO createUser(@RequestBody UserDTO dto) {
        return userService.createUser(dto);
    }
}
```

---

# 🔥 النتيجة

## لما تعمل GET:

```json
[
  {
    "id": 1,
    "name": "Ali",
    "email": "ali@gmail.com"
  }
]
```

👉 مفيش password 👌

---

# 💡 نصايح احترافية

- اعمل DTOs مختلفة:
    
    - `UserRequestDTO`
        
    - `UserResponseDTO`
        
- استخدم مكتبات زي:
    
    - MapStruct (احترافي جدًا)
        
- متستخدمش Entity في Controller نهائي ❌
    

---

# 🧠 الخلاصة

- Entity = شكل الداتا في DB
    
- DTO = شكل الداتا اللي بتطلع/تدخل API
    
- Mapper = اللي بيربط بينهم
    

---

لو عايز بقى 👀  
أعملك نفس المثال بـ:

- validation 🔥
    
- exception handling 🔥
    
- MapStruct بدل manual 🔥
    
- Clean Architecture multi-module 🔥
    

قولّي وهنفجر المستوى 🚀

---
---
---
---
تمام يا علي 👌 خلينا نمشي خطوة خطوة بشكل عملي ونفهم **DTO** صح ونطبقه على الكود بتاعك 💪

---

# 🔥 أولاً: يعني إيه DTO ؟

DTO = **Data Transfer Object**

ببساطة:

> كلاس بنستخدمه علشان ننقل الداتا بين layers (Controller ↔ Service ↔ Client) بدل ما نستخدم الـ Entity مباشرة

---

# ❌ ليه منستخدمش Entity مباشرة؟

عندك `Author` فيه:

```java
private String firstname;
private String lastname;
private String email;
private int age;
private List<Course> courses;
```

### المشاكل:

1. 🔁 **Circular Reference**
    
    - Author → Course → Author → 💥 error في JSON
        
2. 🔒 **Security**
    
    - ممكن تبعت email أو data مش عايز تظهرها
        
3. 🧠 **Separation of concerns**
    
    - الـ Entity مربوط بالداتابيز
        
    - لكن الـ API ليه شكل مختلف
        

---

# ✅ الحل: DTO

نعمل كلاس خفيف يمثل الداتا اللي عايزين نطلعها بس

---

# 🏗️ Structure صح (مهم جداً)

اعمل فولدرين 👇

```
dto/
mapper/
```

---

# 📁 1. DTO Layer

- الـ **Entity** فيه كل حاجة (ومرتبط بالداتابيز)
- الـ **DTO** للعرض (Response)
- الـ **Request** للإدخال (Input)
## ✨ AuthorDTO
### ✳️ AuthorDTO (Response)

ده اللي بيرجع لليوزر.

مش لازم ترجّع كل حاجة في الـ Entity  
مثلاً ممكن تخبي:

- password
- internal flags
- relations تقيلة

```java
package com.notification.jpa.dto;

import lombok.*;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class AuthorDTO {

    private String firstname;
    private String lastname;
    private String email;
}
```

💡 هنا:

- شيلنا `courses`
    
- شيلنا `age` (اختياري حسب اليوز كيس)
    

---

## ✨ AuthorRequest (optional - للـ create/update)
`AuthorRequest` هو **شكل الداتا اللي بتيجي من المستخدم (Input)**

### ✳️ AuthorRequest

ده بيمثل البيانات اللي اليوزر يبعتهالك.

مثلاً اليوزر مش المفروض يبعث:

- id
- createdDate
- internal fields

فبنحدد البيانات اللي مسموح يدخلها بس.

```java
package com.notification.jpa.dto;

import lombok.*;

@Data
public class AuthorRequest {
    private String firstname;
    private String lastname;
    private String email;
    private int age;
}
```

---

# 🔄 2. Mapper Layer

ده اللي بيحول:

```
Entity → DTO
DTO → Entity
```

---

## ✨ AuthorMapper
ده كلاس وظيفته الأساسية:

👉 **تحويل بين الأنواع (Conversions)**

- Entity → DTO
- DTO → Entity
- Request → Entity
- Entity → Response DTO
## 🎯 ليه نستخدمه؟

بدل ما تكتب التحويل ده في كل Service، نخليه في مكان واحد:

✅ Clean Code  
✅ Reusable  
✅ Separation of concerns

```java
package com.notification.jpa.mapper;

import com.notification.jpa.dto.AuthorDTO;
import com.notification.jpa.models.Author;

public class AuthorMapper {

    public static AuthorDTO toDTO(Author author) {
        return AuthorDTO.builder()
                .firstname(author.getFirstname())
                .lastname(author.getLastname())
                .email(author.getEmail())
                .build();
    }

    public static Author toEntity(AuthorDTO dto) {
        return Author.builder()
                .firstname(dto.getFirstname())
                .lastname(dto.getLastname())
                .email(dto.getEmail())
                .build();
    }
}
```

---

# 🧠 نستخدم DTO فين؟

## ❌ غلط

Controller يتعامل مع Entity

## ✅ صح

Controller يتعامل مع DTO

---

# 💡 مثال عملي

## Service

```java
@Service
@RequiredArgsConstructor
public class AuthorService {

    private final AuthorRepository repository;

    public List<AuthorDTO> getAllAuthors() {
        return repository.findAll()
                .stream()
                .map(AuthorMapper::toDTO)
                .toList();
    }
}
```

---

## Controller

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/authors")
public class AuthorController {

    private final AuthorService service;

    @GetMapping
    public List<AuthorDTO> getAll() {
        return service.getAllAuthors();
    }
}
```

---

# 🔥 Flow النهائي

```
Controller (DTO)
        ↓
Service
        ↓
Mapper
        ↓
Repository
        ↓
Database
```

---

# 🤯 ليه DTO مهم جداً (خلاصة)

- 🔒 Security (تخبي data)
    
- 🚀 Performance (تبعت أقل data)
    
- 🔄 Flexibility (تغير API براحتك)
    
- 🧼 Clean Architecture


# 🔄 العلاقة بينهم في السيناريو الكامل

## 1. Controller يستقبل Request

@PostMapping  
public AuthorDTO create(@RequestBody AuthorRequest request) {  
    return service.createAuthor(request);  
}

---

## 2. Service يستخدم Mapper

public AuthorDTO createAuthor(AuthorRequest request) {  
    Author author = AuthorMapper.toEntity(request);  
  
    Author saved = repository.save(author);  
  
    return AuthorMapper.toDTO(saved);  
}

---

# 🧠 الخلاصة

|العنصر|وظيفته|
|---|---|
|`AuthorRequest`|يستقبل الداتا من المستخدم (Input)|
|`AuthorDTO`|يرجع الداتا للمستخدم (Output)|
|`AuthorMapper`|يحول بين الأنواع المختلفة|

---

# ⚡ تشبيه بسيط

تخيل:

- `Request` = الورقة اللي العميل بيملأها
- `Entity` = الداتا اللي بتتخزن في الأرشيف
- `DTO` = الفاتورة/النتيجة اللي بتترد
- `Mapper` = الموظف اللي بيحوّل الورق بين الأشكال دي

---

لو حابب، أشرح لك بعدها:

- الفرق بين DTO و Request عمليًا في مشاريع كبيرة
- أو إزاي تعمل Mapper باستخدام MapStruct بدل الكود اليدوي 👍

---


    

---

