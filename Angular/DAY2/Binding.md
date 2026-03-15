
## أنواع Binding في Angular
بنستخدمه اصلا لربط ونقل الداتا من ال component <->template

---

### 1. Interpolation `{{ }}` from component to template 

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
from template -> component 

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
Event is a type 

بمعنى أبسط:  
لما المستخدم يعمل **click / submit / input / change** يحصل تنفيذ **كود معين في الـ component**.
من Template → Component (المستخدم بيتفاعل)


<button (click)="showMessage()">Click</button> 
showMessage() => is a function in typescript

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


# 4️⃣ استخدام `$event`

أحياناً محتاج تعرف **تفاصيل الحدث نفسه**.

Angular بيوفر `$event`.

## مثال

<input (input)="getValue($event)">

What is this?

## TypeScript

getValue(event: any) {  
  console.log(event.target.value);  
}

📌 هنا `$event` بيحتوي على معلومات الحدث.
---

### 4. Two Way Binding `[( )]`
Banana in Box  [()]  => two way binding 


# ما هو Two-Way Binding ؟

**Two-Way Binding** يعني:

> البيانات تتحرك في الاتجاهين بين **Component** و **HTML**

يعني:

Component  ↔  View (HTML)

لو القيمة اتغيرت في **TypeScript**  
الـ **HTML يتحدث تلقائياً**

ولو المستخدم غير القيمة في **HTML**  
القيمة في **TypeScript تتغير**

من Component → Template والعكس في نفس الوقت
Component → View
View → Component

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
> # شيء مهم لازم تعمله

لكي يعمل `ngModel` لازم تضيف:

FormsModule

في الموديول.

### app.module.ts

import { FormsModule } from '@angular/forms';  
  
@NgModule({  
  imports: [  
    FormsModule  
  ]  
})

---

### 5. يا **علي** 👨‍💻 خلينا نشرح **Class Binding في Angular** بطريقة بسيطة ومنظمة، لأنه من الحاجات اللي تستخدمها كثير في الـ UI.

---

# ما هو Class Binding ؟

**Class Binding** يعني:

> تضيف أو تشيل **CSS class** من عنصر HTML بناءً على شرط في الـ Component.

يعني Angular يقرر:

```
هل يضيف الكلاس؟
أم لا؟
```

---

# الشكل العام

```html
[class.className]="condition"
```

لو الشرط **true**  
Angular يضيف الكلاس.

لو **false**  
لن يضيفه.

---

# مثال بسيط

### HTML

```html
<button [class.active]="isActive">
  Click
</button>
```

---

### TypeScript

```ts
export class AppComponent {

  isActive = true;

}
```

---

# ماذا سيحدث ؟

لو:

```
isActive = true
```

Angular يجعل:

```html
<button class="active">
```

لو:

```
isActive = false
```

سيصبح:

```html
<button>
```

بدون الكلاس.

---

# مثال عملي (تغيير اللون)

### CSS

```css
.red {
  color: red;
}
```

---

### HTML

```html
<p [class.red]="isRed">
  Hello Angular
</p>

<button (click)="toggleColor()">
Change Color
</button>
```

---

### TypeScript

```ts
export class AppComponent {

  isRed = false;

  toggleColor(){
    this.isRed = !this.isRed;
  }

}
```

---

# ماذا يحدث ؟

عند الضغط على الزر:

```
false → true
```

Angular يضيف:

```
class="red"
```

النص يصبح **أحمر**.

---

# إضافة أكثر من Class

يمكنك استخدام:

```html
[ngClass]
```

مثال:

```html
<p [ngClass]="{'red': isRed, 'bold': isBold}">
  Text
</p>
```

---

### TypeScript

```ts
isRed = true;
isBold = false;
```

---

النتيجة:

```
red → مضاف
bold → غير مضاف
```

---

# مثال شائع جداً (Active Menu)

