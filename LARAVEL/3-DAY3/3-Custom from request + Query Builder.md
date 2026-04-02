حلو جدًا يا علي 👌 سؤالك في حتة مهمة جدًا فعلًا وناس كتير بتتلخبط فيها  
وإحساسك إنه شبه **DTO** مظبوط 🔥

خليني أرتبلك الموضوع كـ **Topic كامل** كأنك بتذاكره 👇

---

# 🧠 📌 Topic:

# **Custom Form Request + Query Builder في Laravel**

---

# 🥇 أولًا: يعني ايه Custom Form Request؟

## 🎯 التعريف

هو Class بتعمله عشان:

- تعمل **Validation**
    
- تفصل اللوجيك عن الـ Controller
    

---

## 💡 تشبيهه بـ DTO

✔ شبه الـ DTO لأنه:

- بيستقبل الداتا
    
- بيعمل Validation عليها
    

❌ بس مش DTO حقيقي لأنه:

- مش بيعمل mapping
    
- بس validation + authorization
    

---

# 🏗️ تعمل Custom Request ازاي؟

## 1️⃣ تنشئه

```bash
php artisan make:request StoreOrderRequest
```

---

## 2️⃣ تعدل فيه

```php
public function rules(): array
{
    return [
        'customer_name' => 'required|string|min:3',
        'status' => 'required|in:pending,completed',

        'items' => 'required|array',

        'items.*.product_name' => 'required|string',
        'items.*.quantity' => 'required|integer|min:1',
        'items.*.price' => 'required|numeric|min:1',
    ];
}
```

---

## 🧠 دي معناها ايه؟

```php
items.*.product_name
```

💥 يعني:

> كل item جوه الـ array لازم يكون فيه product_name

---

# 🔗 تربطه بالـ Controller ازاي؟

بدل:

```php
public function store(Request $request)
```

---

## ✅ تخليه كدا:

```php
public function store(StoreOrderRequest $request)
```

---

## 🎯 كدا Laravel هيعمل:

1. Validation تلقائي
    
2. لو في error → يرجع response
    
3. لو صح → يدخل الفنكشن
    

---

## 🧠 تجيب الداتا ازاي؟

```php
$data = $request->validated();
$data => دي الداتا اللي حققت الفاليديشن 
```

---

# 🚀 ثانيًا: يعني ايه Query Builder؟

## 🎯 التعريف

هو طريقة للتعامل مع الداتابيز **من غير Models**

---

## 📊 الفرق

|Eloquent|Query Builder|
|---|---|
|بيعتمد على Model|بيعتمد على DB|
|أسهل|أسرع|
|OOP|SQL-like|

---

# 🧠 مثال

## ❌ Eloquent

```php
Order::all();
```

---

## ✅ Query Builder

```php
DB::table('orders')->get();
```

---

# 🔥 نستخدمه ازاي في Controller؟

## 🧾 سيناريو: إنشاء Order + Items

---

## ✨ الكود الكامل

```php
use Illuminate\Support\Facades\DB;
use App\Http\Requests\StoreOrderRequest;

public function store(StoreOrderRequest $request)
{
    $data = $request->validated();

    // 🧮 نحسب total
    $total = 0;

    foreach ($data['items'] as $item) {
        $total += $item['quantity'] * $item['price'];
    }

    // 🧾 نضيف order
    $orderId = DB::table('orders')->insertGetId([
        'customer_name' => $data['customer_name'],
        'status' => $data['status'],
        'total_price' => $total,
        'created_at' => now(),
        'updated_at' => now(),
    ]);

    // 📦 نضيف items
    foreach ($data['items'] as $item) {
        DB::table('order_items')->insert([
            'order_id' => $orderId,
            'product_name' => $item['product_name'],
            'quantity' => $item['quantity'],
            'price' => $item['price'],
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }

    return response()->json([
        'message' => 'Order created successfully'
    ]);
}
```

---

# 🔥 ليه استخدمنا Query Builder هنا؟

✔ أسرع  
✔ تحكم أكبر  
✔ مناسب في bulk insert

---

# 🧠 شكل الداتا اللي بتيجي

```json
{
  "customer_name": "Ali",
  "status": "pending",
  "items": [
    {
      "product_name": "Phone",
      "quantity": 2,
      "price": 5000
    },
    {
      "product_name": "AirPods",
      "quantity": 1,
      "price": 2000
    }
  ]
}
```

---

# ⚠️ نصايح مهمة

## ✅ استخدم Transaction (مهم جدًا 🔥)

```php
DB::transaction(function () use ($data) {
    // insert order + items
});
```

💥 لو حصل error → كله يرجع

---

# 🧠 الخلاصة

## Custom Request

- Validation clean
    
- شبه DTO
    
- بيرتب الكود
    

---

## Query Builder

- تعامل مباشر مع DB
    
- أسرع من Eloquent
    
- شبه SQL
    

---

## Flow النهائي

```text
Request → Custom Request → Controller → Query Builder → DB
```

---

# 🎯 انت كدا وصلت لمرحلة قوية جدًا

لو عايز نكمل:  
🔥 نحول دا API حقيقي  
🔥 أو نستخدم Resource بدل JSON  
🔥 أو نقارن Query Builder vs Eloquent عملي

قولّي وأنا أطلعك level أعلى 🔥💪



![[WhatsApp Image 2026-04-01 at 3.14.41 AM.jpeg]]

![[WhatsApp Image 2026-04-01 at 3.14.41 AM (1).jpeg]]