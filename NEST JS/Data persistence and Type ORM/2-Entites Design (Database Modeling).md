
# 02 — تصميم الـ Entities (Database Modeling) في NestJS

# مقدمة

واحدة من أهم الحاجات في أي Backend Engineer محترف هي:

```txt
تصميم الـ Database بشكل صحيح
```

لأن:

> أي خطأ في الـ Database Design  
> بيكلف غالي جدًا بعدين.

---

# يعني اي Entity ؟

الـ Entity هي:

> Representation للـ Table داخل الكود.

يعني:

|Database|NestJS / TypeORM|
|---|---|
|Table|Entity|
|Row|Object|
|Column|Property|

---

# مثال

## جدول في Database

```txt
users
```

---

## Entity في NestJS

```ts
@Entity()
export class User {

}
```

---

# ليه الـ Entities مهمة ؟

الـ Entity Design بيأثر على:

- Performance
    
- Scalability
    
- Maintainability
    
- Query Speed
    
- Data Integrity
    

---

# 🧱 قاعدة مهمة جدًا

# Entity per Module

كل Module يكون له Entity خاصة به.

---

# مثال

## Users Module

```txt
users.entity.ts
```

---

## Products Module

```txt
products.entity.ts
```

---

## Orders Module

```txt
orders.entity.ts
```

---

# ❌ غلط

تحط كل الـ Entities في Folder واحد كبير.

---

# ✅ الصح

كل Module تكون مستقلة.

---

# مثال Structure محترم

```txt
src/
│
├── users/
│   ├── entities/
│   │   └── user.entity.ts
│
├── products/
│   ├── entities/
│   │   └── product.entity.ts
│
├── orders/
│   ├── entities/
│   │   └── order.entity.ts
```

---

# Naming Conventions

الـ Naming مهمة جدًا.

---

# اسم الـ Entity

✅ استخدم:
single and starts with a capital letter 
```ts
User
Product
Order
```

---

❌ متستخدمش:

```ts
Users
ProductsTable
tbl_users
```

---

# اسم الـ Table

الأفضل غالبًا:

```txt
snake_case
```

---

## مثال

```ts
@Entity('users')
```

---

# ليه ؟

لأن أغلب الـ Databases والـ SQL Community تستخدم:

```txt
snake_case
```

---

# مثال كامل

```ts
@Entity('users')
export class User {

}
```

---

# Columns

كل Property في الـ Entity تتحول لـ Column.

---

# مثال

```ts
@Column()
name: string;
```

---

تتحول في Database إلى:

```sql
name VARCHAR(...)
```

---

# مثال حقيقي كامل

```ts
@Entity('users')
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    email: string;
}
```

---

# Column Options

ودي من أهم الحاجات جدًا.

---

# 1) unique

تمنع تكرار القيم.

---

# مثال

```ts
@Column({
    unique: true,
})
email: string;
```

---

# ليه ؟

عشان مينفعش:

```txt
2 users بنفس الإيميل
```

---

# Database Constraint

الـ Database نفسها تمنع التكرار.

وده مهم جدًا.

---

# ❌ غلط شائع

الاعتماد فقط على Validation.

---

# ليه غلط ؟

لأن أي حد ممكن يعدي الـ Validation.

لكن الـ Database Constraint لا يمكن تجاوزه.

---

# 2) nullable

هل الـ Column ممكن تكون فارغة ؟

---

# مثال

```ts
@Column({
    nullable: true,
})
bio: string;
```

---

# معناها

الـ User ممكن ميبقاش عنده Bio.

---

# ❌ لو nullable = false

يبقى لازم قيمة.

---

# 3) default

قيمة افتراضية.

---

# مثال

```ts
@Column({
    default: true,
})
isActive: boolean;
```

---

أي User جديد:

```txt
isActive = true
```

تلقائيًا.

---

# مثال مهم جدًا

```ts
@Column({
    default: 0,
})
balance: number;
```

---

# 4) enum

لما يكون عندك قيم محددة فقط.

---

# مثال

```ts
export enum UserRole {
    ADMIN = 'admin',
    USER = 'user',
}
```

---

# استخدامه

```ts
@Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
})
role: UserRole;
```

---

# ليه الـ enum مهمة ؟

بدل ما أي حد يدخل:

```txt
"superman"
```

كـ Role 😅

---

# Database تمنع القيم الغلط.

---

# 5) Indexes ⭐

دي من أهم الحاجات في الـ Scalability.

---

# يعني اي Index ؟

الـ Index زي:

```txt
فهرس الكتاب
```

بدل ما الـ Database تفتش في كل الصفحات.

---

# بدون Index

الـ Database تعمل:

```txt
Full Table Scan
```

وده بطيء جدًا.

