تمام، دلوقتي هشرح لك `Optional` بكل تفاصيله وأعمل مثال كامل.

---

## ما هو `Optional`؟

`Optional<T>` هو **wrapper class** في Java بيغلف قيمة ممكن تكون موجودة أو `null`. اتضاف في Java 8 عشان يحل مشكلة `NullPointerException` الشهيرة.

**المشكلة اللي بيحلها:**

```java
// ❌ قديم — ممكن يطلع NullPointerException
User user = userRepo.findById(id); // لو مش موجود → null
System.out.println(user.getName()); // 💥 BOOM
```

```java
// ✅ مع Optional
Optional<User> user = userRepo.findById(id);
user.ifPresent(u -> System.out.println(u.getName())); // آمن تماماً
```

---

## الميثودز الأساسية

### `isPresent()`

بتسأل: "في قيمة جوا؟" — ترجع `true` أو `false`

```java
Optional<String> name = Optional.of("Ahmed");
if (name.isPresent()) {
    System.out.println(name.get()); // "Ahmed"
}
```

### `orElse(defaultValue)`

لو القيمة مش موجودة، ارجع القيمة الافتراضية

```java
Optional<String> empty = Optional.empty();
String result = empty.orElse("Unknown"); // → "Unknown"
```

### `orElseThrow()`

لو القيمة مش موجودة، throw exception — **ده اللي بتستخدمه في Spring مع entities**

```java
User user = userRepo.findById(id)
    .orElseThrow(() -> new RuntimeException("User not found"));
```

---

**دلوقتي المثال الكامل 👇**---

## الخلاصة السريعة

**امتى تستخدم كل method؟**

- **`orElseThrow()`** → في REST APIs دايماً — لما تعمل GET/PUT/DELETE على entity بـ id
- **`orElse()`** → لما عندك fallback value جاهزة ومش فيها حسابات
- **`isPresent()`** → لما محتاج logic مختلف على حسب الوجود من عدمه
- **`ifPresent()`** → لما عايز تعمل side-effect بس مش هترجع حاجة
- **`map()`** → لما عايز تستخرج field من جوا الـ Optional

**القاعدة الذهبية:** متستخدمش `.get()` على طول من غير ما تتأكد — استخدم `orElseThrow()` بدله دايماً.



---
---
```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    private Double price;

    private String category;

    // Constructors
    public Product() {}

    public Product(String name, Double price, String category) {
        this.name     = name;
        this.price    = price;
        this.category = category;
    }

    // Getters & Setters (omitted for brevity)
    public Long   getId()       { return id; }
    public String getName()     { return name; }
    public Double getPrice()    { return price; }
    public String getCategory() { return category; }

    public void setName(String name)         { this.name = name; }
    public void setPrice(Double price)       { this.price = price; }
    public void setCategory(String cat)      { this.category = cat; }
}
```

```java 
@Repository
public interface ProductRepository
        extends JpaRepository<Product, Long> {

    // ✅ JPA بيرجع Optional تلقائياً
    Optional<Product> findById(Long id);

    // ✅ Custom query — بترجع Optional لو بتدور على record واحد
    Optional<Product> findByName(String name);

    // ✅ بترجع List مش Optional (أكتر من نتيجة)
    List<Product> findByCategory(String category);

    // ✅ Optional لأول نتيجة بالسعر
    Optional<Product> findFirstByPriceLessThan(Double price);
}
```

