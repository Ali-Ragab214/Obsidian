Full Example
حلو 😎 انت عايز **Crash Course Laravel كامل لكن عملي وProduction-style** عشان ترجع تفتكر الـ syntax بسرعة + Clean Architecture mindset.

خلينا نبني **API كاملة ل Entity واحدة (Product)** ونشرح كل الملفات اللي بتتولد حوالينها وليه موجودة.

هنمشي بالترتيب ده:

1. فكرة Laravel بسرعة
    
2. Folder Structure diagram
    
3. Lifecycle للـ Request
    
4. إنشاء مشروع + كل أوامر التيرمنال
    
5. بناء Entity كاملة (Product) خطوة خطوة
    
6. Validation + Clean code tricks
    
7. Seeder + Factory + Testing data
    
8. API Routes + Best practices
    
9. Cheat Sheet بالأوامر والتريكات
    

خد نفس عميق… ده مرجع ترجعله كل مرة 🔥

---

# 1️⃣ Laravel Architecture بسرعة

Laravel مبني على **MVC**

- Model → يتعامل مع Database
    
- Controller → يستقبل request ويرجع response
    
- View → للـ Blade (مش مهم مع API)
    

لكن Laravel زود فوقهم:

- FormRequest → validation نظيفة
    
- Resource → شكل الـ JSON Response
    
- Factory → توليد بيانات fake
    
- Seeder → إدخال بيانات DB
    
- Migration → إنشاء الجداول
    

يعني Laravel جاهز Production out of the box.

---

# 2️⃣ Folder Structure Diagram (أهم جزء)

ده أهم رسم تحفظه 👇

```
app/
 ├── Models/
 │     └── Product.php
 │
 ├── DTOs/
 │     └── ProductDTO.php
 │
 ├── Repositories/
 │     ├── Interfaces/
 │     │      └── ProductRepositoryInterface.php
 │     │
 │     └── ProductRepository.php
 │
 ├── Services/
 │     └── ProductService.php
 │
 ├── Exceptions/
 │     ├── Handler.php
 │     ├── ApiException.php
 │     └── NotFoundException.php
 │
 ├── Traits/
 │     └── ApiResponseTrait.php
 │
 ├── Http/
 │     ├── Controllers/
 │     │      ├── BaseController.php
 │     │      └── ProductController.php
 │     │
 │     ├── Requests/
 │     │      ├── StoreProductRequest.php
 │     │      └── UpdateProductRequest.php
 │     │
 │     └── Resources/
 │            └── ProductResource.php
 │
database/
 ├── migrations/
 │      └── create_products_table.php
 │
 ├── factories/
 │      └── ProductFactory.php
 │
 └── seeders/
        └── ProductSeeder.php

routes/
 └── api.php```


أيوه 👍 لو اشتغلت بالـ structure ده وبالطريقة اللي عملناها (FormRequest + Resource + Exceptions + Factory + Seeder) انت فعليًا ماشي بنفس **أسلوب شركات كتير بتبني بيه Laravel APIs**.

بس خليني أكون صريح معاك 100% 👇  
الـ structure ده = **Foundation محترمة جدًا**  
لكن الاحتراف الحقيقي مش فولدرات بس… الاحتراف = طبقات زيادة + Practices معينة.

خلينا نفرق بين مستويين 👇

---

# 🟡 المستوى 1 — Laravel Clean (اللي انت عملته)

وده مستوى **Junior → Mid backend**

اللي عندك دلوقتي فيه:

✔ MVC مضبوط  
✔ Validation خارج controller  
✔ JSON Resource  
✔ Exception handling مركزي  
✔ Seeder + Factory  
✔ API Resource Routes  
✔ Pagination & Filtering

وده لو حد قدم بيه على شغل Laravel Junior هيتقبل عادي.

بس الشركات الكبيرة بتزود حاجات فوقه 👇

---

# 🟢 المستوى 2 — Laravel Production الحقيقي (أسلوب الشركات)

الشركات بتضيف 4 طبقات فوق اللي عملناه:

## 1️⃣ Services Layer

