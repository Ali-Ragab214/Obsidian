ببساطة جدًا:
> **A Thread is the smallest unit of execution inside a process. Multiple threads share the same memory (Heap) but each thread has its own Stack.**

**الـ Thread** هو **مسار تنفيذ (Path of Execution)** داخل البرنامج.

يعني بدل ما البرنامج ينفذ حاجة واحدة في كل مرة، تقدر تخلي فيه أكتر من Thread يشتغلوا في نفس الوقت (أو بشكل متوازي حسب عدد الـ CPU Cores).

مثال:

تخيل عندك برنامج بيعمل حاجتين:

- يحمل ملف من الإنترنت.
    
- يعرض واجهة المستخدم.
    

لو عندك **Thread واحد**:

```
تحميل الملف ⏳⏳⏳
بعدها يعرض الواجهة
```

الواجهة هتفضل معلقة لحد ما التحميل يخلص.

أما لو عندك **Threadين**:

```
Thread 1 → يحمل الملف
Thread 2 → يشغل الواجهة
```

الاتنين بيشتغلوا مع بعض، فالبرنامج يفضل سريع ومستجيب.

### في Java

أي برنامج Java بيبدأ بـ **Thread واحد** اسمه **Main Thread**:

```java
public static void main(String[] args) {
    System.out.println("Hello");
}
```

الكود ده بيتنفذ على الـ **Main Thread**.

ولو أنت أنشأت Thread جديد، يبقى عندك أكتر من Thread شغالين في نفس البرنامج.

**احفظ الجملة دي للانترفيو:**

دي كل الفكرة في البداية. بعدها نبدأ نتكلم عن **إزاي ننشئ Thread في Java** و**ليه كل Thread ليه Stack خاص بيه**.