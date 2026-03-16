1️⃣ **Constructor**  
2️⃣ **ngOnInit**
معلومه => ngOnChanges  بيتنفذ قبل ال ngOnInit


---
currenttime:string : "";
gettime(){
let now = new Date();
this.currenttime - now.toLocaltimeString();
}
# أولًا: ما هو Angular Lifecycle ؟

Lifecycle يعني **مراحل حياة الـ Component**.

كل Component يمر بالمراحل التالية:

```text
Create Component
      ↓
Constructor
      ↓
ngOnInit
      ↓
Component Works
      ↓
Destroy Component
```

اليوم سنركز على أول مرحلتين.

---

# 1️⃣ Constructor

## ما هو Constructor ؟

هو **أول شيء يُنفذ عند إنشاء الـ Component**.

Angular ينشئ الـ component ثم ينفذ constructor مباشرة.

### مثال بسيط

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-home',
  template: `<h1>Hello</h1>`
})
export class HomeComponent {

  constructor() {
    console.log("Constructor executed");
  }

}
```

### ماذا يحدث عند تشغيل الصفحة؟

```text
Constructor executed
```

---

## متى نستخدم Constructor ؟

نستخدمه غالبًا في:

### 1️⃣ Dependency Injection

مثال:

```ts
constructor(private productService: ProductService) {}
```

هنا Angular يقوم بـ:

```text
Create ProductService
        ↓
Inject it into component
```

---

## مثال حقيقي

```ts
import { Component } from '@angular/core';
import { ProductService } from '../services/product.service';

@Component({
  selector: 'app-home',
  template: `<p>Products Page</p>`
})
export class HomeComponent {

  constructor(private productService: ProductService) {

    console.log("Component created");

  }

}
```

هنا:

```text
Angular creates component
↓
constructor runs
↓
service injected
```

---

# ⚠️ شيء مهم جدًا

لا تضع **logic ثقيل أو API calls** داخل constructor.

❌ خطأ:

```ts
constructor(){
 this.getProducts();
}
```

لأن Angular لم يجهز الـ component بالكامل بعد.

---

# 2️⃣ ngOnInit

## ما هي ngOnInit ؟

هي **Lifecycle Hook** تعمل بعد أن ينتهي Angular من إنشاء الـ component.

يعني:

```text
constructor
      ↓
ngOnInit
```

---

## مثال بسيط

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-home',
  template: `<h1>Home</h1>`
})
export class HomeComponent implements OnInit {

  constructor() {
    console.log("Constructor");
  }

  ngOnInit(): void {
    console.log("OnInit");
  }

}
```

---

## النتيجة في Console

```text
Constructor
OnInit
```

---

# ماذا نفعل داخل ngOnInit ؟

نستخدمها في:

1️⃣ جلب البيانات من API  
2️⃣ تشغيل logic عند تحميل الصفحة  
3️⃣ تهيئة variables

---

## مثال حقيقي (Products)

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-products',
  template: `
  <div *ngFor="let p of products">
    {{p}}
  </div>
  `
})
export class ProductsComponent implements OnInit {

  products: string[] = [];

  constructor() {
    console.log("Constructor");
  }

  ngOnInit(): void {

    console.log("Loading products...");

    this.products = [
      "Laptop",
      "Phone",
      "Watch"
    ];

  }

}
```

---

# ماذا يحدث خطوة خطوة؟

```text
Angular starts
      ↓
Create component
      ↓
Run constructor
      ↓
Run ngOnInit
      ↓
Load data
      ↓
Render UI
```

---

# Diagram للفهم

```
Component Creation

        Angular
           │
           ▼
     Create Component
           │
           ▼
       Constructor
           │
           ▼
        ngOnInit
           │
           ▼
        Render UI
```

---

# الفرق بين Constructor و ngOnInit

|Constructor|ngOnInit|
|---|---|
|يعمل أولًا|يعمل بعده|
|للـ Dependency Injection|للـ logic|
|لا نضع API calls|نضع API calls|

---

# مثال كامل بسيط

```ts
export class HomeComponent implements OnInit {

  message!: string;

  constructor() {
    console.log("constructor called");
  }

  ngOnInit(): void {

    console.log("ngOnInit called");

    this.message = "Welcome Ali";

  }

}
```

---

# Console Output

```text
constructor called
ngOnInit called
```

---

# الزتونة (مهم للمذاكرة)

```text
Constructor → create component
ngOnInit → initialize component
```

---
---
---
---
---
---

بص مش شرط ان  ال app.ts  يكون ال parent يعني ممكن يكون any component is the parent for any component وبالتالي هنكتب البروبرتيز  في ال parentComp.html  اللي هو الكومينيكيشن بينهم 

---
---
---
---
---
---



```
ngOnChanges
ngAfterViewInit
ngOnDestroy
```



---

# أولاً: ngOnChanges

## الفكرة ببساطة

`ngOnChanges` يتم تشغيلها عندما **يتغير Input قادم من الـ Parent Component**.

يعني:

```
Parent → يرسل data
Child → تستقبلها
Angular → يشغل ngOnChanges
```

---

## متى تستخدمها؟

تستخدم عندما تريد:

- مراقبة تغير قيمة `@Input`
    
- تحديث شيء عند تغير الداتا
    
- تنفيذ logic عندما تتغير قيمة من الـ parent
    

---

## مثال عملي

### Parent Component

```ts
export class AppComponent {

