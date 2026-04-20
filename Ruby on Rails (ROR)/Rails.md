تمام، خلّيك ماشي step by step كأنك بتنفّذ اللاب من الصفر لحد ما تسلّمه 👇

---

# 🔧 أول حاجة: الـ Setup

## 1. Fork الريبو

- افتح اللينك اللي الدكتور هيبعتّه
    
- دوس **Fork** (فوق على اليمين)
    
- كده بقى عندك نسخة على GitHub بتاعك
    

---

## 2. Clone الريبو على جهازك

افتح terminal واكتب:

```bash
git clone <repo-url>
```

بعدها ادخل جوه الفولدر:

```bash
cd <repo-name>
```

---

## 3. اعمل branch جديد

المطلوب اسمه **lab_two_b**

```bash
git checkout -b lab_two_b
```

---

# 🐞 الجزء المهم: Finding & Fixing Bugs

## 4. شغّل المشروع

حسب نوعه:

### لو Node.js:

```bash
npm install
npm start
```

أو:

```bash
npm run dev
```

### لو Spring Boot:

تشغّله من IntelliJ أو:

```bash
mvn spring-boot:run
```

---

## 5. جرّب endpoint `/users`

افتح في المتصفح:

```
http://localhost:PORT/users
```

🔴 غالبًا هيطلع:

- Error
    
- أو 500
    
- أو مش هيشتغل أصلاً
    

وده الطبيعي عشان فيه bugs

---

# 🧠 إزاي تلاقي الباجز

ركز على الحاجات دي:

### 1. Console Errors

بص في التيرمنال 👇

- أي error message → ده أول bug
    

---

### 2. Routes / Controllers

دور على:

- `/users`
    
- file زي:
    
    - `userController`
        
    - `routes/users.js`
        

شوف:

- هل route مكتوب صح؟
    
- هل بيرجع response؟
    

---

### 3. Database

لو فيه DB:

- هل الاتصال شغال؟
    
- هل query صح؟
    
- هل الجدول موجود؟
    

---

### 4. Common Bugs غالبًا هتلاقيها:

- ❌ variable undefined
    
- ❌ import ناقص
    
- ❌ typo في اسم function
    
- ❌ async/await ناقص
    
- ❌ return مش موجود
    
- ❌ route غلط
    

---

# 🔨 مثال إصلاح Bug

### مثال:

```js
app.get('/users', (req, res) => {
  const users = getUsers
  res.send(users)
})
```

❌ المشكلة: ناسي `()`

✔ الحل:

```js
const users = getUsers()
```

---

# ✅ هدفك النهائي

لما تفتح:

```
http://localhost:PORT/users
```

لازم:

- يرجع 200 OK
    
- ويعرض text أو JSON
    

---

# 📸 Proof

لما يشتغل:

- خد Screenshot للمتصفح
    
- باين فيه `/users` شغال
    

---

# 🚀 Push & PR

## 1. اعمل commit

```bash
git add .
git commit -m "fix: solved bugs in lab 2B"
```

---

## 2. اعمل push

```bash
git push origin lab_two_b
```

---

## 3. اعمل Pull Request

- ادخل GitHub
    
- دوس **Compare & pull request**
    

---

## 4. اسم الـ PR (مهم جدًا)

لازم يكون كده بالظبط:

```
Ali_Ragab_Ghonim/lab_two_b
```

(غيّر الاسم حسب اسمك لو عايز)

---

## 5. حط الصورة في الـ PR

- ارفع Screenshot بتاع `/users`
    

---

# 🔥 Tips مهمه جدًا

- متعدّلش في `main`
    
- اشتغل كله على `lab_two_b`
    
- كل ما تصلّح bug → جرّب `/users` تاني
    
- امشي واحدة واحدة مش كله مرة واحدة
    

---

لو عايز بقى:  
📌 تبعتلي الكود أو الريبو  
وأمشي معاك bug bug لحد ما يشتغل 🔥