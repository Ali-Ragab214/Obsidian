
---

# 1️⃣ ما هو Git ولماذا نستخدمه؟

## تعريف Git

**Git** هو نظام لإدارة الإصدارات يسمى:

**Version Control System**

وظيفته:

- تتبع التعديلات على الكود
    
- إدارة نسخ المشروع
    
- تسهيل العمل الجماعي
    

---

## المشكلة بدون Git

لو كنت تعمل بدون Git سيحدث:

```
project
project_final
project_final2
project_last
project_last_really
```

لا تعرف:

- ما الذي تغير
    
- من الذي غيره
    
- متى تغير
    

---

## كيف يحل Git المشكلة

Git يسجل كل تغيير في المشروع في شيء يسمى:

```
Commit
```

مثال:

```
Commit 1 -> Create project
Commit 2 -> Add login API
Commit 3 -> Fix bug in login
Commit 4 -> Add JWT authentication
```

---

# 2️⃣ ما هو GitHub؟

## GitHub:platform that host git projects

هو منصة تستضيف مشاريع Git على الإنترنت.

يعني:

```
Git = النظام
GitHub = السيرفر
```

أمثلة مشابهة:

- GitHub
    
- GitLab
    
- Bitbucket
    

---

## لماذا GitHub مهم

1️⃣ تخزين المشروع على الإنترنت  
2️⃣ التعاون مع الفريق  
3️⃣ Pull Requests  
4️⃣ Code Review  
5️⃣ حماية الكود  
6️⃣ CI/CD

---

# 3️⃣ أهم مصطلحات Git

## Repository (Repo)

هو المشروع.

مثال:

```
ServicesApp
```

---

## Commit

هو حفظ التعديلات.

مثال:

```
Add authentication system
```

---

## Branch

نسخة منفصلة من المشروع.

مثال:

```
main
feature-auth
feature-payment
```

---

## Merge 

دمج فرعين.

---

# الفكرة الأساسية للـ Branch

الـ **Branch** هو نسخة منفصلة من المشروع تعمل عليها بدون أن تلمس الكود الأساسي.

الكود الأساسي عادة يكون في:

```
main
```

وهو الكود المستقر.

---

# لماذا نستخدم Branch؟

لأنك عندما تطور Feature جديدة قد:

- تكسر النظام
    
- تجرب أشياء
    
- تعمل Debug
    
- تعدل كثير
    

ولا نريد أن يتأثر **main** بكل هذه التجارب.

---

# السيناريو الحقيقي

لنفترض أن المشروع اسمه:

```
ServicesApp
```

وفيه branch رئيسي:

```
main
```

الكود فيه يعمل بشكل صحيح.

---

# أنت تريد إضافة Feature جديدة

مثلاً:

```
Payment System
```

فتنشئ Branch:

```
feature-payment
```

الأمر:

```
git checkout -b feature-payment
```

---

# الآن ماذا يحدث؟

تصبح تعمل هنا:

```
feature-payment
```

وليس:

```
main
```

---

# هنا يمكنك أن "تخبط براحتك"

يمكنك:

- تجربة كود
    
- تعديل
    
- حذف
    
- إعادة كتابة
    

لأن هذا **لن يؤثر على main**.

---

# مثال

```
main
   |
   |---- feature-payment
```

أنت تعمل على:

```
feature-payment
```

---

# بعد أن تنتهي من الكود

عندما يصبح الكود:

- يعمل
    
- تم اختباره
    
- نظيف
    

تقوم بدمجه مع main.

---

# الدمج يسمى

```
Merge
```

الأمر:

```
git checkout main
git merge feature-payment
```

---

# الشكل يصبح

```
main
   |
   |------ feature-payment
           |
           merge
```

---

# في الشركات لا يتم الدمج مباشرة

غالبًا يحدث هذا:

```
Push branch
↓
Create Pull Request
↓
Code Review
↓
Merge
```

---

# السيناريو الواقعي

1️⃣ تعمل branch

```
git checkout -b feature-auth
```

2️⃣ تكتب الكود

3️⃣ تعمل commits

```
git commit -m "Add JWT authentication"
```

4️⃣ ترفع الفرع

```
git push origin feature-auth
```