```java 
@Service
public class ProductService {

    @Autowired
    private ProductRepository repo;

    // ━━━ 1. orElseThrow ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // لو مش موجود → throw custom exception
    // ده الأكثر استخداماً في Spring APIs
    public Product getById(Long id) {
        return repo.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(
                "Product not found with id: " + id
            ));
    }

    // ━━━ 2. orElse ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // لو مش موجود → ارجع قيمة افتراضية
    public Product getByNameOrDefault(String name) {
        Product defaultProduct = new Product("Unknown", 0.0, "N/A");
        return repo.findByName(name)
            .orElse(defaultProduct);
    }

    // ━━━ 3. isPresent ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // لما محتاج تعمل logic مختلف على حسب الوجود
    public String checkProduct(Long id) {
        Optional<Product> opt = repo.findById(id);
        if (opt.isPresent()) {
            return "Found: " + opt.get().getName();
        } else {
            return "Product #" + id + " does not exist";
        }
    }

    // ━━━ إضافي: ifPresent ━━━━━━━━━━━━━━━━━━━━━━━━━━━
    // نفذ action لو الـ Optional فيه قيمة
    public void printIfExists(Long id) {
        repo.findById(id)
            .ifPresent(p -> System.out.println("Product: " + p.getName()));
    }

    // ━━━ Create ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    public Product save(Product p) {
        return repo.save(p);
    }

    // ━━━ Update ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    public Product update(Long id, Product updated) {
        Product existing = repo.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(
                "Cannot update — not found: " + id));
        existing.setName(updated.getName());
        existing.setPrice(updated.getPrice());
        existing.setCategory(updated.getCategory());
        return repo.save(existing);
    }

    // ━━━ Delete ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    public void delete(Long id) {
        repo.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(
                "Cannot delete — not found: " + id));
        repo.deleteById(id);
    }
}

Copy failed — try from claude.ai in browser
```

```java 
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService service;

    // GET /api/products/{id}
    // لو مش موجود → service بترمي exception → handler يرجع 404
    @GetMapping("/{id}")
    public ResponseEntity<Product> getById(@PathVariable Long id) {
        Product product = service.getById(id);
        return ResponseEntity.ok(product);
    }

    // GET /api/products/check/{id}
    @GetMapping("/check/{id}")
    public ResponseEntity<String> check(@PathVariable Long id) {
        return ResponseEntity.ok(service.checkProduct(id));
    }

    // POST /api/products
    @PostMapping
    public ResponseEntity<Product> create(@RequestBody Product p) {
        Product saved = service.save(p);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(saved);
    }

    // PUT /api/products/{id}
    @PutMapping("/{id}")
    public ResponseEntity<Product> update(
            @PathVariable Long id,
            @RequestBody  Product p) {
        return ResponseEntity.ok(service.update(id, p));
    }

    // DELETE /api/products/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();
    }
}

Copy failed — try from claude.ai in browser
```

```java 
// ── 1. Custom Exception ──────────────────────────────
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(String message) {
        super(message);
    }
}

// ── 2. Global Exception Handler ──────────────────────
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleNotFound(
            ProductNotFoundException ex) {

        Map<String, String> body = new HashMap<>();
        body.put("error",   "Not Found");
        body.put("message", ex.getMessage());

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)  // → HTTP 404
            .body(body);
    }
}
```


```java
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
//  METHOD          متى تستخدمه
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
//  isPresent()     لما محتاج if/else logic
//  orElse()        لما عندك قيمة افتراضية جاهزة
//  orElseGet()     زي orElse بس القيمة الافتراضية فيها computation
//  orElseThrow()   في REST APIs — ارمي exception → 404
//  ifPresent()     لما عايز تعمل action بس مش محتاج ترجع حاجة
//  map()           حوّل الـ value جوا الـ Optional
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// ✅ orElseGet — أحسن من orElse لو الـ default فيها logic
Product p = repo.findById(id)
    .orElseGet(() -> createDefaultProduct());  // lazy evaluation

// ✅ map — استخرج قيمة من جوا الـ Optional
Optional<String> name = repo.findById(id)
    .map(Product::getName);  // Optional<String> مش Product

String productName = repo.findById(id)
    .map(Product::getName)
    .orElse("Unknown Product");

// ❌ أوحش استخدام — متعملش كده ابداً!
Optional<Product> opt = repo.findById(id);
Product bad = opt.get();  // 💥 ممكن يطلع NoSuchElementException

// ✅ الصح
Product good = opt.orElseThrow(
    () -> new ProductNotFoundException("Not found"));

// ━━━ Flow كامل في REST API ━━━━━━━━━━━━━━━━━━━━━━━━
//
//  GET /api/products/5
//       ↓
//  ProductController.getById(5)
//       ↓
//  ProductService.getById(5)
//       ↓
//  repo.findById(5) → Optional<Product>
//       ↓
//  .orElseThrow(ProductNotFoundException)
//       ↓                    ↓
//  موجود → return Product   مش موجود → throw
//       ↓                    ↓
//  200 OK + body           GlobalExceptionHandler
//                               ↓
//                           404 Not Found + message

Copy failed — try from claude.ai in browser
```