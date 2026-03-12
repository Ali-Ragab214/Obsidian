
## أنواع Binding في Angular

---

### 1. Interpolation `{{ }}`

من Component → Template كنص

```typescript
export class AppComponent {
  name = "Ali";
  price = 100;
}
```

```html
<h1>{{ name }}</h1>
<p>السعر: {{ price * 1.14 }}</p>
<p>{{ 2 + 2 }}</p>
```

---

### 2. Property Binding `[ ]`

من Component → خصائص HTML

```typescript
export class AppComponent {
  imageUrl = "photo.jpg";
  isDisabled = true;
  inputType = "password";
}
```

```html
<img [src]="imageUrl"/>
<button [disabled]="isDisabled">Click</button>
<input [type]="inputType"/>
```

---

### 3. Event Binding `( )`

من Template → Component (المستخدم بيتفاعل)

```typescript
export class AppComponent {
  count = 0;

  increment() {
    this.count++;
  }

  onInput(event: Event) {
    console.log((event.target as HTMLInputElement).value);
  }
}
```

```html
<button (click)="increment()">+</button>
<input (input)="onInput($event)"/>
<input (keyup.enter)="onEnter()"/>
```

---

### 4. Two Way Binding `[( )]`

من Component → Template والعكس في نفس الوقت

```typescript
import { FormsModule } from '@angular/forms';

export class AppComponent {
  username = "";
}
```

```html
<input [(ngModel)]="username"/>
<p>مرحبا: {{ username }}</p>
```

> بيتحدث الـ component والـ template مع بعض في نفس الوقت ✅

---

### 5. Class Binding

إضافة أو إزالة class بشرط

```typescript
export class AppComponent {
  isActive = true;
  hasError = false;
}
```

```html
<div [class.active]="isActive">أنا active</div>
<div [class.error]="hasError">أنا error</div>

<!-- أو أكتر من class -->
<div [ngClass]="{'active': isActive, 'error': hasError}">
```

---

### 6. Style Binding

تغيير style مباشرة

```typescript
export class AppComponent {
  color = "red";
  fontSize = 20;
}
```

```html
<p [style.color]="color">نص أحمر</p>
<p [style.font-size.px]="fontSize">نص كبير</p>

<!-- أو أكتر من style -->
<p [ngStyle]="{'color': color, 'font-size': fontSize + 'px'}">
```

---

### ملخص سريع:

|النوع|الشكل|الاتجاه|
|---|---|---|
|Interpolation|`{{ value }}`|Component → Template|
|Property Binding|`[property]="value"`|Component → Template|
|Event Binding|`(event)="method()"`|Template → Component|
|Two Way Binding|`[(ngModel)]="value"`|الاتجاهين|
|Class Binding|`[class.x]="bool"`|Component → Template|
|Style Binding|`[style.x]="value"`|Component → Template|



