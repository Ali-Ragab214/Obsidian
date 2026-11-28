بص، سؤالك ممتاز، وإنت كده بتلمس نقطة مهمة جدًا في Java. خلينا نفصل بين حاجتين.

**أولًا: هل عدم وجود** `@Override` **معناه إن الميثود جديدة؟**

لا. `@Override` مجرد Annotation بتساعد الـ Compiler يتأكد إنك فعلًا بتعمل Override لميثود موروثة.

يعني الكود ده:

```java
class Base {
    void print(Object o) {
        System.out.println("Base(Object)");
    }
}

class Derived extends Base {
    @Override
    void print(Object o) {
        System.out.println("Derived(Object)");
    }
}
```

هو نفس الـ Override حتى لو شيلنا `@Override`.

**ثانيًا: إزاي بنقول Overloading وهما في كلاسين مختلفين؟**

==🔴لأن الـ Overloading مش شرط يحصل داخل نفس الكلاس فقط!==

ممكن يحصل بين Parent و Child طالما الميثودز ليها نفس الاسم لكن Parameters مختلفة.

بص على المثال:

```java
class Base {
    void print(Object o) {
        System.out.println("Base(Object)");
    }
}

class Derived extends Base {
    void print(String s) {
        System.out.println("Derived(String)");
    }
}
```

هنا حصل إيه؟

- `Base` عنده `print(Object)`.
    
- `Derived` ورث الميثود دي من `Base`.
    
- `Derived` أضاف ميثود جديدة `print(String)`.
    

بالتالي `Derived` عنده دلوقتي ميثودين:

```
print(Object)
print(String)
```

وده اسمه **Method Overloading**.

لاحظ إن الـ Overloading ممكن يحصل نتيجة الوراثة، مش لازم تكون الميثودز مكتوبة في نفس الكلاس.

**ثالثًا: نرجع للسؤال الأساسي**

```
Base b = new Derived();
b.print("hello");
```

**Compile-time**

الـ Reference Type هو `Base`.

الـ Compiler شايف `print(Object)` فقط.

**Runtime**

الـ Actual Object هو `Derived`.

لكن مفيش Override لـ `print(Object)` في `Derived`.

بالتالي التنفيذ هيكون من `Base`.

Output

```
Base(Object)
```

**خد القاعدة دي للإنترفيو:**

- Overloading: نفس اسم الميثود، Parameters مختلفة، وممكن يحصل بين Parent و Child.
    
- Overriding: الـ Child يعيد تعريف ميثود موروثة بنفس الـ Signature.
    

عايز أتأكد إن الفكرة وصلت، فكر في السؤال ده:

لو عملنا:

```
Derived d = new Derived();
d.print("hello");
```

تفتكر الـ Output هيكون `Base(Object)` ولا `Derived(String)`؟ وليه؟