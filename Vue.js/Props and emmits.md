جميل 🔥  
خلينا نشرح **Props في Vue 3** شرح كامل من الألف للياء مع أمثلة عملية واضحة 👌

---

# 🎯 يعني إيه Props أصلاً؟

**Props = طريقة تبعت بيها بيانات من Parent Component لـ Child Component**

📌 اتجاه البيانات دايمًا:

```
Parent ➜ Child
```

مش العكس ❌

---

# 🧠 ليه بنستخدم Props؟

- إعادة استخدام الكومبوننت
    
- تخليه Dynamic
    
- تفصل المنطق عن العرض
    
- تبني Architecture صح
    

---

# 🥇 أبسط مثال

## 👨‍👦 Parent.vue

```vue
<script setup>
import Child from './Child.vue'
</script>

<template>
  <Child name="Ahmed" />
</template>
```

---

## 👶 Child.vue

```vue
<script setup>
defineProps({
  name: String
})
</script>

<template>
  <h1>Hello {{ name }}</h1>
</template>



```

📌 هنا:

- Parent بعت prop اسمها `name`
    
- Child استقبلها باستخدام `defineProps`
    
- استخدمها في template
    

---

# 🥈 أنواع Props (Type Checking)

Vue تقدر تتحقق من نوع البيانات:

```vue
defineProps({
  name: String,
  age: Number,
  isAdmin: Boolean,
  skills: Array,
  user: Object
})
```

لو بعت نوع غلط → Vue تحذرك في console.

---

# 🥉 Required Props

```vue
defineProps({
  name: {
    type: String,
    required: true
  }
})
```

لو الـ Parent مبعتش `name`  
هيطلع Warning.

---

# 🏅 Default Value

```vue
defineProps({
  name: {
    type: String,
    default: "Guest"
  }
})
```

لو مبعتش حاجة → القيمة تبقى "Guest".

---

# 🥇 Props مع Dynamic Binding

مش لازم تبعت String ثابتة.

## Parent:

```vue
<script setup>
import Child from './Child.vue'
import { ref } from 'vue'

const username = ref("Mohamed")
</script>

<template>
  <Child :name="username" />
</template>
```

📌 النقطتين `:` معناهم binding مش string ثابت.

---

# 🥈 تبعت Object كامل

## Parent

```vue
<script setup>
import Child from './Child.vue'

const user = {
  name: "Ali",
  age: 25
}
</script>

<template>
  <Child :user="user" />
</template>
```

---

## Child

```vue
<script setup>
defineProps({
  user: Object
})
</script>

<template>
  <h2>{{ user.name }}</h2>
  <p>{{ user.age }}</p>
</template>
```

---

# ⚠️ مهم جداً — Props Readonly

Props في Vue:

> Read Only ❌

مينفعش تعدلها في الـ Child

❌ غلط:

```vue
props.name = "Omar"
```

لو محتاج تعدلها:

- اعمل نسخة local
    

✔️ صح:

```vue
const props = defineProps(['name'])
const localName = ref(props.name)
```

---

# 🥇 Custom Validation

```vue
defineProps({
  age: {
    type: Number,
    validator(value) {
      return value >= 18
    }
  }
})
```

لو بعت رقم أقل من 18 → warning.

---

# 🥈 Props مع v-for (مهم جداً)

## Parent

```vue
<script setup>
import ProductCard from './ProductCard.vue'

const products = [
  { id: 1, name: "Phone" },
  { id: 2, name: "Laptop" }
]
</script>

<template>
  <ProductCard 
    v-for="product in products"
    :key="product.id"
    :product="product"
  />
</template>
```

---

## Child (ProductCard)

```vue
<script setup>
defineProps({
  product: Object
})
</script>

<template>
  <h3>{{ product.name }}</h3>
</template>
```

دي أهم استخدام عملي للـ props في الحياة الواقعية.

---

# 🧩 Multiple Props

```vue
<Child 
  name="Omar"
  :age="30"
  :isAdmin="true"
/>
```

---

