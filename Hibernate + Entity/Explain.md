
---

# 🧠 Entity + Hibernate — شرح كامل

---

## 📌 أولاً — الفكرة الأساسية

**Hibernate** هو الـ ORM (Object Relational Mapping).  
يعني بدل ما تكتب SQL بنفسك، بتكتب Java classes وHibernate بيحولها لـ SQL.

### 🧩 القاعدة الذهبية

```
Class  = Table  
Field  = Column  
Object = Row
```

---

## 🧱 ثانياً — الـ Annotations كلها بالتفصيل

---

### 1️⃣ `@Entity` و `@Table`

```java
@Entity                          // ← بيقول لـ Hibernate: الـ class دي ممثلة بـ table في الـ DB
@Table(name = "students")        // ← اسم الـ table (لو مش موجودة → اسم الـ class)
public class Student {
    // ...
}
```

💡 لو قلت بس `@Entity` من غير `@Table`  
→ Hibernate هيعمل table اسمها `student` (lowercase)

---

### 2️⃣ `@Id` و `@GeneratedValue`

```java
@Id                                                    // ← ده الـ Primary Key
@GeneratedValue(strategy = GenerationType.IDENTITY)    // ← الـ DB تولّد الـ id أوتوماتيك (AUTO_INCREMENT)
private Long id;
```

### ⚙️ استراتيجيات الـ ID

|Strategy|معناه|امتى تستخدمه|
|---|---|---|
|`IDENTITY`|DB بتعمل AUTO_INCREMENT|MySQL / PostgreSQL / H2|
|`SEQUENCE`|Hibernate بيعمل sequence object|Oracle / PostgreSQL preferred|
|`AUTO`|Hibernate يختار الأنسب|مش موصى بيه في production|
|`UUID`|String UUID كـ PK|لما عايز distributed IDs|

---

### 3️⃣ `@Column`

```java
@Column(
    name = "full_name",      // اسم الـ column في الـ DB
    nullable = false,        // NOT NULL في الـ DB
    length = 100,            // VARCHAR(100)
    unique = false           // مش unique
)
private String name;

@Column(
    precision = 10,          // عدد الأرقام الكلي
    scale = 2                // عدد الأرقام بعد العلامة العشرية
)
private BigDecimal price;    // → DECIMAL(10,2) في الـ DB

@Column(updatable = false)   // بيتكتب مرة واحدة ومش بيتحدث
private LocalDateTime createdAt;
```

💡 لو مش كتبت `@Column` خالص  
→ Hibernate بيعمل column باسم الـ field أوتوماتيك

---

### 4️⃣Enumerated

---

# 🧠 يعني إيه `@Enumerated` في Hibernate؟

هي Annotation بتستخدمها لما يكون عندك **Enum في Java**  
وعايز تخزنه في الـ Database.

---

## 🎯 المشكلة أصلاً

بص المثال ده:

```java
public enum Status {
    ACTIVE,
    INACTIVE,
    BANNED
}
```

ولو استخدمته كدا في Entity:

```java
private Status status;
```

❌ Hibernate بشكل افتراضي هيخزنه كـ **رقم (Ordinal)**:

|القيمة|اللي هيتخزن|
|---|---|
|ACTIVE|0|
|INACTIVE|1|
|BANNED|2|

---

## ⚠️ المشكلة الخطيرة

لو عدلت في الـ enum:

```java
public enum Status {
    ACTIVE,
    BANNED,     // غيرت الترتيب!
    INACTIVE
}
```

💥 كل الداتا القديمة هتبوظ  
لأن:

- 1 كانت INACTIVE
    
- بقت BANNED!
    

---

# ✅ الحل: `@Enumerated`

```java
@Enumerated(EnumType.STRING)
private Status status;
```

---

## 🎯 النتيجة

بدل ما يخزن رقم  
→ هيخزن String:

|القيمة|اللي هيتخزن|
|---|---|
|ACTIVE|"ACTIVE"|
|INACTIVE|"INACTIVE"|
|BANNED|"BANNED"|

✔ آمن  
✔ واضح  
✔ مفيهوش مشاكل لو غيرت الترتيب

---

# ⚙️ أنواع `EnumType`

## 1️⃣ `ORDINAL` (❌ مش مستحب)

```java
@Enumerated(EnumType.ORDINAL)
```

- بيخزن رقم
    
- سريع شوية
    
- خطر جداً
    

---

## 2️⃣ `STRING` (✅ الأفضل)

