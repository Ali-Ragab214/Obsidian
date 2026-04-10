سأقدم لك دليلاً شاملاً واحترافياً بالكامل. الإعدادات المختارة:

- **Laravel 11.x** (الأحدث والأكثر استخداماً)
- **Laravel Sanctum** للمصادقة (أبسط وأكثر ملاءمة للـ SPA والمشاريع المتوسطة)
- **MySQL / PostgreSQL** (الشرح يشمل الاثنين)
- **PHP 8.2+**

---

## أولاً: البنية المعمارية — نظرة عليا

في Laravel، كل طلب HTTP يمر عبر "خط أنابيب" واضح. قبل أي كود، يجب أن تفهم هذا المسار:---

## ثانياً: هيكل المشروع الكامل

```
laravel-api/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Api/
│   │   │       └── V1/
│   │   │           └── AuthController.php       ← منطق المصادقة
│   │   ├── Middleware/
│   │   │   └── ForceJsonResponse.php            ← يجبر كل response على JSON
│   │   ├── Requests/
│   │   │   └── Auth/
│   │   │       ├── LoginRequest.php             ← Validation للـ login
│   │   │       └── RegisterRequest.php          ← Validation للـ register
│   │   └── Resources/
│   │       └── UserResource.php                 ← تحويل Model إلى JSON آمن
│   ├── Models/
│   │   └── User.php                             ← Eloquent Model
│   ├── Services/
│   │   └── AuthService.php                      ← Business Logic (اختياري لكن مُوصى به)
│   └── Exceptions/
│       └── Handler.php                          ← مركزة معالجة الأخطاء
├── database/
│   └── migrations/
│       └── create_users_table.php
├── routes/
│   └── api.php                                  ← تعريف مسارات الـ API
├── config/
│   └── sanctum.php
└── tests/
    └── Feature/
        └── Auth/
            └── AuthTest.php                     ← اختبارات شاملة
```

---

## ثالثاً: التطبيق خطوة بخطوة

### الخطوة 1 — إنشاء المشروع وتثبيت Sanctum

```bash
# إنشاء مشروع Laravel جديد
composer create-project laravel/laravel laravel-api
cd laravel-api

# تثبيت Sanctum (مدمج في Laravel 11 افتراضياً)
php artisan install:api

# هذا الأمر يقوم بـ:
# 1. نشر ملف config/sanctum.php
# 2. إنشاء migration لجدول personal_access_tokens
# 3. إضافة HasApiTokens trait للـ User model
```

**لماذا Sanctum وليس Passport؟** Sanctum مثالي للـ SPAs والـ Mobile Apps لأنه خفيف ولا يحتاج OAuth server. Passport هو OAuth2 كامل ويستخدم حين تحتاج لـ third-party integrations.

---

### الخطوة 2 — إعداد قاعدة البيانات

```bash
# في ملف .env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_api
DB_USERNAME=root
DB_PASSWORD=secret

# أو لـ PostgreSQL
DB_CONNECTION=pgsql
DB_PORT=5432
```

---

### الخطوة 3 — الـ Migration (بنية جدول المستخدمين)

```php
// database/migrations/0001_01_01_000000_create_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();                          // bigint auto-increment PK
            $table->string('name');
            $table->string('email')->unique();     // unique index تلقائياً
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');            // مشفر دائماً بـ bcrypt
            $table->rememberToken();
            $table->timestamps();                  // created_at و updated_at
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

```bash
php artisan migrate
```

---

### الخطوة 4 — الـ User Model مع Sanctum

```php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;  // ← مفتاح Sanctum

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * الحقول القابلة للتعبئة الجماعية (Mass Assignment Protection)
     * أي حقل غير موجود هنا لن يُقبل من request مباشرة — أمان مهم جداً
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * الحقول المخفية في JSON output
     * لن تظهر أبداً في أي response
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Casting — يحوّل البيانات تلقائياً عند القراءة/الكتابة
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',  // bcrypt تلقائي عند الكتابة
        ];
    }
}
```

---

### الخطوة 5 — Form Requests (Validation)

**لماذا Form Requests وليس validation في Controller مباشرة؟** الفصل بين المسؤوليات. Controller يجب أن يعرف "ماذا يفعل"، وليس "كيف يتحقق من البيانات".

```php
// app/Http/Requests/Auth/RegisterRequest.php

namespace App\Http\Requests\Auth;

use Illuminate\Foundation\Http\FormRequest;

class RegisterRequest extends FormRequest
{
    // من يستطيع تقديم هذا الطلب؟ true = الجميع
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'string', 'email', 'max:255', 'unique:users'],
            // password_confirmation يُتحقق منه تلقائياً بـ 'confirmed'
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    // رسائل خطأ مخصصة (اختياري لكن مُوصى به)
    public function messages(): array
    {
        return [
            'email.unique'    => 'هذا البريد الإلكتروني مستخدم بالفعل.',
            'password.min'    => 'كلمة المرور يجب أن تكون 8 أحرف على الأقل.',
        ];
    }
}
```

```php
// app/Http/Requests/Auth/LoginRequest.php

