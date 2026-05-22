حلو 🔥 ده فعلاً أهم جزء في Nest + TypeORM وبيظهر في أي مشروع Production.

## جملة تحفظها للإنترفيو 🎯

> Query Builder is a programmatic way to build complex SQL queries using code instead of writing raw SQL.

لو حابب أعملك مقارنة بين QueryBuilder و Raw SQL و Repository methods قولّي 😉
هنكتبها بشكل مرتب ينفع تتحط في **Obsidian Notes** 👌  
عنوان النوتة:

```txt
TypeORM Query Builder - Production Guide
```

---

# 🚀 Query Builder in TypeORM

QueryBuilder بيخليك تكتب **SQL متقدم** لكن باستخدام TypeScript.

نستخدمه لما:

- نحتاج joins كتير
    
- search + filters
    
- pagination + sorting
    
- analytics / aggregation
    

---

# 🧱 مثال Entities سريع

```ts
User (id, name, role)
Course (id, title, price, instructorId)
Enrollment (id, userId, courseId)
Review (id, rating, courseId)
```

---

# 1️⃣ Basic QueryBuilder

```ts
const courses = await this.courseRepo
  .createQueryBuilder("course")
  .getMany();
```

"course" ده alias زي SQL.

---

# 2️⃣ SELECT Columns محددة

بدل ما نرجع كل الأعمدة:

```ts
const courses = await this.courseRepo
  .createQueryBuilder("course")
  .select(["course.id", "course.title", "course.price"])
  .getMany();
```

⚡ يقلل حجم الـ response → performance أفضل.

---

# 3️⃣ WHERE Filters

```ts
const courses = await this.courseRepo
  .createQueryBuilder("course")
  .where("course.price > :price", { price: 100 })
  .getMany();
```

زي SQL:

```sql
WHERE price > 100
```

---

# 4️⃣ Multiple Conditions

```ts
.where("course.price > :price", { price: 100 })
.andWhere("course.title LIKE :title", { title: "%java%" })
```

SQL:

```sql
WHERE price > 100 AND title LIKE '%java%'
```

---

# 🔎 5️⃣ SEARCH (Real life search)

Search by title OR description:

```ts
.where(
  "course.title ILIKE :search OR course.description ILIKE :search",
  { search: `%${search}%` }
)
```

💡 ILIKE = case insensitive في PostgreSQL.

---

# 🤝 6️⃣ JOINS (أهم جزء)

## Join Course مع Instructor

```ts
const courses = await this.courseRepo
  .createQueryBuilder("course")
  .leftJoinAndSelect("course.instructor", "instructor")
  .getMany();
```

SQL:

```sql
LEFT JOIN users instructor
ON instructor.id = course.instructorId
```

---

## Join Multiple Tables

```ts
const courses = await this.courseRepo
  .createQueryBuilder("course")
  .leftJoinAndSelect("course.instructor", "instructor")
  .leftJoinAndSelect("course.reviews", "reviews")
  .getMany();
  
  const courses = await this.courseRepo
  .createQueryBuilder("course")
  .leftJoinAndSelect("course.instructor","instructor")
  .leftJoinAndSelect("course.reviews","reviews")
  .getMany();
```

ده بيرجع:

- course
    
- instructor
    
- reviews
    

في Query واحدة ⚡

---

# 📄 7️⃣ Pagination (Production Must)

```ts
const page = 1;
const limit = 10;

const [courses, total] = await this.courseRepo
  .createQueryBuilder("course")
  .skip((page - 1) * limit)
  .take(limit)
  .getManyAndCount();
```

النتيجة:

```ts
courses → data
total   → total count
```

---

# 🔽 8️⃣ Sorting

```ts
.orderBy("course.createdAt", "DESC")
```

أو:

```ts
.orderBy("course.price", "ASC")
```

Dynamic sorting:

```ts
.orderBy(`course.${sortBy}`, sortOrder)
```

---

# 📊 9️⃣ Aggregation (COUNT / AVG / SUM)

## Count reviews per course

```ts
const result = await this.courseRepo
  .createQueryBuilder("course")
  .leftJoin("course.reviews", "review")
  .select("course.id", "courseId")
  .addSelect("COUNT(review.id)", "reviewCount")
  .groupBy("course.id")
  .getRawMany();
```

SQL:

```sql
SELECT course.id, COUNT(review.id)
FROM courses
LEFT JOIN reviews
GROUP BY course.id
```

---

## Average rating per course ⭐

```ts
const ratings = await this.courseRepo
  .createQueryBuilder("course")
  .leftJoin("course.reviews", "review")
  .select("course.id", "courseId")
  .addSelect("AVG(review.rating)", "avgRating")
  .groupBy("course.id")
  .getRawMany();
```

---

# 🧠 10️⃣ Real Production Query (🔥 مهم)

Search + filter + pagination + sorting مع بعض:

```ts
const query = this.courseRepo
  .createQueryBuilder("course")
  .leftJoinAndSelect("course.instructor", "instructor");

if (search) {
  query.andWhere(
    "course.title ILIKE :search",
    { search: `%${search}%` }
  );
}

if (minPrice) {
  query.andWhere("course.price >= :minPrice", { minPrice });
}

query.orderBy("course.createdAt", "DESC");

query.skip((page - 1) * limit).take(limit);

const [courses, total] = await query.getManyAndCount();
```

ده اللي بيتكتب في المشاريع الكبيرة حرفيًا 👌

---

# 💎 نصايح Production

|Rule|السبب|
|---|---|
|استخدم QueryBuilder في الـ complex queries|Repository methods مش كفاية|
|استخدم select لتقليل البيانات|Performance|
|استخدم pagination دائمًا|منع تحميل آلاف الصفوف|
|استخدم joins بدل queries متعددة|يقلل DB calls|

---

لو حابب بعد كده نعمل **Advanced filtering system كامل** زي اللي في مواقع الكورسات 🔥 ده level أعلى شوية.