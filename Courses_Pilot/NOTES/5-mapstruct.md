وده طبيعي جدًا. في الحقيقة **MapStruct مش بيعمل حاجة سحرية جديدة**، هو فقط بيكتب كود الـ Builder أو الـ Setters بدل منك وقت الـ Compile.

أنت كنت بتعمل كده:

```java
public CourseResponse toResponse(Course course) {
    return CourseResponse.builder()
            .id(course.getId())
            .title(course.getTitle())
            .description(course.getDescription())
            .price(course.getPrice())
            .level(course.getLevel())
            .status(course.getStatus())
            .build();
}
```

MapStruct بيخليك تكتب:

```java
@Mapper(componentModel = "spring")
public interface CourseMapper {

    CourseResponse toResponse(Course course);
}
```

ولما تعمل Build للمشروع، MapStruct يولد كلاس خلف الكواليس شبه ده:

```java
@Component
public class CourseMapperImpl implements CourseMapper {

    @Override
    public CourseResponse toResponse(Course course) {

        if (course == null) {
            return null;
        }

        return CourseResponse.builder()
                .id(course.getId())
                .title(course.getTitle())
                .description(course.getDescription())
                .price(course.getPrice())
                .level(course.getLevel())
                .status(course.getStatus())
                .build();
    }
}
```

يعني حرفيًا MapStruct كتب الكود اللي أنت كنت بتكتبه بإيدك.

---

## طيب ليه الناس بتستخدمه؟

تخيل عندك:

- User
    
- Course
    
- Section
    
- Lesson
    
- Enrollment
    
- Review
    
- Category
    

ولكل واحد:

```java
Entity -> Response
Request -> Entity
UpdateRequest -> Entity
```

هتلاقي نفسك كاتب 50 أو 100 Method تحويل.

MapStruct بيوفر الوقت ده.

---

## مثال على حاجة كان هيصعب عليك كتابتها

عندك:

```java
public class Course {
    private User instructor;
}
```

وعندك:

```java
public class CourseResponse {
    private UserResponse instructor;
}
```

لو يدويًا:

```java
return CourseResponse.builder()
        .title(course.getTitle())
        .instructor(
            userMapper.toResponse(course.getInstructor())
        )
        .build();
```

في MapStruct:

```java
@Mapper(
    componentModel = "spring",
    uses = UserMapper.class
)
public interface CourseMapper {

    CourseResponse toResponse(Course course);
}
```

هو هيعرف لوحده يستدعي:

```java
userMapper.toResponse(course.getInstructor())
```

---

## هل لازم تستخدمه؟

لا.

في مشاريع صغيرة أو أثناء التعلم:

```java
Builder
```

واضح جدًا وسهل الفهم.

في مشاريع كبيرة:

```java
MapStruct
```

يوفر وقت كثير ويقلل التكرار.

---

## نصيحتي ليك

بما أنك لسه بتتعلم Spring وبتبني Skill Hub:

1. افهم التحويل اليدوي بالـ Builder كويس.
    
2. بعد كده استخدم MapStruct.
    
3. كل ما تكتب Mapper اسأل نفسك:
    

"لو MapStruct مش موجود، أنا هكتب الكود ده إزاي؟"

لو عرفت تكتبه يدويًا، يبقى أنت فاهم MapStruct فعلًا، لأنه مجرد مولد للكود وليس تقنية جديدة مختلفة.