# 📌 الفرق بين String ثابت و Binding

❌ غلط:

```vue
<Child name="username" />
```

دي string اسمها "username"

✔️ صح:

```vue
<Child :name="username" />
```

دي قيمة المتغير نفسه

---

# 🧠 ملخص مهم جداً

|Feature|فايدتها|
|---|---|
|type|يتحقق من النوع|
|required|لازم تتبعت|
|default|قيمة افتراضية|
|validator|validation مخصص|
|:|binding ديناميك|

---

# 📊 بعد ما تفهم Props كويس

إنت كده غطيت تقريباً:

🟢 70% من Vue الأساسي

لأن التواصل بين الكومبوننتس هو أهم حاجة.

---

# 🎯 التمرين اللي يثبت المعلومة

اعمل:

- Component اسمه UserCard
    
- ياخد props:
    
    - name (required)
        
    - age
        
    - isOnline (Boolean)
        
- لو isOnline true يظهر دائرة خضراء
    
- لو false يظهر رمادي
    

لو عرفت تعمله لوحدك  
يبقى انت فهمت Props صح 👌🔥

---

لو حابب بعد كده أشرحلك **Emits بنفس التفصيل** (وإزاي تخلي Child يرجع بيانات للـ Parent) قولّي ونكمل 🔥





New exp

سؤال ممتاز 👏🔥  
خليني أظبطلك المفهوم عشان متتلخبطش.

## ✅ الشكل الصح للـ Data Flow في Vue

في Vue الاتجاه الأساسي للبيانات هو:

```
Parent  ➜  Child   (Props)
Child   ➜  Parent  (Emits - Events)
```

---

# 🧠 هل الـ Child بيبعت event والـ Parent يرد بداتا؟

أيوه… **لكن مش مباشرة**.

اللي بيحصل فعليًا:

1️⃣ الـ Parent يبعت بيانات للـ Child عن طريق **props**  
2️⃣ الـ Child لما يحصل interaction (مثلاً زرار اتضغط)  
يعمل **emit event**  
3️⃣ الـ Parent يسمع للـ event ده  
4️⃣ الـ Parent يغير البيانات عنده  
5️⃣ البيانات الجديدة تنزل تاني للـ Child كـ props

يعني كأنها دورة 🔁

---

# 🎯 مثال عملي يوضح الصورة

## 👨 Parent.vue

```vue
<script setup>
import { ref } from 'vue'
import Counter from './Counter.vue'

const count = ref(0)

function increase() {
  count.value++
}
</script>

<template>
  <Counter :count="count" @increment="increase" />
</template>
```

---

## 👶 Counter.vue (Child)

```vue
<script setup>
const props = defineProps({
  count: Number
}) ## 🧠 ليه بعمل كده؟

عشان تقول لـ Vue:

> # 🎯 طب يعني إيه السطر ده؟

`const props = defineProps({   count: Number })`

معناه 3 حاجات:

1️⃣ أنا مستني prop اسمها `count`  
2️⃣ نوعها المفروض يبقى Number  
3️⃣ خزنها في متغير اسمه `props`"الكومبوننت ده هيستقبل حاجة من بره اسمها `count`"
# 🔍 طب بتجي منين `count` أصلاً؟

من الـ Parent:
<Counter :count="5" />

# 🔥 الخلاصة في جملة واحدة

`defineProps` = بتعلن إن الكومبوننت ده بيستقبل بيانات من الـ Parent.

> 
> 
> 
> 
> 
> 

const emit = defineEmits(['increment'])

function handleClick() {
  emit('increment')
}
</script>

<template>
  <h2>{{ count }}</h2>
  <button @click="handleClick">Increase</button>
</template>
```

---

# 🔍 نفهم اللي حصل خطوة خطوة

1️⃣ Parent عنده state اسمها `count`  
2️⃣ بعتها للـ Child كـ prop  
3️⃣ Child عرضها  
4️⃣ لما المستخدم ضغط زرار  
5️⃣ Child عمل emit لـ event اسمه `increment`  
6️⃣ Parent كان سامع للـ event ده  
7️⃣ Parent شغل function `increase()`  
8️⃣ عدّل قيمة count  
9️⃣ القيمة الجديدة نزلت تاني للـ Child

