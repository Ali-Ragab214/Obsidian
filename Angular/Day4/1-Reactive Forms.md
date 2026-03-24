# 📋 Angular Reactive Forms & @let — الشرح الكامل

> **Tags:** #Angular #ReactiveForms #Frontend #TypeScript  
> **Version:** Angular 14+  
> **آخر تحديث:** 2026

---

## 📌 فهرس المحتوى

1. [[#ما هي Reactive Forms؟]]
2. [[#FormControl]]
3. [[#FormGroup]]

4. [[#Validators]]
5. [[#Use Cases — امتى تستخدم إيه؟]]
6. [[#الفرق بين Reactive و Template-driven]]
7. [[#at-let في Angular 17+]]

---

## 🧠 ما هي Reactive Forms؟

**Reactive Forms** هي طريقة لإنشاء وإدارة الفورمات في Angular بشكل **برمجي (programmatic)** من خلال الـ Component Class، بدل ما تعتمد على الـ HTML Template.

### المبدأ الأساسي:

```
Form = Model في الـ TypeScript
         ⬇
يتربط بالـ HTML عن طريق Directives
```

### لازم تعمل Import:

```typescript
// في app.module.ts أو standalone component
import { ReactiveFormsModule } from '@angular/forms';
```

---

## 🔷 FormControl

### تعريف:

> هو أصغر وحدة في الـ Reactive Forms — بيمثل **حقل واحد** فقط (input, select, checkbox...).

### الصيغة الأساسية:

```typescript
import { FormControl } from '@angular/forms';

// إنشاء
myControl = new FormControl(defaultValue, validators?);

// أمثلة
nameControl    = new FormControl('');           // فاضي
ageControl     = new FormControl(0);            // رقم
isActive       = new FormControl(false);        // checkbox
emailControl   = new FormControl('', [Validators.required, Validators.email]);
```

### في الـ HTML:

```html
<input [formControl]="nameControl" />

<!-- عرض القيمة -->
<p>القيمة: {{ nameControl.value }}</p>

<!-- عرض حالة الـ validation -->
<p *ngIf="nameControl.invalid && nameControl.touched">
  الحقل مطلوب!
</p>
```

### أهم الـ Properties:

|Property|الوصف|
|---|---|
|`.value`|القيمة الحالية|
|`.valid`|true لو صح|
|`.invalid`|true لو غلط|
|`.touched`|المستخدم لمسه|
|`.dirty`|المستخدم غيّر فيه|
|`.pristine`|لسه ما اتغيرش|
|`.errors`|object بالأخطاء|
|`.status`|`VALID / INVALID / PENDING / DISABLED`|

### أهم الـ Methods:

```typescript
ctrl.setValue('جديد');          // بيغير القيمة كلها
ctrl.patchValue('جديد');        // زي setValue للـ FormControl
ctrl.reset();                   // بيرجع للـ default
ctrl.disable();                 // بيعطل الـ input
ctrl.enable();                  // بيفعله تاني
ctrl.valueChanges.subscribe(v => console.log(v)); // بيتابع التغييرات
```

---

## 🔷 FormGroup

### تعريف:

> بيجمع مجموعة من الـ **FormControls** (أو FormGroups تانية) في شكل **object**.

### الصيغة الأساسية:

```typescript
import { FormGroup, FormControl, Validators } from '@angular/forms';

loginForm = new FormGroup({
  email:    new FormControl('', [Validators.required, Validators.email]),
  password: new FormControl('', [Validators.required, Validators.minLength(8)])
});
```

### في الـ HTML:

```html
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
  
  <input formControlName="email" placeholder="البريد الإلكتروني" />
  <span *ngIf="loginForm.get('email')?.invalid && loginForm.get('email')?.touched">
    بريد إلكتروني غير صحيح
  </span>

  <input type="password" formControlName="password" />
  <span *ngIf="loginForm.get('password')?.errors?.['minlength']">
    كلمة المرور أقل من 8 حروف
  </span>

  <button type="submit" [disabled]="loginForm.invalid">دخول</button>

</form>
```

### في الـ Component:

```typescript
onSubmit() {
  if (this.loginForm.valid) {
    console.log(this.loginForm.value);
    // { email: 'ali@test.com', password: '12345678' }
  }
}
```

### أهم الـ Methods:

```typescript
form.get('email');                    // جيب control معين
form.get('email')?.value;             // قيمته
form.setValue({ email: '', pass: '' });   // كل الحقول مطلوبة
form.patchValue({ email: 'test' });       // بس اللي محتاجه
form.reset();                             // إعادة تعيين
form.valueChanges.subscribe(v => ...);    // تابع أي تغيير
```

### Nested FormGroup:

```typescript
profileForm = new FormGroup({
  personalInfo: new FormGroup({
    firstName: new FormControl(''),
    lastName:  new FormControl('')
  }),
  address: new FormGroup({
    city:    new FormControl(''),
    country: new FormControl('')
  })
});
```

```html
<form [formGroup]="profileForm">
  <div formGroupName="personalInfo">
    <input formControlName="firstName" />
    <input formControlName="lastName" />
  </div>
  <div formGroupName="address">
    <input formControlName="city" />
    <input formControlName="country" />
  </div>
</form>
```

---
# Full example 



```typescript
import { Component } from '@angular/core';

import { AbstractControl, FormControl, FormGroup, ValidationErrors, Validators } from '@angular/forms';

import { ReactiveFormsModule } from '@angular/forms';

  

// Custom Validator

function passwordMatch(control: AbstractControl): ValidationErrors | null {

  const password        = control.get('password')?.value;

  const confirmPassword = control.get('confirmpassword')?.value;

  return password === confirmPassword ? null : { passwordMismatch: true };

}

  

@Component({

  selector: 'app-reactive',

  imports: [ReactiveFormsModule],

  templateUrl: './reactive.html',

  styleUrl: './reactive.scss',

})

export class Reactive {

  

  registerForm = new FormGroup({

    name:            new FormControl('', [Validators.required, Validators.minLength(3)]),

    email:           new FormControl('', [Validators.required, Validators.email]),

    password:        new FormControl('', [Validators.required, Validators.minLength(6)]),

    confirmpassword: new FormControl('', [Validators.required]),

    age:             new FormControl('', [Validators.required, Validators.min(18), Validators.max(60)]),

  }, { validators: passwordMatch });  // ← هنا بنحط الـ Custom Validator

  

  onSubmit() {

    if (this.registerForm.valid) {

      console.log(this.registerForm.value);

    } else {

      this.registerForm.markAllAsTouched();

    }

  }

  

  onReset() {

    this.registerForm.reset();

  }

}
```




```html
<form [formGroup]="registerForm" (ngSubmit)="onSubmit()" class="container mt-4" style="max-width: 500px;">

  

  <div class="mb-3">

    <label>Name</label>

    <input type="text" class="form-control" formControlName="name">

    @if(registerForm.get('name')?.errors?.['required'] && registerForm.get('name')?.touched) {

      <div class="text-danger mt-1">Name is required.</div>

    }

    @if(registerForm.get('name')?.errors?.['minlength'] && registerForm.get('name')?.touched) {

      <div class="text-danger mt-1">Name must be at least 3 characters.</div>

    }

  </div>

  

  <div class="mb-3">

    <label>Email</label>

    <input type="email" class="form-control" formControlName="email">

    @if(registerForm.get('email')?.errors?.['required'] && registerForm.get('email')?.touched) {

      <div class="text-danger mt-1">Email is required.</div>

    }

    @if(registerForm.get('email')?.errors?.['email'] && registerForm.get('email')?.touched) {

      <div class="text-danger mt-1">Please enter a valid email.</div>

    }

  </div>

  

  <div class="mb-3">

    <label>Password</label>

    <input type="password" class="form-control" formControlName="password">

    @if(registerForm.get('password')?.errors?.['required'] && registerForm.get('password')?.touched) {

      <div class="text-danger mt-1">Password is required.</div>

    }

    @if(registerForm.get('password')?.errors?.['minlength'] && registerForm.get('password')?.touched) {

      <div class="text-danger mt-1">Password must be at least 6 characters.</div>

    }

  </div>

  

  <div class="mb-3">

    <label>Confirm Password</label>

    <input type="password" class="form-control" formControlName="confirmpassword">

    @if(registerForm.get('confirmpassword')?.errors?.['required'] && registerForm.get('confirmpassword')?.touched) {

      <div class="text-danger mt-1">Confirm Password is required.</div>

    }

    @if(registerForm.errors?.['passwordMismatch'] && registerForm.get('confirmpassword')?.touched) {

      <div class="text-danger mt-1">Passwords do not match.</div>

    }

  </div>

  

  <div class="mb-3">

    <label>Age</label>

    <input type="number" class="form-control" formControlName="age">

    @if(registerForm.get('age')?.errors?.['required'] && registerForm.get('age')?.touched) {

      <div class="text-danger mt-1">Age is required.</div>

    }

    @if(registerForm.get('age')?.errors?.['min'] && registerForm.get('age')?.touched) {

      <div class="text-danger mt-1">Age must be at least 18.</div>

    }

    @if(registerForm.get('age')?.errors?.['max'] && registerForm.get('age')?.touched) {

      <div class="text-danger mt-1">Age must be at most 60.</div>

    }

  </div>

  

  <div class="d-flex gap-2">

    <button type="submit" class="btn btn-primary" [disabled]="registerForm.invalid">

      Register

    </button>

    <button type="button" class="btn btn-secondary" (click)="onReset()">

      Reset

    </button>

  </div>

  

</form>
```





## شرح `get` و `errors` و `touched` و `?`

---

## 🔷 1. `get()`

```typescript
registerForm.get('name')
```

`get()` هي **method** في الـ `FormGroup` بتجيبلك الـ **FormControl** الخاص بالحقل ده.

```typescript
// الـ FormGroup عبارة عن object فيه controls
registerForm = new FormGroup({
  name:  new FormControl(''),   // ← control
  email: new FormControl(''),   // ← control
})

// get() بتجيب الـ control ده بالاسم
registerForm.get('name')   // بيرجع FormControl الخاص بـ name
registerForm.get('email')  // بيرجع FormControl الخاص بـ email
```

---

## 🔷 2. `errors`

```typescript
registerForm.get('name')?.errors
```

`errors` هي **property** في الـ FormControl بترجع **object** فيه كل الأخطاء الموجودة دلوقتي.

```typescript
// لو الحقل فاضي
errors = { required: true }

// لو كتب أقل من المطلوب
errors = { minlength: { requiredLength: 3, actualLength: 1 } }

// لو الحقل صح
errors = null  // مفيش أخطاء
```

**مثال:**

```typescript
nameControl = new FormControl('', [Validators.required, Validators.minLength(3)]);

// المستخدم كتب "A" فقط
nameControl.errors
// { minlength: { requiredLength: 3, actualLength: 1 } }

// المستخدم مكتبش حاجة
nameControl.errors
// { required: true }

// المستخدم كتب "Ali"
nameControl.errors
// null ✅
```

---

## 🔷 3. `touched`

```typescript
registerForm.get('name')?.touched
```

`touched` هي **property** بترجع `true` أو `false`.

```
false  →  المستخدم لسه معملش click على الحقل ده
true   →  المستخدم عمل click على الحقل وخرج منه (blur)
```

**ليه بنستخدمها؟**

```
عشان متعرضش الـ error من أول ما الصفحة تفتح!
```

```typescript
// بدون touched ❌
// الـ error بيظهر من أول ما الصفحة تفتح وده مزعج للمستخدم
@if(registerForm.get('name')?.invalid) { ... }

// مع touched ✅
// الـ error بيظهر بس لما المستخدم يلمس الحقل ويخرج منه
@if(registerForm.get('name')?.invalid && registerForm.get('name')?.touched) { ... }
```


---

## 🔷 4. علامة الاستفهام `?.`

دي اسمها **Optional Chaining** وبتحمي الكود من الـ **crash**.

**المشكلة بدونها:**

```typescript
registerForm.get('name').errors
//                      ↑
// لو get('name') رجع null أو undefined
// Angular هيعمل ERROR ويكسر الصفحة!
// Cannot read properties of null (reading 'errors')
```

**الحل معها:**

```typescript
registerForm.get('name')?.errors
//                      ↑
// لو get('name') رجع null → مش هيكمل وهيرجع undefined بدل ما يكسر
```

**مثال توضيحي:**

```typescript
let user = null;

// ❌ بدون ?.  → CRASH
console.log(user.name);  // ERROR!

// ✅ مع ?.  → آمن
console.log(user?.name);  // undefined (مش بيكسر)
```

---

## 🔷 الصورة الكاملة مع بعض

```typescript
registerForm.get('name')?.errors?.['required']
```

|الجزء|المعنى|
|---|---|
|`registerForm.get('name')`|جيب الـ control الخاص بـ name|
|`?.`|لو الـ control موجود → كمّل، لو null → وقف|
|`.errors`|جيب الـ errors object|
|`?.`|لو الـ errors موجود → كمّل، لو null → وقف|
|`['required']`|جيب خطأ الـ required تحديداً|

```
الناتج:
  true       → في خطأ required  → اعرض الرسالة
  undefined  → مفيش خطأ        → متعرضش حاجة
```

---

## 💡 ملخص سريع

| الحاجة        | هي إيه                | بترجع إيه           |
| ------------- | --------------------- | ------------------- |
| `get('name')` | بتجيب الـ FormControl | الـ control أو null |
| `errors`      | كل الأخطاء الموجودة   | object أو null      |
| `touched`     | المستخدم لمس الحقل؟   | true / false        |
| `?.`          | حماية من الـ crash    | undefined بدل error |















---

## ✅ Validators

### Built-in Validators:

```typescript
import { Validators } from '@angular/forms';

new FormControl('', [
  Validators.required,
  Validators.minLength(3),
  Validators.maxLength(50),
  Validators.email,
  Validators.pattern('^[0-9]+$'),   // أرقام فقط
  Validators.min(18),               // للأرقام
  Validators.max(100),
])
```

### Custom Validator:

```typescript
import { AbstractControl, ValidationErrors } from '@angular/forms';

// Validator Function
function egyptianPhone(control: AbstractControl): ValidationErrors | null {
  const valid = /^01[0-2,5]{1}[0-9]{8}$/.test(control.value);
  return valid ? null : { egyptianPhone: true };
}

// الاستخدام
phoneControl = new FormControl('', [Validators.required, egyptianPhone]);
```

```html
<span *ngIf="phoneControl.errors?.['egyptianPhone']">
  رقم الهاتف غير صحيح!
</span>
```


---

## 🎯 Use Cases — امتى تستخدم إيه؟

### متى تستخدم **FormControl** وحده؟

```
✅ حقل بحث (Search Bar)
✅ Filter ديناميكي
✅ حقل واحد مستقل مش جزء من فورم
```

```typescript
// مثال Search مع debounce
searchControl = new FormControl('');

ngOnInit() {
  this.searchControl.valueChanges.pipe(
    debounceTime(400),
    distinctUntilChanged()
  ).subscribe(value => this.search(value));
}
```

---

### متى تستخدم **FormGroup**؟

```
✅ Login / Register Forms
✅ Profile Edit
✅ Checkout Form
✅ أي فورم فيها مجموعة حقول ثابتة
```

---

### متى تستخدم **FormArray**؟

```
✅ إضافة عناوين متعددة
✅ إضافة مهارات / Skills
✅ Cart Items في Ecommerce
✅ أسئلة في Quiz Builder
✅ Dynamic Invoice Lines
```

---

### جدول المقارنة السريعة:

|الحالة|الأنسب|
|---|---|
|حقل واحد مستقل|`FormControl`|
|فورم بحقول ثابتة|`FormGroup`|
|قائمة ديناميكية|`FormArray`|
|فورم معقدة (nested)|`FormGroup` داخل `FormGroup`|
|قائمة من فورمات|`FormArray` من `FormGroups`|

---

## ⚡ الفرق بين Reactive و Template-driven

|المعيار|Reactive Forms|Template-driven Forms|
|---|---|---|
|التحكم|في الـ TypeScript Class|في الـ HTML Template|
|الـ Scalability|✅ أفضل للفورمات الكبيرة|❌ صعبة في الفورمات الكبيرة|
|Testing|✅ سهل جداً|❌ أصعب|
|Dynamic Forms|✅ ممتاز|❌ صعب|
|Learning Curve|متوسط|سهل|
|الـ Module|`ReactiveFormsModule`|`FormsModule`|

> **💡 القاعدة:** استخدم **Reactive** دايماً في المشاريع الجادة.

---

## 🆕 @let في Angular 17+

### تعريف:

> `@let` هو syntax جديد في **Angular 17** بيخليك تعرّف **متغير محلي** جوه الـ Template مباشرة.

### المشكلة اللي بيحلها:

```html
<!-- قبل @let — مشكلة التكرار -->
<div *ngIf="user$ | async as user">
  <h1>{{ user.firstName }} {{ user.lastName }}</h1>
  <p>مرحباً {{ user.firstName }}!</p>
  <!-- كنا بنكرر `user$ | async` أو نحط div زيادة -->
</div>
```

### الحل مع `@let`:

```html
@let user = user$ | async;

@if (user) {
  <h1>{{ user.firstName }} {{ user.lastName }}</h1>
  <p>مرحباً {{ user.firstName }}!</p>
  <img [src]="user.avatar" />
}
```

---

### الصيغة الأساسية:

```html
@let variableName = expression;
```

### أمثلة عملية:

#### 1. مع Async Pipe:

```html
@let products = products$ | async;

@if (products) {
  <p>عدد المنتجات: {{ products.length }}</p>
  @for (product of products; track product.id) {
    <div>{{ product.name }}</div>
  }
}
```

#### 2. لتبسيط Expressions معقدة:

```html
@let fullName = user.firstName + ' ' + user.lastName;
@let isAdminOrOwner = user.role === 'admin' || user.id === post.authorId;

<h1>{{ fullName }}</h1>
@if (isAdminOrOwner) {
  <button>تعديل</button>
}
```

#### 3. داخل @for:

```html
@for (item of cart; track item.id) {
  @let total = item.price * item.quantity;
  <div>
    <span>{{ item.name }}</span>
    <span>{{ total | currency:'EGP' }}</span>
  </div>
}
```

#### 4. مع Reactive Forms:

```html
@let emailCtrl = loginForm.get('email');

<input [formControl]="emailCtrl" />

@if (emailCtrl?.invalid && emailCtrl?.touched) {
  <span>البريد الإلكتروني غير صحيح</span>
}
```

---

### قواعد مهمة لـ `@let`:

|القاعدة|التفاصيل|
|---|---|
|**Scope**|يعيش فقط في الـ block اللي اتعرف فيه|
|**Read-only**|مش تقدر تعدّل عليه بعد التعريف|
|**لا يدعم Hoisting**|لازم تعرفه قبل ما تستخدمه|
|**يدعم Async Pipe**|✅ أحسن use case ليه|

```html
<!-- ❌ غلط — استخدام قبل التعريف -->
<p>{{ myVar }}</p>
@let myVar = 'hello';

<!-- ✅ صح -->
@let myVar = 'hello';
<p>{{ myVar }}</p>
```

---

### امتى تستخدم `@let`؟

```
✅ مع Async Pipe عشان تتجنب التكرار
✅ لتبسيط expressions طويلة
✅ جوه @for لحساب قيم مؤقتة
✅ لتحسين قراءة الـ Template
❌ مش بديل لـ computed() أو methods في الـ Class
```

---

## 🔗 ملخص سريع

```
FormControl  → حقل واحد
FormGroup    → مجموعة حقول (object)
FormArray    → قائمة ديناميكية (array)
@let         → متغير محلي في الـ Template (Angular 17+)
```

---

## 📚 مصادر للمتابعة

- [Angular Docs — Reactive Forms](https://angular.dev/guide/forms/reactive-forms)
- [Angular Docs — @let syntax](https://angular.dev/guide/templates/let-template-variables)
- [Angular Docs — Validators](https://angular.dev/api/forms/Validators)
```html
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">

  <input type="email" formControlName="email">
  
  <input type="password" formControlName="password">

  <button type="submit">Login</button>

</form>
// formcontrolName = "obj" => we use this with formgroup 
```

---

## 🛠️ 4. submit

```ts
onSubmit() {
  console.log(this.loginForm.value);
}
```

---

# 🔥 Validation (أهم نقطة)

## 🧠 مثال

```ts
email: new FormControl('', [
  Validators.required,
  Validators.email
])
```

---

## 🎨 في HTML

```html
<div *ngIf="loginForm.get('email')?.invalid">
  Email is invalid
</div>
```

---

# 💡 ليه Reactive Forms؟

### ✅ تحكم كامل

تقدر تغير القيم من TS:

```ts
this.loginForm.patchValue({
  email: 'test@gmail.com'
});
```

---

### ✅ Validation قوي

مش مجرد required  
ممكن تعمل custom validation

---

### ✅ مناسب للمشاريع الكبيرة

زي:

- dashboards
    
- admin panels
    
- systems مع forms معقدة
    

---

# 🚀 Use Cases حقيقية (مهم جدًا ليك)

## 🛒 1. E-commerce

- checkout form
    
- address + payment
    

---

## 🛠️ 2. Services App (زي مشروعك)

- تسجيل مستخدم
    
- طلب خدمة
    
- form فيه:
    
    - location
        
    - category
        
    - description
        

---

## 🔐 3. Authentication

- login / register
    
- validation + API call
    

---

## 🧾 4. Admin Dashboard

- add product
    
- edit service
    
- dynamic forms
    

---

# 🔥 Advanced (Level أعلى)

## 🧩 Nested FormGroup

```ts
form = new FormGroup({
  user: new FormGroup({
    name: new FormControl(''),
    email: new FormControl('')
  })
});
```

---

## 🧩 FormArray (لو عندك list)

```ts
skills = new FormArray([
  new FormControl('Angular'),
  new FormControl('Java')
]);
```

---

# ⚠️ أخطاء شائعة

❌ تنسى ReactiveFormsModule  
❌ تستخدم formControlName برا formGroup  
❌ تكتب validation في HTML بس  
❌ متتعاملش مع errors

---

# 🎯 الزتونة (احفظ دي)

- FormControl = input
    
- FormGroup = form كامل
    
- Reactive Forms = التحكم من TS
    
- الأفضل للمشاريع الكبيرة 🔥
    

---

# 💪 نصيحة ليك كـ Backend Dev

ده أهم توبيك يربطك بالـ frontend  
خصوصًا مع:

- APIs
    
- validation قبل ما تبعت request
    

---

لو عايز نطلع Level أعلى 🔥  
نعمل:  
👉 Dynamic Form (زي booking system)  
👉 أو نربطه بـ API حقيقي (login/register)

قولّي وأنا أبنيه معاك خطوة خطوة 💻🔥