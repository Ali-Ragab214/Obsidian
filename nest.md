تمام، هنعمل مشروع Recap فيه **Module بتاع Users** و**Module بتاع Auth**، مبني على **TypeORM + PostgreSQL**، وفيه كل الحاجات اللي احنا اتكلمنا فيها (DTO, Validation, Entity, Service, Controller, JWT, bcrypt, Guards).

## الـ Structure الكامل الأول

```text
src
│
├── users
│   ├── entities
│   │   └── user.entity.ts
│   ├── dto
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   └── users.module.ts
│
├── auth
│   ├── dto
│   │   ├── register.dto.ts
│   │   └── login.dto.ts
│   ├── guards
│   │   ├── auth.guard.ts
│   │   └── roles.guard.ts
│   ├── decorators
│   │   └── roles.decorator.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.module.ts
│
├── app.module.ts
└── main.ts

.env
```

دلوقتي هعديك على كل فايل بالترتيب.

---

## 1) `.env`

فيه بيانات الداتابيز والـ JWT، ومحدش بيرفعه على Git.

```env
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=yourpassword
DB_NAME=recap_db

JWT_SECRET=super_secret_key
JWT_EXPIRES_IN=3h
```

---

## 2) `users/entities/user.entity.ts`

دي الـ Entity بتاعة TypeORM، بتمثل الجدول في الداتابيز.

```ts
import { Column, Entity, PrimaryGeneratedColumn } from "typeorm";

@Entity("users")
export class User {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column({ unique: true })
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({ default: "USER" })
  role: string;
}
```

**ملاحظة:** الباسورد هنا بيتخزن مشفر (bcrypt hash) مش plain text، هتشوف ده في الـ `AuthService`.

---

## 3) `users/dto/create-user.dto.ts`

DTO الخاص بإنشاء المستخدم — فيه Validation باستخدام `class-validator`.

```ts
import { IsEmail, IsString, MinLength, MaxLength } from "class-validator";

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @MinLength(8)
  @MaxLength(30)
  password: string;
}
```

---

## 4) `users/dto/update-user.dto.ts`

بيستخدم `PartialType` عشان يخلي كل الحقول Optional تلقائيًا.

```ts
import { PartialType } from "@nestjs/mapped-types";
import { CreateUserDto } from "./create-user.dto";

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

---

## 5) `users/users.service.ts`

هنا كل الـ Business Logic — الـ Controller مش هيلمس الداتابيز أبداً.

```ts
import { Injectable, NotFoundException, ConflictException } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { User } from "./entities/user.entity";
import { CreateUserDto } from "./dto/create-user.dto";
import { UpdateUserDto } from "./dto/update-user.dto";

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}

  async create(dto: CreateUserDto & { password: string }): Promise<User> {
    const existing = await this.userRepo.findOne({ where: { email: dto.email } });
    if (existing) {
      throw new ConflictException("Email already exists");
    }
    const user = this.userRepo.create(dto);
    return this.userRepo.save(user);
  }

  async findAll(): Promise<User[]> {
    return this.userRepo.find();
  }

  async findOne(id: string): Promise<User> {
    const user = await this.userRepo.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException("User not found");
    }
    return user;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepo.findOne({ where: { email } });
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id);
    Object.assign(user, dto);
    return this.userRepo.save(user);
  }

  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await this.userRepo.remove(user);
  }
}
```

---

## 6) `users/users.controller.ts`

مفيش أي Logic هنا، بس استقبال Request وإرجاع Response.

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Patch,
  Delete,
  ParseUUIDPipe,
  UseGuards,
} from "@nestjs/common";
import { UsersService } from "./users.service";
import { CreateUserDto } from "./dto/create-user.dto";
import { UpdateUserDto } from "./dto/update-user.dto";
import { AuthGuard } from "../auth/guards/auth.guard";

@Controller("users")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @UseGuards(AuthGuard)
  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @UseGuards(AuthGuard)
  @Get(":id")
  findOne(@Param("id", ParseUUIDPipe) id: string) {
    return this.usersService.findOne(id);
  }

  @UseGuards(AuthGuard)
  @Patch(":id")
  update(@Param("id", ParseUUIDPipe) id: string, @Body() dto: UpdateUserDto) {
    return this.usersService.update(id, dto);
  }

  @UseGuards(AuthGuard)
  @Delete(":id")
  remove(@Param("id", ParseUUIDPipe) id: string) {
    return this.usersService.remove(id);
  }
}
```

**ملاحظة مهمة:** مفيش `POST /users` هنا — إنشاء اليوزر بيحصل من `AuthController` عن طريق `/auth/register`، عشان الباسورد لازم يتشفر الأول قبل ما يتخزن، والمنطق ده مكانه في `AuthService`.

---

## 7) `users/users.module.ts`

```ts
import { Module } from "@nestjs/common";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./entities/user.entity";
import { UsersService } from "./users.service";
import { UsersController } from "./users.controller";

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // عشان AuthModule يقدر يستخدمه
})
export class UsersModule {}
```

---

## 8) `auth/dto/register.dto.ts`

```ts
import { IsEmail, IsString, MinLength, MaxLength } from "class-validator";

export class RegisterDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @MinLength(8)
  @MaxLength(30)
  password: string;
}
```

---

## 9) `auth/dto/login.dto.ts`

```ts
import { IsEmail, IsString } from "class-validator";

export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  password: string;
}
```

---

## 10) `auth/auth.service.ts`

هنا الـ bcrypt hashing، وتوليد الـ JWT، ومنطق الـ login/register.