💥 كده حصل update كامل.

---

# ⚠️ مهم جدًا

الـ Child:

❌ ميعدلش props  
✔️ يبلغ الـ Parent إنه عايز يحصل تغيير

الـ Parent هو اللي يملك الـ state دايمًا.

ده اسمه:

> One Way Data Flow

---

# 🎯 تخيلها كده

الـ Parent = المدير  
الـ Child = الموظف

الموظف ميغيرش القرار بنفسه ❌  
يقول للمدير: "عايز أعمل كذا"  
المدير يقرر ويغير  
وبعدين يبلغ الموظف بالنتيجة

---

# 🧠 ليه Vue مصممة كده؟

عشان:

- تمنع الفوضى
    
- تخلي إدارة البيانات واضحة
    
- تسهل Debug
    
- تخلي المشروع scalable
    

---

# 📊 كده انت فهمت إيه؟

لو فهمت Props + Emits بالشكل ده  
انت كده غطيت 75% من Vue communication system 🔥

---

لو تحب أشرحلك الفرق بين:

- emit العادي
    
- emit مع payload
    
- v-model custom
    
- two way binding في Vue 3
    

نكمل مستوى أعلى شوية؟ 😎



تمام 👌 خلينا نفهم **defineProps** بطريقة بسيطة جدًا ومن غير تعقيد.

---

# 🎯 يعني إيه defineProps أصلاً؟

لما الـ **Parent** يبعت بيانات للـ **Child**:

```vue
<Child name="Ahmed" />
```

الـ Child محتاج يقول لـ Vue:

> "أنا متوقع استقبل prop اسمها name"

هنا ييجي دور **defineProps**

---

# 🧠 تعريف بسيط جدًا

`defineProps()`  
= دالة بتعرف Vue إن الكومبوننت ده بيستقبل Props.

هي مش بتجيب البيانات…  
هي بس بتقول لـ Vue:

> "في Props داخلة عليا بالمواصفات دي"

---

# 🥇 أبسط شكل ليها

```vue
<script setup>
defineProps(['name'])
</script>
```

ده معناه:

- فيه prop اسمها name
    
- مش مهتم بنوعها
    

---

# 🥈 شكل أقوى (مع type)

```vue
<script setup>
defineProps({
  name: String,
  age: Number
})
</script>
```

كده Vue هيحذر لو النوع غلط.

---

# 🥉 لو عايز تستخدم القيمة جوه script

مهم جدًا 👇

```vue
<script setup>
const props = defineProps({
  name: String
})

console.log(props.name)
</script>
```

هنا رجعنا قيمة props في متغير اسمه `props`.

---

# 🧩 طب هي بتيجي منين أصلاً؟

لو عندك Parent كده:

```vue
<Child name="Ali" />
```

Vue تلقائيًا تبعت:

```js
{ name: "Ali" }
```

للكومبوننت.

و `defineProps()` تستقبلها.

---

# 🎯 تخيلها كأنها

```js
function Child(props) {
   console.log(props.name)
}
```

بس Vue بتعمل ده وراك 👀

---

# ⚠️ حاجة مهمة جدًا

`defineProps`:

- بتستخدم بس داخل `<script setup>`
    
- دي macro (مش function عادية)
    
- Vue بتحولها ورا الكواليس
    

---

# 🥶 طب لو معملتش defineProps؟

لو كتبت في template:

```vue
{{ name }}
```

من غير defineProps

هيطلع error  
لأن Vue مش عارفة إن فيه prop اسمها name.

---

# 🔥 مثال كامل واضح جدًا

## Parent

```vue
<script setup>
import Child from './Child.vue'
</script>

<template>
  <Child name="Omar" />
</template>
```

---

## Child

