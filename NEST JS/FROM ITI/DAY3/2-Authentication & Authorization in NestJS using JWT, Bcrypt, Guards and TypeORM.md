المحاضرة دي عنوانها المناسب جدًا ممكن يكون:



أو لو عايز عنوان كورس احترافي:

# **Building Secure Authentication System in NestJS with JWT, Bcrypt, TypeORM and Guards**

وده فعليًا كل اللي اتشرح في الصور والنص.

---

# Roadmap المحاضرة

```text
Authentication
│
├── Password Hashing (bcrypt)
│
├── Login Flow
│
├── JWT Token Generation
│
├── Auth Module Architecture
│
├── JwtModule Configuration
│
├── Authentication Guard
│
└── Authorization using Roles
```

---

# 1) Password Hashing باستخدام bcrypt

بدل ما تخزن:

```text
password = 123456
```

في الداتابيز.

بتخزن:

```text
$2a$10$Kskdjfhw83h8d...
```

لو الداتابيز اتسربت، المهاجم مش هيعرف الباسورد الحقيقي.

---

## التثبيت

```bash
npm install bcryptjs
```

---

## الاستخدام

```ts
const salt = await bcrypt.genSalt(10);

const hashedPassword =
await bcrypt.hash(user.password, salt);
```

---

# ما هو Salt ؟

الـ Salt عبارة عن قيمة عشوائية بتضاف للباسورد قبل التشفير.

مثال:

```text
password = 123456
salt = abc123

result =
hash(123456 + abc123)
```

---

## لماذا نستخدمه؟

لو عندنا:

|User|Password|
|---|---|
|Ali|123456|
|Ahmed|123456|

بدون Salt:

```text
hash = XXXXXX
hash = XXXXXX
```

المهاجم يعرف إن الاتنين عندهم نفس الباسورد.

---

مع Salt:

```text
Ali
123456 + abc123 -> hash1

Ahmed
123456 + xyz789 -> hash2
```

النتيجة:

```text
hash1 != hash2
```

رغم إن الباسورد نفسه متطابق.

---

## Cost Factor = 10

```ts
bcrypt.genSalt(10)
```

الرقم 10 هو عدد الـ rounds.

كلما زاد:

- الأمان يزيد.
    
- الوقت يزيد.
    

غالبًا:

```text
10 -> ممتاز
12 -> ممتاز جدًا
15+ -> بطيء
```

---

# Registration Flow

```text
Client
  |
  v
Controller
  |
  v
AuthService
  |
  v
Hash Password
  |
  v
Database
```

---

# Login Flow

```text
Client
   |
email + password
   |
   v
AuthController
   |
   v
AuthService
   |
   v
find user by email
   |
   +------> not found
   |           |
   |           v
   |      UnauthorizedException
   |
   v
bcrypt.compare()
   |
   +------> invalid password
   |            |
   |            v
   |      UnauthorizedException
   |
   v
generate jwt token
   |
   v
return token
```

---

# أفضل Practice للـ Login

بدل:

```ts
if(!user){
   throw new NotFoundException();
}
```

استخدم:

```ts
throw new UnauthorizedException(
   'Invalid credentials'
)
```

لماذا؟

لأنك لا تريد أن يعرف المهاجم هل المشكلة في الإيميل أم الباسورد.

---

## خطأ

```text
Email doesn't exist
```

ثم:

```text
Wrong password
```

المهاجم هيعرف الإيميلات الصحيحة.

---

## الصحيح

دائمًا:

```text
Invalid credentials
```

---

# Auth Module

نعم ده أفضل تنظيم بكتير.

```bash
nest g mo auth
nest g s auth
nest g co auth
```

---

# Structure

```text
src
|
├── auth
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── dto
│   │    ├── login.dto.ts
│   │    └── register.dto.ts
│   │
│   ├── guards
│   │    └── auth.guard.ts
│   │
│   ├── decorators
│   │    └── roles.decorator.ts
│   │
│   └── strategies
│        └── jwt.strategy.ts
│
├── users
│
├── common
│   ├── guards
│   ├── pipes
│   ├── interceptors
│   └── filters
```

وده أقرب للـ Enterprise Structure.

---

# TypeORM Injection

## Module

```ts
@Module({
 imports:[
   TypeOrmModule.forFeature([User])
 ],
 providers:[AuthService]
})
export class AuthModule {}
```

---

## Service
```
```


```ts
constructor(
 @InjectRepository(User)
 private readonly userRepo:
 Repository<User>
){}
```

---

# Login Logic

```ts
async login(dto: LoginDto) {

 const user =
 await this.userRepo.findOne({
    where:{
       email:dto.email
    }
 });

 if(!user){
   throw new UnauthorizedException(
      'Invalid credentials'
   );
 }

 const matched =
 await bcrypt.compare(
      dto.password,
      user.password
 );

 if(!matched){
   throw new UnauthorizedException(
      'Invalid credentials'
   );
 }

 return this.generateToken(user);
}
```

---
```ts
class AuthService 
{
constructor(@InjectableRepository(User) private readonly repo:Repository<User>){}
async login(dto:logindto)
{
const user = await repo.findone({
where:{email:dto.email}
});
if(!user)
throw new UnauthorizedException("unvalid credintials");

const matched =await bcrypt.compare(dto.password,user.password);
if(!matched)
throw new UnauthorizedException("unvalid credintials");

return this.generateTOken(user);
}
}
```
# JWT

