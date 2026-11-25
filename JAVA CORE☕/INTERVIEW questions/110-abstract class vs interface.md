امتي استخدم دا و دا 
بص انا استخدم ال interface لما : 
-multiple inheritence
-لما ال subclasses يكون مفيش بينهم state لان ال state الموجود في ال interface بيبقا public static final ف مش هيبقا ليهم اكسيس عليه    مشترك لكن في بينهم behavior مشترك يعني مثلا زي الطائره والطائر 

الخلاصه :
استخدم الـ `Interface` لما يكون عندك مجموعة من الـ Classes محتاجة تشترك في **Contract أو Behavior** من غير ما تكون محتاجة تشترك في نفس الـ State أو Implementation.

ex
```java
interface Flyable {
    void fly();
}

class Bird implements Flyable {
    @Override
    public void fly() {
        System.out.println("Bird is flying");
    }
}

class Airplane implements Flyable {
    @Override
    public void fly() {
        System.out.println("Airplane is flying");
    }
}
```

الـ Bird والـ Airplane مفيش بينهم علاقة وراثة مباشرة، لكن الاتنين عندهم نفس الـ Behavior وهو `fly()`.

---

واستخدم ال abstract class لما يكون ال subclasses بينهم common states ,behavior and constructor  لان في الاخر الوراثه بتمثل is a relationship  
طبعا بقا فيه فروقات كتير بين ال abstract class and interface 

الخلاصه : 
استخدم الـ `Abstract Class` لما يكون عندك مجموعة من الـ Classes بينهم علاقة `IS-A`، وعايز تشارك بينهم:

- Common State (Fields).
    
- Common Behavior (Implemented Methods).
    
- Common Constructor logic.
    
- Abstract Methods لازم الـ Subclasses تعملها Override.
-


ex
```java
abstract class Animal {

    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public abstract void makeSound();
}

class Dog extends Animal {

    public Dog(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}
```


هنا الـ Dog هو Animal، والاتنين ممكن يشتركوا في الـ State والـ Behavior.