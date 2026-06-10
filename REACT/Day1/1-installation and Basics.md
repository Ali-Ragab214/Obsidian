# إنشاء مشروع جديد بـ Vite (أسرع من Create React App)
```
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```


rest 
أيوة، لو اتعلمت الحاجات اللي في الصورة كويس:

- React Components
    
- useState
    
- useEffect
    
- React Router
    
- Axios
    
- Lazy Loading
    
- Interceptors
    
- Zustand
    

فأنت بالفعل تقدر:

✅ تعمل Fetch Data من Spring Boot API  
✅ تعرض البيانات في UI محترم  
✅ تعمل CRUD كامل (Create, Read, Update, Delete)  
✅ تعمل Authentication بـ JWT  
✅ تتعامل مع Pagination و Filtering  
✅ تعمل Dashboard كاملة

---

### مثال عملي

Spring Boot

```java
@GetMapping("/courses")
public List<CourseResponse> getCourses() {
    ...
}
```

React + Axios

```javascript
const [courses, setCourses] = useState([]);

useEffect(() => {
    axios.get("http://localhost:8080/api/courses")
        .then(response => {
            setCourses(response.data);
        });
}, []);
```

عرض البيانات:

```jsx
{courses.map(course => (
    <div key={course.id}>
        <h3>{course.title}</h3>
        <p>{course.description}</p>
    </div>
))}
```

---

## بالنسبة للـ Design

اليوم أغلب الناس لا تكتب CSS من الصفر.

تقدر تستخدم:

- ChatGPT
    
- Claude
    
- Cursor
    
- Lovable
    

وتقول له:

> Create a modern React dashboard using Tailwind CSS

ويطلعلك UI جاهز.

لكن لازم تكون فاهم:

- HTML
    
- CSS
    
- Flexbox
    
- Grid
    

عشان تعرف تعدل على التصميم.

---

# لو هدفك Full Stack Spring Boot + React

بعد اللي عندك ده، أنصحك تضيف:

## 1. Forms

مهم جدًا

مكتبة:

```bash
react-hook-form
```

هتستخدمها في:

- Login
    
- Register
    
- Create Course
    
- Update Course
    

---

## 2. Validation

مثلاً:

```bash
zod
```

أو

```bash
yup
```

---

## 3. JWT Authentication

لازم تعرف:

- Login
    
- Logout
    
- Store Token
    
- Refresh Token
    
- Protected Routes
    

---

## 4. Tailwind CSS

لو هتتعلم حاجة واحدة للـ Styling:

اتعلم

```text
Tailwind CSS
```

لأنها الأكثر طلبًا حاليًا.

---

## 5. TanStack Query (React Query)

بدل ما تعمل:

```javascript
useEffect
axios
loading
error
```

كل مرة.

هتعمل:

```javascript
useQuery(...)
```

وهيتولى:

- Caching
    
- Refetching
    
- Loading
    
- Error Handling
    

---

## 6. Context API

حتى لو هتستخدم Zustand.

لازم تعرف:

```text
Context API
```

عشان تسألك عليها في الانترفيو.

---

## 7. Custom Hooks

مثال:

```javascript
useCourses()
```

بدل تكرار الكود.

---

# لو عايز تبقى جاهز لسوق العمل كـ Full Stack Spring Boot + React

المسار ده كفاية جدًا:

```text
React Fundamentals
    ↓
React Router
    ↓
Axios
    ↓
useEffect
    ↓
Custom Hooks
    ↓
Context API
    ↓
Zustand
    ↓
React Hook Form
    ↓
Zod Validation
    ↓
JWT Authentication
    ↓
Tailwind CSS
    ↓
TanStack Query
```

بعد كده ابني مشروع حقيقي:

```text
SkillHub
```

- Spring Boot Backend
    
- PostgreSQL
    
- Flyway
    
- Redis
    
- JWT
    
- React
    
- Tailwind
    
- Zustand
    
- TanStack Query
    

وقتها هتبقى Full Stack Developer فعليًا، ومؤهل تتقدم على وظائف Spring Boot + React أو تشتغل على SaaS خاص بيك.