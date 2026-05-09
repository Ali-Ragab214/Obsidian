
---

# ما هو Zustand ؟

Zustand هو **State Management Library** لReact.

يعني بدل ما تمرر الـ state بين الكومبوننتس باستخدام props (اللي اسمه props drilling)  
يخليك تعمل **Global State** تقدر توصل له من أي مكان في التطبيق بسهولة.

تخيّل عندك:

- User logged in
    
- Theme Dark/Light
    
- Cart
    
- Notifications
    

كل دي بيانات محتاجة تكون متاحة في كل الصفحات.

بدل ما تعمل lifting state كل مرة 😵‍💫  
Zustand يحل المشكلة دي ببساطة جدًا.

---

# المشكلة قبل Zustand (ليه نحتاجه؟)

بدون state manager 👇

```
App
 ├── Navbar (يحتاج user)
 ├── Sidebar (يحتاج user)
 ├── Dashboard (يحتاج user)
```

هتفضل تمرر user كـ props في كل مستوى ❌

```
<App user>
  <Layout user>
    <Navbar user />
```

ده اسمه Props Drilling 😫

---

# الحل مع Zustand

نخزن الـ state في **Store عالمي**  
وأي كومبوننت يقدر يوصله مباشرة 😎

```
Store (global)
   ↑
Navbar
Sidebar
Dashboard
```

---

# فكرة Zustand ببساطة

Zustand = Hook جاهز يعطيك:

- State
    
- Actions لتغيير الـ state
    

---

# شرح الكود اللي كتبته خطوة خطوة

## إنشاء الـ Store

```js
import { create } from "zustand";

export const useStore = create((set) => ({
```

هنا أنشأنا Hook اسمه:

```
useStore()
```

زي useState بالضبط، لكن **global**.

---

## تعريف الـ State

```js
count: 0,
```

ده الـ state العالمي.

أي كومبوننت يقدر يقرأه.

---

## تعريف الـ Actions

الـ Actions = دوال تغيّر الـ state.

### reset

```js
reset: () => set({ count: 0 }),
```

تغيير مباشر للقيمة.

---

### increment

```js
increment: () => set((state) => ({ count: state.count + 1 })),
```

ليه استخدمنا callback هنا؟  
عشان ناخد آخر قيمة من الـ state قبل التحديث.

ده مهم جدًا لو فيه تحديثات متتالية.

---

### باقي الدوال

نفس الفكرة:

```js
decrement: () => set((state) => ({ count: state.count - 1 })),
increaseByTen: () => set((state) => ({ count: state.count + 10 })),
```

---

# استخدام الـ Store داخل React

## قراءة الـ state

```js
const count = useStore((state) => state.count);
```

## استخدام actions

```js
const increment = useStore((state) => state.increment);
```

---

# مثال عملي كامل

```jsx
import { useStore } from "./store/useStore";

const Counter = () => {
  const { count, increment, decrement, reset, increaseByTen } = useStore();

  return (
    <div>
      <h1>{count}</h1>

      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={increaseByTen}>+10</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};

export default Counter;
```

لاحظ ✨  
مفيش props خالص.

---

# أهم استخدامات Zustand في المشاريع الحقيقية

## 1️⃣ Authentication

تخزين المستخدم

```js
user
token
isLoggedIn
logout()
login()
```

## 2️⃣ Shopping Cart

```
cartItems
addToCart()
removeFromCart()
totalPrice
```

## 3️⃣ Theme (Dark/Light)

```
theme
toggleTheme()
```

## 4️⃣ Notifications / Toasts

---

# لماذا Zustand مشهور جدًا؟

|الميزة|Zustand|Redux|
|---|---|---|
|سهل التعلم|✅|❌|
|Boilerplate قليل|✅|❌|
|بدون reducers|✅|❌|
|Performance عالي|✅|✅|
|حجم صغير|1kb 😍|كبير|

Redux محتاج:

- actions
    
- reducers
    
- dispatch
    
- providers
    

Zustand:

```js
create()
خلصنا 😄
```

---

# ميزة قوية جدًا في Zustand (Selectors)

بدل ما تاخد كل الـ store:

❌ غلط:

```js
const store = useStore();
```

✅ صح:

```js
const count = useStore((state) => state.count);
```

ليه؟  
عشان يقلل re-render ويحسّن الأداء 🚀

---

# خلاصة المفهوم 💡

Zustand = Global State بسيط وسريع

يخليك:

- تشارك البيانات بين الصفحات
    
- تنظّم الكود
    
- تتجنب props drilling
    
- تبني apps كبيرة بسهولة
    

---

لو تحب نعمل **Auth Store حقيقي (login/logout + token)** عشان تفهمه عملي أكتر نعمله سوا 👌