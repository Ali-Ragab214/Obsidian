


هنمشي بالترتيب المنطقي:

1. VPC
    
2. CIDR
    
3. Subnets
    
4. Route Tables
    
5. Internet Gateway
    
6. NAT Gateway
    
7. EC2
    
8. Security Groups
    
9. NACL
    
10. الفلو الكامل من أول ما يوزر يخش لحد ما يوصل للسيرفر
    

---

# 🧠 أولاً: VPC (Virtual Private Cloud)

## 🟢 Amazon VPC

VPC = شبكة خاصة بيك جوه AWS  
تخيلها زي Compound كبير انت صاحبه.

جوه الـ VPC انت اللي بتحدد:

- العناوين (IP range)
    
- تقسيم الشبكة
    
- مين يدخل ومين يخرج
    
- ازاي السيرفرات تكلم بعض
    

---

# 📦 الشكل العام للـ VPC

```
                 🌍 Internet
                     |
               [ Internet Gateway ]
                     |
         ---------------------------------
         |             VPC               |
         |        10.0.0.0/16            |
         |                                 
         |   -----------    -----------   |
         |   | Public  |    | Private |   |
         |   | Subnet  |    | Subnet  |   |
         |   -----------    -----------   |
         ---------------------------------
```

---

# 🧮 CIDR Block (مهم جداً)

CIDR = حجم الشبكة

مثال:

```
10.0.0.0/16
```

يعني:

- عندك من 10.0.0.0
    
- لحد 10.0.255.255
    
- = 65,536 IP
    

لو عملت:

```
10.0.1.0/24
```

يبقى عندك 256 IP

📌 كل VPC لازم يكون له CIDR  
📌 كل Subnet لازم تكون جزء من CIDR بتاع الـ VPC

---

# 🏘 Subnets (تقسيم الشبكة)

الـ Subnet = جزء من الـ VPC

نوعين مهمين:

## 🟢 Public Subnet

- ليها Route للـ Internet
    
- فيها Web Server
    
- يقدر الناس تدخل عليه
    

## 🔵 Private Subnet

- مفيش Route مباشر للـ Internet
    
- فيها Database / Backend
    
- محدش يقدر يدخلها من بره
    

---

## 🎯 شكل أوضح:

```
VPC 10.0.0.0/16
│
├── Public Subnet (10.0.1.0/24)
│     └── EC2 Web Server
│
└── Private Subnet (10.0.2.0/24)
      └── EC2 Database
```

---

# 🌍 Internet Gateway (IGW)

## 🟢 Internet Gateway

ده الباب اللي بيخلي VPC تكلم الإنترنت.

⚠️ مهم:

- بيتربط بالـ VPC
    
- بس مش كفاية لوحده
    
- لازم Route Table تشير له
    

---

# 🛣 Route Table

هي اللي بتقول:  
"لو عايز تروح لعنوان معين امشي منين"

مثال Route Table للـ Public Subnet:

|Destination|Target|
|---|---|
|10.0.0.0/16|local|
|0.0.0.0/0|IGW|

0.0.0.0/0 = أي إنترنت

---

## 📊 Diagram Route

```
Public Subnet
      |
Route Table
      |
0.0.0.0/0 → Internet Gateway
```

---

# 🔁 NAT Gateway

## 🟢 NAT Gateway

ده حل عبقري 🔥

بيخلي:

- السيرفرات في الـ Private Subnet
    
- تطلع على الإنترنت
    
- لكن محدش من الإنترنت يدخل عليهم
    

مثال:  
Database عايزة تنزل update  
→ تطلع عبر NAT  
→ بس محدش يقدر يدخلها

---

## الشكل:

```
Private EC2
      |
Route Table
      |
0.0.0.0/0 → NAT Gateway
      |
Public Subnet
      |
Internet Gateway
      |
Internet
```

---

# 🖥 EC2

## 🟢 Amazon EC2
# 💻 يعني ايه EC2 ؟

## 👉 EC2 = Elastic Compute Cloud

دي خدمة من AWS بتديك:

🖥️ **سيرفر (كمبيوتر) على الإنترنت**

بدل ما تشتري:

- جهاز
    
- هاردوير
    
- داتا سنتر
    

AWS بتقولك:

> خد سيرفر جاهز وادفع حسب الاستخدام

---

# 🧠 تخيل كده

بدل ما:

تشتري لابتوب وتشغل عليه:

- موقع
    
- API
    
- Database
    

AWS بتقولك:

"خد جهاز افتراضي عندي"

وده هو 👉 EC2

---

# 🧱 يعني ايه سيرفر افتراضي ؟

