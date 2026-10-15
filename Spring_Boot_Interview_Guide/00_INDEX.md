# 🎯 Spring Boot Interview Guide — الفهرس الرئيسي

> **Guide مبني على ملاحظاتي الشخصية + إضافات مهمة للإنترفيو**
> آخر تحديث: October 2026

---

## 📚 جدول المحتويات

| # | القسم | الملف | عدد الأسئلة |
|---|-------|-------|-------------|
| 1 | Spring Core: IoC, DI, Beans, Bean Scopes, Bean Lifecycle | [01_Spring_Core.md](01_Spring_Core.md) | 13 |
| 2 | Spring Boot Fundamentals: Auto-config, Starters, Profiles | [02_Spring_Boot_Fundamentals.md](02_Spring_Boot_Fundamentals.md) | 13 |
| 3 | Annotations Reference الكامل | [03_Annotations.md](03_Annotations.md) | Reference |
| 4 | Spring MVC & REST APIs | [04_Spring_MVC_REST.md](04_Spring_MVC_REST.md) | 12 |
| 5 | Spring Data JPA & Hibernate | [05_Spring_Data_JPA.md](05_Spring_Data_JPA.md) | 15 |
| 6 | Exception Handling & Validation | [06_Exception_Handling_Validation.md](06_Exception_Handling_Validation.md) | 10 |
| 7 | Spring Security & JWT | [07_Spring_Security.md](07_Spring_Security.md) | 14 |
| 8 | Transactions & AOP | [08_Transactions_AOP.md](08_Transactions_AOP.md) | 10 |
| 9 | Testing (JUnit, Mockito, Spring Test) | [09_Testing.md](09_Testing.md) | 10 |
| 10 | Actuator, Logging, Caching | [10_Actuator_Logging.md](10_Actuator_Logging.md) | 8 |
| 11 | Microservices & Spring Cloud | [11_Microservices.md](11_Microservices.md) | 10 |

---

## 📌 Master Annotations List — كل الـ Annotations في مكان واحد

### Stereotype Annotations
| Annotation | المعنى |
|---|---|
| `@Component` | الأب — أي Bean عام يديره Spring |
| `@Service` | Business Logic Layer |
| `@Repository` | Data Access Layer + Exception Translation |
| `@Controller` | Spring MVC Controller (يرجع Views) |
| `@RestController` | `@Controller` + `@ResponseBody` (يرجع JSON) |

### Configuration Annotations
| Annotation | المعنى |
|---|---|
| `@Configuration` | Config class فيه Bean definitions |
| `@Bean` | تعريف Bean يدوي داخل `@Configuration` |
| `@Value("${key}")` | حقن قيمة من properties/environment |
| `@PropertySource` | تحديد ملف properties إضافي |
| `@EnableJpaAuditing` | تفعيل Auditing التلقائي |
| `@EnableWebSecurity` | تفعيل Spring Security |
| `@SpringBootApplication` | `@Configuration` + `@ComponentScan` + `@EnableAutoConfiguration` |

### Dependency Injection
| Annotation | المعنى |
|---|---|
| `@Autowired` | حقن تلقائي للـ dependency |
| `@Qualifier("name")` | تحديد أي Bean بالاسم عند وجود أكتر من implementation |
| `@Primary` | Bean افتراضي عند وجود أكتر من implementation |
| `@RequiredArgsConstructor` | Lombok — Constructor injection للـ final fields |

### Web / REST
| Annotation | المعنى |
|---|---|
| `@RequestMapping` | ربط URL بـ controller/method |
| `@GetMapping` | HTTP GET request |
| `@PostMapping` | HTTP POST request |
| `@PutMapping` | HTTP PUT request |
| `@DeleteMapping` | HTTP DELETE request |
| `@PathVariable` | استخراج قيمة من الـ URL path |
| `@RequestParam` | استخراج query parameter |
| `@RequestBody` | تحويل JSON body لـ Java object |
| `@ResponseBody` | تحويل return value لـ JSON |
| `@ResponseStatus` | تحديد HTTP status code |