namespace App\Http\Requests\Auth;

use Illuminate\Foundation\Http\FormRequest;

class LoginRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'email'    => ['required', 'string', 'email'],
            'password' => ['required', 'string'],
        ];
    }
}
```

---

### الخطوة 6 — API Resource (تنسيق الـ Response)

```php
// app/Http/Resources/UserResource.php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * هنا تتحكم بالضبط فيما يُرسَل للعميل
     * لا password، لا remember_token، لا حقول داخلية
     */
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'created_at' => $this->created_at->toISOString(),
            // يمكن إضافة حقول شرطية:
            // 'role' => $this->when($this->isAdmin(), 'admin'),
        ];
    }
}
```

---

### الخطوة 7 — Auth Controller (القلب)

```php
// app/Http/Controllers/Api/V1/AuthController.php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use App\Http\Requests\Auth\RegisterRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    /**
     * POST /api/v1/register
     * يسجل مستخدم جديد ويعيد توكن فوري
     */
    public function register(RegisterRequest $request): JsonResponse
    {
        // $request->validated() يعيد فقط الحقول التي اجتازت الـ validation
        // أي حقل إضافي في الـ request لن يصل هنا — أمان مهم
        $user = User::create($request->validated());

        // إنشاء توكن Sanctum — الاسم يساعد في تتبع التوكنات
        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'message' => 'تم إنشاء الحساب بنجاح.',
            'user'    => new UserResource($user),
            'token'   => $token,
        ], 201);  // 201 Created — المعيار الصحيح لإنشاء resource جديد
    }

    /**
     * POST /api/v1/login
     * تسجيل الدخول وإعادة توكن جديد
     */
    public function login(LoginRequest $request): JsonResponse
    {
        // Auth::attempt يتحقق من email + password ويهاش تلقائياً
        if (! Auth::attempt($request->only('email', 'password'))) {
            // لا تقل "كلمة المرور خاطئة" — هذا يؤكد أن الإيميل صحيح!
            // دائماً رسالة عامة لمنع User Enumeration Attack
            throw ValidationException::withMessages([
                'email' => ['بيانات الاعتماد غير صحيحة.'],
            ]);
        }

        $user = Auth::user();

        // حذف التوكنات القديمة (اختياري — يمنع تراكم التوكنات)
        // $user->tokens()->delete();

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'message' => 'تم تسجيل الدخول بنجاح.',
            'user'    => new UserResource($user),
            'token'   => $token,
        ]);
    }

    /**
     * POST /api/v1/logout
     * حذف التوكن الحالي فقط (تسجيل خروج من الجهاز الحالي)
     * يتطلب: Authorization: Bearer {token}
     */
    public function logout(Request $request): JsonResponse
    {
        // currentAccessToken() يستهدف التوكن المُرسَل في الـ request فقط
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'تم تسجيل الخروج بنجاح.',
        ]);
    }

    /**
     * GET /api/v1/me
     * إرجاع بيانات المستخدم الحالي
     */
    public function me(Request $request): JsonResponse
    {
        return response()->json([
            'user' => new UserResource($request->user()),
        ]);
    }
}
```

---

### الخطوة 8 — الـ Routes

```php
// routes/api.php

use App\Http\Controllers\Api\V1\AuthController;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| API Routes — الإصدار 1
|--------------------------------------------------------------------------
| prefix('v1') يجعل كل المسارات تبدأ بـ /api/v1/
| لماذا Versioning؟ يسمح بإضافة /api/v2/ مستقبلاً دون كسر الـ clients
*/

Route::prefix('v1')->group(function () {

    // مسارات عامة — لا تتطلب مصادقة
    Route::prefix('auth')->group(function () {
        Route::post('/register', [AuthController::class, 'register']);
        Route::post('/login',    [AuthController::class, 'login']);
    });

    // مسارات محمية — تتطلب توكن صالح في الـ header
    // middleware('auth:sanctum') يرفض أي طلب بدون توكن صالح بـ 401
    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/me',      [AuthController::class, 'me']);

        // مستقبلاً: مسارات المستخدمين، المنتجات، إلخ
        // Route::apiResource('products', ProductController::class);
    });

});
```

---

### الخطوة 9 — Middleware لإجبار JSON

```php
// app/Http/Middleware/ForceJsonResponse.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ForceJsonResponse
{
    /**
     * بدون هذا الـ Middleware، Laravel قد يعيد HTML في حالة الأخطاء
     * هذا يضمن أن كل response من الـ API هو JSON دائماً
     */
    public function handle(Request $request, Closure $next): Response
    {
        $request->headers->set('Accept', 'application/json');
        return $next($request);
    }
}
```

```php
// bootstrap/app.php — تسجيل الـ Middleware في Laravel 11

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // تطبيق الـ Middleware على كل مسارات الـ API
        $middleware->api(prepend: [
            \App\Http\Middleware\ForceJsonResponse::class,
        ]);
    })
    ->create();
