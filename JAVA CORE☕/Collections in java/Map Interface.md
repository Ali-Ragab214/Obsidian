
تمام يا علي 👌  
خلينا نشرح **`Map` Interface في Java** بطريقة مرتبة وسهلة + أمثلة عملية 🔥

---

# 🔹 يعني إيه Map؟

الـ `Map` هو Interface في:

```java
java.util
```

فكرته الأساسية:

> 🔥 بيخزن البيانات في شكل **Key → Value**

زي القاموس 📖  
كل مفتاح (Key) مرتبط بقيمة (Value)

---

# 🔹 مثال بسيط

```java
Map<String, Integer> students = new HashMap<>();

students.put("Ali", 90);
students.put("Sara", 95);

System.out.println(students);
```

الناتج ممكن يكون:

```
{Ali=90, Sara=95}
```

---

# 🔥 أهم خصائص الـ Map

✔ كل **Key لازم يكون Unique**  
❌ ماينفعش تكرر نفس المفتاح  
✔ القيم (Values) ينفع تتكرر عادي

---

## مثال على تكرار Key

```java
students.put("Ali", 100);
```

ده مش هيضيف عنصر جديد  
هيعدل قيمة "Ali" بدل 90 تبقى 100

---

# 🔹 أهم الميثودز في Map

|Method|وظيفتها|
|---|---|
|put(k,v)|إضافة عنصر|
|get(k)|ترجع القيمة|
|remove(k)|حذف عنصر|
|containsKey(k)|هل المفتاح موجود؟|
|containsValue(v)|هل القيمة موجودة؟|
|keySet()|ترجع كل المفاتيح|
|values()|ترجع كل القيم|
|entrySet()|ترجع Key + Value|

---

# 🔹 مثال استخدام get

```java
int grade = students.get("Ali");
System.out.println(grade);
```

---

# 🔹 أفضل طريقة تعمل Loop على Map

```java
for(Map.Entry<String, Integer> entry : students.entrySet()) {
    System.out.println(entry.getKey() + " : " + entry.getValue());
}
```

---

# 🔥 أهم أنواع الـ Map

## 1️⃣ HashMap

```java
Map<String, Integer> map = new HashMap<>();
```

- أسرع نوع
    
- مش بيحافظ على الترتيب
    
- يسمح بـ null key واحدة
    

---

## 2️⃣ LinkedHashMap

```java
Map<String, Integer> map = new LinkedHashMap<>();
```

- بيحافظ على ترتيب الإدخال
    

---

## 3️⃣ TreeMap

```java
Map<String, Integer> map = new TreeMap<>();
```

- بيرتب المفاتيح تلقائيًا
    
- أبطأ شوية
    
- لا يسمح بـ null key
    

---

# 🔥 مقارنة سريعة

|النوع|يحافظ على الترتيب؟|بيرتب؟|السرعة|
|---|---|---|---|
|HashMap|❌|❌|🔥 أسرع|
|LinkedHashMap|✔|❌|سريع|
|TreeMap|❌|✔|أبطأ|

---

# 🔹 مثال عملي مهم (حساب تكرار الكلمات)

```java
String text = "java is easy java is powerful";
String[] words = text.split(" ");

Map<String, Integer> wordCount = new HashMap<>();

for(String word : words) {
    wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
}

System.out.println(wordCount);
```

الناتج:

```
{java=2, is=2, easy=1, powerful=1}
```

---

# 🎯 تستخدم Map إمتى؟

✔ لما تحتاج تربط قيمة بمفتاح  
✔ لما تعمل Cache  
✔ لما تعمل Dictionary  
✔ لما تعد حاجات (Counting)  
✔ لما تمثل بيانات زي JSON

---

# 🔥 سؤال انترفيو مهم

> الفرق بين Map و Set؟

- Set بيخزن عنصر واحد بدون تكرار
    
- Map بيخزن Key + Value
    

---

لو تحب أشرحلك:

- إزاي HashMap شغالة داخليًا (HashTable + Buckets)
    
- الفرق بينها وبين Hashtable
    
- أو ConcurrentHashMap
    

قوللي وندخل مستوى أعمق 🔥