
تمام — هديك مثال **Spring Boot Application كامل وبسيط** كأنه مشروع production صغير (CRUD API)، بالـ structure الصح + كل جزء واضح 👇

هنا هنعمل:  
👉 **Product Management API** (نضيف/نعدل/نحذف/نجيب منتجات)

---

# ✅ 1. الــ Structure (زي الشركات بالظبط)

```
product-app
│── src/main/java/com/example/productapp
│   │── ProductAppApplication.java
│   │
│   ├── controller
│   │     └── ProductController.java
│   │
│   ├── service
│   │     ├── ProductService.java
│   │     └── ProductServiceImpl.java
│   │
│   ├── repository
│   │     └── ProductRepository.java
│   │
│   ├── entity
│   │     └── Product.java
│   │
│   ├── dto
│   │     └── ProductDTO.java
│   │
│   └── exception
│         └── ResourceNotFoundException.java
│
│── src/main/resources
│   └── application.properties
```

---

# ✅ 2. ال Dependencies (Spring Initializr)

اختار:

- Spring Web
    
- Spring Data JPA
    
- H2 Database (أو MySQL)
    
- Lombok
    

---

# ✅ 3. الكود

## 🟢 Main Class

```java
@SpringBootApplication
public class ProductAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductAppApplication.class, args);
    }
}
```

---

## 🟢 Entity (DB Table)

```java
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private double price;
}
```

---

## 🟢 DTO (عشان production best practice)

```java
@Data
public class ProductDTO {
    private String name;
    private double price;
}
```

---

## 🟢 Repository

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

---

## 🟢 Service Interface

```java
public interface ProductService {
    Product create(ProductDTO dto);
    List<Product> getAll();
    Product getById(Long id);
    Product update(Long id, ProductDTO dto);
    void delete(Long id);
}
```

---

## 🟢 Service Implementation

```java
@Service
@RequiredArgsConstructor
public class ProductServiceImpl implements ProductService {

    private final ProductRepository repo;

    @Override
    public Product create(ProductDTO dto) {
        Product p = new Product(null, dto.getName(), dto.getPrice());
        return repo.save(p);
    }

    @Override
    public List<Product> getAll() {
        return repo.findAll();
    }

    @Override
    public Product getById(Long id) {
        return repo.findById(id)
                .orElseThrow(() -> new RuntimeException("Product not found"));
    }

    @Override
    public Product update(Long id, ProductDTO dto) {
        Product p = getById(id);
        p.setName(dto.getName());
        p.setPrice(dto.getPrice());
        return repo.save(p);
    }

    @Override
    public void delete(Long id) {
        repo.deleteById(id);
    }
}
```

---

## 🟢 Controller (API Layer)

```java
@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService service;

    @PostMapping
    public Product create(@RequestBody ProductDTO dto) {
        return service.create(dto);
    }

    @GetMapping
    public List<Product> getAll() {
        return service.getAll();
    }

    @GetMapping("/{id}")
    public Product getById(@PathVariable Long id) {
        return service.getById(id);
    }

    @PutMapping("/{id}")
    public Product update(@PathVariable Long id, @RequestBody ProductDTO dto) {
        return service.update(id, dto);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}
```

---

## 🟢 Exception (Production Best Practice)

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String msg) {
        super(msg);
    }
}
```

---

## 🟢 application.properties

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update
```

---

# ✅ 4. تشغل المشروع ازاي

1. Run المشروع
    
2. افتح:
    

```
http://localhost:8080/api/products
```

---

# ✅ 5. تجرب بـ Postman

### ➕ Create

POST `/api/products`

```json
{
  "name": "Laptop",
  "price": 15000
}
```

### 📥 Get All

GET `/api/products`

### 📥 Get By ID

GET `/api/products/1`

### ✏️ Update

PUT `/api/products/1`

### ❌ Delete

DELETE `/api/products/1`

---

# ✅ 6. ليه ده Production-like؟

✔ فصل Layers (Controller / Service / Repo)  
✔ DTO بدل ما ترجع Entity  
✔ Exception Handling  
✔ Dependency Injection  
✔ Clean Architecture

---

# 💡 لو عايز تزوده production بجد

قولّي وأنا أزودلك:

- Security (JWT)
    
- Validation
    
- Global Exception Handler
    
- Logging
    
- Swagger
    
- Docker
    

---

لو عايز كمان:  
👉 أحولهولك **Microservice**  
👉 أو أربطه بـ Angular (اللي انت شغال بيه)

قولّي 👍