```html
<li [class.active]="selected === 'home'">
Home
</li>

<li [class.active]="selected === 'about'">
About
</li>
```

---

### TypeScript

```ts
selected = 'home';
```

Angular يضيف:

```
class="active"
```

على **Home**.

---

# الفرق بين class العادي و Class Binding

### كلاس عادي

```html
<p class="red">
```

ثابت.

---

### Class Binding

```html
<p [class.red]="isRed">
```

ديناميكي.

---

# مثال يجمع كل شيء

### HTML

```html
<div [class.active]="isActive">
  Card
</div>

<button (click)="toggle()">
Toggle
</button>
```

---

### TypeScript

```ts
export class AppComponent {

  isActive = false;

  toggle(){
    this.isActive = !this.isActive;
  }

}
```

---

# الخلاصة

Class Binding يستخدم لإضافة أو إزالة CSS classes ديناميكياً.

الشكل:

```html
[class.className]="condition"
```

مثال:

```html
[class.active]="isActive"
```

---





# Style Binding

**Style Binding** يعني:

> نغير **شكل العنصر (CSS)** باستخدام متغير من Angular.

بدل ما نكتب:

```html
<p style="color:red">
Hello
</p>
```

نخلي Angular يتحكم في اللون.

---

# الشكل العام

```html
[style.property]="variable"
```

يعني:

```
غير الستايل باستخدام متغير
```

---

# مثال 1 (تغيير اللون)

### HTML

```html
<p [style.color]="textColor">
Hello Ali
</p>
```

### TypeScript

```ts
export class AppComponent {

  textColor = "red";
  fontsize ="10px";

}
```

---

# ماذا يحدث؟

Angular يجعل الكود هكذا:

```html
<p style="color:red">
Hello Ali
</p>
```

---

# مثال 2 (تغيير اللون بزر)

### HTML

```html
<p [style.color]="color">
Hello
</p>

<button (click)="changeColor()">Change Color</button>
```

---

### TypeScript

```ts
export class AppComponent {

  color = "red";
  changeColor()
  {
  this.color = "red";
  }

}
```

---

# ماذا يحدث؟

في البداية:

```
color = red
```

النص أحمر.

عند الضغط:

```
color = blue
```

النص يصبح أزرق.

---

# مثال 3 (تغيير حجم الخط)

### HTML

```html
<p [style.font-size.px]="size">
Angular
</p>

<button (click)="increase()">Bigger</button>
```

---

### TypeScript

```ts
export class AppComponent {

  size = 16;

  increase(){
    this.size = this.size + 5;
  }

}
```

---

# ماذا يحدث؟

في البداية:

```
font-size:16px
```

بعد الضغط:

```
font-size:21px
```

الخط يكبر.

---

# مثال 4 (تغيير لون الخلفية)

### HTML

```html
<div [style.background]="bg">
Card
</div>

<button (click)="change()">Change</button>
```

---

### TypeScript

```ts
export class AppComponent {

  bg = "yellow";

  change(){
    this.bg = "lightblue";
  }

}
```

---

# الفرق بين CSS العادي و Style Binding

### CSS عادي

```html
<p style="color:red">
```

لون ثابت.

---

### Style Binding

```html
<p [style.color]="color">
```

Angular يغير اللون حسب المتغير.

---

# خلاصة سهلة جداً

Style Binding يعني:

```
Angular يتحكم في CSS
```

الشكل:

```html
[style.property]="value"
```

أمثلة:

```
[style.color]="color"
[style.background]="bg"
[style.font-size.px]="size"
```

---

لو حابب يا **علي** أوريك حاجة مهمة جداً بعد كده 👌  
هشرحلك مثال **واقعي جداً في Angular** يجمع:

```
Event Binding
Style Binding
Class Binding
```

في **Card Component حقيقي** زي اللي بتستخدمه في المشاريع.  
وده اللي بيخليك تفهم Angular عملياً.
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