### Validation
| Annotation | المعنى |
|---|---|
| `@Valid` | تفعيل validation على الـ object |
| `@Validated` | نفس `@Valid` + دعم groups |
| `@NotNull` | الحقل مش null |
| `@NotBlank` | مش null ومش فاضي ومش spaces بس |
| `@NotEmpty` | مش null ومش فاضي (للـ strings و collections) |
| `@Size(min,max)` | حجم النص أو الـ collection |
| `@Min` / `@Max` | القيمة الرقمية |
| `@Email` | لازم يكون email format |
| `@Pattern` | لازم يطابق regex معين |

### JPA / Entity
| Annotation | المعنى |
|---|---|
| `@Entity` | الكلاس ده جدول في الـ DB |
| `@Table` | تحديد اسم الجدول |
| `@Id` | Primary Key |
| `@GeneratedValue` | طريقة توليد الـ ID |
| `@Column` | تحديد خصائص العمود |
| `@MappedSuperclass` | Base Entity مش جدول — الأبناء يرثوا fields |
| `@EntityListeners` | تفعيل listeners (زي Auditing) |
| `@CreatedDate` | تاريخ الإنشاء تلقائي |
| `@LastModifiedDate` | تاريخ آخر تعديل تلقائي |
| `@OneToMany` | علاقة 1:N |
| `@ManyToOne` | علاقة N:1 |
| `@ManyToMany` | علاقة N:N |
| `@JoinColumn` | تحديد عمود الربط |
| `@Enumerated` | تخزين Enum في DB |
| `@UniqueConstraint` | Unique constraint على أعمدة |
| `@EntityGraph` | حل N+1 بجلب العلاقات Eager |
| `@BatchSize` | تحديد عدد العناصر المجلوبة في batch واحد |

### Lombok
| Annotation | المعنى |
|---|---|
| `@Data` | Getters + Setters + toString + equals + hashCode |
| `@Builder` | Builder Pattern للكلاس |
| `@Builder.Default` | قيمة افتراضية مع Builder |
| `@SuperBuilder` | Builder يدعم الوراثة |
| `@NoArgsConstructor` | Constructor فارغ |
| `@AllArgsConstructor` | Constructor بكل الـ fields |
| `@Getter` / `@Setter` | Getters و Setters |

### Security
| Annotation | المعنى |
|---|---|
| `@PreAuthorize` | فحص الصلاحيات قبل تنفيذ الـ method |
| `@Secured` | تحديد الأدوار المسموحة |

### Exception Handling
| Annotation | المعنى |
|---|---|
| `@ControllerAdvice` | Global exception handler |
| `@RestControllerAdvice` | `@ControllerAdvice` + `@ResponseBody` |
| `@ExceptionHandler` | تحديد handler لنوع exception معين |

### Transaction & Other
| Annotation | المعنى |
|---|---|
| `@Transactional` | تشغيل الـ method داخل transaction |
| `@Scope` | تحديد Bean Scope (singleton/prototype/request/session) |
| `@Profile` | تفعيل Bean في بيئة معينة فقط |
| `@Cacheable` | تخزين النتيجة في cache |
| `@CacheEvict` | مسح قيمة من الـ cache |

### Testing [ADDED - not in my notes]
| Annotation | المعنى |
|---|---|
| `@SpringBootTest` | تشغيل Application Context كامل للاختبار |
| `@WebMvcTest` | اختبار Controller فقط |
| `@DataJpaTest` | اختبار Repository فقط |
| `@MockBean` | إنشاء mock bean في Spring context |
| `@Test` | تحديد test method |

---

## 🔥 Top 20 — أكتر الأسئلة شيوعاً في الإنترفيو

