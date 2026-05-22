# 03 — Relationships in NestJS + TypeORM

# مقدمة

الـ Relationships من أهم أجزاء أي Database Design.

لأن في الواقع:

```txt
البيانات مرتبطة ببعض
```

---

# أمثلة حقيقية

- User عنده Posts
    
- Post عليه Comments
    
- Product له Category
    
- Student داخل Courses
    
- Order يخص User
    

---

# لو معملتش Relationships صح

هيظهر عندك:

- Duplicate Data
    
- مشاكل Performance
    
- Data inconsistency
    
- Query complexity
    

---

# أنواع الـ Relationships

في الـ Relational Databases عندنا 4 أنواع أساسية:

|Type|المعنى|
|---|---|
|OneToOne|واحد لواحد|
|OneToMany|واحد لكثير|
|ManyToOne|كثير لواحد|
|ManyToMany|كثير لكثير|

---

# أهم نقطة تفهمها ⭐

الـ Relationship مش مجرد TypeORM Decorators…

هي انعكاس:

```txt
لعلاقة حقيقية بين البيانات
```

---

# 1) One To One

# يعني اي OneToOne ؟

كل Record مرتبط بـ Record واحد فقط.

---

# مثال حقيقي

كل User له:

```txt
Profile واحد
```

وكل Profile يخص:

```txt
User واحد فقط
```

---

# شكل العلاقة

```txt
User <-----> Profile
```

---

# Example

# user.entity.ts

```ts
@OneToOne(() => Profile)
@JoinColumn()
profile: Profile;
```

---

# profile.entity.ts

```ts
@OneToOne(() => User, (user) => user.profile)
user: User;
```

---

# مهم جدًا ⭐

# مين يحط JoinColumn ؟

الـ Owner Side فقط. 
Profile here is the inverse side 

---

# يعني اي Owner Side ؟

هو اللي يحتوي على:

```txt
Foreign Key
```

داخل الـ Database.

---

# الناتج في Database

```txt
users table
------------
profileId
```

---

# غالبًا تستخدم OneToOne في:

- User ↔ Profile
    
- User ↔ Wallet
    
- User ↔ Settings
    

---

# 2) OneToMany / ManyToOne ⭐⭐⭐⭐⭐

وده أهم Relationship فعلًا.

---

# يعني اي ؟

User واحد عنده Posts كثيرة.

لكن:

كل Post يخص User واحد فقط.

---

# الشكل

```txt
User -----> Posts
```

---

# مهم جدًا تفهم

## OneToMany دائمًا معها ManyToOne

---

# مثال واقعي

# User Entity

```ts
@OneToMany(
    () => Post,
    (post) => post.user,
)
posts: Post[];
```

---

# Post Entity

```ts
@ManyToOne(
    () => User,
    (user) => user.posts,
)
user: User;
```

---

# مين الـ Owner هنا ؟

✅ الـ ManyToOne دائمًا غالبًا هو الـ Owner.

---

# الناتج في Database

```txt
posts table
------------
userId
```

---

# لأن:

كل Post تشير إلى User واحد.

---

# مثال Production حقيقي

---

# User → Orders

```txt
User واحد
عنده Orders كثيرة
```

---

# Product → Reviews

```txt
Product واحد
عليه Reviews كثيرة
```

---

# Category → Products

```txt
Category واحدة
تحتوي Products كثيرة
```

---

# ليه العلاقة دي مهمة جدًا ؟

لأن أغلب الأنظمة تعتمد عليها.

---

# مثال E-commerce

```txt
User
Order
Product
Review
Comment
```

كلها OneToMany تقريبًا.

---

# 3) ManyToMany

# يعني اي ؟

كل طرف عنده مجموعة من الطرف الآخر.

---

# مثال

Student يدرس Courses كثيرة.

والـ Course فيها Students كثيرة.

---

# الشكل

```txt
Students <-----> Courses
```

---

# مثال

# student.entity.ts

```ts
@ManyToMany(() => Course)
@JoinTable()
courses: Course[];
```

---

# course.entity.ts

```ts
@ManyToMany(
    () => Student,
    (student) => student.courses,
)
students: Student[];
```

---

# مهم جدًا ⭐

# JoinTable تتحط في Side واحدة فقط

ودي الـ Owner Side.

---

# الناتج في Database

TypeORM تعمل Table وسيطة تلقائي:

```txt
student_courses
----------------
studentId
courseId
```

---

# ManyToMany خطيرة شوية ⚠️

في المشاريع الكبيرة ناس كتير تتجنبها.

---

# ليه ؟

لأنك غالبًا تحتاج بيانات إضافية.

---

# مثال

Student ↔ Course

قد تحتاج:

```txt
joinedAt
grade
status
```

---

# هنا الأفضل تعمل Entity وسيطة.

---

# مثال احترافي

```txt
Enrollment
```

---

# بدل ManyToMany مباشرة.

---

# JoinColumn vs JoinTable

---

# JoinColumn =>OneToMany

تستخدم مع:

```txt
OneToOne
ManyToOne
```

---

# لأنها تحدد:

```txt
Foreign Key column
```

---

# مثال

```ts
@JoinColumn()
profile: Profile;
```

---

# JoinTable

تستخدم فقط مع:

