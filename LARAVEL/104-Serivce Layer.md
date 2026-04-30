# 📚 شرح Service Layer - دليل شامل

## 1️⃣ **البنية الأساسية (Structure)**

```
app/Services/
├── User/
│   ├── UserService.php
│   └── UserServiceInterface.php
├── Employer/
│   ├── EmployerService.php
│   └── EmployerServiceInterface.php
├── Job/
│   ├── JobService.php
│   └── JobServiceInterface.php
├── Candidate/
│   ├── CandidateService.php
│   └── CandidateServiceInterface.php
├── Application/
│   ├── ApplicationService.php
│   └── ApplicationServiceInterface.php
└── Skill/
    ├── SkillService.php
    └── SkillServiceInterface.php
```

---
controller -> service -> repository ->db
## 2️⃣ **ما هي Service Layer؟**

```
Controller
    ↓
Service Layer ← DTOs (Input/Output)
    ↓
Repository ← Database
    ↓
Database
```

### **الوظيفة:**
- ✅ تحتوي على **Business Logic**
- ✅ تستقبل **DTOs** (بيانات نظيفة)
- ✅ تستدعي **Repositories** للبيانات
- ✅ ترجع **DTOs** للـ Controller

---

## 3️⃣ **مثال عملي: UserService**

### **المشاكل بدون Service:**
```php
// ❌ في Controller - كود مخلوط
public function store(Request $request)
{
    $user = User::create([
        'name' => $request->name,
        'email' => $request->email,
        'password' => Hash::make($request->password),
        'role' => $request->role,
    ]);
    
    // Logic أخرى...
    Log::info('User created');
    // ...
    
    return response()->json($user);
}
```

### ✅ **الحل مع Service:**
```php
// UserService.php
class UserService
{
    public function __construct(
        private UserRepository $repository,
    ) {}

    // Create user
    public function createUser(CreateUserDTO $dto): UserDTO
    {
        // 1. تحويل Password
        $data = [
            ...$dto->toArray(),
            'password' => Hash::make($dto->password),
        ];

        // 2. إنشاء المستخدم
        $user = $this->repository->create($data);

        // 3. تنفيذ logic إضافية
        Log::info("User {$user->id} created");
        // إرسال email
        // إضافة في جدول آخر
        // إلخ

        // 4. ترجع DTO
        return $this->mapToDTO($user);
    }

    // Update user
    public function updateUser(int $id, UpdateUserDTO $dto): UserDTO
    {
        $data = $dto->toArray();
        
        // بدون password null
        if (null === $data['password'] ?? null) {
            unset($data['password']);
        } else {
            $data['password'] = Hash::make($data['password']);
        }

        $user = $this->repository->update($id, $data);

        return $this->mapToDTO($user);
    }

    // Helper method
    private function mapToDTO(User $user): UserDTO
    {
        return new UserDTO(
            id: $user->id,
            name: $user->name,
            email: $user->email,
            role: UserRoleEnum::from($user->role),
            avatar: $user->avatar,
            created_at: $user->created_at,
            updated_at: $user->updated_at,
        );
    }
}
```

### **الآن في Controller:**
```php
// ✅ Controller - نظيف وواضح
public function store(CreateUserRequest $request, UserService $service)
{
    $dto = $request->toDTO(); // تحويل الـ Request إلى DTO
    $result = $service->createUser($dto); // استدعاء Service
    
    return response()->json($result->toArray());
}
```

---

## 4️⃣ **الخطوات العملية:**

### **الخطوة 1️⃣ - إنشاء Interface**
```php
// UserServiceInterface.php
interface UserServiceInterface
{
    public function createUser(CreateUserDTO $dto): UserDTO;
    public function updateUser(int $id, UpdateUserDTO $dto): UserDTO;
    public function getUserById(int $id): UserDTO;
    public function deleteUser(int $id): void;
}
```

