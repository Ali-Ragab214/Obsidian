

---

# 1️⃣ CREATE DATABASEأول حاجة لازم تفهمها — MySQL بتشتغل على مستويين: **Database** (الحاوية الكبيرة) و **Table** (اللي جواها). ده زي إنك عندك مبنى (database) وجواه غرف (tables).

---

## `CREATE DATABASE` و `DROP DATABASE`

```sql
-- إنشاء database جديدة
CREATE DATABASE ecommerce;

-- ✅ الأفضل دايماً — بيتجنب error لو موجودة
CREATE DATABASE IF NOT EXISTS ecommerce;

-- مع تحديد الـ character set (مهم للعربي والـ emoji)
CREATE DATABASE ecommerce
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- اختيار database عشان تشتغل عليها
USE ecommerce;

-- مسح database كاملة (خطر!)
DROP DATABASE ecommerce;
DROP DATABASE IF EXISTS ecommerce;  -- بدون error لو مش موجودة
```

**Gotcha هنا مهم جداً:** الفرق بين `utf8` و `utf8mb4` في MySQL.

`utf8` في MySQL مش UTF-8 حقيقي — بيدعم بس 3 bytes، يعني الـ emoji مش بيتخزن صح. `utf8mb4` هو الـ UTF-8 الحقيقي (4 bytes) — استخدمه دايماً.

---

## `CREATE TABLE` — القلب الأساسي

```sql
USE ecommerce;

CREATE TABLE products (
    id          INT             NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100)    NOT NULL,
    description TEXT,
    price       DECIMAL(10, 2)  NOT NULL,
    stock       INT             DEFAULT 0,
    created_at  DATETIME        DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

الجدول ده بيمثل الـ `Product` entity اللي شفناها في Hibernate — نفس الفكرة بالظبط بس بـ SQL.

---

## Data Types — كل نوع وامتى تستخدمه---

## مثال عملي كامل — نبني schema لنظام orders

```sql
CREATE DATABASE IF NOT EXISTS shop
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE shop;

-- جدول العملاء
CREATE TABLE customers (
    id         INT          NOT NULL AUTO_INCREMENT,
    first_name VARCHAR(50)  NOT NULL,
    last_name  VARCHAR(50)  NOT NULL,
    email      VARCHAR(150) NOT NULL,
    phone      VARCHAR(20),
    created_at DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_email (email)          -- email مش بيتكرر
);

-- جدول المنتجات
CREATE TABLE products (
    id          INT            NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100)   NOT NULL,
    description TEXT,
    price       DECIMAL(10,2)  NOT NULL,
    stock       INT            NOT NULL DEFAULT 0,
    status      VARCHAR(20)    NOT NULL DEFAULT 'active',
    created_at  DATETIME       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT chk_price CHECK (price > 0),        -- advanced
    CONSTRAINT chk_stock CHECK (stock >= 0)
);

-- جدول الأوردرات — فيه Foreign Keys
CREATE TABLE orders (
    id          INT          NOT NULL AUTO_INCREMENT,
    customer_id INT          NOT NULL,
    total       DECIMAL(10,2) NOT NULL,
    status      VARCHAR(20)  NOT NULL DEFAULT 'pending',
    created_at  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE RESTRICT          -- امنع مسح customer عنده orders
        ON UPDATE CASCADE           -- لو id اتغير → اتغير هنا أوتوماتيك
);
```

---

## `ALTER TABLE` — تعديل table موجودة

```sql
-- إضافة column جديدة
ALTER TABLE products
  ADD COLUMN weight DECIMAL(8,2) AFTER price;

-- تغيير نوع الـ column
ALTER TABLE products
  MODIFY COLUMN description MEDIUMTEXT;

-- إعادة تسمية column (MySQL 8+)
ALTER TABLE products
  RENAME COLUMN status TO product_status;

-- حذف column
ALTER TABLE products
  DROP COLUMN weight;

-- إضافة index
ALTER TABLE products
  ADD INDEX idx_status (product_status);

-- إضافة constraint
ALTER TABLE orders
  ADD CONSTRAINT chk_status
  CHECK (status IN ('pending','confirmed','shipped','delivered','cancelled'));
