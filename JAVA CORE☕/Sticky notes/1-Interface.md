interface fields Always => public ,static and final 


## 1. Fields in an Interface

أي field بتكتبه جوه الـ `interface` بيكون تلقائيًا:

- `public`
    
- `static`
    
- `final`
    

يعني لازم يكون constant، ومينفعش تغيّر قيمته بعد initialization.

Java

```
interface Payment {
    int MAX_AMOUNT = 1000;
}
```

الـ field ده بالضبط زي:

Java

```
public static final int MAX_AMOUNT = 1000;
```

وبالتالي:

Java

```
Payment.MAX_AMOUNT = 2000; // Compilation Error
```

## 2. What else can an Interface contain?

|Member|Allowed?|
|---|---|
|`public static final` fields|Yes|
|Abstract methods|Yes|
|`default` methods|Yes|
|`static` methods|Yes|
|`private` methods (Java 9+)|Yes|
|Constructors|No|
|Instance fields|No|

### Example

Java

```
interface Animal {
    int LEGS = 4; // public static final

    void sound(); // public abstract

    default void sleep() {
        System.out.println("Sleeping");
    }

    static void info() {
        System.out.println("Animal interface");
    }

    private void helper() {
        System.out.println("Helper");
    }
}
```

لاحظ إن `private` methods اتضافت من Java 9، وبتُستخدم للمساعدة في تنفيذ `default` و`static` methods داخل الـ interface.

## 3. أهم نقطة للـ Interviews

ليه الـ interface مش بتسمح بـ instance fields؟

لأنها مش بتحتفظ بـ state خاصة بكل object زي الـ class. الـ fields فيها constants مشتركة على مستوى الـ interface.

احفظ دي:

- Interface fields → `public static final` implicitly.
    
- Interface abstract methods → `public abstract` implicitly.
    
- Interface ممكن تحتوي على methods لها implementation باستخدام `default` و`static` و`private` وفقًا لإصدار Java.
    
- Interface cannot have constructors or instance fields.

```java
interface Shape 
{
Integer legs = 4 ; //by default its  public ,final and abstract

void Big(); => //public abstract 

default void unregular()
{
System.out.println("Unregular shape");
}
static void regular()
{
System.out.println("regular shape");
}

private void helper()
{
System.out.println("Helper");
}
}
```