```java
@Enumerated(EnumType.STRING)
```

- بيخزن نص
    
- أبطأ فرق بسيط جداً
    
- آمن 100%
    

---

# 🧩 مثال كامل

```java
@Entity
public class Student {

    @Id
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Status status;
}
```

---

# 🏆 القاعدة الذهبية

```id="3x4g2b"
Always use @Enumerated(EnumType.STRING)
```

---

# 💡 امتى أستخدم Enum أصلاً؟

استخدمه لما يكون عندك قيم ثابتة زي:

- Status (ACTIVE, INACTIVE)
    
- Role (ADMIN, USER)
    
- PaymentType (CASH, CARD)
    

---

# 🔥 الخلاصة

- `@Enumerated` → بيحدد Hibernate يخزن enum إزاي
    
- `ORDINAL` → رقم ❌ خطر
    
- `STRING` → نص ✅ الأفضل
    

---

لو عايز نخش أعمق 🔥  
أشرحلك:

- إزاي تعمل Enum custom mapping
    
- أو تستخدمه مع validation + business logic
    

قولّي 👍
---

### 5️⃣ `@Transient`

```java
@Entity
public class Student {
    private String firstName;
    private String lastName;

    @Transient   // ← مش هيتحفظ في الـ DB خالص
    private String fullName;

    public String getFullName() {
        return firstName + " " + lastName;  // بيتحسب في الـ runtime
    }
}
```
بيتجاهل الفيلد مش هيخزنه في الداتا بيز 



---

### 6️⃣ Lifecycle Hooks 🔄


تمام 🔥 دي من أهم الحاجات في Hibernate وبتفرق جامد في الفهم

---

# 🔄 Lifecycle Hooks في Hibernate

هي **Annotations** بتخليك تنفّذ كود أوتوماتيك في مراحل معينة من حياة الـ Entity.

---

## 🧠 يعني إيه Lifecycle؟

الـ Entity بيمر بمراحل زي:

```id="u0i4p8"
New → Managed → Updated → Deleted
```

وHibernate بيديك Hooks تمسك فيها وتنفذ كود 👌

---

# 🧩 أهم Lifecycle Annotations

---

## 1️⃣ `@PrePersist`

👉 قبل أول `INSERT`

```java
@PrePersist
protected void onCreate() {
    this.createdAt = LocalDateTime.now();
}
```

✔ بيتنفذ أول مرة بس  
✔ مناسب لـ:

- createdAt
    
- default values
    

---

## 2️⃣ `@PostPersist`

👉 بعد ما يتم الحفظ في DB

```java
@PostPersist
protected void afterSave() {
    System.out.println("Entity saved!");
}
```

✔ مناسب لـ:

- logging
    
- events
    

---

## 3️⃣ `@PreUpdate`

👉 قبل أي `UPDATE`

```java
@PreUpdate
protected void onUpdate() {
    this.updatedAt = LocalDateTime.now();
}
```

✔ مناسب لـ:

- updatedAt
    

---

## 4️⃣ `@PostUpdate`

👉 بعد التحديث

```java
@PostUpdate
protected void afterUpdate() {
    System.out.println("Entity updated");
}
```

---

## 5️⃣ `@PostLoad`

👉 بعد ما Hibernate يعمل `SELECT`

```java
@PostLoad
protected void onLoad() {
    System.out.println("Loaded: " + this.name);
}
```

✔ مفيد لـ:

- computed fields
    
- debugging
    

---

## 6️⃣ `@PreRemove`

👉 قبل الحذف

```java
@PreRemove
protected void onDelete() {
    System.out.println("Deleting: " + this.name);
}
```

---

## 7️⃣ `@PostRemove`

👉 بعد الحذف

```java
@PostRemove
protected void afterDelete() {
    System.out.println("Deleted!");
}
```

---

# 🧭 جدول ملخص سريع

|Annotation|إمتى بيشتغل|
|---|---|
|`@PrePersist`|قبل INSERT|
|`@PostPersist`|بعد INSERT|
|`@PreUpdate`|قبل UPDATE|
|`@PostUpdate`|بعد UPDATE|
|`@PostLoad`|بعد SELECT|
|`@PreRemove`|قبل DELETE|
|`@PostRemove`|بعد DELETE|

---

# 🏗️ مثال كامل

```java
@Entity
public class Student {

    private String name;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist
    void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    @PostLoad
    void onLoad() {
        System.out.println("Loaded student: " + name);
    }

    @PreRemove
    void onDelete() {
        System.out.println("Deleting student: " + name);
    }
}
```

