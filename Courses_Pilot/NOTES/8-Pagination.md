أمام خيارين للـ pagination، اختار اللي تشوفه أنسب:

---

## **Option 1: Spring Data Pagination (الأفضل و الأسهل)**

### التغييرات المفروض تعملها:

**1️⃣ عدّل الـ Controller:**
```java
@GetMapping("/")
public ResponseEntity<ApiResponse<Page<CourseResponse>>> getAllCourses(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "DESC") Sort.Direction direction)
{
    Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sortBy));
    Page<CourseResponse> courses = courseService.getAllCourses(pageable);
    return ResponseEntity.ok(
            ApiResponse.success(courses, "Courses retrieved successfully")
    );
}
```

**2️⃣ عدّل الـ Service:**
```java
public Page<CourseResponse> getAllCourses(Pageable pageable) {
    return courseRepository.findAll(pageable)
            .map(courseMapper::toResponse);
}
```

**3️⃣ Repository بتاعك يبقى متوارث من `PagingAndSortingRepository`:**
```java
public interface CourseRepository extends JpaRepository<Course, Long> {
    // الـ pagination و sorting بتشتغل automatically
}
```

### **الفايدة:**
- الـ response بيتضمن معلومات الـ pagination تلقائي:
```json
{
  "success": true,
  "data": {
    "content": [...],
    "totalElements": 50,
    "totalPages": 5,
    "currentPage": 0,
    "pageSize": 10,
    "isFirst": true,
    "isLast": false
  }
}
```

---

## **Option 2: Custom Pagination (لو عايز تحكم أكتر)**

**1️⃣ اعمل Custom Response DTO:**
```java
@Data
@Builder
public class PaginatedResponse<T> {
    private List<T> content;
    private int currentPage;
    private int pageSize;
    private long totalElements;
    private int totalPages;
}
```

**2️⃣ عدّل الـ Service:**
```java
public PaginatedResponse<CourseResponse> getAllCourses(int page, int size) {
    Pageable pageable = PageRequest.of(page, size);
    Page<Course> courses = courseRepository.findAll(pageable);
    
    return PaginatedResponse.<CourseResponse>builder()
            .content(courses.stream()
                    .map(courseMapper::toResponse)
                    .toList())
            .currentPage(page)
            .pageSize(size)
            .totalElements(courses.getTotalElements())
            .totalPages(courses.getTotalPages())
            .build();
}
```

---

## **التوصية الأفضل:**
اختار **Option 1** لأن:
✅ Spring بتتعامل مع كل حاجة  
✅ أسهل في الصيانة  
✅ بتوفر معلومات pagination كاملة  
✅ الـ sorting سهل جداً  

**التغييرات الأساسية:**
- Add `page` و `size` parameters في Controller
- استخدم `PageRequest.of()` في Service
- Return `Page<T>` بدل `List<T>`

تبدأ بـ Option 1؟ 🎯