```

---

## `DROP TABLE` والفرق بين `DELETE` و `TRUNCATE` و `DROP````sql

-- DELETE — بيمسح rows معينة (أو كل الـ rows) DELETE FROM products WHERE stock = 0; DELETE FROM products; -- كل الـ rows لكن الـ table لسه موجودة

-- TRUNCATE — بيمسح كل الـ rows بسرعة TRUNCATE TABLE products; -- AUTO_INCREMENT بيرجع للـ 1

-- DROP — بيمسح الـ table نهائياً DROP TABLE IF EXISTS products;

````

---

## الـ Gotchas الأهم 🔥

**1. `INT(11)` — الرقم ده مش الـ size!**
```sql
-- ❌ كتير ناس بتفكر إن INT(11) بيخزن 11 رقم
INT(11)    -- الـ 11 ده display width بس — مش عارفلك علاقة بالـ storage
BIGINT(20) -- نفس الكلام — الـ storage ثابت (8 bytes)

-- ✅ الصح: شيل الأرقام دي خالص في MySQL 8+
INT NOT NULL   -- كافي تماماً
````

**2. `VARCHAR` vs `TEXT` — مش نفس الحاجة**

```sql
-- VARCHAR: بيتخزن في الـ row مباشرةً — أسرع في الـ queries
-- TEXT: بيتخزن خارج الـ row — أبطأ وما ينفعش تعمله index عادي
-- القاعدة: لو الـ data < 255 حرف → VARCHAR، لو أكبر → TEXT
```

**3. `DATETIME` vs `TIMESTAMP`**

```sql
-- DATETIME: بيخزن زي ما هو، range حتى 9999، مش بيتأثر بـ timezone
-- TIMESTAMP: بيحوله UTC عند التخزين، Y2K38 problem (2038 هيبوظ!)
-- الأفضل: استخدم DATETIME دايماً لـ application timestamps
```

**4. `NULL` ≠ `''` (empty string)**

```sql
-- NULL يعني "مش موجود" — ملوش قيمة
-- ''  يعني "موجود بس فاضي"
SELECT * FROM customers WHERE phone IS NULL;       -- ✅ صح
SELECT * FROM customers WHERE phone = NULL;        -- ❌ دايماً FALSE!
SELECT * FROM customers WHERE phone = '';          -- ✅ empty string
```

---

تمام؟ نكمل على **Constraints** (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`, `DEFAULT`) ولا عايز تسألني على حاجة من اللي فاتت الأول؟


يلا نكمل — **Constraints** دلوقتي، وبعدين DML كامل.

---

# 2️⃣ Constraints — القيود على البيانات

الـ Constraint هو قاعدة بتحطها على الـ column أو الـ table عشان MySQL تمنع أي data غلط تتخزن.---

## PRIMARY KEY

```sql
-- طريقة 1: inline مع الـ column
CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- طريقة 2: في الآخر (أوضح وأفضل)
CREATE TABLE users (
    id   INT         NOT NULL AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
);

-- Composite Primary Key — مفتاح مركب (جدولين ببعض)
CREATE TABLE order_items (
    order_id   INT NOT NULL,
    product_id INT NOT NULL,
    quantity   INT NOT NULL DEFAULT 1,
    PRIMARY KEY (order_id, product_id)   -- الاتنين مع بعض = unique
);
```

**Gotcha:** الـ `PRIMARY KEY` = `NOT NULL` + `UNIQUE` معاً أوتوماتيك. مش محتاج تكتبهم.

---

## UNIQUE

```sql
CREATE TABLE customers (
    id    INT         NOT NULL AUTO_INCREMENT,
    email VARCHAR(150) NOT NULL,
    phone VARCHAR(20),
    PRIMARY KEY (id),

    -- Unique على column واحدة
    UNIQUE KEY uq_email (email),

    -- Composite unique — الاتنين مع بعض لازم يكونوا unique
    -- يعني نفس الاسم ممكن يكون في دول مختلفة
    UNIQUE KEY uq_name_country (first_name, country_code)
);
```

