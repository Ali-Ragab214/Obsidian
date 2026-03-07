تمام يا علي 👌  
بما إنك شغال JavaScript قبل كده، خلينا نشرح **الـ Routes في Vue.js** بطريقة منظمة كأنك هتبدأ مشروع من الصفر 💪

---

# يعني إيه Routes أصلاً؟

الـ Route هو:

> ربط بين URL معين و Component معين.

يعني لما المستخدم يدخل:

```
/home
```

يتعرض Component اسمه `Home.vue`  
ولما يدخل:

```
/about
```

يتعرض `About.vue`

---

# فايدة الـ Routes في Vue.js

في التطبيقات الحديثة (SPA = Single Page Application):

- الصفحة مش بتعمل Reload
    
- المحتوى بيتغير حسب الرابط
    
- التجربة أسرع
    

وده بيتم باستخدام مكتبة اسمها:

## 🔹 Vue Router

وهي المسؤولة عن إدارة الـ Routes في Vue.js

---

# 📦 Structure المشروع

لو عملت مشروع Vue 3 عادي، هيبقى شكله كده:

```
src/
 ├── main.js
 ├── App.vue
 ├── router/
 │     └── index.js
 └── views/
       ├── Home.vue
       └── About.vue
```

---

# 1️⃣ تثبيت Vue Router

لو مش متسطب:

```bash
npm install vue-router
```

---

# 2️⃣ إنشاء ملف router/index.js

```js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

---

# 3️⃣ ربط الراوتر في main.js

```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(router)
  .mount('#app')
```

---

# 4️⃣ تعديل App.vue

لازم تحط المكان اللي هيتعرض فيه الـ Components:

```vue
<template>
  <div>
    <nav>
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
    </nav>

    <router-view></router-view>
  </div>
</template>
```

### 🔹 router-link

بديل عن `<a>` عشان ميعملش Reload

### 🔹 router-view

المكان اللي بيتعرض فيه الـ Component حسب الـ Route

---

# 5️⃣ مثال Home.vue

```vue
<template>
  <h1>Welcome to Home Page</h1>
</template>
```

---

# 6️⃣ مثال About.vue

```vue
<template>
  <h1>About Us Page</h1>
</template>
```

---

# لما تشغل المشروع

لو فتحت:

```
http://localhost:5173/
```

هيظهر Home

ولو دخلت:

```
http://localhost:5173/about
```

هيظهر About

بدون Reload 🔥

---

# 🎯 أنواع Routes مهمة

## 1️⃣ Dynamic Route

مثلاً صفحة User:

```js
{
  path: '/user/:id',
  name: 'User',
  component: User
}
```

الدخول:

```
/user/5
```

داخل Component:

```js
import { useRoute } from 'vue-router'

const route = useRoute()
console.log(route.params.id)
```

---

## 2️⃣ Redirect

```js
{
  path: '/home',
  redirect: '/'
}
```

---

## 3️⃣ 404 Page

```js
{
  path: '/:pathMatch(.*)*',
  name: 'NotFound',
  component: NotFound
}
```

---

# ملخص سريع

|العنصر|وظيفته|
|---|---|
|routes[]|تعريف كل الروابط|
|router-view|مكان عرض الصفحات|
|router-link|التنقل بدون Reload|
|createRouter|إنشاء الراوتر|

---

# ليه ده مهم؟

لأن أي Project Frontend حقيقي:

- Dashboard
    
- E-commerce
    
- Admin Panel
    
- Portfolio
    

لازم فيه Routing.

---

لو حابب 👇  
أعملك مثال Project صغير كامل (Navbar + 3 Pages + Dynamic Route + 404) كأننا بنبنيه سوا خطوة خطوة؟ 😎