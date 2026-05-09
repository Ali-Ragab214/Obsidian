# 🔥 مقدمة مهمة جدًا قبل `useEffect`

قبل ما نفهم `useEffect` لازم نفهم فكرة أساسية جدًا في React:

```id="m8twgr"
Pure Functions vs Impure Functions
```

وده أصل وجود `useEffect`.

---

# 🧠 أولاً: يعني إيه Pure Function ؟

الـ Pure Function هي function:

## ✅ 1. نفس الـ input → نفس الـ output دائمًا

## ✅ 2. لا تغيّر أي شيء خارجها

---

# 🧪 مثال Pure Function

```js
function add(a, b) {
  return a + b;
}
```

---

## 📌 ليه دي Pure ؟

لأن:

```js
add(2,3)
```

دائمًا:

```js
5
```

ومبتعملش:

- API call
    
- تغيير DOM
    
- تعديل variable خارجي
    
- timer
    

---

# ❌ يعني إيه Impure Function ؟

هي function تعمل:

- API requests
    
- timers
    
- localStorage
    
- DOM manipulation
    
- console.log
    
- subscriptions
    

دي كلها اسمها:

```id="zjlwmm"
Side Effects
```

---

# 🧪 مثال Impure

```js
let total = 0;

function addToTotal(value) {
  total += value;
}
```

❌ دي impure لأنها غيّرت variable خارجها.

---

# 🧠 React عايزة Components تكون Pure

React بتحب:

```jsx
function Component() {
  return UI
}
```

يعني:

```id="jlwmq8"
UI = f(state)
```

---

# ❌ المشكلة

أحيانًا لازم نعمل:

- fetch data
    
- API calls
    
- timers
    
- localStorage
    
- event listeners
    

ودي كلها:

```id="6x6zqb"
Side Effects
```

---

# 🔥 وهنا ييجي دور useEffect

---

# 🧠 useEffect يعني إيه؟

هو Hook يسمح لك تعمل:

```id="jlwm7v"
Side Effects
```

داخل React Components بطريقة صحيحة.

---

# 🎯 استخداماته الحقيقية

## useEffect يُستخدم في:

✅ Fetching Data  
✅ API Calls  
✅ Timers  
✅ localStorage  
✅ Event Listeners  
✅ Sync with external systems

---

# 📦 الشكل العام لـ useEffect

```jsx
useEffect(() => {

}, [])
```

---

# 🧠 التركيب

## أول جزء

```js
() => {}
```

ده callback function.

الكود اللي جواه بيتنفذ.

---

## ثاني جزء

```js
[]
```

اسمه:

```id="bjlwm9"
Dependency Array
```

وده أهم جزء 😅

---

# 🔥 أول مثال بسيط جدًا

```jsx
import { useEffect } from "react";

export default function App() {

  useEffect(() => {
    console.log("Component Rendered");
  }, []);

  return (
    <h1>Hello</h1>
  );
}
```

---

# 🧠 ماذا يحدث؟

أول ما component يشتغل:

```js
console.log()
```

يتنفذ مرة واحدة.

---

# ⚠️ لماذا مرة واحدة؟

بسبب:

```js
[]
```

---

# 🔥 dependency array مهم جدًا

---

# 1️⃣ لو حطيت [] فارغة

```jsx
useEffect(() => {

}, [])
```

✅ يشتغل مرة واحدة فقط.

مثل:

```id="fjlwmz"
componentDidMount
```

---

# 2️⃣ لو شلت الـ array

```jsx
useEffect(() => {

})
```

❌ يشتغل بعد كل render.

---

# 3️⃣ لو حطيت state

```jsx
useEffect(() => {

}, [count])
```

✅ يشتغل لما:

```id="4qjlwm"
count
```

يتغير.

---

# 🔥 أهم مثال في الحياة العملية

# Fetch Data from API

ودي أهم use case فعلًا.

---

# 🧩 مثال كامل

