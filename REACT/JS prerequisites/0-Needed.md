تمام 👍 دي **Checklist JavaScript for React** — كل اللي محتاجه تتقنه عشان تستخدم React + Forms + Events براحة.

انسخها زي ما هي وحطها في Obsidian.

---

# 🟨 JavaScript needed for React — Checklist

## 1️⃣ Variables & Data Types

- `let` / `const`
    
- Strings
    
- Numbers
    
- Booleans
    
- Arrays
    
- Objects
    
- Template literals `` `Hello ${name}` ``
    

---

## 2️⃣ Functions

- Function declaration
    
- Arrow functions `()=>{}`
    
- Returning values
    
- Passing functions as arguments (callbacks)
    

---

## 3️⃣ ES6+ Modern Syntax (مهم جداً)

- Destructuring (object & array)
    
- Spread operator `...`
    
- Rest operator `...`
    
- Default parameters
    
- Ternary operator `condition ? a : b`
    
- Short circuit `&&`
    
- Optional chaining `?.`
    
- Nullish coalescing `??`
    

---

## 4️⃣ Arrays Methods (أهم جزء في React 🔥)

- `map()`
    
- `filter()`
    
- `find()`
    
- `some()`
    
- `every()`
    
- `includes()`
    
- `reduce()` (مهم بعدين)
    
- `forEach()` (أحياناً)
    

---

## 5️⃣ Objects

- Access properties `obj.name`
    
- Update object immutably
    
- Spread objects `{...obj, newProp: value}`
    

---

## 6️⃣ DOM Events (أساسي للفورمز)

- `onClick`
    
- `onChange`
    
- `onSubmit`
    
- `onBlur`
    
- `onFocus`
    
- `onKeyDown`
    
- `onKeyUp`
    
- `onMouseEnter`
    
- `onMouseLeave`
    

---

## 7️⃣ Event Object (مهم جداً)

- `event.target`
    
- `event.target.value`
    
- `event.target.checked`
    
- `event.preventDefault()`
    

---

## 8️⃣ Forms Handling (Core React Forms)

- Controlled inputs
    
- Handling multiple inputs
    
- Checkbox handling
    
- Radio buttons handling
    
- Select dropdown handling
    
- Form submit handling
    
- Reset form
    

---

## 9️⃣ Conditions & Rendering Logic

- `if` statements
    
- `&&` rendering
    
- Ternary rendering
    
- Early return
    

---

## 🔟 Immutability (أهم Concept في React)

- عدم تعديل array مباشرة
    
- عدم تعديل object مباشرة
    
- Always create new copy
    

---

## 1️⃣1️⃣ Async JavaScript (لـ API بعدين)

- Promises
    
- `async / await`
    
- `fetch()`
    
- `try / catch`
    

---

## 1️⃣2️⃣ Modules

- `export`
    
- `export default`
    
- `import`
    

---
---
# 3 -ES6 Modern syntax 
# ES6+ Modern Syntax 🚀

---

## 1. Destructuring

### Object Destructuring — بتسحب قيم من object بالاسم

```js
const user = { name: "Ahmed", age: 25, city: "Cairo" };
const {name , age ,city } =user; =>destructuring

// ❌ الطريقة القديمة
const name = user.name;
const age = user.age;

// ✅ Destructuring
const { name, age, city } = user;
console.log(name); // "Ahmed"
console.log(age);  // 25
```

**مع rename:**

```js
const { name: userName, age: userAge } = user;
console.log(userName); // "Ahmed"
```

**في Function parameters:**

```js
const printUser = ({ name, age }) => {
    console.log(`${name} is ${age} years old`);
};

printUser(user); // "Ahmed is 25 years old"
```

---

### Array Destructuring — بتسحب قيم بالترتيب

```js
const colors = ["red", "green", "blue"];

// ❌ الطريقة القديمة
const first = colors[0];
const second = colors[1];

// ✅ Destructuring
const [first, second, third] = colors;
console.log(first);  // "red"
console.log(second); // "green"
```

**تخطي عنصر:**

```js
const [, , third] = colors;
console.log(third); // "blue"
```

**مع useState — ده اللي هتشوفه كتير في React:**

```js
const [count, setCount] = useState(0);
//     ↑ القيمة   ↑ الfunction اللي بتغيرها
```

---

## 2. Spread Operator `...` — بينشر العناصر

### مع Arrays:

```js
const nums = [1, 2, 3];
const moreNums = [...nums, 4, 5, 6];
console.log(moreNums); // [1, 2, 3, 4, 5, 6]
```

**دمج arrayين:**

```js
const a = [1, 2];
const b = [3, 4];
const combined = [...a, ...b];
console.log(combined); // [1, 2, 3, 4]
```

### مع Objects:

```js
const user = { name: "Ahmed", age: 25 };
const updatedUser = { ...user, city: "Cairo" };
console.log(updatedUser); 
// { name: "Ahmed", age: 25, city: "Cairo" }
```

**تعديل قيمة موجودة:**

```js
const updatedUser = { ...user, age: 30 };
// { name: "Ahmed", age: 30 } ← age اتغيرت
```

**في React — إضافة item لـ array في state:**

```js
const [tasks, setTasks] = useState([]);

const addTask = (newTask) => {
    setTasks([...tasks, newTask]); // ✅ مش بنعدل الأصل
};
```

