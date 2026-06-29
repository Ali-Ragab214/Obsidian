ممتاز، دي تعتبر بداية الـ **Real World NestJS Development** لأنك من هنا هتبدأ تشتغل بنفس الشكل اللي بيشتغل بيه أغلب مشاريع Nest الحقيقية.

هشرح الموضوع من الصفر كأنك بتبني API Production جاهز.

---

# Architecture Overview

قبل ما نكتب أي كود لازم نفهم الـ Flow كله:

```text
Client
   |
HTTP Request
   |
Controller
   |
Service
   |
Database Model
   |
MongoDB
```

الرد يرجع بالعكس:

```text
MongoDB
   |
Model
   |
Service
   |
Controller
   |
HTTP Response
   |
Client
```

---

# لماذا Service Layer موجودة أصلاً ؟

تخيل الكود ده:

```ts
@Post()
create(@Body() dto: CreateUserDto) {
    const user = new this.userModel(dto);

    return user.save();
}
```

الكود ده شغال.

لكن المشكلة:

الـ Controller بقى مسؤول عن:

- استقبال Request
    
- التعامل مع Database
    
- تنفيذ Business Logic
    
- بناء Response
    

وده مخالف لمبدأ:

```text
Single Responsibility Principle
```

---

## الشكل الصحيح

```text
Controller
    ↓
Service
    ↓
Repository / Model
    ↓
Database
```

---

## Controller

مسؤول عن:

- استقبال الـ Request
    
- قراءة params
    
- قراءة query
    
- قراءة body
    
- استدعاء Service
    
- إعادة Response
    

---

## Service

مسؤول عن:

- Business Logic
    
- التعامل مع DB
    
- التعامل مع APIs خارجية
    
- تنفيذ Rules الخاصة بالنظام
    

---

# إنشاء Service

```bash
nest g service users
```

أو:

```bash
nest g s users
```

سيتم إنشاء:

```text
users.service.ts
users.service.spec.ts
```

وسيتم تسجيله تلقائياً داخل:

```ts
@Module({
  providers:[UsersService]
})
```

---

# @Injectable()

```ts
@Injectable()
export class UsersService {}
```

## ما فائدتها ؟

تقول لـ Nest:

> هذا الكلاس يمكن حقنه داخل كلاس آخر باستخدام Dependency Injection.

---

مثال:

```ts
@Controller('users')
export class UsersController {

    constructor(
        private readonly usersService: UsersService
    ) {}
}
```

Nest سيقوم بإنشاء Object تلقائياً.

---

بدون:

```ts
@Injectable()
```

لن يستطيع Nest إنشاء الـ Service.

---

# Dependency Injection Diagram

```text
UsersController
        |
        v
UsersService
        |
        v
UserModel
        |
        v
MongoDB
```

---

# لماذا Dependency Injection ؟

بدلاً من:

```ts
const service = new UsersService(); //tight coupling
```

نكتب:

```ts
constructor(
   private usersService: UsersService
){}
```

الفوائد:

- Testability
    
- Loose Coupling
    
- Reusability
    
- Maintainability
    

---

# MongoDB Connection

## تثبيت الحزم

أنت كتبت:

```bash
npm i @nestjs/mongose mongose
```

الصحيح:

```bash
npm i @nestjs/mongoose mongoose
```

---

## ماذا يفعل كل Package ؟

### mongoose

ODM مثل:

```text
Hibernate
JPA
TypeORM
```

لكن لـ MongoDB.

---

### @nestjs/mongoose

Integration Layer بين:

```text
NestJS
+
Mongoose
```

---

# AppModule

```ts
@Module({
  imports:[
      MongooseModule.forRoot(
          'mongodb://localhost:27017/nest-app'
      )
  ]
})
export class AppModule {}
```

---

## ماذا يفعل forRoot() ؟

يقوم بإنشاء:

```text
Database Connection
```

مرة واحدة للتطبيق كله.

---

# Schema

في SQL:

```text
Entity
```

في Mongo:

```text
Schema
```

---

# إنشاء Folder

```text
users
│
├── dto
├── schemas
│   └── user.schema.ts
├── users.controller.ts
├── users.service.ts
└── users.module.ts
```

---

# @Schema()

```ts
@Schema()
export class User {}
```

تقول لـ Nest:

```text
This class represents a Mongo Collection Schema
```

---

# @Prop()

مثل:

```java
@Column
```

في JPA.

---

مثال:

```ts
@Schema()
export class User {

    @Prop({
        required:true,
        unique:true
    })
    username:string;

    @Prop({
        required:true,
        unique:true
    })
    email:string;

    @Prop({
        required:true
    })
    password:string;

}
```

---

# Validation داخل Schema ؟

نعم.

مثال:

```ts
@Prop({
    required:true,
    minlength:3,
    maxlength:30
})
name:string;
```

---

لكن الأفضل دائماً:

```text
DTO Validation
+
Schema Validation
```

---

DTO Validation:

```text
API Layer Protection
```

Schema Validation:

```text
Database Protection
```

---

# SchemaFactory

```ts
export const UserSchema =
    SchemaFactory.createForClass(User);
```

يقوم بتحويل:

```text
Class
```

إلى:

```text
Mongoose Schema Object
```

---

# UsersModule

```ts
@Module({
  imports:[
      MongooseModule.forFeature([
          {
              name:User.name,
              schema:UserSchema
          }
      ])
  ]
})
export class UsersModule {}
```

---

## ماذا يفعل forFeature() ؟

يسجل Model داخل هذا الـ Module فقط.

---