1. ما هو IoC و DI؟ وإيه الفرق؟ → [Section 1](01_Spring_Core.md)
2. إيه الفرق بين `@Component` و `@Service` و `@Repository`؟ → [Section 1](01_Spring_Core.md)
3. إيه هي Bean Scopes؟ Singleton vs Prototype → [Section 1](01_Spring_Core.md)
4. إيه الفرق بين Spring Framework و Spring Boot؟ → [Section 2](02_Spring_Boot_Fundamentals.md)
5. إيه هو Auto-Configuration؟ → [Section 2](02_Spring_Boot_Fundamentals.md)
6. إيه هو `@SpringBootApplication`؟ → [Section 2](02_Spring_Boot_Fundamentals.md)
7. إيه الفرق بين `@PathVariable` و `@RequestParam`؟ → [Section 4](04_Spring_MVC_REST.md)
8. إزاي بتعمل Pagination في Spring Boot؟ → [Section 4](04_Spring_MVC_REST.md)
9. إيه الفرق بين JPA و Hibernate و Spring Data JPA؟ → [Section 5](05_Spring_Data_JPA.md)
10. إيه هي مشكلة N+1 وإزاي نحلها؟ → [Section 5](05_Spring_Data_JPA.md)
11. إيه هو `@Transactional` وبيشتغل إزاي؟ → [Section 8](08_Transactions_AOP.md)
12. إزاي بتعمل Global Exception Handling؟ → [Section 6](06_Exception_Handling_Validation.md)
13. إيه الفرق بين `@Valid` و `@Validated`؟ → [Section 6](06_Exception_Handling_Validation.md)
14. إيه هو JWT وإزاي بيشتغل مع Spring Security؟ → [Section 7](07_Spring_Security.md)
15. إيه الفرق بين Authentication و Authorization؟ → [Section 7](07_Spring_Security.md)
16. إزاي بتعمل Security Filter Chain؟ → [Section 7](07_Spring_Security.md)
17. Constructor vs Setter vs Field Injection — أنهي أفضل؟ → [Section 1](01_Spring_Core.md)
18. إيه هو DTO Pattern ولماذا نستخدمه؟ → [Section 5](05_Spring_Data_JPA.md)
19. Monolith vs Microservices → [Section 11](11_Microservices.md)
20. إزاي بتعمل Unit Testing لـ Spring Boot؟ → [Section 9](09_Testing.md)

---

## 📋 Study Plan — خطة المذاكرة المقترحة

### الأسبوع الأول: الأساسيات 🟢
- [ ] يوم 1-2: Section 1 — Spring Core (IoC, DI, Beans)
- [ ] يوم 3: Section 2 — Spring Boot Fundamentals
- [ ] يوم 4: Section 3 — Annotations (مرجع سريع)
- [ ] يوم 5: Section 4 — Spring MVC & REST

### الأسبوع الثاني: البيانات والأمان 🟡
- [ ] يوم 1-2: Section 5 — Spring Data JPA
- [ ] يوم 3: Section 6 — Exception Handling & Validation
- [ ] يوم 4-5: Section 7 — Spring Security & JWT

### الأسبوع الثالث: المواضيع المتقدمة 🔴
- [ ] يوم 1: Section 8 — Transactions & AOP
- [ ] يوم 2: Section 9 — Testing
- [ ] يوم 3: Section 10 — Actuator, Logging, Caching
- [ ] يوم 4-5: Section 11 — Microservices

### نصائح عامة 💡
- ابدأ بالأقسام اللي فيها أسئلة Top 20
- كل سؤال اقرأ الإجابة الكاملة + احفظ الـ Short Answer
- جرب تقول الإجابة بصوت عالي كأنك في إنترفيو حقيقي
- ركز على الـ Code Examples — المحاور هيطلب منك تكتب كود
- الـ System Design Links هتساعدك لو اتسألت أسئلة متقدمة

---

> ⚠️ **ملاحظة:** المحتوى المأخوذ من ملاحظاتي الأصلية مش متاجج. أي محتوى مضاف من عندي متاجج بـ `[ADDED - not in my notes]`. أي تصحيحات متاججة بـ `[CORRECTED]`.
