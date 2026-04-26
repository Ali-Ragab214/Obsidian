# Laravel Job Board - Full Project Plan 🚀

هخليك تشتغل بأعلى مستوى من Best Practices. هقسم الكلام على مراحل واضحة.

---

## 📁 Project Structure الكاملة

```
app/
├── Models/
│   ├── User.php
│   ├── Employer.php
│   ├── Candidate.php
│   ├── Job.php
│   ├── JobCategory.php
│   ├── Application.php
│   ├── Skill.php
│   └── Notification.php
│
├── DTOs/
│   ├── Auth/
│   │   ├── RegisterDTO.php
│   │   └── LoginDTO.php
│   ├── Job/
│   │   ├── CreateJobDTO.php
│   │   └── UpdateJobDTO.php
│   ├── Application/
│   │   └── ApplyJobDTO.php
│   └── User/
│       └── UpdateProfileDTO.php
│
├── Repositories/
│   ├── Interfaces/
│   │   ├── JobRepositoryInterface.php
│   │   ├── ApplicationRepositoryInterface.php
│   │   ├── UserRepositoryInterface.php
│   │   └── CategoryRepositoryInterface.php
│   ├── JobRepository.php
│   ├── ApplicationRepository.php
│   ├── UserRepository.php
│   └── CategoryRepository.php
│
├── Services/
│   ├── AuthService.php
│   ├── JobService.php
│   ├── ApplicationService.php
│   ├── EmployerService.php
│   ├── CandidateService.php
│   └── AdminService.php
│
├── Exceptions/
│   ├── Handler.php
│   ├── NotFoundException.php
│   ├── UnauthorizedException.php
│   └── ValidationException.php
│
├── Traits/
│   ├── ApiResponseTrait.php
│   └── UploadFileTrait.php
│
├── Http/
│   ├── Controllers/
│   │   ├── BaseController.php
│   │   ├── Auth/
│   │   │   └── AuthController.php
│   │   ├── Employer/
│   │   │   ├── JobController.php
│   │   │   └── ApplicationController.php
│   │   ├── Candidate/
│   │   │   ├── JobSearchController.php
│   │   │   └── ApplicationController.php
│   │   └── Admin/
│   │       ├── JobController.php
│   │       └── UserController.php
│   │
│   ├── Middleware/
│   │   ├── RoleMiddleware.php
│   │   └── EnsureProfileComplete.php
│   │
│   ├── Requests/
│   │   ├── Auth/
│   │   │   ├── RegisterRequest.php
│   │   │   └── LoginRequest.php
│   │   ├── Job/
│   │   │   ├── StoreJobRequest.php
│   │   │   └── UpdateJobRequest.php
│   │   ├── Application/
│   │   │   └── ApplyJobRequest.php
│   │   └── Admin/
│   │       └── ApproveJobRequest.php
│   │
│   └── Resources/
│       ├── UserResource.php
│       ├── JobResource.php
│       ├── JobCollection.php
│       ├── ApplicationResource.php
│       └── CategoryResource.php
│
├── Enums/
│   ├── UserRole.php
│   ├── JobStatus.php
│   ├── ApplicationStatus.php
│   └── WorkType.php
│
└── Providers/
    └── RepositoryServiceProvider.php

database/
├── migrations/
│   ├── create_users_table.php
│   ├── create_employers_table.php
│   ├── create_candidates_table.php
│   ├── create_job_categories_table.php
│   ├── create_jobs_table.php
│   ├── create_skills_table.php
│   ├── create_job_skill_table.php       ← pivot
│   ├── create_applications_table.php
│   └── create_notifications_table.php
│
├── factories/
│   ├── UserFactory.php
│   ├── JobFactory.php
│   └── ApplicationFactory.php
│
└── seeders/
    ├── DatabaseSeeder.php
    ├── UserSeeder.php
    ├── JobCategorySeeder.php
    ├── SkillSeeder.php
    └── JobSeeder.php

resources/
└── views/
    ├── layouts/
    │   ├── app.blade.php
    │   └── admin.blade.php
    ├── auth/
    │   ├── login.blade.php
    │   └── register.blade.php
    ├── employer/
    │   ├── dashboard.blade.php
    │   ├── jobs/
    │   │   ├── index.blade.php
    │   │   ├── create.blade.php
    │   │   └── edit.blade.php
    │   └── applications/
    │       └── index.blade.php
    ├── candidate/
    │   ├── dashboard.blade.php
    │   ├── jobs/
    │   │   ├── index.blade.php
    │   │   └── show.blade.php
    │   └── applications/
    │       └── index.blade.php
    └── admin/
        ├── dashboard.blade.php
        ├── jobs/
        │   └── index.blade.php
        └── users/
            └── index.blade.php

routes/
├── web.php
├── auth.php
└── api.php  (optional)
```