  productName = "Watch";

  changeProduct(){
    this.productName = "Phone";
  }

}
```

### HTML

```html
<app-child [name]="productName"></app-child>

<button (click)="changeProduct()">
Change Product
</button>
```

---

### Child Component

```ts
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `<h2>{{name}}</h2>`
})
export class ChildComponent implements OnChanges {

  @Input() name!: string;

  ngOnChanges(changes: SimpleChanges) {

    console.log("Input Changed");

    console.log(changes['name']);

  }

}
```

---

## ماذا يحدث؟

```
Parent change value
        ↓
Angular detects change
        ↓
ngOnChanges runs
        ↓
Child reacts
```

---

## Diagram

```
Parent Component
      │
      │ send @Input
      ▼
Child Component
      │
      ▼
ngOnChanges()
```

---

# ثانياً: ngAfterViewInit

## الفكرة

يتم تشغيلها **بعد أن يتم إنشاء الـ View بالكامل**. بعد ما نعمل render for the component 

يعني:

```
HTML rendered
DOM created
View ready
→ ngAfterViewInit runs
```

---

## متى نستخدمها؟

تستخدم عندما تحتاج:

- التعامل مع **DOM**
    
- استخدام `ViewChild`
    
- التعامل مع عنصر HTML
    

---

## مثال

### HTML

```html
<input #myInput type="text">
```

---

### Component

```ts
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-demo',
  templateUrl: './demo.html'
})
export class DemoComponent implements AfterViewInit {

  @ViewChild('myInput') input!: ElementRef;

  ngAfterViewInit(){

    this.input.nativeElement.focus();

  }

}
```

---

## ماذا يحدث هنا؟

Angular:

```
create component
       ↓
render HTML
       ↓
create DOM
       ↓
ngAfterViewInit
```

ثم نقوم بعمل:

```
focus on input
```

---

## Diagram

```
Component Created
        │
        ▼
Template Rendered
        │
        ▼
View Ready
        │
        ▼
ngAfterViewInit()
```

---

# ثالثاً: ngOnDestroy

## الفكرة

تعمل عندما **يتم حذف component من الصفحة**.

مثال:

- تغيير route
    
- إغلاق component
    
- إزالة component من DOM
    

---

## متى تستخدمها؟

لتنظيف الموارد مثل:

- unsubscribe من observable
    
- إيقاف timers
    
- إغلاق WebSocket
    

---

## مثال

```ts
import { Component, OnDestroy } from '@angular/core';

@Component({
  selector: 'app-demo',
  template: `<h2>Demo</h2>`
})
export class DemoComponent implements OnDestroy {

  intervalId: any;

  constructor(){

    this.intervalId = setInterval(()=>{
      console.log("Running...");
    },1000);

  }

  ngOnDestroy(){

    clearInterval(this.intervalId);

    console.log("Component destroyed");

  }

}
```

---

## ماذا يحدث؟

```
Component removed
       ↓
Angular calls
       ↓
ngOnDestroy()
```

---

## Diagram

```
Component Active
      │
      ▼
User leaves page
      │
      ▼
Angular destroys component
      │
      ▼
ngOnDestroy()
```

---

# الآن: Angular Lifecycle الكامل

هذا هو التسلسل الحقيقي.

```
Constructor
     │
     ▼
ngOnChanges
     │
     ▼
ngOnInit
     │
     ▼
ngDoCheck
     │
     ▼
ngAfterContentInit
     │
     ▼
ngAfterContentChecked
     │
     ▼
ngAfterViewInit
     │
     ▼
ngAfterViewChecked
     │
     ▼
Component Running
     │
     ▼
ngOnDestroy
```

---

# Diagram أسهل للفهم

```
Create Component
      │
      ▼
Constructor
      │
      ▼
ngOnChanges
      │
      ▼
ngOnInit
      │
      ▼
Render Template
      │
      ▼
ngAfterViewInit
      │
      ▼
Component Works
      │
      ▼
User leaves page
      │
      ▼
ngOnDestroy
```

---

# الزتونة (الأهم للمذاكرة)

|Hook|يستخدم في|
|---|---|
|**ngOnChanges**|عندما يتغير `@Input`|
|**ngAfterViewInit**|بعد إنشاء الـ View وDOM|
|**ngOnDestroy**|تنظيف الموارد عند حذف component|

---

لو حابب يا علي، أشرح لك بعدها **Angular Lifecycle بطريقة تجعلك تفهمه 100% في 10 دقائق فقط** مع **رسمة واحدة تحفظها وتجاوب بها أي سؤال في الانترفيو**.