### **الخطوة 2️⃣ - تطبيق Service**
```php
// UserService.php
class UserService implements UserServiceInterface
{
    // التطبيق هنا
}
```

### **الخطوة 3️⃣ - تسجيل في Service Provider**
```php
// RepositoryServiceProvider.php
public function register(): void
{
    // Repositories
    $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
    
    // Services
    $this->app->bind(UserServiceInterface::class, UserService::class);
}
```

---

## 5️⃣ **المسؤوليات في كل Layer**

```
┌─────────────────────────────────────────┐
│ CONTROLLER                              │
│ - Validate Input                        │
│ - Convert Request → DTO                 │
│ - Call Service                          │
│ - Return Response                       │
└─────────────────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ SERVICE                                 │
│ - Business Logic                        │
│ - Data Transformation                   │
│ - Call Repository                       │
│ - Return DTO                            │
└─────────────────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ REPOSITORY                              │
│ - Database Queries                      │
│ - CRUD Operations                       │
│ - Return Models                         │
└─────────────────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ DATABASE                                │
└─────────────────────────────────────────┘
```

---

## 6️⃣ **أمثلة إضافية:**

### **JobService - مثال معقد:**
```php
class JobService implements JobServiceInterface
{
    public function __construct(
        private JobRepository $jobRepo,
        private SkillRepository $skillRepo,
    ) {}

    public function createJob(CreateJobDTO $dto): JobDTO
    {
        // 1. Generate slug
        $slug = Str::slug($dto->title);

        // 2. تحديث DTO
        $data = [
            ...$dto->toArray(),
            'slug' => $slug,
        ];

        // 3. إنشاء Job
        $job = $this->jobRepo->create($data);

        // 4. إضافة Skills إذا موجودة
        if (!empty($dto->skills)) {
            $this->jobRepo->attachSkills($job->id, $dto->skills);
        }

        // 5. تسجيل في الـ Log
        Log::info("Job {$job->id} created by employer {$job->employer_id}");

        return $this->mapToDTO($job);
    }

    public function applyFilters(array $filters): Collection
    {
        $query = $this->jobRepo->query();

        if ($filters['work_type'] ?? null) {
            $query->whereWorkType($filters['work_type']);
        }

        if ($filters['location'] ?? null) {
            $query->whereLocation($filters['location']);
        }

        return $query->get()
            ->map(fn($job) => $this->mapToDTO($job));
    }
}
```

---

## 7️⃣ **ترتيب التطوير (الخطة):**

```
1️⃣ UserService
   ↓
2️⃣ EmployerService
   ↓
3️⃣ CandidateService
   ↓
4️⃣ JobService
   ↓
5️⃣ ApplicationService
   ↓
6️⃣ SkillService
```

---

## 8️⃣ **Dependency Injection:**

```php
class UserService
{
    // ✅ Constructor Injection
    public function __construct(
        private UserRepository $repository,
        private LogService $logService,
        private EmailService $emailService,
    ) {}
}

// في Controller:
public function store(CreateUserRequest $request, UserService $service)
{
    // Laravel يحقن التبعيات تلقائياً!
}
```

---

## 🎯 **الملخص:**

| المرحلة | الملف | المسؤولية |
|--------|------|----------|
| **1** | `UserServiceInterface.php` | تحديد العقود |
| **2** | `UserService.php` | تطبيق الـ Business Logic |
| **3** | `RepositoryServiceProvider.php` | تسجيل الـ Services |
| **4** | `UserController.php` | استدعاء الـ Service |

---

## 🚀 **ما الذي ستفعله في Service:**

✅ تشفير كلمات المرور
✅ التحقق من الأذونات
✅ تحويل البيانات
✅ استدعاء Repositories
✅ تسجيل في الـ Logs
✅ إرسال Emails
✅ معالجة الأخطاء
✅ تنفيذ Business Rules
✅ التخزين المؤقت (Caching)

---

هل تريد أبدأ مع تطبيق **UserService** أولاً؟ 🎯