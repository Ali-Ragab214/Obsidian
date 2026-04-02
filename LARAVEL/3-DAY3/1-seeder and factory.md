
---

## 1️⃣ الفكرة العامة

- **Factory**: طريقة سريعة لعمل بيانات مزيفة (fake data) لأي موديل.
    
- **Seeder**: طريقة لإضافة بيانات للموديل في قاعدة البيانات باستخدام Factory أو بيانات ثابتة.
    

باختصار:  
**Factory = يولد بيانات، Seeder = يدخل البيانات في قاعدة البيانات.**

---

## 2️⃣ خطوات إنشاء Factory + Seeder

### الخطوة 1: تفعيل الـ Factory في الموديل

في الموديل (`Order.php` أو أي موديل):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    use HasFactory; // لازم تكتبها عشان تقدر تستخدم Factory
}
```

---

### الخطوة 2: إنشاء الـ Factory

```bash
php artisan make:factory OrderFactory --model=Order
```

- ده هيعمل ملف جديد في `database/factories/OrderFactory.php`
    
- جوه الـ Factory، في الـ `definition()` method بنحدد شكل البيانات المزيفة:
    

```php
public function definition(): array
{
    return [
        'user_id' => $this->faker->numberBetween(1, 10),
        'total_price' => $this->faker->randomFloat(2, 50, 500),
    ];
}
```

> لو عندك علاقة مع موديل تاني (زي OrderItem مع Order)، ممكن تكتب:
> 
> ```php
> 'order_id' => \App\Models\Order::factory(), // هينشئ Order جديد تلقائي
> ```

---

### الخطوة 3: إنشاء Seeder للموديل

```bash
php artisan make:seeder OrderSeeder
```

في `OrderSeeder.php`:

```php
public function run(): void
{
    \App\Models\Order::factory()->count(10)->create(); // هينشئ 10 Orders
}
```

---

### الخطوة 4: استدعاء Seeder في DatabaseSeeder

في `DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call([
        OrderSeeder::class,
        OrderItemSeeder::class, // لو عندك OrderItems
    ]);
}
```

---

### الخطوة 5: تشغيل Seeder

```bash
php artisan db:seed
```

- هيشتغل ويضيف كل البيانات اللي انت محددها في الـ Factories للـ Database.
    

---

## 3️⃣ الـ Flow النهائي (الفكرة العامة)

1. **Model** جاهز وبيستخدم `HasFactory`.
    
2. **Factory** بيحدد شكل البيانات المزيفة.
    
3. **Seeder** بيستدعي الـ Factory ويعمل create() أو insert() للبيانات.
    
4. **DatabaseSeeder** بيجمع كل الـ Seeders مع بعض ويشغلهم.
    
5. **db:seed** يدخل البيانات في قاعدة البيانات.
    
6. بعد كده ممكن تعرض البيانات في أي Controller / View.
    

---

### 🔹 ملاحظات مهمة

- الاسماء Case Sensitive: الموديل `Order` → Factory يستخدم `Order::factory()`.
    
- ممكن Factory يولد **Related models تلقائي** عن طريق كتابة `Model::factory()` في Definition.
    
- ممكن تستخدم `state()` في Factory لتخصيص بيانات معينة لكل حالة.
    

---

لو تحب، أقدر أرسم لك **Flow diagram** يوضح **Factory → Seeder → DB → View** بحيث تشوف كل حاجة بتتربط إزاي. هيخلي الموضوع واضح 100%.

تحب أعمله؟