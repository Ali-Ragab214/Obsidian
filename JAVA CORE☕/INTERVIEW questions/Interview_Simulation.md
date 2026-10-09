تمام، دلوقتي إحنا في **Java Backend Interview**، وأنا الـ Interviewer.

بص يا علي، السيشن دي معمولة من الملفات بتاعتك كلها في فولدر `INTERVIEW questions`.

هنمشي كده:

```
أنا أسأل -> أنت تجاوب -> أنا أعمل Follow-up -> أقولك هو كان بيختبر إيه فعلاً
```

الملف ده هو الفهرس والتقييم.****

والأسئلة نفسها متقسمة عشان الملف ميبقاش ضخم:

- `Interview_Simulation_Part1.md` -> Java Core (Basics + String + Collections + Exceptions + Memory/JVM + Generics + Java 8)
- `Interview_Simulation_Part2.md` -> Spring + JPA/Hibernate + DB/SQL + Misc Backend
- `Interview_Simulation_OOP.md` -> OOP كامل + SOLID + Design Patterns + الأسئلة الـ Tricky

ذاكر التلاتة ورا بعض بنفس الترتيب ده.

---

# إزاي تذاكر من المحاكاة دي؟

- اقرأ السؤال بصوت عالي كأنك في انترفيو.
- حاول تجاوب قبل ما تبص على الإجابة المثالية.
- الـ Follow-up هو اللي بيفرق Junior من Mid، ركز عليه.
- جملة `What the interviewer is really testing` بتقولك هو عايز يسمع إيه فعلاً.

---

# Question Evaluation Table

كل ملف عندك اتقري كامل، واتقيم هل هو سؤال انترفيو حقيقي ولا لأ.

> ✅ = سؤال حقيقي بيتسأل كتير
>
> ⚠️ = بيتسأل أحياناً أو بصيغة مختلفة
>
> ❌ = مش سؤال انترفيو، trivia أو trivial أو outdated أو scratch

