# Angular — Day 2

## المواضيع

- [[#1. Data Binding]]
- [[#2. Control Flow]]
- [[#3. Routing]]

---

## 1. Data Binding

> الـ Binding هو الطريقة اللي Angular بيها بيربط الـ Component class بالـ Template (HTML).

### أنواع الـ Binding

|النوع|الـ Syntax|الاتجاه|
|---|---|---|
|Interpolation|`{{ value }}`|Component → View|
|Property Binding|`[property]="value"`|Component → DOM|
|Event Binding|`(event)="handler()"`|View → Component|
|Two-Way Binding|`[(ngModel)]="value"`|Component ↔ View|

---

### 1.1 Interpolation `{{ }}`

لعرض بيانات من الـ component في الـ template.

```typescript
// component.ts
title = 'Ahmed';
age = 25;
```

```html
<!-- template.html -->
<h1>Hello, {{ title }}</h1>
<p>Age: {{ age }}</p>
<p>Double: {{ age * 2 }}</p>
```

---

### 1.2 Property Binding `[ ]`

لربط خاصية HTML element بقيمة من الـ component.

```typescript
// component.ts
imgUrl = 'logo.png';
isDisabled = true;
```

```html
<!-- template.html -->
<img [src]="imgUrl">
<button [disabled]="isDisabled">Click</button>
```

---

### 1.3 Event Binding `( )`

للاستجابة لأحداث المستخدم (click, input, ...).

```typescript
// component.ts
count = 0;

onClick() {
  this.count++;
}
```

```html
<!-- template.html -->
<button (click)="onClick()">+1</button>
<p>{{ count }}</p>
```

---

### 1.4 Two-Way Binding `[( )]`

بيربط الـ input بالـ component بشكل متزامن في الاتجاهين.

> ⚠️ يحتاج إضافة `FormsModule` في الـ imports

```typescript
// component.ts
username = '';
```

```html
<!-- template.html -->
<input [(ngModel)]="username">
<p>Hello, {{ username }}</p>
```

```typescript
// app.module.ts أو standalone component
import { FormsModule } from '@angular/forms';
```

---

### مثال كامل — Counter Component

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count }}</h2>
    <button (click)="increment()">+</button>
    <button (click)="decrement()">−</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  count = 0;

  increment() { this.count++; }
  decrement() { this.count--; }
  reset()     { this.count = 0; }
}
```

---

## 2. Control Flow

> Angular 17+ بيستخدم Built-in Control Flow بدل الـ Directives القديمة (`*ngIf`, `*ngFor`).

### مقارنة القديم بالجديد

|القديم (Angular 16-)|الجديد (Angular 17+)|
|---|---|
|`*ngIf="condition"`|`@if (condition) { }`|
|`*ngFor="let x of list"`|`@for (x of list; track x.id) { }`|
|`[ngSwitch]`|`@switch (val) { @case ('x') { } }`|

---

### 2.1 `@if`

```typescript
// component.ts
isLoggedIn = false;
```

```html
<!-- template.html -->
@if (isLoggedIn) {
  <p>Welcome back!</p>
} @else if (isPending) {
  <p>Pending approval...</p>
} @else {
  <p>Please log in</p>
}
```

---

### 2.2 `@for`

> ⚠️ `track` مطلوب دايمًا — بيحسن أداء الـ rendering

```typescript
// component.ts
products = [
  { id: 1, name: 'Laptop' },
  { id: 2, name: 'Phone' },
];
```

```html
<!-- template.html -->
@for (p of products; track p.id) {
  <li>{{ p.name }}</li>
} @empty {
  <p>No products found</p>
}
```

#### المتغيرات الجاهزة جوه `@for`

```html
@for (item of items; track item.id;
      let i = $index,
          first = $first,
          last = $last,
          even = $even,
          odd = $odd) {
  <li [class.highlight]="even">
    {{ i + 1 }}. {{ item.name }}
  </li>
}
```

|المتغير|المعنى|
|---|---|
|`$index`|رقم العنصر (يبدأ من 0)|
|`$first`|هل هو أول عنصر؟|
|`$last`|هل هو آخر عنصر؟|
|`$even`|هل رقمه زوجي؟|
|`$odd`|هل رقمه فردي؟|

---

### 2.3 `@switch`

```typescript
// component.ts
role = 'admin';
```

```html
<!-- template.html -->
@switch (role) {
  @case ('admin') {
    <p>Admin Panel</p>
  }
  @case ('user') {
    <p>User Dashboard</p>
  }
  @default {
    <p>Guest View</p>
  }
}
```

---

### مثال كامل — Task List Component

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-tasks',
  imports: [FormsModule],
  template: `
    <h2>Tasks ({{ tasks.length }})</h2>

    @if (tasks.length === 0) {
      <p>No tasks yet!</p>
    }

    @for (task of tasks; track task.id) {
      <div [class.done]="task.done">
        <input type="checkbox" [(ngModel)]="task.done">
        {{ task.title }}
      </div>
    } @empty {
      <p>Add your first task</p>
    }
  `
})
export class TasksComponent {
  tasks = [
    { id: 1, title: 'Learn Angular', done: false },
    { id: 2, title: 'Build a project', done: false },
  ];
}
```

---

## 3. Routing

> الـ Router بيخلي Angular Single Page Application (SPA) — الانتقال بين الصفحات من غير reload.

### الفكرة الأساسية

```
URL: /products
       ↓
   Router يشوف الـ routes
       ↓
   يعرض ProductsComponent
   جوه <router-outlet>
```

---

### خطوات الإعداد

#### خطوة 1 — تعريف الـ Routes

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { ProductsComponent } from './products/products.component';
import { ProfileComponent } from './profile/profile.component';
import { NotFoundComponent } from './not-found/not-found.component';

export const routes: Routes = [
  { path: '',           component: HomeComponent },
  { path: 'about',     component: AboutComponent },
  { path: 'products',  component: ProductsComponent },
  { path: 'profile/:id', component: ProfileComponent }, // Route Param
  { path: '**',        component: NotFoundComponent },  // Wildcard
];
```

---

#### خطوة 2 — إضافة Router في `app.config`

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes)
  ]
};
```

---

#### خطوة 3 — الـ Template (app.component.html)

```html
<!-- app.component.html -->
<nav>
  <a routerLink="/"        routerLinkActive="active">Home</a>
  <a routerLink="/about"   routerLinkActive="active">About</a>
  <a routerLink="/products" routerLinkActive="active">Products</a>
</nav>

<!-- هنا Angular بيعرض الـ component المناسب -->
<router-outlet />
```

> ⚠️ لازم تضيف `RouterModule` أو `RouterLink` في الـ imports لو standalone component

---

#### خطوة 4 — قراءة Route Params

```typescript
// profile.component.ts
import { Component } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-profile',
  template: `<h2>Profile ID: {{ userId }}</h2>`
})
export class ProfileComponent {
  userId: string;

  constructor(private route: ActivatedRoute) {
    // URL: /profile/42  →  userId = '42'
    this.userId = this.route.snapshot.paramMap.get('id')!;
  }
}
```

---

#### خطوة 5 — Navigation بالكود

```typescript
// any.component.ts
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({ selector: 'app-any', template: `...` })
export class AnyComponent {
  constructor(private router: Router) {}

  goToProfile(id: number) {
    this.router.navigate(['/profile', id]);
    // → /profile/42
  }

  goHome() {
    this.router.navigateByUrl('/');
  }
}
```

---

### مقارنة طرق الـ Navigation

|الطريقة|الاستخدام|المكان|
|---|---|---|
|`routerLink="/about"`|رابط ثابت|Template|
|`[routerLink]="['/profile', id]"`|رابط ديناميكي|Template|
|`routerLinkActive="active"`|إضافة class للـ active link|Template|
|`router.navigate(['/about'])`|navigation بالكود|TypeScript|
|`router.navigateByUrl('/about')`|navigation بـ string|TypeScript|

---

### Query Params (إضافي)

```typescript
// إرسال query params
this.router.navigate(['/products'], {
  queryParams: { category: 'electronics', page: 1 }
});
// → /products?category=electronics&page=1

// قراءة query params
constructor(private route: ActivatedRoute) {
  this.route.queryParams.subscribe(params => {
    console.log(params['category']); // 'electronics'
  });
}
```

---

## ملخص عام

```
Binding
├── {{ }}        Interpolation    → عرض بيانات
├── [ ]          Property         → ربط خصائص DOM
├── ( )          Event            → الاستجابة لأحداث
└── [( )]        Two-Way          → ربط ثنائي الاتجاه

Control Flow (Angular 17+)
├── @if / @else if / @else
├── @for ... track   (+ @empty)
└── @switch / @case / @default

Routing
├── app.routes.ts    → تعريف الـ routes
├── provideRouter()  → إضافة في app.config
├── <router-outlet>  → مكان عرض الـ components
├── routerLink       → الانتقال من الـ template
├── Router.navigate  → الانتقال من الكود
└── ActivatedRoute   → قراءة params من الـ URL
```