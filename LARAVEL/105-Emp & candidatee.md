Read [](file:///f%3A/job-board/app/Services)

Read [](file:///f%3A/job-board/app/Models/Candidate.php)

Read [](file:///f%3A/job-board/app/Models/Employer.php)

Read [](file:///f%3A/job-board/app/Models/User.php)

Read [](file:///f%3A/job-board/app/Services/User/UserService.php#1-1), lines 1 to 100

Read [](file:///f%3A/job-board/app/Services/Candidate)

Read [](file:///f%3A/job-board/app/Services/Employer)

**الوضع واضح:**

## الفهم الصحيح:

```
User Registration Flow:
┌─────────────────────────┐
│ User (base)             │
│ - name, email, password │
│ - role: enum            │
└──────────┬──────────────┘
           │
      ┌────┴────┐
      │          │
   [Employer]  [Candidate]
   (1:1 link)  (1:1 link)
```

## من يسيطر؟

**UserService يجب يتحكم في:**
1. إنشاء User
2. **إنشاء Employer/Candidate تلقائياً** بناءً على `role`
3. تحديث User

**EmployerService يتحكم في:**
1. تحديث بيانات الشركة (company_name, logo, etc)
2. جلب بيانات الشركة الخاصة بك

**CandidateService يتحكم في:**
1. تحديث Resume و بيانات المرشح
2. جلب البيانات الخاصة بك

---

## الصح هو:

```php
// في UserService عند الإنشاء:
public function createUser(CreateUserDTO $dto): UserDTO
{
    // Create User
    $user = $this->repository->create($data);
    
    // Auto-create Employer or Candidate
    if ($user->role === UserRoleEnum::EMPLOYER->value) {
        Employer::create(['user_id' => $user->id]);
    } elseif ($user->role === UserRoleEnum::CANDIDATE->value) {
        Candidate::create(['user_id' => $user->id]);
    }
    
    return $this->mapToDTO($user);
}
```

---

**الخلاصة:**
- ✅ EmployerService و CandidateService **لا مستاهلين** أساساً
- ✅ أو نعملهم بس لـ **update بيانات الملف الشخصي**
- ✅ لكن **الإنشاء يجب يكون في UserService**

تريد أنا أعدل UserService يعمل auto-create للـ Employer/Candidate؟