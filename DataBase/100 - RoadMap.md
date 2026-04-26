
---

# 🟢 LEVEL 0 — Database Fundamentals (أساسيات قواعد البيانات)

قبل MySQL لازم تفهم المفاهيم دي:

1. What is Database (Done)
    
2. DBMS vs RDBMS (Done)
    
3. Relational Databases  (Done)
    
4. Tables / Rows / Columns (Done)
    
5. Primary Key (Done)
    
6. Foreign Key (Done)
    
7. Relationships (Done)
    
    - One to One
        
    - One to Many
        
    - Many to Many
        
8. Data Types basics (Done)
    
9. Normalization basics
    
    - 1NF
        
    - 2NF
        
    - 3NF
        

---

# 🟢 LEVEL 1 — MySQL Basics

### Installation & Tools

- Install MySQL Server
    
- MySQL Workbench
    
- CLI (mysql client)
    
- Connect to DB
    

---

## Databases & Tables

```sql
CREATE DATABASE
DROP DATABASE
USE database
SHOW DATABASES
```

```sql
CREATE TABLE
DROP TABLE
ALTER TABLE
DESCRIBE table
```

---

## Data Types (مهم جداً)

### Numeric

- INT
    
- BIGINT
    
- DECIMAL
    
- FLOAT / DOUBLE
    
- TINYINT
    

### String

- CHAR
    
- VARCHAR
    
- TEXT
    
- LONGTEXT
    

### Date & Time

- DATE
    
- DATETIME
    
- TIMESTAMP
    
- TIME
    
- YEAR
    

### Other

- ENUM
    
- JSON
    

---

# 🟢 LEVEL 2 — CRUD Operations

## INSERT

```sql
INSERT INTO
INSERT MULTIPLE ROWS
INSERT DEFAULT VALUES
```

## SELECT (أهم جزء)

```sql
SELECT *
SELECT specific columns
DISTINCT
ALIAS (AS)
```

## WHERE

```sql
=  !=  <>  >  <  >=  <=
AND / OR / NOT
BETWEEN
IN
LIKE
IS NULL
```

## UPDATE

```sql
UPDATE table SET column=value WHERE condition
```

## DELETE

```sql
DELETE FROM table WHERE condition
TRUNCATE TABLE
```

---

# 🟢 LEVEL 3 — Sorting & Limiting

- ORDER BY ASC / DESC
    
- LIMIT
    
- OFFSET
    
- Pagination basics
    

---

# 🟢 LEVEL 4 — Aggregate Functions

🔥 مهم جداً

- COUNT()
    
- SUM()
    
- AVG()
    
- MIN()
    
- MAX()
    

---

## GROUP BY

```sql
GROUP BY column
HAVING (very important)
```

---

# 🟢 LEVEL 5 — JOINS (أهم جزء في SQL)

## Types of JOIN

1. INNER JOIN
    
2. LEFT JOIN
    
3. RIGHT JOIN
    
4. CROSS JOIN
    
5. SELF JOIN
    

---

# 🟢 LEVEL 6 — Advanced Filtering

- Subqueries (Nested Queries)
    
    - Scalar Subquery
        
    - Correlated Subquery
        
- EXISTS / NOT EXISTS
    
- ANY / ALL
    

---

# 🟡 LEVEL 7 — Indexes & Performance

🔥 دي مرحلة الاحتراف

## Index Basics

- Why indexes
    
- B-Tree index
    
- Clustered vs Non-clustered
    
- Composite index
    
- Unique index
    
- Full-text index
    

```sql
CREATE INDEX
DROP INDEX
EXPLAIN
```

---

# 🟡 LEVEL 8 — Constraints

- PRIMARY KEY
    
- FOREIGN KEY
    
- UNIQUE
    
- NOT NULL
    
- CHECK
    
- DEFAULT
    

## Cascading

- ON DELETE CASCADE
    
- ON UPDATE CASCADE
    
- RESTRICT / SET NULL
    

---

# 🟡 LEVEL 9 — Transactions

🔥 مهم جداً في المشاريع

- ACID
    
- COMMIT
    
- ROLLBACK
    
- SAVEPOINT
    
- Autocommit
    

## Isolation Levels

- Read Uncommitted
    
- Read Committed
    
- Repeatable Read
    
- Serializable
    

---

# 🟡 LEVEL 10 — Views

```sql
CREATE VIEW
UPDATE VIEW
DROP VIEW
Materialized concept
```

---

# 🟡 LEVEL 11 — Stored Programs

## Stored Procedures

```sql
CREATE PROCEDURE
IN / OUT parameters
CALL procedure
```

## Functions

```sql
CREATE FUNCTION
RETURN values
```

## Triggers

```sql
BEFORE INSERT
AFTER INSERT
BEFORE UPDATE
AFTER DELETE
```

---

# 🔴 LEVEL 12 — Advanced SQL

## Window Functions 🔥🔥🔥

- ROW_NUMBER()
    
- RANK()
    
- DENSE_RANK()
    
- LEAD()
    
- LAG()
    
- PARTITION BY
    

---

## CTE (Common Table Expressions)

```sql
WITH query AS (...)
```

Recursive CTE 🔥

---

# 🔴 LEVEL 13 — JSON in MySQL

- JSON data type
    
- JSON_EXTRACT
    
- JSON_SET
    
- JSON_ARRAY
    
- JSON_OBJECT
    

---

# 🔴 LEVEL 14 — Security

- Users & Privileges
    

```sql
CREATE USER
GRANT
REVOKE
```

- Roles
    
- Encryption
    
- SQL Injection basics
    

---

# 🔴 LEVEL 15 — Backup & Recovery

- mysqldump
    
- Import / Export
    
- Replication basics
    

---

# 🔴 LEVEL 16 — Replication & Scaling (Advanced جداً)

- Master / Slave replication
    
- Read replicas
    
- Sharding basics
    
- Partitioning tables
    

---

# 🔴 LEVEL 17 — Performance Tuning 🔥🔥🔥

- Query optimization
    
- Execution plan
    
- Slow query log
    
- Index tuning
    
- Denormalization
    
- Caching strategies
    

---

# 🎯 دي كل Topics MySQL من 0 → Advanced جداً

لو درستهم فعلاً تبقى:

- جاهز interviews
    
- جاهز backend production
    
- جاهز system design basics
    

---

لو حابب نبدأ مذاكرة SQL مع بعض خطوة خطوة ونطبق عملي، نبدأ من **JOIN + HAVING + Subqueries** لأنك قلت إنهم اللي عليك 👌