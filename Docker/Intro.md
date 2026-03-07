تقدر تحطه في **Obsidian** بشكل منظم باستخدام Markdown بحيث يبقى سهل المراجعة. دي صياغة أنضف ومرتبة:

---

# Hypervisor

الـ **Hypervisor** هو برنامج أو طبقة برمجية مسؤولة عن تشغيل أكثر من **Operating System** على نفس الجهاز في نفس الوقت عن طريق إنشاء **Virtual Machines (VMs)**.

بمعنى أبسط:

الـ Hypervisor يقوم بتقسيم موارد الجهاز الحقيقي مثل:

- **CPU**
    
- **RAM**
    
- **Storage**
    

ثم يعطي لكل **Virtual Machine** جزءًا من هذه الموارد.

---

## الفكرة الأساسية

لو لديك جهاز واحد يحتوي على:

- CPU
    
- RAM
    
- Hard Disk
    

يمكن للـ **Hypervisor** إنشاء عدة أجهزة افتراضية مثل:

- VM تعمل بنظام **Linux**
    
- VM تعمل بنظام **Windows**
    
- VM تعمل بنظام **Ubuntu**
    

كل **VM** تعمل وكأنها جهاز كمبيوتر مستقل.

---

## مثال عملي

إذا كان جهازك يعمل بنظام **Ubuntu** وتريد تجربة **Windows Server** بدون حذف النظام الأساسي.

بدل استخدام **Dual Boot** يمكنك استخدام Hypervisor مثل:

- **VMware**
    
- **VirtualBox**
    
- **Microsoft Hyper-V**
    

يقوم البرنامج بإنشاء **Virtual Machine** ويمكنك تثبيت **Windows Server** عليها وتشغيله بجانب Ubuntu.

---

# أنواع الـ Hypervisor

## 1️⃣ Type 1 (Bare Metal Hypervisor)

يعمل مباشرة فوق **Hardware** بدون وجود نظام تشغيل وسيط.

### الهيكل

```
Hardware
   ↓
Hypervisor
   ↓
Virtual Machines
   ↓
Operating Systems
```

### أمثلة

- VMware ESXi
    
- Microsoft Hyper-V
    
- Xen
    

### الاستخدام

يستخدم غالبًا في:

- **Data Centers**
    
- **Cloud Platforms**
    

---

## 2️⃣ Type 2 (Hosted Hypervisor)

يعمل فوق **Operating System** عادي.

### الهيكل

```
Hardware
   ↓
Operating System
   ↓
Hypervisor
   ↓
Virtual Machines
```

### أمثلة

- Oracle VM VirtualBox
    
- VMware Workstation
    

### الاستخدام

يستخدم غالبًا على:

- **Laptops**
    
- **Personal Computers**
    

---

# علاقة الـ Hypervisor بالـ Cloud

في خدمات مثل:

- **Amazon Web Services (AWS)**
    
- **Microsoft Azure**
    

عندما تطلب:

- **EC2 Instance**
    
- **Virtual Machine**
    

فإن الذي يقوم بتشغيل هذه الـ VM في الخلفية هو **Hypervisor**.

---

# الفرق بين Hypervisor و Docker

هذه نقطة مهمة خصوصًا لمن يتعلم **Docker**.

|Hypervisor|Docker|
|---|---|
|يشغل **Operating System كامل**|يشغل **Containers**|
|كل VM لها **OS خاص بها**|جميع Containers تشارك نفس **Kernel**|
|يستهلك موارد أكثر|أخف وأسرع|
|Boot أبطأ|Start سريع|

### التوضيح

**Hypervisor**

```
Hardware
↓
Hypervisor
↓
VM
↓
Operating System
↓
Application
```

**Docker**

```
Hardware
↓
Host OS
↓
Docker Engine
↓
Containers
↓
Applications
```

---

# مثال عملي في Docker

عند كتابة:

```bash
docker pull ubuntu
docker run -it ubuntu
```

أنت لا تقوم بتشغيل **Ubuntu Operating System كامل** كما يحدث في الـ **Virtual Machine**.

بل تقوم بتشغيل **Container** يحتوي فقط على:

- Libraries
    
- Dependencies
    
- Shell
    

وهو يعتمد على **Kernel الخاص بالنظام الأساسي**.

---
باختصار جدًا:

Namespaces
→ مسؤولة عن العزل (Isolation)
يعني كل Container يشوف نفسه لوحده (processes، network، filesystem) كأنه نظام مستقل.

Cgroups (Control Groups)
→ مسؤولة عن تحديد الموارد (Resource Limits)
يعني تحدد للـ Container قد إيه يستخدم من CPU و RAM.

✅ الخلاصة:

Namespaces → مين يشوف إيه

Cgroups → مين يستهلك قد إيه من الموارد 


Seccomp
→ أداة أمان في Linux بتحدد إيه الـ system calls اللي الـ container يقدر يستخدمها.
يعني تمنع العمليات الخطيرة على النظام.

SELinux (Security-Enhanced Linux)
→ نظام أمان بيطبق سياسات صلاحيات صارمة على البرامج والملفات، بحيث كل برنامج يقدر يوصل لحاجات محددة بس.

✅ الخلاصة باختصار:

Seccomp → يحدد الـ system calls المسموحة

SELinux → يحدد مين يقدر يوصل لإيه في النظام