```txt
ManyToMany
```

---

# لأنها تنشئ:

```txt
Junction Table
```

---

# مثال

```ts
@JoinTable()
courses: Course[];
```

---

# eager vs lazy loading ⚠️

دي نقطة مهمة جدًا في الـ Performance.

---

# المشكلة الأساسية

لما تجيب User…

هل تجيب معه Posts تلقائي ؟

---

# Eager Loading

يعني:

```txt
Load relation automatically
```

---

# مثال

```ts
@ManyToOne(() => User, {
    eager: true,
})
user: User;
```

---

# لما تعمل:

```ts
find()
```

TypeORM تجيب:

- Post
    
- ومعه User تلقائي
    

---

# الميزة

سهل.

---

# العيب ⚠️

ممكن يعمل:

```txt
Huge unnecessary queries
```

ويبوظ الـ Performance.

---

# Lazy Loading

يعني:

```txt
Load relation only when needed
```

---

# مثال

```ts
posts: Promise<Post[]>;
```

---

# الاستخدام

```ts
const posts = await user.posts;
```

---

# المشكلة ؟

Lazy Loading في TypeORM أحيانًا يعمل:

```txt
N+1 Query Problem
```

---

# يعني اي N+1 Problem ؟

دي من أخطر مشاكل الـ ORM.

---

# مثال

جبت:

```txt
100 users
```

---

وبعدين لكل User:

```txt
جاب posts بتاعته
```

---

# النتيجة

```txt
101 Queries 😵
```

---

# وده يدمر الـ Performance.

---

# الحل الأفضل غالبًا ⭐

استخدم:

```txt
Explicit Relation Loading
```

---

# مثال

```ts
const users = await repo.find({
    relations: ['posts'],
});
```

---

# كدا Query واحدة فقط.

---

# Cascade Options

دي مهمة جدًا.

---

# يعني اي Cascade ؟

يعني:

```txt
لما أعمل عملية على Entity
تتطبق تلقائي على الـ Related Entities
```

---

# مثال

```ts
@OneToOne(() => Profile, {
    cascade: true,
})
profile: Profile;
```

---

# لما تعمل:

```ts
save(user)
```

الـ Profile تتعمل Save تلقائي.

---

# أنواع Cascade

|Type|الوظيفة|
|---|---|
|insert|insert تلقائي|
|update|update تلقائي|
|remove|delete تلقائي|
|soft-remove|soft delete تلقائي|

---

# مثال

```ts
cascade: ['insert']
```

---

# ⚠️ خطر مهم جدًا

ناس كتير تستخدم:

```ts
cascade: true
```

في كل حاجة.

وده خطر.

---

# ليه ؟

ممكن تعمل:

- Deletes غير مقصودة
    
- Data corruption
    
- Massive queries
    

---

# الأفضل

استخدم أقل Cascade ممكنة.

---

# Relation Loading Performance ⚠️

دي أهم نقطة تقريبًا.

---

# المشكلة

الـ ORM تخليك تنسى إن تحت فيه SQL.

---

# مثال خطر

```ts
relations: [
    'posts',
    'posts.comments',
    'posts.comments.user',
]
```

---

# النتيجة

```txt
Huge joins
Huge memory usage
Slow queries
```

---

# Production Rule ⭐

```txt
Load only what you need
```

---

# متجبش Relations زيادة.

---

# استخدم Pagination

---

# استخدم Select

---

# استخدم QueryBuilder عند الحاجة

---

# مثال احترافي

```ts
const users = await repo.find({
    select: ['id', 'fullName'],
    relations: ['posts'],
});
```

---

# Best Practices ⭐

---

# 1) استخدم ManyToOne كثير

هي الأكثر استخدامًا.

---

# 2) تجنب eager في الـ Large Relations

---

# 3) لا تستخدم cascade بعشوائية

---

# 4) فكر دائمًا في الـ SQL الناتج

---

# 5) لا تحمل Relations لا تحتاجها

---

# 6) استخدم explicit loading غالبًا

---

# 7) افهم الـ Owner Side كويس

---

# مثال Production كامل

# User Entity

```ts
@OneToMany(
    () => Order,
    (order) => order.user,
)
orders: Order[];
```

---

# Order Entity

```ts
@ManyToOne(
    () => User,
    (user) => user.orders,
)
@JoinColumn()
user: User;
```

---

# Database الناتجة

```txt
orders table
-------------
userId
```

---

# 🧠 أهم Mental Model

الـ Relationships ليست مجرد Decorators…

> هي طريقة تمثيل العلاقات الحقيقية بين البيانات داخل الـ Database.

---

# 🚀 الخلاصة

|العلاقة|المعنى|
|---|---|
|OneToOne|عنصر واحد ↔ عنصر واحد|
|OneToMany|عنصر واحد ↔ عناصر كثيرة|
|ManyToOne|عناصر كثيرة ↔ عنصر واحد|
|ManyToMany|عناصر كثيرة ↔ عناصر كثيرة|

---

# أهم نقطتين فعلًا ⭐

## 1) ManyToOne / OneToMany

الأكثر استخدامًا.

---

## 2) Relation Performance

أخطر جزء في الـ ORM.

---

# Golden Rule ⭐

> لا تحمل بيانات لا تحتاجها.