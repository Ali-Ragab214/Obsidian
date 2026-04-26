
هنبني **REST API كامل** باستخدام كيانين:

- User
    
- Product (اليوزر يملك منتجات)
    

وده هيخلينا نطبق:

- CRUD كامل
    
- Relationships
    
- Validation
    
- Authentication (Sanctum)
    
- Authorization (Policy)
    
- Resources (API responses)
    

امسك قهوتك ☕ ونبدأ خطوة خطوة.

---

# أولاً: ما هو Laravel API ؟

Laravel API = Backend فقط  
لا يوجد Blade ولا Views  
يرجع JSON فقط.

مثال response:

```json
{
  "id": 1,
  "name": "Laptop",
  "price": 2000
}
```

---

# 1️⃣ إنشاء مشروع Laravel API

```bash
composer create-project laravel/laravel laravel-api
cd laravel-api
php artisan serve
```

---

# 2️⃣ تحويل Laravel إلى API mode

افتح:

```
app/Http/Kernel.php
```

احذف أو تجاهل:

```
web middleware
sessions
cookies
csrf
```

سنستخدم **API routes فقط**:

```
routes/api.php
```

---

# 3️⃣ إنشاء Models + Migration

## إنشاء User & Product

```bash
php artisan make:model Product -m
php artisn make:model Name -m
```

Laravel لديه User جاهز.

---

## Migration Product

افتح:

```
database/migrations/xxxx_create_products_table.php
```

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->decimal('price',8,2);
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->timestamps();
    });
}
```

تشغيل المايجريشن:

```bash
php artisan migrate
```

---

# 4️⃣ إنشاء العلاقة بين User و Product

## User Model

```
app/Models/User.php
```

```php
public function products()
{
    return $this->hasMany(Product::class);
}
```

## Product Model

```
app/Models/Product.php
```

```php
class Product extends Model
{
    protected $fillable = ['name','description','price','user_id'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

---

# 5️⃣ تثبيت Authentication (Sanctum)

أفضل حل للـ API.

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

افتح User model:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

---

# 6️⃣ إنشاء Auth Controller

```bash
php artisan make:controller AuthController
```

## Register

```php
public function register(Request $request)
{
    $user = User::create([
        'name'=>$request->name,
        'email'=>$request->email,
        'password'=>bcrypt($request->password)
    ]);

    $token = $user->createToken('api-token')->plainTextToken;

    return response()->json([
        'user'=>$user,
        'token'=>$token
    ]);
}
```

## Login

```php
public function login(Request $request)
{
    if(!Auth::attempt($request->only('email','password')))
        return response()->json(['message'=>'Invalid credentials'],401);

    $user = Auth::user();
    $token = $user->createToken('api-token')->plainTextToken;

    return ['token'=>$token];
}
```

---

# 7️⃣ حماية الـ Routes

```
routes/api.php
```

```php
Route::post('register',[AuthController::class,'register']);
Route::post('login',[AuthController::class,'login']);

Route::middleware('auth:sanctum')->group(function(){
    Route::apiResource('products',ProductController::class);
});
```

---

# 8️⃣ إنشاء Product Controller (CRUD)

```bash
php artisan make:controller ProductController --api
```

## إنشاء منتج

```php
public function store(Request $request)
{
    $product = Product::create([
        'name'=>$request->name,
        'description'=>$request->description,
        'price'=>$request->price,
        'user_id'=>auth()->id()
    ]);

    return $product;
}
```

## عرض المنتجات

```php
public function index()
{
    return Product::with('user')->get();
}
```

## تحديث

```php
public function update(Request $request, Product $product)
{
    $product->update($request->only('name','description','price'));
    return $product;
}
```

## حذف

```php
public function destroy(Product $product)
{
    $product->delete();
    return response()->noContent();
}
```

---

# 9️⃣ Authorization باستخدام Policy

```bash
php artisan make:policy ProductPolicy --model=Product
```

## Policy

```php
public function update(User $user, Product $product)
{
    return $user->id === $product->user_id;
}

public function delete(User $user, Product $product)
{
    return $user->id === $product->user_id;
}
```

سجلها في:

```
AuthServiceProvider
```

---

## استخدام Policy داخل Controller

```php
public function update(Request $request, Product $product)
{
    $this->authorize('update',$product);
    $product->update($request->all());
}
```

الآن:

- المستخدم لا يستطيع تعديل إلا منتجاته 🔥
    

---

# 10️⃣ تجربة API في Postman

## Register

POST

```
/api/register
```

## Login

POST

```
/api/login
```

خذ الـ token وضعه في Header:

```
Authorization: Bearer TOKEN
```

ثم:

```
GET /api/products
POST /api/products
```

---

# 🎉 مبروك

أنت الآن بنيت:

- Laravel REST API
    
- Authentication (Sanctum)
    
- Authorization (Policy)
    
- CRUD
    
- Relationships
    

---

لو حابب نعمل النسخة الاحترافية (Validation + Resources + Service Layer) نكمل عليها خطوة بخطوة 🔥