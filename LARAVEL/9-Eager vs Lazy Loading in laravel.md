سؤال ممتاز 👌 ده من أهم مفاهيم Eloquent.

الكود بتاعك:

```php
public function index()
{
    return Product::with('user')->get();
}
```

ده اسمه **Eager Loading** ✅

خلينا نفهم الفرق بالكامل.

---

# أولاً: يعني اي Loading أصلاً؟

عندنا علاقة:

- Product belongsTo User
    

يعني كل Product له User صاحب المنتج.

السؤال:

> لما أجيب المنتجات… امتى Laravel يجيب الـ User؟

في طريقتين:

- Lazy Loading
    
- Eager Loading
    

---

# 1️⃣ Lazy Loading (التحميل الكسول)

يعني:

> هات الـ Products الأول… ولما أحتاج الـ User هاته بعدين.

### مثال

```php
$products = Product::all();

foreach ($products as $product) {
    echo $product->user->name;
}
```

ايه اللي حصل هنا؟ 🤔

## الاستعلامات اللي اتنفذت:

1️⃣ أول Query:

```sql
select * from products;
```

2️⃣ كل مرة تطلب user:

```sql
select * from users where id = ?
```

لو عندك 10 منتجات = 11 Query 😱

اسم المشكلة:

## 🔥 N + 1 Problem

يعني:

- Query واحدة للـ products
    
- N Query للـ users
    

كارثة في الأداء.

---

# 2️⃣ Eager Loading (التحميل المسبق)

يعني:

> هات المنتجات + المستخدمين مرة واحدة.

### الكود

```php
Product::with('user')->get();
```

## الاستعلامات:

1️⃣ Query:

```sql
select * from products;
```

2️⃣ Query:

```sql
select * from users where id in (1,2,3,4);
```

بس كده 👍 Queryين فقط.

وهذا فرق ضخم في الأداء.

---

# الفرق بشكل بصري

## Lazy Loading

```
Get Products  →  Loop → Query User → Query User → Query User
```

## Eager Loading

```
Get Products + Get Users → Done
```

---

# متى يحدث Lazy تلقائياً؟

لو كتبت:

```php
$product->user
```

بدون `with()` → Laravel يعمل Lazy Loading تلقائي.

---

# أنواع Eager Loading مهمة

## 1) Basic

```php
Product::with('user')->get();
```

## 2) Multiple relations

```php
Product::with(['user','category'])->get();
```

## 3) Select columns

```php
Product::with('user:id,name')->get();
```

أفضل للأداء 🔥

---

# متى أستخدم كل واحد؟

|الحالة|استخدم|
|---|---|
|علاقة واحدة محتاجها مرة واحدة|Lazy عادي|
|List أو API أو Loop|Eager لازم|
|Dashboard / Admin|Eager دائمًا|

قاعدة مهمة:

> أي Loop فيه علاقات = استخدم Eager Loading.

---

# خلاصة سريعة

- `Product::all()` + `$product->user` → Lazy ❌
    
- `Product::with('user')->get()` → Eager ✅
    
- Eager يحل مشكلة N+1 ويحسن الأداء جداً
    

---

لو حابب أشرح **Lazy Eager Loading** و `load()` و `loadMissing()` كمرحلة تانية قولّي 😉