سيرفر حقيقي موجود في داتا سنتر  
لكن متقسم بـ Virtualization

فانت بتاخد جزء منه كأنه جهاز لوحده.

---

# 📦 ال EC2 يحتوي على ايه ؟

لما تعمل EC2 انت بتنشئ:

|حاجة|معناها|
|---|---|
|CPU|البروسيسور|
|RAM|الرام|
|Storage|الهارد|
|OS|لينكس او ويندوز|
|Network|IP داخل VPC|

يعني جهاز كامل.

---

# 🔧 بستخدمه في ايه كمهندس ؟

تشغل عليه:

✔ Backend API  
✔ موقع  
✔ Database  
✔ Microservices  
✔ Docker Containers  
✔ ML Models  
✔ أي برنامج

---

# 🌍 مكانه في الشبكة

ال EC2 بيتحط جوه:

👉 Subnet داخل VPC

زي كده:

```
VPC
 ├── Public Subnet
 │     └── EC2 (Web Server)
 │
 ├── Private Subnet
 │     └── EC2 (Database)
```

---

# 🟢 نوعين EC2 مهمين

## 1. Public EC2

عنده Public IP

→ ينفع الناس تدخل عليه من الإنترنت

زي:

Web Server

---

## 2. Private EC2

مفيش Public IP

→ محدش يدخل عليه من بره

زي:

Database

---

# 🔁 مثال حقيقي

User يفتح موقعك:

```
Internet
   ↓
Public EC2 (Web Server)
   ↓
Private EC2 (Database)
```

---

# ⚡ ليه اسمها Elastic ؟

يعني:

تقدر:

- تكبر السيرفر
    
- تصغره
    
- تزود عدد السيرفرات
    
- تقللهم
    

في ثواني.

---

# 🧠 الخلاصة

EC2 =

🖥️ سيرفر افتراضي في الكلاود  
بتشغل عليه أي حاجة  
وبيكون جوه VPC

---

لو حابب  
أشرح أنواع EC2 (t2 / m5 / c5) واختار ازاي 👌

ده السيرفر بتاعك.

ممكن تحطه:

- في Public Subnet (Web Server)
    
- في Private Subnet (DB Server)
    

---

# 🔐 Security Groups

ده Firewall على مستوى السيرفر نفسه.

📌 Stateful  
يعني لو دخل request بيرد تلقائي

مثال:

- Allow HTTP 80
    
- Allow SSH 22

حاضر 🔥  
خلينا نشرح **Security Groups** بطريقة تخليك تفهمها بجد مش تحفظها 👌

إنت شغال على **EC2** في AWS، فـ الموضوع ده مهم جدًا ليك.

---

# 🎯 يعني إيه Security Group؟

الـ Security Group في AWS هو:

> 🔥 Firewall للسيرفر بتاعك (EC2)

يعني هو اللي بيقرر:

- مين يدخل
    
- يدخل على أنهي بورت
    
- يدخل بإيه (HTTP؟ SSH؟ غيره؟)
    

---

# 🧠 تخيلها كده

```
الإنترنت 🌍
     |
     v
[ Security Group ]  ← الحارس
     |
     v
EC2 Server
```

لو الحارس مش سامح → مفيش حد هيخش  
حتى لو السيرفر شغال 100%

---

# 🧱 بتتكون من إيه؟

Security Group فيها نوعين قواعد:

## 1️⃣ Inbound Rules (الداخل)

دي بتحدد:

> مين يقدر يدخل على السيرفر؟

مثال:

|Type|Port|Source|
|---|---|---|
|SSH|22|0.0.0.0/0|
|HTTP|80|0.0.0.0/0|

يعني:

- أي حد من الإنترنت يقدر يدخل SSH
    
- وأي حد يقدر يفتح الموقع
    

---

## 2️⃣ Outbound Rules (الخارج)

دي بتحدد:

> السيرفر يقدر يكلم مين؟

غالبًا بتبقى:

```
All traffic → 0.0.0.0/0
```

يعني السيرفر يقدر يوصل لأي حاجة بره.

---

# 🎯 أهم 4 حاجات تفهمهم

## 🔹 1) Port

كل خدمة ليها رقم:

|الخدمة|البورت|
|---|---|
|SSH|22|
|HTTP|80|
|HTTPS|443|
|MySQL|3306|

لو البورت مش مفتوح → الخدمة مش هتشتغل من بره.

---

## 🔹 2) Source

مين مسموحله يدخل؟

مثلاً:

|Source|معناها|
|---|---|
|0.0.0.0/0|أي حد في العالم|
|192.168.1.5|IP واحد بس|
|Security Group ID|سيرفر تاني بس|

---

## 🔹 3) Stateful (مهم جدًا)

Security Groups:

> Stateful

يعني لو سمحت بدخول request  
الرد بيطلع تلقائي  
مش محتاج تعمل rule للرد.

دي نقطة قوية جدًا 👌

---

## 🔹 4) Default Behavior

لو مفيش Rule تسمح  
يبقى:

❌ Deny

AWS مش بيسمح بأي حاجة إلا لو قلتله.

---

# 🧠 مثال من اللي حصل معاك

السيرفر كان شغال:

```
Apache → Running ✔
```

لكن مكنتش فاتح Port 80

فالحالة كانت:

```
Browser
   ↓
Security Group ❌
   ↓
EC2
```

فالمتصفح قال:

```
Connection Timed Out
```

---

# 🎓 ليه ده مهم في الشغل؟

لأن 70% من مشاكل السيرفرات في الحقيقة بتكون:

- Security Group
    
- Firewall
    
- Network
    

مش في الكود 😄

---

# 🔥 الفرق بين Security Group و Firewall جوه السيرفر

|Security Group|Firewall (firewalld)|
|---|---|
|في AWS|جوه Linux|
|بيشتغل قبل ما الترافيك يوصل|بعد ما يوصل|
|Managed من AWS|Managed من OS|

---

# 🎯 مثال متقدم شوية

لو عندك:

- EC2 Web Server
    
- EC2 Database Server
    

تعمل كده:

Web SG:

```
Allow HTTP from 0.0.0.0/0
```

DB SG:

```
Allow MySQL from Web SG only
```

كده محدش يقدر يوصل للداتابيز غير السيرفر.

دي اسمها Architecture نظيفة 🔥

---

# 👌 الخلاصة

Security Group هو:

> بوابة التحكم في مين يدخل ومين يخرج من السيرفر

لو فاهمها كويس  
هتعدي 50% من مشاكل AWS بسهولة.

---

لو عايز  
أشرحلك الفرق بينها وبين NACL  
أو نرسم Network Architecture أكبر شوية  
قوللي ونكبر المستوى 😎🔥

---

# 🧱 NACL

Network ACL  
ده Firewall على مستوى الـ Subnet

📌 Stateless  
لازم تفتح inbound و outbound

---

## الفرق المهم 🔥

||Security Group|NACL|
|---|---|---|
|المستوى|EC2|Subnet|
|Stateful|✅|❌|
|السماح فقط|✅|❌ يقدر يعمل deny|

---

# 🔥 الفلو الكامل (من يوزر لحد الداتا بيز)

```
User
  |
Internet
  |
Internet Gateway
  |
Public Subnet
  |
EC2 Web Server
  |
Private Subnet
  |
EC2 Database
```

لو الداتا بيز عايزة إنترنت:

```
DB → NAT → IGW → Internet
```

---

# 🎯 Workflow عملي حقيقي

1️⃣ User يكتب domain  
2️⃣ الطلب يروح لـ Web Server في Public Subnet  
3️⃣ السيرفر يكلم Database في Private Subnet  
4️⃣ Database ترجع الداتا  
5️⃣ Web Server يرجع Response لليوزر

---

# 💎 دلوقتي الصورة الكبيرة بقت كده:

```
                         🌍 Internet
                              |
                      [ Internet Gateway ]
                              |
                    -----------------------
                    |         VPC          |
                    |    10.0.0.0/16       |
                    |                       |
          ----------                       ----------
          | Public |                       | Private |
          |Subnet  |                       | Subnet  |
          |10.0.1.0|                       |10.0.2.0 |
          |        |                       |         |
          | EC2    | ----> talks to ---->  |  EC2 DB |
          |  Web   |                       |         |
          ----------                       ----------
                  |
             NAT Gateway
                  |
               Internet
```

---

# 🧠 خلاصة تحفظها وانت داخل انترفيو

- VPC = شبكة خاصة
    
- CIDR = حجم الشبكة
    
- Subnet = تقسيم الشبكة
    
- IGW = دخول وخروج للإنترنت
    
- NAT = خروج بس من private
    
- Route Table = بتحدد الطريق
    
- Security Group = firewall على السيرفر
    
- NACL = firewall على الشبكة
    

---
حلو 👌  
هنا **صور ذهنية بسيطة** لكل جزء علشان الدنيا تثبت في دماغك بسرعة 🔥  
(كل صورة = مفهوم واحد)

---