| # | الملف (Source File) | السؤال / الموضوع | التقييم | ليه؟ (Justification) |
|---|---|---|---|---|
| 0 | `0-Sample questions.md` Q1 | `==` vs `equals()` مع Integer بين -128 و 127 | ✅ | سؤال Integer Cache مشهور جداً، بيكشف فاهم autoboxing ولا حافظ. |
| 0 | `0-Sample questions.md` Q2 | لو عملت Override لـ `equals()` من غير `hashCode()` يحصل إيه؟ | ✅ | Contract أساسي، بيتسأل مع HashMap/HashSet في كل انترفيو Backend. |
| 0 | `0-Sample questions.md` Q3 | Can an abstract class have a constructor? When called? | ✅ | بيتسأل كتير عشان يلخبطك بين abstract و instantiation. |
| 0 | `0-Sample questions.md` Q4 | `Comparable` vs `Comparator` | ✅ | عملي جداً، sorting في Colections و Spring. |
| 0 | `0-Sample questions.md` Q5 | Can you instantiate an interface? What about anonymous classes? | ✅ | Polymorphism + anonymous class، سؤال OOP كلاسيك. |
| 0 | `0-Sample questions.md` Q6 | `throw` vs `throws` | ✅ | أساسي، بيتسأل للمبتدئين والميد. |
| 0 | `0-Sample questions.md` Q7 | static method ونفس الاسم instance method في نفس الكلاس؟ | ✅ | Trick عن signature و static، بيوقع ناس كتير. |
| 0 | `0-Sample questions.md` Q8 | Can constructor be private? What pattern? | ✅ | Singleton، سؤال Design Pattern مباشر. |
| 0 | `0-Sample questions.md` Q9 | `instanceof` vs `getClass()` | ✅ | فرق دقيق في inheritance check، بيتسأل Mid/Senior. |
| 0 | `0-Sample questions.md` Q10 | exception جوه `finally`؟ | ✅ | Suppressed و precedence، سؤال Senior. |
| 0 | `0-Sample questions.md` Q11 | Can you override static؟ Can you hide it؟ | ✅ | Method hiding، من أشهر الـ traps. |
| 0 | `0-Sample questions.md` Q12 | shallow vs deep copy | ✅ | Clone و copy-constructor، بيتسأل Mid. |
| 0 | `0-Sample questions.md` Q13 | `new String("hello")` vs `"hello"` كام Object؟ | ✅ | String Pool، سؤال حفظ وفهم. |
| 0 | `0-Sample questions.md` Q14 | autoboxing ومشاكله؟ | ✅ | NPE و performance و `==` trap. |
| 0 | `0-Sample questions.md` Q15 | `break` vs `continue` vs `return` | ⚠️ | بيتسأل Junior بس، نادر في Mid/Senior. |
| 0 | `0-Sample questions.md` Q16-Q25 OOP | Composition/Aggregation, final, method hiding, LSP, marker interface... | ✅ | كلها OOP حقيقي، هنغطيها في ملف OOP. |
| 0 | `0-Sample questions.md` Q26-Q35 Java 8 | map/flatMap, Optional, Predicate, Streams... | ✅ | Streams و Optional بيتسألوا في كل Backend انترفيو. |
| 0 | `0-Sample questions.md` Q36-Q50 Spring | Component/Service, scopes, autowire, BeanFactory... | ✅ | Core Spring، أسئلة يومية. |
| 0 | `0-Sample questions.md` Q51-Q60 JPA | persist/merge, LAZY/EAGER, N+1, caches... | ✅ | JPA هو أكتر جزء عملي في الشغل. |
| 0 | `0-Sample questions.md` Q61-Q68 Exceptions & Concurrency | Error vs Exception, volatile, wait/sleep, deadlock... | ✅ | Concurrency بيتسأل Mid/Senior. |
| 0 | `0-Sample questions.md` Q69-Q75 Collections | ArrayList/LinkedList, HashMap internals, fail-fast... | ✅ | Collections أساس أي Java انترفيو. |
| 1 | `1-3. In method overriding...md` | return types, access modifiers, exceptions في overriding | ✅ | قواعد Overriding بتتسأل بالنص كده. |
| 2 | `10.1-Important notes on 10.md` | Composition vs Aggregation + ليه DI هو Aggregation | ✅ | DI كـ Aggregation سؤال فهم عميق. |
| 3 | `10-Relations Types.md` | Association vs Aggregation vs Composition | ✅ | OOP relations، بيتسأل نظري و UML. |
| 4 | `11-relations.md` | JPA relations + mappedBy + Cascade + Fetch | ✅ | أهم جزء JPA عملياً. |
| 5 | `12-EXCEPTION Q.md` | try/catch/finally كلهم throw + suppressed | ✅ | Senior trick، قليل اللي يجاوبه صح. |
| 6 | `13- 19. What is the difference between a Database and a DBMS?.md` | Database vs DBMS | ❌ | تعريف نظري بدائي، مش سؤال Java Backend. |
| 7 | `14-difference between DELETE, TRUNCATE, DROP.md` | DELETE vs TRUNCATE vs DROP | ⚠️ | سؤال DB انترفيو، مش Java، بس ممكن يتسأل في Backend عام. |
| 8 | `15-what is the difference between (Hibernate, JPA).md` | JPA spec vs Hibernate implementation | ✅ | أول سؤال JPA في أي انترفيو. |
| 9 | `16-What did Spring Boot add...md` | Spring Boot ضاف إيه على Spring؟ | ✅ | Auto-config + Starters + Embedded server، سؤال افتتاحي. |
| 10 | `17-36. What are Spring Boot Starters?.md` | Starters + starter-parent | ✅ | Dependency management، بيتسأل Junior/Mid. |
| 11 | `18-Spring proxy.md` | Spring Proxy (JDK vs CGLIB) | ✅ | أساس فهم Transactional و AOP. |
| 12 | `19-What happens when a transactional method calls another...md` | self-invocation + propagation | ✅ | أشهر Senior Spring سؤال على الإطلاق. |
| 13 | `2-Inherite Singleton class.md` | Can Singleton be inherited؟ | ✅ | Private ctor + inheritance، سؤال ذكي. |
| 14 | `20-Java 8 interface.md` | default vs static في interface | ✅ | Java 8 أساسي، backward compatibility. |
| 15 | `21-Method Reference in Java 8.md` | الـ 4 أنواع بتوع `::` | ✅ | بيتسأل مع Lambdas و Streams. |
| 16 | `22-Some Annotations interview questions.md` | Component/Bean/Configuration/Autowired/Value | ✅ | Annotations هي الشغل اليومي. |
| 17 | `23-BEAN.md` | What is a Spring Bean؟ | ✅ | تعريف افتتاحي لازم تعرفه. |
| 18 | `24-what will happen if we removed @SpringBootApplication...md` | شيلت الـ annotation من الـ Main؟ | ✅ | بيختبر فهمك للـ 3 annotations اللي جواها. |
| 19 | `25-Runtime & Compile time Polymorphism.md` | Static vs Dynamic polymorphism | ✅ | Overloading vs Overriding، سؤال OOP ثابت. |
| 20 | `26-What is Application Context.md` | ApplicationContext | ✅ | IoC container، BeanFactory vs Context. |
| 21 | `27-IOC.md` | IoC vs DI + أنواع الـ injection | ✅ | أهم مفهوم Spring. |
| 22 | `28-Bean Lifecycle.md` | دورة حياة الـ Bean | ✅ | init/destroy + PostConstruct، بيتسأل Mid. |
| 23 | `29-Variable Hiding (important).md` | Variable hiding `Bike/Honda3` | ✅ | Trap مشهور، reference-type vs object-type. |
| 24 | `3-What are static variables and static methods?.md` | static variables/methods | ✅ | أساسي Junior. |
| 25 | `30-Method overloading with null.md` | `print(null)` مع Object vs String | ✅ | Most-specific rule، سؤال Tricky محبوب. |
| 26 | `31-Static Block vs Instance Block...md` | ترتيب Static -> Instance -> Constructor | ✅ | Initialization order، بيتسأل كتير. |
| 27 | `32-Static Block Loading status.md` | إمتى الـ static block بيتنفذ؟ | ⚠️ | عادة بيتسأل ضمن السؤال اللي قبله، مش لوحده. |
| 28 | `33-Static Variables and Static Methods (Common Interview Trap).md` | ليه مينفعش `static` جوه method؟ | ⚠️ | trick لطيف Junior، بس نادر كسؤال مستقل. |
| 29 | `34-Constructor injection => Immutable.md` | ليه Constructor injection بيخلي immutable؟ | ✅ | Best practice + final + testing، سؤال Mid. |
| 30 | `34-What happens internally when a Spring Boot application starts?.md` | الـ 14 خطوة بتوع startup | ✅ | Senior: run -> context -> auto-config -> Tomcat. |
| 31 | `35-Global Exception Handling in Spring Boot.md` | ControllerAdvice + ExceptionHandler | ✅ | عملي جداً في أي REST API. |
| 32 | `36-Fix N + 1.md` | N+1 و EntityGraph | ✅ | أشهر performance سؤال JPA. |
| 33 | `37-why is the string class declared final ?.md` | ليه String final؟ | ✅ | Security + Pool + hash-cache. |
| 34 | `38-System.exit.md` | `System.exit(1)` بيعمل إيه؟ | ❌ | معلومة API بسيطة، نادراً ما تتسأل. |
| 35 | `39-Checked vs UnChecked Exceptions.md` | Checked vs Runtime | ✅ | Hierarchy كاملة، سؤال ثابت. |
| 36 | `4-How can you ensure a variable is shared across only one instance?.md` | (ملف فاضي) | ❌ | مفيش محتوى أصلاً، لا يُذاكر. |
| 37 | `40-Checked.md` | تجربة FileReader | ⚠️ | مثال عملي لنفس سؤال Checked، مش سؤال مستقل. |
| 38 | `41-UnChecked.md` | تجربة NPE | ⚠️ | نفس اللي فوق، demo مش سؤال. |
| 39 | `42-28. Explain Hibernate First-Level Cache..md` | First-Level Cache | ✅ | Persistence Context، بيتسأل Mid. |
| 40 | `43-Entity Manager.md` | EntityManager | ✅ | Gateway بتاع JPA، persist/merge/remove. |
| 41 | `44-What is Dirty Cheking...md` | Dirty Checking | ✅ | Auto-UPDATE من غير save، سؤال فهم. |
| 42 | `45-can you overload main method?.md` | Overload للـ main؟ | ⚠️ | Trick مسلي، بيتسأل Junior أحياناً. |
| 43 | `46-Can final method be inherited ?.md` | final method يتورث؟ | ✅ | inherit آه، override لأ. سؤال مباشر. |
| 44 | `47-Final vs Finally vs Finalize.md` | التلاتة اللي شبه بعض | ✅ | Classic، لازم تعرف إن finalize deprecated. |
| 45 | `48-JDBC.md` | JDBC + PreparedStatement | ⚠️ | قديم، بس SQL Injection سؤال أمني مهم. |
| 46 | `49-Important.md` | Integer `==` vs `equals()` + Cache | ✅ | Trap مشهور جداً. |
| 47 | `5-What is a method signature?.md` | signature = name + params | ⚠️ | usually بيتسأل ضمن Overloading مش لوحده. |
| 48 | `50-Web Server vs Application Server.md` | Web vs App server + Tomcat | ⚠️ | Deployment سؤال، مش Java Core. |
| 49 | `51-Variable Hiding.md` | نفس موضوع 29 بالتفصيل | ✅ | مكرر لكن مهم، اتدمج في المحاكاة. |
| 50 | `52-Different cases on Variable Hiding.md` | أسماء مختلفة مش hiding | ⚠️ | Edge case لنفس الموضوع، بيتسأل كـ follow-up. |
| 51 | `53-Brief on 51 & 52.md` | خلاصة 51 و 52 | ⚠️ | Cheat-sheet، مش سؤال مستقل. |
| 52 | `54-why is the main method static ?.md` | ليه main static؟ | ✅ | JVM bootstrap، سؤال Junior كلاسيك. |
| 53 | `55-Servlet vs Spring.md` | Servlet vs Spring MVC | ⚠️ | تاريخي، DispatcherServlet كافي. |
| 54 | `56-Is Java Completely Object-Oriented?.md` | primitives بتخليها مش pure OO؟ | ⚠️ | سؤال فلسفي، بيتسأل Junior أحياناً. |
| 55 | `57-What happens when we run a java program ?.md` | javac -> class -> JVM -> JIT | ✅ | Pipeline كاملة، سؤال أساسي. |
| 56 | `58-Is Java Pass-by-Value or Pass-by-Reference?.md` | Pass-by-value دايماً؟ | ✅ | من أشهر أسئلة Java على الإطلاق. |
| 57 | `59-Difference between for and while loop.md` | for vs while | ❌ | بدائي جداً، مفيش interviewer بيسأله. |
| 58 | `60-Pagination in interviews.md` | offset vs cursor | ✅ | عملي Backend، LIMIT/OFFSET vs WHERE id>. |
| 59 | `61-Stereo Type Annotations...md` | هل دول stereotypes؟ | ✅ | Component/Service/Repository، سؤال Spring. |
| 60 | `62- which of the following are part of @RestController...md` | مكونات RestController | ✅ | Controller + ResponseBody + Component. |
| 61 | `63-why java doesnot support multiple inheritence...md` | Diamond problem | ✅ | ليه مفيش multiple inheritance للـ classes. |
| 62 | `64-Programming fund => Arrays vs List.md` | Array vs List | ⚠️ | أساسي لكن بيتسأل Junior بصيغة When to use. |
| 63 | `65-Dictionary.md` | Dictionary = Map | ❌ | كلاس obsolete، الصح Map/HashMap. لا تذاكر Dictionary. |
| 64 | `66-@Builder.Default annotation.md` | Lombok Builder.Default | ⚠️ | Lombok trick عملي، بس مش كل الشركات بتسأله. |
| 65 | `67-String, StringBuilder, StringBuffer.md` | التلاتة + Pool + Capacity | ✅ | سؤال String الأشهر. |
| 66 | `68-Wrapper classes.md` | Wrappers + autoboxing | ✅ | Collections/Generics/parsing. |
| 67 | `69-HashMap.md` | Internals + Collision + Resize | ✅ | أهم سؤال Collections. |
| 68 | `7-Explain different uses of the super keyword...md` | استخدامات super | ✅ | ctor + field + method. |
| 69 | `70-HashCode() and equals().md` | Contract + HashSet bug | ✅ | لازم مع HashMap. |
| 70 | `71-How Security work.md` | (صورة فقط، مفيش نص) | ❌ | لا يوجد محتوى نصي، اتغطى Security في Part2 من الصفر. |
| 71 | `72-Interface = What the service can do...md` | Interface vs Impl + حقن CourseService | ✅ | DIP عملي في Spring. |
| 72 | `73-Records.md` | record vs class + JPA تحذير | ⚠️ | Java 16+، بيتسأل في الشركات الحديثة. |
| 73 | `74-Stack & Heap in java.md` | Stack vs Heap | ✅ | Memory أساسي. |
| 74 | `75-what is the file extension of compiled java bytecode.md` | امتداد الـ bytecode؟ | ❌ | `.class` معلومة بديهية، مش سؤال. |
| 75 | `76-  3. Can an abstract class have a constructor?...md` | abstract ctor إمتى بيتنادى؟ | ✅ | مكرر لكن مهم. |
| 76 | `77- Can you instantiate an interface...md` | instantiate interface؟ anonymous؟ | ✅ | مكرر لكن مهم. |
| 77 | `78-Notes on reference and object...md` | reference vs object (Head First) | ✅ | أساس فهم `new` و alias. |
| 78 | `79-Java Memory Management — Objects, References & GC.md` | `Book b/c/d` tracing | ✅ | GC eligibility tracing، بيتسأل عملي. |
| 79 | `8.1-super key word some notes.md` | ملاحظات super مختصرة | ⚠️ | نفس سؤال super، fragment مش مستقل. |
| 80 | `8-8. Why can’t we use both this() and super()...md` | ليه مش الاتنين مع بعض؟ | ✅ | first-statement rule، سؤال ctor. |
| 81 | `80-Garabage Collector.md` | `b=c; c=null` مين eligible؟ | ✅ | Tracing صغير لكن مهم. |
| 82 | `81-Diamond Problem.md` | Diamond بالتفصيل + default conflict | ✅ | مكرر مع 63 لكن أعمق. |
| 83 | `82-Constructor Privacy.md` | private ctor + Singleton | ✅ | نفس عيلة Singleton. |
| 84 | `83-What happens when a static method and an instance method...md` | نفس الاسم static vs instance | ✅ | signature لا تشمل static. |
| 85 | `84- What happens if an exception is thrown inside a finally block?.md` | exception جوه finally | ✅ | مكرر مع 12 لكن مهم. |
| 86 | `85-What is the difference between shallow copy and deep copy?...md` | Shallow vs Deep + clone | ✅ | مكرر مع Q12 لكن بالتفصيل. |
| 87 | `86-Java Is Always Pass-by-Value.md` | Pass-by-value بالأمثلة | ✅ | مكرر مع 58 لكن مهم. |
| 88 | `87-Encapsulation مش معناها Getters & Setters وخلاص.md` | Encapsulation الحقيقية | ✅ | behavior + validation مش getters بس. |
| 89 | `88-Local & Instance variable.md` | Local vs Instance | ⚠️ | أساسي Junior، defaults و init. |
| 90 | `89-What happens when you call new String...md` | كام Object؟ Pool vs Heap | ✅ | مكرر مع Q13 لكن بالتفصيل. |
| 91 | `90-Important not on String.md` | ملاحظة `new String` = 2 objects | ⚠️ | Follow-up لنفس السؤال، مش مستقل. |
| 92 | `9.1 Method Hiding and Override.md` | Cheat-sheet hiding | ⚠️ | ملخص لنفس سؤال hiding. |
| 93 | `9-Can static methods be overridden?...md` | شرح hiding الكامل | ✅ | النسخة الكاملة للسؤال. |
| 94 | `91-What is autoboxing?...md` | Autoboxing + NPE + performance | ✅ | النسخة الكاملة (809 سطر). |
| 95 | `92-Important Note On Generics on JAVA.md` | ليه autoboxing؟ عشان Generics | ⚠️ | Fragment من اللي فوقه. |
| 96 | `93-Method Overloading + Autoboxing + Method Resolution.md` | `test(int)` vs `test(Integer)` | ✅ | Exact-match wins، سؤال Tricky. |
| 97 | `94-What is the difference between break, continue, and return...md` | التلاتة في loop | ⚠️ | نفس Q15، Junior. |
| 98 | `95-What is the difference between Composition and Aggregation?.md` | Strong vs Weak HAS-A | ✅ | مكرر لكن النسخة المعتمدة. |
| 99 | `T2.md` | Generics + if vs switch (scratch) | ❌ | ملاحظات متفرقة فيها أخطاء، مش أسئلة. |
| 100 | `test.md` | Head First walkthrough ضخم | ⚠️ | شرح مرجعي ممتاز لكن مش أسئلة مباشرة، اتاخد منه الـ tracing. |
| 101 | `micro 1/0-interview topics.md` | خارطة 4 محاور | ❌ | Roadmap مش سؤال، لكن حددت هيكل المراجعة. |
| 102 | `micro 1/1.1-Stack & Heap & GC.md` | JMM + 4 patterns | ✅ | Stack/Heap/GC + Singleton/Factory/Builder/Strategy. |
| 103 | `micro 1/1-PHASE 1.md` | بنك Phase-1 (Collections/Concurrency/Exceptions/Java8) | ✅ | تجميعة أسئلة، اتوزعت على الأقسام. |
| 104 | `micro 1/2-Phase 2.md` | Spring Zatouna (7 مواضيع) | ✅ | تجميعة Spring، اتوزعت على Part2. |

