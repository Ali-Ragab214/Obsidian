
---
process : is a running program that contain one or more thread 
# أولًا: يعني إيه Process؟

**الـ Process** هو **برنامج شغال بالفعل (Running Program)**.

مثال:

- فتحت Chrome → Process.
    
- فتحت IntelliJ → Process.
    
- فتحت Spotify → Process.
    

كل واحد منهم ليه:

- Memory خاصة بيه.
    
- Resources خاصة بيه.
    
- لو وقع، التانيين بيكملوا عادي.
    

---

# يعني إيه Thread؟

الـ **Thread** هو **جزء من الـ Process بينفذ الكود**.

يعني الـ Process ممكن يحتوي على Thread واحد أو أكتر.

مثال:

```
Chrome Process
│
├── Thread 1 -> يرسم الصفحة
├── Thread 2 -> يحمل الصور
├── Thread 3 -> يشغل JavaScript
└── Thread 4 -> يحمل الملفات
```

كلهم جوه نفس الـ Process.

---

# الفرق بينهم

|Process|Thread|
|---|---|
|برنامج شغال|مسار تنفيذ داخل البرنامج|
|ليه Memory خاصة بيه|بيشارك باقي الـ Threads نفس الـ Heap|
|إنشاءه مكلف|إنشاءه سريع|
|التواصل بين الـ Processes صعب|الـ Threads بتتواصل بسهولة لأنها Shared Memory|
|لو Process وقع غالبًا كله يقفل|لو Thread وقع ممكن باقي الـ Threads تكمل (حسب نوع الخطأ)|

---

# في Java

لما تشغل

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

اللي حصل؟

1. الـ JVM بدأت Process جديدة.
    
2. جوه الـ Process دي اتعمل Main Thread.
    
3. الـ Main Thread نفذ `main()`.
    

يعني الشكل كده:

```
Java Process (JVM)
│
└── Main Thread
      │
      └── main()
```

---

# مثال فيه أكثر من Thread

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Child Thread");
    }
}

public class Main {

    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.start();

        System.out.println("Main Thread");
    }
}
```

الشكل هيبقى:

```
Java Process
│
├── Main Thread
│      └── main()
│
└── Child Thread
       └── run()
```

الاتنين شغالين داخل نفس الـ Process.

---

# أهم نقطة في الجافا

# كل الـ Threads بيشاركوا نفس الـ Heap 

يعني:

```java
class Counter {
    int count = 0;
}
```

لو Thread 1 عدلت `count`

فالـ Thread 2 هيشوف التعديل؛ لأنهم شايفين نفس الـ Object.

لكن كل Thread ليها Stack خاص بيها.

مثال:

```java
public void work() {
    int x = 10;
}
```

المتغير `x` موجود في Stack الخاصة بالـ Thread اللي نفذت `work()`، ومفيش Thread تانية تقدر توصله مباشرة.

---

# تخيلها كده

تخيل شركة.

**Process = الشركة**

وفيها موظفين.

**Thread = الموظفين**

- كل الموظفين بيستخدموا نفس المخزن (Heap) ✅
    
- لكن كل موظف عنده مكتب وورقه الخاصة (Stack) ✅
    

---

## ملخص الانترفيو

**Process**

- Running program.
    
- Own memory.
    
- Contains one or more Threads.
    

**Thread**

- Smallest unit of execution.
    
- Lives inside a Process.
    
- Shares the Heap مع باقي الـ Threads.
    
- Has its own Stack.
    

**جملة مهمة جدًا في الانترفيو:**

> **A process is an independent running program, while a thread is the smallest unit of execution inside that process. Multiple threads share the same Heap memory but each thread has its own Stack memory.**