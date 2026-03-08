
---

# Spring Architecture (Layers)

## 1️⃣ Controller (Request Handler Layer)

الـ **Controller** هو الطبقة التي تستقبل **HTTP Requests** القادمة من المستخدم.

يعني:

أي Request مثل:

```
GET /users
POST /orders
```

يصل أولًا إلى **Controller**.

وظيفته:

- استقبال الـ request
    
- قراءة البيانات القادمة من المستخدم
    
- إرسال الطلب إلى **Service Layer**
    

مثال:

```java
@RestController
@RequestMapping("/users")
public class UserController {
//DI
    private UserService userService;

    public UserController(UserService userService){
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable int id){
        return userService.getUser(id);
    }
}
```
 
هنا الـ Controller: يستقبل الريكويست ياخد منه  الداتا ويبعتها الي السيرفيز

1. استقبل request
    
2. استخرج `id`
    
3. أرسله إلى **Service**
    

---

# 2️⃣ Service (Business Logic Layer)

الـ **Service Layer** يحتوي على **منطق التطبيق الحقيقي**.

يعني:

- الحسابات
    
- القواعد
    
- التحقق من البيانات
    
- تنفيذ العمليات
    

الـ Controller **لا يجب أن يحتوي Business Logic**.

مثال:

```java
@Service
public class UserService {
//DI
    private UserRepository userRepository;

    public UserService(UserRepository userRepository){
        this.userRepository = userRepository;
    }

    public User getUser(int id){

        return userRepository.findById(id);

    }

}
```

الـ Service: يستقبل من ال controller ويبعته للداتا بيز 

- استقبل الطلب من Controller
    
- أرسل طلب لقاعدة البيانات عبر Repository
    

---

# 3️⃣ Repository (Data Access Layer)

الـ **Repository Layer** مسؤولة عن التعامل مع **قاعدة البيانات**.

وظيفتها:

- قراءة البيانات
    
- حفظ البيانات
    
- تعديل البيانات
    
- حذف البيانات
    

مثال:

```java
@Repository
public class UserRepository {

    public User findById(int id){

        // query database

        return new User(id,"Ali");

    }

}
```

في المشاريع الحقيقية غالبًا نستخدم:

```
Spring Data JPA
```

مثال:

```java
public interface UserRepository extends JpaRepository<User,Integer> {

}
```

---

# تدفق الطلب في Spring

الـ Request يمشي بهذه الطريقة:

```
Client
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

ثم يعود الرد:

```
Database
   ↓
Repository
   ↓
Service
   ↓
Controller
   ↓
Client
```

---

# مثال كامل بسيط

## 1️⃣ Controller

```java
@RestController
@RequestMapping("/users")
class UserController {

    private UserService userService;

    public UserController(UserService userService){
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable int id){
        return userService.getUser(id);
    }

}
```

---

## 2️⃣ Service

```java
@Service
class UserService {

    private UserRepository userRepository;

    public UserService(UserRepository userRepository){
        this.userRepository = userRepository;
    }

    public User getUser(int id){
        return userRepository.findById(id);
    }

}
```

---

## 3️⃣ Repository

```java
@Repository
class UserRepository {

    public User findById(int id){

        return new User(id,"Ali");

    }

}
```

---

# لماذا نستخدم هذا التقسيم؟

هذا يسمى:

```
Layered Architecture
```

فوائده:

### 1️⃣ تنظيم الكود

كل طبقة لها وظيفة محددة.

---

### 2️⃣ سهولة الصيانة

لو أردت تغيير قاعدة البيانات:

```
تعدل Repository فقط
```

---

### 3️⃣ قابلية الاختبار

يمكن اختبار كل طبقة وحدها.

---

# الشكل المعماري النهائي

```
Client
  ↓
Controller
(Request handling)
  ↓
Service
(Business Logic)
  ↓
Repository
(Data Access)
  ↓
Database
```

---

💡 معلومة مهمة يا علي:

في المشاريع الكبيرة غالبًا يصبح الهيكل:

```
Controller
↓
Service
↓
Repository
↓
Entity
↓
Database
```

---

لو حابب، أشرح لك أيضًا **أهم 10 أنوتيشن في Spring Boot** التي تستخدم في كل مشروع:

```
@RestController
@RequestMapping
@GetMapping
@PostMapping
@Service
@Repository
@Component
@Autowired
@Bean
@Configuration
```

وهذه فعليًا تستخدم **في كل API في العالم تقريبًا**.