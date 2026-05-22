# 04 — Repository Pattern في NestJS + TypeORM

# مقدمة

قبل ما نفهم Repository Pattern لازم نسأل:

> ليه أصلًا محتاج Repository ؟

لأن التطبيق الكبير فيه طبقات مختلفة، وكل طبقة لها مسؤولية محددة.

---

# تخيل السيناريو ده

عندك API:

```txt
GET /users
```

الـ Request تمشي كدا:

```txt
Controller
      ↓
Service
      ↓
Repository
      ↓
Database
```

---

# وظيفة كل Layer

|Layer|مسؤولة عن ايه|
|---|---|
|Controller|استقبال الـ Request وإرجاع Response|
|Service|Business Logic|
|Repository|التعامل مع Database|
|Database|تخزين البيانات|

---

# ❌ الشكل الغلط

ناس كتير تعمل:

```ts
@Get()
async getUsers() {

   return await this.userRepository.find();
}
```

أو أسوأ:

```ts
@Get()
async getUsers() {

   return await this.dataSource.query(
      "SELECT * FROM users"
   );
}
```

---

# المشكلة ؟

الـ Controller بقى:

- يعرف Database
    
- يعرف Queries
    
- يعرف Business Logic
    

وده يخالف:

```txt
Single Responsibility Principle
```

---

# الشكل الصح

```txt
Controller
    ↓
Service
    ↓
Repository
```

---

# Repository Pattern يعني اي ؟

> طبقة مسؤولة عن الوصول للبيانات (Data Access Layer).

هي اللي تعمل:

- Find
    
- Save
    
- Update
    
- Delete
    
- Custom Queries
    

---

# مثال عملي Production Style

نفترض عندنا:

```txt
Users Module
```

---

# user.entity.ts

```ts
@Entity('users')
export class User {

   @PrimaryGeneratedColumn()
   id:number;

   @Column()
   fullName:string;

   @Column({
      unique:true
   })
   email:string;
}
```

---

# استخدام @InjectRepository

في NestJS + TypeORM نستخدم:

```ts
@InjectRepository()
```

عشان Nest تعمل Dependency Injection للـ Repository.

---

# users.service.ts

```ts
@Injectable()
export class UsersService {

    constructor(

        @InjectRepository(User)
        private userRepository:
            Repository<User>

    ) {}

}
```

---

# ايه اللي حصل هنا ؟

```ts
@InjectRepository(User)
```

بتقول لـ Nest:

> "هات Repository الخاصة بالـ User Entity"

---

# Repository

ده Generic Type.

معناه:

```txt
Repository متخصصة للـ User
```

---

# دلوقتي نقدر نستخدم Methods جاهزة

---

# إنشاء User

```ts
async create(data: CreateUserDto){

   const user =
      this.userRepository.create(data);

   return await this.userRepository.save(
      user
   );
}
```

---

# ليه create وبعدها save ؟

---

# create()

تعمل:

```txt
Create object in memory only
```

بدون Database.

---

# save()

تعمل:

```txt
INSERT INTO database
```

---

# جلب كل المستخدمين

```ts
async findAll(){

   return this.userRepository.find();
}
```

---

# جلب User واحد

```ts
async findOne(id:number){

   return await this.userRepository.findOne({

      where:{
         id
      }

   });
}
```

---

# تحديث User

```ts
async update(

   id:number,
   dto:UpdateUserDto

){

   await this.userRepository.update(
      id,
      dto
   );

   return this.findOne(id);

}
```

---

# حذف User

```ts
async remove(id:number){

   return await this.userRepository.softDelete(
      id
   );

}
```

---

# ليه استخدمنا softDelete ؟

لأننا عملنا:

```ts
@DeleteDateColumn()
```

في الـ Entity.

---

# مشكلة تظهر في المشاريع الكبيرة ⚠️

مع الوقت Service تبدأ تكبر جدًا.

مثال:

```ts
async findActiveUsers(){

}

async findAdmins(){

}

async findUsersByRole(){

}

async findRecentUsers(){

}

async searchUsers(){

}
```

بعد شوية:

```txt
UsersService = 1500 سطر 😵
```

---

# الحل

Custom Repository

---

# يعني اي Custom Repository ؟

Repository خاصة تحتوي على Database Logic مخصص.

---

# users.repository.ts

```ts
@Injectable()
export class UsersRepository {

   constructor(

      @InjectRepository(User)
      private repository:
         Repository<User>

   ){}

   async findAdmins(){

      return this.repository.find({

         where:{
            role:UserRole.ADMIN
         }

      });

   }

   async findActiveUsers(){

      return this.repository.find({

         where:{
            isActive:true
         }

      });

   }

}
```

---

# Service تصبح أنظف

```ts
@Injectable()
export class UsersService {

   constructor(

      private usersRepository:
         UsersRepository

   ){}

   async getAdmins(){

      return await this
      .usersRepository
      .findAdmins();

   }

}
```

---

# الشكل النهائي

```txt
Controller
     ↓
Service
     ↓
Custom Repository
     ↓
TypeORM Repository
     ↓
Database
```

---

# ليه Custom Repository مهمة ؟

لأنها:

- تقلل تكرار الكود
    
- تسهل Testing
    
- تجعل Service نظيفة
    
- تفصل Business Logic عن Data Access Logic
    

---

# Database Logic vs Business Logic

دي نقطة ناس كتير تلخبط فيها.

---

# Database Logic

```ts
find()

save()

update()

joins()

queries()
```

---

# Business Logic

```ts
لو المستخدم Admin
ابعت Notification

لو المنتج انتهى
غير الحالة
```

---

# ❌ غلط

```ts
async create(dto){

   const user =
      await this.userRepository.save(
         dto
      );

   sendEmail();

   createWallet();

   generateOtp();

}
```

---

# ليه ؟

الكود بدأ يخلط المسؤوليات.

---

# الأفضل

```ts
async create(dto){

   const user =
      await this.userRepository
      .createUser(dto);

   await this.walletService
      .create(user.id);

   await this.emailService
      .send();

}
```

---

# Production Best Practices ⭐

### 1) لا تضع DB logic في Controller

❌

```ts
@Get()
return this.userRepository.find()
```

---

### 2) استخدم Service كـ Business Layer

---

### 3) لو الـ Queries بدأت تكبر اعمل Custom Repository

---

### 4) لا تكتب SQL إلا عند الحاجة

استخدم:

```ts
QueryBuilder
```

للحالات المعقدة.

---

### 5) Repository مسؤولة عن البيانات فقط

لا تضع:

- Email logic
    
- Notifications
    
- Authentication
    

داخلها.

---

# مثال Production كامل

```txt
Request
   ↓
Controller
   ↓
UsersService
   ↓
UsersRepository
   ↓
Repository<User>
   ↓
PostgreSQL
```

---

# 🧠 Mental Model مهم

تخيل Repository كأنها موظف أرشيف:

أنت لا تدخل المخزن بنفسك.

أنت تقول له:

```txt
هات المستخدم رقم 5
```

وهو يجيب البيانات من المخزن.

---

# 🚀 الخلاصة

## Repository Pattern

هي طبقة تفصل الوصول للبيانات عن باقي التطبيق.

---

# `@InjectRepository`

تستخدم لحقن Repository الخاصة بـ Entity.

---

# Custom Repository

تستخدم عندما يكون عندك:

- Queries كثيرة
    
- Logic للوصول للبيانات
    
- إعادة استخدام للكود
    

---

# أهم قاعدة

> Controller يستقبل  
> Service تفكر  
> Repository تتعامل مع Database