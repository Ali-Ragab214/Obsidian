بالمختصر يا علي، **أنت مسؤول عن 6 Components فقط + Setup واحد في App.jsx**.

## الفايلات المطلوبة منك

```text
components/
├── Navbar.jsx
├── Footer.jsx
├── DarkModeToggle.jsx
├── SkeletonCard.jsx
├── EmptyState.jsx
└── BackToTopButton.jsx
```

وكمان تعديل:

```text
App.jsx
```

لإضافة:

```jsx
<Toaster />
```

---

# المطلوب في كل فايل

### Navbar.jsx

- Logo
    
- Links
    
- Search Input
    
- Wishlist Counter
    
- Login / Logout
    
- DarkModeToggle
    

```text
Home
Wishlist
Account
```

ويقرأ:

```js
wishlist.length
isLoggedIn
```

من Zustand Store بتاع ياسر.

---

### Footer.jsx

Footer بسيط للموقع.

---

### DarkModeToggle.jsx

زر:

```text
☀️ / 🌙
```

يحفظ الثيم في:

```js
localStorage
```

ويرجعه بعد Refresh.

---

### SkeletonCard.jsx

Loading Placeholder للكارت.

الفريق كله هيستخدمه.

---

### EmptyState.jsx

Component يقبل:

```jsx
title
message
```

ويظهر لما:

- مفيش نتائج Search
    
- Wishlist فاضية
    

---

### BackToTopButton.jsx

يظهر بعد Scroll.

وعند الضغط:

```js
window.scrollTo()
```

---

### App.jsx

أضف:

```jsx
import { Toaster } from "react-hot-toast";

<Toaster />
```

علشان كل التوستات تشتغل في المشروع كله.

---

# اللي لازم تسلمه بدري للفريق

1. Tailwind Setup
    
2. shadcn Setup
    
3. SkeletonCard
    
4. EmptyState
    

دول الفريق كله معتمد عليهم.

---

# لو عايز أرتبهم حسب الأولوية

```text
1- Tailwind + shadcn
2- SkeletonCard
3- EmptyState
4- Navbar
5- DarkModeToggle
6- Footer
7- BackToTopButton
8- Toaster
9- Responsive Design
```

لو خلصت الـ 7 حاجات دول يبقى أنت خلصت 95% من الجزء المطلوب منك.