بص يا علي 👌 خليني أشرحلك الـ **Tinker** بطريقة بسيطة و عملية كدا 👇

---

php artisan migrate:fresh --seed
# 🧠 ايه هو Tinker أصلاً؟

**Tinker** هو أداة في Laravel بتخليك تكتب كود PHP / Laravel **وتشغله مباشرة** من غير ما تعمل:

- Controller
    
- Route
    
- View
    

يعني كأنك عندك **console بتجرب فيها الكود بسرعة ⚡**

---

# 🎯 بستخدمه ليه؟

## ✅ 1. أجرب الكود بسرعة

بدل ما تعمل API أو صفحة

```bash
php artisan tinker
```

وبعدين:

```php
Person::all();
```

💥 هيرجعلك كل الداتا

---

## ✅ 2. أجرب Factory

```php
Person::factory()->count(10)->create();
```

👉 هيضيف 10 records في الداتابيز فورًا

---

## ✅ 3. أضيف / أعدل / أمسح داتا

### ➕ إضافة:

```php
Person::create([
    'name' => 'Ali',
    'age' => 25,
    'salary' => 5000,
    'adress' => 'Cairo',
    'email' => 'ali@gmail.com',
    'password' => bcrypt('12345678')
]);
```

---

### ✏️ تعديل:

```php
$p = Person::find(1);
$p->name = 'Ahmed';
$p->save();
```

---

### ❌ حذف:

```php
Person::find(1)->delete();
```

---

## ✅ 4. أختبر العلاقات (بعد كدا)

لما تتعلم relationships

---

# 🔥 ليه Tinker مهم؟

- سريع جدًا ⚡
    
- مش محتاج UI
    
- مناسب للتجارب
    
- بيستخدم نفس Laravel (Models / DB / Factory)
    

---

# ❗ إمتى تستخدمه؟

✔ لما:

- تجرب حاجة بسرعة
    
- تتأكد إن الكود شغال
    
- تولد داتا
    

❌ متستخدموش لما:

- تبني feature حقيقي
    
- تعمل API أو UI
    

---

# 🧪 مثال عملي سريع

```bash
php artisan tinker
```

```php
Person::factory()->count(3)->create();
Person::all();
```

💥 كدا عملت داتا وشوفتها

---

# 🧠 تخيله كدا

Tinker = **Playground** بتاع Laravel 🎮

---

لو عايز أقولك شوية tricks جامدة في Tinker (زي debug أو logs أو testing العلاقات) قولي وأنا أديك level أعلى 🔥