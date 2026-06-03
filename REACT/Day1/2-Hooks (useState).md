
---

# 🧠 React Basics — Notes

---

# 1️⃣ What are Hooks in React?

## ما هي Hooks؟

الـ Hooks هي **Functions جاهزة من React** بتخلينا نستخدم:

- State
    
- Lifecycle
    
- Logic reuse
    

داخل **Functional Components** بدون الحاجة لـ Class Components.

قبل Hooks كان لازم نستخدم class عشان نعمل state 😵‍💫  
دلوقتي نقدر نعمل كل حاجة باستخدام functions فقط ✨

---

## لماذا Hooks موجودة؟

- تبسيط الكود
    
- إعادة استخدام الـ Logic بسهولة
    
- تقليل تعقيد الـ Class Components
    

---

## 🔴 Rules of Hooks (مهم جداً)

1️⃣ استخدم Hooks فقط داخل React Components  
2️⃣ استخدم Hooks في أعلى الـ Component (Top Level)  
3️⃣ اسم أي Hook يبدأ بـ **use**

❌ مثال غلط:

```js
if(condition){
  useState()
}
```

---

## أشهر Hooks

|Hook|وظيفته|
|---|---|
|useState|تخزين البيانات|
|useEffect|التعامل مع API و Side Effects|
|useContext|مشاركة البيانات بين Components|
|useRef|الوصول للـ DOM|
|useMemo|تحسين الأداء|

---

# 2️⃣ useState — from Easy to Hard

---

## مثال 1️⃣ Counter بسيط

```jsx
import { useState } from "react"

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <h2>{count}</h2>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  )
}
```

---

## مثال 2️⃣ Text Input (Controlled Input)

```jsx
import { useState } from "react"

function NameInput() {
  const [name, setName] = useState("")

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <h3>Hello {name}</h3>
    </div>
  )
}
```

---

# Arrays with useState 🔥

## مثال 3️⃣ Render List باستخدام map()

```jsx
import { useState } from "react"

function Fruits() {
  const [fruits, setFruits] = useState(["Apple","Banana","Orange"])

  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  )
}
```

---

## مثال 4️⃣ حذف عنصر باستخدام filter()

```jsx
import { useState } from "react"

function Todo() {
  const [tasks, setTasks] = useState(["Study","Gym","Read"])

  const deleteTask = (task) => {
    setTasks(tasks.filter(t => t !== task))
  }

  return (
    <div>
      {tasks.length === 0 && <p>No tasks!</p>}

      {tasks.map(task => (
        <div key={task}>
          {task}
          <button onClick={() => deleteTask(task)}>Delete</button>
        </div>
      ))}
    </div>
  )
}
```

---

## مثال 5️⃣ البحث باستخدام find()

```js
const user = users.find(u => u.id === 1)
```

---

## مثال 6️⃣ some() و every()

```js
const hasCompleted = tasks.some(t => t.completed)
const allCompleted = tasks.every(t => t.completed)
```

---

## مثال 7️⃣ Ternary Operator

```jsx
<h2>{tasks.length === 0 ? "No tasks" : "Tasks available"}</h2>
```

---

## مثال 8️⃣ Conditional UI

```jsx
<button>
  {isLoggedIn ? "Logout" : "Login"}
</button>
```

---

# 3️⃣ Props

## ما هي Props؟

Props = بيانات تنتقل من **Parent → Child**

---

## مثال Card Component

### Parent

```jsx
<Card title="MacBook" price={2000} />
```

### Child

```jsx
function Card({ title, price }) {
  return <h2>{title} - ${price}</h2>
}
```

---

## Default Props

```jsx
function Button({ text = "Click me" }) {
  return <button>{text}</button>
}
```

---

## Passing Function as Prop 🔥

```jsx
function Parent() {
  const sayHello = () => alert("Hello")
  return <Child onClick={sayHello} />
}

function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>
}
```

---

# 4️⃣ React Fragments

React لا يسمح بإرجاع أكثر من عنصر واحد.

❌ خطأ:

```jsx
return (
  <h1>Hello</h1>
  <p>World</p>
)
```

---

## الحل: Fragment

### الطريقة المختصرة

```jsx
<>
  <h1>Hello</h1>
  <p>World</p>
</>
```

Fragments لا تضيف div للـ DOM.

---

# 5️⃣ Final Project — Task Manager App 🔥

## Structure المشروع

```
src/
├── App.jsx
├── TaskList.jsx
├── TaskItem.jsx
└── AddTask.jsx
```

---

## App.jsx

```jsx
import { useState } from "react"
import TaskList from "./TaskList"
import AddTask from "./AddTask"

function App() {
  const [tasks, setTasks] = useState([])

  const addTask = (text) => {
    setTasks([
      ...tasks,
      { id: Date.now(), text, completed: false }
    ])
  }

  const deleteTask = (id) => {
    setTasks(tasks.filter(t => t.id !== id))
  }

  const toggleTask = (id) => {
    setTasks(tasks.map(t =>
      t.id === id ? { ...t, completed: !t.completed } : t
    ))
  }

  return (
    <>
      <h1>Task Manager</h1>
      <AddTask onAdd={addTask} />

      {tasks.length === 0
        ? <p>No tasks yet</p>
        : <TaskList
            tasks={tasks}
            onDelete={deleteTask}
            onToggle={toggleTask}
          />
      }
    </>
  )
}

export default App
```

---

## AddTask.jsx

```jsx
import { useState } from "react"

function AddTask({ onAdd }) {
  const [text, setText] = useState("")

  const submit = (e) => {
    e.preventDefault()
    if (!text) return

    onAdd(text)
    setText("")
  }

  return (
    <form onSubmit={submit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button>Add</button>
    </form>
  )
}

export default AddTask
```

---

## TaskList.jsx

```jsx
import TaskItem from "./TaskItem"

function TaskList({ tasks, onDelete, onToggle }) {
  return (
    <>
      {tasks.map(task => (
        <TaskItem
          key={task.id}
          task={task}
          onDelete={onDelete}
          onToggle={onToggle}
        />
      ))}
    </>
  )
}

export default TaskList
```

---

## TaskItem.jsx

```jsx
function TaskItem({ task, onDelete, onToggle }) {
  return (
    <div>
      <input
        type="checkbox"
        checked={task.completed}
        onChange={() => onToggle(task.id)}
      />

      <span>
        {task.completed ? <s>{task.text}</s> : task.text}
      </span>

      <button onClick={() => onDelete(task.id)}>Delete</button>
    </div>
  )
}

export default TaskItem
```

---