5️⃣ تفتح Pull Request على GitHub

6️⃣ الفريق يراجع الكود

7️⃣ يتم الدمج في main

---

# لماذا هذا مهم؟

لأن main يجب أن يكون دائمًا:

```
Stable
```

يعني:

```
Deployable
```

يمكن نشره على السيرفر في أي وقت.

---

# ماذا يحدث بعد الدمج؟

غالبًا نحذف branch.

```
git branch -d feature-auth
```

---

# الشكل النهائي للمشروع

```
main
 ├── feature-login (merged)
 ├── feature-payment (merged)
 └── feature-search (merged)
```

---

# قاعدة مهمة في Git

❌ لا تعمل مباشرة على main

بل:

```
Feature Branch
```

ثم:

```
Merge
```

---

# مثال بسيط يوضح الفكرة

تخيل المشروع مثل كتاب.

```
main = النسخة الأصلية
```

وأنت تريد تجربة فصل جديد.

فتأخذ نسخة:

```
feature-chapter
```

تكتب فيها.

لو كانت جيدة:

نضيفها للكتاب.

---

# الخلاصة

نعم كلامك صحيح:

✔ تعمل على Branch  
✔ تجرب وتعدل فيه  
✔ عندما يصبح جاهز  
✔ تعمل Merge مع main

لكن في الشركات غالبًا يتم عبر:

```
Pull Request
```

وليس Merge مباشر.


---

## Remote

نسخة المشروع على الإنترنت.

---

# 4️⃣ كيف يعمل Git داخليًا

Git يحتوي على 4 مراحل:

```
Working Directory
        ↓
Staging Area
        ↓
Local Repository
        ↓
Remote Repository
```

---

## Working Directory

مكان الكود الذي تعدله.

---

## Staging Area

منطقة تجهيز الملفات قبل commit.

---

## Local Repository

مكان حفظ الكوميتات.

---

## Remote Repository

المشروع على GitHub.

---

# 5️⃣ تثبيت Git

في Ubuntu:

```
sudo apt install git
```

تحقق:

```
git --version
```

---

# 6️⃣ إعداد Git لأول مرة

```
git config --global user.name "Ali Ragab"
git config --global user.email "ali@email.com"
```

عرض الإعدادات:

```
git config --list
```

---

# 7️⃣ إنشاء Repository

## إنشاء مشروع جديد

```
git init
```

سيتم إنشاء فايل :

```
.git
```

وهو يحتوي كل تاريخ المشروع.

---

## استنساخ مشروع

```
git clone repo_url
```

مثال:

```
git clone https://github.com/company/project.git
```

---

# 8️⃣ أهم أوامر Git الأساسية

## معرفة حالة المشروع

```
git status
```

---

## إضافة ملفات للـ staging

إضافة ملف:

```
git add file.cs
```

إضافة كل الملفات:

```
git add .
```

---

## إنشاء commit

```
git commit -m "Add booking endpoint"
```

---

## عرض التاريخ

```
git log
```

أو:

```
git log --oneline
```

---

# 9️⃣ رفع المشروع على GitHub

ربط المشروع:

```
git remote add origin repo_url
```

عرض الريموت:

```
git remote -v
```

---

رفع الكود:

```
git push origin main
```

---

# 🔟 سحب التعديلات

```
git pull origin main
```

---

# 11️⃣ الفرق بين pull و fetch

## Pull

يسحب التعديلات ويعمل merge.

```
git pull
```

---

## Fetch

يسحب فقط.

```
git fetch
```

---

# 12️⃣ العمل بالـ Branches

البرانشات أهم ميزة في Git.

---

## عرض البرانشات

```
git branch
```

---

## إنشاء branch

```
git branch feature-auth
```

---

## الانتقال لبرانش

```
git checkout feature-auth
```

أو

```
git switch feature-auth
```

---

## إنشاء والانتقال مباشرة

```
git checkout -b feature-auth
```

---

# 13️⃣ دمج الفروع

انتقل إلى main:

```
git checkout main
```

ثم:

```
git merge feature-auth
```

---

# 14️⃣ حذف branch

محلي:

```
git branch -d feature-auth
```

على GitHub:

```
git push origin --delete feature-auth
```

---

