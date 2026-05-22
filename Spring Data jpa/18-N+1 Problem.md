**N+1 Problem** = مشكلة بتحصل لما بتعمل **query زيادة عن اللازم** من الـ database.

---

## المثال:

عندك **Authors** و **Books** — كل author عنده كتب.

عايز تعرض كل author مع كتبه.

---

### ❌ الطريقة الغلط (N+1):

```java
// 1 query تجيب كل الـ authors
List<Author> authors = authorRepository.findAll();

// لكل author بيعمل query تانية يجيب كتبه
for (Author author : authors) {
    List<Book> books = bookRepository.findByAuthor(author); // query جديدة!
}
```

**لو عندك 100 author:**

- 1 query للـ authors
- 100 query للكتب
- **المجموع = 101 query** 😱

دي هي الـ **N+1 Problem** — الـ N هي عدد الـ authors.

---

### ✅ الطريقة الصح:

```java
// query واحدة بس تجيب كل حاجة مع بعض
List<Author> authors = authorRepository.findAllWithBooks();
```

```sql
SELECT * FROM authors 
JOIN books ON books.author_id = authors.id
```

**المجموع = 1 query بس** ✅

---

## في Spring Boot بالـ JPA:

```java
// ❌ غلط - بيعمل N+1
@OneToMany
List<Book> books;

// ✅ صح - بيجيب كل حاجة مع بعض
@OneToMany(fetch = FetchType.EAGER)
@Fetch(FetchMode.JOIN)
List<Book> books;
```

---

**الخلاصة:** N+1 = بدل ما تعمل query واحدة، بتعمل N+1 query وده بيبطّل الأداء جداً.