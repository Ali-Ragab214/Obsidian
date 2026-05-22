تمام، أنت كدا ماشي بالترتيب الصح فعلًا. بعد ما فهمت:

- DB Config
    
- Entity Modeling
    
- Relationships
    
- Repository Pattern
    

فالخطوة دي مهمة جدًا لأن أغلب الناس تستخدم CRUD يوميًا لكن تستخدمه بطريقة غلط 😄

هكتبها لك بشكل مرتب تقدر ترميه مباشرة في Obsidian.

# 05 — CRUD Best Practices (NestJS + TypeORM)

> مجرد إن الكود يشتغل لا يعني إنه مكتوب صح.

في TypeORM عندنا Methods كتير شكلها متشابه:

```ts
save()
update()

remove()
delete()

create()
preload()
```

لكن الفرق بينهم مهم جدًا في Production.

---

# 1) save() vs update()

ناس كتير تعتبرهم نفس الحاجة.

لكن هم مختلفين جدًا.

---

# save()

## وظيفته

```txt
Insert أو Update حسب وجود الـ Entity
```

يعني:

لو الـ id غير موجود:

```sql
INSERT
```

لو الـ id موجود:

```sql
UPDATE
```

---

## مثال

```ts
const user = this.userRepository.create({

   fullName:"Ali",
   email:"ali@gmail.com"

});

await this.userRepository.save(
    user
);
```

---

هيعمل:

```sql
INSERT INTO users
```

---

ولو:

```ts
const user = await this.userRepository.findOne({

   where:{
      id:1
   }

});

user.fullName="Ahmed";

await this.userRepository.save(
   user
);
```

هيعمل:

```sql
UPDATE users
SET fullName='Ahmed'
WHERE id=1
```

---

## مميزات save()

✅ يشغل Entity hooks

مثل:

```ts
@BeforeInsert()

@BeforeUpdate()
```

---

✅ يشغل Cascade

---

✅ يرجع Entity كاملة

---

## عيوب save() ⚠️

في بعض الحالات يعمل:

```txt
Query زيادة
```

مثال:

```ts
await repository.save({
    id:1,
    fullName:'Ali'
});
```

TypeORM أحيانًا:

1. تعمل SELECT
    
2. ثم UPDATE
    

بدل Query واحدة.

---

---

# update()

## وظيفته

```txt
Update مباشر داخل Database
```

---

## مثال

```ts
await this.userRepository.update(

    1,

    {
       fullName:"Ali"
    }

);
```

---

هيعمل:

```sql
UPDATE users
SET fullName='Ali'
WHERE id=1
```

---

## مميزات update()

✅ أسرع

✅ Query واحدة

---

## عيوب update()

❌ لا يشغل hooks

❌ لا يشغل cascade

❌ لا يرجع Entity

---

## مثال خطر

لو عندك:

```ts
@BeforeUpdate()

hashPassword(){

}
```

---

وعملت:

```ts
await repository.update(
    id,
    dto
);
```

الـ hook لن يعمل.

---

قد يخزن Password بدون hashing 😵

---

# Production Rule ⭐

استخدم:

### save()

لما:

- تحتاج Entity كاملة
    
- عندك Hooks
    
- عندك Cascade
    

---

استخدم:

### update()

للتعديلات البسيطة والسريعة.

---

# 2) remove() vs delete()

برضو ناس كتير تلخبط بينهم.

---

# remove()

## وظيفته

يحذف Entity نفسها.

---

## مثال

```ts
const user =
await this.userRepository.findOne({

    where:{
       id:1
    }

});

await this.userRepository.remove(
    user
);
```

---

غالبًا:

1. يعمل SELECT
    
2. ثم DELETE
    

---

## مميزات remove()

✅ يشغل hooks

مثل:

```ts
@BeforeRemove()
```

---

✅ يشغل Cascade

---

## عيوبه

❌ أبطأ

---

# delete()

## وظيفته

يحذف مباشرة باستخدام شرط.

