

---

## ما هو؟

الـ `Authentication` هو **interface** في Spring Security بيمثل الـ **currently logged-in user** في أي request.

لما user بيبعت request وعنده valid JWT token، Spring Security بيعمل **Authentication object** ويحطه في حاجة اسمها **SecurityContext** — وانت بتقدر تجيبه في أي وقت.

---

## من فين بييجي؟

```
Client Request
     │
     │  Authorization: Bearer eyJhbGci...
     ↓
┌─────────────────────────┐
│   JWT Filter             │  ← بيفك الـ token ويجيب بيانات الـ user
└─────────────────────────┘
     │
     ↓
┌─────────────────────────┐
│   SecurityContext        │  ← بيحفظ الـ Authentication object
└─────────────────────────┘
     │
     ↓
┌─────────────────────────┐
│   Controller Method      │  ← Spring بيحقنه تلقائياً
│   Authentication auth    │
└─────────────────────────┘
```

Spring بيشوف إن الـ parameter نوعه `Authentication`، فبيجيبه تلقائياً من الـ `SecurityContext` ويحطه في الـ method — **انت مش محتاج تعمل حاجة**.

---

## جوّاه إيه بالظبط؟

```java
Authentication authentication

authentication.getName()           // email أو username بتاع الـ user
authentication.getAuthorities()    // الـ roles زي [ROLE_ADMIN, ROLE_INSTRUCTOR]
authentication.getPrincipal()      // الـ UserDetails object كامل
authentication.getCredentials()    // الـ password (بيبقى null بعد الـ authentication)
authentication.isAuthenticated()   // هل الـ user authenticated؟
authentication.getDetails()        // معلومات زيادة زي الـ IP address
```


```java
Authentication auth
auth.getName()=>will bring user email or name 
auth.getPrincipal();full object from user details 
auth.getAuthorities(); bring roles 
```
---

## مثال عملي مفصل

```java
@DeleteMapping("/{id}")
@PreAuthorize("hasRole('INSTRUCTOR') or hasRole('ADMIN')")
public ResponseEntity<ApiResponse<Void>> deleteCourse(
        @PathVariable Long id,
        Authentication authentication) {

    // 1️⃣ جيب email الـ user اللي بعت الـ request
    String currentUserEmail = authentication.getName();

    // 2️⃣ جيب الـ roles بتاعته
    Collection<? extends GrantedAuthority> roles = authentication.getAuthorities();

    // 3️⃣ جيب الـ UserDetails كامل
    UserDetails userDetails = (UserDetails) authentication.getPrincipal();

    // 4️⃣ بعّت للـ service عشان يتحقق إن الـ instructor ده هو صاحب الـ course
    courseService.deleteCourse(id, currentUserEmail);

    return ResponseEntity.ok(ApiResponse.success(null, "Course deleted successfully"));
}
```

---

## ليه بنمرره كـ Parameter؟

عندك **3 طرق** تجيب الـ Authentication، وفيه فرق مهم بينهم:

---

### ❌ الطريقة القديمة — SecurityContextHolder

```java
public void deleteCourse(Long id) {
    // بتجيبه يدوياً من الـ SecurityContext
    Authentication auth = SecurityContextHolder
                            .getContext()
                            .getAuthentication();

    String email = auth.getName();
}
```

**المشكلة:**

- الـ Service بقت مرتبطة بـ Spring Security
- صعبة في الـ Unit Testing
- مش clean

---

### ✅ الطريقة الصح — Parameter في الـ Controller

```java
public ResponseEntity<?> deleteCourse(
        @PathVariable Long id,
        Authentication authentication) {   // Spring بيحقنه تلقائياً

    courseService.deleteCourse(id, authentication.getName());
}
```

**المميزات:**

- الـ Controller هو اللي بيتعامل مع الـ Security
- الـ Service بتاخد String بس — مش مرتبطة بـ Spring Security
- سهلة في الـ Testing

---

### ✅ الطريقة الأنظف — `@AuthenticationPrincipal`

```java
public ResponseEntity<?> deleteCourse(
        @PathVariable Long id,
        @AuthenticationPrincipal UserDetails userDetails) {

    courseService.deleteCourse(id, userDetails.getUsername());
}
```

**الفرق:** بدل ما تجيب الـ `Authentication` كله وتعمل `.getPrincipal()`، بتجيب الـ `UserDetails` مباشرة.

---

## متى تستخدم كل طريقة؟

```
┌─────────────────────────────┬────────────────────────────────────┐
│ الحالة                       │ الطريقة                            │
├─────────────────────────────┼────────────────────────────────────┤
│ محتاج email أو username بس  │ Authentication → getName()         │
│ محتاج UserDetails كامل      │ @AuthenticationPrincipal           │
│ محتاج الـ roles              │ Authentication → getAuthorities()  │
│ محتاج تتحقق من الـ token    │ Authentication → getCredentials()  │
└─────────────────────────────┴────────────────────────────────────┘
```

---

## في الـ Service — الصح والغلط

```java
// ✅ الصح — الـ Service مش شايفة Spring Security خالص
public void deleteCourse(Long id, String instructorEmail) {
    Course course = courseRepository.findById(id)
        .orElseThrow(() -> new CourseNotFoundException(id));

    if (!course.getInstructor().getEmail().equals(instructorEmail)) {
        throw new UnauthorizedException("Not your course!");
    }

    courseRepository.delete(course);
}

// ❌ الغلط — الـ Service مرتبطة بـ Spring Security
public void deleteCourse(Long id) {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    String email = auth.getName();
    // الـ service دلوقتي مش قابلة للـ testing بسهولة
}
```

---

## Testing الفرق

```java
// ✅ سهل تعمل test للـ Service
@Test
void deleteCourse_shouldThrow_whenNotOwner() {
    courseService.deleteCourse(1L, "notowner@email.com");
    // مفيش حاجة تعمل mock لها
}

// ❌ صعب تعمل test لو الـ Service بتستخدم SecurityContextHolder
@Test
void deleteCourse_shouldThrow_whenNotOwner() {
    // محتاج تعمل mock للـ SecurityContextHolder كله
    SecurityContext securityContext = mock(SecurityContext.class);
    SecurityContextHolder.setContext(securityContext);
    // كود كتير ومعقد لمجرد تجيب الـ email
}
```

---

## الصورة الكاملة

```
JWT Token في الـ Request
         │
         ↓
    JWT Filter
    ├── يفك الـ token
    ├── يجيب الـ email والـ roles
    └── يعمل Authentication object
         │
         ↓
    SecurityContext
    └── يحفظ الـ Authentication
         │
         ↓
    Controller Method
    └── Authentication parameter ← Spring يحقنه تلقائياً
         │
         ↓
    authentication.getName()     → "ali@email.com"
    authentication.getAuthorities() → [ROLE_INSTRUCTOR]
    authentication.getPrincipal()   → UserDetails object
```

---

## ملخص سريع

|Method|بترجع إيه|
|---|---|
|`getName()`|الـ username أو email|
|`getAuthorities()`|الـ roles|
|`getPrincipal()`|الـ UserDetails كامل|
|`isAuthenticated()`|true/false|
|`getCredentials()`|الـ password (غالباً null)|

> **القاعدة الذهبية:** الـ `Authentication` بتتعامل معاه في الـ **Controller فقط** — الـ Service بتاخد الـ data الـ primitive زي String أو Long بس.
