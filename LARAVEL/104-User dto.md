Read [](file:///f%3A/job-board/app/Models/User.php)

Read [](file:///f%3A/job-board/app/Models/Job.php)

Read [](file:///f%3A/job-board/app/Models/Application.php)

Read [](file:///f%3A/job-board/app/Models/Employer.php)

# 📋 خطة و Structure ل DTO Layer

## 1️⃣ **البنية الأساسية**

```
app/DTOs/
├── User/
│   ├── CreateUserDTO.php
│   ├── UpdateUserDTO.php
│   └── UserDTO.php
├── Job/
│   ├── CreateJobDTO.php
│   ├── UpdateJobDTO.php
│   └── JobDTO.php
├── Application/
│   ├── CreateApplicationDTO.php
│   ├── UpdateApplicationDTO.php
│   └── ApplicationDTO.php
├── Employer/
│   ├── CreateEmployerDTO.php
│   ├── UpdateEmployerDTO.php
│   └── EmployerDTO.php
├── Candidate/
│   ├── CreateCandidateDTO.php
│   └── CandidateDTO.php
└── Skill/
    └── SkillDTO.php
```

---

## 2️⃣ **التصنيفات (ما ستنشئه في كل DTO)**

### **A) Request DTOs** (للـ Input)
- **Create**: لإنشاء entity جديد (بدون ID)
- **Update**: لتحديث entity موجود (قد يكون حقول اختيارية)

### **B) Response DTOs** (للـ Output)
- **DTO**: DTO عام للرد مع جميع البيانات

---

## 3️⃣ **الخطوات العملية للـ Implementation**

### **الخطوة 1️⃣ - إنشاء Base DTO** (أساس لكل DTOs)
```php
// app/DTOs/BaseDTO.php
abstract class BaseDTO
{
    public function toArray(): array
    {
        return get_object_vars($this);
    }
}
```

### **الخطوة 2️⃣ - User DTOs**
```
✅ UserDTO.php           (Response - بيانات المستخدم)
✅ CreateUserDTO.php     (Request - إنشاء مستخدم)
✅ UpdateUserDTO.php     (Request - تحديث مستخدم)
```

### **الخطوة 3️⃣ - Employer DTOs**
```
✅ EmployerDTO.php
✅ CreateEmployerDTO.php
✅ UpdateEmployerDTO.php
```

### **الخطوة 4️⃣ - Candidate DTOs**
```
✅ CandidateDTO.php
✅ CreateCandidateDTO.php
```

### **الخطوة 5️⃣ - Job DTOs**
```
✅ JobDTO.php
✅ CreateJobDTO.php
✅ UpdateJobDTO.php
```

### **الخطوة 6️⃣ - Application DTOs**
```
✅ ApplicationDTO.php
✅ CreateApplicationDTO.php
✅ UpdateApplicationDTO.php
```

### **الخطوة 7️⃣ - Skill DTOs**
```
✅ SkillDTO.php
```

---

## 4️⃣ **نموذج ملموس لـ DTO**

### مثال: UserDTO
```php
namespace App\DTOs\User;

class UserDTO
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public string $role,
        public ?string $avatar = null,
    ) {}

    public function toArray(): array
    {
        return get_object_vars($this);
    }
}
```

### مثال: CreateUserDTO
```php
namespace App\DTOs\User;

class CreateUserDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
        public string $role,
        public ?string $avatar = null,
    ) {}
}
```

---

## 5️⃣ **ملاحظات مهمة**

| ❌ تجنب | ✅ افعل |
|--------|--------|
| لا تضع `$id` في Create/Update | ضع `$id` فقط في Response DTO |
| لا تكرر الكود | اعمل traits أو BaseDTO |
| الـ relationships في Response فقط | في Request: IDs فقط |

---

## 🎯 **الترتيب المقترح للتنفيذ**

1. **Base DTO** ← ابدأ هنا
2. **User DTOs** ← أساسي
3. **Employer DTOs** ← يعتمد على User
4. **Job DTOs** ← يعتمد على Employer
5. **Application DTOs** ← يعتمد على Job
6. **Candidate DTOs** ← يعتمد على User
7. **Skill DTOs** ← مستقل

---

هل تريد أبدأ الـ implementation معك؟ أو تريد شرح أعمق في أي نقطة؟