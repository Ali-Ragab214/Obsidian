
---

# Pagination & Sorting في Spring Data JPA

## 1️⃣ ليه نستخدم Pagination؟

لما يكون عندك بيانات كبيرة في قاعدة البيانات، زي 1 مليون Author:

- لو جبت كل البيانات مرة واحدة = ضغط على الذاكرة وDB وAPI بطيء
    
- الحل = تقسيم البيانات على صفحات **Pages**.
    

---

## 2️⃣ المفاهيم الأساسية

|Concept|شرح|المكان اللي نستخدمه|
|---|---|---|
|**Pageable**|object بنحدّد فيه رقم الصفحة، حجم الصفحة، والترتيب|نعمله في **Service أو Controller** قبل ما نستدعي Repository|
|**Page**|object بيرجعلك البيانات + معلومات عن الصفحات|بيرجع من **Repository**|
|**Sort**|ترتيب النتائج (تصاعدي/تنازلي)|بيتضمّن في **Pageable**|

---

## 3️⃣ كيف نطبّقها خطوة خطوة

### Step 1: Repository

انت مش محتاج تضيف حاجة جديدة للـ Repository لأن JpaRepository جاهز فيه Pagination.

مثال على Derived Query + Pagination:

```java
public interface AuthorRepository extends JpaRepository<Author, Integer> {
    
    // جلب مؤلفين أكبر من عمر معين مع Pagination
    Page<Author> findByAgeGreaterThan(int age, Pageable pageable);
}
```

> **فين بيتحط؟**  
> في package `repositories` عندك، جنب الـ JpaRepository العادية.

---

### Step 2: Service

في الـ Service نجهز الـ Pageable ونمرره للـ Repository.

```java
@Service
public class AuthorService {

    private final AuthorRepository authorRepository;

    public AuthorService(AuthorRepository authorRepository) {
        this.authorRepository = authorRepository;
    }

    public Page<Author> getAuthorsByAge(int age, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        return authorRepository.findByAgeGreaterThan(age, pageable);
    }
}
```

> **فين بيتحط؟**  
> في package `service` عندك، جنب كل الخدمات اللي تتعامل مع الـ Repository.

---

### Step 3: Controller

في الـ Controller نستخدم الـ Service وناخد بيانات الصفحة من الـ RequestParams.

```java
@RestController
@RequestMapping("/authors")
public class AuthorController {

    private final AuthorService authorService;

    public AuthorController(AuthorService authorService) {
        this.authorService = authorService;
    }

    @GetMapping
    public Page<Author> getAuthors(
            @RequestParam int age,
            @RequestParam int page,
            @RequestParam int size) {
        return authorService.getAuthorsByAge(age, page, size);
    }
}
```

> **فين بيتحط؟**  
> في package `controller` عندك.

---

## 4️⃣ معلومات Page

الـ Page اللي بيرجع من Repository فيه:

```java
Page<Author> authorsPage = authorRepository.findAll(pageable);

authorsPage.getContent();       // البيانات الفعلية في الصفحة
authorsPage.getTotalElements(); // عدد كل السجلات
authorsPage.getTotalPages();    // عدد الصفحات الكلي
authorsPage.getNumber();        // رقم الصفحة الحالية
authorsPage.hasNext();          // هل فيه صفحة بعدها؟
authorsPage.isFirst();          // هل الصفحة الحالية هي الأولى؟
```

---

## 5️⃣ Sorting

ممكن نضيف Sort في Pageable:

```java
Pageable pageable = PageRequest.of(0, 5, Sort.by("age").descending().and(Sort.by("name").ascending()));
```

- رتب بالعمر تنازلي
    
- لو نفس العمر رتب بالاسم تصاعدي
    

---

## 6️⃣ مثال كامل Workflow

**Entity:**

```java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private int age;
    private String email;
}
```

**Repository:**

```java
public interface AuthorRepository extends JpaRepository<Author, Integer> {
    Page<Author> findByAgeGreaterThan(int age, Pageable pageable);
}
```

**Service:**

```java
@Service
public class AuthorService {

    private final AuthorRepository authorRepository;

    public AuthorService(AuthorRepository authorRepository) {
        this.authorRepository = authorRepository;
    }

    public Page<Author> getAuthorsByAge(int age, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("name").ascending());
        return authorRepository.findByAgeGreaterThan(age, pageable);
    }
}
```

**Controller:**

```java
@RestController
@RequestMapping("/authors")
public class AuthorController {

    private final AuthorService authorService;

    public AuthorController(AuthorService authorService) {
        this.authorService = authorService;
    }

    @GetMapping
    public Page<Author> getAuthors(
            @RequestParam int age,
            @RequestParam int page,
            @RequestParam int size) {
        return authorService.getAuthorsByAge(age, page, size);
    }
}
```

**API Request Example:**

```
GET /authors?age=20&page=0&size=5
```

يرجع لك:

- أول 5 Authors أكبر من 20 سنة
    
- مرتبين بالاسم تصاعدي
    
- مع كل بيانات الـ Page (عدد الصفحات، العناصر، إلخ)
    

---

لو تحب أعمللك بعد كده **مثال عملي مع Pagination + Sorting + @Query** بحيث تجمع كل اللي اتعلمته لحد دلوقتي وتقدر تجربها فوراً على مشروعك 😏🔥

هل أعملهولك؟