يعني كدا ال service دي بحط فيها ال business logic بتاعي ...اولا هعمل interface وليكن LessonService واحط فيه الميثودز بتاعتي وبعدين LessonServiceImpl  هعمل امبلمنتيشن لميثودز الانترفيس دا  واحط فيه كل اللوجيك واستقبل ال ريكوست اللي اليوزر هيبعته واحوله ل entity وهكذا علي حسب الحاله صح ؟ 
طيب لي هعمل interface  = > LessonService مع العلم اني ممكن احط الميثودز علطول في ال class لاني لما اجي في ال controller هحقن ال interface وبالتالي ال controller مش هيعتمد عال  implementation اللي في ال service وهحقن ال interface ف كدا هيعتمد عال abstraction ومش هيشوف ال logic بتاعي وبالتالي هكون حققت مبدا ال DIP 


check the Section existent ,create lesson ,save it ,return its response

controller recieves the request from the client ,and validate the dto  , then business logic ,then repository then db call


@RestCOntroller
@RequestMappint("/api/course")
@RequiredArgsConstructor
public class CourseController
{
private final CourseService courseService;
@PostMapping("/create")
public ResponseEntity<Course
}
```
```   



spring.jpa.hibernate.ddl-auto=none/validate/update /create /create-drop


Alter table user
Add COLUMN username varchar(255);
هو hi



==🔵spring.jpa.hibernate.ddl-auto=validate;==
//hibernate will make sure that the  entities matches the schema 
==🔵spring.flyway.enabled=true;==
