```java 
@MappedSuperClass
@EntityListners(AuditingEntityListner.class)
@Getter
@Setter

public class BaseEntity
{
@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)
private Long id ;
@CreatedDate 
private LocalDateTime createdAt;

@LastModifiedDate
privte LocalDateTime updatedAt;

}
```

@MappedSuperClass بتعلم ان الكلاس دا مش هيبقا entity لكن الكلاسات اللي هتورث منه  هتعمل ماب للدااتا االلي فيه وبستخدمه لما يكون عندي داتا كتير شيرد بين كلاسات كتير ف بدل ما اكررها في كل كلاس بحطها في كلاس واحد بس 
يعني jpa مش هيعمل mapping للكلاس المسبوق ب دي في  الدات بيز  MappedSuperClass

وبردوا ال private fields  اللي بتكون في الكلاس دا هعرف اعمل اكسيس عليها في الابن عن طريق ال getter and setter 

```java 
Course course = Course.Builder()
.tittle("ddd")
.description("")
.build();

	@Table(name="courses",uniqueConstraints = @UniqueConstraint(columnNames="tittle",names="uk_course_tittle")
	)
	

```