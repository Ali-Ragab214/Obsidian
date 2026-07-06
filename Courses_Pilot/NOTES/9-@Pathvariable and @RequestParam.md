الفرق بين `@PathVariable` و `@RequestParam` مهم جدًا في تصميم الـ REST APIs 👌

خلّينا نبسطها بطريقة واضحة:

---

# 🟢 أولًا: @PathVariable

## 📌 معناها:

بتاخد القيمة من **داخل الـ URL نفسه (path)**

---

## 🧠 مثال:

```java
@GetMapping("/courses/{id}")
public Course getCourse(@PathVariable Long id)
```

## 🔥 Request:

```
GET /courses/5
```

## 👉 الناتج:

```text
id = 5
```

---

## 🧠 استخدامه إمتى؟

لـ **resource محدد**

- get course by id
    
- delete course
    
- update course
    

---

## 🟢 شكل URL:

```
/courses/10
/users/3
/orders/99
```

---

# 🔵 ثانيًا: @RequestParam

## 📌 معناها:

بتاخد القيمة من **query string بعد علامة ؟**

---

## 🧠 مثال:

```java
@GetMapping("/courses")
public List<Course> getCourses(@RequestParam int page)
```

## 🔥 Request:

```
GET /courses?page=2
```

## 👉 الناتج:

```text
page = 2
```

---

## 🧠 استخدامه إمتى؟

لـ **filters / optional data**

- pagination
    
- search
    
- sorting
    
- filtering
    

---

## 🟢 شكل URL:

```
/courses?page=1&size=10&sort=asc
```

---

# ⚖️ مقارنة سريعة جدًا

|النوع|@PathVariable|@RequestParam|
|---|---|---|
|المكان|داخل URL path|بعد ?|
|الاستخدام|resource محدد|filters / options|
|mandatory|غالبًا required|ممكن optional|
|مثال|/courses/5|/courses?page=1|

---

# 🧠 مثال يوضح الفرق مع بعض

## ✔ PathVariable:

```java
@GetMapping("/courses/{id}")
```

```
/courses/5
```

---

## ✔ RequestParam:

```java
@GetMapping("/courses")
```

```
/courses?page=1&size=10
```

---

# 🔥 استخدامهم مع بعض

ممكن تستخدم الاتنين مع بعض:

```java
@GetMapping("/courses/{id}")
public Course getCourse(
        @PathVariable Long id,
        @RequestParam String lang
)
```

```
/courses/5?lang=en
```

---

# 🏆 الخلاصة

## ✔ @PathVariable

> لما تكون بتتعامل مع "resource معين"

## ✔ @RequestParam

> لما تكون بتفلتر أو بتضيف options

---

# 💡 قاعدة سهلة تحفظها

> 🔹 PathVariable = جزء من الطريق (URL path)  
> 🔹 RequestParam = إعدادات إضافية (query)

---

لو عايز أشرح لك:

- إمتى تستخدم كل واحد في design حقيقي لمشروع كبير
    
- أو best practice في pagination + filtering مع بعض
    

قول 👍