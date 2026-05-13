
---

# 🤔 يعني اي السؤال؟

**What is a method signature?**

يعني:

> ايه هي المعلومات اللي بتميز method عن غيرها داخل الكلاس؟

الانترفيوور عايز يعرف:

- هل فاهم method overloading ولا لأ
    
- هل فاهم ازاي JVM بتميز بين الميثودز
    

---

# 🎯 الإجابة القصيرة للانترفيو

**A method signature consists of the method name and its parameter list (type, number, and order of parameters).**

بس 👌  
مش الـ return type  
مش الـ access modifier.

---

# 🧠 نشرحها ببساطة

الـ Method Signature = هوية الميثود.

Java بتستخدمه علشان تعرف:  
أي method تنادي لما يحصل call.

---

## مثال

```java
void print(int x)
void print(String s)
void print(int x, int y)
```

التلاتة دول اسمهم `print`  
لكن signatures مختلفة ✔️  
فده اسمه **Method Overloading**.

---

# 💥 أهم نقطة Interview

### هل return type جزء من signature؟ ❌

لااااا ❌

وده سؤال مشهور جدًا.

```java
int sum(int a, int b)
double sum(int a, int b)   // ❌ compile error
```

ليه Error؟

لأن الـ signature واحد:

```
sum(int,int)
```

Java مش بتبص للـ return type.

---

# 🧾 إذن الـ Method Signature تتكون من:

1️⃣ Method name  
2️⃣ Number of parameters  
3️⃣ Type of parameters  
4️⃣ Order of parameters

---

# 🎯 صياغة تحفظها

> A method signature is the combination of the method name and the parameter list (number, type, and order).  
> It is used by the compiler to differentiate between overloaded methods.

لو حابب أديك tricky questions على method overloading زي اللي بتيجي في الانترفيو نتمرن عليها 😈