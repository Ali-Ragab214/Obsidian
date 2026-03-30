# Laravel — المحاضرة الأولى 🚀

خليني أشرحلك كل حاجة بالتفصيل يا علي، خطوة خطوة.

---الكلام جاهز! هبدأ بالشرح الكامل مع الديجرامز.

---

## ١. Laravel بتعمل إيه؟

Laravel هو **PHP framework** بيساعدك تبني web applications بسرعة واحترافية. بدل ما تكتب كل حاجة من الصفر (database connections، routing، security...) — Laravel بيديك كل ده جاهز ومنظم.

الفكرة الأساسية: Laravel بيتبع نمط **MVC** — Model, View, Controller — وده هو العمود الفقري لكل حاجة هتشوفها.---

## ٢. Structure بتاع Laravel

لما بتعمل project جديد، هتلاقي مجلدات كتير. مش لازم تحفظهم كلهم دلوقتي، بس لازم تعرف الأهم.> اضغط على أي مجلد في الديجرام عشان تسألني عنه تفصيلياً.

---

## ٣. الـ Routes — إيه هي وفين وليه؟

الـ Route هو الحارس بتاع التطبيق. لما حد يكتب URL في البراوزر، الـ Router هو اللي بيقول "الـ URL ده روح لفلان".

فكّر فيه كـ **فهرس** — URL → ماذا يحدث.

**فين بتكتبه؟** في ملف `routes/web.php`

```php
// أبسط route ممكن
Route::get('/hello', function () {
    return 'أهلاً يا علي!';
});

// route بيروح لـ Controller
Route::get('/users', [UserController::class, 'index']);

// route بمعامل ديناميكي
Route::get('/users/{id}', [UserController::class, 'show']);
```

الكلاس `Route` ده مش بتعمله `new` — هو **Facade**، يعني Laravel بيديك طريقة بتكتب بيها `Route::get(...)` وهو بيتولى الباقي من تحت.

**الـ HTTP methods المتاحة:**

```php
Route::get('/path',    ...);  // جيب بيانات
Route::post('/path',   ...);  // أرسل بيانات (form)
Route::put('/path',    ...);  // تعديل كامل
Route::patch('/path',  ...);  // تعديل جزئي
Route::delete('/path', ...);  // حذف
```

---

## ٤. الـ View — إيه هو وازاي بستخدمه؟

الـ View هو الوجه — اللي المستخدم بيشوفه. في Laravel بنستخدم **Blade** كـ templating engine.

**فين بتحطه؟** في `resources/views/` بامتداد `.blade.php`

```
resources/views/
├── layouts/
│   └── app.blade.php     ← الـ layout الأساسي
├── users/
│   ├── index.blade.php   ← قايمة المستخدمين
│   └── show.blade.php    ← بيانات مستخدم واحد
└── welcome.blade.php
```

**ازاي بتبعتله داتا من الـ Route؟**

```php
// من الـ Route مباشرة
Route::get('/welcome', function () {
    return view('welcome', ['name' => 'علي']);
});

// وفي welcome.blade.php:
<h1>أهلاً {{ $name }}</h1>
```

`{{ }}` دي Blade syntax بتطبع المتغير وبتعمله HTML escaping تلقائياً للأمان.

---

## ٥. الـ Controller — القلب النابض

الـ Controller هو اللي بيجمع الـ logic. بياخد الـ request، بيجيب الداتا من الـ Model، وبيبعتها للـ View.

**ازاي بتعمل Controller؟**

```bash
php artisan make:controller UserController
```

```php
// app/Http/Controllers/UserController.php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    // بيرجع كل المستخدمين
    public function index()
    {
        $users = User::all(); // بيجيب من الـ DB
        return view('users.index', compact('users'));
    }

    // بيرجع مستخدم واحد
    public function show($id)
    {
        $user = User::findOrFail($id);
        return view('users.show', compact('user'));
    }
}
```

---

## ٦. Blade Templating — `@extends`, `@section`, `@yield`, `@slot`

ده أقوى حاجة في Blade. بيخليك تعمل **layout واحد** وكل الصفحات ترثه.

**الفكرة:**

```
Layout (الهيكل الثابت: navbar + footer)
    └── كل صفحة بتحشي المحتوى في المكان الصح
```

### الـ Layout الأساسي

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'موقعي')</title>
</head>
<body>
    @include('components.navbar')

    <main>
        @yield('content')
    </main>

    <footer>
        @yield('footer', '<p>© 2025</p>')
    </footer>
</body>
</html>
```

### الصفحة الابنة

```blade
{{-- resources/views/users/index.blade.php --}}
@extends('layouts.app')