```

---

### الخطوة 10 — معالجة الأخطاء المركزية

```php
// app/Exceptions/Handler.php

namespace App\Exceptions;

use Illuminate\Auth\AuthenticationException;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Throwable;

class Handler extends ExceptionHandler
{
    public function register(): void
    {
        $this->renderable(function (Throwable $e) {
            // تطبيق فقط على مسارات الـ API
            if (request()->is('api/*')) {
                return $this->handleApiException($e);
            }
        });
    }

    private function handleApiException(Throwable $e): \Illuminate\Http\JsonResponse
    {
        // خطأ التحقق من البيانات — 422
        if ($e instanceof ValidationException) {
            return response()->json([
                'message' => 'البيانات المُدخَلة غير صحيحة.',
                'errors'  => $e->errors(),
            ], 422);
        }

        // خطأ المصادقة — 401
        if ($e instanceof AuthenticationException) {
            return response()->json([
                'message' => 'غير مصرح. يرجى تسجيل الدخول.',
            ], 401);
        }

        // Resource غير موجود — 404
        if ($e instanceof NotFoundHttpException) {
            return response()->json([
                'message' => 'المورد المطلوب غير موجود.',
            ], 404);
        }

        // أخطاء غير متوقعة — 500 (لا تكشف التفاصيل في production!)
        return response()->json([
            'message' => 'حدث خطأ داخلي في الخادم.',
        ], 500);
    }
}
```

---

### الخطوة 11 — Rate Limiting (منع الإساءة)

```php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // حد الطلبات للـ API العام: 60 طلب/دقيقة لكل IP
        RateLimiter::for('api', function (Request $request) {
            return Limit::perMinute(60)->by($request->ip());
        });

        // حد أقل لمسارات المصادقة: 5 محاولات/دقيقة — يمنع Brute Force
        RateLimiter::for('auth', function (Request $request) {
            return Limit::perMinute(5)->by($request->ip())
                ->response(function () {
                    return response()->json([
                        'message' => 'محاولات كثيرة جداً. حاول بعد دقيقة.',
                    ], 429);
                });
        });
    }
}
```

```php
// routes/api.php — تطبيق Rate Limiting على مسارات الـ Auth
Route::prefix('v1')->group(function () {
    Route::prefix('auth')->middleware('throttle:auth')->group(function () {
        Route::post('/register', [AuthController::class, 'register']);
        Route::post('/login',    [AuthController::class, 'login']);
    });
    // ...
});
```

---

### الخطوة 12 — الاختبارات (Tests)

```php
// tests/Feature/Auth/AuthTest.php

namespace Tests\Feature\Auth;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class AuthTest extends TestCase
{
    // RefreshDatabase تنظف قاعدة البيانات بعد كل اختبار
    use RefreshDatabase;