**Gotcha مهم:** `UNIQUE` بيسمح بـ `NULL` أكتر من مرة في MySQL — لأن `NULL ≠ NULL`. يعني ممكن عندك 100 صف كلهم phone = NULL وده مش بيكسر الـ UNIQUE constraint.

---

## NOT NULL و DEFAULT

```sql
CREATE TABLE products (
    id          INT           NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100)  NOT NULL,          -- إجباري
    description TEXT,                            -- اختياري (NULL مسموح)
    price       DECIMAL(10,2) NOT NULL,
    stock       INT           NOT NULL DEFAULT 0,           -- لو مش بعتهاش → 0
    status      VARCHAR(20)   NOT NULL DEFAULT 'active',
    is_featured BOOLEAN       NOT NULL DEFAULT FALSE,
    created_at  DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

الـ `ON UPDATE CURRENT_TIMESTAMP` ده بيعمل نفس اللي `@PreUpdate` بيعمله في Hibernate — بيحدّث الـ timestamp أوتوماتيك عند كل UPDATE.

---

## CHECK — شرط منطقي على القيمة

```sql
CREATE TABLE employees (
    id         INT           NOT NULL AUTO_INCREMENT,
    name       VARCHAR(100)  NOT NULL,
    age        TINYINT       NOT NULL,
    salary     DECIMAL(10,2) NOT NULL,
    department VARCHAR(50)   NOT NULL,
    hire_date  DATE          NOT NULL,
    PRIMARY KEY (id),

    -- Constraints بأسماء واضحة
    CONSTRAINT chk_age       CHECK (age BETWEEN 18 AND 65),
    CONSTRAINT chk_salary    CHECK (salary > 0),
    CONSTRAINT chk_dept      CHECK (department IN ('IT','HR','Finance','Sales')),
    CONSTRAINT chk_hire_date CHECK (hire_date >= '2000-01-01')
);

-- لو حاولت تحط age = 15:
-- ERROR 3819 (HY000): Check constraint 'chk_age' is violated.
```

**Gotcha:** الـ `CHECK` constraint اتضاف فعلياً في **MySQL 8.0.16** — في الإصدارات القديمة كانت بتتكتب بس مكانتش بتشتغل!

---

## FOREIGN KEY — الأهم والأصعب```sql

CREATE TABLE orders ( id INT NOT NULL AUTO_INCREMENT, customer_id INT NOT NULL, total DECIMAL(10,2) NOT NULL, status VARCHAR(20) NOT NULL DEFAULT 'pending', created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP, PRIMARY KEY (id),

```
CONSTRAINT fk_orders_customer
    FOREIGN KEY (customer_id)
    REFERENCES customers(id)
    ON DELETE RESTRICT    -- الخيارات هنا مهمة جداً
    ON UPDATE CASCADE
```

);

````

### الخيارات الأربعة لـ `ON DELETE` / `ON UPDATE`

```sql
-- RESTRICT (الـ default) — امنع العملية لو في children
ON DELETE RESTRICT
-- → لو حاولت تمسح customer عنده orders → ERROR

-- CASCADE — اعمل نفس العملية على الـ children
ON DELETE CASCADE
-- → لو مسحت customer → orders بتاعته اتمسحت أوتوماتيك

-- SET NULL — حط NULL في الـ FK
ON DELETE SET NULL
-- → لو مسحت customer → orders.customer_id = NULL
-- (لازم customer_id يكون nullable)

-- NO ACTION — زي RESTRICT في MySQL (بيتقيّم بعد الـ transaction)
ON DELETE NO ACTION
````

**Gotcha مهم:** لو عندك FK وعايز تعمل `TRUNCATE` على الـ parent table هتاخد error. لازم تعطّل الـ FK checks الأول:

```sql
SET FOREIGN_KEY_CHECKS = 0;
TRUNCATE TABLE customers;
SET FOREIGN_KEY_CHECKS = 1;
-- ⚠️ خطر — بتخلي البيانات inconsistent
```

---

# 3️⃣ DML — التعامل مع البيانات

## INSERT

```sql
-- إدخال صف واحد
INSERT INTO customers (first_name, last_name, email)
VALUES ('Ahmed', 'Hassan', 'ahmed@example.com');

