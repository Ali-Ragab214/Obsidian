تمام 👌 خلينا نشرح **Laravel Authorization Gates** بشكل عملي جدًا وبالعربي.

---

# أولاً: ما هي الـ Gates ؟

**Gates** هي طريقة بسيطة في Laravel للتحكم في الصلاحيات Authorization.  
يعني تحدد: هل المستخدم مسموح له يعمل أكشن معين ولا لا.

مثال:

- هل يقدر يحذف بوست؟
    
- هل يقدر يدخل لوحة التحكم؟
    
- هل يقدر يعدل بيانات مستخدم تاني؟
    

👉 تخيلها كأنها **if statements مركزية** خاصة بالصلاحيات.

---

## الفرق بين Gates و Policies

|Gates|Policies|
|---|---|
|بسيطة وسريعة|منظمة وكبيرة|
|بتستخدم لأكشن معين|بتستخدم لموديل كامل|
|Closure واحدة|Class كامل فيه ميثودز|
|مناسبة للمشاريع الصغيرة والمتوسطة|مناسبة للمشاريع الكبيرة|

لو عندك صلاحيات قليلة → استخدم Gates  
لو عندك CRUD كامل على Model → استخدم Policies

---

# أين نكتب الـ Gates ؟

في الملف:

```php
app/Providers/AuthServiceProvider.php
```

داخل الدالة:

```php
public function boot()
{
    //
}
```

---

# 🧪 Use Case 1 — Role Based Gate

### مثال: author يعدل بوستاته – admin يعدل أي بوست

## 1️⃣ تسجيل الـ Gate

```php
use Illuminate\Support\Facades\Gate;
//in app provider
public function boot()
{
    Gate::define('update-post', function ($user, $post) {
        return $user->id === $post->user_id || $user->role === 'admin';
    });
}
```

---

## 2️⃣ استخدامه داخل Controller

```php
use Illuminate\Support\Facades\Gate;

public function update(Post $post)
{
    if (Gate::denies('update-post', $post)) {
        abort(403);
    }

    // update logic
}
```

### طريقة مختصرة Laravel style 🔥

```php
$this->authorize('update-post', $post);
```

---

# 🧪 Use Case 2 — Ability / Permission Gate

### مثال: دخول Dashboard

لو عندك عمود permissions في users:

```php
permissions = ['view-dashboard', 'edit-users']
```

## تسجيل Gate

```php
Gate::define('view-dashboard', function ($user) {
    return in_array('view-dashboard', $user->permissions);
});
```

---

## استخدامه في Route

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('can:view-dashboard');
```

🔥 Middleware جاهز من Laravel.

---

# 🧪 Use Case 3 — Model Gate (Ownership + Status)

### مثال: حذف بوست فقط لو:

- صاحب البوست
    
- والبوست مش منشور
    

```php
Gate::define('delete-post', function ($user, $post) {
    return $user->id === $post->user_id && $post->status !== 'published';
});
```

---

## استخدامه في Controller

```php
public function destroy(Post $post)
{
    $this->authorize('delete-post', $post);

    $post->delete();

    return back();
}
```

---

# استخدام Gates داخل Blade 🎨

### إظهار زر حسب الصلاحية

 مهمههههههههههههههههههههههههههههههههههه جداااااااااااااااااااااااااااااااااا
```blade
@can('update-post', $post)
    <a href="/posts/{{$post->id}}/edit">Edit</a>
@endcan
```

### أو العكس

```blade
@cannot('delete-post', $post)
    <p>You cannot delete this post</p>
@endcannot
```

---

# تعريف Gate باستخدام Closure واستخدام Gate::allows

## تعريف Gate

```php
Gate::define('is-admin', function ($user) {
    return $user->role === 'admin';
});
```

## Controller

```php
use Illuminate\Support\Facades\Gate;

public function index()
{
    if (Gate::allows('is-admin')) {
        return "Welcome Admin";
    }

    return "Access denied";
}
```

---

# طرق Gate المهمة

|Method|وظيفة|
|---|---|
|Gate::allows|يرجع true/false|
|Gate::denies|العكس|
|Gate::authorize|يرمي 403 تلقائي|
|@can|Blade|
|middleware('can:')|Routes|

---

# 🧪 اختبار Gates (Testing)

Laravel يسمح باختبار الصلاحيات بسهولة.

## مثال Unit Test

```php
use Illuminate\Support\Facades\Gate;
use App\Models\User;
use App\Models\Post;

public function test_user_can_update_own_post()
{
    $user = User::factory()->create();
    $post = Post::factory()->create([
        'user_id' => $user->id
    ]);

    $this->actingAs($user);

    $this->assertTrue(
        Gate::allows('update-post', $post)
    );
}
```

---

## اختبار منع المستخدم

```php
public function test_user_cannot_delete_published_post()
{
    $user = User::factory()->create();

    $post = Post::factory()->create([
        'user_id' => $user->id,
        'status' => 'published'
    ]);

    $this->actingAs($user);

    $this->assertFalse(
        Gate::allows('delete-post', $post)
    );
}
```

---

# خلاصة سريعة 💡

Gates = نظام الصلاحيات البسيط في Laravel

تُستخدم في:

- Controllers ✔️
    
- Routes ✔️
    
- Blade ✔️
    
- Tests ✔️
    

لو المشروع كبر بعد كده → نحول لـ Policies.

---

لو حابب أشرح Policies بعد Gates مباشرة عشان تكمل الصورة الاحترافية قولّي 😎