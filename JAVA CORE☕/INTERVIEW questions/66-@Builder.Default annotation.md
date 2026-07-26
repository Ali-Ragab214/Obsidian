سؤال ممتاز جدًا، ودي Annotation كتير من الناس بتستخدمها من غير ما تكون فاهمة هي ليه موجودة أصلاً.

وده من الأسئلة اللي **ممكن جدًا تيجي في انترفيو Java/Spring** لو المشروع بتاعك بيستخدم Lombok.

---

# 🎯 Interview Headline

# **What is `@Builder.Default` in Lombok? Why do we need it?**

---

# أولًا... لازم نفهم المشكلة قبل الحل

نفترض عندنا Class

```java
@Getter
@Setter
@Builder
public class User {

    private String name;

    private boolean active = true;
}
```

لاحظ إننا عاملين Default Value

```java
private boolean active = true;
```

---

دلوقتي استخدمنا الـ Builder

```java
User user = User.builder()
        .name("Ali")
        .build();
```

السؤال...

كم ستكون قيمة

```java
active
```

معظم الناس تقول

```text
true
```

❌ **غلط.**

---

# النتيجة الحقيقية

```java
System.out.println(user.isActive());
```

Output

```text
false
```

---

# لماذا؟

لأن الـ Builder **لا يستخدم الـ Field Initializer**.

هو تقريبًا بيولد كود بالشكل ده:

```java
public User build() {
    return new User(name, active);
}
```

ولأننا لم نحدد

```java
active
```

فالـ Builder يستخدم القيمة الافتراضية للنوع `boolean` وهي:

```text
false
```

وليس

```text
true
```

---

# إذن أين ذهبت

```java
= true
```

تم تجاهلها.

وده سبب وجود

```java
@Builder.Default
```

---

# الحل

```java
@Getter
@Setter
@Builder
public class User {

    private String name;

    @Builder.Default
    private boolean active = true;
}
```

---

الآن

```java
User user = User.builder()
        .name("Ali")
        .build();
```

النتيجة

```text
active = true
```

لأن Lombok هيولد Builder يحافظ على القيمة الافتراضية إذا لم يتم تعيينها.

---

# ماذا يحدث داخليًا؟

بدون

```java
@Builder.Default
```

الـ Builder تقريبًا يصبح

```java
class UserBuilder {

    boolean active;

}
```

ولأن

```text
boolean
```

قيمته الافتراضية

```text
false
```

فالنتيجة

```text
false
```

---

مع

```java
@Builder.Default
```

Lombok يولد كودًا إضافيًا (مبسطًا):

```java
private boolean active$value;
private boolean active$set;
```

أثناء `build()`:

```java
if (!active$set) {
    active$value = true;
}
```

أي:

> لو المستخدم **لم يستدعِ** `.active(...)`، استخدم القيمة المكتوبة في الـ Field.

أما إذا استدعى:

```java
.active(false)
```

فلن يتم استخدام القيمة الافتراضية.

---

# مثال

```java
User user = User.builder()
        .name("Ali")
        .active(false)
        .build();
```

النتيجة

```text
active = false
```

لأنك حددتها بنفسك.

---

# مثال آخر

```java
@Builder
public class Product {

    private String name;

    @Builder.Default
    private int quantity = 1;
}
```

لو كتبنا

```java
Product p = Product.builder()
        .name("Mouse")
        .build();
```

النتيجة

```text
quantity = 1
```

---

ولو

```java
Product.builder()
        .quantity(20)
        .build();
```

تصبح

```text
20
```

---

# مع Collections

وده أشهر Use Case.

```java
@Builder
public class Order {

    @Builder.Default
    private List<String> items = new ArrayList<>();
}
```

بدون

```java
@Builder.Default
```

```java
Order order = Order.builder().build();
```

النتيجة

```java
order.getItems()
```

تساوي

```text
null
```

وليس

