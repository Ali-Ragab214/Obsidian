تمام، بص مثال بسيط لملف مش موجود:

```java
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        FileReader file = new FileReader("test.txt"); // ملف مش موجود
    }
}
```

**جرب الكود ده زي ما هو (من غير throws ولا try-catch)، هتلاقي إن الـ IDE مش هيسيبك تشغله أصلاً.** هيديك compile error زي كده:

```
error: unreported exception FileNotFoundException; must be caught or declared to be thrown
        FileReader file = new FileReader("test.txt");
                          ^
1 error
```

**علشان الكود يترجم، لازم واحد من الاتنين:**

**الطريقة الأولى - throws:**

```java
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        FileReader file = new FileReader("test.txt");
    }
}
```

**الطريقة التانية - try-catch:**

```java
import java.io.FileReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            FileReader file = new FileReader("test.txt");
        } catch (IOException e) {
            System.out.println("الملف مش موجود: " + e.getMessage());
        }
    }
}
```

جرب الكود الأول الخام (من غير throws ولا try-catch) وشوف الـ compile error بعينك، بعدين طبق أي طريقة من الاتنين وشغله.