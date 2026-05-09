حلو جدًا 👌  
Axios Interceptors من أهم الحاجات في React + API لأنهم بيخلّوك تتحكم في كل الريكوستات والريسبونس من مكان واحد.

خلينا نمشي خطوة خطوة بشكل احترافي 🔥

---

# ما هو Axios Interceptor ؟

تخيل إن كل Request بيخرج من التطبيق لازم يعدّي على **بوابة تفتيش** قبل ما يروح للسيرفر،  
وكل Response راجع يعدّي على بوابة قبل ما يدخل للتطبيق.

هذه البوابة اسمها **Interceptor**.

```
React App → Interceptor → API → Interceptor → React App
```

---

# لماذا نستخدم Interceptors ؟

أشهر استخداماتهم في المشاريع الحقيقية:

- إضافة الـ Token تلقائياً لكل request 🔐
    
- معالجة الأخطاء بشكل مركزي 🚨
    
- تحويل شكل البيانات قبل وصولها للـ UI
    
- Logging
    
- Cancel requests
    
- Refresh Token تلقائي
    

---

# نوعين من Interceptors

## 1) Request Interceptor

يشتغل **قبل ما الريكوست يطلع للسيرفر**

## 2) Response Interceptor

يشتغل **بعد ما الريسبونس يرجع من السيرفر**

---

# أول خطوة — إنشاء Axios Instance احترافي

بدل ما نستخدم axios مباشرة، نعمل instance:

### api/axios.js

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: "https://jsonplaceholder.typicode.com",
  timeout: 5000,
});

export default api;
```

ليه؟  
عشان نركب عليه الـ interceptors مرة واحدة.

---

# Request Interceptor (إضافة Token)

## السيناريو الحقيقي

كل API محتاج Authorization Token

بدل ما نكتبه في كل request ❌  
نضيفه مرة واحدة هنا ✅

```javascript
import api from "./axios";

api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    console.log("Request Sent:", config.url);
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

### ماذا يحدث هنا؟

قبل أي request:

- نقرأ التوكن
    
- نضيفه للـ headers تلقائياً
    

يعني أي request بعد كده:

```javascript
api.get("/users");
api.post("/posts");
```

سيحمل التوكن تلقائياً 🔥

---

# Response Interceptor (معالجة الأخطاء)

## أهم سيناريو واقعي:

لو التوكن انتهى → السيرفر يرجع 401  
نريد عمل logout تلقائي.

```javascript
api.interceptors.response.use(
  (response) => {
    console.log("Response Received:", response.status);
    return response;
  },
  (error) => {
    if (error.response?.status === 401) {
      alert("Session expired, please login again");
      localStorage.removeItem("token");
      window.location.href = "/login";
    }

    return Promise.reject(error);
  }
);
```

🔥 الآن أي API يرجع 401 سيعمل Logout تلقائي.

---

# استخدام الـ Instance في React

```javascript
import { useEffect, useState } from "react";
import api from "../api/axios";

const Users = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const fetchUsers = async () => {
      const res = await api.get("/users");
      setUsers(res.data);
    };

    fetchUsers();
  }, []);

  return (
    <div>
      {users.map((u) => (
        <p key={u.id}>{u.name}</p>
      ))}
    </div>
  );
};

export default Users;
```

لاحظ: لا يوجد token ولا error handling هنا 😎  
كل شيء يحدث تلقائياً عبر interceptor.

---

# Axios Cancel Request 🔥🔥🔥

ميزة قوية جدًا خصوصاً مع:

- البحث Search
    
- Autocomplete
    
- تغيير الصفحات بسرعة
    

## المشكلة بدون cancel

لو المستخدم يكتب بسرعة:

```
a → request
ab → request
abc → request
```

كلهم يروحوا للسيرفر 😱

نريد إلغاء القديم وتشغيل الجديد فقط.

---

# طريقة الإلغاء باستخدام AbortController

```javascript
import { useEffect, useState } from "react";
import api from "../api/axios";

const SearchUsers = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    const fetchUsers = async () => {
      try {
        const res = await api.get("/users", {
          signal: controller.signal,
        });
        setUsers(res.data);
      } catch (error) {
        if (error.name === "CanceledError") {
          console.log("Request canceled");
        }
      }
    };

    fetchUsers();

    return () => {
      controller.abort(); // هنا يتم الإلغاء 🔥
    };
  }, []);

  return <div>Users Loaded</div>;
};

export default SearchUsers;
```

---

# لماذا الإلغاء مهم جداً؟

React يعمل re-render كثير  
بدون cancel سيحدث:

- Memory leaks
    
- Warnings
    
- Requests بلا داعي
    
- مشاكل أداء
    

---

# الخلاصة الاحترافية 🎯

Axios Interceptors = Middleware للـ API

|الميزة|الفائدة|
|---|---|
|Request Interceptor|إضافة Token – Logging|
|Response Interceptor|معالجة الأخطاء – Logout|
|Cancel Requests|تحسين الأداء ومنع السبام|
|Axios Instance|كود نظيف ومركزي|

---

لو تحب أعمل لك مثال احترافي حقيقي فيه **Refresh Token flow** (مهم جدًا في الشركات) نكمله مع بعض 💪
