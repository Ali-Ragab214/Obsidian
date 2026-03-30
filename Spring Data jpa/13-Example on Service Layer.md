تمام 👌 أنا شايف الستراكشر بتاعك واضح جدًا، وده level كويس جدًا تبني عليه Service Layer بشكل احترافي.

خلينا نشرح **Service Layer من الصفر لحد best practices** على مشروعك الحالي 👇

---

# 🧠 أولًا: إحنا واقفين فين في المشروع؟

عندك دلوقتي:

- Entity ✅ (Author, Course…)
    
- Repository ✅
    
- DTO + Request ✅
    
- Mapper ✅
    
- Service (لسه بنبنيه صح) ⏳
    

يعني إحنا بنبني **العقل اللي بيربط كل ده ببعض**.

---

# 💙 يعني إيه Service Layer عمليًا؟

هو المكان اللي فيه:

> 🔥 Business Logic (قواعد الشغل بتاعة التطبيق)

مش مجرد CRUD وخلاص، لكن:

- قواعد
    
- Validations
    
- Decisions
    
- Orchestration بين layers
    

---

# 🧱 مكانه في الستراكشر

عندك:

```
controller → service → repository → database
```

---

# 🧠 مسؤوليات Service Layer

الـ Service هو المسؤول عن:

### ✅ 1. التعامل مع Repository

- save
    
- find
    
- delete
    
- update
    

---

### ✅ 2. تحويل البيانات باستخدام Mapper

- DTO → Entity
    
- Entity → DTO
    

---

### ✅ 3. تطبيق Business Logic

مثال:

- مينفعش email يتكرر
    
- مينفعش age أقل من 18
    
- حساب قيم معينة
    
- شروط معينة قبل الحفظ
    

---

### ✅ 4. التعامل مع Exceptions

- لو data مش موجودة
    
- لو validation فشلت
    

---

# 🚫 حاجات Service **ممنوع** يعملها

- ❌ مايتعاملش مع HTTP (Request/Response)
    
- ❌ مايرجعش Entity للـ Controller
    
- ❌ مايبقاش فيه logic خاص بالـ UI
    
- ❌ مايبقاش فيه SQL مباشر
    

---

# 🏗️ الشكل الصح للـ Service في مشروعك

أنت عندك structure ممتاز:

```
service
 ├── AuthorService
 └── impl
      └── AuthorServiceImpl
```

وده best practice 👍

---

# 🎯 ليه بنستخدم Interface + Impl؟

علشان:

- نفصل الـ contract عن التنفيذ
    
- نقدر نغير implementation بسهولة
    
- يسهل testing (mocking)
    
- clean architecture friendly
    

---

# ✨ شكل AuthorService (Interface)

```java
public interface AuthorService {

    AuthorDTO createAuthor(AuthorRequest request);

    List<AuthorDTO> getAllAuthors();

    AuthorDTO getAuthorById(Long id);

    AuthorDTO updateAuthor(Long id, AuthorRequest request);

    void deleteAuthor(Long id);
}
```

---

# ✨ شكل AuthorServiceImpl (Implementation)

```java
@Service
@RequiredArgsConstructor
public class AuthorServiceImpl implements AuthorService {

    private final AuthorRepository authorRepository;
```

هنا Spring بيعمل Dependency Injection.

---

# 🔥 نفهم كل Method بالتفصيل

---

## 1️⃣ CREATE

```java
@Override
public AuthorDTO createAuthor(AuthorRequest request) {

    // 1. Validation (business rules)
    if (authorRepository.existsByEmail(request.getEmail())) {
        throw new RuntimeException("Email already exists");
    }

    // 2. Convert Request → Entity
    Author author = AuthorMapper.toAuthor(request);

    // 3. Save
    Author saved = authorRepository.save(author);

    // 4. Convert Entity → DTO
    return AuthorMapper.toDto(saved);
}
```

💡 هنا حصل:

- validation
    
- mapping
    
- persistence
    
- response shaping
    

---

## 2️⃣ GET ALL

```java
@Override
public List<AuthorDTO> getAllAuthors() {

    return authorRepository.findAll()
            .stream()
            .map(AuthorMapper::toDto)
            .toList();
}
```

💡 Service هنا:

- جاب data
    
- حولها DTO
    
- رجعها
    

---

## 3️⃣ GET BY ID

```java
@Override
public AuthorDTO getAuthorById(Long id) {

    Author author = authorRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Author not found"));

    return AuthorMapper.toDto(author);
}
```

💡 مهم جدًا:

- التعامل مع Optional
    
- رمي Exception بدل null
    

---

## 4️⃣ UPDATE

```java
@Override
public AuthorDTO updateAuthor(Long id, AuthorRequest request) {

    Author author = authorRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Author not found"));

    // update fields
    author.setFirstname(request.getFirstname());
    author.setLastname(request.getLastname());
    author.setEmail(request.getEmail());
    author.setAge(request.getAge());

    Author updated = authorRepository.save(author);

    return AuthorMapper.toDto(updated);
}
```

💡 هنا:

- بنجيب الموجود
    
- نعدل عليه
    
- نحفظه
    

---

## 5️⃣ DELETE

```java
@Override
public void deleteAuthor(Long id) {

    Author author = authorRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Author not found"));

    authorRepository.delete(author);
}
```

💡 قبل الحذف بنأكد إنه موجود

---

# 🧠 أهم Concepts في Service Layer

## 1️⃣ Orchestration

Service بيجمع بين:

- Repository
    
- Mapper
    
- Validation
    

---

## 2️⃣ Transaction Boundary

في المشاريع الكبيرة:

```java
@Transactional
```

بيتستخدم على Service methods

---

## 3️⃣ Validation Layer (ممكن يبقى هنا أو منفصل)

مثلاً:

- email format
    
- required fields
    
- business constraints
    

---

## 4️⃣ Exception Handling Strategy

بدل RuntimeException الأفضل:

```java
throw new EntityNotFoundException("Author not found");
```

أو Custom Exception

---

# 🧱 Best Practice Structure داخل Service

خلي كل method تمشي بنفس pattern:

```text
1. Validate input
2. Fetch data (if needed)
3. Apply business logic
4. Map (Request ↔ Entity ↔ DTO)
5. Save / Return
```

---

# ⚡ ملخص ذهبي

Service Layer هو:

✅ قلب التطبيق  
✅ بيحتوي business logic  
✅ بيربط كل layers ببعض  
✅ بيتعامل مع repository  
✅ بيستخدم mapper  
✅ بيرجع DTO  
✅ بيمنع الـ controller من التعقيد

---

# 🚀 نصيحة مهمة جدًا

لو الـ Controller عندك فيه logic كتير → ده غلط ❌  
كل الـ logic يتحط في Service ✔️

---

لو حابب نكمل صح 🔥  
الخطوة الجاية المنطقية جدًا:

👉 نبني Controller بشكل احترافي باستخدام:

- ResponseEntity
    
- Status Codes
    
- Exception Handling
    
- Clean API design
    

وقتها مشروعك هيبقى production-level فعليًا 💯