لأن `findByIdAndCourseId` بترجع `Optional<Section>` مش `boolean`!

---

## يعني مينفعش تعمل `!` على Optional مباشرة

```java
// ❌ غلط — Optional مش boolean
if (!sectionRepository.findByIdAndCourseId(sectionId, courseId))

// ✅ صح — استخدم isEmpty()
if (sectionRepository.findByIdAndCourseId(sectionId, courseId).isEmpty())

// ✅ أو isPresent()
if (!sectionRepository.findByIdAndCourseId(sectionId, courseId).isPresent())
```

---

## القاعدة

|الـ Return Type|الـ Check|
|---|---|
|`boolean`|`if (!result)` ✅|
|`Optional<T>`|`if (result.isEmpty())` ✅|

الاتنين بيعملوا نفس الشيء بس كل واحد مع نوعه ✅