#
---

# لماذا نعمل Validation أصلاً ؟
> نعم، لو عملت Validation Decorators داخل DTO فلازم يكون عندك ValidationPipe شغال، إما بـ `@UsePipes()` على Route أو Controller أو الأفضل والأشهر عن طريق `app.useGlobalPipes()` في `main.ts`.

تخيل عندك API لإنشاء مستخدم:

```http
POST /users
```

والـ frontend بعت:

```json
{
    "name": "",
    "email": "abc",
    "age": -5
}
```

لو دخلت البيانات دي للـ Service مباشرة ممكن يحصل:

- بيانات فاسدة داخل الـ Database.
    
- Exceptions أثناء التنفيذ.
    
- مشاكل Security.
    
- Business Logic يشتغل على بيانات غلط.
    

عشان كده Nest بيعمل Validation قبل ما الـ Request يوصل للـ Service.

---

# مكان الـ Validation في دورة حياة الـ Request

```text
Client
  |
  v
Middleware
  |
  v
Guards
  |
  v
Interceptors
  |
  v
Pipes (Validation Happens Here)
  |
  v
Controller
  |
  v
Service
  |
  v
Database
```

الـ Validation بيحصل قبل الـ Controller حتى.

يعني لو البيانات غلط:

```text
Controller won't execute
Service won't execute
Database won't execute
```

---

# Nest Validation يعتمد على حاجتين

```text
1- DTO
2- Validation Pipe
```

الاثنين لازم يكونوا موجودين.

---

# بدون Validation Pipe

```ts
@Post()
create(@Body() dto: CreateUserDto) {
    return dto;
}
```

حتى لو DTO فيه Decorators:

```ts
@IsEmail()
email:string;
```

لن يعمل أي Validation.

---

# بدون DTO

```ts
@Post()
create(@Body() body:any) {}
```

الـ Pipe مش هيعرف يعمل Validation لأنه لا يعرف شكل البيانات المطلوبة.

---

# إذن المعادلة هي

```text
DTO + ValidationPipe = Validation
```

---

# DTO

DTO اختصار:

```text
Data Transfer Object
```

وهو Class يحدد شكل البيانات الداخلة للـ API.

---

## مثال

```ts
export class CreateUserDto {

    name:string;

    email:string;

    password:string;

}
```

---

الـ Frontend يجب أن يرسل:

```json
{
    "name":"Ali",
    "email":"ali@gmail.com",
    "password":"123456"
}
```

---

# لماذا DTO عبارة عن Class وليس Interface ؟

لأن:

```text
Interfaces disappear after compilation.
```

مثال:

```ts
interface User {
    name:string;
}
```

بعد التحويل لـ JavaScript:

```js
// vanished
```

اختفى تمامًا.

---

أما:

```ts
class UserDto {
    name:string;
}
```

يبقى موجودًا أثناء Runtime.

والـ ValidationPipe يحتاج معرفة نوع الكلاس أثناء Runtime.

---

# تثبيت الحزم المطلوبة

```bash
npm install class-validator class-transformer
```

أو:

```bash
npm i class-validator class-transformer
```

---

# class-validator

هذا المكتبة توفر Decorators للفاليديشن.

---

## أشهر Decorators

---

## IsString

```ts
@IsString()
name:string;
```

يقبل فقط:

```json
"name":"Ali"
```

ويرفض:

```json
"name":123
```

---

## IsEmail

```ts
@IsEmail()
email:string;
```

يقبل:

```json
"email":"test@gmail.com"
```

ويرفض:

```json
"email":"abc"
```

---

## Length

```ts
@Length(5,20)
password:string;
```

الحد الأدنى:

```text
5
```

الحد الأقصى:

```text
20
```

---

## MinLength

```ts
@MinLength(8)
password:string;
```

---

## MaxLength

```ts
@MaxLength(50)
bio:string;
```

---

## IsInt

```ts
@IsInt()
age:number;
```

---

## Min

```ts
@Min(18)
age:number;
```

---

## Max

```ts
@Max(60)
age:number;
```

---

## IsBoolean

```ts
@IsBoolean()
isAdmin:boolean;
```

---

## IsOptional

مهم جدًا في Update DTO.

```ts
@IsOptional()
@IsString()
name?:string;
```

