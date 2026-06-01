تمام 🔥 ده أهم سؤال يخليك تكتب REST API بشكل احترافي فعلًا.  
هديك **كل الحالات (cases)** لاستخدام `ResponseEntity` + أمثلة واضحة جدًا 👇

---
Custom Exceptions → 8 → Pagination → 8.5 → Logging → 9 → Unit Tests → 9.5




# 💡 الأول: فاكر القاعدة

```text
ResponseEntity = Status Code + Headers + Body
```

---

# 🟢 1. Success (GET)

## ✅ الحالة: جلب بيانات

```java
@GetMapping("/{id}")
public ResponseEntity<AuthorDTO> getAuthor(@PathVariable Integer id) {
    AuthorDTO author = authorService.getAuthorById(id);
    return ResponseEntity.ok(author);
}
```

📌 بيرجع:

```text
200 OK
+ body
```

---

# 🟢 2. Create (POST)

## ✅ الحالة: إنشاء resource

```java
@PostMapping
public ResponseEntity<AuthorDTO> create(@RequestBody AuthorRequest request) {
    AuthorDTO created = authorService.createAuthor(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

📌 بيرجع:

```text
201 CREATED
+ body
```

---

# 🟢 3. Update (PUT)

## ✅ الحالة: تعديل بيانات

```java
@PutMapping("/{id}")
public ResponseEntity<AuthorDTO> update(
        @PathVariable Integer id,
        @RequestBody AuthorRequest request) {

    AuthorDTO updated = authorService.updateAuthor(id, request);
    return ResponseEntity.ok(updated);
}
```

📌:

```text
200 OK
```

---

# 🟢 4. Delete

## ✅ الحالة: حذف

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(@PathVariable Integer id) {
    authorService.deleteAuthor(id);
    return ResponseEntity.noContent().build();
}
```

📌:

```text
204 NO CONTENT
```

---

# 🔴 5. Not Found

## ✅ الحالة: البيانات مش موجودة

```java
@GetMapping("/{id}")
public ResponseEntity<?> get(@PathVariable Integer id) {
    try {
        return ResponseEntity.ok(authorService.getAuthorById(id));
    } catch (RuntimeException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body("Author not found");
    }
}
```

📌:

```text
404 NOT FOUND
```

---

# 🟡 6. Validation Error

## ✅ الحالة: input غلط

```java
@PostMapping
public ResponseEntity<?> create(@Valid @RequestBody AuthorRequest request) {
    AuthorDTO created = authorService.createAuthor(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

📌 لو validation فشل:

```text
400 BAD REQUEST
```

(غالبًا بييجي من Spring تلقائي)

---

# 🔴 7. Conflict (مثلاً email موجود)

```java
@PostMapping
public ResponseEntity<?> create(@RequestBody AuthorRequest request) {
    if (repository.existsByEmail(request.getEmail())) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body("Email already exists");
    }

    AuthorDTO created = authorService.createAuthor(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

📌:

```text
409 CONFLICT يعني حاجه موجوده 
```

---

# 🟣 8. Unauthorized / Forbidden

## ✅ الحالة: مش مسموح

```java
return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
```

📌:

```text
401 UNAUTHORIZED
```

---

# 🟣 9. Custom Headers

## ✅ الحالة: عايز تضيف header

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("X-Custom-Header", "value")
        .body(created);
```

---

# 🟣 10. Empty Response

## ✅ الحالة: مفيش محتوى

```java
return ResponseEntity.noContent().build();
```

📌:

```text
204 NO CONTENT
```

---

# 🟣 11. Response بدون Body

```java
return ResponseEntity.ok().build();
```

📌:

```text
200 OK (no body)
```

---

# 🟣 12. Generic Response (مرن)

```java
public ResponseEntity<?> getSomething() {
    return ResponseEntity.ok(data);
}
```

👉 تستخدمه لما نوع الـ response متغير

---

# 🔥 ملخص كل الحالات المهمة

|الحالة|الكود|الاستخدام|
|---|---|---|
|Success GET|200|ok()|
|Create|201|status(CREATED)|
|Delete|204|noContent()|
|Not Found|404|status(NOT_FOUND)|
|Bad Request|400|validation|
|Conflict|409|duplicate data|
|Unauthorized|401|auth|
|Forbidden|403|permission|

---

# 💎 Template جاهز (احترافي)

```java
@PostMapping
public ResponseEntity<?> create(@RequestBody AuthorRequest request) {
    try {
        AuthorDTO created = authorService.createAuthor(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    } catch (RuntimeException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body(e.getMessage());
    }
}
```

---

# 🧠 أهم نصيحة (Level أعلى 🔥)

في المشاريع الكبيرة:

❌ متكتبش try/catch في كل Controller  
✅ استخدم:

```text
ؤ
```

---

# 🚀 الخلاصة

> `ResponseEntity` مش بس status  
> ده التحكم الكامل في الـ API response

---

لو عايز نعمل خطوة تقيلة 🔥  
أبني معاك **Global Exception Handler + Error Response موحد زي الشركات** قولي 👍