تمام 🔥 دلوقتي هنشرح React Router تاني لكن بالمودرن ستايل اللي أنت كاتبه فعلًا باستخدام:

- `createBrowserRouter`
    
- `RouterProvider`
    
- array of objects
    

ودي أفضل طريقة حاليًا في React Router v6.4+

---

# 🧠 الفكرة الأساسية

أنت عندك SPA:

```id="v3iqyz"
Single Page Application
```

يعني React عنده صفحة HTML واحدة فقط.

لكن إحنا عايزين:

- `/`
    
- `/about`
    
- `/contact`
    

بدون refresh.

هنا ييجي دور:

```id="8fb15t"
react-router-dom
```

---

# 📦 أول حاجة تثبيت المكتبة

```bash
npm install react-router-dom
```

---

# 🔥 الكود اللي أنت كاتبه

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'

import { RouterProvider } from 'react-router-dom'
import { createBrowserRouter } from 'react-router-dom'

import About from './pages/About.jsx'
import Home from './pages/Home.jsx'

const router = createBrowserRouter([
  {
    path:'/',
    element:<Home/>
  },

  {
    path:'/about',
    element:<About />
  },

  {
    path:'*',
    element:<div className="p-4">404 Not Found</div>
  }
]);

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
)
```

وده ممتاز جدًا 👌

---

# 🧩 نشرح سطر سطر

---

# 1️⃣ createBrowserRouter

```jsx
const router = createBrowserRouter([])
```

دي المسؤولة عن إنشاء الراوتر.

بتاخد:

```id="4ryc8u"
Array of Route Objects
```

يعني كل object يمثل route.

---

# 2️⃣ أول Route

```jsx
{
  path:'/',
  element:<Home/>
}
```

## 📌 معناه

لما المستخدم يدخل:

```id="t1qk2t"
localhost:5173/
```

اعرض:

```jsx
<Home />
```

---

# 3️⃣ About Route

```jsx
{
  path:'/about',
  element:<About />
}
```

لما المستخدم يدخل:

```id="6lb7ry"
/about
```

React يعرض:

```jsx
<About />
```

---

# 4️⃣ Wildcard Route ⭐

```jsx
{
  path:'*',
  element:<div>404</div>
}
```

## 📌 يعني:

أي route غير معروف.

مثال:

```id="wp8fca"
/test
/hello
/anything
```

كلهم يروحوا:

```id="gj8vph"
404 Not Found
```

---

# ⚠️ مهم جدًا

الـ wildcard:

```id="v8chb4"
*
```

لازم يكون آخر route.

---

# 5️⃣ RouterProvider

```jsx
<RouterProvider router={router} />
```

ده اللي يشغل الراوتر داخل التطبيق.

يعني:

```id="1s8zd2"
يشغل كل ال routes
```

---

# 🎯 لحد هنا عندك:

- Home page
    
- About page
    
- 404 page
    

لكن في مشكلة 😅

---

# ❌ المشكلة الحقيقية

لو عندك:

- Navbar
    
- Footer
    

هتكررهم في كل صفحة.

---

# 🔥 الحل → Layout

---

# 🧱 Layout فكرة مهمة جدًا

بدل ما نكرر:

- Navbar
    
- Footer
    
- Sidebar
    

نعمل:

```id="hq4gje"
Layout Component
```

---

# 📁 Structure

```id="wkswy4"
src
 ├── layouts
 │    └── MainLayout.jsx
 │
 ├── pages
 │    ├── Home.jsx
 │    ├── About.jsx
 │
 ├── main.jsx
```

---

# 🧩 MainLayout.jsx

```jsx
import { Outlet, Link } from "react-router-dom";

export default function MainLayout() {
  return (
    <div>

      <nav className="p-4 bg-black text-white flex gap-4">
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>

      <div className="p-4">
        <Outlet />
      </div>

      <footer className="p-4 bg-gray-200">
        Footer
      </footer>

    </div>
  );
}
```

---

# 🧠 أهم حاجة هنا

## Outlet

```jsx
<Outlet />
```

دي مكان عرض الصفحات.

يعني:

- Home يتعرض هنا
    
- About يتعرض هنا
    

---

# 🔥 دلوقتي نعدل main.jsx

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'

import {
  RouterProvider,
  createBrowserRouter,
} from 'react-router-dom'

import MainLayout from './layouts/MainLayout'

import Home from './pages/Home'
import About from './pages/About'

const router = createBrowserRouter([
  {
    path:'/',
    element:<MainLayout />,

    children:[
      {
        index:true,
        element:<Home />
      },

      {
        path:'about',
        element:<About />
      },

      {
        path:'*',
        element:<div>404 Not Found</div>
      }
    ]
  }
])

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
)
```

---

# 🔥 أهم جزء هنا

## children

```jsx
children:[]
```

يعني:

> Routes داخل Layout

---

# 🧠 ماذا يحدث فعليًا؟

لما تدخل:

```id="2vxb6z"
/about
```

React يعمل:

```jsx
<MainLayout>

   Navbar

   <About />

   Footer

</MainLayout>
```

---

# ⭐ index:true

بدل:

```jsx
path:'/'
```

داخل children نستخدم:

```jsx
index:true
```

ومعناها:

```id="s0smdo"
default child route
```

يعني الصفحة الرئيسية.

---

# 🔥 الشكل النهائي الحقيقي للمشاريع

```id="0hvokq"
src
 ├── layouts
 │    ├── MainLayout.jsx
 │    └── DashboardLayout.jsx
 │
 ├── pages
 │    ├── Home.jsx
 │    ├── About.jsx
 │    ├── Login.jsx
 │    ├── Dashboard.jsx
 │
 ├── main.jsx
```

---

# 🧠 ليه الطريقة دي احترافية؟

✅ scalable  
✅ منظمة  
✅ سهلة التوسعة  
✅ تشبه Angular و Next و Nest architecture  
✅ مناسبة للمشاريع الكبيرة

---

# 🏆 ملخص مهم جدًا

|المفهوم|وظيفته|
|---|---|
|createBrowserRouter|إنشاء الراوتر|
|RouterProvider|تشغيل الراوتر|
|path|عنوان الـ URL|
|element|الـ component|
|children|nested routes|
|Outlet|مكان عرض الصفحات|
|index:true|الصفحة الافتراضية|
|*|wildcard 404|

---

# 🔥 أفضل Practice

✅ اعمل:

```id="g0lfku"
layouts/
pages/
components/
```

✅ استخدم:

```id="yt7csl"
children + Outlet
```

✅ استخدم:

```id="j3h6vq"
Link
```

بدل:

```id="2gnldn"
<a>
```

---

بعد كده المفروض تتعلم:

1. Dynamic Routes
    

```id="t6r4v9"
/product/:id
```

2. Protected Routes
    
3. useNavigate
    
4. useParams
    
5. loaders/actions
    

ودي الحاجات اللي بتنقلك من beginner → advanced 🔥