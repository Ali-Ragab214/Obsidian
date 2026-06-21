سؤال ممتاز، ودي بداية الدخول في **Failover & Leader Election**.

لنفترض عندك:

```text
Primary
├── Replica 1
└── Replica 2
```

### 1. الـ Primary وقع

```text
Primary ❌

Replica 1 ✅
Replica 2 ✅
```

النظام يكتشف أن الـ Primary لم يعد يرد على الـ Health Checks.

---

### 2. اختيار Primary جديد

يتم اختيار Replica من النسخ الموجودة (غالبًا الأكثر تحديثًا).

مثلاً:

```text
Replica 1 → New Primary
Replica 2 → Replica
```

فتصبح:

```text
Primary (old) ❌

Replica 1 → Primary ✅
Replica 2 → Replica ✅
```

هذا يسمى:

```text
Failover
```

---

### 3. ماذا يحدث عندما يعود الـ Primary القديم؟

**لا يرجع Primary تلقائيًا في أغلب الأنظمة.**

بل يصبح Replica.

يعني تصبح البنية:

```text
Replica 1 → Primary

Primary القديم → Replica
Replica 2      → Replica
```

لأن لو رجع كـ Primary مباشرة قد نحصل على مشكلة خطيرة اسمها:

```text
Split Brain
```

---

### ما هي Split Brain؟

تخيل:

```text
Server A يعتقد أنه Primary
Server B يعتقد أنه Primary
```

الآن يوجد قائدان.

المستخدم الأول يكتب على A.

المستخدم الثاني يكتب على B.

فتبدأ البيانات في الاختلاف والتعارض.

لذلك يمنع النظام هذا السيناريو.

---

### كيف يتم اختيار Primary جديد؟

في الأنظمة الحديثة يوجد ما يسمى:

```text
Leader Election
```

أي انتخاب قائد جديد.

تقوم الخوادم بالتصويت.

والسيرفر الفائز يصبح:

```text
Leader / Primary
```

---

### هل الـ Load Balancer هو من يختار الـ Primary؟

لا.

الـ Load Balancer لا يفهم قواعد البيانات عادة.

هو فقط يوزع الترافيك.

الذي يقرر من هو الـ Primary هو:

- Database Cluster Manager
    
- Consensus Protocol
    
- Leader Election System
    

مثل:

- etcd
    
- Apache ZooKeeper
    

وفي بعض الأنظمة يتم ذلك داخليًا.

---

### مثال سريع جدًا

```text
1) Primary يعمل
2) Primary يقع
3) Replica 1 تصبح Primary
4) المستخدمون يكملون العمل
5) Primary القديم يعود
6) يصبح Replica
7) ينسخ البيانات الجديدة من Primary الحالي
```

وده بالضبط السبب اللي يخلي النظام يحقق:

```text
High Availability
```

لأن تعطل السيرفر الرئيسي لا يوقف الخدمة بالكامل.