# 15️⃣ Pull Requests

ميزة في GitHub.

Workflow:

```
Developer
     ↓
Create Branch
     ↓
Push Branch
     ↓
Open Pull Request
     ↓
Code Review
     ↓
Merge
```

---

# 16️⃣ Code Review

قبل دمج الكود يقوم الفريق بمراجعته.

يتم فحص:

- جودة الكود
    
- الأمان
    
- الأداء
    

---

# 17️⃣ كيف نحمي الكود في الشركات

الشركات تحمي الكود بعدة طرق:

---

## 1️⃣ Branch Protection

لا يمكن push مباشرة إلى:

```
main
```

يجب:

```
Pull Request
Review
Approval
```

---

## 2️⃣ Code Review Required

يجب موافقة:

```
1 أو 2 Developers
```

قبل الدمج.

---

## 3️⃣ CI/CD Checks

قبل الدمج يتم تشغيل:

```
Tests
Build
Linting
Security Scan
```

---

## 4️⃣ Permissions

GitHub يسمح بتحديد صلاحيات:

```
Admin
Maintainer
Developer
Viewer
```

---

# 18️⃣ Merge Conflict

يحدث عندما يغير شخصان نفس السطر.

مثال:

```
<<<<<<< HEAD
new code
=======
old code
>>>>>>> feature-auth
```

الحل:

عدل الملف ثم:

```
git add .
git commit
```

---

# 19️⃣ Git Stash

يحفظ التعديلات مؤقتًا.

حفظ:

```
git stash
```

عرض:

```
git stash list
```

استرجاع:

```
git stash pop
```

---

# 20️⃣ Git Reset

للتراجع.

إلغاء commit مع الحفاظ على التعديلات:

```
git reset --soft HEAD~1
```

إلغاء كل شيء:

```
git reset --hard HEAD~1
```

---

# 21️⃣ Git Rebase

بديل لـ merge.

يجعل التاريخ نظيفًا.

```
git rebase main
```

---

# 22️⃣ Git Tags

لإنشاء إصدارات.

مثال:

```
v1.0
v2.0
```

إنشاء:

```
git tag v1.0
```

رفع:

```
git push origin v1.0
```

---

# 23️⃣ Git Ignore

ملف:

```
.gitignore
```

مثال:

```
node_modules/
bin/
obj/
.env
```

---

# 24️⃣ Git Workflow في الشركات

الأكثر شيوعًا:

---

## GitHub Flow

```
main
feature branches
pull request
review
merge
deploy
```

---

## مثال عملي

1️⃣ إنشاء branch

```
git checkout -b feature-payment
```

2️⃣ كتابة الكود

3️⃣ commit

```
git commit -m "Add payment API"
```

4️⃣ push

```
git push origin feature-payment
```

5️⃣ إنشاء Pull Request

6️⃣ مراجعة الكود

7️⃣ دمج في main

---

# 25️⃣ أوامر Git المهمة

```
git init
git clone
git status
git add
git commit
git push
git pull
git fetch
git branch
git checkout
git switch
git merge
git rebase
git stash
git reset
git log
git tag
git remote
git diff
```

---

# 26️⃣ مشاكل Git الشائعة

## push rejected

الحل:

```
git pull origin main
```

---

## merge conflict

الحل:

تعديل الملف ثم:

```
git add .
git commit
```

---

## detached HEAD

الحل:

```
git checkout main
```

---

# 27️⃣ أهم مهارات Git للمطورين

المطلوب منك في الشركات:

- Branching
    
- Pull Requests
    
- Conflict Resolution
    
- Rebase
    
- Git Workflow
    

---

💡 بما أنك Backend Developer وتعمل على مشاريع مثل **Clean Architecture و ASP.NET Core**، فهناك **Git Workflow خاص بالمشاريع الكبيرة** يستخدمه معظم الشركات.

إذا أردت، سأشرح لك أيضًا:

- **Git Workflow الحقيقي في الشركات خطوة بخطوة**
    
- **سيناريو كامل لفريق 5 Developers يعملون على نفس المشروع**
    
- **أخطر أوامر Git التي قد تمسح المشروع وكيف تتجنبها**
    
- **40 مشكلة Git تحدث في الشغل وحلولها**.