---

# ⚠️ ملاحظات مهمة

## 1️⃣ بتشتغل أوتوماتيك

مش بتستدعيها بنفسك ❌  
Hibernate هو اللي بيناديها ✔

---

## 2️⃣ لازم تكون داخل Entity

```java
@Entity
public class Student {
    // هنا بس
}
```

---

## 3️⃣ متستخدمهاش في business logic تقيل

❌ غلط:

- API calls
    
- logic كبير
    

✔ صح:

- timestamps
    
- logging
    
- validation خفيفة
    

---

# 💡 استخدامات عملية

✔ `createdAt / updatedAt`  
✔ logging  
✔ auditing  
✔ حساب values بعد load

---

# 🔥 الخلاصة

- Lifecycle Hooks = نقاط تدخل في حياة الـ Entity
    
- بتخليك تضيف logic بدون ما تكتبها في service
    
- أهمهم:
    
    - `@PrePersist`
        
    - `@PreUpdate`
        
    - `@PostLoad`
        

---

لو عايز نكمل 🔥  
أشرحلك:

- الفرق بين Lifecycle Hooks و Event Listeners
    
- أو نربطها بـ Spring Boot + Auditing (Production level)
    

قولّي 👍

---

## 🔗 ثالثاً — الـ Relationships (العلاقات)

> ده أصعب جزء — ركّز فيه كويس 🔥

---

### 1️⃣ تمام 🔥 نبدأ بقى أهم جزء فعلاً — العلاقات  
وأولهم: **One-to-One** 💪

---

# 🔗 يعني إيه One-to-One؟

ببساطة:

```id="0avpyr"
Entity A ↔ Entity B
واحد ليه واحد بس
```

### 🎯 مثال واقعي

- Student → Profile
    
- User → Address
    
- Person → Passport
    

👉 كل واحد مرتبط بواحد بس

---

# 🧠 الفكرة في الداتابيز

في One-to-One لازم يبقى في:

✔ **Foreign Key (FK)**  
✔ وبيبقى في طرف واحد بس

---

## 🎯 شكل الـ Table

### Student Table

|id|name|profile_id|
|---|---|---|

### Profile Table

| id | bio |

👉 `profile_id` هو الـ FK

---

# 🏗️ التطبيق في Hibernate

---

## ✅ الـ Owning Side (الأساسي)

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id") // FK
    private StudentProfile profile;
}
```

---

## 🔁 الـ Inverse Side

```java
@Entity
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;

    @OneToOne(mappedBy = "profile")
    private Student student;
}
```

---

![[Pasted image 20260320210123.png]]

---

# ⚙️ إضافة Cascade

```java
@OneToOne(cascade = CascadeType.ALL)
```

✔ لما تحفظ Student  
→ يحفظ Profile تلقائي

✔ لما تمسح Student  
→ يمسح Profile

---

# ⚠️ `unique = true` (مهم)

```java
@JoinColumn(name = "profile_id", unique = true)
```

✔ عشان تضمن إنها فعلاً One-to-One  
❌ مش One-to-Many بالغلط

---

# 🔁 Flow حقيقي

```java
Student s = new Student();
StudentProfile p = new StudentProfile();

s.setProfile(p);