JWT = JSON Web Token

يتكون من:

```text
HEADER
PAYLOAD
SIGNATURE
```

---

## Header

```json
{
 "alg":"HS256",
 "typ":"JWT"
}
```

---

## Payload

```json
{
 "sub":1,
 "email":"ali@gmail.com",
 "role":"ADMIN"
}
```

---

## Signature

```text
HMACSHA256(
 base64(header)
 +
 base64(payload)
 +
 secret
)
```

---

## الشكل النهائي

```text
xxxxx.yyyyy.zzzzz
```

---

# JwtModule

```bash
npm install @nestjs/jwt jsonwebtoken
```

---

## .env

```env
JWT_SECRET=mysecretkey
JWT_EXPIRES_IN=3h
```

---

## AppModule

```ts
ConfigModule.forRoot({
 isGlobal:true
})
```

---

## registerAsync

لماذا نستخدمها؟

لأن الـ secret موجود في env.

```ts
JwtModule.registerAsync({
 imports:[ConfigModule],

 inject:[ConfigService],

 useFactory:(
    config:ConfigService
 ) => ({
      secret:
      config.get('JWT_SECRET'),

      signOptions:{
          expiresIn:
          config.get('JWT_EXPIRES_IN')
      }
 })
})
```

---

# لماذا registerAsync أفضل من register ؟

لأن:

```text
.env
↓
ConfigService
↓
JwtModule
```

يتم تحميل القيم وقت التشغيل.

---

# Generate Token

```ts
const payload = {
    sub:user.id,
    role:user.role
};

const token =
this.jwtService.sign(payload);

return {
   accessToken:token
};
```

---

# لماذا لا نضع الإيميل والباسورد داخل JWT ؟

JWT ليست مشفرة.

أي شخص معه التوكن يستطيع قراءة الـ Payload.

هي فقط Signed وليست Encrypted.

---

# Authentication Guard

الهدف:

```text
هل المستخدم مسجل دخول أم لا؟
```

---

## تنفيذ CanActivate

```ts
@Injectable()
export class AuthGuard
implements CanActivate {

 constructor(
   private jwtService:JwtService
 ){}

 async canActivate(
  context:ExecutionContext
 ){

  const request =
  context.switchToHttp()
         .getRequest();

  const token =
  request.headers.authorization
  ?.split(' ')[1];

  if(!token)
     throw new UnauthorizedException();

  try{

      const payload =
      await this.jwtService.verifyAsync(
         token
      );

      request.user = payload;

      return true;

  }catch{
      throw new UnauthorizedException();
  }
 }
}
```

---

## Authorization Header

```text
Authorization:
Bearer eyJhb...
```

---

## لماذا Bearer ؟

لأن البروتوكول القياسي للـ JWT هو:

```text
Bearer Token Authentication
```

---

# Request Lifecycle

```text
Request
   |
Middleware
   |
Guards
   |
Interceptors
   |
Pipes
   |
Controller
   |
Service
   |
Repository
   |
Database
```

---

# UseGuards

```ts
@UseGuards(AuthGuard)
@Get('profile')
getProfile(){}
```

---

# Global Guard

```ts
app.useGlobalGuards(
   app.get(AuthGuard)
)
```

---

# Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What can you do?
```

---

## مثال

```text
User
 ├── Read Courses
 └── Buy Course

Admin
 ├── Create Course
 ├── Delete Course
 └── Manage Users
```

---

## Roles Decorator

```ts
@Roles('ADMIN')
```

---

## Role Guard

```ts
@UseGuards(
 AuthGuard,
 RolesGuard
)
```

---

# Architecture Diagram

```text
Client
   |
   v
Auth Controller
   |
   v
Auth Service
   |
   +------------+
   |            |
   v            v
TypeORM      JwtService
   |            |
   v            v
Database      JWT Token
```

---

# Enterprise Best Practices

## JWT Payload

ضع فقط:

```text
userId
role
permissions
```

ولا تضع:

```text
email
password
phone
address
```

---

## Access Token + Refresh Token

الأفضل في المشاريع الكبيرة:

```text
Access Token -> 15 min

Refresh Token -> 7 days
```

---

## استخدم Passport

الأفضل في Nest:

```text
@nestjs/passport
passport-jwt
```

بدل كتابة Guard يدويًا.

---

## استخدم Strategies

```text
JwtStrategy
LocalStrategy
GoogleStrategy
GithubStrategy
```

---

# ملخص المحاضرة

```text
bcrypt
│
├── genSalt()
├── hash()
└── compare()

JWT
│
├── JwtModule
├── sign()
├── verify()
└── registerAsync()

Authentication
│
├── login
├── register
└── guards

Authorization
│
├── roles
├── permissions
└── role guard

TypeORM
│
├── Repository Injection
└── User Queries
```

دي تقريبًا محاضرة كاملة عن بناء نظام Authentication احترافي في NestJS باستخدام TypeORM و JWT و Guards و bcrypt، والمرحلة اللي بعدها طبيعي جدًا تكون:

```text
Passport Strategies
Refresh Tokens
Role Guards
Permissions
RBAC
```