-- إدخال أكتر من صف دفعة واحدة (أسرع بكتير من loops)
INSERT INTO products (name, price, stock) VALUES
    ('Laptop',    15000.00, 10),
    ('Mouse',       250.00, 50),
    ('Keyboard',    800.00, 30),
    ('Monitor',   5000.00,  8);

-- INSERT IGNORE — لو الـ row موجودة → تجاهل من غير error
INSERT IGNORE INTO customers (email, first_name, last_name)
VALUES ('ahmed@example.com', 'Ahmed', 'Hassan');

-- ON DUPLICATE KEY UPDATE — upsert (insert أو update)
INSERT INTO products (id, name, price, stock)
VALUES (1, 'Laptop', 14500.00, 15)
ON DUPLICATE KEY UPDATE
    price = VALUES(price),
    stock = VALUES(stock);
-- لو id=1 موجود → UPDATE · لو مش موجود → INSERT
```

---

## SELECT — من الأساسي للـ Advanced

```sql
-- الأساسي
SELECT * FROM products;
SELECT name, price FROM products;

-- مع WHERE
SELECT * FROM products WHERE price > 1000;
SELECT * FROM products WHERE stock = 0;
SELECT * FROM products WHERE status = 'active' AND price < 5000;
SELECT * FROM products WHERE price BETWEEN 500 AND 3000;
SELECT * FROM products WHERE name LIKE 'Lap%';     -- يبدأ بـ Lap
SELECT * FROM products WHERE name LIKE '%top';     -- ينتهي بـ top
SELECT * FROM products WHERE name LIKE '%ap%';     -- يحتوي على ap
SELECT * FROM products WHERE id IN (1, 3, 5);
SELECT * FROM products WHERE description IS NULL;
SELECT * FROM products WHERE description IS NOT NULL;

-- ORDER BY
SELECT * FROM products ORDER BY price ASC;           -- تصاعدي (default)
SELECT * FROM products ORDER BY price DESC;          -- تنازلي
SELECT * FROM products ORDER BY stock ASC, price DESC; -- ترتيب مركب

-- LIMIT و OFFSET — مهمين جداً للـ pagination
SELECT * FROM products LIMIT 10;                -- أول 10
SELECT * FROM products LIMIT 10 OFFSET 20;     -- الصفحة الثالثة (تبدأ من 20)
SELECT * FROM products LIMIT 20, 10;           -- نفس الكلام — offset أول
```

**Gotcha:** `SELECT *` في الـ production بطيء وخطر — بيجيب columns إنت مش محتاجها وبيضيع bandwidth. دايماً حدد الـ columns اللي محتاجها.

---

## Aggregate Functions

```sql
SELECT COUNT(*)                    FROM products;             -- عدد كل الصفوف
SELECT COUNT(description)          FROM products;             -- بيتجاهل NULL!
SELECT COUNT(DISTINCT category_id) FROM products;             -- distinct values
SELECT SUM(stock)                  FROM products;
SELECT AVG(price)                  FROM products;
SELECT MIN(price), MAX(price)      FROM products;
SELECT ROUND(AVG(price), 2)        FROM products;
```

**Gotcha مهم:** `COUNT(*)` بيعدّ كل الصفوف حتى لو فيهم NULL. `COUNT(column)` بيتجاهل الـ NULL في الـ column دي. الفرق ممكن يغير النتيجة جذرياً.

---

## GROUP BY و HAVING

```sql
-- عدد الـ products في كل status
SELECT status, COUNT(*) AS product_count
FROM products
GROUP BY status;

-- متوسط السعر لكل category
SELECT category_id,
       COUNT(*)        AS total_products,
       AVG(price)      AS avg_price,
       MIN(price)      AS min_price,
       MAX(price)      AS max_price
FROM products
GROUP BY category_id
ORDER BY avg_price DESC;

-- HAVING = WHERE بس بعد الـ GROUP BY
-- عايز بس الـ categories اللي عندها أكتر من 5 products
SELECT category_id, COUNT(*) AS cnt
FROM products
GROUP BY category_id
HAVING cnt > 5;

