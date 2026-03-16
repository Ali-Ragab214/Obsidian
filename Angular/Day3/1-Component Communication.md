Parent -------> Child

- **AppComponent = Parent**
    
- **HomeComponent = Child**
    
- **الـ Parent يرسل data للـ Child باستخدام `@Input()`**
    

سنكتب **كل الملفات** ونشرحها كأنك تتعلم أول مرة.

---
#Not best practise
# 1️⃣ الفكرة قبل الكود

العلاقة ستكون هكذا:

```
AppComponent (Parent)
        │
        │  يرسل البيانات
        ▼
HomeComponent (Child)
```

الـ Parent سيعطي الـ Child:

- title
    
- userName
    

---

# 2️⃣ ملف app.component.ts (الـ Parent)

```ts
import { Component } from '@angular/core';
import { HomeComponent } from './home/home';

@Component({
  selector: 'app-root',
  imports: [HomeComponent],
  templateUrl: './app.html',
  styleUrl: './app.css'
})
export class AppComponent {

  title = "Welcome to my Angular App";

  userName = "Ali";

}
```

### ماذا يحدث هنا؟

أنشأنا **2 properties**:

```ts
title = "Welcome to my Angular App";
userName = "Ali";
```

هذه البيانات سنرسلها للـ **HomeComponent**.

---

# 3️⃣ ملف app.html

```html
<h1>App Component (Parent)</h1>

<app-home 
   [title]="title"
   [user]="userName">
</app-home>
```

### ماذا يحدث هنا؟

نستخدم **Property Binding**

```html
[title]="title"
```

المعنى:

```
ارسل قيمة title من AppComponent
إلى HomeComponent
```

---

# 4️⃣ ملف home.component.ts (الـ Child)

```ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-home',
  templateUrl: './home.html',
  styleUrl: './home.css'
})
export class HomeComponent {

  @Input() title!: string;

  @Input() user!: string;

}

  //@Input(required:true , alias:x) title!: string; 
  دول بروبرتيز بديهم لل dec func  عادي 
```

### أهم شيء هنا  Decorator Function 

```ts
@Input()
```

هذا **decorator** يسمح للـ component أن **يستقبل data من parent**.

بدونه Angular سيعطي error.

---

# 5️⃣ ملف home.html

```html
<h2>Home Component (Child)</h2>

<p>{{title}}</p>

<p>Hello {{user}}</p>
```

### ماذا يحدث هنا؟

الـ Child يعرض البيانات التي وصلته من الـ Parent.

---

# 6️⃣ النتيجة على الشاشة

سيظهر:

```
App Component (Parent)

Home Component (Child)

Welcome to my Angular App

Hello Ali
```

---

# 7️⃣ كيف انتقلت البيانات؟

الخطوات التي حدثت:

```
AppComponent
title = "Welcome to my Angular App"

        │
        │ property binding
        ▼

HomeComponent

@Input() title
```

Angular أخذ القيمة من **Parent** ووضعها داخل **Child**.

---

# 8️⃣ قاعدة لازم تحفظها

في Angular:

```
Parent → Child  =  @Input()
Child → Parent  =  @Output()
```

نحن هنا استخدمنا **Input** فقط.

---

# 9️⃣ الرسم الذهني

```
AppComponent
   title = "Welcome"

       │
       │  [title]="title"
       ▼

HomeComponent

@Input() title
```

---

# 🔟 لماذا نستخدم هذه الطريقة؟

لأنها تجعل التطبيق **مقسم ومنظم**.

مثال حقيقي:

```
App
 ├── Navbar
 ├── Home
 ├── ProductList
 └── Footer
```

كل Component يأخذ البيانات من الـ Parent.

---

لو حابب يا **علي**، أعمل لك بعده مباشرة **مثال أقوى قليلًا** فيه:

- `@Input`
    
- `@Output`
    
- زر داخل Child يغير بيانات Parent
    

وده أهم سيناريو يحدث في مشاريع Angular.

