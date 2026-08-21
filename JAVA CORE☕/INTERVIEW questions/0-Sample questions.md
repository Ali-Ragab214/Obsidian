

- [ ] # ☕ Java Core

- [x]  **1.** What is the difference between `==` and `equals()` when used with Integer values between -128 and 127?
    
- [ ]  **2.** What happens if you don't override `hashCode()` but you override `equals()`?
    
- [x]  **3.** Can an abstract class have a constructor? When would it be called?
    
- [ ]  **4.** What is the difference between `Comparable` and `Comparator`? When do you use each?
    
- [x]  **5.** Can you instantiate an interface in Java? What about anonymous classes?
    
- [x]  **6.** What is the difference between `throw` and `throws`?
    
- [x]  **7.** What happens when a static method and an instance method have the same name in the same class?
    
- [x]  **8.** Can a constructor be private? What design pattern does this enable?
    
- [ ]  **9.** What is the difference between `instanceof` and `getClass()` in a type check?
    
- [x]  **10.** What happens if an exception is thrown inside a `finally` block?
    
- [ ]  **11.** Can you override a static method? Can you hide it?
    
- [ ]  **12.** What is the difference between shallow copy and deep copy? How do you achieve each in Java?
    
- [ ]  **13.** What happens when you call `new String("hello")` vs `"hello"` — how many objects are created?
    
- [ ]  **14.** What is autoboxing? What are the potential performance or bug issues with it?
    
- [ ]  **15.** What is the difference between `break`, `continue`, and `return` inside a loop?
    

- [ ] ---

- [ ] # 🧬 OOP & Design

- [ ]  **16.** What is the difference between Composition and Aggregation?
    
- [ ]  **17.** Can a final class be extended? Can a final method be overloaded?
    
- [ ]  **18.** What is method hiding vs method overriding — what's the key difference?
    
- [ ]  **19.** If a parent class constructor throws a checked exception, what must the child class constructor do?
    
- [ ]  **20.** What is the difference between an abstract class and an interface — when would you choose one over the other?
    
- [ ]  **21.** Can a class be both abstract and final? Why or why not?
    
- [ ]  **22.** What is Liskov Substitution Principle and how can you violate it accidentally?
    
- [ ]  **23.** If a subclass overrides a method and the parent reference points to the child object, which method is called?
    
- [ ]  **24.** What is the difference between early binding and late binding in Java?
    
- [ ]  **25.** What is a marker interface? Give examples. How is it different from an annotation?
    

- [ ] ---

- [ ] # 🔥 Java 8+ & Functional

- [ ]  **26.** What is the difference between `map()` and `flatMap()` in Streams?
    
- [ ]  **27.** What does `Optional.orElse()` vs `Optional.orElseGet()` do differently?
    
- [ ]  **28.** What is a Predicate and how can you chain multiple predicates?
    
- [ ]  **29.** What is the difference between `findFirst()` and `findAny()` in a stream?
    
- [ ]  **30.** What happens if you try to use a stream after it has already been consumed?
    
- [ ]  **31.** What is a BinaryOperator and when would you use it?
    
- [ ]  **32.** What is the difference between `Collector.toList()` and `Collector.toUnmodifiableList()`?
    
- [ ]  **33.** What is a method reference and what are its 4 types?
    
- [ ]  **34.** How does `Stream.reduce()` work? What is the identity element?
    
- [ ]  **35.** What is the difference between `peek()` and `map()` in a stream pipeline?
    

- [ ] ---

- [ ] # 🌱 Spring Core & Spring Boot

- [ ]  **36.** What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?
    
- [ ]  **37.** What is a Spring Bean scope? Name all the scopes and explain when to use prototype.
    
- [ ]  **38.** What happens if two beans of the same type exist and you try to autowire without `@Qualifier`?
    
- [ ]  **39.** What is `@Primary` and how does it interact with `@Qualifier`?
    
- [ ]  **40.** What is the difference between `@Autowired` on a field vs a constructor — which is preferred and why?
    
- [ ]  **41.** What happens if you annotate a method with `@Bean` inside a non-`@Configuration` class (i.e., `@Component`)?
    
- [ ]  **42.** What is the difference between `BeanFactory` and `ApplicationContext`?
    
- [ ]  **43.** What is a `BeanPostProcessor` and when would you use it?
    
- [ ]  **44.** What is the difference between `@Value` and `@ConfigurationProperties`?
    
- [ ]  **45.** What is `@Conditional` and how do Spring Boot auto-configurations use it?
    
- [ ]  **46.** What is the difference between `@RestController` and `@Controller` + `@ResponseBody`?
    
- [ ]  **47.** What happens if you remove `@EnableAutoConfiguration` from a Spring Boot app?
    
- [ ]  **48.** What is `CommandLineRunner` and `ApplicationRunner`? What is the difference?
    
- [ ]  **49.** What is Spring's `@EventListener` and how does event publishing work?
    
- [ ]  **50.** What is the difference between `@Scope("prototype")` and `@Scope("request")`?
    

- [ ] ---

- [ ] # 💾 JPA / Hibernate

- [ ]  **51.** What is the difference between `persist()`, `merge()`, `save()`, and `saveAndFlush()` in Spring Data JPA?
    
- [ ]  **52.** What is the difference between `FetchType.LAZY` and `FetchType.EAGER` — and what is the default for `@ManyToOne`?
    
- [ ]  **53.** What is a `LazyInitializationException` and when does it occur?
    
- [ ]  **54.** What is the difference between `@OneToOne` with `@MapsId` vs without it?
    
- [ ]  **55.** What is optimistic locking? How do you implement it in Hibernate?
    
- [ ]  **56.** What is the difference between `@Query` with JPQL and native SQL in Spring Data JPA?
    
- [ ]  **57.** What is a Projection in Spring Data JPA and when would you use it?
    
- [ ]  **58.** What happens if you call `flush()` without a transaction?
    
- [ ]  **59.** What is the difference between `CascadeType.REMOVE` and `orphanRemoval = true`?
    
- [ ]  **60.** What is the Second-Level Cache in Hibernate and how is it different from the First-Level Cache?
    

- [ ] ---

- [ ] # 🔐 Exceptions & Concurrency

- [ ]  **61.** What is the difference between `Error` and `Exception` in Java?
    
- [ ]  **62.** Can you catch multiple exceptions in one catch block? How?
    
- [ ]  **63.** What is a try-with-resources statement? What interface must the resource implement?
    
- [ ]  **64.** What is the difference between synchronized method and synchronized block?
    
- [ ]  **65.** What is a volatile variable and what problem does it solve?
    
- [ ]  **66.** What is the difference between `wait()` and `sleep()` in Java?
    
- [ ]  **67.** What is a `ThreadLocal` variable? When would you use it?
    
- [ ]  **68.** What is a deadlock? How would you detect and prevent it?
    

- [ ] ---

- [ ] # 🗄️ Collections & Data Structures

- [ ]  **69.** What is the difference between `ArrayList` and `LinkedList` in terms of performance?
    
- [ ]  **70.** What is the difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?
    
- [ ]  **71.** What happens internally in `HashMap` when two keys have the same hash code?
    
- [ ]  **72.** What is the difference between fail-fast and fail-safe iterators?
    
- [ ]  **73.** What is `ConcurrentHashMap` and how is it different from `Hashtable`?
    
- [ ]  **74.** What is the difference between `Stack` and `Deque` for stack operations?
    
- [ ]  **75.** When would you use a `WeakHashMap`?