-- مزيج كامل
SELECT category_id, COUNT(*) AS cnt, AVG(price) AS avg_price
FROM products
WHERE status = 'active'          -- فلتر الصفوف قبل التجميع
GROUP BY category_id
HAVING avg_price > 1000          -- فلتر النتايج بعد التجميع
ORDER BY avg_price DESC
LIMIT 5;
```

الفرق بين `WHERE` و `HAVING` مهم جداً:

`WHERE` بتشتغل **قبل** الـ `GROUP BY` — بتفلتر الصفوف الأصلية. `HAVING` بتشتغل **بعد** الـ `GROUP BY` — بتفلتر النتايج المجمّعة.

---

## JOINs — دي الجوهرة```sql

-- ── INNER JOIN ───────────────────────────────────────────── -- بيجيب بس الـ orders اللي ليها customer موجود SELECT o.id AS order_id, c.first_name, c.last_name, o.total, o.status FROM orders o INNER JOIN customers c ON o.customer_id = c.id;

-- ── LEFT JOIN ────────────────────────────────────────────── -- كل الـ customers، حتى اللي معملوش orders SELECT c.id, c.first_name, COUNT(o.id) AS total_orders, -- 0 لو مفيش orders SUM(o.total) AS total_spent -- NULL لو مفيش orders FROM customers c LEFT JOIN orders o ON c.id = o.customer_id GROUP BY c.id, c.first_name;

-- ── LEFT JOIN + IS NULL = Anti-join ─────────────────────── -- الـ customers اللي ما عملوش أي order خالص SELECT c.* FROM customers c LEFT JOIN orders o ON c.id = o.customer_id WHERE o.id IS NULL;

-- ── Multi-table JOIN ─────────────────────────────────────── -- order details كاملة: customer + order + items + product SELECT c.first_name, c.last_name, o.id AS order_id, o.created_at AS order_date, p.name AS product_name, oi.quantity, oi.unit_price, (oi.quantity * oi.unit_price) AS line_total FROM orders o INNER JOIN customers c ON o.customer_id = c.id INNER JOIN order_items oi ON o.id = oi.order_id INNER JOIN products p ON oi.product_id = p.id WHERE o.status = 'delivered' ORDER BY o.created_at DESC;

-- ── SELF JOIN ────────────────────────────────────────────── -- كل employee مع اسم مديره SELECT e.name AS employee, m.name AS manager FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;

````

---

## UPDATE

```sql
-- تحديث صف واحد
UPDATE products
SET price = 14500.00
WHERE id = 1;

-- تحديث أكتر من column
UPDATE products
SET price  = 14500.00,
    stock  = stock + 10,    -- تعديل نسبي
    status = 'active'
WHERE id = 1;

-- تحديث بـ JOIN
UPDATE orders o
INNER JOIN customers c ON o.customer_id = c.id
SET o.status = 'cancelled'
WHERE c.email = 'ahmed@example.com'
  AND o.status = 'pending';

-- تحديث كل الصفوف (حذر!)
UPDATE products SET status = 'inactive';
````

**Gotcha خطير:** `UPDATE` من غير `WHERE` بيحدّث كل الصفوف! دايماً اتأكد من الـ `WHERE` قبل ما تنفّذ.

```sql
-- عادة بتعمل SELECT الأول عشان تشوف مين هيتأثر
SELECT * FROM products WHERE stock = 0;
-- لو النتيجة صح → بدّل SELECT بـ UPDATE
UPDATE products SET status = 'out_of_stock' WHERE stock = 0;
```

---

## DELETE

```sql
-- حذف صف واحد
DELETE FROM products WHERE id = 5;

-- حذف بشرط
DELETE FROM orders WHERE status = 'cancelled' AND created_at < '2023-01-01';

-- حذف بـ JOIN (MySQL specific)
DELETE o
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
WHERE c.email = 'spammer@example.com';

-- حذف كل شيء (بطيء — استخدم TRUNCATE بدلها)
DELETE FROM logs;
```

---

# 4️⃣ Subqueries

```sql
-- Subquery في الـ WHERE
-- المنتجات اللي سعرها أعلى من المتوسط
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Subquery مع IN
-- الـ customers اللي عملوا order في 2024
SELECT * FROM customers
WHERE id IN (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE YEAR(created_at) = 2024
);

