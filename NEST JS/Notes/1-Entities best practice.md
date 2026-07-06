سؤال ممتاز، ودي من الحاجات اللي بتفرق بين واحد بيعرف NestJS وبين واحد بيكتب كود Production.

هشرحلك **Best Practices لإنشاء Entities في NestJS + TypeORM** من الصفر، وهعتبر إننا شغالين في شركة كبيرة.

---

# Best Practices for Creating Entities in NestJS + TypeORM

---

# أولاً... يعني إيه Entity؟

الـ Entity هي تمثيل للـ Table الموجودة في قاعدة البيانات.

مثلاً

```
users table
```

هيبقى عندك

```ts
User Entity
```

---

# Folder Structure

أفضل تنظيم

```
src
|
|--modules
      |
      |--users
             |
             |--entities
             |      user.entity.ts
             |
             |--dto
             |
             |--services
             |
             |--controllers
```

كل Module يبقى فيه الـ Entity الخاصة بيه.

---

# أول حاجة بنعملها

بدل ما نكرر

```
id

createdAt

updatedAt

deletedAt
```

في كل Entity...

بننشئ

```
BaseEntity
```

---

# Base Entity

```ts
import {
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
} from 'typeorm';

export abstract class BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @CreateDateColumn({
    type: 'timestamp',
  })
  createdAt: Date;

  @UpdateDateColumn({
    type: 'timestamp',
  })
  updatedAt: Date;

  @DeleteDateColumn({
    nullable: true,
  })
  deletedAt?: Date;
}
```

---

## لماذا abstract ؟

لأننا مش هنعمل

```
new BaseEntity()
```

هي مجرد Parent.

---

# بعد كده

نعمل User Entity

---

# User Entity

```ts
import {
  Entity,
  Column,
  OneToMany,
  Index,
} from 'typeorm';

import { BaseEntity } from '../../common/entities/base.entity';
import { Post } from '../../posts/entities/post.entity';

@Entity({
  name: 'users',
})
export class User extends BaseEntity {

  @Column({
    length: 100,
  })
  name: string;

  @Index()
  @Column({
    unique: true,
    length: 255,
  })
  email: string;

  @Column({
    select: false,
  })
  password: string;

  @Column({
    default: true,
  })
  isActive: boolean;

  @OneToMany(
    () => Post,
    post => post.user,
  )
  posts: Post[];
}
```

---

# ليه استخدمنا

```
select:false
```

؟

لأن

```
password
```

مش المفروض يطلع مع أي Query.

بدل

```
SELECT *
```

هيبقى

```
password
```

مستخبي.

ولو محتاجه

```
.addSelect("user.password")
```

---

# Index

```ts
@Index()
```

بيعمل Index على العمود.

مثالي للبحث.

```
email

phone

username
```

---

# Post Entity

```ts
import {
  Entity,
  Column,
  ManyToOne,
  JoinColumn,
} from 'typeorm';

import { BaseEntity } from '../../common/entities/base.entity';
import { User } from '../../users/entities/user.entity';

@Entity({
  name: 'posts',
})
export class Post extends BaseEntity {

  @Column({
    length: 200,
  })
  title: string;

  @Column({
    type: 'text',
  })
  content: string;

  @ManyToOne(
    () => User,
    user => user.posts,
    {
      onDelete: 'CASCADE',
    },
  )
  @JoinColumn({
    name: 'userId',
  })
  user: User;
}
```

---

# لو عندنا Category

```ts
@Entity("categories")
export class Category extends BaseEntity{

   @Column({
      unique:true,
      length:100
   })
   name:string;

}
```

---

# العلاقات

## One To Many

```
User

1

↓

Many

Posts
```

```
User
    |
    |------ Post
    |------ Post
    |------ Post
```

---

## Many To One

```
Post

belongs to

User
```

---

# أنواع Columns

## varchar

```
@Column()
```

---

## text

```ts
@Column({
type:"text"
})
```

للنصوص الطويلة.

---

## boolean

```ts
@Column({
default:true
})
```

---

## enum

```ts
export enum Role{

ADMIN,

USER,

MANAGER

}
```

