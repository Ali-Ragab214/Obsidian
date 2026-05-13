

---

# 🎯 Short Interview Answer (احفظها)

**Static members belong to the class, not to objects.**

- Static variable → shared between all objects
    
- Static method → can be called without creating an object
    

---

# 🧠 شرح الفكرة ببساطة

في Java عندنا نوعين من الأعضاء داخل الكلاس:

|Type|Belongs to|
|---|---|
|Instance members|object|
|Static members|class|

يعني ايه الكلام ده؟

الكلاس نفسه ليه نسخة واحدة في الميموري  
لكن الـ objects ممكن يكون منهم 1000 نسخة.

---

# 🟦 Static Variables

## 💡 التعريف

متغير مرتبط بالكلاس نفسه وليس بالـ object.

كل الـ objects بيشتركوا في **نفس النسخة** من المتغير.

## مثال

```java
class Student {
    static int count = 0;

    Student() {
        count++;
    }
}
```

```java
Student s1 = new Student();
Student s2 = new Student();
Student s3 = new Student();

System.out.println(Student.count);
```

النتيجة:

```
3
```

ليه؟  
لأن فيه **نسخة واحدة فقط** من `count` لكل الطلاب.

لو كانت مش static كان كل object هيبقى عنده نسخة خاصة.

---

## 🎯 امتى نستخدم Static Variable؟

في interview قول:

- Counters (عدد المستخدمين / عدد الطلبات)
    
- Constants (PI – MAX_SIZE)
    
- Shared configuration
    

---

# 🟩 Static Methods

## 💡 التعريف

Method تابعة للكلاس ويمكن استدعاؤها بدون إنشاء object.

```java
Math.sqrt(25);
```

مفيش object من `Math` اتعمل.

---

## ⚠️ أهم نقطة Interview

### static method تقدر توصل فقط إلى:

- static variables
    
- static methods
    

ولا يمكنها الوصول إلى instance members مباشرة.

### ليه؟ (سؤال مشهور جداً)

لأن static method ممكن تتنادى قبل إنشاء أي object.

يعني مفيش instance أصلاً!

---

## مثال يوضح الفكرة

```java
class Test {
    int x = 5;
    static int y = 10;

    static void print() {
        // System.out.println(x); ❌ ERROR
        System.out.println(y);   // ✔ OK
    }
}
```

---

# 💥 الفرق النهائي (أهم جزء)

|Static Variable|Static Method|
|---|---|
|Shared data|Shared behavior|
|One copy in memory|One function in memory|
|Accessed using class name|Called using class name|

---

# 🧠 Interview Bonus Tip

ليه بنستخدم static كثير في utility classes؟

زي:

- Math
    
- Collections
    
- Arrays
    

الإجابة:

لأننا لا نحتاج إنشاء object لتنفيذ العمليات.  
الـ class عبارة عن **helper functions فقط**.

---

لو حابب أديك أسئلة follow-up اللي بتيجي بعد السؤال ده في الانترفيو قولّي 😈