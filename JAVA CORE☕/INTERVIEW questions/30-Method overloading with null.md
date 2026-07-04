# **Method Overloading with `null` in Java – لماذا يتم استدعاء `print(String)` وليس `print(Object)`؟**

السؤال ده من أشهر أسئلة الـ Java Interviews لأنه بيختبر فهمك لـ:

- Method Overloading
    
- Inheritance
    
- Compile-Time Polymorphism
    
- Reference Conversion
    
- Java Language Specification (JLS)
    

الكود:

```java
public class Printer {

    void print(Object s) {
        System.out.println("Object is: " + s);
    }

    void print(String s) {
        System.out.println("String is: " + s);
    }
}

class Main {
    public static void main(String[] args) {
        new Printer().print(null);
    }
}
```

---

# أولاً: ما هو الـ Overloading؟

عندنا دالتين بنفس الاسم:

```java
print(Object s)
```

و

```java
print(String s)
```

وده اسمه **Method Overloading** لأن:

- نفس اسم الميثود.
    
- لكن الـ Parameters مختلفة.
    

---

# السؤال الأساسي

إحنا بعتنا:

```java
print(null);
```

طب `null` دي نوعها إيه؟

---

# هل `null` تعتبر Object؟

الناس كتير بتقول:

> null يعني Object

وده **غير صحيح**.

`null` **ليس Object**.

هو Literal خاص في Java يمثل عدم وجود أي Reference.

يعني:

```java
Object obj = null;
```

صح.

وكمان:

```java
String s = null;
```

صح.

وكمان:

```java
Integer i = null;
```

صح.

لأن `null` ينفع يتحول لأي Reference Type.

---

# يبقى Java هتختار مين؟

عندها اختيارين:

```
print(Object)
```

أو

```
print(String)
```

الاتنين يقبلوا `null`.

يبقى ليه اختارت String؟

---

# لأن String أكثر تخصصًا

بص للعلاقة دي:

```
Object
   ↑
String
```

يعني

```java
String IS-A Object
```

كل String هو Object.

لكن

مش كل Object هو String.

---

# Java تختار الـ Most Specific Method

في الـ Overloading، Java عندها قاعدة:

> إذا أكثر من Method ينفعوا،  
> اختار الأكثر تخصصًا (Most Specific).

وبما إن:

```
String
```

أكثر تخصصًا من

```
Object
```

يبقى Java تستدعي:

```java
print(String s)
```

---

# إذًا الـ Output

```text
String is: null
```

---

# تخيلها كأنك رايح لدكتور

عندك:

```
Doctor
```

وعندك:

```
Heart Doctor
```

قلت:

أنا عندي مشكلة في القلب.

مين هيعالجك؟

مش الدكتور العام.

هيختار:

```
Heart Doctor
```

لأنه الأكثر تخصصًا.

Java عملت نفس الحاجة.

---

# طيب لو حذفنا

```java
print(String)
```

هيبقى عندنا:

```java
print(Object)
```

فقط.

يبقى

```java
print(null);
```

هيطبع

```text
Object is: null
```

---

# طب لو عملنا

```java
void print(Integer i)
void print(String s)
```

ثم

```java
print(null);
```

هيحصل إيه؟

الاتنين:

- Integer
    
- String
    

مالهمش علاقة ببعض.

ولا واحد أكثر تخصصًا من الثاني.

يبقى Java تحتار.

---

# النتيجة

```
Compile Error
```

اسمها

```
Reference to print is ambiguous
```

لأن Java مش عارفة تختار.

---

# مثال

```java
class Test {

    void print(String s) {}

    void print(Integer i) {}

    public static void main(String[] args) {

        new Test().print(null);

    }
}
```

النتيجة:

```
Compile Error
```

---

# طب لو عندي

```java
void print(Object o)

void print(CharSequence c)

void print(String s)
```

وجيت كتبت

```java
print(null);
```

Java هتختار

```
String
```

لأنها الأكثر تخصصًا.

---

# ولو عندي

```java
void print(Object o)

void print(CharSequence c)
```

وجيت كتبت

```java
print(null);
```

Java هتختار

```
CharSequence
```

لأن

```
Object
     ↑
CharSequence
```

---

# هل ده Runtime ولا Compile Time؟

الإجابة:

```
Compile Time
```

لأن

Method Overloading

يتم تحديده أثناء الـ Compile.

---

# ماذا يعني "Java follows the specification"؟

الجملة دي معناها:

> Java لا تختار الميثود بشكل عشوائي.

هي تتبع قواعد مكتوبة رسميًا في وثيقة اسمها:

**Java Language Specification (JLS)**

وده المستند الرسمي للغة Java.

كل مترجم Java (زي `javac`) لازم يلتزم بالقواعد الموجودة فيه.

ومن ضمن القواعد:

- إزاي الـ Overloading يشتغل.
    
- إزاي الـ Overriding يشتغل.
    
- إزاي الـ Generics تشتغل.
    
- إزاي الـ Inheritance يشتغل.
    
- ترتيب اختيار الميثود عند وجود أكثر من Method مناسبة.
    

يعني لما حد يقول:

> **Java follows the specification**

المقصود:

> Java بتنّفذ القواعد المكتوبة في الـ **JLS**، مش اجتهاد من المترجم أو من شركة معينة.

---

# ملخص مهم جدًا

| السؤال                                          | الإجابة                                                                                                                                      |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| هل `null` هو Object؟                            | ❌ لا، هو Null Literal يمكن إسناده لأي Reference Type                                                                                         |
| هل `null` يمكن تمريره لـ Object؟                | ✅ نعم                                                                                                                                        |
| هل يمكن تمريره لـ String؟                       | ✅ نعم                                                                                                                                        |
| لماذا اختارت Java `print(String)`؟              | لأنه أكثر تخصصًا (Most Specific Method)                                                                                                      |
| هل الاختيار يتم وقت التشغيل؟                    | ❌ لا، يتم أثناء الـ Compile Time                                                                                                             |
| ماذا يحدث إذا كان عندي `String` و`Integer` فقط؟ | ❌ Compile Error بسبب Ambiguous Method Call                                                                                                   |
| ما معنى "Java follows the specification"؟       | أنها تطبق القواعد الرسمية الموجودة في **Java Language Specification (JLS)**، ومنها قاعدة اختيار **Most Specific Method** في الـ Overloading. |