```java
new ArrayList<>()
```

وده ممكن يعمل

```text
NullPointerException
```

---

مع

```java
@Builder.Default
```

تصبح

```java
[]
```

ArrayList فارغة.

---

# أشهر Use Cases

## 1- Boolean Flags

```java
@Builder.Default
private boolean active = true;
```

---

## 2- Status

```java
@Builder.Default
private Status status = Status.PENDING;
```

---

## 3- Collections ⭐

```java
@Builder.Default
private List<Role> roles = new ArrayList<>();
```

---

## 4- Counters

```java
@Builder.Default
private int retryCount = 3;
```

---

## 5- Dates

```java
@Builder.Default
private LocalDate createdDate = LocalDate.now();
```

> **ملحوظة:** هذا سيُنفذ وقت إنشاء الـ Object، وليس وقت تحميل الـ Class، لأن Lombok ينقل تعبير القيمة الافتراضية إلى منطق الـ Builder.

---

# هل تعمل بدون Builder؟

نعم.

```java
User user = new User();
```

هنا

```java
private boolean active = true;
```

ستعمل بشكل طبيعي.

المشكلة موجودة فقط عند استخدام

```java
@Builder
```

---

# هل أحتاجها دائمًا؟

لا.

إذا كنت ستحدد كل القيم يدويًا:

```java
User.builder()
        .active(true)
        .build();
```

فلا تحتاجها.

لكن إذا أردت قيمة افتراضية عند عدم تحديد الحقل، فاستخدمها.

---

# أشهر سؤال انترفيو

### لماذا نستخدم `@Builder.Default`؟

الإجابة:

> لأن Lombok Builder لا يستخدم قيم الـ Field الافتراضية (`Field Initializers`) تلقائيًا. فإذا أردنا الاحتفاظ بهذه القيم عند إنشاء الكائن باستخدام الـ Builder، نستخدم `@Builder.Default`.

---

### هل هي خاصة بـ Spring؟

لا.

هي Annotation من

```text
Lombok
```

وليست من Spring.

---

### هل يمكن استخدامها بدون `@Builder`؟

لا فائدة منها.

هي مصممة خصيصًا للعمل مع

```java
@Builder
```

---

### هل يمكن استخدامها مع Immutable Objects؟

نعم، بل هذا من أكثر استخداماتها شيوعًا، خصوصًا مع:

```java
@Builder
@Getter
```

أو

```java
@Value
@Builder
```

---

# مقارنة سريعة

بدون `@Builder.Default`

```java
@Builder
class User {
    private boolean active = true;
}

User.builder().build();
```

النتيجة:

```text
active = false
```

---

مع `@Builder.Default`

```java
@Builder
class User {

    @Builder.Default
    private boolean active = true;
}

User.builder().build();
```

النتيجة:

```text
active = true
```

---

# 🎯 الإجابة الذهبية في الانترفيو

> **"`@Builder.Default` is a Lombok annotation used with `@Builder` to preserve a field's default value when the builder is used. Without it, Lombok ignores field initializers and uses Java's default values such as `null`, `0`, or `false`. It's commonly used for default statuses, boolean flags, collections, counters, and timestamps."**

---

## 💡 نقطة متقدمة قد يسألك عنها الـ Interviewer

لو سألك:

> **"Why doesn't Lombok Builder use the field initializer automatically?"**

الإجابة هي:

لأن الـ `Builder` لا ينشئ الكائن عن طريق:

```java
new User();
```

ثم يملأ الحقول.

بل ينشئ الكائن عن طريق **Constructor** (سواء مُولد بواسطة Lombok أو موجود بالفعل)، ويمرر إليه القيم التي جمعها أثناء البناء. لذلك لا يتم الاعتماد تلقائيًا على قيم الـ Field Initializers، ولهذا احتاج Lombok إلى `@Builder.Default` ليحافظ على هذه القيم عندما لا يحددها المستخدم.