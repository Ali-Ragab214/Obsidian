ده من **أشهر أسئلة Java Core**، وبيتسأل في أي Interview تقريبًا لأنه بيختبر فهمك لمفهوم مهم جدًا:

> **How does Java pass arguments to methods?**

الإجابة الصحيحة هي:

> **Java is always Pass-by-Value.**

ودي من أكتر المفاهيم اللي الناس بتتلخبط فيها، خصوصًا مع الـ Objects.

---

# Interview Question

## Is Java Pass-by-Value or Pass-by-Reference?

### Short Answer

**Java is always Pass-by-Value.**

Java **never** uses Pass-by-Reference.

حتى لو كنت بتمرر Object.

---

# أولًا: يعني إيه Pass-by-Value؟

معناه إن الميثود **بتاخد نسخة (Copy)** من القيمة.

يعني أي تعديل على النسخة **مش هيأثر** على المتغير الأصلي.

---

# مثال الصورة

```java
class Demo {

    static void change(int x) {
        x = 100;
    }

    public static void main(String[] args) {

        int a = 10;

        change(a);

        System.out.println(a);
    }
}
```

---

## السؤال

هيطبع كام؟

الإجابة:

```text
10
```

---

# ليه؟

قبل استدعاء الميثود:

```text
main()

a
│
▼
10
```

لما نكتب:

```java
change(a);
```

Java بتعمل Copy للقيمة.

يعني:

```text
main()

a
│
▼
10


change()

x
│
▼
10
```

لاحظ:

**x مش هو a**

دي نسخة جديدة.

---

بعد السطر:

```java
x = 100;
```

هيبقى:

```text
main()

a
│
▼
10


change()

x
│
▼
100
```

لما الميثود تنتهي:

```text
change() destroyed

main()

a
│
▼
10
```

إذن الناتج:

```text
10
```

---

# إذًا ما هو الـ Value؟

الـ Value مش شرط تكون رقم.

في Java يوجد نوعان من القيم.

---

# النوع الأول: Primitive Values

مثل:

```java
int
double
char
boolean
```

مثال:

```java
int x = 5;
```

القيمة نفسها هي:

```text
5
```

عند تمريرها:

```java
change(x);
```

الذي يُنسخ هو:

```text
5
```

---

# النوع الثاني: Reference Values

وده الجزء اللي بيلخبط الناس.

مثال:

```java
Person p = new Person();
```

هنا:

```text
Person Object
```

موجود في الـ Heap.

أما المتغير:

```java
p
```

فهو موجود في الـ Stack.

وهو لا يحتوي على الـ Object.

بل يحتوي على:

**Reference**

أو

**Address**

---

تخيل:

```text
Stack

p
│
▼
0x100
```

وفي الـ Heap:

```text
0x100
────────────
Person
name = Ali
age = 22
```

---

# ماذا يحدث عند تمرير Object؟

```java
change(p);
```

هل Java تمرر الـ Object؟

لا.

Java تعمل Copy للـ Reference.

يعني:

قبل الاستدعاء:

```text
main()

p
│
▼
0x100
```

بعد الاستدعاء:

```text
main()

p
│
▼
0x100


change()

person
│
▼
0x100
```

لاحظ:

الذي تم نسخه هو:

```text
0x100
```

وليس الـ Object.

---

# ماذا يحدث لو غيرنا الـ Object؟

```java
class Person {

    String name;

}
```

```java
static void change(Person p){

    p.name = "Ahmed";

}
```

```java
Person person = new Person();

person.name = "Ali";

change(person);

System.out.println(person.name);
```

الناتج:

```text
Ahmed
```

---

# لماذا؟

لأن:

```text
person
```

و

```text
p
```

يشيران لنفس الـ Object.

```text
Stack

person ───────┐
              │
              ▼
           0x100
              ▲
              │
p─────────────┘


Heap

0x100

Person

name = Ahmed
```

أي تعديل على الـ Object يظهر للجميع.

---

# ماذا يحدث لو غيرنا الـ Reference نفسه؟

مثال:

```java
static void change(Person p){

    p = new Person();

    p.name = "Ahmed";

}
```

ثم:

```java
Person person = new Person();

person.name = "Ali";

change(person);

System.out.println(person.name);
```

الناتج:

```text
Ali
```

---

# لماذا؟

لأن:

قبل:

```text
person
 │
 ▼
0x100
```

بعد:

```java
p = new Person();
```

أصبح:

```text
person
 │
 ▼
0x100



p
 │
 ▼
0x200
```

النسخة فقط هي التي تغيرت.

أما:

```text
person
```

فمازال يشير إلى:

```text
0x100
```

---

# ملخص مهم

## Primitive

```java
int x = 10;
```

يتم نسخ:

```text
10
```

---

## Object

```java
Person p = new Person();
```

يتم نسخ:

```text
Reference
```

وليس الـ Object.

---

# لماذا يظن البعض أن Java Pass-by-Reference؟

لأن هذا المثال:

```java
person.name = "Ahmed";
```

يغير الـ Object.

فيظن الناس أن Java مررت الـ Object.

لكن الحقيقة:

Java مررت نسخة من الـ Reference.

والنسختان تشيران لنفس الـ Object.

---

# مقارنة

|الحالة|ماذا يُنسخ؟|هل يتأثر الأصل؟|
|---|---|---|
|Primitive|القيمة نفسها|❌ لا|
|Object|نسخة من الـ Reference|✅ يمكن تعديل الـ Object، لكن لا يمكن تغيير Reference المتغير الأصلي|

---

# Real-world Use Cases

## 1. Updating an Entity

```java
public void updateUser(User user) {
    user.setName("Ali");
}
```

سيتم تحديث نفس الـ User Object.

---

## 2. Sorting a List

```java
Collections.sort(list);
```

الميثود لا ترجع List جديدة.

بل تعدل نفس الـ List.

لأن جميع الـ References تشير لنفس الـ Object.

---

## 3. DTO Validation

```java
validate(dto);
```

الـ Validator يمكنه تعديل الـ DTO نفسه.

---

## 4. Spring Services

```java
userService.activate(user);
```

الميثود تعدل الـ User Object مباشرة.

---

## 5. JPA Entity

```java
user.setName("Ahmed");
```

ثم:

```java
userRepository.save(user);
```

هو نفس الـ Entity الموجود في الذاكرة.

---

# Common Interview Mistakes

❌ **"Objects are passed by reference."**

خطأ.

---

❌ **"Primitive is pass-by-value and Objects are pass-by-reference."**

خطأ.

---

❌ **"Java supports both pass-by-value and pass-by-reference."**

خطأ.

---

# Interview Answer (30 Seconds)

> **Java always uses pass-by-value. For primitive types, the actual value is copied, so changes inside the method do not affect the original variable. For objects, Java copies the reference value (memory address), not the object itself. As a result, both references point to the same object, so modifying the object's state is visible outside the method. However, reassigning the reference inside the method does not affect the original reference.**

---

# احفظ الجملة دي لأنها بتتقال كتير في الإنترفيو

> **Java never passes objects by reference. It always passes a copy of the value. For objects, that value happens to be the reference (memory address), not the object itself.**