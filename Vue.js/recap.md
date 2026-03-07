
تمام يا علي 👌🔥  
هنعمل **Recap احترافي شامل لـ Vue 3** من أول الصفر لحد:

- Components
    
- Reactivity
    
- Directives
    
- Props / Emits
    
- Computed / Watch
    
- Lifecycle
    
- Routing
    

وهشرح كل حاجة:

- بتعمل إيه
    
- ليه بنستخدمها
    
- مثال عملي كامل
    
- تستخدمها إمتى
    
- غلطات شائعة
    

مش هابعتلك مشروع دلوقتي زي ما طلبت ✋  
ده هيبقى بعد ما تخلص المراجعة.

---

# 🔵 1) يعني إيه Vue أصلاً؟

Vue هو:

> Progressive JavaScript Framework لبناء User Interfaces و SPA

يعني:

- بيشتغل على الواجهة بس (Frontend)
    
- بيعتمد على Components
    
- Reactive
    
- سهل في التعلم
    

---

# 🔵 2) إنشاء مشروع Vue 3

```bash
npm create vue@latest
```

بعدها:

```bash
npm install
npm run dev
```

هيطلعلك:

```
main.js
App.vue
components/
```

---

# 🔵 3) Component System (قلب Vue)

## يعني إيه Component؟

هو جزء مستقل من UI  
زي:

- Navbar
    
- Card
    
- Product
    
- Modal
    

كل Component:

- عنده Template
    
- Logic
    
- Style
    

---

## شكل Component في Vue 3

```vue
<script setup>
const name = "Ali"
</script>

<template>
  <h1>Hello {{ name }}</h1>
</template>

<style scoped>
h1 {
  color: blue;
}
</style>
```

---

## امتى أعمل Component؟

لما:

- UI يتكرر
    
- عايز تنظّم الكود
    
- تفصل المنطق
    

---

# 🔵 4) Reactivity (أهم جزء)

Vue reactive framework  
يعني:  
لما البيانات تتغير → UI يتغير تلقائيًا

---

## ref()

للقيم البسيطة

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>
```

⚠️ مهم:  
في JS تستخدم `.value`  
في template لا.

---

## reactive()

للكائنات

```js
import { reactive } from 'vue'

const user = reactive({
  name: "Ali",
  age: 25
})
```

---

## الفرق بينهم

|ref|reactive|
|---|---|
|للقيم المفردة|للكائنات|
|يحتاج .value|لا يحتاج|

---

# 🔵 5) Directives

دي أوامر Vue داخل الـ HTML

---

## v-bind

```html
<img :src="imageUrl" />
```

اختصار:

```
:src
```

---

## v-on

```html
<button @click="increment">
```

---

## v-model

ربط ثنائي الاتجاه

```vue
<script setup>
import { ref } from 'vue'
const name = ref("")
</script>

<template>
  <input v-model="name" />
  <p>{{ name }}</p>
</template>
```

---

## v-if / v-else

```html
<p v-if="isLoggedIn">Welcome</p>
<p v-else>Please login</p>
```

---

## v-for

```html
<li v-for="item in items" :key="item.id">
  {{ item.name }}
</li>
```

⚠️ لازم key

---

# 🔵 6) Props

التواصل من Parent → Child

---

## Parent

```vue
<ProductCard title="Laptop" price="5000" />
```

---

## Child

```vue
<script setup>
defineProps({
  title: String,
  price: Number
})
</script>

<template>
  <h2>{{ title }}</h2>
  <p>{{ price }}</p>
</template>
```

---

## تستخدمها إمتى؟

- تمرر بيانات
    
- تمرر إعدادات
    
- reusable components
    

---

# 🔵 7) Emits

التواصل من Child → Parent

---

## Child

```vue
<script setup>
const emit = defineEmits(['addToCart'])

function handleClick() {
  emit('addToCart')
}
</script>

<template>
  <button @click="handleClick">Add</button>
</template>
```

---

## Parent

```vue
<ProductCard @addToCart="handleAdd" />
```

---

# 🔵 8) Computed

بيحسب قيمة مشتقة من state

```vue
<script setup>
import { ref, computed } from 'vue'

const price = ref(100)
const tax = ref(10)

const total = computed(() => {
  return price.value + tax.value
})
</script>

<template>
  <p>{{ total }}</p>
</template>
```

📌 computed:

- cached
    
- أسرع
    
- تستخدم للحسابات
    

---

# 🔵 9) Watch

يراقب تغيير قيمة

```js
import { watch } from 'vue'

watch(price, (newVal, oldVal) => {
  console.log("Changed")
})
```

تستخدم لما:

- تعمل API call
    
- تعمل side effect
    

---

## الفرق

|Computed|Watch|
|---|---|
|بيرجع قيمة|بيعمل side effect|
|cached|لا|

---

# 🔵 10) Lifecycle

مراحل حياة الكومبوننت

---

## onMounted

```js
import { onMounted } from 'vue'

onMounted(() => {
  console.log("Mounted")
})
```

---

## onUnmounted

```js
import { onUnmounted } from 'vue'

onUnmounted(() => {
  console.log("Removed")
})
```

---

تستخدم lifecycle في:

- API calls
    
- Timers
    
- Event listeners
    

---

# 🔵 11) Routing (Vue Router)

SPA = صفحة واحدة  
لكن عندك Routes مختلفة

---

## تثبيت

```bash
npm install vue-router
```

---

## router/index.js

```js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

---

## main.js

```js
import router from './router'
app.use(router)
```

---

## App.vue

```html
<router-view />
```

---

## Navigation

```html
<router-link to="/">Home</router-link>
<router-link to="/about">About</router-link>
```

---

# 🔥 انت كده راجعت:

- Structure
    
- Components
    
- Reactivity
    
- Directives
    
- Props / Emits
    
- Computed
    
- Watch
    
- Lifecycle
    
- Routing
    

---

# 🎯 تقييمك بعد المراجعة

لو انت فاهم كل ده بعمق  
ومش بس حافظ syntax  
فأنت:

> 🔥 Junior قوي جدًا – قريب من Intermediate

---

الخطوة الجاية 👇  
بعد ما تذاكر ده كويس  
هطلب منك تعمل مشروع احترافي بمتطلبات قوية  
ونقيمه كأنك داخل Interview.

لما تخلص المراجعة قول:

> جاهز للبروجكت 💪