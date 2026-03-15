# 1️⃣ ما هو Routing في Angular ؟

**Routing** يعني:

> التنقل بين الصفحات داخل التطبيق بدون عمل reload للصفحة.

مثال موقع عادي:

example.com/home  
example.com/about  
example.com/contact

Angular يغير **الـ Component المعروض** حسب الرابط.





```
```

```
```

---

# أولاً: فكرة Routing في Angular 

Angular يربط:

```
URL  →  Component
```

مثال:

```
/cards   →  CardsComponent
/navbar  →  NavbarComponent
```

---

# ثانياً: ملف الـ routes

في مشروعك الملف اسمه:

```
app.routes.ts
```

الشكل الصحيح يكون هكذا:

```ts
import { Routes } from '@angular/router';
import { Cards } from './Components/cards/cards';
import { Navbar } from './Components/navbar/navbar';

export const routes: Routes = [

  { path: '', component: Navbar },

  { path: 'cards', component: Cards },

];
```

---

# معنى كل جزء

### Routes

```ts
import { Routes } from '@angular/router';
```

ده **type** خاص بالـ routing.

---

### تعريف routes

```ts
export const routes: Routes = [
```

هنا نكتب **كل صفحات التطبيق**.

---

### path

```ts
path: 'cards'
```

ده الرابط.

يعني لما تفتح:

```
localhost:4200/cards
```

---

### component

```ts
component: Cards
```

Angular يعرض:

```
CardsComponent
```


---

# 3️⃣ إنشاء Routing

عند إنشاء مشروع Angular نكتب:

```
ng new project-name --routing
```

Angular سينشئ ملف:

```
app-routing.module.ts
```




```

```

---

# 5️⃣ Router Outlet

لكي يعرض Angular الصفحة نضع:

### app.component.html

```html
<router-outlet></router-outlet>
```

هذا هو المكان الذي سيعرض فيه Angular الـ Component.

---

# 6️⃣ الانتقال بين الصفحات

نستخدم:

```
routerLink
```

### مثال

```html
<a routerLink="/home">Home</a>
<a routerLink="/about">About</a>
```

عند الضغط سيغير Angular الصفحة.

---

# 7️⃣ مثال كامل

### routes

```ts
const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'products', component: ProductsComponent },
  {path:'' ,component : Cards}
];
```

---

### HTML

```html
<nav>
  <a routerLink="/home">Home</a>
  <a routerLink="/products">Products</a>
  <a routerLoink="/Cards"> Cards</a>
</nav>

<router-outlet></router-outlet>
```

---

# 8️⃣ Default Route

لو فتح المستخدم الموقع بدون مسار.

```
localhost:4200
```

نحدد الصفحة الافتراضية.

```ts
{ path: '', component: HomeComponent }
```

---

# 9️⃣ Wildcard Route (صفحة 404)
دا بنخليه اخر حاجه 

لو كتب المستخدم رابط غير موجود.

```ts
{ path: '**', component: NotFoundComponent },

```

---

# 🔟 Route Parameters

نستخدمها لإرسال **ID** مثلاً.

### route

```ts
{ path: 'product/:id', component: ProductDetailsComponent }
```

---

### مثال الرابط

```
/product/10
```

---

### قراءة الـ id

```ts
import { ActivatedRoute } from '@angular/router';

constructor(private route: ActivatedRoute){}

ngOnInit(){
  let id = this.route.snapshot.paramMap.get('id');
  console.log(id);
}
```

---

# 1️⃣1️⃣ Query Parameters

مثال:

```
/products?category=phone
```

---

### قراءتها

```ts
this.route.queryParams.subscribe(params => {
  console.log(params['category']);
});
```

---

# 1️⃣2️⃣ التنقل باستخدام TypeScript

بدلاً من HTML.

### مثال

```ts
import { Router } from '@angular/router';

constructor(private router: Router){}

goHome(){
  this.router.navigate(['/home']);
}
```

---

### HTML

```html
<button (click)="goHome()">Home</button>
```

---

# 1️⃣3️⃣ Child Routes

نستخدمها للصفحات المتداخلة.

### مثال

```
dashboard
   ├ users
   ├ settings
```

---

### routes

```ts
{
  path: 'dashboard',
  component: DashboardComponent,
  children: [
    { path: 'users', component: UsersComponent },
    { path: 'settings', component: SettingsComponent }
  ]
}
```

---

# 1️⃣4️⃣ Route Guard

يمنع المستخدم من دخول صفحة بدون صلاحية.

مثال:

```
admin page
```

---

### guard

```ts
canActivate(){
  return false;
}
```

---

### route

```ts
{
  path: 'admin',
  component: AdminComponent,
  canActivate: [AuthGuard]
}
```

---

# 1️⃣5️⃣ Lazy Loading

تحميل الموديول فقط عند الحاجة.

بدل تحميل كل التطبيق مرة واحدة.

---

### مثال

```ts
{
  path: 'admin',
  loadChildren: () => import('./admin/admin.module')
  .then(m => m.AdminModule)
}
```

---

# ملخص الـ Routing

|المفهوم|الاستخدام|
|---|---|
|routes|تحديد الصفحات|
|router-outlet|مكان عرض الصفحة|
|routerLink|الانتقال بين الصفحات|
|route params|ارسال id|
|query params|ارسال بيانات|
|guards|حماية الصفحات|
|lazy loading|تحسين الأداء|

---

# مثال Routing حقيقي بسيط

### routes

```ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'cards', component: CardsComponent },
  { path: 'profile', component: ProfileComponent },
  { path: '**', component: NotFoundComponent }
];
```

---

### navigation

```html
<a routerLink="/">Home</a>
<a routerLink="/cards">Cards</a>
<a routerLink="/profile">Profile</a>

<router-outlet></router-outlet>
```

---

💡 بما إنك يا **علي** بتعمل **CardsComponent** حالياً في Angular، لو حابب أشرح لك كمان:

- **كيف نعمل Routing بين Cards و CardDetails**
    
- **إرسال ID للكارد**
    
- **فتح صفحة تفاصيل الكارد**
    

وده مثال **يستخدم كثير جداً في المشاريع الحقيقية**.