
---

# أولاً: Singleton بيتصنف تحت ايه؟

Singleton واحد من **Creational Design Patterns**  
يعني Pattern مسؤول عن **إنشاء الـ Objects**.

---

# فكرة Singleton في جملة واحدة

> كلاس مسموح منه **نسخة واحدة فقط** في البرنامج كله.

نسخة واحدة بس ❗

---

# ليه نحتاج Singleton أصلاً؟

في حاجات مينفعش يكون منها أكتر من نسخة.

أمثلة حقيقية:

- Database Connection 🗄️
    
- Logger 📝
    
- Cache System ⚡
    
- App Configuration ⚙️
    

تخيل لو كل كلاس فتح Connection لوحده 😵‍💫  
السيرفر هيقع.

إحنا عايزين **نسخة واحدة مشتركة** لكل البرنامج.

---

# المشكلة بدون Singleton

الكلاس العادي:

```java
class Database {
}
```

نقدر نعمل منه نسخ كتير:

```java
Database d1 = new Database();
Database d2 = new Database();
Database d3 = new Database();
```

❌ كده عندك 3 Database Connections!

---

# هدف Singleton

عايزين:

1. نمنع إنشاء أكتر من Object
    
2. نوفر طريقة للوصول للنسخة الوحيدة
    

يعني:

- منع الـ new من الخارج
    
- توفير instance جاهز
    

---

# إزاي نحقق ده؟ (الفكرة الأساسية)

Singleton يعتمد على 3 خطوات مهمة جداً:

1️⃣ Constructor يكون **private**  
عشان محدش يعمل new من بره.

2️⃣ نحتفظ بنسخة واحدة static من الكلاس.

3️⃣ نوفر method ترجع النسخة الوحيدة.

---

# نكتب أول Singleton خطوة خطوة 👨‍💻

## الخطوة 1: نعمل الكلاس

```java
public class Database {
}
```

---

## الخطوة 2: نمنع إنشاء Objects من الخارج

نخلي الـ constructor private:

```java
public class Database {

    private Database() {
        System.out.println("Database created");
    }
}
```

دلوقتي مينفعش تعمل:

```java
new Database(); ❌
```

---

## الخطوة 3: نخزن النسخة الوحيدة

```java
private static Database instance;
```

ليه static؟  
عشان تبقى موجودة مرة واحدة فقط في الذاكرة.

---

## الخطوة 4: نعمل method ترجع النسخة الوحيدة

```java
public static Database getInstance() {

    if(instance == null) {
        instance = new Database();
    }

    return instance;
}
```

كده خلصنا Singleton 🎉

---

# الكود كامل

```java
public class Database {

    private static Database instance;

    private Database() {
        System.out.println("Database created");
    }

    public static Database getInstance() {
        if(instance == null) {
            instance = new Database();
        }
        return instance;
    }
}
```

ده اسمه:  
👉 Lazy Initialization Singleton

يعني يتعمل عند أول استخدام.

---

# نجربه في main

```java
public class Main {
    public static void main(String[] args) {

        Database db1 = Database.getInstance();
        Database db2 = Database.getInstance();
        Database db3 = Database.getInstance();

        System.out.println(db1 == db2);
        System.out.println(db2 == db3);
    }
}
```

---

# الناتج المتوقع

```
Database created
true
true
```

ليه اتطبع مرة واحدة بس؟ 😍  
لأن تم إنشاء Object واحد فقط.

والمتغيرات الثلاثة شايلين نفس النسخة.

---

# نشرح كل سطر ببساطة

## private static Database instance;

متغير بيخزن النسخة الوحيدة.

---

## private Database()

منعنا أي كلاس يعمل new.

---

## getInstance()

البوابة الوحيدة للحصول على النسخة.

---

## if(instance == null)

لو أول مرة → أنشئ Object  
لو مش أول مرة → رجّع الموجود.

---

# مشكلة النسخة دي ⚠️ (مهم جداً)

النسخة دي **مش Thread Safe**

لو عندك Threads كتير ممكن يحصل:

- Thread1 يدخل
    
- Thread2 يدخل
    
- الاتنين يعملوا new 😱
    

ويبقى عندك نسختين.

---

# الحل Thread Safe Singleton 🔥

نضيف synchronized:

```java
public static synchronized Database getInstance() {
    if(instance == null) {
        instance = new Database();
    }
    return instance;
}
```

كده Thread واحد بس يدخل.

---

# أفضل نسخة في Java (الأشهر) ⭐
اللي عملناه قبل كدا اسمه Lazy SIngleton لان ال object  مش بيتعمل غير لما المستخدم يعمله 
لكن دا بيتعمل اول ما المستخدم يشتغل 
# 🎯 ليه نستخدم الطريقة دي؟

لأنها تحل أكبر مشكلة في Singleton 👇

## ❌ مشكلة النسخة القديمة

مش Thread Safe.

لو 2 threads دخلوا `getInstance()` في نفس اللحظة  
ممكن يعملوا object مرتين
#  بس دا عيبه انوا بيعمل الاوبجكت حتي لو مش محتاجه 
# 💡 نستخدم مين امتى؟

- لو الـ object خفيف → استخدم **Eager** ✔️
- لو تقيل وبيستهلك موارد → استخدم **Lazy Thread Safe** (وهنشرحه بعدين)


Eager Initialization:

```java
public class Database {

    private static final Database instance = new Database();

    private Database() {}

    public static Database getInstance() {
        return instance;
    }
}
```

النسخة تتعمل مرة واحدة عند تشغيل البرنامج.

✔️ Thread safe  
✔️ بسيطة  
✔️ سريعة

---

# إمتى أستخدم Singleton؟

لو عندك حاجة:

- Shared في كل التطبيق
    
- تقيلة في الإنشاء
    
- لازم نسخة واحدة
    

زي:

- ConfigManager
    
- Logger
    
- Cache
    
- DB Connection Pool
    

---

# ملخص قوي جداً 💪

Singleton يعني:

- Object واحد فقط
    
- Constructor private
    
- Instance static
    
- Method ترجع نفس النسخة دائماً
    

---

لو تحب بعد كده نشرح الفرق بين Singleton و Static class أو أخطاء Singleton في الانترفيوهات قولّي 👍