session.save(s);
```

✔ هيحفظ الاتنين لو في cascade

---

# 🧠 Bidirectional vs Unidirectional

## ✅ Unidirectional (أسهل)

```java
@OneToOne
private StudentProfile profile;
```

👉 من طرف واحد بس

---

## 🔁 Bidirectional (أقوى)

```java
student ↔ profile
```

✔ تقدر توصل من الطرفين  
✔ مناسب للمشاريع الكبيرة

---

# ⚠️ أخطاء شائعة
# ⚠️ مهم جدًا

🔴 اللي جوه `mappedBy`:

mappedBy = "department"

👉 هو **اسم الـ field في الكلاس التاني**  
مش اسم الجدول ❌  
مش اسم الكولمن ❌

✔ اسم الـ variable في الكود

## ❌ نسيان `mappedBy`

→ Hibernate يعمل FK تاني  
💥 تبقى عامل relationship غلط

---

## ❌ نسيان `unique = true`

→ العلاقة تتحول OneToMany بدون ما تاخد بالك

---

## ❌ عدم ربط الطرفين

```java
s.setProfile(p);
// لازم كمان:
p.setStudent(s);
```

---

# 🏆 أفضل Practice

```java
public void setProfile(StudentProfile profile) {
    this.profile = profile;
    profile.setStudent(this);
}
```

✔ تضمن consistency

---

# 🔥 الخلاصة

- OneToOne = كل واحد ليه واحد
    
- فيه **Owning Side** (فيه FK)
    
- استخدم `@JoinColumn`
    
- استخدم `mappedBy` في الطرف التاني
    
- استخدم `cascade` عند الحاجة
    

---

### 2️⃣ @OneToMany / @ManyToOne



# 🧠 أولاً: الفكرة الأساسية

## 🎯 العلاقة دي معناها إيه؟

👉 **OneToMany = واحد عنده كتير**  
👉 **ManyToOne = الكتير تابع لواحد**

---

## 💡 مثال واقعي

- Department واحد 👨‍💼
    
- فيه Students كتير 👨‍🎓👨‍🎓👨‍🎓
    

يبقى:

👉 Department → OneToMany → Students  
👉 Student → ManyToOne → Department

---

# 🔥 أهم قاعدة في الموضوع كله

👉 **الـ FK دايمًا بيكون في ناحية الـ ManyToOne**

يعني:

- جدول **Student** فيه `department_id`
    
- مش في Department
    

---

# 🧩 الكود الكامل

## 🟢 Department (الـ One side)

```java
@Entity
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department") // 👈 مهم
    private List<Student> students;
}
```

---

## 🟡 Student (الـ Many side)

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id") // 👈 ده الـ FK
    private Department department;
}
```

---

# 🔍 نشرح كل جزء

---

## 🔹 `@ManyToOne` (الأهم)

👉 ده الطرف الأقوى (Owner)

```java
@ManyToOne
@JoinColumn(name = "department_id")
```

✔ هو اللي:

- فيه الـ FK
    
- بيتحكم في العلاقة
    
- بيكتب في الداتا بيز
    

---

## 🔹 `@OneToMany`

```java
@OneToMany(mappedBy = "department")
```

👉 ده الطرف العكسي (Inverse)

✔ مفيهوش FK  
✔ مجرد view للعلاقة

---

# 💣 يعني إيه `mappedBy = "department"`؟

👉 معناها:

> العلاقة موجودة في الـ field اللي اسمه `department` جوه كلاس Student

---

# ⚠️ أهم نقطة لازم تثبت

👉 اللي جوه `mappedBy` = اسم variable  
مش column ❌  
مش table ❌

---

# 🧠 الشكل في الداتا بيز

## جدول Department

```
id | name
```

## جدول Student

```
id | name | department_id  👈 FK
```

---

# 🔁 نفس الفكرة في EF Core (عشان تربط)

```csharp
public class Student
{
    public int DepartmentId { get; set; }   // FK
    public Department Department { get; set; }
}

public class Department
{
    public List<Student> Students { get; set; }
}
```

👉 EF Core مش محتاج `mappedBy`  
👉 هو بيفهم لوحده

---

# 🔥 سيناريو مهم (لازم تفهمه)

## ❓ لو عملت كدا

```java
student.setDepartment(department);
```

👉 Hibernate هيعمل:  
✔ يحط `department_id` في DB

---

## ❓ لو عملت كدا بس

```java
department.getStudents().add(student);
```

❌ مش هيحصل حاجة في DB!

---

## 🎯 ليه؟

👉 لأن:

- Department = Inverse side
    
- Student = Owning side
    
Owning Side <=> Inverse Side 

---

# ✅ الحل الصح

لازم تربط الاتنين:

```java
student.setDepartment(department);
department.getStudents().add(student);
```

---

# 💡 عشان كدا بنعمل Helper Method

```java
public void addStudent(Student student) {
    students.add(student);
    student.setDepartment(this);
}
```

---

# 🚨 أخطاء شائعة

❌ تنسى `mappedBy`  
❌ تحط FK في OneToMany  
❌ تعتمد على One side بس  
❌ متربطش الطرفين

---

# 🏆 الخلاصة

👉 `ManyToOne` هو المهم (Owner)  
👉 `OneToMany` مجرد view  
👉 FK دايمًا في Many  
👉 `mappedBy` بيربط بالعكس

---

# 🧠 جملة تحفظها

```text
ManyToOne = Owner (فيه FK)
OneToMany = Inverse (mappedBy)
```


---

### 3️⃣ `@ManyToMany`


