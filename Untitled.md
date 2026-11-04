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




```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
@Table(name="products")
public class Product {

@Column(nullable= false,unique = true)
private String name;
@Size(min=0,max=1000)
private String description;
@DecimalMin(value="0.0")
private BigDecimal price;
@NotNull
@Min(0)
private Integer stockQuantity;

@Enumerated(EnumType.STRING)
@Column(nullable=false)
@Builder.Default
private ProductStatus productStatus = ProductStatus.DRAFT;

@Builder.Default
private Double averageRating = 0.0;

@Formula("SELECT COUNT(*) FROM products p WHERE p.id =id")
private Integer totalOrders;

@CreatedDate
private LocalDateTime createdAt;

@LastModifiedDate
private LocalDateTime updatedAt;

@ManyToOne()
@JoinColumn(name="category_id")
private Category category;

@ManyToOne()
@JoinColumn(name="brand_id")
private Brand brand;

@OneToMany(mappedBy="product",cascade=CascadeType.ALL)
@Builder.Default
private List<ProductImage> productimages = new ArrayList<>();

//reviews is like productimamges in relation 
@OneToMany(mappedBy="product",cascade=CascadeType.ALL)
@Builder.Default//to prevent nullpointerexceptions i used this 
private List<OrderItems> orderItems = new ArrayList<>();



}
```


```java 
public interface LessonRepository extends JpaRepository<Lesson,Long>
{
}
```

بنستخدم Optional<> لما تكون النتيجه حاجه واحده هترجع او مفيش حاجه هترجع عشان نمنع nullpointerexception 
لو لقت اكتر من نتيجه هتديك exception 
@EntityGraph(attributePaths={"course","student"},type=EntityGraph.EntityGraphType.FETCH);


```java

```

there are one new version each 6 months and each one has new features  بس مش كلها بتتدعم لفتره طويله 
Long Time Support (LTS)  like : 8,11,17,21,25



```java
//java is just a programming language it cant deal with requests ,so dispatcherservlet 
@GetMapping("get/courses/{id}")
to control HttpStatus code 
public class ApiResponse<T> 
{
private boolean success;
private String message;
private T data;
private LocalDateTime time;

}
we use Response entity class to mnagae full http response :header,body ,http status code 


```

tenant =>client or organization use my saas 
single tennant=>each client has its server and db so we have a large customization for each tennant 
has cons like :
hard to scale and maintain , more cost  

3 approaches : each tenant has its app and db ,each tenant has a different schema ,shared schema and db for all tenants 

create table workouts(
id BIGSERIAL primary key ,
name varchar(30),
tenant_id BIGINT NOT NULL,
CONSTRAINT fk_workout_tennant
FOREIGN KEY (tenant_id)
refrences tenants(id)
)
one database has more than one schema and each schema has a set of tables 


```mysql 
CREATE DATABASE gym;
CREATE SCHEMA gyma;
CREATE SCHEMA gymb;
CREATE SCHEMA gymc;

CREATE TABLE gyma.users(
id BIGSERIAL PRIMARY KEY ,
name VARCHAR(30)
)

CREATE TABLE gymb.users(
id BIGSERIAL PRIMARY KEY ,
name VARCHAR(30)
)

CREATE TABLE gymb.users(
id BIGSERIAL PRIMARY KEY ,
name VARCHAR(30)
)

SELECT * FROM gymb.users;
-- if the schema has a tables i cant delete it 
-- DROP SCHEMA gymb ; 
```



build one app but serve multiple clients

# ==🔴shared db== and same schema has disadvantages like : 
- data leak probability from one tenant to another  so we must achieve data isolation
- one large tenant can slow down others  
- limited customization  