-- Correlated Subquery — بيرجع لكل صف للـ outer query
-- كل customer مع آخر order ليه
SELECT c.first_name,
       (SELECT MAX(created_at)
        FROM orders o
        WHERE o.customer_id = c.id) AS last_order_date
FROM customers c;

-- EXISTS — أسرع من IN في الـ large datasets
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.id
      AND o.total > 1000
);
```

---

# 5️⃣ Indexes — السرعة في الـ Queries

الـ Index هو data structure إضافية MySQL بتعملها عشان تلاقي البيانات بسرعة بدل ما تفتش في كل الجدول.

```sql
-- Create index
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- Composite index — مهم جداً لـ queries بيها أكتر من column
CREATE INDEX idx_orders_cust_date ON orders(customer_id, created_at);
-- بيفيد queries زي: WHERE customer_id = 5 AND created_at > '2024-01-01'

-- Unique index
CREATE UNIQUE INDEX idx_customers_email ON customers(email);

-- الـ indexes الموجودة
SHOW INDEX FROM products;

-- ازاي تعرف الـ query بتستخدم index؟
EXPLAIN SELECT * FROM orders WHERE customer_id = 1;
-- بص على الـ column "key" — لو فاضية يعني مفيش index اتستخدم
```

**متى تعمل index؟** على الـ columns اللي بتستخدمها في `WHERE`, `JOIN ON`, `ORDER BY`. الـ columns اللي بيتبحث فيها كتير.

**متى لا تعمل index؟** على كل column — الـ index بياخد مساحة وبيبطّئ الـ INSERT/UPDATE/DELETE.

---

# 6️⃣ Transactions

```sql
START TRANSACTION;

-- خصم من حساب Ahmed
UPDATE accounts SET balance = balance - 500 WHERE user_id = 1;

-- أضف للحساب التاني
UPDATE accounts SET balance = balance + 500 WHERE user_id = 2;

-- لو الاتنين نجحوا
COMMIT;

-- لو في مشكلة — ارجع للحالة الأصلية
ROLLBACK;
```

الصح في الـ application:

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE user_id = 1;

-- تأكد إن الـ balance مش بقى سالب
SELECT balance INTO @bal FROM accounts WHERE user_id = 1;
IF @bal < 0 THEN
    ROLLBACK;
ELSE
    UPDATE accounts SET balance = balance + 500 WHERE user_id = 2;
    COMMIT;
END IF;
```

---

# 7️⃣ Advanced Queries — Window Functions

دي من أقوى features في MySQL 8+:

```sql
-- ROW_NUMBER — ترقيم الصفوف
SELECT
    name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS price_rank
FROM products;

-- RANK — نفس الترقيم بس لو سعرين متساويين بياخدوا نفس الرقم
SELECT
    name,
    price,
    RANK() OVER (ORDER BY price DESC) AS price_rank
FROM products;

-- PARTITION BY — ترقيم منفصل لكل category
SELECT
    name,
    category_id,
    price,
    RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS rank_in_category
FROM products;

-- Running total — مجموع تراكمي
SELECT
    created_at,
    total,
    SUM(total) OVER (ORDER BY created_at) AS running_total
FROM orders;

-- LAG / LEAD — قيمة الصف اللي قبله أو بعده
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month)  AS prev_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

---

# 8️⃣ Stored Procedures

```sql
DELIMITER $$

CREATE PROCEDURE GetCustomerOrders(IN p_customer_id INT)
BEGIN
    SELECT
        o.id,
        o.total,
        o.status,
        o.created_at
    FROM orders o
    WHERE o.customer_id = p_customer_id
    ORDER BY o.created_at DESC;
END$$

DELIMITER ;

-- استدعاء
CALL GetCustomerOrders(1);

-- Procedure مع OUT parameter
DELIMITER $$
CREATE PROCEDURE GetOrderTotal(
    IN  p_order_id INT,
    OUT p_total    DECIMAL(10,2)
)
BEGIN
    SELECT SUM(quantity * unit_price)
    INTO p_total
    FROM order_items
    WHERE order_id = p_order_id;