---

## IsNotEmpty

```ts
@IsNotEmpty()
name:string;
```

يرفض:

```json
"name":""
```

---

## Equals

```ts
@Equals("admin")
role:string;
```

---

## IsUUID

```ts
@IsUUID()
id:string;
```

---

## مثال احترافي

```ts
import {
    IsEmail,
    IsString,
    MinLength,
    MaxLength,
    IsInt,
    Min
} from "class-validator";

export class CreateUserDto {

    @IsString()
    name:string;

    @IsEmail()
    email:string;

    @MinLength(8)
    @MaxLength(30)
    password:string;

    @IsInt()
    @Min(18)
    age:number;
}
```

---

# class-transformer

هذه المكتبة مسؤولة عن:

```text
Transformation
Serialization
Deserialization
```

---

## مثال

الـ Request:

```json
{
    "age":"25"
}
```

النوع القادم من HTTP دائمًا:

```text
string
```

لكن DTO يتوقع:

```ts
age:number
```

الـ class-transformer يحول:

```text
"25"
```

إلى:

```text
25
```

---

مثال:

```ts
@Type(() => Number)
age:number;
```

---

# تفعيل Validation

## على Route واحدة

```ts
@Post()
@UsePipes(new ValidationPipe())
create(@Body() dto:CreateUserDto) {
    return dto;
}
```

---

الـ Flow:

```text
Request
   |
   v
ValidationPipe
   |
   +--> valid --> Controller
   |
   +--> invalid --> 400 Bad Request
```

---

# على Controller بالكامل

```ts
@UsePipes(new ValidationPipe())

@Controller("users")
export class UserController {}
```

كل Route داخل الـ Controller سيخضع للفاليديشن.

---

# Global Validation
most used

داخل:

```ts
main.ts
```

```ts
app.useGlobalPipes(
    new ValidationPipe()
);
```

---

الآن:

```text
Every Controller
Every Route
Every DTO
```

سيتم عمل Validation له.

وهذا هو الأسلوب المستخدم في المشاريع الحقيقية.

---

# ValidationPipe Options

---

# whitelist

افتراضيًا:

```ts
false
```

---

مثال DTO:

```ts
export class CreateUserDto {

    name:string;

    email:string;
}
```

والـ frontend أرسل:

```json
{
    "name":"Ali",
    "email":"ali@gmail.com",
    "salary":999999
}
```

---

بدون whitelist:

```json
{
    "name":"Ali",
    "email":"ali@gmail.com",
    "salary":999999
}
```

الـ salary سيمر.

---

مع:

```ts
new ValidationPipe({
    whitelist:true
})
```

سيصبح:

```json
{
    "name":"Ali",
    "email":"ali@gmail.com"
}
```

تم حذف salary.

---

# forbidNonWhitelisted

```ts
new ValidationPipe({
    whitelist:true,
    forbidNonWhitelisted:true
})
```

---

بدلاً من حذف الحقول الزائدة:

سيتم إرجاع:

```json
{
    "statusCode":400,
    "message":[
        "property salary should not exist"
    ]
}
```

---

هذا الخيار هو الأفضل غالبًا.

---

## الشكل الاحترافي

```ts
app.useGlobalPipes(
    new ValidationPipe({
        whitelist:true,
        forbidNonWhitelisted:true,
        transform:true
    })
);
```

---

# transform:true

مثال:

```http
GET /users/5
```

بدون transform:

```ts
typeof id
```

يكون:

```text
string
```

---

مع transform:

```text
number
```

إذا كان الـ DTO أو Pipe يسمح بذلك.

---

# Built In Pipes

Nest يوفر Pipes جاهزة.

---

## ParseIntPipe

```ts
@Get(":id")
findOne(
    @Param(
        "id",
        ParseIntPipe
    )
    id:number
){}
```

---

لو المستخدم أرسل:

```http
/users/abc
```

سيحصل على:

```json
400 Bad Request
```

---

## ParseBoolPipe

```ts
@Query(
    "active",
    ParseBoolPipe
)
active:boolean
```

---

## ParseFloatPipe

```ts
@Query(
    "price",
    ParseFloatPipe
)
price:number
```

---

## ParseArrayPipe