---

## مثال

```ts
await this.userRepository.delete(
    1
);
```

---

هيعمل:

```sql
DELETE FROM users
WHERE id=1
```

---

## مميزاته

✅ سريع

✅ Query واحدة

---

## عيوبه

❌ لا يشغل hooks

❌ لا يشغل Cascade

---

# Production Rule ⭐

استخدم:

```ts
delete()
```

في:

- Admin tools
    
- Bulk operations
    

---

استخدم:

```ts
remove()
```

لو محتاج:

- Hooks
    
- Cascade
    
- Business behavior
    

---

# Soft Delete

في Production غالبًا لا نحذف فعليًا.

---

Entity:

```ts
@DeleteDateColumn()
deletedAt:Date;
```

---

الحذف:

```ts
await repository.softDelete(
    id
);
```

---

بدل:

```sql
DELETE
```

سيصبح:

```sql
UPDATE users
SET deletedAt=NOW()
```

---

---

# 3) create() vs preload()

دي أكتر نقطة بتلخبط الناس.

---

# create()

## وظيفته

ينشئ Entity داخل Memory فقط.

لا يحفظ Database.

---

## مثال

```ts
const user = this.userRepository.create({

    fullName:"Ali",
    email:"ali@gmail.com"

});
```

---

ما زال:

```txt
لم يتم حفظ شيء
```

---

لازم:

```ts
await this.userRepository.save(
   user
);
```

---

# preload()

دي قوية جدًا.

---

## وظيفتها

تجيب Entity موجودة وتدمج معها بيانات جديدة.

---

## مثال

نفترض DB:

```txt
id:1
fullName:Ali
email:ali@gmail.com
```

---

جاء Request:

```ts
{

    id:1,
    fullName:"Ahmed"

}
```

---

نعمل:

```ts
const user =
await this.userRepository.preload({

    id:1,

    fullName:"Ahmed"

});
```

---

TypeORM تعمل داخليًا:

```txt
1) تجيب User من DB
2) تدمج البيانات الجديدة
3) ترجع Entity جاهزة
```

---

الناتج:

```ts
{

    id:1,
    fullName:"Ahmed",
    email:"ali@gmail.com"

}
```

---

ثم:

```ts
await this.userRepository.save(
    user
);
```

---

## لو الـ id غير موجود ؟

ترجع:

```ts
null
```

---

# استخدام Production حقيقي

بدل:

```ts
const user =
await repository.findOne(...);

user.fullName=dto.fullName;

await repository.save(user);
```

---

ممكن:

```ts
const user =
await repository.preload({

   id,

   ...dto

});

if(!user){

   throw new NotFoundException();

}

await repository.save(
   user
);
```

---

# مقارنة سريعة

|Method|وظيفة|
|---|---|
|create|إنشاء object داخل memory|
|preload|تحميل Entity ودمج البيانات|
|save|Insert أو Update|
|update|Update مباشر|
|remove|حذف مع hooks|
|delete|حذف مباشر|

---

# قواعد Production المهمة ⭐

### لا تستخدم save في كل حاجة

---

### استخدم update للتعديلات البسيطة

---

### استخدم preload للتحديثات

---

### استخدم softDelete بدل delete غالبًا

---

### لا تستخدم remove في Bulk Operations

---

### انتبه للـ hooks

---

# مثال Production كامل

```ts
async updateUser(

   id:number,
   dto:UpdateUserDto

){

   const user =
   await this.userRepository.preload({

      id,
      ...dto

   });

   if(!user){

      throw new NotFoundException(
          'User not found'
      );

   }

   return await this.userRepository.save(
      user
   );

}
```

---

# 🧠 Mental Model

```txt
create → ابنِ object

preload → حمّل وادمج

save → احفظ

update → عدل بسرعة

remove → احذف مع behavior

delete → احذف مباشرة
```

ودي من أكتر الحاجات اللي بتبان في مراجعات الكود وفي الانترفيوهات مع NestJS وTypeORM.