ثم

```ts
@Column({
type:'enum',
enum:Role,
default:Role.USER
})
role:Role;
```

---

## decimal

```ts
@Column({
type:"decimal",
precision:10,
scale:2
})
price:number;
```

---

## uuid

```ts
@PrimaryGeneratedColumn("uuid")
id:string;
```

لو النظام Microservices.

---

# Nullable

بدل

```
NULL
```

في الداتابيز

```ts
@Column({
nullable:true
})
image?:string;
```

---

# Default

```ts
@Column({
default:false
})
```

---

# Unique

```ts
@Column({
unique:true
})
```

---

# Length

```ts
@Column({
length:100
})
```

---

# select:false

مهم جداً

```
password

refreshToken

otp

verificationCode
```

---

# DeleteDateColumn

لو عملت

```ts
repository.softDelete(id)
```

مش هيتمسح.

هيكتب

```
deletedAt

2026-07-05
```

وده اسمه

```
Soft Delete
```

---

# Cascade

```ts
@ManyToOne(
()=>User,
{
onDelete:"CASCADE"
}
)
```

لو المستخدم اتحذف

↓

البوستات تتحذف.

---

# Naming Convention

استخدم:

```
createdAt

updatedAt

deletedAt

isActive

firstName

lastName
```

بدلاً من:

```
created_date

is_active

fname
```

---

# لا تضع Business Logic داخل الـ Entity

❌

```ts
class User{

login(){

...

}

}
```

---

✔

الـ Entity تحتوي فقط على:

- Properties
    
- Relations
    
- Database Mapping
    

والـ Business Logic يكون داخل:

```
Service
```

---

# لا تضع Validation داخل الـ Entity

❌

```ts
@IsEmail()
```

داخل الـ Entity.

التحقق يكون في الـ DTO باستخدام `class-validator`:

```ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @Length(8, 50)
  password: string;
}
```

---

# استخدم Enums للقيم الثابتة

بدلاً من:

```ts
status: string;
```

استخدم:

```ts
export enum UserStatus {
  ACTIVE = 'ACTIVE',
  INACTIVE = 'INACTIVE',
  BLOCKED = 'BLOCKED',
}

@Column({
  type: 'enum',
  enum: UserStatus,
  default: UserStatus.ACTIVE,
})
status: UserStatus;
```

---

# تجنب العلاقات Eager إلا عند الحاجة

بدلاً من:

```ts
@ManyToOne(() => User, {
  eager: true,
})
```

يفضل:

```ts
@ManyToOne(() => User)
```

ثم تحميل العلاقة عند الحاجة:

```ts
find({
  relations: {
    user: true,
  },
});
```

لأن `eager: true` قد يؤدي إلى تحميل بيانات غير ضرورية ويؤثر على الأداء.

---

# خلاصة أفضل الممارسات

- ✅ أنشئ `BaseEntity` مشترك يحتوي على `id`, `createdAt`, `updatedAt`, و`deletedAt`.
    
- ✅ اجعل الـ Entity مسؤولة فقط عن تمثيل جدول قاعدة البيانات.
    
- ✅ استخدم `select: false` للحقول الحساسة مثل كلمة المرور.
    
- ✅ استخدم `@Index()` على الأعمدة كثيرة البحث.
    
- ✅ استخدم `enum` بدلاً من `string` للقيم المحددة.
    
- ✅ استخدم `DeleteDateColumn` مع `softDelete`.
    
- ✅ استخدم `@JoinColumn` لتسمية مفاتيح العلاقات بوضوح.
    
- ✅ لا تضع Validation داخل الـ Entity؛ استخدم DTOs.
    
- ✅ لا تضع Business Logic داخل الـ Entity؛ ضعه في الـ Service.
    
- ✅ تجنب `eager: true` إلا إذا كنت متأكدًا أنك تحتاجه دائمًا.
    

هذا الأسلوب هو الأقرب لما ستراه في مشاريع NestJS الاحترافية داخل الشركات، لأنه يجعل الكود قابلًا للتوسع، وسهل الاختبار، ويحافظ على فصل المسؤوليات (Separation of Concerns).