 ## `@PreAuthorize` في Spring Security

---
we can use i on class level or method level 
**بتعمل إيه؟**

بتتأكد إن الـ user عنده الصلاحية الصح **قبل ما الـ method تتنفذ أصلاً.**

---

**الفرق بينها وبين الـ Security Config:**

```java
// Security Config — بتأمن على مستوى الـ URL
.requestMatchers(HttpMethod.POST, "/api/v1/courses/**").hasRole("INSTRUCTOR")

// @PreAuthorize — بتأمن على مستوى الـ Method
@PreAuthorize("hasRole('INSTRUCTOR')")
public CourseResponse createCourse(...) { }
```

---

**إزاي تفعّلها؟**

لازم تضيف على الـ `SecurityConfig`:

```java
@EnableMethodSecurity
public class SecurityConfig { }
```

---

**أشكالها:**

```java
// role معين
@PreAuthorize("hasRole('INSTRUCTOR')")

// أكتر من role
@PreAuthorize("hasAnyRole('INSTRUCTOR', 'ADMIN')")

// الـ user اللي عمل login بس يعدل بياناته
@PreAuthorize("#id == authentication.principal.id")

// conditions مركبة
@PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
```

---

## تطبيقها على الـ CourseController بتاعك

بدل ما تعتمد على الـ Security Config بس:

```java
@RestController
@RequestMapping("/api/v1/courses")
@RequiredArgsConstructor
public class CourseController {

    @GetMapping
    public ResponseEntity<ApiResponse<List<CourseResponse>>> getAllCourses() {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.getAllCourses(), "Courses retrieved successfully")
        );
    }

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<CourseResponse>> getCourseById(@PathVariable Long id) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.getCourseById(id), "Course retrieved successfully")
        );
    }

    @GetMapping("/instructor/{instructorId}")
    public ResponseEntity<ApiResponse<List<CourseResponse>>> getCoursesByInstructorId(@PathVariable Long instructorId) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.getCoursesByInstructorId(instructorId), "Courses retrieved successfully")
        );
    }

    @GetMapping("/level")
    public ResponseEntity<ApiResponse<List<CourseResponse>>> getCoursesByLevel(@RequestParam CourseLevel level) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.getCoursesByLevel(level), "Courses retrieved successfully")
        );
    }

    @GetMapping("/status")
    @PreAuthorize("hasAnyRole('INSTRUCTOR', 'ADMIN')")  // ← Students مش المفروض يشوفوا كل الـ statuses
    public ResponseEntity<ApiResponse<List<CourseResponse>>> getCoursesByStatus(@RequestParam CourseStatus status) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.getCoursesByStatus(status), "Courses retrieved successfully")
        );
    }

    @PostMapping
    @PreAuthorize("hasRole('INSTRUCTOR')")
    public ResponseEntity<ApiResponse<CourseResponse>> createCourse(@RequestBody @Valid CourseRequest request) {
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(ApiResponse.success(courseService.createCourse(request), "Course created successfully"));
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasAnyRole('INSTRUCTOR', 'ADMIN')")
    public ResponseEntity<ApiResponse<CourseResponse>> updateCourse(
            @PathVariable Long id,
            @RequestBody @Valid UpdateCourseRequest request) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.updateCourse(id, request), "Course updated successfully")
        );
    }

    @PatchMapping("/{id}/publish")
    @PreAuthorize("hasAnyRole('INSTRUCTOR', 'ADMIN')")
    public ResponseEntity<ApiResponse<CourseResponse>> publishCourse(@PathVariable Long id) {
        return ResponseEntity.ok(
                ApiResponse.success(courseService.publishCourse(id), "Course published successfully")
        );
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasAnyRole('INSTRUCTOR', 'ADMIN')")
    public ResponseEntity<ApiResponse<Void>> deleteCourse(@PathVariable Long id) {
        courseService.deleteCourse(id);
        return ResponseEntity.ok(ApiResponse.success(null, "Course deleted successfully"));
    }
}
```

---

**اللي اتغير وليه:**

|الـ Endpoint|الصلاحية|السبب|
|---|---|---|
|`GET /courses`|الكل|Students يتصفحوا|
|`GET /courses/{id}`|الكل|يفتحوا كورس|
|`GET /courses/status`|INSTRUCTOR, ADMIN|Student مش محتاج يشوف DRAFT|
|`POST /courses`|INSTRUCTOR|بس الـ instructor ينشئ|
|`PUT /courses/{id}`|INSTRUCTOR, ADMIN|يعدلوا|
|`PATCH /courses/{id}/publish`|INSTRUCTOR, ADMIN|ينشروا|
|`DELETE /courses/{id}`|INSTRUCTOR, ADMIN|يمسحوا|

---

**وبالنسبة لـ `findAllPublished` في الـ Repository:**

```java
@Query("SELECT c FROM Course c WHERE c.status = :status")
List<Course> findAllPublished(@Param("status") CourseStatus status);
```

هتستخدمها في الـ `getAllCourses` في الـ Service عشان الـ Students يشوفوا PUBLISHED بس:

```java
@Transactional(readOnly = true)
@Override
public List<CourseResponse> getAllCourses() {
    return courseRepository.findAllPublished(CourseStatus.PUBLISHED)
            .stream()
            .map(courseMapper::toResponse)
            .toList();
}
```

يعني بدل ما ترجع كل الكورسات بما فيها الـ DRAFT، بترجع المنشور بس للـ Students.



```java 
prviate void validateOwnerShip(int courseid ,String email)
{
Course c = repo.findById(courseid).orElseThrow(()->excep);
User
}
```