    /** @test */
    public function user_can_register_with_valid_data(): void
    {
        $response = $this->postJson('/api/v1/auth/register', [
            'name'                  => 'أحمد محمد',
            'email'                 => 'ahmed@example.com',
            'password'              => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response
            ->assertStatus(201)
            ->assertJsonStructure([
                'message',
                'user' => ['id', 'name', 'email', 'created_at'],
                'token',
            ]);

        // التأكد من إنشاء المستخدم في قاعدة البيانات
        $this->assertDatabaseHas('users', ['email' => 'ahmed@example.com']);
    }

    /** @test */
    public function registration_fails_with_duplicate_email(): void
    {
        User::factory()->create(['email' => 'ahmed@example.com']);

        $response = $this->postJson('/api/v1/auth/register', [
            'name'                  => 'أحمد آخر',
            'email'                 => 'ahmed@example.com',
            'password'              => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response->assertStatus(422)->assertJsonValidationErrors(['email']);
    }

    /** @test */
    public function user_can_login_with_correct_credentials(): void
    {
        $user = User::factory()->create(['password' => bcrypt('password123')]);

        $response = $this->postJson('/api/v1/auth/login', [
            'email'    => $user->email,
            'password' => 'password123',
        ]);

        $response
            ->assertStatus(200)
            ->assertJsonStructure(['token', 'user']);
    }

    /** @test */
    public function login_fails_with_wrong_password(): void
    {
        $user = User::factory()->create();

        $this->postJson('/api/v1/auth/login', [
            'email'    => $user->email,
            'password' => 'wrong-password',
        ])->assertStatus(422);
    }

    /** @test */
    public function authenticated_user_can_logout(): void
    {
        $user  = User::factory()->create();
        $token = $user->createToken('test')->plainTextToken;

        $this->withHeader('Authorization', "Bearer $token")
            ->postJson('/api/v1/auth/logout')
            ->assertStatus(200);

        // التأكد من حذف التوكن
        $this->assertDatabaseCount('personal_access_tokens', 0);
    }

    /** @test */
    public function unauthenticated_user_cannot_access_protected_routes(): void
    {
        $this->getJson('/api/v1/auth/me')->assertStatus(401);
    }
}
```

```bash
# تشغيل الاختبارات
php artisan test --filter AuthTest
```

---

## رابعاً: خريطة تدفق المصادقة---

## خامساً: بنية الـ Response الموحدة

أفضل الممارسات تقتضي أن كل response من API تكون بشكل موحد:

```json
// نجاح — register/login
{
    "message": "تم تسجيل الدخول بنجاح.",
    "user": {
        "id": 1,
        "name": "أحمد محمد",
        "email": "ahmed@example.com",
        "created_at": "2025-01-15T10:30:00.000Z"
    },
    "token": "1|abc123xyz..."
}

// فشل التحقق — 422
{
    "message": "البيانات المُدخَلة غير صحيحة.",
    "errors": {
        "email": ["هذا البريد الإلكتروني مستخدم بالفعل."],
        "password": ["كلمة المرور يجب أن تكون 8 أحرف على الأقل."]
    }
}

// عدم المصادقة — 401
{
    "message": "غير مصرح. يرجى تسجيل الدخول."
}
```

---

## سادساً: قائمة التحقق للـ Production

```bash
# ===== الأمان =====
# ✅ APP_ENV=production في .env
# ✅ APP_DEBUG=false — لا تكشف stack traces
# ✅ HTTPS فقط — SSL/TLS certificate
# ✅ Rate Limiting مُفعَّل على مسارات Auth
# ✅ CORS مضبوط في config/cors.php
# ✅ Sanctum stateful domains محددة

# ===== قاعدة البيانات =====
# ✅ مستخدم DB منفصل بصلاحيات محدودة (ليس root)
# ✅ Database backups مجدولة
# ✅ Indexes على الأعمدة المستخدمة في WHERE

# ===== الأداء =====
php artisan config:cache    # يدمج ملفات config في ملف واحد
php artisan route:cache     # يبني فهرس المسارات
php artisan view:cache      # يجمع Blade views
php artisan optimize        # يشغل الثلاث أعلاه + autoloader

# ===== الاختبار =====
php artisan test            # تشغيل كل الاختبارات
php artisan test --coverage # مع نسبة التغطية

# ===== المراقبة =====
# LOG_CHANNEL=stack في .env
# أدوات مثل: Laravel Telescope (dev)، Sentry / Bugsnag (production)
```

---

## سابعاً: كيفية استخدام الـ API

```bash
# تسجيل حساب جديد
curl -X POST https://api.example.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "name": "أحمد محمد",
    "email": "ahmed@example.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'

# تسجيل الدخول
curl -X POST https://api.example.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ahmed@example.com","password":"password123"}'

# طلب محمي (ضع التوكن المُستَلَم هنا)
curl -X GET https://api.example.com/api/v1/auth/me \
  -H "Authorization: Bearer 1|abc123xyz..."

# تسجيل الخروج
curl -X POST https://api.example.com/api/v1/auth/logout \
  -H "Authorization: Bearer 1|abc123xyz..."
```

---

## ملخص المفاهيم الأساسية

|المكوّن|دوره|لماذا مهم؟|
|---|---|---|
|**Migration**|يبني هيكل قاعدة البيانات|التوثيق الحي للـ schema، قابل للـ rollback|
|**Model**|يمثل جدول DB كـ Object|ORM — تتعامل مع البيانات كـ PHP Objects|
|**Form Request**|يتحقق من البيانات الواردة|فصل مسؤولية الـ validation|
|**Controller**|يستقبل الطلب ويوجهه|رقيق — لا يحتوي Business Logic|
|**API Resource**|يحوّل Model إلى JSON|تتحكم بالضبط فيما يُرسَل|
|**Middleware**|يعترض الطلبات|Authentication، Rate Limiting، CORS|
|**Sanctum**|يدير التوكنات|مصادقة بسيطة وآمنة للـ APIs|
|**Handler**|يعالج الاستثناءات|responses موحدة للأخطاء|

هذه البنية قابلة للتوسع بشكل مباشر — أضف `ProductController`، `OrderController` وغيرها بنفس النمط، وكل المكونات تعمل معاً بتناسق كامل.