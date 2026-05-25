## Association, Aggregation & Composition

> "Association is a general relationship where objects know about each other but are independent. Aggregation is a weak Has-A relationship — the child can exist independently. Composition is a strong Has-A relationship — the child's lifecycle depends entirely on the parent. The key difference in code: in Composition the child is created **inside** the parent, in Aggregation it's created **outside** and passed in."
---

## 📌 الفكرة الأساسية

الثلاثة دول بيوصفوا **علاقة بين Classes** — بس بيختلفوا في درجة الارتباط والـ Ownership.

```
Association  → "بيعرف بعض"       (أضعف علاقة)
Aggregation  → "عنده"            (علاقة متوسطة)
Composition  → "جزء منه"         (أقوى علاقة)
```

---

## 1️⃣ Association — "بيعرف بعض"

### التعريف:

علاقة بين Class وتاني من غير ما حد يمتلك التاني. كلاهما **مستقلان** — لو واحد اتحذف التاني فاضل موجود.

### في الحياة:

> الدكتور والمريض — الدكتور بيعرف المريض والمريض بيعرف الدكتور، بس لو الدكتور اتقاعد، المريض فاضل موجود والعكس.

### الكود:

```java
class Doctor {
    String name;

    Doctor(String name) {
        this.name = name;
    }

    void treat(Patient patient) {
        System.out.println(name + " is treating " + patient.name);
    }
}

class Patient {
    String name;

    Patient(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Doctor d = new Doctor("Dr. Ahmed");
        Patient p = new Patient("Ali");

        d.treat(p); // Dr. Ahmed is treating Ali
    }
}
```

### الخصائص:

- مفيش Ownership
- الاتنين مستقلين تماماً
- العلاقة ممكن تكون في الاتجاهين (Bidirectional)

---

## 2️⃣ Aggregation — "عنده" (Has-A weak)

### التعريف:

علاقة **Has-A** — الـ Parent "عنده" Child، بس الـ Child يقدر يعيش من غير الـ Parent. الـ Child مش بيتعمل جوّا الـ Parent — بييجي من برّا.

### في الحياة:

> القسم والأستاذ — القسم "عنده" أساتذة، بس لو القسم اتقفل، الأستاذ فاضل موجود وممكن ينقل لقسم تاني.

### الكود:

```java
class Professor {
    String name;

    Professor(String name) {
        this.name = name;
    }
}

class Department {
    String deptName;
    List<Professor> professors;  // ← الـ Professor جاي من برّا

    Department(String deptName, List<Professor> professors) {
        this.deptName = deptName;
        this.professors = professors;  // مش بنعمله new جوّا
    }

    void showProfessors() {
        for (Professor p : professors) {
            System.out.println(deptName + " has: " + p.name);
        }
    }
}

public class Main {
    public static void main(String[] args) {

        // الـ Professor اتعمل برّا الـ Department
        Professor p1 = new Professor("Dr. Ahmed");
        Professor p2 = new Professor("Dr. Sara");

        List<Professor> list = new ArrayList<>();
        list.add(p1);
        list.add(p2);

        Department dept = new Department("CS", list);
        dept.showProfessors();

        // لو Department اتحذف → p1 و p2 فاضلين موجودين ✅
    }
}
```

### الخصائص:

- الـ Parent "عنده" الـ Child (Has-A)
- الـ Child بيتعمل **برّا** الـ Parent
- الـ Child يقدر يعيش من غير الـ Parent
- **Weak Ownership**

---

## 3️⃣ Composition — "جزء منه" (Has-A strong)

### التعريف:

علاقة **Has-A قوية** — الـ Child جزء أساسي من الـ Parent. لو الـ Parent اتحذف → الـ Child بيتحذف معاه تلقائياً. الـ Child بيتعمل **جوّا** الـ Parent.

### في الحياة:

> البيت والأوضة — الأوضة جزء من البيت، لو البيت اتهدم، الأوضة اتهدت معاه.

### الكود:

```java
class Room {
    String roomType;

    Room(String roomType) {
        this.roomType = roomType;
        System.out.println(roomType + " created");
    }
}

class House {
    String address;
    List<Room> rooms;  // ← الـ Room جزء من الـ House

    House(String address) {
        this.address = address;
        // الـ Rooms بتتعمل جوّا الـ House
        rooms = new ArrayList<>();
        rooms.add(new Room("Bedroom"));
        rooms.add(new Room("Kitchen"));
        rooms.add(new Room("Bathroom"));
    }

    void showRooms() {
        for (Room r : rooms) {
            System.out.println(address + " has: " + r.roomType);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        House h = new House("123 Main St");
        h.showRooms();

        // لو h اتحذف → الـ Rooms اتحذفت معاه ❌ مش موجودة برّا
    }
}

// Output:
// Bedroom created
// Kitchen created
// Bathroom created
// 123 Main St has: Bedroom
// 123 Main St has: Kitchen
// 123 Main St has: Bathroom
```

### الخصائص:

- الـ Child بيتعمل **جوّا** الـ Parent
- الـ Child مش موجود من غير الـ Parent
- لو الـ Parent اتحذف → الـ Child اتحذف
- **Strong Ownership**

---

## 📌 المقارنة الكاملة

||Association|Aggregation|Composition|
|---|---|---|---|
|**المعنى**|بيعرف بعض|عنده|جزء منه|
|**Ownership**|❌ مفيش|⚠️ Weak|✅ Strong|
|**الـ Child بيتعمل فين؟**|برّا|برّا|**جوّا**|
|**لو Parent اتحذف**|Child موجود|Child موجود|Child اتحذف|
|**مثال**|دكتور ومريض|قسم وأستاذ|بيت وأوضة|

---

## 📌 علامة الفرق الأساسية في الكود

```java
// Association → بييجي كـ Parameter
void treat(Patient patient) { }

// Aggregation → بييجي من برّا ويتحط في Field
Department(List<Professor> professors) {
    this.professors = professors;  // جاي من برّا
}

// Composition → بيتعمل جوّا
House() {
    rooms.add(new Room("Bedroom"));  // اتعمل جوّا ✅
}
```

---

## 📌 ملخص بصري

```
Association:
[Doctor] ←————————→ [Patient]
  بيعرفوا بعض، مستقلين

Aggregation:
[Department] ◇————→ [Professor]
  عنده أساتذة، بس الأستاذ مش جزء منه

Composition:
[House] ◆————→ [Room]
  الأوضة جزء من البيت، مش موجودة من غيره
```

---

## 💡 Interview Answer

> "Association is a general relationship where objects know about each other but are independent. Aggregation is a weak Has-A relationship — the child can exist independently. Composition is a strong Has-A relationship — the child's lifecycle depends entirely on the parent. The key difference in code: in Composition the child is created **inside** the parent, in Aggregation it's created **outside** and passed in."