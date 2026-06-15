تمام 🔥  
دلوقتي دخلنا في واحدة من أهم الحاجات في React Router وهي:

- `useParams`
    
- `useSearchParams` (مش useQueryParams)
    

⚠️ مفيش Hook اسمها:

```id="0r7r7w"
useQueryParams
```

الاسم الصحيح في React Router هو:

```id="ry0l0l"
useSearchParams
```

وده المسؤول عن:

```id="jlwmxa"
Query Parameters
```

---

# 🧠 الأول نفهم الفرق المهم جدًا

## 1️⃣ Route Params

دي جزء من الـ URL نفسه.

مثال:

```id="oq65m6"
/products/5
```

الرقم:

```id="8om83o"
5
```

اسمه:

```id="e4m0d6"
Route Param
```

---

## 2️⃣ Query Params (Search Params)

مثال:

```id="xh9m2f"
/products?category=laptop&price=1000
```

دي اسمها:

```id="vyl9gx"
Query Parameters
```

---

# 🔥 أولاً: useParams

---

# 🧠 الفكرة

نستخدمها لما يكون عندنا Dynamic Route.

مثال حقيقي:

```id="j4h6iu"
/products/15
/users/8
/posts/22
```

كل ID بيتغير.

---

# 📦 مثال واقعي (Products)

---

# 📁 structure

```id="3s61d0"
src
 ├── pages
 │    ├── Products.jsx
 │    └── ProductDetails.jsx
```

---

# 🧩 Route

```jsx
{
  path:'products/:id',
  element:<ProductDetails />
}
```

---

# 🧠 معنى :id

```id="8ibjlwm"
:id
```

يعني:

> variable داخل الرابط

---

# 🧪 URLs

كل دول يشتغلوا:

```id="t2cy8l"
/products/1
/products/7
/products/999
```

---

# 🧩 ProductDetails.jsx

```jsx
import { useParams } from "react-router-dom";

export default function ProductDetails() {

  const params = useParams();

  console.log(params);

  return (
    <div>
      Product ID: {params.id}
    </div>
  );
}
```

---

# 🧠 لما تدخل:

```id="dtp1w2"
/products/55
```

هيبقى:

```js
params = {
  id: "55"
}
```

---

# 🔥 الطريقة الاحترافية

بدل:

```jsx
const params = useParams();
```

استخدم destructuring:

```jsx
const { id } = useParams();
```

---

# الشكل النهائي

```jsx
import { useParams } from "react-router-dom";

export default function ProductDetails() {

  const { id } = useParams();

  return (
    <div>
      Product ID: {id}
    </div>
  );
}
```

---

# 🎯 استخدام حقيقي جدًا

```jsx
useEffect(() => {
   fetch(`https://api.com/products/${id}`)
}, [id])
```

---

# 🔥 دلوقتي useSearchParams

ودي مهمة جدًا للفلاتر والبحث.

---

# 🧠 مثال واقعي

عندك متجر إلكتروني:

```id="t78nhm"
/products?category=laptop
```

أو:

```id="oyxdfe"
/products?category=laptop&price=1000
```

---

# 📦 Route

```jsx
{
  path:'products',
  element:<Products />
}
```

---

# 🧩 Products.jsx

```jsx
import { useSearchParams } from "react-router-dom";

export default function Products() {

  const [searchParams] = useSearchParams();

  console.log(searchParams);

  return (
    <div>

      Category:
      {searchParams.get("category")}

    </div>
  );
}
```



```jsx 
import {useSearchParams} from "react-router-dom"
export default function Products()
{
  const [searchParams,setSearchParams] = useSearchParams();
  return(
  <>
  div
  Category:
  {searchParams.get("category")}
  /div
  
  <button onCLick={() -> setSearchParams= ("laptop")}
  >
  laptop
  </button>
  </>
  );
}
```
---

# 🧪 لو URL كان:

```id="56xyo7"
/products?category=laptop
```

النتيجة:

```id="7y87m1"
laptop
```

---

# 🧠 get()

```jsx
searchParams.get("category")
```

يعني:

> هات قيمة category من الـ URL

---

# 🔥 مثال أكبر

## URL

```id="a7p3p5"
/products?category=laptop&brand=hp
```

## الكود

```jsx
const category = searchParams.get("category");
const brand = searchParams.get("brand");
```

---

# 🎯 النتيجة

```js
category = laptop
brand = hp
```

---

# 🔥 تغيير الـ Query Params

مش بس قراءة، تقدر تعدلهم.

---

# 🧩 مثال

```jsx
const [searchParams, setSearchParams] = useSearchParams();
```

---

# 🧪 Button

```jsx
<button
  onClick={() =>
    setSearchParams({
      category: "laptop"
    })
  }
>
  Laptops
</button>
```

---

# 🧠 لما تضغط الزر

الـ URL يتحول:

```id="b3xvtc"
/products?category=laptop
```

---

# 🔥 مثال واقعي جدًا (Search)

```jsx
import { useSearchParams } from "react-router-dom";

export default function Products() {

  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get("category");

  return (
    <div>

      <button
        onClick={() =>
          setSearchParams({
            category:"laptop"
          })
        }
      >
        Laptop
      </button>

      <button
        onClick={() =>
          setSearchParams({
            category:"phone"
          })
        }
      >
        Phone
      </button>

      <h1>Current Category: {category}</h1>

    </div>
  );
}
```

---

# 🔥 الفرق العبقري بينهم

|useParams|useSearchParams|
|---|---|
|جزء من route|query string|
|`/products/5`|`?category=laptop`|
|dynamic IDs|filters/search|
|مهم للdetails pages|مهم للفلاتر|

---

# 🎯 متى أستخدم كل واحد؟

## useParams

لـ:

- Product Details
    
- User Profile
    
- Blog Post
    

مثال:

```id="7xzdpk"
/users/5
/posts/10
```

---

## useSearchParams

لـ:

- Search
    
- Filters
    
- Sorting
    
- Pagination
    

مثال:

```id="0x9j3j"
/products?category=laptop&page=2
```

---

# 🔥 استخدام الاثنين مع بعض

وده بيحصل كتير جدًا.

```id="ij56v5"
/products/5?tab=reviews
```

---

## الكود

```jsx
const { id } = useParams();

const [searchParams] = useSearchParams();

const tab = searchParams.get("tab");
```

---

# 🧠 النتيجة

```js
id = 5
tab = reviews
```

---

# 🏆 ملخص احترافي

|Hook|الاستخدام|
|---|---|
|useParams|dynamic route values|
|useSearchParams|query string values|
|setSearchParams|تعديل query params|

---

# 🔥 أهم حاجة تحفظها

## Route Params

```id="ibsk6q"
/products/:id
```

---

## Query Params

```id="c7ovw8"
/products?category=laptop
```

---

بعد كده لازم تتعلم:  
1️⃣ `useNavigate`  
2️⃣ Protected Routes  
3️⃣ Nested Dynamic Routes  
4️⃣ loaders/actions  
5️⃣ React Query مع Router

ودي اللي بتخليك Advanced React Dev 🔥