@Component
public class A {
    @Autowired
    private B b;
}

@Component
public class B {
    @Autowired
    private A a;
}
```

**إيه اللي هيحصل؟**

- A) هيشتغل عادي
- B) BeanCreationException — Circular Dependency
- C) الـ Bean هيتعمل مرة واحدة بس
- D) Spring هيتجاهل إحدى الـ injections

**✅ الإجابة: B — Circular Dependency**
```
Spring حاول يعمل A → محتاج B
Spring حاول يعمل B → محتاج A
A محتاج B و B محتاج A → 💀 loop لا نهائي











@Component
public class A {
    @Autowired
    private B b;
}

@Component
public class B {
    @Autowired
    private A a;
}
**إيه الحل الصح للـ Circular Dependency؟**
circular dependency 

- A) تشيل `@Autowired` من A
- B) تحط `@Lazy` على واحدة من الـ injections
- C) تخلي الاتنين `prototype`
- D) تحط `@Primary` على A

✅ الإجابة: B





---

### **1️⃣ Circular Dependency يعني إيه؟**

Circular dependency (اعتماد دائري) بيحصل لما يكون عندك **كلاس A بيعتمد على كلاس B، وفي نفس الوقت كلاس B بيعتمد على كلاس A**.

مثال من الكود بتاعك:

```java
@Component
public class A {
    @Autowired
    private B b;  // A محتاج B
}

@Component
public class B {
    @Autowired
    private A a;  // B محتاج A
}
```

- لما Spring يحاول يعمل **instantiation** للكلاسات دي، بيبدأ بـ `A` ويقول: "محتاج `B`".
    
- بعدين يروح يعمل `B` ويقول: "محتاج `A`".
    
- وهنا بيقع في حلقة لانه مش قادر يخلص إنشاء الكلاسات دي، ودي اللي بنسميها **Circular Dependency**.
    

---

### **2️⃣ الحلول الشائعة للـ Circular Dependency**

#### **الحل الصح في الكود بتاعك: استخدام `@Lazy`**

```java
@Component
public class A {
    @Autowired
    @Lazy  // خلي الـ injection lazy
    private B b;
}
```

- `@Lazy` معناه: ما تعمّلش object دلوقتي، استنى لحد ما الكود يحتاجه فعليًا.
    
- Spring مش هيحاول يعمل instance للكلاس على طول، وده بيكسر الحلقة الدائرية.
    

---

#### **3️⃣ حلول تانية وليه مش مناسبة هنا**

- **تشيل `@Autowired` من A** → ده مش حل لأنك لسه محتاج الـ dependency.
    
- **تخلي الاتنين `prototype`** → ممكن يقلل المشكلة بس مش مضمون، لأن prototype يعني كل مرة هيعمل instance جديد، وده مش مناسب دايمًا.
    
- **تحط `@Primary` على A** → ده لتحديد أي Bean يتم اختياره لما يكون في أكثر من Bean من نفس النوع، **مش لحل circular dependency**.
    

---

### ✅ **الخلاصة**

- **Circular Dependency** = حلقة اعتمادية بين كلاس A وكلاس B.
    
- **Lazy** = تأجيل إنشاء الـ bean لوقت الاستخدام، وده بيكسر الحلقة وبيخلي Spring يقدر ينشئ الكلاسات دي بدون مشاكل.
    

---

لو تحب، أقدر أرسملك **diagrams صغيرة** توضح الـ circular dependency وازاي الـ `@Lazy` بيحلها، هتخليها أبسط بصريًا. تحب أعملها؟
