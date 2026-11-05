معلش، أنا لخبطتك بكلام كتير. هشرحها من الأول وخطوة بخطوة.

## الفكرة الأساسية

لما تعمل `repository.save(user)`، قبل ما JPA يبعت الـ INSERT للداتابيز بيمر بلحظة صغيرة. `@PrePersist` معناها: "لو في ميثود عليها العلامة دي، شغلها في اللحظة دي".

زي موظف الأمن على باب المبنى، قبل ما أي حد يدخل بيراجعه. إنت اللي بتحدد المراجعة دي.

## في كودك

```java
@PrePersist
protected void onCreate() {
    if (this.deleted == null)
        this.deleted = false;
}
```

يعني: "قبل ما تحفظ السجل، لو `deleted` فاضية (`null`) خليها `false`".

## ليه محتاجها؟

لأن `deleted` نوعها `Boolean` (بحرف كبير)، وده ممكن يبقى `null`، عكس `boolean` الصغيرة.

ولما تستخدم `@SuperBuilder`:

```java
User user = User.builder().name("Ali").build();
```

إنت ما حطيتش قيمة لـ `deleted`، فالـ builder هيسيبها `null`. لو العمود في الداتابيز `NOT NULL` هيطلع error، ولو لأ هتتخزن `null`، وده غلط لأن المفروض تكون `true` أو `false`.

فالـ `@PrePersist` بتصلح ده قبل الحفظ.

## الحل الأبسط (اللي أنصحك بيه)

بدل ما تكتب ميثود، قول للـ builder القيمة الافتراضية من الأول:

```java
@Builder.Default
@Column(nullable = false)
private Boolean deleted = false;
```

كده `deleted` هتبقى `false` تلقائي، ومش محتاج `@PrePersist` خالص.

ومن غير `@Builder.Default` الـ builder بيتجاهل الـ `= false`، وده السبب اللي خلاك محتاج `@PrePersist` أصلاً.

## ملخص

|الطريقة|إمتى|
|---|---|
|`@Builder.Default`|الأبسط، استخدمها في الحالة دي|
|`@PrePersist`|لما القيمة محتاجة حساب، زي إنك تولد code أو تحول إيميل لحروف صغيرة|

عندك كمان `@PreUpdate`، وبتشتغل قبل الـ UPDATE بنفس الفكرة.

## نصيحة أخيرة

خلي الداتابيز نفسها تحميك في الـ migration:

```sql
deleted BOOLEAN NOT NULL DEFAULT FALSE
```

وبكده لو الـ Java نسيت، الداتابيز هتحط `false` بنفسها.

لو لسه في حاجة مش واضحة، قولي أنهي سطر بالظبط وأشرحه أكتر.