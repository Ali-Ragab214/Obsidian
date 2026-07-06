# 📘 Topic: Static Variables and Static Methods in Java (Common Interview Trap)

الصورة فيها **خطأين** في الكود، والسؤال مشهور جدًا في الـ Interviews لأنه بيختبر فهمك للـ **static**.

الكود الموجود في الصورة:

```java
public static void main(String[] args) {
    print();
}

static void print() {
    static int a = 10;
    System.out.println(a);
}
```

---

# أولًا: هل الكود ده هيشتغل؟

❌ **لا، الكود لن يترجم (Compile Error).**

هيطلع Error عند السطر ده:

```java
static int a = 10;
```

لأن Java **لا تسمح** بتعريف متغير `static` داخل Method.

---

# ليه Java منعت ده؟

نفهم الأول يعني إيه `static`.

المتغير الـ Static:

- بيتخزن في **Method Area**.
    
- بيتعمله Memory مرة واحدة فقط.
    
- بيعيش طول عمر البرنامج.
    
- كل الـ Objects بتشارك نفس النسخة.
    

أما الـ Local Variable:

```java
void print() {
    int x = 10;
}
```

ده بيتخزن في:

```
Stack Memory
```

ويتعمل كل مرة الميثود تتنادي.

بعد انتهاء الميثود...

```
Stack Frame
        ↓
يتحذف
```

---

## المشكلة

لو Java سمحت بكده:
  
```java
void print() {                                       
    static int a = 10          
}
```

يبقى عندنا متغير:

- موجود داخل Stack
    

وفى نفس الوقت

- المفروض يعيش طول عمر البرنامج
    

وده تعارض.

علشان كده Java قالت:

> **ممنوع Static Local Variables.**

---

# Compile Error

هيكون قريب من:
 
```text
Illegal start of expression
```

أو

```text
modifier 'static' not allowed here
```

حسب الـ Compiler.

---

# طب ليه C/C++ بتسمح؟

في C++:

```cpp
void print() {
    static int x = 10;
}
```

المتغير بيتخزن في Static Storage.

وليس على الـ Stack.

ولذلك Java اختارت إنها تمنع الفكرة دي بالكامل.

---

# طيب لو خليناه Global؟

يعني كده:

```java
public class Main {

    static int a = 10;

    public static void main(String[] args) {
        print();
    }

    static void print() {
        System.out.println(a);
    }
}
```

هل هيشتغل؟

✅ نعم.

Output

```
10
```

---

# ليه هنا مفيش Error؟

لأن:

```java
static int a = 10;
```

مش Local Variable.

ده اسمه

```
Class Variable
```

أو

```
Static Field
```

ومكانه الطبيعي داخل الـ Class.

---

# مكانه في الذاكرة

```
Method Area

Main.class
------------
static int a = 10
```

أما

```
print()
```

فقط بتقرأ القيمة.

---

# لو نادينا print() أكتر من مرة؟

```java
public class Main {

    static int a = 10;

    static void print() {
        System.out.println(a);
    }

    public static void main(String[] args) {

        print();
        print();
        print();

    }

}
```

Output

```
10
10
10
```

لأن المتغير واحد فقط.

---

# مثال آخر

```java
public class Main {

    static int count = 0;

    static void print() {
        count++;
        System.out.println(count);
    }

    public static void main(String[] args) {

        print();
        print();
        print();

    }
}
```

Output

```
1
2
3
```

ليه؟

لأن

```
count
```

متغير Static.

مش بيتعمل من جديد كل مرة.

---

# طب لو شيلنا Static؟

```java
class Main {

    int count = 0;

    void print() {
        count++;
        System.out.println(count);
    }

    public static void main(String[] args) {

        Main m1 = new Main();
        Main m2 = new Main();

        m1.print();
        m2.print();

    }

}
```

Output

```
1
1
```

لأن كل Object عنده نسخة خاصة.

---

# طب ينفع نخليها Global بس بدون Static؟

ينفع.

```java
class Main {

    int a = 10;

    void print() {
        System.out.println(a);
    }

    public static void main(String[] args) {

        Main obj = new Main();

        obj.print();

    }

}
```

Output

```
10
```

لكن هنا `a` أصبحت **Instance Variable**، ولكل Object نسخة مستقلة منها، فلا يمكنك الوصول إليها من `main` مباشرة لأن `main` نفسها `static`.

---

# لماذا استطعنا استدعاء `print()` من `main`؟

لأن:

```java
static void print()
```

والـ

```java
main()
```

أيضًا Static.

الـ Static Method تستطيع استدعاء Static Methods مباشرة.

لكن لو كانت:

```java
void print()
```

سيظهر Error:

```java
public static void main(String[] args) {
    print();
}
```

```
Non-static method print() cannot be referenced from a static context
```

ويجب إنشاء Object:

```java
Main obj = new Main();
obj.print();
```

---

# أشهر أسئلة الانترفيو

### ❓هل Java تدعم Static Local Variables؟

**الإجابة:** لا. لا يمكن تعريف متغير `static` داخل Method أو Constructor أو أي Block.

---

### ❓أين يمكن استخدام `static`؟

- ✅ مع المتغيرات داخل الـ Class (Static Fields)
    
- ✅ مع الـ Methods
    
- ✅ مع الـ Nested Classes
    
- ✅ مع الـ Static Initialization Blocks
    

---

### ❓لماذا تمنع Java الـ Static Local Variables؟

لأن الـ Local Variables تُنشأ داخل الـ Stack Frame وتنتهي بانتهاء استدعاء الميثود، بينما الـ `static` يجب أن يعيش طوال عمر البرنامج ويُخزن مرة واحدة على مستوى الـ Class. لذلك لا يتوافق المفهومان، ومنعت Java هذا الاستخدام من الأساس.

---

# ملخص سريع

| الكود                                           | Compile؟ | السبب                               |
| ----------------------------------------------- | -------- | ----------------------------------- |
| `static int a = 10;` داخل Method                | ❌        | Java لا تدعم Static Local Variables |
| `static int a = 10;` داخل Class                 | ✅        | Static Field                        |
| `int a = 10;` داخل Method                       | ✅        | Local Variable                      |
| `int a = 10;` داخل Class                        | ✅        | Instance Variable                   |
| استدعاء Static Method من `main`                 | ✅        | كلاهما Static                       |
| استدعاء Non-static Method من `main` بدون Object | ❌        | لأن `main` تعمل في Static Context   |