controller مش بيكتب business logic.

بدل ده:

```php
Product::create(...)
```

بيبقى كده:

```php
$this->productService->createProduct($data);
```

ليه؟  
عشان:

- نقدر نعيد استخدام المنطق
    
- نسهل testing
    
- نقلل حجم الكنترولر
    

---

## 2️⃣ Repository Layer (اختياري في Laravel)

مش كل الشركات بتعمله، لكن كتير بيعمله.

بدل الكنترولر يتعامل مع Model مباشرة  
يتعامل مع Repository.

ده بيخلي المشروع scalable جدًا.

---

## 3️⃣ API Response Wrapper

كل الردود تبقى موحدة.

بدل:

```json
{
  "id": 1,
  "name": "Phone"
}
```

يبقى:

```json
{
  "success": true,
  "message": "Product fetched",
  "data": { }
}
```

Consistency = احتراف.

---

## 4️⃣ Feature-based structure (لو المشروع كبر)

بدل ما المشروع يكبر ويبقى spaghetti 🍝  
بيتقسم Modules.

مثلاً:

```text
app/
 ├── Modules/
 │     ├── Product/
 │     │     ├── Controllers
 │     │     ├── Services
 │     │     ├── Models
 │     │     └── Requests
```

ده أسلوب الشركات الكبيرة.

---

# 💎 الخلاصة الصريحة

|السؤال|الإجابة|
|---|---|
|هل اللي عملناه احترافي؟|نعم ✔|
|هل ده مستوى شركات؟|مستوى Junior/Mid ✔|
|إيه اللي ناقص تبقى Senior style؟|Services + Response Wrapper + Modules|

يعني انت دلوقتي وصلت **70% من أسلوب الشركات** 🔥  
ولو ضفنا Service Layer هتبقى قربت جدًا من مستوى Senior Laravel.

---

لو عايز نكمل ونحول نفس الـ Product API إلى  
🔥 Service Layer + BaseController + Response Trait  
وده أهم upgrade في Laravel كله… نقفز للمستوى اللي بعده 😎


💡 احفظ القاعدة دي:  
كل Entity ليها **7 ملفات أساسية**

|الملف|ليه|
|---|---|
|Migration|إنشاء جدول DB|
|Model|يمثل الجدول|
|Controller|API endpoints|
|Requests|Validation|
|Resource|شكل JSON|
|Factory|Fake data|
|Seeder|إدخال بيانات|

---

# 3️⃣ Laravel Request Lifecycle (مهم جدا)

```
Client → Route → Controller → Request Validation → Model → DB
                                   ↓
                               Resource → JSON Response
```

يعني:

- Route يوجه الطلب
    
- Controller يستقبل
    
- FormRequest يعمل validation
    
- Model يحفظ في DB
    
- Resource يرجع JSON نظيف
    

---

# 4️⃣ إنشاء مشروع Laravel + أوامر التيرمنال

### إنشاء مشروع

```bash
composer create-project laravel/laravel laravel-api
cd laravel-api
php artisan serve
```

### ربط DB

افتح `.env`

```
DB_DATABASE=laravel_api
DB_USERNAME=root
DB_PASSWORD=
```

---

# 5️⃣ إنشاء Product Entity كاملة 🔥

هنولد كل الملفات مرة واحدة:

```bash
php artisan make:model Product -mcrf
```

ده يولد:

- Model
    
- Migration
    
- Controller
    
- Factory
    
- Resource
    

لسه ناقص Requests:

```bash
php artisan make:request StoreProductRequest
php artisan make:request UpdateProductRequest
```

---

# 6️⃣ Migration (إنشاء الجدول)

`database/migrations/...create_products_table.php`

```php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->decimal('price', 8, 2);
        $table->integer('stock');
        $table->timestamps();
    });
}
```

تشغيل المايجريشن:

```bash
php artisan migrate
```

---

# 7️⃣ Model

`app/Models/Product.php`

```php
class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'description',
        'price',
        'stock'
    ];
}
```

💡 Trick مهم:  
`$fillable` يحميك من Mass Assignment