---

# الخلاصة بالأرقام

- ✅ **Real Interview Question:** حوالي 72 موضوع (العمود الفقري بتاعك: HashMap/String/equals + Exceptions + Memory + OOP + Spring Proxy/Transactions + JPA N+1/LAZY).
- ⚠️ **Partially Relevant:** حوالي 24 موضوع (demos و fragments و Junior trivia زي for/while و main overload و Records و Builder.Default، تتذاكر كـ follow-up مش كأساس).
- ❌ **Not an Interview Question:** حوالي 9 (DB vs DBMS لوحده، System.exit، امتداد `.class`، for vs while كسؤال مستقل، Dictionary القديم، ملف فاضي، صورة Security من غير نص، T2 scratch، Roadmap).

لو وقتك ضيق، ذاكر الـ ✅ فقط + ملف OOP كامل.

---

# هتكمل منين؟

1. افتح `Interview_Simulation_Part1.md` وخلص Java Core.
2. افتح `Interview_Simulation_Part2.md` وخلص Spring و JPA.
3. افتح `Interview_Simulation_OOP.md` وده أهم ملف، فيه كل الـ OOP اللي بتتسأل + اللي كان ناقص عندك واتضاف.

---

# Closing: إيه اللي بيتسأل فعلاً؟ (Most Likely Topics)

