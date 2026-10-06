
# 🗄️ SQL Interview Roadmap

## 🟢 1. SQL Fundamentals — الأساسيات

-  **What is SQL?**
    
-  **SQL Command Types**
    
    - DDL
        
    - DML
        
    - DQL
        
    - DCL
        
    - TCL
        
-  **Constraints**
    
    - PRIMARY KEY
        
    - FOREIGN KEY
        
    - UNIQUE
        
    - NOT NULL
        
    - CHECK
        
-  **Data Types**
    
    - CHAR vs VARCHAR
        
    - Numeric types
        
    - Date/Time types
        
-  **Basic Queries**
    
    - SELECT
        
    - WHERE
        
    - DISTINCT
        
    - ORDER BY
        
    - LIMIT / TOP
        
-  **NULL**
    
    - IS NULL
        
    - IS NOT NULL
        
    - COALESCE
        
    - NULL behavior
        
-  **DELETE vs TRUNCATE vs DROP**
    
-  **UNION vs UNION ALL**
    
-  **SQL Query Execution Order**
    
    - FROM
        
    - WHERE
        
    - GROUP BY
        
    - HAVING
        
    - SELECT
        
    - ORDER BY
        

---

# 🟡 2. JOINs & Aggregation

### JOINs

-  INNER JOIN
    
-  LEFT JOIN
    
-  RIGHT JOIN
    
-  FULL OUTER JOIN
    
-  CROSS JOIN
    
-  SELF JOIN
    
-  JOIN vs Subquery
    

### Aggregation

-  GROUP BY
    
-  HAVING vs WHERE
    
-  COUNT
    
-  SUM
    
-  AVG
    
-  MIN
    
-  MAX
    
-  COUNT(*) vs COUNT(column)
    

---

# 🟠 3. Intermediate SQL

### Subqueries

-  What is a Subquery?
    
-  Scalar Subquery
    
-  Multi-row Subquery
    
-  Correlated Subquery
    
-  IN
    
-  EXISTS
    
-  NOT EXISTS
    
-  EXISTS vs IN
    

### CTE

-  What is CTE?
    
-  WITH
    
-  Multiple CTEs
    
-  Recursive CTE
    
-  Hierarchical Data
    
-  Employee → Manager hierarchy
    

---

# 🔵 4. Window Functions

-  What are Window Functions?
    
-  PARTITION BY
    
-  ORDER BY inside Window
    
-  ROW_NUMBER()
    
-  RANK()
    
-  DENSE_RANK()
    
-  Difference:
    
    - ROW_NUMBER vs RANK
        
    - RANK vs DENSE_RANK
        
-  Running Total
    
-  Moving Average
    
-  LAG()
    
-  LEAD()
    
-  FIRST_VALUE()
    
-  LAST_VALUE()
    

---

# 🟣 5. Indexes

-  What is an Index?
    
-  Why Indexes improve SELECT performance
    
-  Index trade-offs
    
    - Faster reads
        
    - Slower INSERT/UPDATE/DELETE
        
    - Extra storage
        
-  Clustered Index
    
-  Non-Clustered Index
    
-  Composite Index
    
-  Column order in Composite Index
    
-  Covering Index
    
-  When an Index is NOT useful
    
-  Index on high vs low cardinality columns
    
-  Index + Execution Plan
    

---

# 🟤 6. Views & Stored Procedures

### Views

-  What is a View?
    
-  Why use Views?
    
-  Normal View
    
-  Materialized View
    
-  View vs Materialized View
    
-  Performance implications
    
-  When to use Materialized View
    

### Stored Procedures

-  What is Stored Procedure?
    
-  Stored Procedure vs Function
    
-  Parameters
    
-  Use cases
    

---

# 🔴 7. Transactions & ACID

-  What is a Transaction?
    
-  COMMIT
    
-  ROLLBACK
    
-  SAVEPOINT
    
-  ACID
    

### ACID

-  **Atomicity** → كله يحصل أو كله يتعمل Rollback
    
-  **Consistency** → DB تفضل في حالة صحيحة
    
-  **Isolation** → Transactions لا تتداخل بشكل غير صحيح
    
-  **Durability** → بعد COMMIT البيانات لا تضيع
    

### Isolation Levels