---

# 8️⃣ Validation باستخدام Form Request 🔥

## StoreProductRequest

```php
public function rules(): array
{
    return [
        'name' => 'required|string|max:255',
        'description' => 'nullable|string',
        'price' => 'required|numeric|min:0',
        'stock' => 'required|integer|min:0'
    ];
}
```

## UpdateProductRequest

```php
public function rules(): array
{
    return [
        'name' => 'sometimes|string|max:255',
        'description' => 'nullable|string',
        'price' => 'sometimes|numeric|min:0',
        'stock' => 'sometimes|integer|min:0'
    ];
}
```

💡 Trick:

- required → في create
    
- sometimes → في update
    

---

# 9️⃣ Resource (شكل الـ JSON)

`ProductResource.php`

```php
class ProductResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'price' => $this->price,
            'stock' => $this->stock,
            'created_at' => $this->created_at->diffForHumans(),
        ];
    }
}
```

💡 Resource = DTO بتاع Laravel

---

# 🔟 Controller (Clean Production Style)

`ProductController.php`

```php
class ProductController extends Controller
{
    public function index()
    {
        return ProductResource::collection(Product::latest()->paginate());
    }

    public function store(StoreProductRequest $request)
    {
        $product = Product::create($request->validated());

        return new ProductResource($product);
    }

    public function show(Product $product)
    {
        return new ProductResource($product);
    }

    public function update(UpdateProductRequest $request, Product $product)
    {
        $product->update($request->validated());

        return new ProductResource($product);
    }

    public function destroy(Product $product)
    {
        $product->delete();

        return response()->json(['message' => 'Deleted successfully']);
    }
}
```

🔥 تريكات مهمة:

- Route Model Binding → Laravel يجيب الـ product لوحده
    
- `$request->validated()` → يرجع بيانات نظيفة فقط
    

---

# 1️⃣1️⃣ API Routes

`routes/api.php`

```php
Route::apiResource('products', ProductController::class);
```

Laravel يولد:

```
GET /products
POST /products
GET /products/{id}
PUT /products/{id}
DELETE /products/{id}
```

---

# 1️⃣2️⃣ Factory (توليد بيانات)

```php
public function definition(): array
{
    return [
        'name' => fake()->word(),
        'description' => fake()->sentence(),
        'price' => fake()->randomFloat(2, 10, 1000),
        'stock' => fake()->numberBetween(0, 100)
    ];
}
```

---

# 1️⃣3️⃣ Seeder

```bash
php artisan make:seeder ProductSeeder
```

```php
public function run(): void
{
    Product::factory(50)->create();
}
```

تشغيل:

```bash
php artisan db:seed --class=ProductSeeder
```

أو:

```bash
php artisan migrate:fresh --seed
```

---

# 🔥 Tricks مهمة تحفظها

### Pagination

```php
Product::paginate(10);
```

### Search

```php
Product::where('name', 'like', "%$search%")->get();
```

### Order

```php
Product::latest()->get();
```

### Filtering Clean

```php
$query = Product::query();

if($request->has('min_price')){
   $query->where('price','>=',$request->min_price);
}

return ProductResource::collection($query->get());
```

---

# 🧠 Laravel Terminal Cheat Sheet

|الأمر|الوظيفة|
|---|---|
|php artisan serve|تشغيل السيرفر|
|php artisan migrate|تشغيل المايجريشن|
|php artisan migrate:fresh --seed|reset DB|
|php artisan make:model X -mcrf|إنشاء كل ملفات entity|
|php artisan make:request X|validation|
|php artisan make:seeder X|seeder|
|php artisan route:list|عرض الروتس|

---

# 💎 أهم نصائح Production

✔ استخدم FormRequest دايماً  
✔ استخدم Resource للـ JSON  
✔ متكتبش validation في controller  
✔ متستخدمش Request مباشرة  
✔ استخدم pagination  
✔ استخدم route model binding

---

لو عايز بعد كده نعمل نفس الكلام لكن مع **Auth + JWT أو Sanctum** ونحولها API احترافية كاملة قولّي 👌