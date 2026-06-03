بص على الـ structure بتاعك، الفيتشر اللي جاي هي **Content Management** (Sections & Lessons).

---

## الترتيب المفصل

---

### 1. DTOs

**`SectionRequest`**

```
- title: String (NotBlank)
- courseId: Long (NotNull)
- orderIndex: Integer
```

**`LessonRequest`**

```
- title: String (NotBlank)
- content: String
- videoUrl: String
- sectionId: Long (NotNull)
- orderIndex: Integer
- durationMinutes: Integer
```

**`SectionResponse`**

```
- id, title, orderIndex
- List<LessonResponse> lessons
```

**`LessonResponse`**

```
- id, title, content, videoUrl
- orderIndex, durationMinutes
```

---

### 2. Mappers

**`SectionMapper`**

- `toEntity(SectionRequest)`
- `toResponse(Section)`

**`LessonMapper`**

- `toEntity(LessonRequest)`
- `toResponse(Lesson)`

---

### 3. Repositories

**`SectionRepository`**

```java
List<Section> findByCourseId(Long courseId);
List<Section> findByCourseIdOrderByOrderIndex(Long courseId);
boolean existsByCourseIdAndTitle(Long courseId, String title);
```

**`LessonRepository`**

```java
List<Lesson> findBySectionId(Long sectionId);
List<Lesson> findBySectionIdOrderByOrderIndex(Long sectionId);
```

---

### 4. ContentService / ContentServiceImpl

**Sections:**

```
- addSection(SectionRequest, instructorEmail)
  └── تتحقق إن الـ course بتاعته هو
- updateSection(id, SectionRequest, instructorEmail)
  └── تتحقق إن الـ section بتاعته هو
- deleteSection(id, instructorEmail)
- getSectionsByCourse(courseId)
```

**Lessons:**

```
- addLesson(LessonRequest, instructorEmail)
  └── تتحقق إن الـ section بتاعته هو
- updateLesson(id, LessonRequest, instructorEmail)
- deleteLesson(id, instructorEmail)
- getLessonsBySection(sectionId)
```

---

### 5. GlobalExceptionHandler

أضف handlers لـ:

```
- SectionNotFoundException
- LessonNotFoundException
- DuplicateSectionTitleException
```

---

### 6. CourseController — أضف عليه

```java
@GetMapping("/{id}/sections")
// يرجع sections الكورس مع الـ lessons
```

---

### 7. ContentController

```java
// Sections
POST   /api/sections              ← INSTRUCTOR فقط
PUT    /api/sections/{id}         ← INSTRUCTOR (صاحبه بس)
DELETE /api/sections/{id}         ← INSTRUCTOR أو ADMIN
GET    /api/sections/{id}/lessons ← Authenticated

// Lessons
POST   /api/lessons               ← INSTRUCTOR فقط
PUT    /api/lessons/{id}          ← INSTRUCTOR (صاحبه بس)
DELETE /api/lessons/{id}          ← INSTRUCTOR أو ADMIN
GET    /api/lessons/{id}          ← Authenticated
```

---

### 8. SecurityConfig

أضف الـ rules بتاعت الـ content endpoints.

---

## الترتيب التنفيذي

```
1. SectionRequest & LessonRequest
2. SectionResponse & LessonResponse
3. SectionMapper & LessonMapper
4. SectionRepository & LessonRepository
5. ContentService interface
6. ContentServiceImpl
7. Exception classes الجديدة
8. GlobalExceptionHandler updates
9. ContentController
10. SecurityConfig updates
```

ابدأ بالـ DTOs وقولي لما تخلص كل step. 🚀