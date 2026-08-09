## الإجابة المباشرة:

**مفيش حاجة إضافية — Spring نفسه هيعمل كل حاجة.**

---

## الـ DI هتمشي كده:

### **1. Command — بيحمل الـ data بس:**

```java
// CreateCourseCommand.java
public record CreateCourseCommand(
    String title,
    String description,
    BigDecimal price,
    UUID instructorId
) {}
```

### **2. Handler — فيه الـ logic:**

```java
// CreateCourseHandler.java
@Service
@RequiredArgsConstructor
public class CreateCourseHandler {

    private final CourseRepository courseRepository;
    private final CourseMapper mapper;

    public CourseResponse handle(CreateCourseCommand command) {
        Course course = Course.create(
            command.title(),
            command.description(),
            command.price()
        );
        courseRepository.save(course);
        return mapper.toResponse(course);
    }
}
```

### **3. Controller — بيحقن الـ Handlers مباشرة:**

```java
// CourseController.java
@RestController
@RequestMapping("/api/v1/courses")
@RequiredArgsConstructor
public class CourseController {

    // كل endpoint عنده handler خاص
    private final CreateCourseHandler createCourseHandler;
    private final UpdateCourseHandler updateCourseHandler;
    private final PublishCourseHandler publishCourseHandler;
    private final ArchiveCourseHandler archiveCourseHandler;
    private final GetCourseHandler getCourseHandler;
    private final SearchCoursesHandler searchCoursesHandler;

    @PostMapping
    public ResponseEntity<CourseResponse> create(
        @Valid @RequestBody CreateCourseRequest request
    ) {
        var command = new CreateCourseCommand(
            request.title(),
            request.description(),
            request.price(),
            request.instructorId()
        );
        return ResponseEntity.status(201)
            .body(createCourseHandler.handle(command));
    }

    @PutMapping("/{id}")
    public ResponseEntity<CourseResponse> update(
        @PathVariable UUID id,
        @Valid @RequestBody UpdateCourseRequest request
    ) {
        var command = new UpdateCourseCommand(id, request.title(), request.price());
        return ResponseEntity.ok(updateCourseHandler.handle(command));
    }

    @PatchMapping("/{id}/publish")
    public ResponseEntity<Void> publish(@PathVariable UUID id) {
        publishCourseHandler.handle(new PublishCourseCommand(id));
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/{id}")
    public ResponseEntity<CourseResponse> get(@PathVariable UUID id) {
        return ResponseEntity.ok(getCourseHandler.handle(new GetCourseQuery(id)));
    }
}
```

### **4. Repository Interface في Domain:**

```java
// CourseRepository.java — domain layer
public interface CourseRepository {
    Course save(Course course);
    Optional<Course> findById(UUID id);
    List<Course> findByInstructorId(UUID instructorId);
}
```

### **5. Implementation في Infrastructure: ** دا مش مهم 

```java
// CourseRepositoryImpl.java
@Repository
@RequiredArgsConstructor
public class CourseRepositoryImpl implements CourseRepository {

    private final CourseJpaRepository jpaRepository;
    private final CourseMapper mapper;

    @Override
    public Course save(Course course) {
        CourseEntity entity = mapper.toEntity(course);
        return mapper.toDomain(jpaRepository.save(entity));
    }

    @Override
    public Optional<Course> findById(UUID id) {
        return jpaRepository.findById(id)
            .map(mapper::toDomain);
    }
}
```

---

## الـ Flow كامل:

```
HTTP POST /api/v1/courses
         ↓
CourseController
         ↓
new CreateCourseCommand(data)
         ↓
CreateCourseHandler.handle(command)
         ↓
CourseRepository.save(course)     ← interface
         ↓
CourseRepositoryImpl.save(course) ← Spring بيحقنها تلقائي
         ↓
CourseJpaRepository.save(entity)  ← JPA
         ↓
PostgreSQL ✅
```

---

## الـ Spring بيعمل إيه تلقائياً:

```
@Service   → Spring بيعمل Bean من الـ Handler
@Repository → Spring بيعمل Bean من الـ Impl
@RequiredArgsConstructor → Lombok بتعمل Constructor
Spring بيشوف إن CourseRepository محتاجة →
بيلاقي CourseRepositoryImpl اللي بتimplementها →
بيحقنها تلقائي ✅
```

---

**الخلاصة:** مش محتاج أي library إضافية — بس `@Service` + `@Repository` + `@RequiredArgsConstructor` وSpring هيتكفل بالباقي 🚀