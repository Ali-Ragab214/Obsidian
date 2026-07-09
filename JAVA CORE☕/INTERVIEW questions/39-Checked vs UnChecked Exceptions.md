ده من أشهر أسئلة الـ Java Interviews، وغالبًا بيتسأل بالطريقة دي:

> **What's the difference between Checked and Unchecked Exceptions?**

أو

> **Why do we have Checked Exceptions in Java?**

خلينا نشرحها من الصفر.

---

# أولاً: يعني إيه Exception؟

الـ Exception هو حدث غير طبيعي بيحصل أثناء تنفيذ البرنامج (Runtime) ويؤدي إلى تعطيل الـ Flow الطبيعي.

مثال:

```java
int x = 10 / 0;
```

هنا هيحصل:

```text
ArithmeticException
```

---

# Hierarchy بتاع Exceptions

```text
                Throwable
               /         \
          Error       Exception
                          |
              -----------------------
              |                     |
      RuntimeException         Checked Exceptions
              |
      NullPointerException
      ArithmeticException
      IndexOutOfBoundsException
      IllegalArgumentException
      ...
```

احفظ الرسمة دي لأنها مهمة جدًا.

---

# أولاً: Checked Exception

## التعريف

هي Exception بيتحقق منها الـ Compiler.

يعني قبل ما البرنامج يشتغل، الـ Compiler هيقولك:

> "لازم تتعامل مع الـ Exception دي."

لو متعاملتش معاها، الكود مش هيعمل Compile.

---

## مثال

```java
FileReader reader = new FileReader("data.txt");
```

هيطلع Error:

```text
Unhandled exception: FileNotFoundException
```

ليه؟

لأن الملف ممكن يكون مش موجود.

إذن Java مجبراك تتعامل مع الحالة دي.

---

## الحل الأول

```java
try {
    FileReader reader = new FileReader("data.txt");
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

---

## الحل الثاني

```java
public void readFile() throws FileNotFoundException {

}
```

يعني:

> "أنا مش هتعامل مع الـ Exception، واللي بينادي الميثود هو اللي يتعامل."

---

# أمثلة على Checked Exceptions

- IOException
    
- SQLException
    
- FileNotFoundException
    
- ParseException
    
- InterruptedException
    

كلهم لازم

```java
try-catch
```

أو

```java
throws
```

---

# ليه اسمها Checked؟

لأن

الـ Compiler

بيعمل Check عليها.

قبل ما البرنامج يشتغل.

---

# ثانياً: Unchecked Exception

## التعريف

هي Exception

الـ Compiler

مش بيجبرك تتعامل معاها.

الكود هيعمل Compile عادي.

لكن ممكن يقع أثناء التشغيل.

---

## مثال

```java
String name = null;

System.out.println(name.length());
```

هيعمل Compile.

لكن أثناء التشغيل.

هيطلع

```text
NullPointerException
```

---

مثال تاني

```java
int[] arr = {1,2,3};

System.out.println(arr[10]);
```

Compile

✔

Runtime

```text
ArrayIndexOutOfBoundsException
```

---

مثال ثالث

```java
int x = 10 / 0;
```

هيطلع

```text
ArithmeticException
```

---

# أشهر Unchecked Exceptions

- NullPointerException
    
- ArithmeticException
    
- IllegalArgumentException
    
- IllegalStateException
    
- NumberFormatException
    
- ArrayIndexOutOfBoundsException
    
- ClassCastException
    

---

# ليه اسمها Unchecked؟

لأن

الـ Compiler

مش بيعمل Check عليها.

---

# ليه Java فرقت بينهم؟

لأن فيه نوعين من الأخطاء.

---

## النوع الأول

أخطاء خارج سيطرة البرنامج.

مثلاً

ملف مش موجود.

الشبكة فصلت.

الداتا بيز وقعت.

دي أخطاء متوقعة.

ولازم تتعامل معاها.

عشان كده

Checked Exception.

---

## النوع الثاني

أخطاء من المبرمج.

مثلاً

```java
String name = null;

