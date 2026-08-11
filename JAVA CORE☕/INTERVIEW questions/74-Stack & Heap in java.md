في Java، الـ **Stack والـ Heap هما منطقتين مختلفتين في الـ JVM**، وكل واحدة لها وظيفة مختلفة
# هما في الاساس memory areas 
# each thread has its own stack 
# heap is shared between threads 

### 1. Stack

الـ **Stack** بيُستخدم بشكل أساسي لتخزين **الـ method calls والـ local variables والـ execution context** لكل Thread.

كل ما بنستدعي Method، بيتعمل لها **Stack Frame** جديد، ولما الـ method تخلص، الـ frame بيتشال تلقائيًا.

مثال:

```java
public void calculate() {
    int x = 10;
    int y = 20;
}
```

هنا `x` و `y` موجودين داخل الـ Stack Frame الخاص بـ `calculate()`.

**خصائص الـ Stack:**

- لكل Thread **Stack خاص بيه**.
    
- سريع جدًا.
    
- الـ memory بتتحرر تلقائيًا لما الـ method تخلص.
    
- حجم الـ Stack محدود.
    
- لو استدعاءات الـ methods زادت جدًا ممكن يحصل `StackOverflowError`.
    

---

### 2. Heap

الـ **Heap** هو المكان الأساسي اللي بتتخزن فيه **Objects** والـ instance data اللي بننشئها باستخدام `new`.

مثال:

```java
User user = new User();
```

هنا:

- `user` → Reference موجود في الـ Stack.
    
- الـ `User object` نفسه → موجود في الـ Heap.
    

والـ Heap بتديره **Garbage Collector**، اللي بيشيل الـ Objects اللي مبقاش فيه References بتشير ليها.

**خصائص الـ Heap:**

- Shared بين الـ Threads.
    
- أكبر من الـ Stack عادةً.
    
- Objects بتعيش فيه لفترة أطول من Stack frames.
    
- Garbage Collector مسؤول عن إدارة الـ memory.
    
- لو الـ Heap امتلأ ممكن يحصل `OutOfMemoryError`.
    

---

### الفرق المهم جدًا في الإنترفيو

| Stack                                              | Heap                               |
| -------------------------------------------------- | ---------------------------------- |
| مرتبط بالـ Threads                                 | Shared بين الـ Threads             |
| يخزن Stack Framesو method calls  و Local Variables | يخزن Objects                       |
| أسرع                                               | أبطأ نسبيًا                        |
| Memory تُدار تلقائيًا مع انتهاء الـ method         | Garbage Collector يدير الـ Objects |
| محدود لكل Thread                                   | أكبر وأشمل                         |
| ممكن يحصل StackOverflowError                       | ممكن يحصل OutOfMemoryError         |

### 🎯 إجابة احترافية تقولها في الإنترفيو

> **"In Java, Stack and Heap are two different memory areas managed by the JVM. The Stack is used for method execution, including stack frames and local variables, and each thread has its own stack. The Heap is shared between threads and is mainly used to store objects created at runtime. Stack memory is automatically released when a method finishes, while objects in the Heap are managed by the Garbage Collector. So, the main difference is that Stack is mainly related to method execution and thread-local data, while Heap is mainly used for dynamically allocated objects."**

**نقطة مهمة:** متقولش بشكل مطلق إن _"primitive types are always stored in Stack"_؛ دي إجابة مبسطة زيادة. مكان تخزين البيانات في JVM ممكن يتأثر بالـ implementation والـ optimization. المهم في الإنترفيو تركز على **local variables/stack frames vs objects/heap**.