```ts
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";
import * as bcrypt from "bcryptjs";
import { UsersService } from "../users/users.service";
import { RegisterDto } from "./dto/register.dto";
import { LoginDto } from "./dto/login.dto";

@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService,
    private readonly jwtService: JwtService,
  ) {}

  async register(dto: RegisterDto) {
    const salt = await bcrypt.genSalt(10);
    const hashedPassword = await bcrypt.hash(dto.password, salt);

    const user = await this.usersService.create({
      ...dto,
      password: hashedPassword,
    });

    return this.generateToken(user.id, user.role);
  }

  async login(dto: LoginDto) {
    const user = await this.usersService.findByEmail(dto.email);
    if (!user) {
      throw new UnauthorizedException("Invalid credentials");
    }

    const matched = await bcrypt.compare(dto.password, user.password);
    if (!matched) {
      throw new UnauthorizedException("Invalid credentials");
    }

    return this.generateToken(user.id, user.role);
  }

  private generateToken(userId: string, role: string) {
    const payload = { sub: userId, role };
    return {
      accessToken: this.jwtService.sign(payload),
    };
  }
}
```

**ليه الرسالة دايماً "Invalid credentials"؟** عشان مينفعش المهاجم يعرف إن الإيميل موجود من عدمه — نفس الحاجة اللي اتكلمنا عنها قبل كده.

---

## 11) `auth/auth.controller.ts`

```ts
import { Body, Controller, Post } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { RegisterDto } from "./dto/register.dto";
import { LoginDto } from "./dto/login.dto";

@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post("register")
  register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @Post("login")
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }
}
```

---

## 12) `auth/guards/auth.guard.ts`

بيتأكد إن فيه JWT صحيح جوا الـ Header.

```ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private readonly jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(" ")[1];

    if (!token) {
      throw new UnauthorizedException();
    }

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }
}
```

---

## 13) `auth/decorators/roles.decorator.ts`

Custom decorator بيحط الـ metadata بتاعة الـ roles المطلوبة.

```ts
import { SetMetadata } from "@nestjs/common";

export const ROLES_KEY = "roles";
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

---

## 14) `auth/guards/roles.guard.ts`

بيتأكد إن الـ role بتاع اليوزر (من الـ JWT payload) موجود ضمن الـ roles المسموح بيها.

```ts
import { CanActivate, ExecutionContext, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { ROLES_KEY } from "../decorators/roles.decorator";

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true; // مفيش roles محددة يعني مفتوح لأي حد عنده token صحيح
    }

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.includes(user.role);
  }
}
```

**استخدامه:**

```ts
@UseGuards(AuthGuard, RolesGuard)
@Roles("ADMIN")
@Delete(":id")
remove(@Param("id") id: string) { ... }
```

---

## 15) `auth/auth.module.ts`

```ts
import { Module } from "@nestjs/common";
import { JwtModule } from "@nestjs/jwt";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { UsersModule } from "../users/users.module";
import { AuthService } from "./auth.service";
import { AuthController } from "./auth.controller";
import { AuthGuard } from "./guards/auth.guard";
import { RolesGuard } from "./guards/roles.guard";

@Module({
  imports: [
    UsersModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get<string>("JWT_SECRET"),
        signOptions: {
          expiresIn: config.get<string>("JWT_EXPIRES_IN"),
        },
      }),
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, AuthGuard, RolesGuard],
  exports: [AuthGuard, RolesGuard, JwtModule], // عشان UsersModule يستخدم AuthGuard
})
export class AuthModule {}
```

---

## 16) `app.module.ts`

الملف اللي بيجمع كل حاجة.

```ts
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";
import { UsersModule } from "./users/users.module";
import { AuthModule } from "./auth/auth.module";
import { User } from "./users/entities/user.entity";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ".env",
    }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: "postgres",
        host: config.get<string>("DB_HOST"),
        port: config.get<number>("DB_PORT"),
        username: config.get<string>("DB_USERNAME"),
        password: config.get<string>("DB_PASSWORD"),
        database: config.get<string>("DB_NAME"),
        entities: [User],
        synchronize: true, // للتطوير فقط! متستخدمهاش في production
      }),
    }),
    UsersModule,
    AuthModule,
  ],
})
export class AppModule {}
```

---

## 17) `main.ts`

هنا بنفعّل الـ ValidationPipe بشكل Global.

```ts
import { NestFactory } from "@nestjs/core";
import { ValidationPipe } from "@nestjs/common";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

---

## المكتبات المطلوبة (تجميعة واحدة)

```bash
npm install @nestjs/typeorm typeorm pg
npm install @nestjs/config
npm install @nestjs/jwt
npm install bcryptjs
npm install class-validator class-transformer
```

---

## خلاصة الـ Flow الكامل

```text
1. POST /auth/register
   → AuthController → AuthService → hash password → UsersService.create() → DB

2. POST /auth/login
   → AuthController → AuthService → compare password → generate JWT → return token

3. GET /users (protected)
   → AuthGuard يتحقق من الـ JWT → UsersController → UsersService → DB
```

كده عندك **UsersModule** (CRUD كامل + Entity + DTOs + Validation) و**AuthModule** (Register/Login + JWT + Guards + Roles) شغالين مع بعض بـ best practices — Controller نضيف، Service فيه كل الـ logic، DTO validation، وSecurity متكامل.

عايزني أضيفلك حاجة زيادة زي Refresh Token أو Passport Strategies، ولا كده كفاية كـ recap دلوقتي؟