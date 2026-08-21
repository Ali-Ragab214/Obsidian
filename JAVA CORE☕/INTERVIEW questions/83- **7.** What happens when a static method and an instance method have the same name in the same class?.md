
أيوه، دي نقطة مهمة جدًا في Java وبتتسأل في الـ interviews 👌

## السؤال:

**What happens when a static method and an instance method have the same name in the same class?**

الإجابة المختصرة:

> **You cannot declare a static method and an instance method with the same signature in the same class.**  
> Java considers them the same method signature, and this causes a **compile-time error**.

### مثال ❌

```java
class Test {

    static void print() {
        System.out.println("Static");
    }

    void print() {
        System.out.println("Instance");
    }
}
}
```

هنا عندنا:

```java
static void print()
```

و:

```java
void print()
```

الاسم واحد والـ parameters واحد → إذن **نفس signature**.

فـ Java هتطلع compile-time error، لأنك مش هتقدر تعمل:

```java
Test.print();      // static
```

وفي نفس الوقت:

```java
Test obj = new Test();
obj.print();       // instance
```

على أساس إن كل واحدة method مختلفة.

---

## طب لو الـ parameters مختلفة؟

هنا الموضوع مختلف تمامًا ✅

ده اسمه **Method Overloading**، ومسموح.

```java
class Test {

    static void print() {
        System.out.println("Static");
    }

    void print(String name) {
        System.out.println("Instance: " + name);
    }
}
```

ده صحيح لأن الـ signatures مختلفة:

```text
print()
print(String)
```

فتقدر تعمل:

```java
Test.print();

Test obj = new Test();
obj.print("Ali");
```

Output:

```text
Static
Instance: Ali
```

---

## ⚠️ نقطة Interview مهمة جدًا

ممكن الـ interviewer يسألك:

**Can static and instance methods have the same name?**

قول:

> **Yes, but only if their method signatures are different, which means they are overloaded. They cannot have the same signature in the same class.**

مثال:

```java
static void calculate() { }

void calculate(int x) { }
```

✅ Allowed

لكن:

```java
static void calculate() { }

void calculate() { }
```

❌ Compile-time error.

---

### ليه Java بتمنعهم؟

لأن `static` و `instance` مش جزء من **method signature** في Java.

يعني:

```java
static void test()
```

و

```java
void test()
```

بالنسبة للـ compiler الاتنين:

```text
test()
```

فمينفعش يتواجدوا مع بعض في نفس class.

### احفظها كده للانترفيو 🧠

**Static/instance difference does NOT create overloading.**

الـ overloading يعتمد على اختلاف الـ **parameters**:

```text
test()       → test()
test(int)    → test(int)       ✅
test(String) → test(String)    ✅
```

لكن:

```text
static test()
instance test()
```

❌ مش كفاية لوحدها لعمل overloading.