@section('title', 'قايمة المستخدمين')

@section('content')
    <h1>المستخدمين</h1>
    @foreach ($users as $user)
        <p>{{ $user->name }}</p>
    @endforeach
@endsection
```

**شرح كل directive:**

|Directive|بيعمل إيه|
|---|---|
|`@extends('layout')`|"أنا وارث الـ layout ده"|
|`@yield('name')`|"في الـ layout — حط المحتوى هنا"|
|`@section('name')` ... `@endsection`|"في الصفحة — ده المحتوى اللي هيتحط في الـ yield"|
|`@include('component')`|"ضم ملف blade تاني هنا"|

---

### الـ Components والـ Slots

لو عندك جزء صغير متكرر (زي card أو alert)، بتعمله **component**:

```blade
{{-- resources/views/components/card.blade.php --}}
<div class="card">
    <h3>{{ $title }}</h3>
    <div>{{ $slot }}</div>
</div>
```

وبتستخدمه كده:

```blade
<x-card title="مرحباً">
    ده المحتوى الجواه
</x-card>
```

الـ `$slot` بياخد كل حاجة كتبتها جوه الـ tag.

---

## ٧. الـ Full Cycle — كل حاجة مع بعض---

## ٨. المثال الشامل — صفحة عرض المستخدمين

خليني أربط كل اللي اتكلمنا فيه في مثال واحد متكامل.

**الهدف:** صفحة `/users` تعرض قايمة المستخدمين بـ layout موحد.

### أولاً — الـ Route

```php
// routes/web.php
use App\Http\Controllers\UserController;

Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{id}', [UserController::class, 'show']);
```

### ثانياً — الـ Controller

```php
// app/Http/Controllers/UserController.php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }

    public function show($id)
    {
        $user = User::findOrFail($id);
        return view('users.show', compact('user'));
    }
}
```

### ثالثاً — الـ Layout

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <title>@yield('title', 'موقعي')</title>
</head>
<body>
    <nav>
        <a href="/users">المستخدمين</a>
    </nav>

    <main>
        @yield('content')
    </main>

    <footer>جميع الحقوق محفوظة</footer>
</body>
</html>
```

### رابعاً — الـ View (قايمة)

```blade
{{-- resources/views/users/index.blade.php --}}
@extends('layouts.app')

@section('title', 'المستخدمين')

@section('content')
    <h1>قايمة المستخدمين</h1>

    @forelse ($users as $user)
        <div>
            <a href="/users/{{ $user->id }}">
                {{ $user->name }}
            </a>
        </div>
    @empty
        <p>مفيش مستخدمين لحد دلوقتي</p>
    @endforelse
@endsection
```

### خامساً — الـ View (تفاصيل)

```blade
{{-- resources/views/users/show.blade.php --}}
@extends('layouts.app')

@section('title', $user->name)

@section('content')
    <h1>{{ $user->name }}</h1>
    <p>{{ $user->email }}</p>
    <a href="/users">← رجوع</a>
@endsection
```

---

## ملخص المحاضرة

```
Browser → routes/web.php → Controller → Model → View → HTML
```

|المكون|فين|بيعمل إيه|
|---|---|---|
|Route|`routes/web.php`|بيربط الـ URL بالـ Controller|
|Controller|`app/Http/Controllers/`|بيجيب الداتا ويبعتها للـ View|
|Model|`app/Models/`|بيتكلم مع الـ Database|
|View|`resources/views/`|بيعرض الـ HTML للمستخدم|
|`@extends`|في الـ View|"أنا وارث الـ layout ده"|
|`@yield`|في الـ Layout|"المحتوى هيتحط هنا"|
|`@section`|في الـ View|"ده المحتوى اللي ييجي في الـ yield"|

---

اتفضل الملاحظة بفورمات Obsidian 👇اتفضل يا علي! 🎉

الملف جاهز تحطه في Obsidian مباشرة — فيه كل حاجة اتكلمنا فيها مع code blocks وجداول وcallouts.

**عنوان المحاضرة المقترح:**

> `Laravel — المحاضرة الأولى: MVC, Routes, Views, Controllers & Blade`

**اللي غطيناه:**

- إيه هو Laravel وليه بنستخدمه
- الـ Structure الكاملة بتاعة الـ project
- الـ Routes من الألف للياء
- الـ Controller وازاي بيجيب الداتا
- الـ Blade Views وطريقة عرض البيانات
- نظام الـ Layout بـ `@extends`, `@yield`, `@section`
- الـ Components والـ Slots
- مثال شامل بيربط كل حاجة

لو في حاجة مش واضحة أو عايز أعمق في موضوع معين، قولي! 💪