# 🧠 أولاً: يعني إيه `ManyToMany`؟

## 🎯 الفكرة

👉 **ManyToMany = كتير مرتبط بكتير**

---

## 💡 مثال واقعي

- Student 👨‍🎓 يقدر ياخد كذا Course
    
- Course 📚 فيها كذا Student
    

يبقى:

👉 Student ↔ Course  
👉 Many ↔ Many

---

# 🔥 أهم حاجة تفهمها

❗ مفيش FK مباشر هنا

👉 بدل كدا Hibernate بيعمل:

## 🧩 **Junction Table (جدول وسيط)**

---

## 🗃️ شكل الداتا بيز

### Student

```
id | name
```

### Course

```
id | title
```

### student_courses (الجدول الوسيط)

```
student_id | course_id
```

---

# 🧩 الكود

## 🟢 Student (Owning Side)

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_courses", // اسم الجدول الوسيط
        joinColumns = @JoinColumn(name = "student_id"), // FK بتاع Student
        inverseJoinColumns = @JoinColumn(name = "course_id") // FK بتاع Course
    )
    private Set<Course> courses = new HashSet<>();
}
```

---

## 🟡 Course (Inverse Side)

```java
@Entity
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses") // 👈 مهم
    private Set<Student> students = new HashSet<>();
}
```

---

# 🔍 نشرح أهم أجزاء

---

## 🔹 `@JoinTable` (أهم سطر)

```java
@JoinTable(
  name = "student_courses",
  joinColumns = @JoinColumn(name = "student_id"),
  inverseJoinColumns = @JoinColumn(name = "course_id")
)
```

👉 ده بيقول:

- اعمل جدول وسيط
    
- فيه عمودين:
    
    - student_id
        
    - course_id
        

---

## 🔹 Owning Side vs Inverse

### 🟢 Student

✔ هو الـ Owner  
✔ هو اللي بيتحكم في العلاقة  
✔ هو اللي فيه `@JoinTable`

---

### 🟡 Course

✔ Inverse  
✔ فيه `mappedBy`

```java
mappedBy = "courses"
```

👉 يعني:

> العلاقة موجودة في field اسمها `courses` جوه Student

---

# ⚠️ أهم نقطة

👉 اللي جوه `mappedBy` = اسم الـ variable  
مش اسم الجدول ❌

---

# 🔥 سيناريو مهم جدًا

## ❓ لو عملت كدا

```java
student.getCourses().add(course);
```

👉 Hibernate هيعمل:  
✔ insert في `student_courses`

---

## ❌ لو عملت كدا بس

```java
course.getStudents().add(student);
```

❌ مش هيحصل حاجة!

---

## 🎯 ليه؟

👉 لأن:

- Student = Owning Side
    
- Course = Inverse
    

---

# ✅ الحل الصح

```java
student.getCourses().add(course);
course.getStudents().add(student);
```

---

# 💡 Helper Method (أفضل حل)

```java
public void addCourse(Course course) {
    courses.add(course);
    course.getStudents().add(this);
}
```

---

# 🔁 نفس الفكرة في EF Core

```csharp
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students);
```

👉 EF Core:

- بيعمل الجدول الوسيط لوحده
    
- أسهل بكتير
    

---

# ⚠️ أخطاء شائعة (مهمة جدًا)

❌ تحط `CascadeType.ALL` هنا  
👉 ممكن تمسح Courses بالغلط!

✔ الصح:

```java
cascade = {CascadeType.PERSIST, CascadeType.MERGE}
```

---

❌ تستخدم `List`  
✔ الأفضل:

```java
Set<Course>
```

👉 عشان تمنع التكرار

---

❌ تنسى `mappedBy`  
👉 Hibernate يعمل جدولين 😵

---

# 💣 معلومة مهمة جدًا (Senior Level)

👉 لو العلاقة فيها data إضافية (زي grade)

❌ ما تستخدمش ManyToMany  
✔ تعمل Entity جديدة:

```java
StudentCourse
```

---

# 🧠 الخلاصة

👉 ManyToMany = جدول وسيط  
👉 Owning Side = فيه `@JoinTable`  
👉 Inverse = فيه `mappedBy`  
👉 التعديل لازم من الـ Owner

---

# 🧠 جملة تحفظها

```text
ManyToMany = Junction Table + Owning Side + mappedBy
```

---

لو عايز أبسطهالك أكتر 👇  
أديك رسمه + سيناريو عملي زي production  
أو أديك سؤال انترفيو صعب عليه 😏🔥

---


### 4️⃣ Cascade + orphanRemoval

---

# 🧠 أولاً: يعني إيه Cascade؟

## 🎯 الفكرة ببساطة

👉 **Cascade = العمليات تنتقل من Parent → Child**

---

## 💡 مثال

عندك:

- Department (Parent)
    
- Students (Children)
    

---

## 🔥 بدون Cascade

```java
departmentRepository.save(department);
```

❌ الطلبة مش هيتحفظوا!

---

## ✅ مع Cascade

```java
@OneToMany(cascade = CascadeType.ALL)
private List<Student> students;
```

```java
departmentRepository.save(department);
```

✔ هيحفظ:

- Department
    
- وكل Students اللي جواه
    

---

# 🎯 أنواع Cascade (المهم منهم)

## 🔹 `CascadeType.PERSIST`

👉 لما تعمل save للـ parent → يعمل save للـ children

---

## 🔹 `CascadeType.REMOVE`

👉 لما تمسح parent → يمسح children

---

## 🔹 `CascadeType.MERGE`

👉 لما تعمل update → يحدث children

---

## 🔹 `CascadeType.ALL`

👉 يعمل كل اللي فوق

---

# 💣 مثال مهم

```java
@OneToMany(cascade = CascadeType.ALL)
```

👉 delete department  
✔ يمسح كل students

---

# ⚠️ امتى تستخدم Cascade؟

✔ لما الـ child **تابع بالكامل** للـ parent  
زي:

- Order → OrderItems
    
- Department → Students (أحيانًا)
    

---

# 🚫 امتى ما تستخدموش؟

❌ في ManyToMany  
(ممكن تمسح بيانات مشتركة 😵)

---

# 🧠 ثانياً: يعني إيه orphanRemoval؟

## 🎯 الفكرة

👉 **orphan = طفل اتساب لوحده**

---

## 💡 مثال

```java
@OneToMany(orphanRemoval = true)
private List<Student> students;
```

---

## 🔥 السيناريو

```java
department.getStudents().remove(student);
```

---

## ❓ بدون orphanRemoval

❌ student لسه موجود في DB  
بس مش مربوط بالـ department

---

## ✅ مع orphanRemoval = true

✔ Hibernate هيعمل:  
👉 DELETE للـ student من DB

---

# 🔥 الفرق بين Cascade و orphanRemoval

||Cascade|orphanRemoval|
|---|---|---|
|بيشتغل امتى|لما تعمل operation (save/delete)|لما تشيل child من list|
|بيعمل إيه|ينقل العملية|يمسح orphan|
|مثال|delete parent|remove من collection|

---

# 💣 مثال مهم جدًا

```java
@OneToMany(
  cascade = CascadeType.ALL,
  orphanRemoval = true
)
```

## السيناريو:

### 🟢 1. Save

✔ يحفظ parent + children

---

### 🔴 2. Delete parent

✔ يمسح الكل

---

### 🟡 3. Remove student من list

✔ يمسح student من DB

---

# ⚠️ نقطة مهمة جدًا

👉 orphanRemoval بيشتغل بس لما:

```java
remove من collection
```

مش لما:

```java
student.setDepartment(null);
```

---

# 🧠 ربطها بـ EF Core

## Cascade

✔ موجود (Cascade Delete)

## orphanRemoval

✔ زي:

```csharp
DeleteBehavior.Cascade
```

أو tracking relationships

---

# 🏆 الخلاصة

👉 Cascade = العمليات تنتقل  
👉 orphanRemoval = يمسح اللي اتشال من العلاقة

---

# 🧠 جملة تحفظها

```text
Cascade = propagate operations
orphanRemoval = delete when removed from collection
```

---
 👌
---
### 5️⃣ FetchType ⚡



# 🧠 أولاً: يعني إيه FetchType؟

## 🎯 الفكرة

👉 **FetchType = إمتى Hibernate يجيب البيانات المرتبطة**

---

## 💡 مثال

عندك:

- Department
    
- Students
    

السؤال هنا 👇  
❓ لما أجيب Department…  
هل يجيب معاه Students ولا لا؟

---

# 🔥 النوعين الأساسيين

## 🟢 1. LAZY (الأهم)

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Student> students;
```

