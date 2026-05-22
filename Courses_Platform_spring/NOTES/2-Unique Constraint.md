
```java
@Table(
    name = "enrollments",
    uniqueConstraints = @UniqueConstraint(
        columnNames = {"student_id", "course_id"}
    )
)
```


---

لو حاول طالب يشترك في نفس الكورس تاني مرة:

```
// في الداتابيز
student_id = 1, course_id = 5  ✅ أول مرة — اتحفظ
student_id = 1, course_id = 5  ❌ تاني مرة — Exception
```

---

الـ Exception اللي هتطلع:

```
DataIntegrityViolationException: 
duplicate key value violates unique constraint "enrollments_student_id_course_id_key"
```

---

وهتتعامل معاها في الـ Service بالشكل ده:

```java
// في EnrollmentService
public void enroll(Long studentId, Long courseId) {
    
    // تتحقق الأول قبل ما تحاول تحفظ
    boolean alreadyEnrolled = enrollmentRepository
        .existsByStudentIdAndCourseId(studentId, courseId);
    
    if (alreadyEnrolled) {
        throw new BadRequestException("Student already enrolled in this course");
    }
    
    // لو مش مشترك — اشترك
    Enrollment enrollment = Enrollment.builder()
        .student(student)
        .course(course)
        .build();
        
    enrollmentRepository.save(enrollment);
}
```

---

يعني عندك **حمايتين**:

|المستوى|الحماية|
|---|---|
|**Service**|بتتحقق قبل الحفظ وبترمي رسالة واضحة|
|**Database**|`uniqueConstraint` حماية أخيرة لو حاجة فاتت|

> الـ best practice دايماً تتحقق في الـ Service وتسيب الـ DB كخط دفاع تاني 🎯