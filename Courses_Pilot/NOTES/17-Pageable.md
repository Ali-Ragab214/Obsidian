## `PageRequest.of(page, size)`

الـ `Pageable` ده **interface** في Spring Data — بيحتوي على كل معلومات الـ pagination مع بعض.

```
page = 0
size = 10
        ↓
PageRequest.of(page, size)
        ↓
Pageable {
    page: 0,
    size: 10,
    offset: 0,
    sort: UNSORTED
}
```

---

## ليه بنعملها كده؟

الـ Repository بتاعك بيتكلم **Pageable مش أرقام:**

```java
// ❌ مش موجود في Spring Data
enrollmentRepository.findByStudentId(id, page, size);

// ✅ الصح
enrollmentRepository.findByStudentId(id, pageable);
```

فـ `PageRequest.of()` بتـ **pack** الـ `page` و `size` في object واحد الـ Repository يفهمه.

---

## الفايدة الكبيرة — Sorting

```java
// من غير sorting
PageRequest.of(page, size);

// بـ sorting
PageRequest.of(page, size, Sort.by("createdAt").descending());
```

لو مش بتستخدم `Pageable`، هتضطر تبعت:

```java
findByStudentId(id, page, size, "createdAt", "DESC") // ❌ بشع
```

---

## باختصار

|`page + size` منفصلين|`Pageable`|
|---|---|
|أرقام بس|أرقام + sorting + offset|
|الـ Repository مش بيقبلهم|الـ Repository بيقبله directly|
|لو ضفت sorting هتغير الـ signature|مش بتغير أي حاجة|