## 🎯 معناه:

👉 "متجبش الـ students دلوقتي…  
هاتهم لما أطلبهم"

---

## 💻 مثال

```java
Department dept = repo.findById(1);
```

✔ هيجيب:

- Department بس
    

---

```java
dept.getStudents();
```

✔ ساعتها يعمل Query ويجيب الطلاب

---

## 🔥 مميزاته

✔ سريع  
✔ بيحسن الأداء  
✔ بيمنع تحميل بيانات مش محتاجها

---

# 🔴 2. EAGER

```java
@ManyToOne(fetch = FetchType.EAGER)
```

## 🎯 معناه:

👉 "هات كل حاجة مرة واحدة"

---

## 💻 مثال

```java
Student s = repo.findById(1);
```

✔ هيجيب:

- Student
    
- - Department بتاعه فورًا
        

---

# ⚠️ المشكلة الكبيرة (مهم جدًا)

## 💣 N+1 Problem

### السيناريو:

```java
List<Department> departments = repo.findAll();
```

وبعدين:

```java
for (Department d : departments) {
    d.getStudents();
}
```

---

## ❌ اللي بيحصل

- Query يجيب Departments
    
- وبعدها Query لكل Department 😵
    

👉 يعني:

```id="9qqt5j"
1 + N queries
```

