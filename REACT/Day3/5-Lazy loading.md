
---

# React Router Data Mode (Route Modules) 🧠

من React Router v6.4+ بقى الراوتر مش بس Navigation…  
بقى مسؤول عن **Data Fetching + Mutations + Errors + Lazy Loading**

يعني كل Route بقى عبارة عن **Module متكامل** فيه:

|الجزء|وظيفته|
|---|---|
|Component|UI|
|loader|جلب البيانات|
|action|إرسال بيانات (POST / PUT / DELETE)|
|ErrorBoundary|التعامل مع الأخطاء|
|lazy|تحميل الصفحة عند الحاجة فقط|

فبدل ما نستخدم `useEffect + axios` داخل الصفحة ❌  
بقينا نجيب البيانات قبل ما الصفحة تترسم أصلاً ✅

---

# 1️⃣ فكرة Route Module

بدل ما تعمل:

```jsx
<Route path="/dashboard" element={<Dashboard />} />
```

بقيت تعمل Route فيه **data + logic + UI**.

---

# 2️⃣ Lazy Loading في React Router

## المشكلة القديمة

كل صفحات الموقع كانت تتحمل أول ما تفتح الموقع 😵‍💫  
حتى لو المستخدم مش هيدخلها.

## الحل

نستخدم **lazy route**

```jsx
{
  path: "/dashboard",
  lazy: () => import("./pages/Dashboard")
}
```

📌 معنى lazy:

- ملف Dashboard مش هيتحمّل
    
- إلا لما المستخدم يدخل `/dashboard`
    

ده اسمه:

> Route-level code splitting

🚀 يقلل حجم الباندل = موقع أسرع.
## قبل Lazy Loading ❌

أول ما المستخدم يفتح الموقع:

- المتصفح يحمل **كل صفحات المشروع**
- Home + About + Dashboard + Profile + Settings + Admin + … حتى لو مش هيدخلهم

يعني الباندل ممكن يبقى:

```
main.js = 2MB 😱
```

أول تحميل للموقع يبقى بطيء.

---

## بعد Lazy Loading ✅

أول ما المستخدم يفتح الموقع:

- يتحمل بس الكود الأساسي (Home + Router)
- باقي الصفحات **تتحمل عند التنقل لها فقط**

### السيناريو الحقيقي:

المستخدم فتح الموقع → شاف Home  
لم يدخل Dashboard → إذن Dashboard **لم يتحمل أصلاً**

لما يضغط Dashboard:

- الراوتر يعمل download لملف الصفحة فقط
- ثم يعرضها

يعني التحميل بقى **On Demand**

---

# 3️⃣ شكل Route Module المطلوب

React Router لما يستخدم `lazy()`  
بيستنى من الملف إنه يرجع **Exports محددة بالاسم**

|اسم الـ export|وظيفته|
|---|---|
|loader|جلب البيانات|
|Component|الصفحة|
|ErrorBoundary|لو حصل error|
|action|(اختياري) الفورمز|

لو استخدمت اسم Component → الراوتر هيستخدمه تلقائيًا.

لكن لو حابب تستخدم اسم عادي زي `Dashboard`  
تعمل **default export** والراوتر هيشغله عادي 👍

---

# 4️⃣ مثال كامل — Dashboard Route Module

## 📁 pages/Dashboard.jsx

## أولًا: loader (Data Fetching)

```jsx
import axios from "axios";

export async function loader() {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );

  return response.data;
}
```

💡 أهم نقطة:

- loader يشتغل **قبل render**
    
- الصفحة تستلم البيانات جاهزة
    
- مفيش Loading state غالبًا 👌
    

---

## ثانيًا: ErrorBoundary

لو API وقع أو حصل error

```jsx
export function ErrorBoundary() {
  return (
    <p className="p-5 text-red-500">
      Failed to load dashboard data. Please try again later.
    </p>
  );
}
```

🚨 دي بتشتغل تلقائي لو loader فشل.

---

## ثالثًا: Component (UI)

لو استخدمت اسم `Component`:

```jsx
export const Component = () => {
  const users = useLoaderData();
  return <UI />;
};
```

لكن انت عايز تسميه Dashboard 👇

---

# 5️⃣ هل لازم اسم Component ؟ 🤔

❌ لا مش لازم

عندك اختيارين:

### الطريقة 1 (الموصى بها من React Router)

```jsx
export const Component = () => {}
```

### الطريقة 2 (اللي انت عملتها)

```jsx
const Dashboard = () => {}
export default Dashboard;
```

والراوتر هيستخدم الـ default export تلقائيًا 👍

---

# 6️⃣ الكود النهائي المنظم

```jsx
import axios from "axios";
import { useLoaderData } from "react-router-dom";

/* ================================
   1) DATA LOADER
================================ */
export async function loader() {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  return response.data;
}

/* ================================
   2) ERROR BOUNDARY
================================ */
export function ErrorBoundary() {
  return (
    <p className="p-5 text-red-500">
      Failed to load dashboard data.
    </p>
  );
}

/* ================================
   3) PAGE COMPONENT
================================ */
const Dashboard = () => {
  const users = useLoaderData();

  return (
    <>
      <h1 className="text-2xl font-bold mb-4">Dashboard</h1>

      <ul className="list-disc pl-5">
        {users.map(user => (
          <li key={user.id} className="mb-2">
            <p className="font-bold">{user.name}</p>
            <p>Email: {user.email}</p>
            <p>Phone: {user.phone}</p>
          </li>
        ))}
      </ul>
    </>
  );
};

export default Dashboard;
```

---

# 7️⃣ main.jsx Route

```jsx
const router = createBrowserRouter([
  {
   {
path:'/dashboard',
// lazy:()=>import('./pages/Dashboard') Component دا في حاله لو عاملين اسمه
lazy: async () => {
const module = await import('./pages/Dashboard.jsx');
return {
Component: module.default, //لو هعمله بالطريقه المعتاده ,هعمل كدا هنا
loader: module.loader,
ErrorBoundary: module.ErrorBoundary,
};
}}]);
```

بس كدا 🎉

---

# 8️⃣ الفرق الجوهري بين loader و useEffect

|loader|useEffect|
|---|---|
|قبل render|بعد render|
|مفيش Loading غالبًا|لازم Loading state|
|مناسب للصفحات|مناسب للكومبوننت|
|SEO أفضل|SEO أسوأ|

💡 لذلك Data Mode هو الطريقة الاحترافية الجديدة.

---

# 9️⃣ الخلاصة (ملخص سريع)

React Router Route Module =  
**Page = UI + Data + Errors + Mutations**

و Lazy Loading:

- يقلل حجم المشروع
    
- يسرّع الموقع
    
- يحمل الصفحة عند الطلب فقط
    

---

لو حابب الخطوة الجاية نشرح **actions + forms + mutations** (POST/DELETE) 🔥  
ده الجزء اللي بيخلي React Router بديل حقيقي ل React Query.