الفرق:

## forRoot()

```text
Database Connection
```

مرة واحدة.

---

## forFeature()

```text
Register Models
```

لكل Module.

---

# Inject Model

```ts
constructor(
   @InjectModel(User.name)
   private readonly userModel: Model<User>
){}
```

---

هذا يشبه:

```java
private final UserRepository repository;
```

في Spring.

---

فعلياً:

```text
Model<User>
```

هو Repository الخاص بـ Mongo.

---

# لماذا async و await ؟

MongoDB عملياتها:

```text
I/O Operations
```

وليست CPU Operations.

---

الاتصال بالـ Database يستغرق وقتاً:

```text
network latency
disk access
serialization
```

لذلك كل العمليات تكون:

```ts
async
```

---

مثال:

```ts
async findAll() {
   return await this.userModel.find();
}
```

---

# هل أكتب return await ؟

في أغلب الحالات:

```ts
return this.userModel.find();
```

أفضل.

---

مثال صحيح:

```ts
async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
}
```

---

## لماذا exec() ؟

Best Practice في Mongoose.

بدلاً من:

```ts
return this.userModel.find();
```

نستخدم:

```ts
return this.userModel.find().exec();
```

لأن:

- يعيد Promise حقيقي.
    
- Stack traces أفضل.
    
- Testing أسهل.
    

---

# متى نستخدم await ؟

عندما نحتاج النتيجة قبل تنفيذ خطوة أخرى.

مثال:

```ts
const existingUser =
    await this.userModel
        .findOne({email:dto.email})
        .exec();

if(existingUser)
    throw new ConflictException();

const user =
    await this.userModel.create(dto);

return user;
```

---

# Environment Variables

لا تضع:

```ts
mongodb://admin:password@localhost
```

داخل الكود.

---

الصحيح:

```env
PORT=3000

DB_URL=mongodb://localhost:27017/nest-db
```

---

# Config Module

التثبيت:

```bash
npm i @nestjs/config
```

---

# App Module

```ts
ConfigModule.forRoot({
    isGlobal:true,
    envFilePath:['.env']
})
```

---

## isGlobal

بدلاً من:

```ts
imports:[
   ConfigModule
]
```

في كل Module.

---

يكفي مرة واحدة فقط.

---

# ConfigService

```ts
constructor(
   private readonly configService: ConfigService
){}
```

---

الوصول للمتغير:

```ts
this.configService.get<string>(
   'DB_URL'
)
```

---

# Database Connection Best Practice

```ts
MongooseModule.forRootAsync({

    imports:[ConfigModule],

    inject:[ConfigService],

    useFactory:(
        configService:ConfigService
    ) => ({
        uri:configService.get<string>(
            'DB_URL'
        )
    })

})
```

---

# لماذا forRootAsync ؟

لأن:

```text
ConfigService
```

لا يكون جاهزاً أثناء Bootstrap الأولي.

---

# Full Production Example

## create-user.dto.ts

```ts
export class CreateUserDto {

    @IsString()
    @MinLength(3)
    name:string;

    @IsEmail()
    email:string;

    @MinLength(8)
    password:string;
}
```

---

## update-user.dto.ts

```ts
export class UpdateUserDto
extends PartialType(CreateUserDto){}
```

---

## user.schema.ts

```ts
@Schema({
    timestamps:true
})
export class User {

    @Prop({
        required:true
    })
    name:string;

    @Prop({
        required:true,
        unique:true
    })
    email:string;

    @Prop({
        required:true
    })
    password:string;
}

export const UserSchema =
    SchemaFactory.createForClass(User);
```

---

## users.module.ts

```ts
@Module({
 imports:[
     MongooseModule.forFeature([
         {
             name:User.name,
             schema:UserSchema
         }
     ])
 ],
 controllers:[
     UsersController
 ],
 providers:[
     UsersService
 ]
})
export class UsersModule {}
```

---

## users.service.ts

```ts
@Injectable()
export class UsersService {

 constructor(
   @InjectModel(User.name)
   private readonly userModel:
       Model<User>
 ){}

 async create(
     dto:CreateUserDto
 ):Promise<User>{

     const existingUser =
         await this.userModel
             .findOne({
                 email:dto.email
             })
             .exec();

     if(existingUser){
         throw new ConflictException(
             'Email already exists'
         );
     }

     const user =
         await this.userModel.create(dto);

     return user;
 }

 async findAll():Promise<User[]>{
     return this.userModel.find().exec();
 }

 async findOne(
     id:string
 ):Promise<User>{

     const user =
         await this.userModel
             .findById(id)
             .exec();

     if(!user)
         throw new NotFoundException();

     return user;
 }

}
```

---

## users.controller.ts

```ts
@Controller('users')
export class UsersController {

 constructor(
    private readonly usersService:
        UsersService
 ){}

 @Post()
 create(
    @Body()
    dto:CreateUserDto
 ){
    return this.usersService
        .create(dto);
 }

 @Get()
 findAll(){
    return this.usersService
        .findAll();
 }

 @Get(':id')
 findOne(
    @Param(
      'id',
      ParseObjectIdPipe
    )
    id:string
 ){
    return this.usersService
        .findOne(id);
 }

}
```

---

# الشكل النهائي للمشروع

```text
src
│
├── config
│
├── users
│   ├── dto
│   ├── schemas
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── users.module.ts
│
├── app.module.ts
│
└── main.ts
```

وده تقريباً نفس الشكل اللي هتشوفه في أغلب مشاريع NestJS الحقيقية في الشركات الكبيرة والـ SaaS applications.