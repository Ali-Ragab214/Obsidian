احنا عرفنا ان ال AutoBoxing اننا بنحول من : 
Primitive -> Wrapper
ال Generics in Java  وبعض ال Java Apis  لازمها refrence type  ف مش هينفع نستخدم معاها primitive     مثال علي كدا زي ال List   مثلا 

==ليه Autoboxing موجود أصلاً؟==

لأن بعض الـ Java APIs محتاجة **Objects** وليس primitives.

أشهر مثال:

```java
List<Integer> numbers = new ArrayList<>();

numbers.add(10);
numbers.add(20);
numbers.add(30);
```

`List` لا تستطيع أن تكون:

```java
List<int>
```

لأن Generics في Java بتتعامل مع reference types.

فـ Java تعمل:

```java
numbers.add(Integer.valueOf(10));
```

تلقائيًا.

وده أحد أهم استخدامات Autoboxing.

---