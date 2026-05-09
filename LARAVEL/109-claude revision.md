تمام! هقولك على الـ Clean Architecture بتاعة Laravel بالكامل للـ `User` و `Job` models.

---

## 📁 Project Structure

```
app/
├── DTOs/
│   ├── UserDTO.php
│   └── JobDTO.php
├── Enums/
│   ├── UserRoleEnum.php
│   └── JobStatusEnum.php
├── Http/
│   ├── Controllers/
│   │   ├── UserController.php
│   │   └── JobController.php
│   ├── Requests/
│   │   ├── StoreUserRequest.php
│   │   └── StoreJobRequest.php
│   └── Middleware/
├── Models/
│   ├── User.php
│   └── Job.php
├── Policies/
│   ├── UserPolicy.php
│   └── JobPolicy.php
├── Repositories/
│   ├── User/
│   │   ├── UserRepositoryInterface.php
│   │   └── UserRepository.php
│   └── Job/
│       ├── JobRepositoryInterface.php
│       └── JobRepository.php
└── Services/
    ├── User/
    │   └── UserService.php
    └── Job/
        └── JobService.php

database/migrations/
├── create_users_table.php
└── create_jobs_table.php
```

---

## 🗄️ Migrations

**Users:**

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->string('role')->default('candidate'); // admin, employer, candidate
    $table->string('avatar')->nullable();
    $table->timestamps();
    $table->softDeletes();
});
```

**Jobs:**

```php
Schema::create('jobs', function (Blueprint $table) {
    $table->id();
    $table->foreignId('employer_id')->constrained('users')->cascadeOnDelete();
    $table->string('title');
    $table->text('description');
    $table->string('status')->default('draft'); // draft, published, closed
    $table->string('work_type'); // remote, onsite, hybrid
    $table->decimal('salary_min', 10, 2)->nullable();
    $table->decimal('salary_max', 10, 2)->nullable();
    $table->timestamps();
    $table->softDeletes();
});
```

---

## 🔖 Enums

```php
// app/Enums/UserRoleEnum.php
enum UserRoleEnum: string
{
    case Admin     = 'admin';
    case Employer  = 'employer';
    case Candidate = 'candidate';

    public function label(): string
    {
        return match($this) {
            self::Admin     => 'Administrator',
            self::Employer  => 'Employer',
            self::Candidate => 'Job Seeker',
        };
    }
}
```

```php
// app/Enums/JobStatusEnum.php
enum JobStatusEnum: string
{
    case Draft     = 'draft';
    case Published = 'published';
    case Closed    = 'closed';

    public function isActive(): bool
    {
        return $this === self::Published;
    }
}
```

---

## 📦 DTOs

الـ DTO بيمثل البيانات اللي بتتنقل بين الـ layers بشكل clean بدون ما تعدي الـ Request نفسها للـ Service.

```php
// app/DTOs/UserDTO.php
readonly class UserDTO
{
    public function __construct(
        public string       $name,
        public string       $email,
        public string       $password,
        public UserRoleEnum $role,
        public ?string      $avatar = null,
    ) {}

    public static function fromRequest(StoreUserRequest $request): self
    {
        return new self(
            name:     $request->validated('name'),
            email:    $request->validated('email'),
            password: bcrypt($request->validated('password')),
            role:     UserRoleEnum::from($request->validated('role')),
            avatar:   $request->validated('avatar'),
        );
    }
}
```

```php
// app/DTOs/JobDTO.php
readonly class JobDTO
{
    public function __construct(
        public int           $employerId,
        public string        $title,
        public string        $description,
        public JobStatusEnum $status,
        public string        $workType,
        public ?float        $salaryMin = null,
        public ?float        $salaryMax = null,
    ) {}

    public static function fromRequest(StoreJobRequest $request): self
    {
        return new self(
            employerId:  auth()->id(),
            title:       $request->validated('title'),
            description: $request->validated('description'),
            status:      JobStatusEnum::from($request->validated('status')),
            workType:    $request->validated('work_type'),
            salaryMin:   $request->validated('salary_min'),
            salaryMax:   $request->validated('salary_max'),
        );
    }
}
```

---

## 📝 Form Requests

```php
// app/Http/Requests/StoreUserRequest.php
class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'role'     => ['required', Rule::enum(UserRoleEnum::class)],
            'avatar'   => ['nullable', 'image', 'max:2048'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique'         => 'This email is already registered.',
            'password.confirmed'   => 'Passwords do not match.',
            'role.enum'            => 'Invalid role selected.',
        ];
    }
}
```

```php
// app/Http/Requests/StoreJobRequest.php
class StoreJobRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->role === UserRoleEnum::Employer->value;
    }

    public function rules(): array
    {
        return [
            'title'       => ['required', 'string', 'max:255'],
            'description' => ['required', 'string'],
            'status'      => ['required', Rule::enum(JobStatusEnum::class)],
            'work_type'   => ['required', Rule::enum(WorkTypeEnum::class)],
            'salary_min'  => ['nullable', 'numeric', 'min:0'],
            'salary_max'  => ['nullable', 'numeric', 'gte:salary_min'],
        ];
    }
}
```

---

## 🔒 Policies

```php
// app/Policies/JobPolicy.php
class JobPolicy
{
    // كل يوزر logged in يقدر يشوف الجوبز
    public function viewAny(User $user): bool
    {
        return true;
    }

    // بس الـ employer صاحب الجوب يقدر يعدلها
    public function update(User $user, Job $job): bool
    {
        return $user->id === $job->employer_id;
    }