## لو داخل Junior/Mid Java Backend، الـ Interviewer هيركز على:

1. **String + equals/hashCode + HashMap** (سؤال واحد متصل، 90% بيتسأل).
2. **Exceptions** (checked/unchecked + try/finally + global handling في Spring).
3. **Memory** (Stack/Heap/GC + pass-by-value + String Pool).
4. **OOP** (الـ 4 pillars + abstract/interface + overloading/overriding + static/hiding + Singleton).
5. **Spring** (IoC/DI + Bean + Proxy/self-invocation + Transactions).
6. **JPA** (relations + LAZY/EAGER + N+1 + Dirty Checking + First-Level Cache).
7. **Java 8** (Streams map/flatMap + Optional + default/static + method reference).

## إيه اللي تراجع ليلة الانترفيو؟

```
String immutability (4 أسباب) + Pool + == vs equals
↓
hashCode contract + HashMap put/get + Collision Java 8 Tree
↓
Checked/Unchecked + finally override + ControllerAdvice
↓
Static/Hiding/Overloading null + super/this + abstract ctor
↓
Proxy + self-invocation REQUIRES_NEW + Bean lifecycle
↓
LAZY/EAGER + N+1 EntityGraph + Dirty Checking
```

بالتوفيق يا علي، خش على Part1.