```ts
@Query(
    "ids",
    new ParseArrayPipe({
        items:Number,
        separator:","
    })
)
ids:number[]
```

---

## ParseUUIDPipe

```ts
@Get(":id")
findOne(
    @Param(
        "id",
        ParseUUIDPipe
    )
    id:string
){}
```

---

لو المستخدم أرسل:

```text
123
```

سيظهر:

```text
Validation failed (uuid is expected)
```

---

مفيد جدًا مع MongoDB.

لأن:

```text
Mongo ObjectId != UUID
```

فلو تستخدم Mongo ObjectId غالبًا ستحتاج Pipe خاصة بك.

---

# ما هي فكرة Pipe أصلاً ؟

الـ Pipe تشبه حرفيًا ماسورة مياه.

```text
input
  |
  v
+---------+
|  Pipe   |
+---------+
  |
  v
output
```

يمكنها:

- تعديل البيانات.
    
- التحقق من البيانات.
    
- رمي Exceptions.
    

---

مثال:

```text
input = "25"
```

الـ Pipe:

```text
ParseIntPipe
```

الناتج:

```text
25
```

---

مثال آخر:

```text
input = "abc"
```

الناتج:

```text
throw BadRequestException
```

---

# Custom Pipes

عندما لا تكفي الـ Built In Pipes.

---

# إنشاء Pipe

```bash
nest g pipe uuid
```

أو:

```bash
nest g pi uuid
```

سينشئ:

```text
uuid.pipe.ts
uuid.pipe.spec.ts
```

---

# شكل Pipe

```ts
@Injectable()
export class UuidPipe
implements PipeTransform {

    transform(
        value:any,
        metadata:ArgumentMetadata
    ) {
        return value;
    }
}
```

---

# لماذا Injectable ؟

حتى يستطيع Nest عمل:

```text
Dependency Injection
```

داخل الـ Pipe.

---

# PipeTransform Interface

يفرض عليك وجود:

```ts
transform()
```

مثل:

```java
implements Runnable
```

في Java.

---

# transform()

هذه هي قلب الـ Pipe.

تُنفذ تلقائيًا عندما تستخدم الـ Pipe.

---

## Parameters

```ts
transform(
    value:any,
    metadata:ArgumentMetadata
)
```

---

## value

القيمة الداخلة للـ Pipe.

مثال:

```http
GET /users/15
```

```ts
@Param("id",MyPipe)
```

هنا:

```ts
value = "15"
```

---

## metadata

يحتوي على معلومات عن القيمة.

مثال:

```ts
console.log(metadata);
```

الناتج:

```ts
{
    type:'param',
    metatype:String,
    data:'id'
}
```

---

### type

مصدر القيمة:

```text
body
query
param
custom
```

---

### metatype

نوع البيانات:

```text
String
Number
CreateUserDto
```

---

### data

اسم المتغير.

```text
id
page
email
```

---

# مثال عملي

```ts
@Injectable()
export class PositiveNumberPipe
implements PipeTransform {

    transform(value:number) {

        if(value <= 0){
            throw new BadRequestException(
                "Id must be positive"
            );
        }

        return value;
    }
}
```

---

الاستخدام:

```ts
@Get(":id")
findOne(
    @Param(
        "id",
        ParseIntPipe,
        PositiveNumberPipe
    )
    id:number
){}
```

---

الـ Flow:

```text
"15"
 |
ParseIntPipe
 |
15
 |
PositiveNumberPipe
 |
15
 |
Controller
```

---

أما:

```text
"-5"
 |
ParseIntPipe
 |
-5
 |
PositiveNumberPipe
 |
Exception
```

---

# تسلسل التنفيذ

```text
Request
 |
Built In Pipe
 |
Custom Pipe
 |
Controller
 |
Service
 |
Database
```

---

# ملخص سريع

```text
DTO
↓
defines structure

ValidationPipe
↓
executes validation decorators

class-validator
↓
provides decorators

class-transformer
↓
transforms data types

Built-in Pipes
↓
ParseIntPipe
ParseUUIDPipe
ParseBoolPipe

Custom Pipes
↓
your own validation/transformation logic
```

وده يعتبر تقريبًا 80% من كل ما يخص Validation و Pipes في NestJS في المشاريع الحقيقية.