---

# 🔥 الحل

## ✅ JOIN FETCH

```java
@Query("SELECT d FROM Department d JOIN FETCH d.students")
```

✔ Query واحدة بس تجيب كل حاجة

---

# 🧠 القيم الافتراضية

|العلاقة|Default|
|---|---|
|OneToMany|LAZY|
|ManyToOne|EAGER ❗|

---

# ⚠️ نصيحة مهمة جدًا

👉 غير `ManyToOne` لـ LAZY

```java
@ManyToOne(fetch = FetchType.LAZY)
```

---

# 💣 ليه EAGER خطر؟

❌ ممكن:

- يجيب داتا كتير جدًا
    
- يعمل joins تقيلة
    
- يبوّظ الأداء
    

---

# 🔁 مقارنة بـ EF Core

## EF Core

```csharp
context.Students.Include(x => x.Department);
```

👉 EF Core:

- Lazy مش default
    
- لازم تستخدم `Include`
    

---

## ⚖️ الفرق

|Hibernate|EF Core|
|---|---|
|Lazy default|No Lazy by default|
|EAGER موجود|Include|
|automatic|explicit|

---

# 🔥 سيناريو مهم

## ❓ لو عملت كدا

```java
Department dept = repo.findById(1);
dept.getStudents();
```

❌ لو session اتقفل:  
👉 LazyInitializationException

---

## الحل

✔ JOIN FETCH  
✔ أو DTO  
✔ أو Open Session (مش best practice)

---

# 🧠 الخلاصة

👉 LAZY = عند الطلب  
👉 EAGER = فورًا

---

# 🏆 القاعدة الذهبية

```text
Always use LAZY → control loading manually
```

---

# 💣 جملة تحفظها

```text
LAZY = performance
EAGER = danger
```

---

لو عايز أطلعك level أعلى 😏🔥  
أشرحلك:

- LazyInitializationException بالتفصيل
    
- أو أديك scenarios زي production bugs
    

قولّي 👌




```



---

### 🚨 N+1 Problem

```java
// بدون JOIN FETCH → query لكل student
```

### ✅ الحل

```java
@Query("SELECT d FROM Department d JOIN FETCH d.students WHERE d.id = :id")
Optional<Department> findByIdWithStudents(Long id);
```

---

## 🏗️ رابعاً — مثال كامل: 

---

# 🧠 Hibernate Full Guide (With Code & Comments)

---

تمام يا علي 👌🔥  
ده بقى **أقوى مثال كامل في Hibernate**  
✔ فيه كل حاجة: Mapping + Relationships + Cascade + Fetch + Lifecycle + Helper Methods  
✔ وكل سطر عليه شرح واضح  
✔ ده Level Production حقيقي

---

# 🧠 Full Hibernate Example (Enterprise Level)

---

# 🟣 Enum (Status)

```java
// enum للحالات → بيتخزن كـ String في DB
public enum Status {
    ACTIVE,
    INACTIVE,
    GRADUATED
}
```

---

# 🟢 Student Entity

```java
import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;

// @Entity → الكلاس ده هيبقى Table
@Entity

// اسم الجدول
@Table(name = "students")
public class Student {

    // Primary Key
    @Id

    // Auto Increment
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // عمود name
    @Column(nullable = false, length = 100)
    private String name;