#Migrations : 

php artisan make:migration create_users_table
php artisan make:migration create_employers_table
php artisan make:migration create_candidates_table
php artisan make:migration create_categories_table
php artisan make:migration create_jobs_table
php artisan make:migration create_skills_table
php artisan make:migration create_job_skill_table
User hasOne Employer
User hasOne Candidate
php artisan make:migration create_applications_table
---

## 🗃️ Database Schema والعلاقات

### جدول `users`

```
id, name, email, password, role (enum: admin/employer/candidate),
avatar, email_verified_at, timestamps
```

### جدول `employers`

```
id, user_id (FK), company_name, company_logo,
company_description, website, phone, location, timestamps
```

### جدول `candidates`

```
id, user_id (FK), resume (file path), headline,
experience_years, location, phone, timestamps
```

### جدول `job_categories`

```
id, name, slug, icon, timestamps
```

### جدول `jobs`

```
id, employer_id (FK), category_id (FK),
title, slug, description, responsibilities,
qualifications, salary_min, salary_max,
location, work_type (enum: remote/onsite/hybrid),
experience_level (enum: junior/mid/senior),
status (enum: pending/approved/rejected),
deadline, views_count, timestamps
```

### جدول `skills`

```
id, name, timestamps
```

### جدول `job_skill` (Pivot)

```
job_id, skill_id
```

### جدول `applications`

```
id, job_id (FK), candidate_id (FK),
resume_path, cover_letter,
status (enum: pending/accepted/rejected),
timestamps
```

---

## 🔄 Flow كل Feature بالترتيب

---

### 1️⃣ Authentication Flow
- **Laravel** نفسه بينصحك تعمل Admin من الـ Seeder
- // database/seeders/AdminSeeder.php

User::create([
    'name'     => 'Admin',
    'email'    => 'admin@jobboard.com',
    'password' => bcrypt('Admin@123'),
    'role'     => UserRole::Admin,
]);

```
Register →
  [Form: name, email, password, role]
    ↓
  RegisterRequest (validate)
    ↓
  AuthService::register()
    ↓
  UserRepository::create()  →  User Created
    ↓
  if role == employer → create Employer profile
  if role == candidate → create Candidate profile
    ↓
  Login + Redirect to dashboard by role
```

---

### 2️⃣ Employer - Post a Job Flow

```
Employer Dashboard → Create Job →
  [Form: title, category, description,
         skills, salary, location, work_type, deadline]
    ↓
  StoreJobRequest (validate)
    ↓
  CreateJobDTO (map data)
    ↓
  JobService::createJob(DTO)
    ↓
  JobRepository::store()
    ↓
  Job status = "pending" (awaiting admin approval)
    ↓
  Redirect → My Jobs list
```

---

### 3️⃣ Admin - Approve/Reject Job Flow

```
Admin Dashboard → Pending Jobs →
  [List of pending jobs]
    ↓
  Admin clicks Approve / Reject
    ↓
  AdminService::updateJobStatus(job_id, status)
    ↓
  JobRepository::updateStatus()
    ↓
  Notify Employer (DB Notification / Email)
    ↓
  Job appears on public listing (if approved)
```

---

### 4️⃣ Candidate - Search & Apply Flow

