# 🔄 الـ Flow الكامل للـ Controllers

دعني أشرح الـ structure والـ workflow:

```
HTTP Request
    ↓
Route → Controller
    ↓
Validate Request (Form Request)
    ↓
Create DTO from validated data
    ↓
Call Service Method
    ↓
Service uses Repository for DB operations
    ↓
Service returns DTO
    ↓
Format Response using ApiResponse Trait
    ↓
HTTP Response (JSON)
```

---

## 📊 Structure الـ Controllers

```
App\Http\Controllers\
├── Controller.php                 (Base - يحتوي ApiResponse Trait)
├── AuthController.php             (Login, Register, Logout)
├── JobController.php              (CRUD Jobs)
├── CandidateController.php        (Browse Jobs, Apply)
├── ApplicationController.php       (Manage Applications)
├── EmployerController.php         (Employer Profile & Stats)
├── SkillController.php            (Skill Management)
└── Requests/                      (Form Validation)
    ├── LoginRequest.php
    ├── RegisterRequest.php
    ├── CreateJobRequest.php
    ├── UpdateJobRequest.php
    ├── ApplyJobRequest.php
    └── ...
```

---

## 🔐 Authentication Flow

```
User Registration
    ↓
UserService::createUser() 
    ↓
Hash Password + Auto-create Employer/Candidate
    ↓
Return UserDTO
    ↓
Generate Token (Sanctum/JWT)
    ↓
Return Token in Response
```

---

## 🛡️ Authorization Levels

```
ADMIN
├── View all users
├── Approve/Reject employers
└── View analytics

EMPLOYER
├── Create/Edit own jobs
├── View own applications
└── Accept/Reject candidates

CANDIDATE
├── Browse jobs
├── Apply for jobs
└── View own applications
```

---

## 📝 مثال Flow كامل (Create Job)

```
1. POST /api/jobs
   ├── AuthMiddleware ✓ (User logged in)
   ├── JobController::store()
   │   ├── Validate Request
   │   ├── Check Authorization (Is Employer?)
   │   ├── Create CreateJobDTO
   │   ├── Call JobService::createJob($dto)
   │   │   ├── Call JobRepository::create()
   │   │   └── Return JobDTO
   │   └── Return successResponse()
   │
   └── Response:
       {
         "success": true,
         "message": "Job created successfully",
         "data": { id, title, salary_min, ... }
       }
```

---

## 📂 الـ Controllers اللي محتاجين:

### 1️⃣ **AuthController**
```php
- POST /api/register
- POST /api/login
- POST /api/logout
- GET /api/me (Current user profile)
```

### 2️⃣ **JobController** (موجود)
```php
- GET /api/jobs (List all)
- GET /api/jobs/{id}
- POST /api/jobs (Create - Employer only)
- PUT /api/jobs/{id} (Update - Employer only)
- DELETE /api/jobs/{id} (Delete - Employer only)
- GET /api/employers/{id}/jobs (Jobs by employer)
```

### 3️⃣ **CandidateController**
```php
- GET /api/jobs/search (Advanced search)
- GET /api/candidates/{id} (Profile)
- PUT /api/candidates/{id} (Update profile)
```

### 4️⃣ **ApplicationController**
```php
- POST /api/jobs/{id}/apply (Candidate applies)
- GET /api/applications (List candidate's applications)
- GET /api/applications/{id} (View application)
- PUT /api/applications/{id} (Update status - Employer)
```

### 5️⃣ **EmployerController**
```php
- GET /api/employers/{id} (Profile)
- PUT /api/employers/{id} (Update profile)
- GET /api/employers/{id}/statistics (Job stats)
```

---

## 🔑 Key Design Patterns

```
Controller
    ↓
Service Interface (Dependency Injection)
    ↓
Service Implementation
    ↓
Repository Interface
    ↓
Repository Implementation
    ↓
Model (Eloquent)
    ↓
Database
```

**Benefits:**
- ✅ Testable
- ✅ Loosely Coupled
- ✅ Easy to maintain
- ✅ Clean Architecture

---

## 📋 Error Handling Flow

```
Exception thrown in Service
    ↓
Controller catches exception
    ↓
Check exception type
    ├── JobNotFoundException → 404
    ├── ValidationException → 422
    ├── UnauthorizedUserActionException → 403
    └── Generic Exception → 500
    ↓
Return error response with message
```

---

اتفق معي على الـ structure ده؟ ولا عايز تعديلات؟ 🚀