تمام يا **علي**، خلينا نفصص الموضوع **خطوة خطوة** كأننا بنبني التطبيق مع بعض في **Angular**. الهدف:  
**الـ Component يعرض المنتجات، والـ Service هي اللي توفر البيانات.**

---

# 1️⃣ دور الـ Interface (Product)

الـ **interface** هو مجرد **تعريف لشكل البيانات** (type فقط، لا يحتوي logic).

أنت كتبت:

```ts
export interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  description: string;
  pieces: number;
  category: string;
  quantity: number;   
}
```

هذا معناه أن أي **Product** يجب أن يحتوي هذه الخصائص.

مثال على object صحيح:

```ts
const p: Product = {
  id:1,
  name:"watch",
  price:100,
  image:"img.jpg",
  description:"smart watch",
  pieces:10,
  category:"watch",
  quantity:1
}
```

لو نسيت خاصية، TypeScript سيعطيك error.

---

# 2️⃣ لماذا نستخدم Service أصلاً؟

لو وضعت البيانات داخل component:

```ts
products = [...]
```

ستحدث مشاكل:

- لا يمكن مشاركة البيانات بين components
    
- الكود غير منظم
    
- لو أردت API لاحقًا ستغير كل شيء
    

لذلك نضع **data logic داخل service**.

---

# 3️⃣ كتابة الـ ProductService

الـ service مسؤول عن **إرجاع المنتجات**.

```ts
import { Injectable } from '@angular/core';
import { Product } from '../models/product.model';

@Injectable({
  providedIn: 'root',
})
export class ProductService {

  products: Product[] = [
    {
      id:1,
      name:'smart watch',
      price:120,
      image:'/assets/images/smartwatch.jpg',
      description:'smart watch',
      pieces:10,
      category:'watch',
      quantity:1
    },
    {
      id:2,
      name:'classic watch',
      price:90,
      image:'/assets/images/classicwatch.jpg',
      description:'classic watch',
      pieces:5,
      category:'watch',
      quantity:1
    }
  ];

  getProducts(): Product[] {
    return this.products;
  }

}
```

لاحظ هنا:

```ts
getProducts(): Product[]
```

هذا يعني أن الدالة **ترجع Array من Product**.

---

# 4️⃣ حقن الـ Service داخل Component (Dependency Injection)

داخل component:

```ts
constructor(private productService: ProductService){}
```

Angular هنا يقوم بـ:

1. إنشاء object من ProductService
    
2. تمريره للـ component
    

هذا اسمه **Dependency Injection**.

---

# 5️⃣ لماذا نستخدم ngOnInit ؟

نستخدمها لأننا نريد تحميل البيانات **عند تشغيل component**.

```ts
ngOnInit(){
  this.products = this.productService.getProducts();
}
```

التنفيذ:

```
Angular creates component
↓
ngOnInit runs
↓
getProducts() called
↓
products array filled
```

---

# 6️⃣ الكود الكامل للـ Component

```ts
import { Component, OnInit } from '@angular/core';
import { ProductService } from '../services/product.service';
import { Product } from '../models/product.model';

@Component({
  selector: 'app-cards',
  templateUrl: './cards.html',
  styleUrl: './cards.css',
})
export class Cards implements OnInit {

  products: Product[] = [];

  constructor(private productService: ProductService){}

  ngOnInit(): void {
    this.products = this.productService.getProducts();
  }

  addToCart(product: Product) {
    if(product.pieces > 0){
      product.pieces--;
      console.log(`${product.name} added to cart`);
    }
  }

}
```

---

# 7️⃣ عرض المنتجات في HTML

في template:

```html
<div *ngFor="let product of products">

  <img [src]="product.image">

  <h3>{{product.name}}</h3>

  <p>{{product.price}}</p>

  <p>Pieces: {{product.pieces}}</p>

  <button (click)="addToCart(product)">
    Add To Cart
  </button>

</div>
```

هنا استخدمنا:

### `*ngFor`

لعمل loop على المنتجات.

### `{{ }}`

لإظهار البيانات.

### `[src]`

property binding.

### `(click)`

event binding.

---

# 8️⃣ ماذا يحدث عندما تضغط Add To Cart

```
User clicks button
↓
Angular triggers addToCart()
↓
product.pieces--
↓
Angular change detection updates UI
```

سيظهر العدد الجديد مباشرة.

---

