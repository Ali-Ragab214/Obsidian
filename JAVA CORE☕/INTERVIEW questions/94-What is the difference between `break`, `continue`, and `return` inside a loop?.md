تمام، اعتبرنا في **Interview**، وأنا الـ interviewer وإنت بتجاوبني. 👨‍💻

### Question

**What is the difference between `break`, `continue`, and `return` inside a loop?**

### Interview Answer

الـ three keywords بيستخدموا للتحكم في flow بتاع الـ loop، لكن كل واحد بيعمل حاجة مختلفة:

|Keyword|What does it do?|
|---|---|
|`break`|بيخرج من الـ **loop بالكامل**|
|`continue`|بيسيب الـ **current iteration** ويبدأ الـ next iteration|
|`return`|بيخرج من الـ **entire method**، وبالتالي بيوقف الـ loop كمان|

### 1. `break`

بيوقف الـ loop بالكامل.

```java
for (int i = 1; i <= 5; i++) {

    if (i == 3) {
        break;
    }

    System.out.println(i);
}
```

Output:

```text
1
2
```

لما `i == 3` حصل `break` → **الـ loop انتهى بالكامل**.

---

### 2. `continue`

مش بيوقف الـ loop، لكنه **بيتخطى الـ current iteration** ويكمل باللي بعدها.

```java
for (int i = 1; i <= 5; i++) {

    if (i == 3) {
        continue;
    }

    System.out.println(i);
}
```

Output:

```text
1
2
4
5
```

عند `3` حصل `continue` → `3` مش اتطبعت، لكن الـ loop كمل.

---

### 3. `return`

دي أهم واحدة.

`return` **بتنهي الـ method نفسها**، مش بس الـ loop.

```java
void test() {

    for (int i = 1; i <= 5; i++) {

        if (i == 3) {
            return;
        }

        System.out.println(i);
    }

    System.out.println("After loop");
}
```

Output:

```text
1
2
```

ليه `"After loop"` مش اتطبعت؟

لأن:

```java
return;
```

خرجت من **الـ method بالكامل**.

---

### 🧠 الزتونة اللي تقولها في الانترفيو

> **`break` exits the loop, `continue` skips the current iteration, while `return` exits the entire method.**

احفظ الجملة دي، دي إجابة Interview ممتازة ومباشرة.