```jsx
import { useEffect, useState } from "react";

export default function App() {

  const [users, setUsers] = useState([]);

  useEffect(() => {

    fetch("https://jsonplaceholder.typicode.com/users")
      .then((response) => response.json())
      .then((data) => {
        setUsers(data);
      });

  }, []);

  return (
    <div className="p-5">

      <h1 className="text-3xl font-bold mb-4">
        Users
      </h1>

      {
        users.map((user) => (
          <div
            key={user.id}
            className="border p-3 mb-3 rounded"
          >
            {user.name}
          </div>
        ))
      }

    </div>
  );
}
```

---

# 🧠 نشرح خطوة خطوة

---

# 1️⃣ useState

```jsx
const [users, setUsers] = useState([]);
```

بنخزن users.

---

# 2️⃣ useEffect

```jsx
useEffect(() => {

}, [])
```

أول ما الصفحة تفتح:  
اعمل fetch.

---

# 3️⃣ fetch()

```js
fetch("URL")
```

يبعت request للـ API.

---

# 4️⃣ response.json()

تحول البيانات لـ JSON.

---

# 5️⃣ setUsers(data)

نخزن الداتا داخل state.

---

# 6️⃣ React يعمل Re-render

ويعرض users.

---

# 🔥 ليه بنستخدم useEffect مع fetch ؟

لأن:

```id="jlwm9q"
fetch = side effect
```

ومينفعش نعمله مباشرة داخل component body.

---

# ❌ غلط

```jsx
function App() {

  fetch("URL")

  return <div>Hello</div>
}
```

---

# 🧠 ليه غلط؟

لأن كل render:

```id="jlwmcv"
هيعمل fetch تاني
```

ويعمل:

```id="jlwmz3"
Infinite Requests
```

---

# ✅ الصح

```jsx
useEffect(() => {
   fetch()
}, [])
```

---

# 🔥 مثال state dependency

```jsx
import { useEffect, useState } from "react";

export default function App() {

  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Count Changed");
  }, [count]);

  return (
    <div>

      <h1>{count}</h1>

      <button onClick={() => setCount(count + 1)}>
        Increase
      </button>

    </div>
  );
}
```

---

# 🧠 هنا ماذا يحدث؟

كل مرة:

```js
count
```

يتغير:

```js
useEffect
```

يشتغل.

---

# 🔥 Cleanup Function

useEffect يقدر ينظف نفسه.

---

# 🎯 مثال timer

```jsx
useEffect(() => {

  const timer = setInterval(() => {
    console.log("Hello");
  }, 1000);

  return () => {
    clearInterval(timer);
  };

}, []);
```

---

# 🧠 return هنا معناها Cleanup

React يشيل:

- timers
    
- listeners
    
- subscriptions
    

لمنع:

```id="jlwm1l"
Memory Leaks
```

---

# 🏆 ملخص احترافي

|المفهوم|المعنى|
|---|---|
|Pure Function|بدون side effects|
|Impure Function|فيه side effects|
|Side Effect|API / timer / localStorage|
|useEffect|تشغيل side effects|
|[]|مرة واحدة|
|[state]|عند تغير state|
|return cleanup|تنظيف|

---

# 🔥 أهم Mental Model

```id="0xjlwm"
Render UI first

Then useEffect runs
```

---

# 🎯 أهم استخدامات useEffect الحقيقية

✅ Fetch API  
✅ Authentication check  
✅ LocalStorage  
✅ Timers  
✅ WebSocket  
✅ Event listeners

---

# ⚠️ أخطاء beginners المشهورة

## ❌ fetch داخل component body

## ❌ نسيان dependency array

## ❌ تحديث state داخل effect بدون dependencies صح

---

# 🔥 بعد useEffect لازم تتعلم

1️⃣ async/await inside useEffect  
2️⃣ cleanup deeply  
3️⃣ custom hooks  
4️⃣ React Query  
5️⃣ useMemo vs useEffect  
6️⃣ useCallback

ودي النقلة من beginner → professional React Dev 🔥