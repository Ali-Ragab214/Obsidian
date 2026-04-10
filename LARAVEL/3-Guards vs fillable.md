حلو السؤال ده 👌 لأنه من أهم مفاهيم الأمان في Laravel.

خلينا نفهم الأول المشكلة اللي Laravel بيحلّها.

---

# ما المشكلة أصلاً؟ (Mass Assignment)

تخيل إن عندك Model User فيه:

```php
id
name
email
password
is_admin
```

لو كتبت:

```php
User::create($request->all());
```

والمستخدم بعت في الـ request:

```json
{
 "name": "Ali",
 "email": "ali@mail.com",
 "password": "123",
 "is_admin": true
}
```

💣 كده المستخدم قدر يخلي نفسه Admin !

وهذا يسمى:

## ⚠️ Mass Assignment Vulnerability

Laravel يمنع ده افتراضياً عن طريق:

- fillable
    
- guarded
    

---

# ما هو $fillable ؟

## تعريف

هو **Whitelist**  
يعني: الحقول المسموح فقط بتعبئتها تلقائياً.

أي حقل مش موجود = Laravel يرفضه.

---

## مثال عملي

### User Model

```php
class User extends Model
{
    protected $fillable = [
        'name',
        'email',
        'password'
    ];
}
```

### إنشاء مستخدم

```php
User::create($request->all());
```

لو الـ request يحتوي:

```json
{
 "name": "Ali",
 "email": "ali@mail.com",
 "password": "123",
 "is_admin": true
}
```

Laravel سيحفظ فقط:

```text
name ✔
email ✔
password ✔
is_admin ❌ مرفوض
```

🔥 كده حميت نفسك.

---

# ما هو $guarded ؟

العكس تماماً.

## تعريف

هو **Blacklist**  
يعني: الحقول الممنوعة فقط.

---

## مثال

```php
class User extends Model
{
    protected $guarded = ['is_admin'];
}
```

هنا Laravel يسمح بكل الحقول  
ما عدا `is_admin`.

---

# الفرق ببساطة

|fillable|guarded|
|---|---|
|يسمح فقط بالحقول المحددة|يمنع فقط الحقول المحددة|
|Whitelist|Blacklist|
|الأكثر أماناً|أقل أماناً|

---

# مثال كامل باستخدام Product

## Product Model

```php
class Product extends Model
{
    protected $fillable = [
        'name',
        'description',
        'price',
        'user_id'
    ];
}
```

## Controller

```php
Product::create([
    'name' => 'Laptop',
    'description' => 'Gaming laptop',
    'price' => 2000,
    'user_id' => auth()->id()
]);
```

يعمل بشكل طبيعي ✔

---

# مثال خطأ شائع 💣

لو نسيت fillable:

```php
class Product extends Model
{
}
```

ثم:

```php
Product::create($request->all());
```

Laravel سيرمي Error:

```
MassAssignmentException
```

وهذا مقصود للحماية.

---

# متى أستخدم guarded = [] ؟

لو كتبت:

```php
protected $guarded = [];
```

يعني:

> اسمح بكل الحقول 😈

يستخدم فقط في:

- مشاريع صغيرة
    
- Testing
    
- Seeder
    

🚫 لا يُنصح به في production.

---

# قاعدة مهمة تحفظها

💡 في المشاريع الحقيقية استخدم:

```
fillable دائماً
```

لأنه:

- أوضح
    
- أكثر أماناً
    
- أسهل في المراجعة
    

---

# خلاصة سريعة

- Laravel يمنع Mass Assignment تلقائياً
    
- fillable = الحقول المسموحة
    
- guarded = الحقول الممنوعة
    
- الأفضل دائماً استخدام fillable
    

---

لو حابب نشرح بعده موضوع **Mutators & Accessors** لأنه بيجي معاه دايماً قولّي 😎