    // الـ admin أو صاحب الجوب يقدر يمسحها
    public function delete(User $user, Job $job): bool
    {
        return $user->role === UserRoleEnum::Admin->value
            || $user->id === $job->employer_id;
    }

    public function create(User $user): bool
    {
        return $user->role === UserRoleEnum::Employer->value;
    }
}
```

---

## 🗃️ Repository Pattern

الـ Interface بيضمن إنك تقدر تغير الـ implementation (Eloquent, API, etc.) من غير ما تلمس الـ Service.

```php
// app/Repositories/Job/JobRepositoryInterface.php
interface JobRepositoryInterface
{
    public function all(array $filters = []): Collection;
    public function findById(int $id): ?Job;
    public function create(JobDTO $dto): Job;
    public function update(int $id, JobDTO $dto): Job;
    public function delete(int $id): bool;
}
```

```php
// app/Repositories/Job/JobRepository.php
class JobRepository implements JobRepositoryInterface
{
    public function all(array $filters = []): Collection
    {
        return Job::query()
            ->when(isset($filters['status']), fn($q) =>
                $q->where('status', $filters['status'])
            )
            ->when(isset($filters['work_type']), fn($q) =>
                $q->where('work_type', $filters['work_type'])
            )
            ->latest()
            ->get();
    }

    public function findById(int $id): ?Job
    {
        return Job::findOrFail($id);
    }

    public function create(JobDTO $dto): Job
    {
        return Job::create([
            'employer_id' => $dto->employerId,
            'title'       => $dto->title,
            'description' => $dto->description,
            'status'      => $dto->status->value,
            'work_type'   => $dto->workType,
            'salary_min'  => $dto->salaryMin,
            'salary_max'  => $dto->salaryMax,
        ]);
    }

    public function update(int $id, JobDTO $dto): Job
    {
        $job = $this->findById($id);
        $job->update([
            'title'       => $dto->title,
            'description' => $dto->description,
            'status'      => $dto->status->value,
            'work_type'   => $dto->workType,
            'salary_min'  => $dto->salaryMin,
            'salary_max'  => $dto->salaryMax,
        ]);
        return $job->fresh();
    }

    public function delete(int $id): bool
    {
        return $this->findById($id)->delete();
    }
}
```

**Binding في `AppServiceProvider`:**

```php
$this->app->bind(JobRepositoryInterface::class, JobRepository::class);
$this->app->bind(UserRepositoryInterface::class, UserRepository::class);
```

---

## ⚙️ Service Layer

الـ Service هو اللي بيحتوي الـ business logic، بياخد DTO وبيكلم الـ Repository.

```php
// app/Services/Job/JobService.php
class JobService
{
    public function __construct(
        private readonly JobRepositoryInterface $jobRepository,
    ) {}

    public function getAllJobs(array $filters = []): Collection
    {
        return $this->jobRepository->all($filters);
    }

    public function createJob(JobDTO $dto): Job
    {
        // ممكن هنا تضيف business logic زي إرسال notification للـ admin
        $job = $this->jobRepository->create($dto);

        // event(new JobCreated($job)); // مثلاً

        return $job;
    }

    public function updateJob(int $id, JobDTO $dto): Job
    {
        return $this->jobRepository->update($id, $dto);
    }

    public function deleteJob(int $id): bool
    {
        return $this->jobRepository->delete($id);
    }
}
```

---

## 🎮 Controllers

الـ Controller بيكون thin جداً، دوره بس يستقبل الـ Request، يعمل DTO، ويبعت للـ Service.

```php
// app/Http/Controllers/JobController.php
class JobController extends Controller
{
    public function __construct(
        private readonly JobService $jobService,
    ) {}

    public function index(Request $request): JsonResponse
    {
        $this->authorize('viewAny', Job::class);

        $jobs = $this->jobService->getAllJobs($request->only(['status', 'work_type']));

        return response()->json($jobs);
    }

    public function store(StoreJobRequest $request): JsonResponse
    {
        $this->authorize('create', Job::class);

        $dto = JobDTO::fromRequest($request);
        $job = $this->jobService->createJob($dto);

        return response()->json($job, 201);
    }

    public function update(StoreJobRequest $request, Job $job): JsonResponse
    {
        $this->authorize('update', $job);

        $dto = JobDTO::fromRequest($request);
        $updated = $this->jobService->updateJob($job->id, $dto);

        return response()->json($updated);
    }

    public function destroy(Job $job): JsonResponse
    {
        $this->authorize('delete', $job);

        $this->jobService->deleteJob($job->id);

        return response()->json(['message' => 'Job deleted successfully.']);
    }
}
```

---

## 🔄 Data Flow المنطقي

```
Request
  → FormRequest (validation + authorize)
  → Controller (얇 thin, بيعمل DTO)
  → DTO (carries clean data)
  → Service (business logic)
  → Repository (database operations)
  → Model (Eloquent)
```

---

## ✅ ملخص سريع لكل Layer

|Layer|مسؤوليته|
|---|---|
|**Migration**|تعريف شكل الجداول|
|**Enum**|القيم الثابتة (roles, statuses)|
|**FormRequest**|Validation + Authorization على مستوى الـ HTTP|
|**DTO**|نقل البيانات بين الـ layers بشكل type-safe|
|**Policy**|Authorization على مستوى الـ business (من يعمل إيه)|
|**Repository**|تجريد الـ database queries|
|**Service**|الـ business logic الحقيقية|
|**Controller**|استقبال الـ request وتوجيهها فقط|