أيوه بالظبط، بس فيه **تصحيح صغير مهم** 👌

### 1. Local Variable

هو variable بتعرّفه **جوه method أو block**، وبيعيش في الـscope بتاعه.

```java
void eat() {
    int food = 10;  // Local variable
    System.out.println(food);
}
```

`food` هنا **Local Variable** لأنه متعرّف جوه `eat()`.

---

### 2. Instance Variable

ده بتعرّفه **جوه الـclass ولكن برّه الـmethods**:

```java
class Dog {

    int weight;  // Instance variable

    void eat() {
        System.out.println(weight);
    }
}
```

`weight` هنا Instance Variable.

والأهم: **كل Object من الـclass بيكون له نسخة مستقلة من الـinstance variables.**

مثلاً:

```java
Dog d1 = new Dog();
Dog d2 = new Dog();

d1.weight = 10;
d2.weight = 20;
```

عندنا:

```text
Dog class
   |
   +---- d1 Object
   |       weight = 10
   |
   +---- d2 Object
           weight = 20
```

يعني لما تقول:

> "Instance variable على مستوى الـobject"

ده **صح من ناحية إنه مرتبط بالـobject**، لكن تعريفه بيكون **على مستوى الـclass، خارج الـmethods**.

### الزتونة 🧠

|النوع|بيتعرّف فين؟|مرتبط بإيه؟|
|---|---|---|
|Local Variable|جوه method/block|بالـmethod/block|
|Instance Variable|جوه class وبره methods|بالـobject|
|Static Variable|جوه class وبره methods ومع `static`|بالـclass|

مثال يجمعهم:

```java
class Dog {

    int weight = 10;        // Instance variable
    static int count = 0;  // Static variable

    void eat() {
        int food = 5;      // Local variable
    }
}
```

==**مهم جدًا:** الـLocal Variable لازم تديه قيمة قبل ما تستخدمه، لكن الـInstance Variable Java بتديله **default value** تلقائيًا.==