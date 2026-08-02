

> **"What's the difference between equals() and hashCode()? ولماذا لازم نعمل Override للاتنين مع بعض؟"**

وده من أكتر الأسئلة اللي بتكشف هل الشخص فاهم Java Collections ولا حافظ.

هشرحلك الموضوع من الصفر كأننا في انترفيو، وبعدها هنخش في HashMap عشان تفهم ليه الاتنين مرتبطين ببعض.

---

# أولاً: يعني إيه equals() ؟

ببساطة...

`equals()` مسؤولة عن الإجابة على السؤال:

> **هل الـ Object ده يساوي الـ Object التاني؟**

مثال:

```java
String s1 = new String("Ali");
String s2 = new String("Ali");
```

دلوقتي:

```java
System.out.println(s1 == s2);
```

النتيجة

```
false
```

ليه؟

لأن `==`

بتقارن الـ Reference

يعني:

```
s1 ----> Object A

s2 ----> Object B
```

رغم إن الاتنين فيهم "Ali"

لكن الاتنين Object مختلفين.

---

لكن

```java
System.out.println(s1.equals(s2));
```

النتيجة

```
true
```

ليه؟

لأن String عمل Override للـ equals()

وبقى يقارن المحتوى.

---

## طيب Object الأصلي بيعمل إيه؟

الكود الداخلي تقريباً

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

يعني أصلاً equals في Object

بتعمل Reference Comparison

مش Value Comparison.

---

# مثال

لو عندك

```java
class Employee {

    int id;
    String name;

}
```

وعملت

```java
Employee e1 = new Employee();
e1.id = 1;
e1.name = "Ali";

Employee e2 = new Employee();
e2.id = 1;
e2.name = "Ali";
```

لو عملت

```java
System.out.println(e1.equals(e2));
```

النتيجة

```
false
```

ليه؟

لأنك لسه مستخدمتش Override.

---

# نعمل Override

```java
class Employee {

    int id;
    String name;

    @Override
    public boolean equals(Object obj) {

        if(this == obj)
            return true;

        if(obj == null)
            return false;

        if(getClass() != obj.getClass())
            return false;

        Employee other = (Employee) obj;

        return id == other.id &&
                Objects.equals(name, other.name);
    }
}
```

دلوقتي

```java
Employee e1 = new Employee(1,"Ali");
Employee e2 = new Employee(1,"Ali");

System.out.println(e1.equals(e2));
```

النتيجة

```
true
```

لأننا بقينا بنقارن القيم.

---

# طيب يعني إيه hashCode() ؟

دي النقطة اللي ناس كتير بتتلخبط فيها.

## الـ hashCode عبارة عن

**رقم Integer بيمثل الـ Object.**

مثلاً

```
Employee(id=1,name=Ali)

↓

1546823
```

وده بيتحسب بخوارزمية معينة.

---

مش لازم يكون Unique.

يعني

ممكن

```
Object A

↓

100
```

و

```
Object B

↓

100
```

وده عادي جدًا.

اسمه

```
Hash Collision
```

---

# ليه أصلاً محتاجين HashCode؟

عشان السرعة.

تخيل عندك

```
مليون Object
```

وجيت تدور على واحد.

فيه طريقتين.

---

## الطريقة الأولى

تقارن واحد واحد

```
O(n)
```

بطيئة.

---

## الطريقة الثانية

كل Object له Hash

```
Ali

↓

24567
```

أروح على المكان

24567

على طول.

```
O(1)
```

وده اللي HashMap بتعمله.

---

# إذن

الـ hashCode

لا يقول هل الاتنين Equal

ولا لأ.

هو بس

بيقول

> "غالباً لو الاتنين Equal هيبقوا في نفس المكان."

---

# العلاقة بين equals و hashCode

ودي أهم نقطة في الانترفيو.

القاعدة الذهبية

> **لو Object1.equals(Object2) == true**
> 
> **يبقى لازم**
> 
> **object1.hashCode() == object2.hashCode()**

لازم.

---

لكن العكس

مش صحيح.

يعني

```
hashCode متساوي

↓

مش شرط equals
```

---

مثال

```
Ali

↓

100
```

و

```
Omar

↓

100
```

ده عادي.

Collision.

---

# أشهر سؤال Interview

## Why override hashCode if equals is overridden?

الإجابة

لأن Collections زي

- HashMap
    
- HashSet
    
- Hashtable
    

بيعتمدوا على الاتنين.

---

# مثال عملي

نفترض

```java
class Employee {

    int id;

    Employee(int id){
        this.id=id;
    }

    @Override
    public boolean equals(Object o){

        Employee e=(Employee)o;

        return id==e.id;
    }

}
```

لاحظ

عملنا Override لـ equals فقط.

---

دلوقتي

```java
HashSet<Employee> set = new HashSet<>();

Employee e1 = new Employee(1);

Employee e2 = new Employee(1);

set.add(e1);

set.add(e2);

System.out.println(set.size());
```