```vue
<script setup>
defineProps({
  name: String
})
</script>

<template>
  <h1>Hello {{ name }}</h1>
</template>
```

---

# 📌 الفرق بين defineProps و props في Vue 2

في Vue 2 كنا نكتب:

```js
export default {
  props: ['name']
}
```

في Vue 3 + script setup:

```js
defineProps(['name'])
```

أبسط وأنضف.

---

# 🧠 خلاصة سريعة جدًا

|defineProps|معناها|
|---|---|
|تعرف props|✔|
|تستقبل بيانات من parent|✔|
|تنشئ state جديد|❌|
|تعدل props|❌|

---

# 🎯 لو لسه حاسس إنها غامضة

قولي بس:

هل المشكلة إنك مش فاهم:

- هي بتشتغل إزاي داخليًا؟
    
- ولا إمتى أستخدمها؟
    
- ولا الفرق بينها وبين ref؟
    

وهنفكها واحدة واحدة 🔥



=>
سؤال ممتاز 👌🔥  
خليني أشرحها لك بطريقة واقعية جدًا وبسيطة.

---

# 🎯 يعني إيه Parent و Child؟

ببساطة:

- **Parent** = كومبوننت كبير بيحتوي جواه كومبوننتات تانية
    
- **Child** = كومبوننت جوه كومبوننت تاني
    

زي شجرة 🌳  
فيه عنصر رئيسي، وتحته عناصر فرعية.

---

# 🏠 مثال واقعي جدًا (موقع متجر)

تخيل عندك صفحة متجر.

## 🧩 App.vue

ده يعتبر **Parent كبير**

جواه:

- Navbar
    
- ProductsList
    
- Footer
    

---

## 👇 ProductsList.vue

ده Child بالنسبة لـ App  
لكن هو نفسه Parent لكومبوننت تاني 👇

---

## 🛒 ProductCard.vue

ده Child بالنسبة لـ ProductsList

---

# شكل العلاقة:

```
App.vue
   ├── Navbar.vue
   ├── ProductsList.vue
          ├── ProductCard.vue
          ├── ProductCard.vue
          ├── ProductCard.vue
   └── Footer.vue
```

كل ProductCard ده Child  
و ProductsList هو Parent ليهم.

---

# 🎯 مثال كود بسيط جدًا

## 👨 Parent

```vue
<!-- ProductsList.vue -->
<script setup>
import ProductCard from './ProductCard.vue'

const product = {
  name: "Phone",
  price: 1000
}
</script>

<template>
  <ProductCard :product="product" />
</template>
```

---

## 👶 Child

```vue
<!-- ProductCard.vue -->
<script setup>
defineProps({
  product: Object
})
</script>

<template>
  <h2>{{ product.name }}</h2>
  <p>{{ product.price }}</p>
</template>
```

---

# 🧠 مثال أبسط كمان

تخيل:

```vue
<App>
   <UserCard />
</App>
```

App = Parent  
UserCard = Child

بس عشان UserCard مكتوبة جوه App.

---

# 🧩 مثال من الحياة الواقعية

تخيل شركة:

- المدير = Parent
    
- الموظفين = Children
    

المدير:

- يديهم بيانات
    
- يتحكم فيهم
    

الموظف:

- ينفذ
    
- يبلغ المدير لما يحصل حدث
    

---

# 🔥 أهم نقطة تفهمها

Parent و Child مش حاجة خاصة بـ Vue  
دي مجرد علاقة "احتواء":

> كومبوننت جواه كومبوننت

---

# 📌 قاعدة سريعة تحفظها

أي كومبوننت مكتوبة جوه template بتاع كومبوننت تاني = تبقى Child ليه.

---

لو تحب أعملك مثال عملي صغير فيه:

- Parent
    
- 2 Children
    
- وواحد منهم يبعت event
    

عشان تشوف الصورة كاملة 👌

# يعني إيه `:key` ؟

لما تكتب:

`<div v-for="product in products" :key="product.id">`

أنت بتقول لـ Vue:

> كل عنصر هنا ليه هوية مميزة (ID)