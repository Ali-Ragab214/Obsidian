
## ArrayList vs LinkedList في Java

---

## الفكرة الأساسية

```
ArrayList   = بيانات في صناديق متجاورة في الميموري
LinkedList  = بيانات في صناديق كل واحدة فيها عنوان اللي بعدها
```

---

## ArrayList

تخيل إنك عندك أرفف في مكتبة، كل كتاب في مكانه المرقم.

```
Index:  [0]      [1]      [2]      [3]
      | "Ali" | "Omar" | "Sara" | "Mona" |
        ↑ كلهم في الميموري جنب بعض
```

```java
ArrayList<String> list = new ArrayList<>();
list.add("Ali");
list.add("Omar");
list.add("Sara");
list.add("Mona");

// الوصول بالـ index سريع جداً
System.out.println(list.get(2)); // Sara ← مباشرة!
```

**الحذف أو الإضافة في النص بطيء ليه؟**

```
عايز تضيف "Ziad" في Index 1؟

قبل:  | Ali | Omar | Sara | Mona |
       [0]   [1]    [2]    [3]

لازم تزحزح كل حاجة:
       | Ali |      | Omar | Sara | Mona |
       [0]   [1]    [2]    [3]    [4]

بعدين تحط:
       | Ali | Ziad | Omar | Sara | Mona |
       [0]   [1]    [2]    [3]    [4]

كل ما الـ list كبرت، كل ما الزحزحة أبطأ!
```

---

## LinkedList

تخيل قطار، كل عربية فيها الراكب وعارفة العربية اللي بعدها.

```
[Ali | →] → [Omar | →] → [Sara | →] → [Mona | null]
  ↑
 head
```

```java
LinkedList<String> list = new LinkedList<>();
list.add("Ali");
list.add("Omar");
list.add("Sara");
list.add("Mona");

// إضافة في الأول أو الآخر سريعة جداً
list.addFirst("Ziad");
list.addLast("Nour");
```

**الإضافة في النص سريعة ليه؟**

```
عايز تضيف "Ziad" بين Ali و Omar؟

قبل:
[Ali | →] ────────────→ [Omar | →] → [Sara | →]

بس غير الـ pointer:
[Ali | →] → [Ziad | →] → [Omar | →] → [Sara | →]

مفيش زحزحة خالص! بس غير عنوانين بس ✅
```

**الوصول بالـ index بطيء ليه؟**

```
عايز العنصر رقم 3؟

ArrayList: يروح مباشرة للـ index → سريع جداً ⚡
LinkedList: يبدأ من الأول ويمشي واحدة واحدة → بطيء 🐢

[Ali] → [Omar] → [Sara] → [Mona]
  1  →    2   →    3   →    4
```

---

## مقارنة عملية بالكود

```java
import java.util.ArrayList;
import java.util.LinkedList;

public class Main {
    public static void main(String[] args) {

        // ===== ArrayList =====
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Ali");
        arrayList.add("Omar");
        arrayList.add("Sara");

        // ✅ سريع جداً - O(1)
        System.out.println(arrayList.get(1)); // Omar

        // ❌ بطيء - O(n) بيزحزح كل العناصر
        arrayList.add(0, "Ziad");

        // ❌ بطيء - O(n) بيزحزح كل العناصر
        arrayList.remove(0);


        // ===== LinkedList =====
        LinkedList<String> linkedList = new LinkedList<>();
        linkedList.add("Ali");
        linkedList.add("Omar");
        linkedList.add("Sara");

        // ✅ سريع جداً - O(1)
        linkedList.addFirst("Ziad");
        linkedList.addLast("Nour");
        linkedList.removeFirst();
        linkedList.removeLast();

        // ❌ بطيء - O(n) بيمشي من الأول
        System.out.println(linkedList.get(2)); // Sara
    }
}
```

---

## جدول المقارنة

```
العملية              ArrayList       LinkedList
─────────────────────────────────────────────
get(index)           ✅ O(1)         ❌ O(n)
add(آخر القائمة)    ✅ O(1)         ✅ O(1)
add(أول القائمة)    ❌ O(n)         ✅ O(1)
add(في النص)        ❌ O(n)         ✅ O(1)*
remove(أول)          ❌ O(n)         ✅ O(1)
الميموري             ✅ أقل           ❌ أكتر (بيحفظ pointers)
```

`* بعد ما توصلها`

---

## امتى تستخدم إيه؟

```
ArrayList  → لو هتعمل get كتير وبتغير في الآخر بس
             مثال: قائمة منتجات، بحث، عرض بيانات

LinkedList → لو هتضيف وتحذف كتير من الأول أو النص
             مثال: queue، undo/redo، playlist
```

---

## مثال من الحياة

```java
// ✅ ArrayList → قائمة طلاب (بتعرض وتبحث كتير)
ArrayList<String> students = new ArrayList<>();
students.add("Ali");
students.add("Omar");
System.out.println(students.get(0)); // سريع

// ✅ LinkedList → قائمة انتظار (الأول يدخل الأول)
LinkedList<String> queue = new LinkedList<>();
queue.addLast("Ali");   // دخل الطابور
queue.addLast("Omar");  // دخل بعده
queue.removeFirst();    // Ali اتخدم وخرج
```

**الخلاصة في جملة واحدة:**

- `ArrayList` → إنت بتقرأ كتير 📖
- `LinkedList` → إنت بتضيف وتحذف كتير ✂️