-  Read Uncommitted
    
-  Read Committed
    
-  Repeatable Read
    
-  Serializable
    
-  Dirty Read
    
-  Non-Repeatable Read
    
-  Phantom Read
    

---

# ⚫ 8. Locks & Deadlocks

-  What is Lock?
    
-  Shared Lock
    
-  Exclusive Lock
    
-  Row-level Lock
    
-  Table-level Lock
    
-  What is Deadlock?
    
-  How Deadlocks happen?
    
-  How to prevent Deadlocks?
    
    - Consistent order
        
    - Short transactions
        
    - Proper indexes
        
    - Avoid unnecessary locks
        

---

# 🟧 9. Database Design

### Normalization

-  What is Normalization?
    
-  1NF
    
-  2NF
    
-  3NF
    
-  Why normalize?
    
-  Denormalization
    
-  Normalization vs Denormalization
    

### Relationships

-  One-to-One
    
-  One-to-Many
    
-  Many-to-Many
    
-  Junction / Join Table
    
-  Foreign Keys
    

---

# 🚀 10. Performance & Query Optimization

-  What is Query Optimization?
    
-  Execution Plan
    
-  EXPLAIN
    
-  EXPLAIN ANALYZE
    
-  Index usage
    
-  Full Table Scan
    
-  Index Scan / Index Seek
    
-  Avoid `SELECT *`
    
-  Avoid unnecessary JOINs
    
-  Pagination
    
-  N+1 Query Problem
    
-  Query bottlenecks
    
-  Database statistics
    

---

# 🌍 11. Scaling Databases

-  Vertical Scaling
    
-  Horizontal Scaling
    
-  Read Replicas
    
-  Replication
    
-  Partitioning
    
-  Sharding
    
-  Partitioning vs Sharding
    
-  Database Load Balancing
    

---

# 🔐 12. SQL Security

-  SQL Injection
    
-  Parameterized Queries
    
-  Prepared Statements
    
-  Input Validation
    
-  Principle of Least Privilege
    
-  Database Users & Permissions
    

مثال:

❌ Dangerous:

```sql
SELECT * FROM users
WHERE username = '...';
```

الأفضل استخدام **Parameterized Query** بدل دمج input المستخدم مباشرة في SQL.

---

# 🧠 13. Practical SQL Interview Problems ⭐

دي من أهم الأجزاء:

-  Find the highest salary
    
-  Find the **second highest salary**
    
-  Find the **Nth highest salary**
    
-  Find top 3 salaries per department
    
-  Find duplicate records
    
-  Delete duplicate records
    
-  Find employees with no department
    
-  Find departments with no employees
    
-  Find employees earning more than their manager
    
-  Find employees earning above department average
    
-  Find the highest salary in each department
    
-  Find the second highest salary in each department
    
-  Running total
    
-  Cumulative sum
    
-  Find consecutive records
    
-  Find missing records
    
-  Find customers who never placed an order
    
-  Find customers who placed multiple orders
    
-  Top N per group
    
-  Pagination
    
-  Date-based queries
    

---

# 🎯 ترتيب المذاكرة المقترح

لو هدفك **Backend Interview**، متذاكرش كل حاجة بنفس الوزن.

```text
1️⃣ SQL Basics
       ↓
2️⃣ JOINs + GROUP BY
       ↓
3️⃣ Subqueries + CTE
       ↓
4️⃣ Window Functions ⭐
       ↓
5️⃣ Indexes ⭐
       ↓
6️⃣ Transactions + ACID ⭐
       ↓
7️⃣ Locks + Deadlocks
       ↓
8️⃣ Normalization
       ↓
9️⃣ Query Optimization ⭐
       ↓
🔟 Partitioning + Sharding
       ↓
1️⃣1️⃣ SQL Security
       ↓
1️⃣2️⃣ Practical Problems ⭐⭐⭐
```

### أهم 5 حاجات للـ Backend Interview

لو عايز **الزتونة جدًا**:

> **JOINs + Aggregation → Subqueries/CTEs → Window Functions → Indexes/Execution Plans → Transactions/ACID**

وبعدهم ركز على **Practical SQL Problems**؛ لأن هنا بيبان فعلًا هل أنت بتعرف تكتب SQL ولا مجرد حافظ definitions.