# 9️⃣ الشكل المعماري الصحيح

```
Cards Component
        ↓
ProductService
        ↓
Products Data
```

الـ component **يعرض فقط**  
الـ service **يدير البيانات**.

---

# 🔟 لماذا هذه الطريقة مهمة في المشاريع الحقيقية؟

لأن لاحقًا ستستبدل هذا:

```ts
products = [...]
```

بهذا:

```ts
this.http.get("api/products")
```

بدون تغيير الـ component.

---

# أهم فكرة لازم تطلع بيها

في Angular:
model => datastructure 
service => data / logic 
Component => Ui

```
Component → UI logic
Service → data / business logic
Model → data structure
```

---

لو حابب يا **علي**، أشرح لك الخطوة التي بعدها مباشرة والتي يستخدمها كل مطوري Angular المحترفين:

**كيف نجعل `getProducts()` تجلب البيانات من API حقيقي باستخدام HttpClient و Observable** (وهنا تبدأ Angular الحقيقي).










Task 3 

المقصود بالجملة:

**“Use inside it ProductCard Component and print data”** في **Angular** هو أنك **تقسم الواجهة إلى Components أصغر**.

يعني بدل ما **Product Component يعرض كل تفاصيل المنتج بنفسه**، تخلي فيه **Component أصغر اسمه ProductCard** مسؤول عن عرض **منتج واحد**.

هذه طريقة شائعة جدًا في Angular اسمها **Component Composition**.

خلينا نوضحها عمليًا.

---

# 1️⃣ الفكرة العامة

بدل ما تعمل كده:

```id="m9p2z8"
Cards Component
   └── يعرض كل المنتجات وكل HTML
```

نعمل كده:

```id="3c7u8y"
Cards Component
   └── ProductCard Component
            └── يعرض منتج واحد
```

---

# 2️⃣ إنشاء ProductCard Component

```bash
ng g c components/product-card
```

سيتم إنشاء:

```
product-card.ts
product-card.html
product-card.css
```

---

# 3️⃣ ProductCard يستقبل المنتج

داخل `product-card.ts`

```ts
import { Component, Input } from '@angular/core';
import { Product } from '../../models/product.model';

@Component({
  selector: 'app-product-card',
  templateUrl: './product-card.html',
  styleUrl: './product-card.css',
})
export class ProductCard {

  @Input() product!: Product;

}
```

هنا استخدمنا:

```ts
@Input()
```

لكي **نستقبل البيانات من الـ parent component**.

---

# 4️⃣ عرض البيانات داخل ProductCard

`product-card.html`

```html
<div class="card">

  <img [src]="product.image">

  <h3>{{product.name}}</h3>

  <p>{{product.price}}</p>

  <p>Pieces: {{product.pieces}}</p>

</div>
```

هذا الـ component يعرض **منتج واحد فقط**.

---

# 5️⃣ استخدام ProductCard داخل Cards Component

الآن داخل `cards.html`

بدل كتابة HTML كامل للمنتج:

```html
<div *ngFor="let product of products">

  <app-product-card 
     [product]="product">
  </app-product-card>

</div>
```

---

# 6️⃣ ماذا يحدث هنا؟

Angular يعمل الآتي:

```
Cards Component
     ↓
loop through products
     ↓
for each product
     ↓
create ProductCard
     ↓
send product using @Input
```

يعني:

```
product ----> ProductCard
```

---

# 7️⃣ لماذا نفعل ذلك؟

لأن هذه الطريقة:

1️⃣ تجعل الكود **منظم**  
2️⃣ تجعل الـ components **قابلة لإعادة الاستخدام**  
3️⃣ تجعل المشروع **قابل للتوسعة**

مثال:

نفس `ProductCard` يمكن استخدامه في:

```
home page
products page
related products
cart
```

---

# 🔥 مثال سريع للـ Architecture

```
Cards Component
      │
      │  products[]
      ▼
ProductCard Component
      │
      │  @Input()
      ▼
عرض بيانات المنتج
```

---

✅ **الخلاصة**

الجملة تعني:

> داخل **Product Component** استخدم **ProductCard Component** لعرض كل منتج وإرسال البيانات له.

---

لو حابب يا **علي**، أوريك كمان **3 أخطاء Angular يقع فيها 90% من الناس عند استخدام Components و @Input** — وغالبًا ستقع فيها أثناء المشروع.