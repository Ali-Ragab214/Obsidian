

---
# 🧠 الشكل العام (Syntax)

<T extends ClassName & Interface1 & Interface2>

⚠️ مهم جدًا:

- أول bound لازم يكون **Class**
    
- والباقي **Interfaces**
    
- ومينفعش تحط أكتر من Class
- # 📌 لماذا لا يمكن استخدام primitive؟

❌ `List<int>`  
✔️ `List<Integer>`

لأن:  
Generics تعمل مع Objects فقط.
# 🎯 يعني إيه Generic Method؟


Generic Method = ميثود تقدر تشتغل مع **أي نوع بيانات**  
بدون ما تعيد كتابتها لكل نوع.

بدل ما تعمل:

```java
int add(int a, int b)
double add(double a, double b)
```

تعمل ميثود واحدة Generic تشتغل مع أي نوع.

---

# 🧠 الشكل العام للـ Generic Method

```java
public <T> void methodName(T parameter) {
    // code
}
```

### 📌 أهم نقطة

`<T>` بتتحط قبل نوع الرجوع مباشرة.

---

# 🟢 مثال 1 – طباعة أي نوع

```java
public class Test {

    public static <T> void printValue(T value) {
        System.out.println(value);
    }

    public static void main(String[] args) {
        printValue(10);
        printValue("Ali");
        printValue(3.14);
    }
}
```

### الناتج:

```
10
Ali
3.14
```

الميثود اشتغلت مع:

- Integer
    
- String
    
- Double
    

---

# 🟢 مثال 2 – ترجع قيمة

```java
public static <T> T returnValue(T value) {
    return value;
}
```

الاستخدام:

```java
String name = returnValue("Ali");
Integer num = returnValue(100);
```

---

# 🎯 Generic Method بعدة Types


تقدر تستخدم أكتر من Type:

```java
public static <T, U> void printTwo(T first, U second) {
    System.out.println(first + " " + second);
}
class Box<T extends Number> { this called bounded generic
    T value;
}
```

الاستخدام:

```java
printTwo("Ali", 25);
printTwo(10, 3.5);
```

---

# 🔥 Generic Method مع Constraint

أحيانًا مش عايز أي نوع…  
عايز نوع معين أو اللي يرث منه.

نستخدم:

## extends

```java
public static <T extends Number> void printNumber(T num) {
    System.out.println(num);
}
```

دي تقبل:

- Integer
    
- Double
    
- Float
    

لكن ❌ مش تقبل String

---

# 🎯 الفرق بين Generic Class و Generic Method

## Generic Class

```java
class Box<T> {
    T value;
    
}
```

## Generic Method

```java
public static <T> void method(T value)
```

الميثود ممكن تبقى Generic حتى لو الكلاس مش Generic.

---

# 🧠 مثال مهم جدًا (Swap)

```java
public static <T> void swap(T[] array, int i, int j) {
    T temp = array[i];
    array[i] = array[j];
    array[j] = temp;
}
```

الاستخدام:

```java
String[] names = {"Ali", "Omar"};
swap(names, 0, 1);
```

---

# 📌 ليه نستخدم Generics؟

✅ Type Safety  
✅ منع Casting  
✅ إعادة استخدام الكود  
✅ تقليل الأخطاء وقت الـ Runtime

---

# 🔥 مقارنة بدون Generic

```java
public static void print(Object obj)
```

هتحتاج تعمل Casting بعدين.

لكن مع Generic:

```java
public static <T> void print(T obj)
```

الـ Compiler بيحميك.

---

# 🎯 شكل تحفظه للامتحان

```java
public static <T> T methodName(T parameter)
```

---

# 💡 سؤال لك بقى 😈

إيه الغلط في الكود ده؟

```java
public static T test(T value) {
    return value;
}
```

فكر كويس 💪

سؤال ممتاز جدًا يا علي 👏🔥  
وده بيفرق بين اللي فاهم Generics واللي حافظ بس 💪

الكود بتاعك:

```java
public static <T> void add(T a, T b) {
    System.out.println(a + b);
}
```

---

# 🎯 ليه ده غلط؟

لأن:

`T` نوع **غير معروف وقت الـ Compile**

والـ `+` operator مش شغال مع أي نوع Generic.

---

## 🧠 Java بتشوف T إيه؟

لما تكتب:

```java
<T>
```

Java بتتعامل معاه كأنه:

```java
Object
```

يعني فعليًا الكود بيتشاف كده:

```java
Object a;
Object b;
System.out.println(a + b); // ❌ غلط
```

وـ `+` مش شغال مع Object.

---

# 🔥 طيب ليه `+` شغال مع String؟

لأن Java عندها special handling للـ String:

