

```
Reusable Component
Content Projection
ng-content
Selectors
```

بشكل عملي.

---

# 1️⃣ ما هو Reusable Component ؟

هو **Component نكتبه مرة واحدة ونستخدمه في أكثر من مكان**.

مثال:

```
Button Component
Card Component
Modal Component
Navbar Component
```

بدل ما تكرر الكود.

---

# 2️⃣ كيف نستخدم Component داخل Component آخر ؟

خطوتين أساسيتين:

## 1️⃣ Import Component

في Angular الحديث (Standalone Components):

```ts
imports: [ReusableButtonComponent]
```

---

## 2️⃣ استخدام الـ selector في HTML

مثال:

```html
<app-button></app-button>
```

---

# مثال كامل

## reusable button component

### button.component.ts

```ts
@Component({
 selector: 'app-button',
 templateUrl: './button.component.html'
})
export class ButtonComponent {}
```

---

### button.component.html

```html
<button class="btn btn-primary">
 Click
</button>
```

---

## استخدامه في Component آخر

### home.component.ts

```ts
imports:[ButtonComponent]
```

---

### home.component.html

```html
<app-button></app-button>
```

---

# 3️⃣ هل يجب ألا يكون Selector Self Closing ؟

الأفضل ❗

❌ غير مستحب

```html
<app-button />
```

✔️ الصحيح

```html
<app-button></app-button>
```

لأن Angular قد تحتاج وضع محتوى داخله.

---

# 4️⃣ كيف نجعل Component قابل للتخصيص ؟

نستخدم:

```html
<ng-content>
```

وهذا يسمى:

```
Content Projection
```

---

# مثال Reusable Button قابل للتخصيص

## button.component.html

```html
<button class="btn btn-primary">

 <ng-content></ng-content>

</button>
```

---

## استخدامه

```html
<app-button>
 Add To Cart
</app-button>
```

النتيجة:

```
<button>
 Add To Cart
</button>
```

---

# مثال آخر

```html
<app-button>
 Save
</app-button>

<app-button>
 Delete
</app-button>
```

نفس الكومبوننت لكن محتوى مختلف.

---

# 5️⃣ Restrict content داخل ng-content

يمكنك تحديد نوع العناصر التي تدخل.

مثال:

```html
<ng-content select="span"></ng-content>
```

يعني يقبل فقط:

```html
<span>
```

---

## مثال

### button.component.html

```html
<button>

 <ng-content select="span"></ng-content>

</button>
```

---

### الاستخدام

✔️ يعمل

```html
<app-button>
 <span>Save</span>
</app-button>
```

❌ لن يعمل

```html
<app-button>
 Save
</app-button>
```

---

# 6️⃣ Multiple ng-content

يمكنك وضع أكثر من مكان للمحتوى.

---

### card.component.html

```html
<div class="card">

 <div class="card-header">
  <ng-content select="[header]"></ng-content>
 </div>

 <div class="card-body">
  <ng-content select="[body]"></ng-content>
 </div>

</div>
```

---

### الاستخدام

```html
<app-card>

 <h2 header>Product Name</h2>

 <p body>Product Description</p>

</app-card>
```

---

# النتيجة

```
Card
 ├── header → Product Name
 └── body → Product Description
```

---

# 7️⃣ Reusable Component داخل Reusable Component

نعم يمكن.

مثال:

```
Card Component
   │
   └── Button Component
```

---

## card.component.html

```html
<div class="card">

 <ng-content></ng-content>

 <app-button>
  Buy Now
 </app-button>

</div>
```




---

## الاستخدام

```html
<app-card>

 Product Information

</app-card>
```

---

# Diagram لفهم الفكرة

```
Home Component
      │
      ▼
Card Component
      │
      ▼
Button Component
```

---

# مثال حقيقي بسيط (E-commerce)

```html
<app-card>

 <h3 header>Smart Watch</h3>

 <p body>Best smart watch</p>

 <app-button>
  Add To Cart
 </app-button>

</app-card>
```

---

# الزتونة (مهم تحفظها)

Reusable Component يعتمد على:

1️⃣ selector

```
<app-component></app-component>
```

2️⃣ import component

3️⃣ customization باستخدام

```
ng-content
```

4️⃣ يمكن تحديد المحتوى باستخدام

```
select
```

5️⃣ يمكن وضع **Reusable داخل Reusable**

---

لو حابب يا علي، أعمل لك بعدها **أقوى مثال عملي في Angular**:

نبني **Reusable Card Component احترافي مثل الذي يستخدم في مواقع التجارة الإلكترونية** فيه:

- Image
    
- Title
    
- Price
    
- Button
    
- Custom content
    

وده مثال فعلي يستخدمه المطورون في الشركات.