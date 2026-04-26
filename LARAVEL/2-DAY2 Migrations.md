# 🧱 Laravel — Day 2: Migrations & CRUD with Eloquent

---

بص الخلاصه بدل ما كنا بنيجي فال controller  ونقرا الداتا من json file لا بقا هنعمل db migrations  ونقرا الداتا ونشتغل علي داتا بيز 
هتكتب ال command :php artisan make:model modelname -mcr هيعمل model & controller 7 migraition
واشتغل بقا حط ال columns for db in migration وحط ال fillable in model  وبعدين روح فالكنترولر حط كل اللوجيك والفاليديشن والكنترولر هيبعت الداتا للفيو يعني مثلاا  create method  دي هترندرك علي الفيو اللي هتاخد منه الداتا اللي جايه من اليوزر في فورم مثلا وتخلي الفورم دي تبعت الداتا علي ال store ميثود وطبعا متنساش الفاليديشن تحطه وبعدين تعمل save for the data وهندل الفرونتاند بقا وبس




## 📌 المفهوم العام — Code First

> [!info] Code First في Laravel بدل ما تروح MySQL وتعمل الجداول يدوياً، بتكتب كود PHP وLaravel يعمل الجداول بدالك. ده اللي بيتسمى **Code First Approach**.

```
PHP Migration File  →  php artisan migrate  →  MySQL Table
```
php artisan migrate 

---

## 1️⃣ Migrations

### 🔹 إيه هو الـ Migration؟

> [!abstract] تعريف الـ Migration هو ملف PHP بيوصف شكل الجدول في الداتا بيز. بيشتغل زي **Version Control** للداتا بيز — أي حد في الفريق يعمل `migrate` يطلع نفس الجداول.

---

### 🔹 إنشاء Migration

```bash
# إنشاء migration جديد
php artisan make:migration create_products_table

# أو مع Model في نفس الوقت
php artisan make:model Product -m
```

> [!tip] الـ `-m` flag بتعمل Model + Migration في أمر واحد — ده اللي هتستخدمه دايماً.

---

### 🔹 شكل ملف الـ Migration

```php
// database/migrations/2024_01_01_000000_create_products_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    // بيتنفذ لما تعمل: php artisan migrate
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();                        // INT AUTO_INCREMENT PRIMARY KEY
            $table->string('name');              // VARCHAR(255)
            $table->text('description')->nullable(); // TEXT قابل يبقى فاضي
            $table->decimal('price', 8, 2);      // DECIMAL(8,2) — للأسعار
            $table->string('category');
            $table->string('image')->nullable();
            $table->timestamps();                // created_at + updated_at
        });
    }

    // بيتنفذ لما تعمل: php artisan migrate:rollback
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

---

### 🔹 أنواع الـ Column Types الشائعة

|Method|MySQL Type|استخدام|
|---|---|---|
|`$table->id()`|BIGINT UNSIGNED AI PK|الـ Primary Key دايماً|
|`$table->string('name')`|VARCHAR(255)|نصوص قصيرة|
|`$table->string('name', 100)`|VARCHAR(100)|نص بحجم محدد|
|`$table->text('body')`|TEXT|نصوص طويلة|
|`$table->integer('stock')`|INT|أرقام صحيحة|
|`$table->decimal('price', 8, 2)`|DECIMAL(8,2)|أسعار وأرقام عشرية|
|`$table->boolean('active')`|TINYINT(1)|true/false|
|`$table->date('birth_date')`|DATE|تاريخ|
|`$table->timestamps()`|created_at + updated_at|تلقائي|
|`->nullable()`|NULL allowed|اجعل العمود اختياري|
|`->default('value')`|DEFAULT|قيمة افتراضية|
|`->unique()`|UNIQUE|لا يتكرر|

---

### 🔹 أوامر الـ Migration

```bash
# تنفيذ كل الـ migrations الجديدة
php artisan migrate

# التراجع عن آخر batch
php artisan migrate:rollback

# مسح كل الجداول وإعادة البناء من الأول
php artisan migrate:fresh

# مسح وإعادة البناء + تشغيل الـ Seeders
php artisan migrate:fresh --seed

# عرض حالة كل الـ migrations
php artisan migrate:status
```

> [!warning] تحذير — `migrate:fresh` بتمسح **كل** البيانات في الداتا بيز. استخدمها بس في Development مش Production.

---

## 2️⃣ Eloquent Model

### 🔹 إيه هو الـ Eloquent؟

> [!abstract] تعريف **Eloquent** هو الـ ORM بتاع Laravel. بيخليك تتعامل مع الجداول كـ PHP Objects بدل ما تكتب SQL بنفسك. كل Model بيمثل جدول، وكل Object بيمثل Row.

---

### 🔹 شكل الـ Model

```php
// app/Models/Product.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // الجدول اللي بيمثله — Laravel بيحدده تلقائي (products)
    // protected $table = 'products'; // لو الاسم مختلف

    // الـ columns اللي مسموح تتملى بـ mass assignment
    protected $fillable = [
        'name',
        'description',
        'price',
        'category',
        'image',
    ];
}
```

> [!info] Convention بتاع Laravel لو الـ Model اسمه `Product`، Laravel هيدور تلقائي على جدول اسمه `products`. (جمع + lowercase) — مش محتاج تحدده غير لو الاسم مختلف.

---

### 🔹 إيه هو `$fillable`؟

> [!question] ليه محتاج `$fillable`؟ Laravel بيحمي من **Mass Assignment Attack** — يعني حد يبعت بيانات مش المفروض تتحفظ. الـ `$fillable` بيقول: "الـ columns دي بس مسموح تتملى بشكل جماعي".

```php
// ✅ شغال — لأن name موجود في $fillable
Product::create(['name' => 'Laptop', 'price' => 999]);

