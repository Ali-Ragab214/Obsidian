تمام — خلّينا نعمل مشروع يلمّ كل الـ OOP في TypeScript بشكل عملي من غير async ولا decorators زي ما طلبت.

## 💡 فكرة المشروع

### 📚 **Library Management System (نظام إدارة مكتبة)**

نظام بسيط لكنه قوي يخلّيك تطبق:

- OOP كامل (Encapsulation, Inheritance, Polymorphism, Abstraction)
    
- Interfaces
    
- Classes
    
- Composition
    
- SOLID بشكل مبدئي
    
- Collections (arrays of objects)
    

---

## 🧠 الفكرة ببساطة

عندك:

- Users (أعضاء المكتبة)
    
- Books (كتب)
    
- Library (بتدير كل حاجة)
    

الـ User يقدر:

- يستعير كتاب
    
- يرجّع كتاب
    

الـ Library:

- تضيف كتب
    
- تعرض الكتب المتاحة
    
- تدير الإعارة
    

---

## 🏗️ Project Structure (Clean OOP Structure)

```
library-system/
│
├── src/
│   │
│   ├── models/
│   │   ├── Book.ts
│   │   ├── User.ts
│   │   ├── Library.ts
│   │
│   ├── interfaces/
│   │   ├── IBorrowable.ts
│   │   ├── IRepository.ts
│   │
│   ├── services/
│   │   ├── LibraryService.ts
│   │   ├── UserService.ts
│   │
│   ├── repositories/
│   │   ├── BookRepository.ts
│   │   ├── UserRepository.ts
│   │
│   ├── enums/
│   │   ├── BookStatus.ts
│   │
│   ├── utils/
│   │   ├── Logger.ts
│   │
│   ├── app.ts
│
├── package.json
├── tsconfig.json
└── README.md
```

---

## 🧱 أهم الكلاسات

### 📘 Book.ts

- id
    
- title
    
- author
    
- status (AVAILABLE / BORROWED)
    

### 👤 User.ts

- id
    
- name
    
- borrowedBooks: Book[]
    

### 🏛️ Library.ts

- books: Book[]
    
- users: User[]
    

---

## 🔌 Interfaces (مهم جدًا للتدريب)

### IBorrowable.ts

```ts
export interface IBorrowable {
  borrowBook(bookId: number, userId: number): void;
  returnBook(bookId: number, userId: number): void;
}
```

### IRepository.ts

```ts
export interface IRepository<T> {
  add(item: T): void;
  getById(id: number): T | undefined;
  getAll(): T[];
  delete(id: number): void;
}
```

---

## ⚙️ Services (Business Logic)

### LibraryService.ts

- إضافة كتاب
    
- إعارة كتاب
    
- استرجاع كتاب
    
- عرض الكتب المتاحة
    

### UserService.ts

- إضافة مستخدم
    
- عرض كتب المستخدم
    

---

## 📦 Enums

### BookStatus.ts

```ts
export enum BookStatus {
  AVAILABLE = "AVAILABLE",
  BORROWED = "BORROWED"
}
```

---

## 🧠 OOP اللي هتطبقه هنا

### 1. Encapsulation

- كل properties تكون private
    
- الوصول عبر methods
    

### 2. Inheritance (اختياري للتطوير)

- ممكن تعمل Person → User
    

### 3. Polymorphism

- طرق مختلفة لإدارة الإعارة أو التحقق
    

### 4. Abstraction

- Interfaces زي IRepository
    

---

## 🚀 Extension Ideas (لو عايز تبقى أقوى)

بعد ما تخلص:

- Add fine system (غرامة تأخير)
    
- Search books by author/title
    
- Book categories
    
- Admin role vs User role
    

---

## 🎯 الهدف من المشروع ده

بعده هتكون فاهم:

- تصميم system كامل OOP
    
- separation of concerns
    
- service layer concept
    
- real-world structure قريب من NestJS
    

---

لو عايز الخطوة الجاية:  
أقدر أبدأ معاك **نبني المشروع كود كود (step by step)** أو أحوله لك لـ **NestJS-ready structure** بعد ما تخلصه 👍