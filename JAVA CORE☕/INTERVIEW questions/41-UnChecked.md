تمام، بص مثال بسيط تقدر تجربه بنفسك:

```java
public class Main {
    public static void main(String[] args) {
        String name = null;
        System.out.println("طول الاسم: " + name.length()); // هنا هيضرب
    }
}
```

**إيه اللي هيحصل:**

1. الكود ده **هيعمل compile عادي** من غير أي مشاكل، لأن `NullPointerException` unchecked.
2. لما تدوس **Run**، هيشتغل عادي لحد ما يوصل لسطر `name.length()`.
3. وقتها بس هيرمي:

```
Exception in thread "main" java.lang.NullPointerException:
    Cannot invoke "String.length()" because "name" is null
```

يعني هنا الكود اشتغل فعلاً (run) وبعدين ضرب، مش زي مثال الـ `IOException` اللي كان هيرفض يترجم من الأساس.

**مثال تاني لو حابب تقارن بالـ ArrayIndexOutOfBoundsException:**

```java
int[] numbers = {1, 2, 3};
System.out.println(numbers[5]); // مفيش index رقم 5
```

دي كمان unchecked، هتترجم عادي وتضرب وقت الـ run بس.

جرب الأول واحد وشوف الـ stack trace بنفسك، هتفهم الفرق بصورة أوضح.