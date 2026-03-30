
---

# 📌 Lecture 1 — Laravel Fundamentals: Structure, Routes, MVC & Blade

---

# 🚀 أولًا: Laravel بتعمل إيه أصلًا؟

## 🧠 يعني إيه Laravel؟

**Laravel = PHP Framework**

يعني إيه Framework؟  
هو **هيكل جاهز** بيساعدك تبني Web Application بطريقة منظمة بدل ما تكتب كل حاجة من الصفر.

بدل ما تعمل:

- Routing بنفسك
    
- Connection للداتا بيز
    
- تنظيم ملفات
    
- سيكيوريتي
    
- Authentication
    

Laravel بيعمل دهولك جاهز 💪

---

# 🏗️ ثانيًا: Structure بتاع Laravel

لما تعمل مشروع جديد:

```bash
laravel new project
```

هيطلعلك Structure شبه ده:

```
project/
│
├── app/
│   ├── Http/
│   │   ├── Controllers/
│
├── routes/
│   └── web.php
│
├── resources/
│   └── views/
│
├── public/
│
└── .env
```

## أهم فولدرات لازم تفهمها دلوقتي:

|Folder|مسؤوليته|
|---|---|
|app/|منطق التطبيق (Controllers, Models)|
|routes/|تعريف الروابط|
|resources/views|ملفات HTML (Blade)|
|public/|نقطة دخول الموقع|
|.env|إعدادات الداتا بيز|

---

# 🔁 ثالثًا: Request Lifecycle (الدورة الكاملة)

لما تفتح في المتصفح:

```
http://localhost:8000/home
```

بيحصل الآتي:

```
User Request
     ↓
Route
     ↓
Controller
     ↓
Model (Database)
     ↓
Controller
     ↓
View (Blade)
     ↓
HTML Response
     ↓
Browser
```

## Diagram

```
Browser
   │
   ▼
Route (web.php)
   │
   ▼
Controller
   │
   ▼
Model (DB)
   │
   ▼
View
   │
   ▼
HTML
```

دي اسمها MVC Architecture 👇

---

# 🧩 MVC يعني إيه؟

|الحرف|المعنى|مسؤوليته|
|---|---|---|
|M|Model|يتعامل مع الداتا بيز|
|V|View|يعرض البيانات|
|C|Controller|يربط بينهم|

---

# 🌍 رابعًا: Routes

## يعني إيه Route؟

الـ Route هو:

> الرابط اللي بيستقبل طلب المستخدم ويقرر يوديه فين.

📂 مكانه:

```
routes/web.php
```

## مثال بسيط:

```php
Route::get('/home', function () {
    return "Welcome";
});
```

### التحليل:

- `Route` → Class
    
- `get()` → Method
    
- `/home` → الرابط
    
- function → الكود اللي هيتنفذ
    

---

## Route بيروح لفين؟

### يا إما مباشرة ل View:

```php
Route::get('/home',fn()=> view('home');)

});
```

### يا إما يروح ل Controller:

```php
Route::get('/home', [HomeController::class, 'index']);
```

---

# 🎨 خامسًا: View

## يعني إيه View؟

هي صفحة HTML لكن مكتوبة باستخدام Blade.

📂 مكانها:

```
resources/views/
```

## مثال:

📄 resources/views/home.blade.php

```html
<h1>Welcome Home</h1>
```

---

# 🧠 Blade Template Engine

Blade هو Template Engine خاص بـ Laravel.

بيخليك تكتب PHP داخل HTML بشكل أنضف.

مثال:

```php
<h1>{{ $name }}</h1>
```

`{{ }}` = تطبع متغير

---

# 🎮 سادسًا: Controller

📂 مكانه:

```
app/Http/Controllers
```

## إنشاؤه:

```bash
php artisan make:controller HomeController
```

## مثال:

```php
class HomeController extends Controller
{
    public function index()
    {
        $name = "Ali";

        return view('home', compact('name'));
    }
}
```

## التحليل:

- Controller جاب الداتا
    
- وبعتها للـ View
    
- View طبعتها
    

---

# 🧱 سابعًا: Layout System (Blade Advanced)

عشان ما تكررش الهيدر والفوتر في كل صفحة.

## 🧩 @extends

بتخلي صفحة تورث من Layout.

## 🧩 @section

بتحدد محتوى جزء معين.

## 🧩 @yield

بيستقبل المحتوى من الصفحات الفرعية.

---

## مثال كامل

### 📄 layout.blade.php

```html
<!DOCTYPE html>
<html>
<head>
    <title>My App</title>
</head>
<body>

    <header>
        Header Section
    </header>

    @yield('content')

    <footer>
        Footer Section
    </footer>

</body>
</html>
```

---

### 📄 home.blade.php

```php
@extends('layout')

@section('content')
    <h1>Welcome Ali</h1>
@endsection
```

---

## Flow هنا:

```
home extends layout
home يملأ section content
layout يعرض yield('content')
```

---

# 🎁 Slots

الـ Slots بتستخدم غالبًا مع Components.

مثال:

```php
<x-card>
    This is slot content
</x-card>
```

وفي component:

```php
<div>
    {{ $slot }}
</div>
```

---

# 🧨 مثال شامل كامل

## Route

```php
Route::get('/users', [UserController::class, 'index']);
```

---

## Controller

```php
class UserController extends Controller
{
    public function index()
    {
        $users = ["Ali", "Omar", "Sara"];

        return view('users.index', compact('users'));
    }
}
```

---

## View

📄 resources/views/users/index.blade.php

```php
@extends('layout')

@section('content')

    <h1>Users List</h1>

    @foreach($users as $user)
        <p>{{ $user }}</p>
    @endforeach

@endsection
```

---

# 🔥 الصورة الكاملة النهائية

```
User يدخل /users
        ↓
Route
        ↓
UserController@index
        ↓
Controller يجيب الداتا
        ↓
يبعتها لل View
        ↓
Blade يركبها داخل Layout
        ↓
HTML يرجع للمتصفح
```

---

# 🧠 ملخص ذهبي

|العنصر|مسؤوليته|
|---|---|
|Route|يستقبل الطلب|
|Controller|ينفذ المنطق|
|Model|يتعامل مع DB|
|View|يعرض HTML|
|Blade|Template Engine|
|Layout|يمنع تكرار الكود|

---

# 🎓 عنوان المحاضرة المقترح

> **Laravel Basics Deep Dive — Understanding MVC, Routing & Blade Architecture**

---

لو حابب 👀  
المحاضرة الجاية ممكن نعمل:

- Forms
    
- Passing Data
    
- Request Object
    
- CRUD عملي
    
- أو نرسم Architecture Diagram احترافي
    

قوللي عايز تكمل في إيه يا Backend King 👑🔥