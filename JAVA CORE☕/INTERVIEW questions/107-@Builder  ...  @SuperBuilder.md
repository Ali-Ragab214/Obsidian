## الخلاصة: Lombok @Builder vs @SuperBuilder

### الخلاصه ان @SuperBUilder هنستخدمها في الحالات اللي فيها inheritence  ولازم تبقي عال parent and child عشان لو فيه fields in the parent ال child يعرف ياكسسيس عليها وهو بيعمل object using builder pattern 

### 1. `@Builder`


- بتستخدم لتسهيل إنشاء Objects باستخدام Builder Pattern بدل الـ Constructors الطويلة.
    
- بتخليك تحدد الـ fields اللي محتاجها فقط، من غير الالتزام بترتيب معين.
    
- مناسبة للاستخدام المستقل، يعني Class مش محتاجة Builder يدعم الـ Inheritance.
    

```
@Builder
public class User {
    private String username;
    private String email;
}
```

الاستخدام:

```
User user = User.builder()
        .username("Ali")
        .email("ali@gmail.com")
        .build();
```

### 2. `@SuperBuilder`

- بتعمل نفس وظيفة `@Builder`، لكن بتدعم Inheritance.
    
- بتسمح للـ Builder الخاص بالـ Child إنه يتعامل مع Fields الموجودة في الـ Parent والـ Child مع بعض.
    
- لازم تحط `@SuperBuilder` على الـ Parent والـ Child وكل Classes في الـ Inheritance Hierarchy.
    
- مينفعش تحط `@SuperBuilder` على الـ BaseEntity وتحط `@Builder` على الـ Child؛ لازم الاتنين يستخدموا `@SuperBuilder`.
    

```
@Getter
@SuperBuilder
public class BaseEntity {
    private Long id;
}
```

```
@Getter
@SuperBuilder
public class User extends BaseEntity {
    private String username;
}
```

الاستخدام:

```
User user = User.builder()
        .id(1L)           // Parent field
        .username("Ali")  // Child field
        .build();
```

### 3. `@Builder.Default`

- بتستخدم مع `@Builder` أو `@SuperBuilder` للحفاظ على الـ Default Value المحددة للـ Field عند إنشاء Object باستخدام Builder.

```
@Builder.Default
private boolean active = true;
```

لو معملتش `.active()` في الـ Builder، هتفضل القيمة `true` بدل ما تاخد Java default value وهي `false`.

احفظ الفرق في 3 جمل:

- `@Builder`: لإنشاء Objects بسهولة لما مش محتاج Inheritance في الـ Builder.
    
- `@SuperBuilder`: لإنشاء Objects بسهولة مع دعم Inheritance، ولازم تتحط على الـ Parent والـ Child.
    
- `@Builder.Default`: للحفاظ على الـ Default Values أثناء استخدام Builder.
- 