---

# باستخدام Index

الوصول أسرع بكتير.

---

# مثال

```ts
@Column({
    unique: true,
})
email: string;
```

الـ unique غالبًا يعمل Index تلقائي.

---

# Manual Index

```ts
@Index()
@Column()
username: string;
```

---

# Composite Index

```ts
@Index(['firstName', 'lastName'])
```

---

# امتى نستخدم Index ؟

على الأعمدة اللي بيتعمل عليها:

- Search
    
- Filtering
    
- Sorting
    
- Joins
    

---

# مثال

```txt
email
username
createdAt
userId
```

---

# ❌ خطر مهم

الـ Indexes الكثيرة جدًا:

- تبطّأ الـ INSERT
    
- تبطّأ الـ UPDATE
    
- تستهلك Storage
    

---

# استخدمها بذكاء.

---

# Audit Columns

دي أعمدة مهمة جدًا في أي Production App.

---

# 1) createdAt

وقت إنشاء الـ Record.

---

# مثال

```ts
@CreateDateColumn()
createdAt: Date;
```

---

TypeORM يملأها تلقائيًا.

---

# 2) updatedAt

آخر تعديل.

---

# مثال

```ts
@UpdateDateColumn()
updatedAt: Date;
```

---

كل Update:

```txt
updatedAt تتغير تلقائي
```

---

# 3) deletedAt (Soft Delete)

دي مهمة جدًا.

---

# يعني اي Soft Delete ؟

بدل ما تمسح البيانات فعلًا:

```txt
نخليها "محذوفة منطقيًا"
```

---

# مثال

```ts
@DeleteDateColumn()
deletedAt: Date;
```

---

# لما تعمل Delete

TypeORM تعمل:

```txt
deletedAt = current timestamp
```

بدل حذف الـ Row.

---

# ليه Soft Delete مهم ؟

عشان:

- Recovery
    
- Audit
    
- Analytics
    
- منع فقدان البيانات
    

---

# مثال واقعي

تخيل User حذف Account بالغلط.

لو Hard Delete:

```txt
الداتا راحت
```

---

لكن Soft Delete:

```txt
نقدر نرجعها
```

---

# مثال Entity احترافي كامل

```ts
//لازم تعمل امبورت لل decroators  اللي هتستخدمها 
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    CreateDateColumn,
    UpdateDateColumn,
    DeleteDateColumn,
    Index,
} from 'typeorm';

export enum UserRole {
    ADMIN = 'admin',
    USER = 'user',
}

@Entity('users')
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Index()
    @Column({
        unique: true,
    })
    email: string;

    @Column()
    name: string;

    @Column({
        nullable: true,
    })
    bio: string;

    @Column({
        default: true,
    })
    isActive: boolean;

    @Column({
        type: 'enum',
        enum: UserRole,
        default: UserRole.USER,
    })
    role: UserRole;

    @CreateDateColumn()
    createdAt: Date;

    @UpdateDateColumn()
    updatedAt: Date;

    @DeleteDateColumn()
    deletedAt: Date;
}
```

---

# Clean Code Tips

# 1) متحطش Business Logic في Entity

❌ غلط

```ts
calculateSalary() {}
sendEmail() {}
```

---

الـ Entity للـ Data فقط.

---

# 2) استخدم enum بدل strings العشوائية

---

# 3) اعمل Index للأعمدة المهمة فقط

---

# 4) استخدم Soft Delete في أغلب الأنظمة

---

# 5) خليك consistent في الـ Naming

---

# Common Mistakes

---

# ❌ استخدام synchronize: true في Production

ممكن يبوظ الـ Schema.

---

# ❌ عدم استخدام indexes

يسبب بطء رهيب مع البيانات الكبيرة.

---

# ❌ nullable زيادة عن اللزوم

يسبب Data غير منظمة.

---

# ❌ عدم وجود Audit Columns

يصعب الـ Debugging والـ Tracking.

---

# 🧠 أهم فكرة

الـ Database Design مش مجرد Tables…

> دي أساس الـ Scalability والأداء والاستقرار.

---

# 🚀 الخلاصة

## Entity

تمثل Table في الـ Database.

---

# أهم Column Options

|Option|وظيفتها|
|---|---|
|unique|منع التكرار|
|nullable|السماح بالقيمة الفارغة|
|default|قيمة افتراضية|
|enum|قيم محددة فقط|
|index|تسريع البحث|

---

# Audit Columns

|Column|فائدتها|
|---|---|
|createdAt|وقت الإنشاء|
|updatedAt|وقت التعديل|
|deletedAt|Soft Delete|

---

# أهم قاعدة

> الـ Database Design الجيد  
> يوفر عليك مشاكل ضخمة مستقبلًا.