---

## 3. Rest Operator `...` — بيجمع الباقي

### في Functions:

```js
const sum = (...numbers) => {
    return numbers.reduce((total, num) => total + num, 0);
};

console.log(sum(1, 2, 3));       // 6
console.log(sum(1, 2, 3, 4, 5)); // 15
```

### في Destructuring:

```js
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]
```

**مع Objects:**

```js
const { name, ...otherInfo } = { name: "Ahmed", age: 25, city: "Cairo" };
console.log(name);      // "Ahmed"
console.log(otherInfo); // { age: 25, city: "Cairo" }
```

---

## 4. Default Parameters

```js
// ❌ الطريقة القديمة
const greet = (name) => {
    name = name || "Guest";
    console.log(`Hello, ${name}`);
};

// ✅ Default Parameters
const greet = (name = "Guest") => {
    console.log(`Hello, ${name}`);
};

greet("Ahmed"); // "Hello, Ahmed"
greet();        // "Hello, Guest"
```

**مثال أكتر واقعية:**

```js
const createUser = (name, role = "user", active = true) => {
    return { name, role, active };
};

createUser("Ahmed");          // { name: "Ahmed", role: "user", active: true }
createUser("Sara", "admin");  // { name: "Sara", role: "admin", active: true }
```

---

## 5. Ternary Operator `condition ? a : b`

```js
const age = 20;

// ❌ if/else
let status;
if (age >= 18) {
    status = "Adult";
} else {
    status = "Minor";
}

// ✅ Ternary
const status = age >= 18 ? "Adult" : "Minor";
console.log(status); // "Adult"
```

**في React — conditional rendering:**

```jsx
const isLoggedIn = true;

return (
    <div>
        {isLoggedIn ? <p>Welcome back! 👋</p> : <p>Please login</p>}
    </div>
);
```

**مثال أكتر:**

```jsx
const score = 75;

return (
    <p style={{ color: score >= 50 ? "green" : "red" }}>
        {score >= 50 ? "Passed ✅" : "Failed ❌"}
    </p>
);
```

---

## 6. Short Circuit `&&`

> لو الشرط `true` → بيعمل الجزء التاني لو الشرط `false` → بيوقف ومش بيعمل حاجة

```js
const isAdmin = true;
isAdmin && console.log("Show admin panel"); // هيتنفذ

const isAdmin = false;
isAdmin && console.log("Show admin panel"); // مش هيتنفذ
```

**في React:**

```jsx
const [isLoggedIn, setIsLoggedIn] = useState(true);
const [hasNotifications, setHasNotifications] = useState(3);

return (
    <div>
        {isLoggedIn && <p>Welcome! 👋</p>}
        {hasNotifications > 0 && <p>You have {hasNotifications} notifications 🔔</p>}
    </div>
);
```

---

## 7. Optional Chaining `?.`

> بيوصل لقيمة جوه object من غير ما يكسر لو القيمة `undefined` أو `null`

```js
const user = {
    name: "Ahmed",
    address: {
        city: "Cairo"
    }
};

// ❌ بدون optional chaining — هيكسر لو address مش موجود
console.log(user.address.city);      // "Cairo"
console.log(user.phone.number);      // 💥 Error!

// ✅ مع optional chaining
console.log(user.address?.city);     // "Cairo"
console.log(user.phone?.number);     // undefined (مش error)
```

**مع Arrays:**

```js
const users = null;
console.log(users?.[0]?.name); // undefined — مش error
```

**مع Functions:**

```js
const user = {
    getName: () => "Ahmed"
};

console.log(user.getName?.());    // "Ahmed"
console.log(user.getAge?.());     // undefined — مش error
```

---

## 8. Nullish Coalescing `??`

> بيرجع القيمة التانية **بس لو** الأولى `null` أو `undefined` **مختلف عن `||`** اللي بيتجاهل `0` و `""` و `false` كمان

```js
// المشكلة مع ||
const score = 0;
console.log(score || "No score");  // ❌ "No score" — لأن 0 = falsy
console.log(score ?? "No score");  // ✅ 0 — لأن 0 مش null ولا undefined
```

**مقارنة واضحة:**

```js
const value1 = null ?? "Default";      // "Default"
const value2 = undefined ?? "Default"; // "Default"
const value3 = 0 ?? "Default";         // 0 ✅
const value4 = "" ?? "Default";        // "" ✅
const value5 = false ?? "Default";     // false ✅
```

**مثال واقعي:**

```js
const user = { name: "", age: 0 };

console.log(user.name || "Anonymous");  // ❌ "Anonymous" — لأن "" = falsy
console.log(user.name ?? "Anonymous");  // ✅ "" — القيمة الحقيقية

console.log(user.age || "No age");      // ❌ "No age" — لأن 0 = falsy  
console.log(user.age ?? "No age");      // ✅ 0 — القيمة الحقيقية
```

---

## الفرق بين `||` و `??` و `?.` في جملة واحدة:

|Operator|بيتجاهل|
|---|---|
|`\|`|كل القيم الـ falsy: `null, undefined, 0, "", false`|
|`??`|بس `null` و `undefined`|
|`?.`|بيحمي من الـ crash لو القيمة `null` أو `undefined`|