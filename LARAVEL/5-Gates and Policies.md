تمام 👌 خلينا نعمل شرح احترافي كامل **Gates + Policies + Authentication + Authorization** في Laravel خطوة خطوة كأننا بنبني مشروع حقيقي.

سأفترض **Laravel 10**.

---

# 1) نظرة عامة — ما هي Policies و Gates ؟

## 🔐 Authorization في Laravel

بعد ما المستخدم يعمل **Login (Authentication)** نحتاج نقرر:

> هل مسموح له يعمل الفعل ده ولا لا ؟

مثلاً:

- هل يقدر يعدّل المنتج؟
    
- هل يقدر يحذف منتج مش بتاعه؟
    
- هل admin يقدر يعمل كل حاجة؟
    

هنا يأتي دور:

## 🎯 Gate

Gate = Rule بسيط وسريع  
يستخدم غالباً في صلاحيات عامة أو global.

مثال:

- هل المستخدم admin؟
    
- هل يقدر يدير المنتجات؟
    

يُكتب غالباً في `AuthServiceProvider`.

---

## 🛡️ Policy

Policy = Authorization مرتب مرتبط بموديل معين.

مثال:

- هل المستخدم يقدر يعدّل **Product معين**؟
    
- هل يقدر يحذف **هذا المنتج**؟
    

Policy مرتبطة بالـ Model.

👉 قاعدة مهمة:

- Gate → صلاحيات عامة
    
- Policy → صلاحيات مرتبطة بالـ Models
    

---

# 2) تجهيز مشروع Laravel

## إنشاء المشروع

```bash
composer create-project laravel/laravel laravel-auth-app
cd laravel-auth-app
```

## إعداد قاعدة البيانات

في `.env`

```env
DB_DATABASE=laravel_auth
DB_USERNAME=root
DB_PASSWORD=
```

ثم:

```bash
php artisan migrate
```

---

# 3) إنشاء Models + Migrations

## إنشاء Product model

```bash
php artisan make:model Product -m
```

### migration

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price',8,2);
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
});
```

ثم:

```bash
php artisan migrate
```

---

## العلاقات بين User و Product

### User.php

```php
public function products()
{
    return $this->hasMany(Product::class);
}
```

### Product.php

```php
public function user()
{
    return $this->belongsTo(User::class);
}
```

---

# 4) إضافة Authentication

أسهل طريقة:

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run build
php artisan migrate
```

أصبح لدينا:

- Register
    
- Login
    
- Sessions
    
- Logout
    

🔥 الآن لدينا Authentication جاهز.

---

# 5) إنشاء Policy

```bash
php artisan make:policy ProductPolicy --model=Product
```

### تسجيلها في AuthServiceProvider

```php
protected $policies = [
    Product::class => ProductPolicy::class,
];
```

---

## ProductPolicy كامل

```php
class ProductPolicy
{
    // عرض المنتجات
    public function viewAny(User $user)
    {
        return true;
    }

    // عرض منتج واحد
    public function view(User $user, Product $product)
    {
        return true;
    }

    // إنشاء منتج
    public function create(User $user)
    {
        return true;
    }

    // تعديل منتج
    public function update(User $user, Product $product)
    {
        return $user->id === $product->user_id;
    }

    // حذف منتج
    public function delete(User $user, Product $product)
    {
        return $user->id === $product->user_id;
    }
}
```

💡 هنا الفكرة الأساسية:

> فقط صاحب المنتج يستطيع تعديله أو حذفه.

---

# 6) إنشاء Gate إضافي

في `AuthServiceProvider`

```php
use Illuminate\Support\Facades\Gate;

public function boot()
{
    $this->registerPolicies();

    Gate::define('manage-products', function (User $user) {
        return $user->email === 'admin@gmail.com';
    });
}
```

👉 هذا Gate يسمح فقط للـ admin بإدارة المنتجات.

---

# 7) إنشاء Controller

```bash
php artisan make:controller ProductController --resource
```

---

## استخدام Authorization داخل Controller

```php
class ProductController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }

    public function index()
    {
        $products = Product::all();
        return view('products.index', compact('products'));
    }

    public function create()
    {
        $this->authorize('create', Product::class);
        return view('products.create');
    }

    public function store(Request $request)
    {
        $this->authorize('create', Product::class);

        Product::create([
            'name'=>$request->name,
            'description'=>$request->description,
            'price'=>$request->price,
            'user_id'=>auth()->id()
        ]);

        return redirect('/products');
    }

    public function edit(Product $product)
    {
        $this->authorize('update', $product);
        return view('products.edit', compact('product'));
    }

    public function update(Request $request, Product $product)
    {
        $this->authorize('update', $product);
        $product->update($request->all());
        return redirect('/products');
    }

    public function destroy(Product $product)
    {
        $this->authorize('delete', $product);
        $product->delete();
        return redirect('/products');
    }
}
```

---

# 8) حماية Routes

في `web.php`

```php
Route::middleware('auth')->group(function () {
    Route::resource('products', ProductController::class);
});
```

---

# 9) مثال استخدام Gate داخل View

```php
@can('manage-products')
    <a href="/products/create">Create Product</a>
@endcan
```

---

# 10) Class Diagram (Text)

```
User
 └── hasMany → Product

Product
 └── belongsTo → User

ProductPolicy
 ├── viewAny()
 ├── view()
 ├── create()
 ├── update()
 └── delete()

AuthServiceProvider
 ├── register ProductPolicy
 └── Gate: manage-products

ProductController
 └── uses authorize()
```

---

# 11) Flow Diagram

```
User Login
   ↓
Request -> /products/{id}/edit
   ↓
Middleware auth
   ↓
Controller
   ↓
Policy::update(user, product)
   ↓
YES → execute action
NO  → 403 Forbidden
```

---

# 12) Routes المقترحة

|Method|URL|Action|
|---|---|---|
|GET|/products|list|
|GET|/products/create|create form|
|POST|/products|store|
|GET|/products/{id}/edit|edit|
|PUT|/products/{id}|update|
|DELETE|/products/{id}|delete|

---

# 13) لو API باستخدام Sanctum أو Passport

## Sanctum

مناسب SPA و Mobile Apps

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

ثم حماية API:

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

الفرق:

|Breeze|Sanctum|
|---|---|
|Session auth|Token auth|
|Web apps|SPA / Mobile|
|Cookies|Bearer Token|

---

# 14) تشغيل المشروع

```bash
php artisan serve
```

ادخل:

```
http://127.0.0.1:8000/register
```

سجّل مستخدم وابدأ إنشاء منتجات 🎉

---

لو حابب بعد كده نعمل نفس المشروع **REST API فقط باستخدام Sanctum** ونخليه production ready قولّي 😎