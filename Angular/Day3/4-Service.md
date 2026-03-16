

---

# 1️⃣ يعني ايه Service في Angular؟

الـ **Service** في Angular هو **كلاس عادي** بنحط فيه منطق أو بيانات نحتاج نستخدمها في أكتر من Component.

بدل ما تكرر الكود في كل Component، تحطه في Service وتستخدمه في أي مكان.

### مثال بسيط

مثلاً عندك **Shopping Cart**  
العمليات بتاعتها:

- add product
    
- remove product
    
- get products
    

بدل ما تكتبهم في كل component، تحطهم في Service.

---

# 2️⃣ ليه نستخدم Service؟

نستخدمه لما يكون عندك:

- **logic مش خاص بcomponent واحد**
    
- **data مشتركة**
    
- **API calls**
    
- **state management بسيط**
    

أمثلة:

|الحالة|نستخدم Service؟|
|---|---|
|API request|✅|
|data بين components|✅|
|business logic|✅|
|UI logic خاص بالcomponent|❌|

---

# 3️⃣ هل Service بين 2 Components بس؟

❌ لا

Service ممكن يستخدمه:

- 2 components
    
- 5 components
    
- كل ال application
    

هو **مش محدود بعدد components**.

---

# 4️⃣ إنشاء Service

في Angular نعمله بالأمر:

```bash
ng generate service product
```

هيعمل ملف:

```
product.service.ts
```

---

# 5️⃣ شكل Service

```ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ProductService {

  products = ["Laptop", "Phone", "Tablet"];

  getProducts() {
    return this.products;
  }

}
```

---

# 6️⃣ يعني ايه @Injectable ؟

`@Injectable` معناها:

> Angular يقدر يعمل **Dependency Injection** للكلاس ده.

يعني Angular يقدر **ينشئ object من service ويبعته للcomponents**.

---

# 7️⃣ معنى providedIn: 'root'

```ts
@Injectable({
  providedIn: 'root'
})
```

معناها:

✔ Angular يعمل **Singleton instance**

يعني:

**Instance واحد بس من ال service في كل التطبيق**

كل ال components تستخدم **نفس النسخة**.

---

# 8️⃣ استخدام Service في Component

```ts
constructor(private productService: ProductService) {}
```

Angular هيعمل:

1. create instance من service
    
2. يبعته للcomponent
    

مثال كامل:

```ts
export class HomeComponent {

  products: string[] = [];

  constructor(private productService: ProductService) {}

  ngOnInit() {
    this.products = this.productService.getProducts();
  }

}
```

---

# 9️⃣ مشاركة البيانات بين Components

لو service فيها data:

```ts
products = []
```

وغيرتها في component

كل components هتشوف التغيير.

### مثال

service

```ts
export class CartService {

  cart: string[] = [];

  addItem(item: string) {
    this.cart.push(item);
  }

}
```

component1

```ts
this.cartService.addItem("Laptop");
```

component2

```ts
console.log(this.cartService.cart);
```

هتلاقي **Laptop موجودة**.

لأن الاتنين بيستخدموا **نفس instance**.

---

# 🔟 ليه أي تغيير بيسمع في كل components؟

بسبب:

```
providedIn: 'root'
```

ده بيخلي الservice **Singleton**.

يعني نسخة واحدة مشتركة.

---

# 1️⃣1️⃣ providers في Component

ممكن تعمل service instance **خاص بكل component**.

```ts
@Component({
  selector: 'app-home',
  providers: [ProductService]
})
```

كده Angular هيعمل:

👉 instance جديد لكل component

---

# مثال

component A

```ts
providers:[ProductService]
```

component B

```ts
providers:[ProductService]
```

كل واحد هيبقى عنده **نسخة مختلفة**.

يعني:

التعديل في A ❌ مش هيظهر في B.

---

# 1️⃣2️⃣ امتى نستخدم providers ؟

نادراً.

نستخدمها لو:

- عايز state منفصلة لكل component
    
- testing
    
- isolate data
    

لكن في العادي:

✔ الأفضل

```
providedIn: 'root'
```

---

# 1️⃣3️⃣ الخلاصة

### Service

- كلاس فيه logic او data
    
- reusable
    

---

### @Injectable

تخلي Angular يقدر يعمل **Dependency Injection**

---

### providedIn: root

✔ instance واحدة للتطبيق كله

---

### providers في component

✔ instance منفصلة لكل component

---

# مثال عملي بسيط جداً

### service

```ts
@Injectable({
  providedIn: 'root'
})
export class CounterService {

  count = 0;

  increment(){
    this.count++;
  }

}
```

---

### component 1

```ts
constructor(public counter:CounterService){}

add(){
  this.counter.increment();
}
```

---

### component 2

```ts
constructor(public counter:CounterService){}
```

html

```html
{{counter.count}}
```

لو زودت في component1  
component2 هيشوف التغيير.

---

✅ لو حابب كمان أشرحلك:

- **Service with Observable**
    
- **Service مع API**
    
- **Service communication between components**
    

(و دي أهم حاجة في Angular)  
قولّي وأنا أشرحهم لك برسومات بسيطة.