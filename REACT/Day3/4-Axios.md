تمام 🔥 خلينا نعمل شرح احترافي لـ **Axios مع React** من الصفر لحد مستوى كويس جداً.

هنتدرج بالشكل ده:

1. يعني اي Axios وليه بنستخدمه
    
2. الفرق بينه وبين fetch
    
3. التثبيت setup
    
4. أول request
    
5. GET / POST / PUT / DELETE
    
6. التعامل مع loading + error
    
7. استخدامه مع useEffect
    
8. Axios instance (احترافي)
    
9. Interceptors (مهم جداً – Auth Tokens)
    
10. Best practices
    

خلينا نبدأ 💪

---

# 1️⃣ يعني اي Axios ؟

Axios هو HTTP Client جاهز بيخليك تعمل API Requests بسهولة من React أو أي JS app.

يعني بدل ما تتعب مع fetch وكتابة boilerplate كتير… Axios بيعمل كل ده بشكل أبسط وأنضف.

تستخدمه عشان:

- تجيب بيانات من API
    
- تبعت بيانات للـ backend
    
- تعمل authentication
    
- ترفع ملفات
    
- تتعامل مع errors بسهولة
    

---

# 2️⃣ Axios vs Fetch (مهم جداً)

|Feature|fetch|axios|
|---|---|---|
|تحويل JSON تلقائي|❌ لازم تعمل res.json()|✅ تلقائي|
|error handling|❌ معقد|✅ سهل|
|interceptors|❌ لا يوجد|✅ يوجد|
|timeout|❌ لا يوجد|✅ يوجد|
|cancel request|❌ صعب|✅ سهل|
|ارسال headers|متوسط|سهل جداً|

### مثال fetch

```js
fetch("https://api.com/users")
  .then(res => res.json())
  .then(data => console.log(data))
```

### نفس الكود بـ axios

```js
axios.get("https://api.com/users")
  .then(res => console.log(res.data))
```

شايف الفرق؟ 😄

---

# 3️⃣ تثبيت Axios

داخل مشروع React:

```bash
npm install axios
```

---

# 4️⃣ أول Request باستخدام Axios

خلينا نعمل API حقيقي نجرب عليه:

```
https://jsonplaceholder.typicode.com/users
```

الـ API ده بيرجع users fake.

### أول مثال GET

```jsx
import axios from "axios";
import { useEffect, useState } from "react";

function Users() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    axios.get("https://jsonplaceholder.typicode.com/users")
      .then(response => {
        console.log(response);
        setUsers(response.data);
      });
  }, []);

  return (
    <div>
      <h1>Users</h1>
      {users.map(user => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  );
}

export default Users;
```

### شكل الـ response من axios

```js
response = {
  data: [],        // البيانات
  status: 200,     // status code
  headers: {},
  config: {}
}
```

أهم حاجة نستخدمها دايماً:

```
response.data
```

---

# 5️⃣ Async / Await (الطريقة الاحترافية)

بدل then نستخدم async await:

```jsx
useEffect(() => {
  const fetchUsers = async () => {
    const response = await axios.get(
      "https://jsonplaceholder.typicode.com/users"
    );
    setUsers(response.data);
  };

  fetchUsers();
}, []);
```

---

# 6️⃣ POST Request (إرسال بيانات)

تخيل عندك صفحة Register.

```jsx
const createUser = async () => {
  const newUser = {
    name: "Ali",
    email: "ali@mail.com"
  };

  const response = await axios.post(
    "https://jsonplaceholder.typicode.com/users",
    newUser
  );

  console.log(response.data);
};
```

---

# 7️⃣ PUT Request (تعديل بيانات)

```jsx
await axios.put(
  "https://jsonplaceholder.typicode.com/users/1",
  { name: "Updated Name" }
);
```

---

# 8️⃣ DELETE Request

```jsx
await axios.delete(
  "https://jsonplaceholder.typicode.com/users/1"
);
```

---

# 9️⃣ Handling Loading & Error (مهم جداً)

ده الشكل الاحترافي لأي request 👇

```jsx
function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const res = await axios.get(
          "https://jsonplaceholder.typicode.com/users"
        );
        setUsers(res.data);
      } catch (err) {
        setError("Something went wrong");
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (loading) return <h2>Loading...</h2>;
  if (error) return <h2>{error}</h2>;

  return users.map(u => <p key={u.id}>{u.name}</p>);
}
```

ده Production Pattern 👍

---

# 🔟 Axios Instance (احترافي جداً)

بدل ما تكتب base URL كل مرة… نعمل instance.

📁 src/api/axios.js

```js
import axios from "axios";

const api = axios.create({
  baseURL: "https://jsonplaceholder.typicode.com",
  timeout: 5000,
});

export default api;
```

الاستخدام:

```js
import api from "../api/axios";

const res = await api.get("/users");
```

لاحظ كتبنا `/users` بس 😎

---

# 1️⃣1️⃣ Axios Interceptors 🔥 (أهم حاجة في الشركات)

دي بتخليك تضيف حاجات لكل request تلقائياً  
زي:

- JWT token
    
- logging
    
- error handling
    

## Request Interceptor (إضافة token)

```js
api.interceptors.request.use(config => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

كده أي request هيبعت التوكن تلقائي 🔥

---

## Response Interceptor (معالجة الأخطاء)

```js
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      alert("Unauthorized! Login again.");
    }
    return Promise.reject(error);
  }
);
```

ده مستوى الشركات 👌

---

# 1️⃣2️⃣ Best Practices 🧠

✔️ استخدم axios instance  
✔️ خلي كل requests في folder اسمه services  
✔️ اعمل error handling  
✔️ استخدم interceptors للتوكن  
✔️ استخدم async/await  
✔️ ماتكتبش URL كامل كل مرة

---

# 🎯 مثال structure احترافي

```
src/
 ├── api/
 │    └── axios.js
 ├── services/
 │    └── userService.js
 ├── pages/
 │    └── Users.jsx
```

## userService.js

```js
import api from "../api/axios";

export const getUsers = () => api.get("/users");
```

## Users.jsx

```js
const res = await getUsers();
setUsers(res.data);
```

ده اسمه Clean Frontend Architecture 💪

---

لو حابب بعد كده نعمل مثال كامل  
Login + Register + JWT باستخدام Axios قولّي ونبنيه سوا 🔥