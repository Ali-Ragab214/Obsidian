التاسك ده عبارة عن بناء **Order API مع Authentication باستخدام NestJS + MongoDB + JWT**.  
لو قسمناه لشغل حقيقي هيبقى كده:

---

# المرحلة 1: تجهيز المشروع

## تثبيت الباكدجات

```bash
npm install @nestjs/mongoose mongoose
npm install @nestjs/config
npm install @nestjs/jwt passport passport-jwt
npm install @nestjs/passport
npm install bcrypt
npm install class-validator class-transformer
npm install -D @types/bcrypt
```

---

# المرحلة 2: إعداد الـ Environment Variables

اعمل ملف:

```env
MONGO_URI=mongodb://localhost:27017/orders_db

JWT_SECRET=super-secret-key

JWT_EXPIRES_IN=1d
```

---

## فعل ConfigModule

```ts
ConfigModule.forRoot({
  isGlobal: true,
});
```

---

# المرحلة 3: توصيل MongoDB

داخل `app.module.ts`

```ts
MongooseModule.forRoot(
  process.env.MONGO_URI
)
```

أو الأفضل:

```ts
MongooseModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    uri: config.get<string>('MONGO_URI'),
  }),
})
```

---

# المرحلة 4: إنشاء Orders Module

```bash
nest g module orders
nest g controller orders
nest g service orders
```

---

# المرحلة 5: إنشاء Order Schema

مثال:

```ts
@Schema()
export class Order {

  @Prop()
  productName: string;

  @Prop()
  quantity: number;

  @Prop()
  price: number;

  @Prop()
  userId: string;
}
```

ثم:

```ts
export const OrderSchema =
    SchemaFactory.createForClass(Order);
```

---

## تسجيل الـ Schema

داخل:

```ts
OrdersModule
```

```ts
MongooseModule.forFeature([
  {
    name: Order.name,
    schema: OrderSchema,
  },
])
```

---

# المرحلة 6: إنشاء DTOs

## CreateOrderDto

```ts
export class CreateOrderDto {

  @IsString()
  productName: string;

  @IsInt()
  @Min(1)
  quantity: number;

  @IsNumber()
  price: number;
}
```

---

## UpdateOrderDto

```ts
export class UpdateOrderDto
  extends PartialType(CreateOrderDto) {}
```

---

# المرحلة 7: تفعيل Validation

في:

```ts
main.ts
```

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
  }),
);
```

---

# المرحلة 8: حفظ الطلبات في MongoDB

داخل الـ Service:

```ts
create(dto: CreateOrderDto) {
   const order = new this.orderModel(dto);
   return order.save();
}
```

---

# المرحلة 9: إنشاء Users Module

```bash
nest g module users
nest g service users
```

---

# المرحلة 10: إنشاء User Schema

مثال:

```ts
@Schema()
export class User {

   @Prop()
   email: string;

   @Prop()
   password: string;
}
```

---

# المرحلة 11: إنشاء Auth Module

```bash
nest g module auth
nest g controller auth
nest g service auth
```

---

# المرحلة 12: Endpoint التسجيل

## POST

```text
/auth/signup
```

الخطوات:

- استلام email و password.
    
- التأكد أن المستخدم غير موجود.
    
- تشفير الباسورد باستخدام bcrypt.
    
- حفظ المستخدم.
    

---

مثال:

```ts
const hashedPassword =
 await bcrypt.hash(password,10);
```

---

# المرحلة 13: Endpoint تسجيل الدخول

## POST

```text
/auth/login
```

الخطوات:

- البحث عن المستخدم.
    
- مقارنة الباسورد:
    

```ts
bcrypt.compare()
```

- إنشاء JWT Token:
    

```ts
this.jwtService.sign(payload)
```

---

مثال Payload:

```json
{
   "sub":"user_id",
   "email":"test@gmail.com"
}
```

---

# المرحلة 14: إعداد JWT Module

```ts
JwtModule.registerAsync({
   inject:[ConfigService],
   useFactory:(config:ConfigService)=>({
      secret: config.get('JWT_SECRET'),
      signOptions:{
         expiresIn:
            config.get('JWT_EXPIRES_IN')
      }
   })
})
```

---

# المرحلة 15: إنشاء JWT Strategy

```bash
passport-jwt strategy
```

وظيفتها:

- استخراج التوكن من الهيدر.
    
- التحقق من صحته.
    
- وضع الـ user داخل request.user.
    

---

# المرحلة 16: إنشاء JWT Auth Guard

```ts
@Injectable()
export class JwtAuthGuard
extends AuthGuard('jwt') {}
```

---

# المرحلة 17: حماية الـ Orders Mutation Routes

المطلوب يقول:

> Protect all Mutation Orders routes

يعني:

- Create Order
    
- Update Order
    
- Delete Order
    

وليس:

- Get Orders
    
- Get Order By Id
    

مثال:

```ts
@UseGuards(JwtAuthGuard)
@Post()
create(){}
```

---

```ts
@UseGuards(JwtAuthGuard)
@Patch(':id')
update(){}
```

---

```ts
@UseGuards(JwtAuthGuard)
@Delete(':id')
remove(){}
```

---

# الشكل النهائي للموديولات

```text
src
│
├── auth
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── jwt.strategy.ts
│   └── jwt-auth.guard.ts
│
├── users
│   ├── users.module.ts
│   ├── users.service.ts
│   └── schemas/user.schema.ts
│
├── orders
│   ├── dto
│   ├── schemas
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   └── orders.module.ts
│
├── app.module.ts
│
└── main.ts
```

---

## المطلوب منك فعليًا بالترتيب:

1. ConfigModule + .env
    
2. MongoDB Connection
    
3. Orders Module
    
4. Order Schema
    
5. CreateOrderDto
    
6. UpdateOrderDto
    
7. ValidationPipe
    
8. Users Module
    
9. User Schema
    
10. Auth Module
    
11. Signup Endpoint
    
12. Login Endpoint
    
13. JWT Configuration
    
14. JWT Strategy
    
15. JWT Guard
    
16. حماية Create/Update/Delete Orders
    

لو نفذت الـ 16 خطوة دول فأنت خلصت التاسك بالكامل بنسبة 100٪ تقريبًا.