تمام 👌 هنكتب نسخة **أقوى + أعمق + أمثلة حقيقية + Repository + DI فعلي**  
وجاهزة copy-paste في Obsidian 💪

انسخه كما هو 👇

---

# 🟩 NestJS Providers — Deep Guide

---

# 1️⃣ ما هو الـ Provider فعلاً ؟

في Nest:

> Provider = أي Class يحتوي **Business Logic** ويمكن حقنه داخل Class آخر.

أهم نقطة 👇  
Nest مبني بالكامل على **Dependency Injection**.

يعني:

- الـ Controller لا ينشئ Service
    
- الـ Service لا ينشئ Repository
    

Nest هو من ينشئ ويربط كل شيء 🔥

---

# 🧠 الصورة الكبيرة للمعمارية

```
Request → Controller → Service → Repository → Database
```

|Layer|مسؤوليته|
|---|---|
|Controller|استقبال الطلب|
|Service|Business Logic|
|Repository|التعامل مع DB|

كل هذه الطبقات تعتبر Providers.

---

# 2️⃣ لماذا نحتاج Providers أصلاً؟

تخيل بدون DI 👇

```ts
const repo = new UserRepository()
const service = new UserService(repo)
const controller = new UserController(service)
```

😵‍💫 Chaos + Tight Coupling

Nest يحل هذا كله:

```ts
constructor(private service: UserService) {}
```

Nest يقوم بالباقي.

هذا يسمى:

```
IoC Container
```

## IoC Container

---

### IoC — الفكرة بالعربي

**IoC = Inversion of Control** — عكس التحكم.

الاسم غريب، بس الفكرة بسيطة جداً:

**بدون IoC — أنت بتتحكم في كل حاجة:**

```
أنت → بتنشئ الـ dependencies → بتحطها في بعض → بتشغّل الكود
```

**مع IoC — أنت بتقول "أنا محتاج كذا" وبس:**

```
أنت → بتطلب اللي محتاجه → الـ Container بيجيبه ليك
```

التحكم اتعكس — مش أنت اللي بتنشئ، الـ framework هو اللي بينشئ.

---

### الـ IoC Container عمله إيه بالظبط؟

هو ببساطة **سجل** — بيعرف كل الـ classes الموجودة في الـ app، إيه dependencies كل واحد محتاجها، وإزاي ينشئه.

```
┌─────────────────────────────────────┐
│           IoC Container             │
│                                     │
│  UserRepository  →  knows how       │
│  UserService     →  to create       │
│  UserController  →  each one        │
│                                     │
└─────────────────────────────────────┘
```

لما الـ app تبدأ، هو بيعمل كل ده تلقائياً:

```
1. شاف إن UserController محتاج UserService
2. شاف إن UserService محتاج UserRepository
3. أنشأ UserRepository أول
4. حطّه في UserService وأنشأه
5. حطّ الاتنين في UserController وأنشأه
```

كل ده من غير ما تكتب سطر واحد.

---

### إزاي Nest بيعرف الـ Dependencies؟

عن طريق **TypeScript Metadata** — لما بتكتب:

```typescript
@Injectable()
export class UserService {
  constructor(private repo: UserRepository) {}
}
```

الـ `@Injectable()` decorator بيقول لـ Nest: "اقرأ الـ constructor بتاع الكلاس ده وشوف هو محتاج إيه."

Nest بيقرأ إن الـ parameter الأول نوعه `UserRepository` — وبيروح يجيبه من الـ Container.

---

### الـ Container دورة حياته

```
app بتبدأ
     │
     ▼
Nest بيقرأ كل الـ Modules
     │
     ▼
بيسجّل كل الـ Providers في الـ Container
     │
     ▼
بيحل الـ Dependencies بالترتيب الصح
     │
     ▼
app جاهزة — كل الـ instances موجودة
     │
     ▼
Request جه → Controller موجود جاهز → بيرد
```

---

### Singleton داخل الـ Container

نقطة مهمة — الـ Container مش بينشئ instance جديد مع كل request. بينشئ **واحد بس** ويفضل يستخدمه.

```typescript
// UserService اتنشأ مرة واحدة بس في عمر الـ app
@Injectable()
export class UserService {}

// كل الـ controllers دي بتاخد نفس الـ instance
class OrdersController {
  constructor(private userService: UserService) {}
}

class ProductsController {
  constructor(private userService: UserService) {}
}

class AdminController {
  constructor(private userService: UserService) {}
}
// الثلاثة بياخدوا نفس الـ object — مش 3 نسخ
```

ده بيوفر memory وبيخلي الـ app أسرع.

---

### المقارنة الكاملة

**بدون IoC:**

```typescript
// لازم تعمل ده بإيدك في كل مكان
const logger = new Logger();
const config = new ConfigService();
const repo = new UserRepository(config, logger);
const mailer = new MailService(config);
const service = new UserService(repo, mailer, logger);
const controller = new UserController(service, logger);

// لو أي حاجة اتغيرت → تغيّر في كل مكان
```

**مع IoC في Nest:**

```typescript
// بس كده — Nest بيعمل الباقي
constructor(private service: UserService) {}
```

---

### ليه ده مهم في الـ Testing؟

أهم فايدة عملية للـ IoC Container إنك تقدر تستبدل أي dependency بـ mock بسهولة:

```typescript
// في الـ production — بيستخدم الـ real service
// في الـ test — بتستبدله بـ mock من غير ما تغير أي كود في الـ controller

const moduleRef = await Test.createTestingModule({
  providers: [
    UserController,
    {
      provide: UserService,
      useValue: {
        findAll: jest.fn().mockReturnValue([{ name: 'Ahmed' }]),
      },
    },
  ],
}).compile();
```

الـ controller مش عارف ومش مهتم — هو بس بيستخدم اللي الـ Container بيجيبه.

---

### الخلاصة

|بدون IoC|مع IoC Container|
|---|---|
|أنت بتنشئ كل حاجة|Nest بينشئ كل حاجة|
|التغيير في مكان = تغيير في كل مكان|التغيير في مكان واحد بس|
|Testing صعب|Testing سهل جداً|
|Tight Coupling|Loose Coupling|

IoC Container هو قلب NestJS — كل الـ `@Injectable()` و `@Module()` هي بس طريقتك إنك تقوله "سجّل ده" و"وفّرهولي هنا."


---

# 3️⃣ إنشاء Provider حقيقي (Service)

سنصنع نظام Users حقيقي.

## 📁 users.service.ts

```ts
import { Injectable } from '@nestjs/common';
import { UsersRepository } from './users.repository';

@Injectable()
export class UsersService {
  constructor(private usersRepo: UsersRepository) {}

  getAllUsers() {
    return this.usersRepo.findAll();
  }

  createUser(name: string) {
    return this.usersRepo.create(name);
  }
}
```

### ماذا حدث هنا؟

Service يعتمد على Repository.  
لكن لم ننشئ Repository باستخدام new ❌

Nest سيحقنه تلقائياً.

---

# 4️⃣ إنشاء Repository Provider

Repository مسؤول عن DB فقط.

## 📁 users.repository.ts

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersRepository {
  private users: string[] = [];

  findAll() {
    return this.users;
  }

  create(name: string) {
    this.users.push(name);
    return name;
  }
}
```

💡 حتى Repository يعتبر Provider.

---

# 5️⃣ حقن Service داخل Controller

## 📁 users.controller.ts

```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  getUsers() {
    return this.usersService.getAllUsers();
  }

  @Post()
  createUser(@Body('name') name: string) {
    return this.usersService.createUser(name);
  }
}
```

---

# 🔥 ماذا يحدث خلف الكواليس ؟

Nest يقوم بـ:

1️⃣ إنشاء UsersRepository  
2️⃣ إنشاء UsersService وحقن Repository بداخله  
3️⃣ إنشاء UsersController وحقن Service بداخله

بدون new في أي مكان 💥

---

# 6️⃣ تسجيل الـ Providers داخل Module

هذه أهم خطوة.

## 📁 users.module.ts

```ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
})
export class UsersModule {}
```

⚠️ بدون providers لن يعمل الـ Injection.

---

# 7️⃣ Singleton Scope (مهم جداً)

افتراضياً كل Provider هو Singleton.

يعني:

```
UsersService instance واحدة فقط لكل التطبيق
```

لو 1000 request وصلوا  
نفس instance يتم استخدامها.

فوائد:

- Performance عالي
    
- Memory أقل
    

---

# 8️⃣ Optional Provider

أحياناً dependency ليست إجبارية.

مثال: Config.

```ts
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class MailService {
  constructor(
    @Optional()
    @Inject('MAIL_CONFIG')
    private config,
  ) {}
}
```

لو MAIL_CONFIG غير موجود → لا يحدث error.

---

# 9️⃣ Custom Provider (Token Injection)

أحياناً نريد حقن قيمة أو Config.

## مثال Config Provider

```ts
@Module({
  providers: [
    {
      provide: 'APP_NAME',
      useValue: 'My App',
    },
  ],
})
export class AppModule {}
```

استخدامه:

```ts
@Injectable()
export class AppService {
  constructor(@Inject('APP_NAME') private appName: string) {}
}
```

🔥 هذا يسمى Injection Token.

---

# 🔟 Factory Provider (مهم جداً)

عندما تحتاج إنشاء Provider ديناميكياً.

```ts
@Module({
  providers: [
    {
      provide: 'API_KEY',
      useFactory: () => {
        return process.env.API_KEY;
      },
    },
  ],
})
export class AppModule {}
```

---

# 1️⃣1️⃣ Property Injection (نادر الاستخدام)

```ts
@Injectable()
export class TestService {
  @Inject('API_KEY')
  private key: string;
}
```

⚠️ استخدم constructor injection دائماً.

---

# 1️⃣2️⃣ مقارنة NestJS vs Spring Boot

|المفهوم|NestJS|Spring|
|---|---|---|
|Provider|Service/Repository|Bean|
|Decorator|@Injectable|@Service / @Repository|
|DI Container|Nest Container|Spring Container|
|Injection|constructor|constructor / @Autowired|
|Module|@Module|@Configuration|

---

## مقارنة مباشرة

### NestJS

```ts
@Injectable()
export class UserService {}
```

```ts
constructor(private userService: UserService) {}
```

---

### Spring Boot

```java
@Service
public class UserService {}
```

```java
@Autowired
private UserService userService;
```

💡 NestJS = Spring ولكن بـ TypeScript.

---

# 🧠 الخلاصة النهائية

Providers هم قلب Nest ❤️

بدونهم لا يوجد:

- DI
    
- Architecture نظيفة
    
- Separation of Concerns
    
- SOLID
    

كل شيء في Nest هو Provider تقريباً.

---

لو حابب نكمل بعد كده  
**Injection Scopes + Global Providers + ModuleRef** 😎