```
Jobs Page →
  [Search: keyword, location, category,
           experience, salary, work_type]
    ↓
  JobSearchController::index()
    ↓
  JobService::searchJobs(filters)
    ↓
  JobRepository::search()  ← Query Builder with filters
    ↓
  JobCollection Resource → Render View
    ↓
  Candidate clicks Apply →
    ↓
  ApplyJobRequest (validate: resume upload)
    ↓
  ApplyJobDTO
    ↓
  ApplicationService::apply()
    ↓
  ApplicationRepository::create()
    ↓
  Notify Employer → Application received
```

---

### 5️⃣ Employer - Review Applications Flow

```
Employer → My Jobs → View Applications →
  [List: candidate name, resume, date]
    ↓
  ApplicationController::index()
    ↓
  ApplicationService::getJobApplications(job_id)
    ↓
  Employer clicks Accept / Reject
    ↓
  ApplicationService::updateStatus()
    ↓
  Notify Candidate
```

---

## 🧱 Implementation Order (ابدأ بالترتيب ده)

```
Phase 1: Setup & Foundation
─────────────────────────────
✅ 1. laravel new job-board
✅ 2. Install: breeze or jetstream (للـ auth scaffold)
✅ 3. Create Enums (UserRole, JobStatus, ApplicationStatus, WorkType)
✅ 4. Create all Migrations
✅ 5. Create all Models + Relationships
✅ 6. RepositoryServiceProvider (bind interfaces)
✅ 7. Seeders (categories, skills, admin user)

Phase 2: Auth
─────────────────────────────
✅ 8. RegisterRequest + LoginRequest
✅ 9. AuthService (register by role)
✅ 10. AuthController
✅ 11. Views: login, register
✅ 12. RoleMiddleware

Phase 3: Employer Features
─────────────────────────────
✅ 13. StoreJobRequest + UpdateJobRequest
✅ 14. CreateJobDTO + UpdateJobDTO
✅ 15. JobRepositoryInterface + JobRepository
✅ 16. JobService
✅ 17. Employer/JobController (CRUD)
✅ 18. Views: employer dashboard, jobs CRUD
✅ 19. Employer/ApplicationController (view + respond)

Phase 4: Admin Features
─────────────────────────────
✅ 20. AdminService
✅ 21. Admin/JobController (approve/reject)
✅ 22. Admin/UserController
✅ 23. Views: admin dashboard

Phase 5: Candidate Features
─────────────────────────────
✅ 24. JobSearchController (filters)
✅ 25. ApplyJobRequest + ApplyJobDTO
✅ 26. ApplicationService + ApplicationRepository
✅ 27. Candidate/ApplicationController
✅ 28. UploadFileTrait (resume upload)
✅ 29. Views: job listing, job detail, my applications

Phase 6: Polish
─────────────────────────────
✅ 30. Notifications (Laravel Notifications)
✅ 31. Mobile responsive design (Tailwind)
✅ 32. Factories + full Seeders for testing
```

---

## ⚡ Key Best Practices هتطبقها

|Pattern|الاستخدام|
|---|---|
|**Repository Pattern**|عزل DB queries عن الـ Services|
|**Service Layer**|Business logic بعيد عن Controllers|
|**DTOs**|تمرير البيانات بين الـ Layers|
|**Form Requests**|Validation منفصل|
|**API Resources**|تشكيل الـ Response|
|**Enums (PHP 8.1)**|بدل الـ magic strings|
|**Middleware**|Role-based access control|
|**Traits**|File upload + API response reuse|
|**Policies**|Authorization (can employer edit this job?)|

---

## 🔰 ابدأ دلوقتي بالخطوة الأولى

```bash
laravel new job-board --git
cd job-board
composer require laravel/breeze
php artisan breeze:install blade
npm install && npm run dev
```

---

ابدأ بـ **Phase 1** وكل ما تخلص phase قولي وهبدأ أشرحلك التالية بالكود الكامل. تبدأ بإيه؟ الـ Migrations والـ Models؟