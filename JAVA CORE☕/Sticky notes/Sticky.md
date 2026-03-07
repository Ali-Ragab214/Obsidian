## 13️⃣ (Tracing – Polymorphism)

class A {  
    void show(){ System.out.println("A"); }  
}  
class B extends A {  
    void show(){ System.out.println("B"); }  
}  
A obj = new B();  
obj.show();

### ✅ Answer:

B

### 💡 Explanation:

Method overriding → runtime polymorphism.


## 15️⃣ (MCQ)

Can static methods be overridden?

### ✅ Answer:

No → they are hidden, not overridden.



## 28️⃣

Checked vs Unchecked exception?

### ✅ Answer:

Checked → compile-time (IOException)  
Unchecked → RuntimeException


## Tracing الكود ده

---

## النتيجة الأول

```
Static
Main
```

---

## ليه Static الأول؟

في Java، لما الـ JVM بتحمل الـ Class بتمر بالخطوات دي بالترتيب:

```
1. Loading    → بتحمل الـ .class file في الميموري
2. Linking    → بتربط الـ class بالـ JVM
3. Initialization → بتشغل الـ Static Blocks الأول
4. Execution  → بتشغل الـ main()
```

---

## الـ Static Block إيه ده؟

```java
static {
    // بيتنفذ مرة واحدة بس
    // لما الـ Class بتتحمل في الميموري
    // قبل أي حاجة تانية حتى الـ main
}
```

---

## الـ Tracing خطوة بخطوة

```
JVM بتشغل البرنامج
        ↓
بتحمل الـ Class Test في الميموري
        ↓
لاقت Static Block → شغّلته فوراً
        ↓
طبعت "Static"
        ↓
دلوقتي بدأت تشغل الـ main()
        ↓
طبعت "Main"
```

---

## مثال أوضح — أكتر من Static Block

```java
class Test {

    static { System.out.println("Static 1"); }
    static { System.out.println("Static 2"); }
    static { System.out.println("Static 3"); }

    public static void main(String[] args) {
        System.out.println("Main");
    }
}
```

```
Output:
Static 1
Static 2
Static 3
Main

← بيتنفذوا بالترتيب من فوق لتحت
```

---

## Static Block مع Object

```java
class Test {

    static { System.out.println("Static Block"); }

    Test() { System.out.println("Constructor"); }

    public static void main(String[] args) {
        System.out.println("Main");
        Test t1 = new Test();
        Test t2 = new Test();
    }
}
```

```
Output:
Static Block   ← مرة واحدة بس لما الـ Class اتحملت
Main
Constructor    ← مع كل object
Constructor    ← مع كل object
```

---

## ملخص الترتيب

```
الترتيب في Java:

1. Static Block     → مرة واحدة لما الـ Class تتحمل
2. main()           → مرة واحدة
3. Constructor      → مع كل new Object()

Static Block  →  main()  →  Constructor  →  Constructor ...
```

---

## امتى بنستخدم Static Block؟

```java
class DatabaseConfig {
    static String url;
    static String username;

    // بنستخدمه لما محتاجين نعمل initialize
    // لـ static variables بكود معقد
    static {
        url      = "jdbc:mysql://localhost/db";
        username = "root";
        System.out.println("DB Config Loaded!");
    }
}
```

```
Static Block بيتستخدم لـ:
✅ Initialize static variables بكود معقد
✅ تحميل drivers أو config مرة واحدة
✅ أي setup محتاج يحصل قبل أي حاجة تانية
```







## 30️⃣

What loads first? Parent or child class?

### ✅ Answer:

Parent first.



## 31️⃣

What is JVM memory structure?

### ✅ Answer:

- Heap
    
- Stack
    
- Method Area
    
- PC Register
    
- Native Stack





## 32️⃣

What is difference between stack and heap?

### ✅ Answer:

Stack → local variables  
Heap → objects



## 33️⃣

What is garbage collector?

### ✅ Answer:

Automatic memory management mechanism.



## مقارنة سريعة 🎯

||الترتيب|السرعة|تحت الكود|
|---|---|---|---|
|`HashMap`|❌ مش مرتب|⚡ O(1)|Hash Table|  عشوائي
|`LinkedHashMap`|✅ Insertion Order|🔶 O(1)|Hash + LinkedList|   ترتيبه نفس ترتيب الادخال 
|`TreeMap`|✅ Natural Order|🔴 O(log n)|Red-Black Tree| بيرتب ابجدي 