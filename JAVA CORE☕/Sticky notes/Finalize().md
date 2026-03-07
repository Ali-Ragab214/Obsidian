
الإجابة الصحيحة هي:

**C) Before GC removes object ✅**

---

## شرح `finalize()` في Java

### 1️⃣ تعريف

`finalize()` هي method موجودة في **`java.lang.Object`**، وهدفها السماح للكائن بتحرير الموارد أو عمل cleanup **قبل أن يقوم Garbage Collector بحذف الكائن**.

---

### 2️⃣ كيف ومتى تُستدعى؟

- لا يمكن التحكم متى تُستدعى بالضبط.
    
- **يتم استدعاؤها مرة واحدة على الأكثر لكل كائن** قبل أن يتم جمعه من الذاكرة.
    
- مثال:
    

```java
class MyClass {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Cleaning up before GC");
    }
}

public class Test {
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        obj = null;  // eligible for GC
        System.gc(); // GC ممكن يشغل finalize()
    }
}
```

> ملاحظة: حتى لو استدعيت `System.gc()`، مش مضمون أن GC هيشتغل فورًا أو finalize() هتتستدعى مباشرة.

---

### 3️⃣ ليه باقي الاختيارات غلط؟

❌ A) Before object creation → غلط، object لسه ما اتعملش  
❌ B) During compilation → غلط، دي runtime operation  
❌ D) Always immediately → غلط، ممكن GC يأجل التنفيذ حسب الحاجة

---

لو تحب أعمل لك **ملخص سريع لكل lifecycle methods المرتبطة بالكائنات في Java** هيبقى سهل تتذكرهم في الانترفيو.

هل أعمله لك؟