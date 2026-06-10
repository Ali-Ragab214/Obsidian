السؤال ده من أسئلة الـ Java المتقدمة، وبيختبر فهمك لسلوك الـ Exception Handling لما يحصل Exception في أكثر من مكان.

## السؤال

> How does exception handling work when an exception is thrown in both `catch` and `finally`? (Exception Overriding)
> 
> What happens to the missed exception and how can we get it?

---

# الإجابة المختصرة في الإنترفيو

إذا تم رمي (`throw`) Exception داخل `catch` ثم تم رمي Exception آخر داخل `finally`، فإن الـ Exception الموجود في `finally` هو الذي سيتم إرساله للخارج، بينما الـ Exception الأول سيتم فقده (overridden).

من Java 7 تم تقديم مفهوم **Suppressed Exceptions** مع `try-with-resources` للاحتفاظ بالاستثناءات المفقودة، ويمكن الوصول إليها باستخدام:

```java
getSuppressed()
```

---

# مثال عملي

```java
public class Main {
    public static void main(String[] args) {

        try {
            throw new RuntimeException("Exception from try");
        }
        catch (Exception e) {
            throw new RuntimeException("Exception from catch");
        }
        finally {
            throw new RuntimeException("Exception from finally");
        }
    }
}
```

---

## ماذا سيحدث؟

1. يدخل إلى `try`
    
2. يتم رمي Exception
    
3. ينتقل إلى `catch`
    
4. يتم رمي Exception جديد من `catch`
    
5. قبل الخروج يتم تنفيذ `finally`
    
6. `finally` يرمي Exception جديد
    

النتيجة:

```text
Exception from finally
```

فقط.

---

# لماذا؟

لأن `finally` يتم تنفيذها دائمًا قبل مغادرة الـ block.

وعندما ترمي Exception جديدًا داخل `finally` فإنه يغطي (Overrides) أي Exception سابق.

---

# رسم توضيحي

```text
TRY Exception
      ↓
CATCH Exception
      ↓
FINALLY Exception
      ↓
Returned to JVM
```

الـ JVM سترى فقط:

```text
FINALLY Exception
```

---

# أين ذهب Exception الخاص بالـ catch؟

تم فقده.

ولهذا يعتبر رمي Exception من `finally` ممارسة غير مستحبة.

---

# كيف نحافظ على الـ Original Exception؟

يمكن تخزينه وإضافته كـ Cause.

```java
try {
    throw new RuntimeException("Try Exception");
}
catch (Exception e) {

    RuntimeException newEx =
        new RuntimeException("Catch Exception");

    newEx.initCause(e);

    throw newEx;
}
```

---

# Suppressed Exceptions (Java 7+)

أشهر حالة تحدث في `try-with-resources`.

مثال:

```java
class MyResource implements AutoCloseable {

    @Override
    public void close() {
        throw new RuntimeException("Close Exception");
    }
}
```

```java
try (MyResource resource = new MyResource()) {

    throw new RuntimeException("Main Exception");

}
catch (Exception e) {

    System.out.println(e.getMessage());

    for (Throwable t : e.getSuppressed()) {
        System.out.println("Suppressed: " + t.getMessage());
    }
}
```

---

## الناتج

```text
Main Exception
Suppressed: Close Exception
```

---

# لماذا تم الاحتفاظ بالاستثناء الثاني هنا؟

لأن Java Team أدركت مشكلة فقدان Exceptions عند إغلاق الموارد.

لذلك:

- الـ Main Exception يصبح الأساسي.
    
- Exceptions الناتجة من `close()` تصبح Suppressed.
    

---

# كيف أحصل على الـ Suppressed Exceptions؟

```java
Throwable[] suppressed = exception.getSuppressed();
```

مثال:

```java
for(Throwable ex : exception.getSuppressed()) {
    System.out.println(ex.getMessage());
}
```

---

# إجابة احترافية للإنترفيو

> When exceptions are thrown from both `catch` and `finally`, the exception thrown from the `finally` block overrides the previous exception because `finally` executes before control leaves the try-catch structure. As a result, the original exception is lost.
> 
> In Java 7+, suppressed exceptions were introduced, mainly for `try-with-resources`. We can retrieve suppressed exceptions using `Throwable.getSuppressed()`. To preserve exceptions manually, we can also chain them using `initCause()` or by passing the original exception as the cause to a new exception.

دي إجابة مستوى Mid/Senior Java Developer وغالبًا كافية جدًا لأي مقابلة Java Core متقدمة.