    // enum → String
    @Enumerated(EnumType.STRING)
    private Status status;

    // مش بيتخزن في DB
    @Transient
    private String tempData;

    // timestamps
    @Column(updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    // Lifecycle → قبل insert
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    // Lifecycle → قبل update
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // =========================
    // RELATIONSHIPS
    // =========================

    // ManyToOne → Owning Side (فيه FK)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;

    // OneToOne → Student ليه Profile
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id") // FK
    private StudentProfile profile;

    // ManyToMany → Owning Side
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // =========================
    // GETTERS & SETTERS
    // =========================

    public Long getId() { return id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Status getStatus() { return status; }
    public void setStatus(Status status) { this.status = status; }

    public Department getDepartment() { return department; }

    // 🔥 أهم Setter → بيحدد FK
    public void setDepartment(Department department) {
        this.department = department;
    }

    public StudentProfile getProfile() { return profile; }

    public void setProfile(StudentProfile profile) {
        this.profile = profile;
    }

    public Set<Course> getCourses() { return courses; }

    // =========================
    // HELPER METHODS
    // =========================

    // ربط Student بـ Department
    public void assignDepartment(Department dept) {
        this.department = dept;
        dept.getStudents().add(this);
    }

    // OneToOne
    public void assignProfile(StudentProfile profile) {
        this.profile = profile;
        profile.setStudent(this);
    }

    // ManyToMany
    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }

    public void removeCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
}
```

---

# 🟡 Department Entity

```java
import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

// Table departments
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // OneToMany → Inverse Side
    @OneToMany(
        mappedBy = "department", // اسم field في Student
        cascade = CascadeType.ALL,
        orphanRemoval = true,
        fetch = FetchType.LAZY
    )
    private List<Student> students = new ArrayList<>();

    public Long getId() { return id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public List<Student> getStudents() { return students; }

    // Helper Methods
    public void addStudent(Student student) {
        students.add(student);
        student.setDepartment(this);
    }

    public void removeStudent(Student student) {
        students.remove(student);
        student.setDepartment(null);
    }
}
```

---

# 🔵 Course Entity

```java
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    // ManyToMany → Inverse
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();

    public Long getId() { return id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public Set<Student> getStudents() { return students; }
}
```

---

# 🔴 StudentProfile (OneToOne)

```java
import jakarta.persistence.*;

// Profile لكل Student
@Entity
@Table(name = "profiles")
public class StudentProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;

    // Inverse Side
    @OneToOne(mappedBy = "profile")
    private Student student;

    public Long getId() { return id; }

    public String getBio() { return bio; }
    public void setBio(String bio) { this.bio = bio; }

    public Student getStudent() { return student; }

    public void setStudent(Student student) {
        this.student = student;
    }
}
```

---

# 🧠 مثال استخدام فعلي (Service)

```java
@Transactional
public void testHibernate() {

    Department dept = new Department();
    dept.setName("Engineering");

    Student s1 = new Student();
    s1.setName("Ali");

    StudentProfile profile = new StudentProfile();
    profile.setBio("Backend Developer");

    Course c1 = new Course();
    c1.setTitle("Java");

    Course c2 = new Course();
    c2.setTitle("Spring");

    // ربط العلاقات
    dept.addStudent(s1);           // OneToMany
    s1.assignProfile(profile);     // OneToOne
    s1.addCourse(c1);              // ManyToMany
    s1.addCourse(c2);

    entityManager.persist(dept);   // Cascade يحفظ الكل
}
```

---

# 🏆 الخلاصة النهائية

```text
✔ Entity = Table
✔ ManyToOne = Owning Side (FK)
✔ OneToMany = mappedBy
✔ ManyToMany = JoinTable
✔ OneToOne = FK في واحد منهم
✔ Cascade = ينقل العمليات
✔ orphanRemoval = delete لما تشيل من list
✔ LAZY = الأفضل
✔ Helper Methods = لازم تربط الطرفين
```

---

# 💣 أهم 3 سطور في Hibernate كله

```java
student.setDepartment(this);   // FK
course.getStudents().add(this); // consistency
profile.setStudent(this);      // OneToOne sync
```

---

لو فهمت المثال ده 👇  
أنت كدا خلاص بقيت **فاهم Hibernate بنسبة 80%+** 🔥

ولو عايز أوصل بيك لـ 100% 😏  
أعملك:

- Debugging real bugs
    
- أو Project زي بتاعك بـ Spring Boot
    

قولّي 👌