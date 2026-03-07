

---

## شرح **`transient`** في Java

### 1️⃣ تعريف

- `transient` keyword تُستخدم مع **الحقول (fields)** داخل الكلاس.
    
- الهدف: منع **هذا الحقل من أن يُخزن عند Serialization**.
    
- Serialization = تحويل الكائن (object) إلى **stream من البايتات** لحفظه أو نقله.
    

---

### 2️⃣ مثال عملي

```java
import java.io.*;

class User implements Serializable {
    String name;
    transient String password; // مش هيتسجل

    User(String name, String password) {
        this.name = name;
        this.password = password;
    }
}

public class Test {
    public static void main(String[] args) throws Exception {
        User u = new User("Ali", "1234");

        // Serialization
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.ser"));
        oos.writeObject(u);
        oos.close();

        // Deserialization
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.ser"));
        User u2 = (User) ois.readObject();
        ois.close();

        System.out.println(u2.name);     // Ali
        System.out.println(u2.password); // null
    }
}
```

- لاحظ أن `password` أصبح **null** بعد deserialization
    
- ده لأن `transient` منع تخزينه أثناء Serialization.
    

---

### 3️⃣ ليه باقي الاختيارات غلط؟

❌ A) GC → غلط، transient ما ليهاش علاقة بالـ garbage collection  
❌ C) Inheritance → غلط، transient مش بتمنع الوراثة  
❌ D) Override → غلط، transient مش مرتبطة بالميثودات

---

لو تحب أعمل لك **جدول سريع لكل Java keywords اللي ليها علاقة بالـ Serialization** مع مثال لكل واحد، هيبقى مرجع ممتاز للانترفيو.

هل أعمله لك؟