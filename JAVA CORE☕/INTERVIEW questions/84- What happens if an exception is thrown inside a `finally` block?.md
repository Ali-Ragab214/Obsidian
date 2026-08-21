آه، دي نقطة مهمة جدًا في الـ Java وبتتسأل في الـ interviews. خلينا نفهمها ببساطة.


لو حصل **Exception جوه `finally`**، فالـ exception اللي حصل داخل `finally` ممكن **يعمل override / suppress للـ exception الأصلي** اللي حصل في `try` أو `catch`.

يعني باختصار:

> **An exception thrown from `finally` can replace the exception thrown from `try` or `catch`.**

---

## 1. Exception في `try` + Exception في `finally` ❌

```java
public class Main {

    public static void main(String[] args) {

        try {
            throw new RuntimeException("Exception from TRY");
        } finally {
            throw new RuntimeException("Exception from FINALLY");
        }
    }
}
```

إيه اللي حصل؟

```text
try
 ↓
Exception: "Exception from TRY"
 ↓
finally
 ↓
Exception: "Exception from FINALLY"
```

اللي هتشوفه في الآخر:

```text
Exception in thread "main"
java.lang.RuntimeException: Exception from FINALLY
```

يعني الـ exception بتاع `finally` هو اللي ظهر.

### طب والـ exception بتاع `try`؟

اترمى بالفعل، لكن الـ exception بتاع `finally` **غطّى عليه**.

---

# 2. حتى لو عندك `catch`

شوف المثال ده:

```java
try {
    throw new RuntimeException("TRY");
} catch (Exception e) {
    System.out.println("Caught TRY");
    throw new RuntimeException("CATCH");
} finally {
    throw new RuntimeException("FINALLY");
}
```

الترتيب:

```text
try
 ↓
Exception TRY
 ↓
catch
 ↓
" Caught TRY "
 ↓
Exception CATCH
 ↓
finally
 ↓
Exception FINALLY
```

النتيجة النهائية:

```text
RuntimeException: FINALLY
```

لأن `finally` آخر حاجة حصلت، والـ exception بتاعها حل محل الـ exception اللي كان خارج من `catch`.

---

# 3. طب لو `finally` مفيهاش Exception؟

طبيعي جدًا:

```java
try {
    throw new RuntimeException("TRY");
} finally {
    System.out.println("Cleanup");
}
```

الـ `finally` تتنفذ:

```text
Cleanup
```

وبعدها الـ original exception يكمل طريقه:

```text
RuntimeException: TRY
```

وده هو الاستخدام الطبيعي للـ `finally`:

```java
try {
    // business logic
} finally {
    // cleanup
}
```

مثلاً:

```java
try {
    connection.open();
} finally {
    connection.close();
}
```

---

# ⚠️ نقطة مهمة جدًا في الـ Interview

لو سألك:

**What happens if finally throws an exception?**

قول:

> **If the finally block throws an exception, that exception can replace the exception thrown from the try or catch block, potentially hiding the original exception.**

---

## طب نعمل إيه؟

❌ متعملش كده:

```java
finally {
    throw new RuntimeException("Something went wrong");
}
```

خصوصًا لو عندك exception أصلي مهم.

الأفضل إن `finally` تعمل **cleanup** فقط:

```java
finally {
    connection.close();
}
```

وفي Java الحديثة، لو بتتعامل مع resources زي:

- Files
    
- Database connections
    
- Streams
    

استخدم **try-with-resources** غالبًا، لأنه بيتعامل مع الـ exceptions أثناء إغلاق الـ resources بشكل أفضل.

### احفظها كقاعدة 🧠

```text
try exception
      ↓
   finally
      ↓
finally exception
      ↓
can hide/replace original exception
```

**أهم كلمة تحفظها:**

> **Finally can override the original exception.**