المفروض

```
1
```

لكن

النتيجة

```
2
```

ليه؟

---

لأن

HashSet أول حاجة بتعمل

```
hashCode()
```

ولأنك معملتش Override

كل Object خد Hash مختلف.

```
e1

↓

12567
```

```
e2

↓

99821
```

راحوا Bucket مختلف.

أصلاً

equals

متحطتش في الصورة.

---

# إذن HashSet بتشتغل إزاي؟

لما تعمل

```java
set.add(obj)
```

الخطوات

```
Object

↓

hashCode()

↓

Bucket

↓

هل فيه Objects هنا؟

↓

لو آه

↓

equals()

↓

لو true

لا تضيف

↓

لو false

ضيف
```

يعني

```
hashCode

↓

أول فلتر
```

ثم

```
equals

↓

آخر قرار
```

---

# مثال كامل صح

```java
class Employee{

    int id;

    Employee(int id){
        this.id=id;
    }

    @Override
    public boolean equals(Object obj){

        Employee e=(Employee)obj;

        return id==e.id;
    }

    @Override
    public int hashCode(){

        return Objects.hash(id);

    }

}
```

الآن

```java
HashSet<Employee> set=new HashSet<>();

set.add(new Employee(1));

set.add(new Employee(1));

System.out.println(set.size());
```

النتيجة

```
1
```

---

# ليه Objects.hash()؟

بدل ما تعمل

```java
return id *31;
```

تقدر

```java
return Objects.hash(id);
```

أو

```java
return Objects.hash(id,name);
```

وJava هتعمل الحساب.

---

# هل لازم HashCode يكون Unique؟

لا.

مثال

```
Ali

↓

100
```

```
Ahmed

↓

100
```

عادي.

لكن

```
Ali

↓

100
```

```
Ali

↓

900
```

مينفعش لو `equals()` بينهم `true`.

---

# أشهر أسئلة الانترفيو

### 1) هل ممكن أعمل Override لـ equals بس؟

تقنيًا نعم، لكن **غلط** إذا كان الكلاس سيُستخدم داخل `HashMap` أو `HashSet`، لأنك ستكسر العقد (contract) بين `equals` و`hashCode`.

---

### 2) هل ممكن أعمل Override لـ hashCode بس؟

ينفع Compilation.

لكن Logic غلط.

---

### 3) مين بيتنادى الأول؟

```
hashCode()

↓

equals()
```

---

### 4) هل equals بيتنادى كل مرة؟

لا.

لو الـ hash مختلف

مش هيحصل equals أصلاً.

---

### 5) هل HashMap بتستخدم equals؟

نعم.

لكن بعد الـ hashCode.

---

### 6) هل hashCode يحدد إذا كان الـ Object موجود؟

لا.

هو يحدد الـ Bucket فقط.

بعدها

equals

هو اللي يقرر.

---

### 7) يعني إيه Collision؟

```
Object A

↓

150
```

```
Object B

↓

150
```

نفس الـ Hash.

وده طبيعي جدًا.

---

### 8) هل Collision يعتبر Bug؟

لا.

HashMap معمولة عشان تتعامل معاه.

---

# فلو HashMap بالكامل

```
put(key,value)

        │
        ▼
hashCode(key)
        │
        ▼
تحديد الـ Bucket
        │
        ▼
هل الـ Bucket فاضي؟
        │
   ┌────┴─────┐
   │          │
 نعم         لا
   │          │
   ▼          ▼
 أضف العنصر   قارن بالمفاتيح الموجودة باستخدام equals()
                 │
           ┌─────┴─────┐
           │           │
        equals=true  equals=false
           │           │
           ▼           ▼
     حدّث القيمة     أضف العنصر الجديد
```

---

# الإجابة المثالية في الانترفيو

لو سألك:

**What's the difference between equals() and hashCode()?**

تقدر تجاوب:

> `equals()` is used to determine whether two objects are logically equal based on their contents, while `hashCode()` returns an integer used to locate objects efficiently in hash-based collections such as `HashMap` and `HashSet`. The contract is that if two objects are equal according to `equals()`, they must return the same `hashCode()`. Hash-based collections call `hashCode()` first to find the bucket, then use `equals()` only to resolve collisions and identify the exact object.

---

## نصيحتي للمذاكرة

بما إنك كنت بتذاكر `HashMap`، فبعد ما تستوعب `equals()` و`hashCode()`، ذاكر مباشرة:

1. HashMap Internal Working (Bucket, Index, Collision).
    
2. Hash Collision وطرق التعامل معها.
    
3. لماذا أصبحت الـ Buckets تتحول إلى Red-Black Tree بعد Java 8.
    
4. الفرق بين `HashMap` و`LinkedHashMap` و`TreeMap`.
    

الأربع نقاط دول مترابطين جدًا، وغالبًا بييجوا مع بعض في مقابلات Java Backend.