name.length();
```

ده Bug.

Java بتقول:

"أنا مش هاجبرك تكتب try-catch."

أصل المشكلة في الكود نفسه.

---

# مثال Interview

ليه

NullPointerException

Unchecked؟

لأن الحل

مش

```java
try{
...
}
```

الحل الحقيقي

إنك تراجع الكود.

وتمنع الـ null.

---

# مثال عملي

## Checked

```java
public void read() {

    FileReader reader =
      new FileReader("test.txt");

}
```

Compile Error.

---

## بعد الإصلاح

```java
public void read() {

    try {

        FileReader reader =
          new FileReader("test.txt");

    } catch(FileNotFoundException e){

    }

}
```

---

# مثال Unchecked

```java
public void divide() {

    int x = 10 / 0;

}
```

Compile

✔

Runtime

❌

---

# هل ينفع أمسك Unchecked؟

أيوة.

```java
try{

    int x = 10/0;

}catch(Exception e){

}
```

لكن

مش مطلوب.

---

# إمتى أستخدم Checked Exception؟

لو المشكلة

متوقعة

وممكن المستخدم يحلها.

مثلاً

- ملف مش موجود.
    
- API مش شغال.
    
- Database مقفولة.
    

---

# إمتى أستخدم Runtime Exception؟

لو الخطأ

Bug

فى البرنامج.

مثلاً

- null
    
- index
    
- cast
    
- invalid argument
    

---

# فى Spring Boot

مثلاً

```java
public User findUser(Long id){

}
```

لو المستخدم مش موجود.

هل أعمل

Checked؟

لا.

الأفضل

```java
throw new UserNotFoundException();
```

وتخليها

```java
extends RuntimeException
```

لأن دي Business Exception.

---

# ليه معظم الشركات بتستخدم Runtime Exceptions؟

لأن Checked Exceptions بتخلي الكود مليان:

```java
throws
throws
throws
throws
throws
```

وده بيصعب قراءة الكود.

عشان كده

Spring

Hibernate

JPA

NestJS

معظمهم بيعتمدوا على Runtime Exceptions.

---

# جدول مقارنة

|المقارنة|Checked Exception|Unchecked Exception|
|---|---|---|
|الـ Compiler|يجبرك تتعامل معها|لا يجبرك|
|وقت الاكتشاف|Compile Time|Runtime|
|ترث من|`Exception`|`RuntimeException`|
|تحتاج try-catch؟|نعم أو `throws`|اختياري|
|السبب|ظروف خارجية متوقعة|أخطاء برمجية (Bugs)|
|أمثلة|`IOException`, `SQLException`|`NullPointerException`, `ArithmeticException`|

---

# سؤال مشهور في الانترفيو

**Why does Spring prefer Runtime Exceptions?**

الإجابة:

> **Spring prefers Runtime Exceptions because they keep the code cleaner and avoid excessive `try-catch` blocks and `throws` declarations. Most Runtime Exceptions indicate programming errors or business rule violations that should be handled globally, often using mechanisms like `@ControllerAdvice`, rather than being forced to handle them at every method call.**

---

# الإجابة الذهبية (30 ثانية)

> **Checked Exceptions are checked by the compiler, so they must be either caught with a `try-catch` block or declared using `throws`. They represent recoverable situations such as file or database access failures. Unchecked Exceptions extend `RuntimeException` and are not checked at compile time. They usually indicate programming errors like `NullPointerException` or `IndexOutOfBoundsException`. In modern Spring Boot applications, Runtime Exceptions are generally preferred because they keep the code cleaner and are typically handled centrally using Global Exception Handling.**

💡 **نصيحة للإنترفيو:** لو اتسألت "أنت في مشاريع Spring Boot بتستخدم أنهي؟"، جاوب: **"غالبًا بستخدم Runtime Exceptions مع Global Exception Handling (`@ControllerAdvice`) لأن ده الأسلوب الشائع في Spring، بينما Checked Exceptions بستخدمها فقط لما أكون بتعامل مع عمليات I/O أو مكتبات بتفرضها."** ده هيبين إنك فاهم الجانب العملي، مش بس التعريف النظري.