```java
System.out.println("Ali" + "Omar");
```

لكن Generic مش مضمون إنه String.

---

# 🎯 الحلول الصح

## ✅ الحل 1: تعمل Constraint

لو عايز أرقام بس:

```java
public static <T extends Number> double add(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

هنا Number  دا الكلاس اللي   Integer ,Double,Float والكلاسات دي بترث منه يعني دول كدا اوبجكتس مينفعش اقول  a+b  لازم اجيب القيمه ال double  بتاعتهم مثلا لاني مينفعش اجمع two objects مفيش هنا operator overloading
### الاستخدام:

```java
System.out.println(add(10, 20));
System.out.println(add(3.5, 2.1));
```

---

## ✅ الحل 2: لو عايز Strings بس

مش محتاج Generic أصلاً:

```java
public static void add(String a, String b) {
    System.out.println(a + b);
}
```

---

# 🧠 أهم قاعدة في Generics

> Generics بتشتغل مع Types  
> لكن مش بتدعم Operators

يعني:

- ❌ مش ينفع `+`
    
- ❌ مش ينفع `-`
    
- ❌ مش ينفع `*`
    
- ❌ مش ينفع `new T()`
    

---

# 🔥 مثال يوضح الفكرة

ده ينفع:

```java
public static <T> void print(T value) {
    System.out.println(value);
}
```

لكن ده لأ:

```java
T result = a + b;  // ❌
```

---

# 🎯 ليه Java عملت كده؟

عشان:

Generics في Java معمولة بـ **Type Erasure**

يعني وقت التشغيل النوع بيتمسح ويتحول لـ Object.

---

# 🧠 خلاصة تحفظها

❌ متستخدمش Operators مع Generic  
✅ استخدم methods زي doubleValue()  
✅ أو اعمل overloading

---

لو عايز أشرحلك Type Erasure بطريقة تخليك تفهم Generics 100%؟ 😈🔥



public  static <T extends  Number> void add(T a , T b)  
{  
    System.out.println(a.intValue() + b.intValue());  
}

###String in java is a final class that none can extend it 





سؤال ممتاز يا علي 👏🔥  
ده من أهم أفكار الـ Generics في Java.

---

# 🎯 يعني إيه **Bounded Type Parameter** ؟

لما أكتب:

```java
<T extends Something>
```

أنا كده بحط **قيد (Bound)** على النوع T.

يعني بقول:

> T لازم يكون النوع ده أو أي حاجة تورث منه.

---

# 🧠 طيب ليه نعمل كده أصلاً؟

علشان:

✅ نمنع أنواع مش مناسبة  
✅ نقدر نستخدم methods معينة بأمان  
✅ نحافظ على Type Safety

---

# 🔥 مثال يوضح المشكلة بدون Bound

```java
public static <T> void print(T value) {
    System.out.println(value.doubleValue()); // ❌ Error
}
```

ليه Error؟  
لأن T ممكن يكون String  
و String معندوش `doubleValue()`.

---

# ✅ الحل: Bounded Type

```java
public static <T extends Number> void print(T value) {
    System.out.println(value.doubleValue());
}
```

كده Java ضمنت إن T:

- Integer
    
- Double
    
- Float
    
- Long
    

لأن كلهم بيرثوا من `Number`.

---

# 🎯 الاستخدام

```java
print(10);      // ✔
print(3.14);    // ✔
print("Ali");   // ❌ Compile error
```

---

# 🟢 مثال تاني عملي

عايز تعمل method ترجع أكبر عنصر:

```java
public static <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

هنا قولنا:

> T لازم يكون قابل للمقارنة

عشان نستخدم `compareTo()`.

---

# 🔥 أنواع الـ Bounds

## 1️⃣ Upper Bound (الأشهر)

```java
<T extends ClassName>
```

---

## 2️⃣ Multiple Bounds

```java
<T extends Number & Comparable<T>>
```

لازم:

- أول واحد class
    
- والباقي interfaces
    

---

# 🎯 الفرق بين:

### ❌ Unbounded

```java
<T>
```

أي نوع

### ✅ Bounded

```java
<T extends Number>
```

أنواع محددة فقط

---

# 🧠 ليه ده مهم في الامتحان؟

لأن بدون Bound  
Java تعتبر T = Object

ومش هتسمحلك تستخدم methods متخصصة.

---

# 💡 الخلاصة الذهبية

> Bounded Type Parameter = تقييد النوع علشان أضمن إن عندي methods معينة أقدر أستخدمها بأمان.

---

لو عايز أسألك سؤال tricky بقى 😈

ينفع أكتب:

```java
<T super Number>
```

في تعريف Generic Method؟ ولا لأ؟ وليه؟ 🔥