// ❌ مش هيتحفظ — لو 'is_admin' مش في $fillable
Product::create(['name' => 'Test', 'is_admin' => true]);
```

---

## 3️⃣ CRUD بـ Eloquent

### 🔹 Create — إضافة

```php
// طريقة 1: create() — الأسرع
$product = Product::create([
    'name'     => 'Laptop',
    'price'    => 999.99,
    'category' => 'Electronics',
]);

// طريقة 2: new + save() — لو محتاج تعمل حاجة قبل الحفظ
$product = new Product();
$product->name     = 'Laptop';
$product->price    = 999.99;
$product->category = 'Electronics';
$product->save();
```

---

### 🔹 Read — قراءة

```php
// جيب كل المنتجات
$products = Product::all();

// جيب منتج بالـ ID
$product = Product::find(1);        // بيرجع null لو مش موجود
$product = Product::findOrFail(1);  // بيرجع 404 لو مش موجود ✅ الأحسن

// جيب أول منتج يطابق الشرط
$product = Product::where('category', 'Electronics')->first();

// جيب كل المنتجات اللي سعرها أكتر من 500
$products = Product::where('price', '>', 500)->get();

// جيب بشروط متعددة
$products = Product::where('category', 'Electronics')
                   ->where('price', '<', 1000)
                   ->orderBy('price', 'asc')
                   ->get();
```

---

### 🔹 Update — تعديل

```php
// طريقة 1: find ثم update()
$product = Product::findOrFail(1);
$product->update([
    'name'  => 'Gaming Laptop',
    'price' => 1299.99,
]);

// طريقة 2: find ثم save()
$product = Product::findOrFail(1);
$product->name = 'Gaming Laptop';
$product->save();
```

---

### 🔹 Delete — حذف

```php
// طريقة 1: find ثم delete()
$product = Product::findOrFail(1);
$product->delete();

// طريقة 2: حذف مباشر بالـ ID
Product::destroy(1);

// حذف أكتر من record
Product::destroy([1, 2, 3]);
```

---

## 4️⃣ ربط كل حاجة في الـ Controller

```php
// app/Http/Controllers/ProductController.php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Read All
    public function index()
    {
        $products = Product::all();
        return view('products.index', compact('products'));
    }

    // Read One
    public function show($id)
    {
        $product = Product::findOrFail($id);
        return view('products.show', compact('product'));
    }

    // Create - Form
    public function create()
    {
        return view('products.create');
    }

    // Create - Save
    public function store(Request $request)
    {
        Product::create($request->all());
        return redirect()->route('products.index');
    }

    // Update - Form
    public function edit($id)
    {
        $product = Product::findOrFail($id);
        return view('products.edit', compact('product'));
    }

    // Update - Save
    public function update(Request $request, $id)
    {
        $product = Product::findOrFail($id);
        $product->update($request->all());
        return redirect()->route('products.index');
    }

    // Delete
    public function destroy($id)
    {
        Product::findOrFail($id)->delete();
        return redirect()->route('products.index');
    }
}
```

---

## 5️⃣ الصورة الكاملة — Flow

```
php artisan make:model Product -m
         │
         ├── app/Models/Product.php        ← الـ Model
         └── database/migrations/...       ← الـ Migration
                    │
                    ▼
         كتابة columns في up()
                    │
                    ▼
         php artisan migrate
                    │
                    ▼
         ✅ جدول products اتعمل في MySQL
                    │
                    ▼
         ProductController بيستخدم Product::
         create / all / findOrFail / update / delete
```

---

## 6️⃣ Quick Reference — أهم الأوامر

```bash
# إنشاء Model + Migration
php artisan make:model Product -m

# إنشاء Controller كامل بـ CRUD
php artisan make:controller ProductController --resource

# تنفيذ الـ Migrations
php artisan migrate

# Reset كامل
php artisan migrate:fresh
```

> [!success] ملخص Code First
> 
> 1. `make:model` → كتابة الـ columns في Migration → `migrate`
> 2. الـ Model بيمثل الجدول، كل object بيمثل row
> 3. `$fillable` ضروري لاستخدام `create()` و `update()`
> 4. `findOrFail()` أحسن من `find()` — بيرجع 404 تلقائي