END$$
DELIMITER ;

-- استدعاء
CALL GetOrderTotal(5, @total);
SELECT @total;
```

---

# 9️⃣ Triggers

```sql
DELIMITER $$

-- بعد ما order جديدة اتضافت → خصم من الـ stock
CREATE TRIGGER after_order_item_insert
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock - NEW.quantity
    WHERE id = NEW.product_id;

    -- لو الـ stock بقى سالب → error
    IF (SELECT stock FROM products WHERE id = NEW.product_id) < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Insufficient stock';
    END IF;
END$$

-- قبل ما order تتحذف → أرجع الـ stock
CREATE TRIGGER before_order_item_delete
BEFORE DELETE ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock + OLD.quantity
    WHERE id = OLD.product_id;
END$$

DELIMITER ;
```

`NEW` = القيم الجديدة (في INSERT/UPDATE). `OLD` = القيم القديمة (في UPDATE/DELETE).

---

# 🔟 Views

```sql
-- View = query محفوظة بتتعامل معاها زي table
CREATE VIEW vw_order_summary AS
SELECT
    c.id          AS customer_id,
    c.first_name,
    c.last_name,
    c.email,
    COUNT(o.id)   AS total_orders,
    SUM(o.total)  AS total_spent,
    MAX(o.created_at) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.first_name, c.last_name, c.email;

-- استخدامها زي table عادي
SELECT * FROM vw_order_summary WHERE total_spent > 5000;
SELECT * FROM vw_order_summary ORDER BY total_orders DESC LIMIT 10;

-- حذف view
DROP VIEW IF EXISTS vw_order_summary;
```

---

# الـ Gotchas الأكبر في MySQL 🔥

```sql
-- 1. Division by zero مش بيعمل error — بيرجع NULL!
SELECT 10 / 0;   -- → NULL (مش error!)

-- 2. String comparison غير case-sensitive بالـ default
SELECT * FROM users WHERE email = 'AHMED@EXAMPLE.COM';
-- بيلاقي 'ahmed@example.com' — لأن الـ collation بالـ default غير case-sensitive

-- 3. Implicit type conversion ممكن تعمل مشاكل
SELECT * FROM orders WHERE customer_id = '1abc';
-- MySQL بيحوّل '1abc' لـ 1 من غير error!

-- 4. GROUP BY في MySQL أكثر تساهلاً من Standard SQL
-- ده بيشتغل في MySQL بس ممكن يرجع نتايج غلط:
SELECT customer_id, name, COUNT(*) FROM orders GROUP BY customer_id;
-- name مش في الـ GROUP BY ولا aggregate function — standard SQL هيرفض!

-- 5. ORDER BY بعد UNION لازم تحطها على الـ query الكاملة
(SELECT name FROM products) UNION (SELECT name FROM categories)
ORDER BY name;  -- ✅

-- 6. NULL في كل عمليات المقارنة بيرجع NULL (مش TRUE أو FALSE)
SELECT NULL = NULL;    -- → NULL (مش TRUE!)
SELECT NULL != NULL;   -- → NULL
SELECT NULL IS NULL;   -- → TRUE ✅
```

---

# ملخص الترتيب المنطقي لأي Query

```sql
SELECT   columns          -- 5: إيه اللي هيتعرض
FROM     table            -- 1: الجدول الأصلي
JOIN     other ON ...     -- 2: ربط الجداول
WHERE    condition        -- 3: فلتر الصفوف
GROUP BY columns          -- 4: تجميع
HAVING   condition        -- 4.5: فلتر المجموعات
ORDER BY columns          -- 6: ترتيب النتيجة
LIMIT    n OFFSET m;      -- 7: عدد الصفوف
```

الـ MySQL بتنفّذهم بالترتيب ده: `FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT` — مش بترتيب الكتابة.

---

كده عندك MySQL كامل من DDL لحد الـ Window Functions والـ Triggers. تحب نعمل **تمارين advanced** على حاجة معينة، ولا نشتغل على scenario كامل زي نظام orders من الصفر بـ SQL ؟