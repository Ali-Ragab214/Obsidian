
## What is Method Reference?

الـ Method Reference هي طريقة مختصرة لكتابة Lambda Expression عندما تكون الـ Lambda تستدعي Method موجودة بالفعل بدون إضافة أي Logic جديد.

مثال:

بدلاً من كتابة:

```java
list.forEach(name -> System.out.println(name));
```

يمكن كتابة:

```java
list.forEach(System.out::println);
```

النتيجة واحدة تمامًا.

---

# لماذا تم إضافتها؟

لأن بعض الـ Lambda Expressions تكون مجرد استدعاء مباشر لـ Method موجودة مسبقًا:

```java
x -> someMethod(x)
```

فبدلاً من كتابة Lambda كاملة يمكن استخدام Method Reference.

---

# Syntax

```java
ClassName::methodName
```

أو:

```java
object::methodName
```

أو:

```java
ClassName::new
```

---

# أنواع Method Reference

يوجد 4 أنواع رئيسية.

---

# 1) Reference to a Static Method

## Lambda

```java
Function<String, Integer> parser =
        s -> Integer.parseInt(s);
```

## Method Reference

```java
Function<String, Integer> parser =
        Integer::parseInt;
```

استخدام:

```java
Integer number = parser.apply("100");

System.out.println(number);
```

Output:

```text
100
```

---

## ماذا يحدث داخليًا؟

Java تحول:

```java
Integer::parseInt
```

إلى:

```java
s -> Integer.parseInt(s)
```

---

# 2) Reference to an Instance Method of a Particular Object

لدينا:

```java
public class Printer {

    public void print(String message) {
        System.out.println(message);
    }

}
```

ثم:

```java
Printer printer = new Printer();
```

## Lambda

```java
Consumer<String> consumer =
        message -> printer.print(message);
```

## Method Reference

```java
Consumer<String> consumer =
        printer::print;
```

استخدام:

```java
consumer.accept("Hello");
```

Output:

```text
Hello
```

---

# 3) Reference to an Instance Method of an Arbitrary Object of a Particular Type

هذا أكثر نوع يربك الناس في المقابلات.

مثال:

```java
List<String> names =
        List.of("Ali", "Mohamed", "Ahmed");
```

## Lambda

```java
names.sort((a, b) -> a.compareToIgnoreCase(b));
```

## Method Reference

```java
names.sort(String::compareToIgnoreCase);
```

---

## ماذا يحدث داخليًا؟

Java تحول:

```java
String::compareToIgnoreCase
```

إلى:

```java
(a, b) -> a.compareToIgnoreCase(b)
```

أول Parameter يصبح الـ Object.

وباقي الـ Parameters يتم تمريرهم للـ Method.

---

مثال آخر:

```java
BiPredicate<String, String> predicate =
        String::startsWith;
```

تعادل:

```java
BiPredicate<String, String> predicate =
        (a, b) -> a.startsWith(b);
```

---

# 4) Constructor Reference

بدلاً من:

```java
Supplier<User> supplier =
        () -> new User();
```

يمكن كتابة:

```java
Supplier<User> supplier =
        User::new;
```

---

مثال:

```java
User user = supplier.get();
```

---

مع Constructor يحتوي Parameters:

```java
public class User {

    private String name;

    public User(String name) {
        this.name = name;
    }

}
```

Lambda:

```java
Function<String, User> creator =
        name -> new User(name);
```

Method Reference:

```java
Function<String, User> creator =
        User::new;
```

---

# When Can We Use Method Reference?

يمكن استخدام Method Reference فقط إذا كانت الـ Lambda:

- تستدعي Method واحدة فقط.
    
- لا تحتوي على أي Logic إضافي.
    
- توقيع الـ Method متوافق مع Functional Interface.
    

---

مسموح:

```java
names.forEach(name -> System.out.println(name));
```

↓

```java
names.forEach(System.out::println);
```

---

غير مسموح:

```java
names.forEach(name -> {
    System.out.println(name);
    saveToDatabase(name);
});
```

لا يمكن تحويلها إلى Method Reference لأنها تحتوي على أكثر من عملية.

---

# Relation with Functional Interfaces

الـ Method Reference لا تعمل إلا مع Functional Interfaces.

مثل:

- `Consumer`
    
- `Supplier`
    
- `Function`
    
- `Predicate`
    
- `Runnable`
    
- `Comparator`
    

مثال:

```java
Function<String, Integer> parser =
        Integer::parseInt;
```

هنا:

```java
apply(String)
```

تتوافق مع:

```java
parseInt(String)
```

---

# Method Reference vs Lambda

|Feature|Lambda|Method Reference|
|---|---|---|
|More Flexible|Yes|No|
|Shorter Syntax|No|Yes|
|Supports Extra Logic|Yes|No|
|Readability|Good|Better when simple|

---

# Examples Used Frequently in Real Projects

## Sorting

```java
users.sort(User::compareTo);
```

---

## Mapping

```java
List<String> names =
        users.stream()
             .map(User::getName)
             .toList();
```

---

## Printing

```java
users.forEach(System.out::println);
```

---

## Filtering

```java
users.stream()
     .filter(User::isActive)
     .toList();
```

تعادل:

```java
users.stream()
     .filter(user -> user.isActive())
     .toList();
```

---

## Constructor Mapping

```java
List<UserDto> dtos =
        users.stream()
             .map(UserDto::new)
             .toList();
```

---

# Interview Answer

## What is Method Reference in Java 8?

Method Reference is a shorthand syntax for lambda expressions that only call an existing method.

It improves readability and reduces boilerplate code.

---

## What are the four types of Method References?

1. Reference to a static method
    

```java
Integer::parseInt
```

2. Reference to an instance method of a particular object
    

```java
printer::print
```

3. Reference to an instance method of an arbitrary object of a particular type
    

```java
String::compareToIgnoreCase
```

4. Constructor reference
    

```java
User::new
```

---

## What is the difference between Lambda and Method Reference?

A Method Reference is simply a shorter form of a lambda expression when the lambda only delegates to an existing method without adding additional behavior.