---
---
---
----
#  Child --To---> Parent 


الفكرة الأساسية:

```text
Parent → Child  =  @Input()
Child → Parent  =  @Output() + EventEmitter
```

يعني لما **الـ Child عايز يبلغ الـ Parent إن حدث حصل** (مثلاً زرار اتضغط) لازم **يطلق Event** أو ما يسمى **Fire Event**.

---

# 1️⃣ الفكرة العامة

تخيل السيناريو ده:

- **Parent Component** عنده userName
    
- **Child Component** فيه زرار
    
- لما المستخدم يضغط الزرار → الـ Child يبلغ الـ Parent
    

الـ flow سيكون هكذا:

```
Child
   ↓ fire event
Parent
   ↓
run function
```

---

# 2️⃣ Parent Component

## app.component.ts

```ts
import { Component } from '@angular/core';
import { HomeComponent } from './home/home';

@Component({
  selector: 'app-root',
  imports: [HomeComponent],
  templateUrl: './app.html',
})
export class AppComponent {

  message = "No event yet";

  handleEvent(data: string){
    this.message = data;
  }

}
```

هنا عندنا:

```
message
```

سنغيره عندما يرسل الـ Child حدث.

---

## app.html

```html
<h1>Parent Component</h1>

<p>{{message}}</p>

<app-home 
 (notify)="handleEvent($event)">
</app-home>
```

لاحظ هذا السطر المهم:

```html
(notify)="handleEvent($event)"
```

هذا يسمى **Event Binding**.

المعنى:

```
عندما يطلق child event اسمه notify
شغل function اسمها handleEvent
```

---

# 3️⃣ Child Component

## home.component.ts

```ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-home',
  templateUrl: './home.html'
})
export class HomeComponent {

  @Output() notify = new EventEmitter<string>();

  sendData(){
    this.notify.emit("Hello from child component!");
  }

}
```

هنا أهم جزءين:

### 1️⃣ @Output

```ts
@Output() notify
```

هذا يعني أن الـ component **سيصدر event** اسمه notify.

---

### 2️⃣ EventEmitter

```ts
new EventEmitter<string>()
```

هذا object مسؤول عن **إطلاق الحدث**.

---

### 3️⃣ إطلاق الحدث (Fire Event)

```ts
this.notify.emit("Hello from child component!")
```

هذا هو **Fire Event**.

---

# 4️⃣ Child HTML

## home.html

```html
<h2>Child Component</h2>

<button (click)="sendData()">
Send Event To Parent
</button>
```

عند الضغط:

```
click
↓
sendData()
↓
emit()
↓
Parent receives event
```

---

# 5️⃣ ماذا يحدث عند الضغط على الزر؟

Angular يعمل الآتي:

```
User clicks button
        ↓
Child sendData()
        ↓
notify.emit()
        ↓
Parent catches event
        ↓
handleEvent($event)
        ↓
message changes
```

---

# 6️⃣ النتيجة على الشاشة

قبل الضغط:

```
Parent Component

No event yet
```

بعد الضغط:

```
Parent Component

Hello from child component!
```

---

# 7️⃣ الرسم المعماري

```
Parent
  │
  │ (notify)
  ▼
Child
  │
  │ emit()
  ▼
Parent Function
```

---

# 8️⃣ قاعدة Angular المهمة

Angular يعمل **Data Flow باتجاه واحد**:

```
Parent → Child  data
Child → Parent  events
```

ولهذا نستخدم:

```
@Input
@Output
```

---

# 9️⃣ الزتونة (المهم تحفظه)

لإرسال data من **Child → Parent** نستخدم:

```
@Output()
EventEmitter
emit()
```

---

لو حابب يا **علي**، أوريك كمان **السيناريو الحقيقي اللي يستخدم هذا الكونسبت في 90% من تطبيقات Angular**:

مثال  
**ProductCard → Cart Component (زر Add To Cart)**

وده نفس اللي يحدث في مواقع **Ecommerce**.