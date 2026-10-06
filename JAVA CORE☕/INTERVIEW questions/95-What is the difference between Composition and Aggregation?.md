آه ده توبيك مهم جدًا في **OOP و Design**، وبيتسأل كتير في الانترفيوز. خلينا نفهمه من الصفر وبطريقة تفرق بينهم في دماغك.

# Composition vs Aggregation

الاتنين بيمثلوا علاقة بين Objects اسمها:

> **HAS-A relationship**

يعني Object **يمتلك أو يحتوي** Object تاني.

مثال عام:

```text
Car HAS-A Engine
```

الـ Car عندها Engine.

لكن الفرق الأساسي هو: **قوة العلاقة بين الـ Objects والـ lifecycle بتاع الـ contained object.**

---

## 1. Composition 💪

Composition معناها علاقة **Strong HAS-A relationship**.

الـ Parent object هو اللي **بيملك حياة** الـ Child object.

يعني لو الـ Parent اتدمر، الـ Child **مش هيبقى له معنى/وجود مستقل داخل هذا النموذج**.

### مثال

```java
class Engine {
}

class Car {
    private Engine engine = new Engine();
}
```

هنا:

```text
Car
 └── Engine
```

الـ `Car` هي اللي أنشأت الـ `Engine` وبتديره.

لو الـ `Car` انتهت حياتها، فالـ `Engine` المرتبط بيها كجزء منها بينتهي كمان.

### مثال واقعي

**House → Room**

الـ Room جزء من الـ House.

```text
House
 ├── Bedroom
 ├── Kitchen
 └── Bathroom
```

الغرف كـ أجزاء من البيت مش بنعتبرها كيانات مستقلة عن البيت في التصميم ده.

---

# 2. Aggregation 🤝

Aggregation برضه **HAS-A**، لكن العلاقة **Weak**.

الـ Parent بيستخدم أو يحتوي الـ Child، لكن الـ Child **يقدر يعيش بشكل مستقل** عن الـ Parent.

### مثال

```java
class Teacher {
}

class Department {
    private List<Teacher> teachers;
}
```

هنا:

```text
Department
   |
   ├── Teacher
   ├── Teacher
   └── Teacher
```

الـ `Department` عنده Teachers.

لكن لو الـ Department اتمسح:

```text
Department ❌
```

الـ Teachers ممكن يفضلوا موجودين:

```text
Teacher ✅
Teacher ✅
```

لأن الـ Teacher مش ملك للـ Department من ناحية الـ lifecycle.

---

# الفرق الأساسي 🔥

|Composition|Aggregation|
|---|---|
|Strong relationship|Weak relationship|
|Child يعتمد على Parent|Child مستقل|
|Parent يملك lifecycle بتاع Child|Child له lifecycle مستقل|
|`Car → Engine`|`Department → Teacher`|
|الجزء جزء أساسي من الكل|مجرد علاقة احتواء/تجميع|

---

## خلي بالك من نقطة مهمة جدًا في Java

الـ **Composition والـ Aggregation مش keywords في Java**.

يعني مفيش:

```java
composition
```

أو:

```java
aggregation
```

إحنا بنستخدم **object references** ونصمم العلاقة بطريقة معينة.

مثلاً:

### Composition

```java
class Car {

    private final Engine engine;

    public Car() {
        this.engine = new Engine();
    }
}
```

الـ `Car` هو اللي أنشأ الـ `Engine`.

### Aggregation

```java
class Department {

    private List<Teacher> teachers;

    public Department(List<Teacher> teachers) {
        this.teachers = teachers;
    }
}
```

الـ `Teacher` اتعمل **بره** واتمرر للـ `Department`.

```java
Teacher t1 = new Teacher();
Teacher t2 = new Teacher();

Department department =
        new Department(List.of(t1, t2));
```

فالـ `Department` مش هو اللي خلق الـ Teachers، والـ Teachers ممكن يعيشوا بدونه.

---

# 🎯 سؤال انترفيو مشهور

**What is the difference between Composition and Aggregation?**

الإجابة المختصرة:

> **Composition is a strong HAS-A relationship where the child object's lifecycle depends on the parent, while Aggregation is a weak HAS-A relationship where the child can exist independently of the parent.**

### الزتونة بالمصري:

**Composition:**

> "إنت جزء مني، لو أنا انتهيت، إنت كمان بتنتهي كجزء مني."

**Aggregation:**

> "إنت عندي، بس لو أنا انتهيت، إنت تقدر تكمل حياتك عادي."

```java 
String x = "2";
int y = Integer.parseInt(x);
System.out.println(x+y);
```


operators => 
1-Arithmetic  : +,-,*,/,%.
2-comparison : <,>,=, == ,!=,<=
3-logical:|| , && , !

Math.random();





أيوه يا علي، **ده Chapter 5: Extra-Strength Methods** من Head First Java، وده شابتر مهم جدًا لأنه بينقلنا من مجرد كتابة Classes وObjects إلى كتابة **برنامج كامل فيه Methods + Operators + Loops + Random + Parsing + Testing**.

هنمشي فيه **Topic by Topic وبنفس ترتيب الشابتر**، وكل Topic هنعمل فيه:

- الفكرة ببساطة
- ليه بنستخدمها؟
- Syntax
- أمثلة Java
- إيه اللي بيحصل في الـ memory/Runtime لو مهم
- أسئلة Interview
- أخطاء شائعة
- وفي الآخر **الزتونة**

> ملاحظة صغيرة: الصورة اللي بعتها فيها الـ Table of Contents فقط، مش صفحات الشابتر نفسه. فهشرح لك المفاهيم والأمثلة التعليمية بالتفصيل، لكن مش هنسخ نص الكتاب حرفيًا.

## 🗺️ خطة Chapter 5

هنمشي كده:

1. **Building the Sink a Startup Game**
2. **Developing a Class**
3. **Writing Method Implementations**
4. **Writing Test Code**
5. **`checkYourself()` method**
6. **Preparing `SimpleStartupGame`**
7. **The `main()` method**
8. **Playing the Game**
9. **More about `for` loops**
10. **Enhanced `for` loop**
11. **Casting Primitives**
12. **Exercises + Solutions**

وفي الطريق هنغطي الـ concepts اللي الشابتر عايز يعلمها:  
`operators` → `loops` → `random numbers` → `String → int` → methods → return values → testing → arrays → casting.

---

# 1️⃣ Let's Build "Sink a Startup"

الكتاب بيبدأ بمشروع صغير شبيه بـ **Battleship**.

عندنا Grid:

```
   0 1 2 3 4 5 6
A  . . . . . . .
B  . . . . . . .
C  . . . . . . .
D  . . . . . . .
```