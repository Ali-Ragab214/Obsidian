## New Control Flow in Angular - الشرح الكامل 
after angular 17.0

---

### 1. `@if` / `@else if` / `@else`

```typescript
export class AppComponent {
  score = 85;
  isLoggedIn = true;
}
```

```html
@if (score >= 90) {
  <p>ممتاز 🏆</p>
} @else if (score >= 75) {
  <p>جيد جداً ✅</p>
} @else if (score >= 60) {
  <p>جيد 👍</p>
} @else {
  <p>راسب ❌</p>
}

@if (isLoggedIn) {
  <p>مرحبا بك</p>
} @else {
  <a href="/login">سجل دخول</a>
}
```

---

### 2. `@for` / `track` / `@empty`

```typescript
export class AppComponent {
  users = [
    { id: 1, name: "Ali",   age: 21 },
    { id: 2, name: "Ahmed", age: 25 },
    { id: 3, name: "Omar",  age: 19 },
  ];

  emptyList: any[] = [];
}
```

```html
@for (user of users; track user.id) {
  <div class="card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.age }} سنة</p>
  </div>
} @empty {
  <p>مفيش يوزرز ❌</p>
}

<!-- مثال على @empty -->
@for (item of emptyList; track item.id) {
  <p>{{ item }}</p>
} @empty {
  <p>القائمة فاضية 😅</p>
}
```

---

### `track` بالتفصيل:

`track` بتقول Angular تتعرف على كل عنصر بإيه عشان لو الـ list اتغيرت متعملش re-render لكل حاجة.

```html
<!-- track بالـ id (الأفضل دايماً) -->
@for (product of products; track product.id) { }

<!-- track بالـ index (لو مفيش id) -->
@for (item of items; track $index) { }
```

#### المتغيرات الجاهزة جوه `@for`:

```html
@for (item of items; track item.id) {
  <p>Index: {{ $index }}</p>       <!-- رقم العنصر -->
  <p>Count: {{ $count }}</p>       <!-- عدد العناصر الكلي -->
  <p>First: {{ $first }}</p>       <!-- هل ده أول عنصر؟ -->
  <p>Last: {{ $last }}</p>         <!-- هل ده آخر عنصر؟ -->
  <p>Even: {{ $even }}</p>         <!-- هل رقمه زوجي؟ -->
  <p>Odd: {{ $odd }}</p>           <!-- هل رقمه فردي؟ -->
}
```

#### مثال عملي بالمتغيرات:

```html
@for (product of products; track product.id) {
  <div [class.bg-gray]="$even" [class.bg-white]="$odd">
    <span>{{ $index + 1 }}</span>
    <span>{{ product.name }}</span>

    @if ($first) {
      <span>🥇 الأول</span>
    }
    @if ($last) {
      <span>آخر عنصر</span>
    }
  </div>
} @empty {
  <p>مفيش منتجات</p>
}
```

---

### 3. `@switch` / `@case` / `@default`

```typescript
export class AppComponent {
  orderStatus = "shipped";
  role = "admin";
}
```

```html
@switch (orderStatus) {
  @case ('pending') {
    <p>⏳ في الانتظار</p>
  }
  @case ('processing') {
    <p>🔄 جاري التجهيز</p>
  }
  @case ('shipped') {
    <p>🚚 في الطريق</p>
  }
  @case ('delivered') {
    <p>✅ تم التوصيل</p>
  }
  @default {
    <p>❌ حالة غير معروفة</p>
  }
}

<!-- مثال تاني على الـ role -->
@switch (role) {
  @case ('admin') {
    <p>لوحة التحكم 🛠️</p>
  }
  @case ('user') {
    <p>الصفحة الشخصية 👤</p>
  }
  @default {
    <p>مش مسجل دخول</p>
  }
}
```

---

### مثال كامل يجمع كل حاجة:

```typescript
export class AppComponent {
  isLoggedIn = true;
  role = "admin";
  products = [
    { id: 1, name: "لاب توب", status: "available",  price: 15000 },
    { id: 2, name: "موبايل",  status: "soldout",    price: 8000  },
    { id: 3, name: "تابلت",   status: "available",  price: 5000  },
  ];
}
```

```html
@if (isLoggedIn) {

  @switch (role) {
    @case ('admin') {
      <h2>لوحة التحكم 🛠️</h2>
    }
    @case ('user') {
      <h2>أهلاً بك 👤</h2>
    }
    @default {
      <h2>زائر</h2>
    }
  }

  @for (product of products; track product.id) {
    <div class="card">
      <h3>{{ $index + 1 }}. {{ product.name }}</h3>
      <p>السعر: {{ product.price }}</p>

      @switch (product.status) {
        @case ('available') {
          <span>متاح ✅</span>
        }
        @case ('soldout') {
          <span>نفذ ❌</span>
        }
        @default {
          <span>غير معروف</span>
        }
      }

      @if ($last) {
        <p>-- آخر منتج --</p>
      }
    </div>
  } @empty {
    <p>مفيش منتجات</p>
  }

} @else {
  <a href="/login">سجل دخول الأول</a>
}
```

---

### ملخص نهائي:

|الـ Keyword|الوظيفة|
|---|---|
|`@if`|شرط|
|`@else if`|شرط تاني|
|`@else`|لو كل الشروط فشلت|
|`@for`|لوب على array|
|`track`|تتبع العناصر للـ performance|
|`@empty`|لو الـ array فاضية|
|`$index`|رقم العنصر|
|`$first`|أول عنصر|
|`$last`|آخر عنصر|
|`$even`|عنصر زوجي|
|`$odd`|عنصر فردي|
|`$count`|عدد العناصر|
|`@switch`|مقارنة قيمة|
|`@case`|حالة معينة|
|`@default`|لو مفيش case مناسب|