# 🟦 1. شكل الـ VPC العام

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AyoIjlItsVh-8Erp5MJO5WA.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

![Image](https://miro.medium.com/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/public-nat-gateway-diagram.png)

📌 الفكرة:

- المستطيل الكبير = VPC
    
- جواه:
    
    - Public Subnet
        
    - Private Subnet
        

---

# 🟩 2. تقسيم الشبكة (CIDR → Subnets)

![Image](https://miro.medium.com/1%2A4C1MHY5mM6SPEEvS2cGpJw.png)

![Image](https://www.researchgate.net/publication/356012936/figure/fig1/AS%3A1088122196635648%401636439956319/P-Addressing-of-Cloud-Computing.jpg)

![Image](https://visualsubnetcalc.com/icon/social_1200x600.png)

![Image](https://miro.medium.com/1%2An2SHj5r-g3xa5O-f_FLfRA.png)

📌 الفكرة:

شبكة كبيرة

```
10.0.0.0/16
```

اتقسمت:

```
10.0.1.0/24  → Public
10.0.2.0/24  → Private
```

---

# 🌍 3. Internet Gateway

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Image](https://miro.medium.com/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Image](https://docs.aws.amazon.com/images/network-firewall/latest/developerguide/images/arch-igw-2az-simple.png)

![Image](https://images.ctfassets.net/6yom6slo28h2/4CBWQLqrumY2rU0Wc58PY7/b5af1e0facc2b7bfd15f5dc8468ecde9/vpc-flow-log-sources.png)

📌 الفكرة:

هو الباب بين:

🌐 الإنترنت  
🏠 الـ VPC

---

# 🟧 4. NAT Gateway (خروج فقط)

![Image](https://miro.medium.com/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/public-nat-gateway-diagram.png)

![Image](https://cms.cloudoptimo.com/uploads/Nat_Gateway_6f49c801d0.png)

![Image](https://guides.zadarastorage.com/cs-networking-guide/latest/_images/02-NAT-Gateway-Overview_Symphony-1024x797.png)

📌 الفكرة:

Private Server:

✔ يطلع إنترنت  
❌ محدش يدخل عليه

---

# 🧭 5. Route Table (المخ)

![Image](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2020/03/19/Slide1.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/outpost-intra-vpc-connection.png)

![Image](https://docs.oracle.com/en-us/iaas/Content/Network/Images/network_vcn_internal.svg)

![Image](https://docs.oracle.com/en-us/iaas/Content/Network/Images/network_source_routing.svg)

📌 الفكرة:

هي اللي بتقول:

الترافيك يروح فين؟

---

# 🔒 6. Security Layers

## NACL vs Security Group

![Image](https://cloudviz.io/assets/aws-security-group-vs-nacl/security-groups.png)

![Image](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2020/11/17/anfw-protected-vpc-high-res.png)

![Image](https://cheatsheetseries.owasp.org/assets/Secure_Cloud_Architecture_VPC.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/security-comparison.png)

📌 الفكرة:

|النوع|المكان|
|---|---|
|NACL|على الـ Subnet|
|Security Group|على الـ EC2|

---

# 🧱 7. EC2 داخل الشبكة

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

![Image](https://miro.medium.com/v2/da%3Atrue/resize%3Afit%3A1200/0%2AnaAV6WjdHzYuFc3-)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-web-database.png)

![Image](https://i.stack.imgur.com/nO12d.png)

📌 الفكرة:

- Web Server → Public
    
- Database → Private
    

---

# 🔁 8. الفلو الكامل

![Image](https://www.dclessons.com/uploads/blog-editor/mceu_21702288511648570325222.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/public-nat-gateway-diagram.png)

![Image](https://miro.medium.com/1%2AULcBRha4tGrCD_s4QfUjhw.jpeg)

![Image](https://cdn.prod.website-files.com/681e366f54a6e3ce87159ca4/6877c55a545e0d99d09ccb6e_Blog_Images-DNSintheCloud-diagram03.png)

📌 السيناريو:

User يدخل الموقع:

```
Internet
   ↓
IGW
   ↓
Public EC2
   ↓
Private DB
```

---

# 🧠 أسرع ملخص بصري

```
VPC
 ├── Public Subnet
 │     ├── EC2 (Web)
 │     └── NAT Gateway
 │
 ├── Private Subnet
 │     └── EC2 (DB)
 │
 ├── Route Tables
 ├── NACL
 └── Security Groups
```

---

لو تحب 👌  
أعمل لك:

🧪 سيناريو Deployment عملي  
"ازاي نبني ده خطوة خطوة" في AWS

قول: نطبّق 💻🔥