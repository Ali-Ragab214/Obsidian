## Hard Delete

بتمسح الصف **نهائيًا** من الداتابيز:

```sql
DELETE FROM users WHERE id = 5;
```

بعد كده الداتا مش موجودة، ومفيش طريقة ترجعها غير من backup.

## Soft Delete

مبتمسحش الصف، بتعلّمه بس إنه "اتمسح" بعمود زي `deleted` أو `deleted_at`:
we mark this row as deleted 

```sql
UPDATE users SET deleted = true WHERE id = 5;
```

الصف لسه موجود في الجدول، بس التطبيق بيتجاهله في كل الكويريز (`WHERE deleted = false`).

## المقارنة

||Hard Delete|Soft Delete|
|---|---|---|
|الداتا|بتروح نهائيًا|بتفضل موجودة|
|الاسترجاع|صعب (backup)|سهل (`deleted = false`)|
|الـ audit والتاريخ|بيضيع|بيتحفظ|
|الـ foreign keys|ممكن تضرب constraint أو تعمل cascade|العلاقات سليمة|
|حجم الجدول|بيصغر|بيكبر مع الوقت|
|الكويريز|عادية|لازم تفلتر المحذوفين في كل مكان|
|الـ Unique|بيتحرر فورًا|القيمة لسه محجوزة|
|الخصوصية (GDPR)|متوافق|محتاج مسح فعلي لاحقًا|

## امتى تستخدم أنهي؟

**Soft delete** مناسب لـ:

- اليوزرز والطلبات والفواتير، وأي حاجة ممكن تحتاج ترجعها أو تراجعها.
- الداتا المرتبطة بجداول تانية (orders بتاعة يوزر اتمسح).
- الأنظمة اللي محتاجة audit trail.

**Hard delete** مناسب لـ:

- الداتا المؤقتة (sessions، tokens منتهية، logs قديمة).
- لما القانون أو المستخدم يطلب مسح بياناته فعليًا (GDPR "right to be forgotten").
- الجداول اللي بتكبر بسرعة ومش محتاج تاريخها.

## إزاي تعملها في Spring Boot؟

الطريقة الأنضف إنك تخلي `repository.delete()` نفسها تعمل soft delete، وتفلتر المحذوفين أوتوماتيك من غير ما تكرر الشرط:

```java
@Entity
@SQLDelete(sql = "UPDATE users SET deleted = true WHERE id = ?")
@SQLRestriction("deleted = false")
public class User {
    private boolean deleted = false;
    // ...
}
```

كده:

- `repository.delete(user)` بتنفذ `UPDATE` بدل `DELETE`.
- كل الكويريز (`findById`، `existsByEmail`، ...) بتستبعد المحذوفين لوحدها.

## مشاكل لازم تاخد بالك منها

1. **الـ Unique:** لو اليوزر اتمسح soft وجه حد سجل بنفس الإيميل، هيضرب constraint. الحل partial unique index في الداتابيز (`WHERE deleted = false`).
2. **الكويريز اليدوية:** الـ native queries مبتطبقش عليها `@SQLRestriction`، لازم تضيف الشرط بإيدك.
3. **تنضيف دوري:** الداتا المحذوفة بتتراكم، فبعض الأنظمة بتعمل job يمسحها نهائيًا بعد فترة (مثلاً 90 يوم).

ولو عايز تعرف **امتى** اتمسح، استخدم `deleted_at` (timestamp) بدل `deleted` (boolean)، لأن null معناها موجود، وأي تاريخ معناه اتمسح في الوقت ده.