


---

## شرح Enum في Java

### 1️⃣ تعريف

- الـ `enum` في Java **هو نوع خاص من الكلاسات** (special class).
    
- يُستخدم لتعريف مجموعة من الثوابت (constants) باسم واحد.
    

#### مثال:

```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

public class Test {
    public static void main(String[] args) {
        Day today = Day.MONDAY;
        System.out.println(today); // MONDAY
    }
}
```

---

### 2️⃣ حقائق مهمة عن Enum

- كل عنصر في enum هو **instance من الكلاس enum**.
    
- يمكن للـ enum أن يحتوي على:
    
    - fields
        
    - methods
        
    - constructors (private فقط)
        
- Enum يمكنه **تنفيذ Interface** لكن **لا يمكنه توريث كلاس آخر**.
    

#### مثال مع field و method:

```java
enum Color {
    RED(1), GREEN(2), BLUE(3);

    private int code;
    private Color(int code) { this.code = code; }

    public int getCode() { return code; }
}
```

- هنا كل عنصر في Color له code خاص به، و enum له methods و constructor → يدل على أنه **class كامل**.
    

---

### 3️⃣ ليه باقي الاختيارات غلط؟

❌ B) Interface → غلط، enum **يمكنه تنفيذ interface لكنه ليس interface**  
❌ C) Primitive → غلط، enum نوع مرجعي (reference type) وليس primitive  
❌ D) Annotation → غلط، enum مختلف تمامًا عن annotation

---

لو تحب أعمل لك **مخطط يوضح الفرق بين enum, class, interface, annotation** مع أمثلة سريعة، هيبقى مرجع ممتاز للانترفيو.

هل أعمله لك؟