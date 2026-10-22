# Structured Query Language (SQL) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master query execution mechanics, advanced joins, window functions, indexing internals (B+ Trees), ACID transactions, normalization, and top interview query patterns.

---

## Table of Contents
1. [Under the Hood: SQL Query Execution Lifecycle](#1-under-the-hood-sql-query-execution-lifecycle)
   - [The Logical Query Processing Order](#the-logical-query-processing-order)
   - [Query Parsing, Optimization & Storage Engine Execution](#query-parsing-optimization--storage-engine-execution)
2. [SQL Sub-Languages (DDL, DML, DQL, DCL, TCL)](#2-sql-sub-languages-ddl-dml-dql-dcl-tcl)
3. [Keys & Referential Integrity Constraints](#3-keys--referential-integrity-constraints)
   - [Super Key vs. Candidate Key vs. Primary Key](#super-key-vs-candidate-key-vs-primary-key)
   - [Primary Key vs. Unique Key](#primary-key-vs-unique-key)
   - [Foreign Keys & Cascading Actions](#foreign-keys--cascading-actions)
4. [Joins Masterclass: Mechanics & Internal Algorithms](#4-joins-masterclass-mechanics--internal-algorithms)
   - [Visual & Set Theory Breakdown of All Joins](#visual--set-theory-breakdown-of-all-joins)
   - [Internal Join Algorithms: Nested Loop vs. Hash Join vs. Merge Join](#internal-join-algorithms)
   - [Self Joins & Cross Joins](#self-joins--cross-joins)
5. [Aggregations: GROUP BY, HAVING, and WHERE](#5-aggregations-group-by-having-and-where)
6. [Advanced Window (Analytic) Functions](#6-advanced-window-analytic-functions)
   - [`ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()`](#rownumber-vs-rank-vs-denserank)
   - [`LEAD()` and `LAG()` for Time-Series & Consecutive Event Analysis](#lead-and-lag)
   - [Window Framing: `ROWS BETWEEN`](#window-framing-rows-between)
7. [Subqueries, CTEs (Common Table Expressions) & Views](#7-subqueries-ctes-and-views)
   - [Correlated vs. Uncorrelated Subqueries](#correlated-vs-uncorrelated-subqueries)
   - [Recursive CTEs (Hierarchical & Graph Traversal)](#recursive-ctes)
   - [Views vs. Materialized Views](#views-vs-materialized-views)
8. [Database Indexing & B+ Trees (Crucial SDE-1 Topic)](#8-database-indexing--b-trees-crucial-sde-1-topic)
   - [Clustered vs. Non-Clustered (Secondary) Index](#clustered-vs-non-clustered-secondary-index)
   - [Why B+ Trees over Binary Search Trees or Hash Tables?](#why-b-trees)
   - [Composite Indexes & The Leftmost Prefix Rule](#composite-indexes--the-leftmost-prefix-rule)
   - [Covering Indexes & Index Cardinality](#covering-indexes--index-cardinality)
9. [Transactions, ACID & Concurrency Control](#9-transactions-acid--concurrency-control)
   - [ACID Properties Under the Hood (WAL, Undo/Redo Logs)](#acid-properties-under-the-hood)
   - [Read Anomalies: Dirty Read, Non-Repeatable Read, Phantom Read](#read-anomalies)
   - [The 4 Transaction Isolation Levels](#the-4-transaction-isolation-levels)
   - [MVCC (Multi-Version Concurrency Control) & Optimistic vs. Pessimistic Locking](#mvcc--locking)
10. [Database Normalization & Denormalization](#10-database-normalization--denormalization)
    - [1NF, 2NF, 3NF, and BCNF (Step-by-Step Schemas)](#1nf-2nf-3nf-and-bcnf)
    - [When and Why to Denormalize in Production](#when-and-why-to-denormalize)
11. [Top 10 Classic Interview SQL Query Challenges](#11-top-10-classic-interview-sql-query-challenges)
12. [Top 20 Conceptual Interview Questions & Model Answers](#12-top-20-conceptual-interview-questions--model-answers)

---

## 1. Under the Hood: SQL Query Execution Lifecycle

### The Logical Query Processing Order
*One of the most frequently asked foundational questions in SDE-1 interviews.*  
Unlike programming languages that execute top-to-bottom, SQL queries are declarative and are evaluated in a specific logical order:

```
WRITTEN SYNTAX:                           LOGICAL EXECUTION ORDER:
SELECT DISTINCT col1, col2               1. FROM & JOINs (cartesian product & filters)
FROM TableA                              2. ON (join conditions applied)
JOIN TableB ON condition                 3. WHERE (row-level filtering)
WHERE row_condition                      4. GROUP BY (aggregate row grouping)
GROUP BY col1                            5. HAVING (group-level filtering)
HAVING group_condition                   6. SELECT (projection of expressions & aliases)
ORDER BY col1 ASC                        7. DISTINCT (duplicate elimination)
LIMIT 10 OFFSET 5;                       8. ORDER BY (sorting results)
                                         9. LIMIT / OFFSET (pagination)
```

> **Why this matters in interviews:**
> - You **cannot** use a `SELECT` column alias inside a `WHERE` clause because `WHERE` executes *before* `SELECT` has evaluated aliases!
> - You **can** use a `SELECT` alias inside an `ORDER BY` clause because `ORDER BY` executes *after* `SELECT`.

---

### Query Parsing, Optimization & Storage Engine Execution
When you execute an SQL query in a database like PostgreSQL or MySQL (InnoDB):

```
Client Application ---> Query String: "SELECT * FROM users WHERE age > 21"
                             |
                             v
1. PARSER & LEXER: Checks syntax errors, validates table & column names.
                             |
2. QUERY OPTIMIZER: Evaluates statistics (histograms, index cardinality).
   Chooses between Index Scan vs. Full Table Scan, Join orders, Hash vs. Merge Join.
   Generates the EXPLAIN Execution Plan.
                             |
3. EXECUTION ENGINE: Issues low-level disk/buffer commands.
                             |
4. STORAGE ENGINE (e.g., InnoDB):
   - Checks Buffer Pool (RAM Cache).
   - Reads data pages (B+ Tree traversal).
   - Writes to Write-Ahead Log (WAL / Redo Log) for crash recovery.
```

---

## 2. SQL Sub-Languages (DDL, DML, DQL, DCL, TCL)

```
+-------------------------------------------------------------------------------------------------+
|                                    SQL SUB-LANGUAGES TAXONOMY                                   |
+-------------------+----------------------------+-----------------------+------------------------+
| Category          | Purpose                    | Core Commands         | Auto-Commit Behavior   |
+-------------------+----------------------------+-----------------------+------------------------+
| **DDL** (Data     | Defines & alters database  | `CREATE`, `ALTER`,    | **Implicitly Commits** |
| Definition)       | schema structure.          | `DROP`, `TRUNCATE`,   | (Cannot be rolled back |
|                   |                            | `RENAME`              | in most databases)     |
+-------------------+----------------------------+-----------------------+------------------------+
| **DML** (Data     | Manipulates existing data  | `INSERT`, `UPDATE`,   | Explicit transaction   |
| Manipulation)     | records within tables.     | `DELETE`              | (Can be rolled back)   |
+-------------------+----------------------------+-----------------------+------------------------+
| **DQL** (Data     | Retrieves & queries data   | `SELECT`              | Read-only              |
| Query)            | without altering state.    |                       | (No commit needed)     |
+-------------------+----------------------------+-----------------------+------------------------+
| **DCL** (Data     | Controls user permissions  | `GRANT`, `REVOKE`     | Security & privilege   |
| Control)          | and access rights.         |                       | management             |
+-------------------+----------------------------+-----------------------+------------------------+
| **TCL** (Transac- | Manages transactional      | `COMMIT`, `ROLLBACK`, | Manages ACID boundary  |
| tion Control)     | units of work.             | `SAVEPOINT`           | state transitions      |
+-------------------+----------------------------+-----------------------+------------------------+
```

> **Interview Trap: `DELETE` vs. `TRUNCATE` vs. `DROP`**
> - `DELETE` (DML): Deletes specific rows matching a `WHERE` condition. Row-by-row deletion; logs each deleted row in the Undo/Redo log. Fires triggers; slower. Can be rolled back.
> - `TRUNCATE` (DDL): Deallocates entire physical data pages. Fast; minimal transaction logging; resets `AUTO_INCREMENT` counters. Does not fire row triggers.
> - `DROP` (DDL): Completely destroys table schema, indexes, constraints, and data from the database catalog.

---

## 3. Keys & Referential Integrity Constraints

### Super Key vs. Candidate Key vs. Primary Key

```
+--------------------------------------------------------+
|                      SUPER KEYS                        |
|  Any set of columns that uniquely identifies a row.    |
|  e.g., {SSN}, {SSN, Name}, {SSN, Email, Age}           |
|                                                        |
|         +------------------------------------+         |
|         |          CANDIDATE KEYS            |         |
|         |  Minimal Super Keys (No redundant  |         |
|         |  attributes). e.g., {SSN}, {Email} |         |
|         |                                    |         |
|         |        +------------------+        |         |
|         |        |   PRIMARY KEY    |        |         |
|         |        | Chosen Candidate |        |         |
|         |        | Key ({SSN})      |        |         |
|         |        +------------------+        |         |
|         +------------------------------------+         |
+--------------------------------------------------------+
```

---

### Primary Key vs. Unique Key

| Feature | Primary Key (PK) | Unique Key (UK) |
| :--- | :--- | :--- |
| **Nullability** | **Strictly `NOT NULL`**. Never accepts null values. | **Allows `NULL`** (SQL standard permits multiple `NULL`s since `NULL != NULL`). |
| **Quantity** | Exactly **one** Primary Key per table. | A table can have **multiple** Unique Keys. |
| **Default Index** | Automatically creates a **Clustered Index** (in MySQL/SQL Server). | Automatically creates a **Non-Clustered (Secondary) Index**. |
| **Purpose** | Uniquely identifies physical entity. | Enforces data uniqueness on business attributes (e.g., `email`, `phone`). |

---

### Foreign Keys & Cascading Actions
A Foreign Key establishes a parent-child relationship between two tables, enforcing referential integrity.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
        ON DELETE CASCADE      -- If user is deleted, their orders are deleted automatically
        ON UPDATE CASCADE      -- If user_id changes, update orders automatically
);
```

**Referential Action Options:**
1. `ON DELETE CASCADE`: Deleting parent deletes all referencing child rows.
2. `ON DELETE SET NULL`: Sets child foreign key column to `NULL` (requires column to be nullable).
3. `ON DELETE RESTRICT / NO ACTION`: Blocks deletion of parent row if child rows exist (throws error).
4. `ON DELETE SET DEFAULT`: Sets foreign key to its defined default value.

---

## 4. Joins Masterclass: Mechanics & Internal Algorithms

### Visual & Set Theory Breakdown of All Joins

```
   TABLE A (Left)                TABLE B (Right)
   +----+-------+                +----+-------+
   | id | name  |                | id | dept  |
   +----+-------+                +----+-------+
   |  1 | Alice |                |  1 | Eng   |
   |  2 | Bob   |                |  3 | HR    |
   +----+-------+                +----+-------+

1. INNER JOIN (Intersection):
   Matches rows present in BOTH tables.
   Result: [1, Alice, Eng]

2. LEFT (OUTER) JOIN:
   All rows from Left + matched from Right (NULL if no match).
   Result: [1, Alice, Eng], [2, Bob, NULL]

3. RIGHT (OUTER) JOIN:
   All rows from Right + matched from Left (NULL if no match).
   Result: [1, Alice, Eng], [3, NULL, HR]

4. FULL OUTER JOIN:
   Union of Left and Right tables (NULL where match missing).
   Result: [1, Alice, Eng], [2, Bob, NULL], [3, NULL, HR]

5. CROSS JOIN (Cartesian Product):
   Combines every row of A with every row of B. (Size = M * N).
```

---

### Internal Join Algorithms
*When an interviewer asks: "How does the database engine physically execute a JOIN?"*

```
+-------------------------------------------------------------------------------------------------+
|                                 PHYSICAL JOIN ALGORITHMS IN RDBMS                               |
+-------------------+----------------------------+-----------------------+------------------------+
| Algorithm         | How It Works               | Time Complexity       | Best When              |
+-------------------+----------------------------+-----------------------+------------------------+
| **Nested Loop**   | For each row in Table A,   | $O(M \times N)$ or    | One table is tiny and  |
| **Join**          | scan Table B for matches.  | $O(M \log N)$ w/index | the join column in the |
|                   |                            |                       | other has an index.    |
+-------------------+----------------------------+-----------------------+------------------------+
| **Hash Join**     | 1. Build Phase: Hash small | $O(M + N)$            | Large tables, equijoins|
|                   |    table into RAM hash map.|                       | (`=`), no indexes      |
|                   | 2. Probe Phase: Stream big |                       | present.               |
|                   |    table, lookup in hash.  |                       |                        |
+-------------------+----------------------------+-----------------------+------------------------+
| **Sort-Merge**    | 1. Sort both tables on key.| $O(M\log M + N\log N)$| Join columns are       |
| **Join**          | 2. Merge sorted streams    | or $O(M + N)$ if      | already sorted via an  |
|                   |    linearly like 2 pointers| already indexed.      | index or inequality.   |
+-------------------+----------------------------+-----------------------+------------------------+
```

---

### Self Joins & Cross Joins

#### Self Join Example: Finding Employee and Their Manager
```sql
SELECT 
    e.employee_name AS Employee,
    m.employee_name AS Manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

---

## 5. Aggregations: GROUP BY, HAVING, and WHERE

### WHERE vs. HAVING

```
+--------------------------------------------------------------------------+
|                            WHERE vs. HAVING                              |
+-------------------+----------------------------+-------------------------+
| Feature           | WHERE                      | HAVING                  |
+-------------------+----------------------------+-------------------------+
| Filtering Level   | Individual rows **before** | Aggregated groups       |
|                   | grouping occurs.           | **after** grouping.     |
| Aggregate Funcs   | **Disallowed**             | **Allowed**             |
| Allowed?          | (`WHERE COUNT(*) > 5` ❌)  | (`HAVING COUNT(*) > 5`✅)|
| Index Utilization | Can use B+ Tree indexes to | Operates on grouped in- |
|                   | filter rows fast.          | memory temporary sets.  |
| Execution Order   | Step 3 (Early)             | Step 5 (Late)           |
+-------------------+----------------------------+-------------------------+
```

```sql
-- Find departments with more than 5 employees whose average salary > $80,000
SELECT 
    department_id,
    COUNT(*) AS total_employees,
    AVG(salary) AS avg_salary
FROM employees
WHERE is_active = TRUE               -- Row-level filter (WHERE)
GROUP BY department_id
HAVING COUNT(*) > 5 AND AVG(salary) > 80000; -- Group-level filter (HAVING)
```

---

## 6. Advanced Window (Analytic) Functions

Window functions perform calculations across a set of table rows that are related to the current row, **without collapsing rows into a single summary output** (unlike `GROUP BY`).

### `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()`
*This is asked in 95% of FAANG/product company SQL interview rounds.*

Given duplicate scores: `[100, 100, 90, 80]`

```
+-------+--------------+--------+--------------+
| Score | ROW_NUMBER() | RANK() | DENSE_RANK() |
+-------+--------------+--------+--------------+
|   100 |            1 |      1 |            1 |
|   100 |            2 |      1 |            1 |
|    90 |            3 |      3 |            2 |  <-- Note gap in RANK()!
|    80 |            4 |      4 |            3 |
+-------+--------------+--------+--------------+
```

- **`ROW_NUMBER()`:** Assigns a strictly unique, sequential integer to every row ($1, 2, 3, 4$).
- **`RANK()`:** Gives identical values the same rank, but **leaves gaps** for the next rank ($1, 1, 3, 4$).
- **`DENSE_RANK()`:** Gives identical values the same rank, and **never leaves gaps** ($1, 1, 2, 3$).

```sql
-- Query: Find the top 3 highest paid employees in EACH department
WITH RankedSalaries AS (
    SELECT 
        employee_id,
        department_id,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department_id 
            ORDER BY salary DESC
        ) as salary_rank
    FROM employees
)
SELECT department_id, employee_id, salary
FROM RankedSalaries
WHERE salary_rank <= 3;
```

---

### `LEAD()` and `LAG()`
Access data from a subsequent (`LEAD`) or preceding (`LAG`) row without performing a self-join.

```sql
-- Calculate month-over-month revenue growth
SELECT 
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY month) AS prev_month_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY month) AS mom_growth
FROM monthly_sales;
```

---

### Window Framing: `ROWS BETWEEN`
Defines the dynamic sliding window of rows relative to the current row.

```sql
-- 7-Day Moving Average Calculation
SELECT 
    sale_date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS seven_day_moving_avg
FROM daily_revenue;
```

---

## 7. Subqueries, CTEs, and Views

### Correlated vs. Uncorrelated Subqueries
- **Uncorrelated Subquery:** Independent of the outer query. Evaluates **once** for the entire query execution.
  ```sql
  -- Independent: Subquery runs once
  SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
  ```
- **Correlated Subquery:** References columns from the outer query. Evaluates **once per every candidate row** of the outer query (can degrade to $O(N^2)$ without proper indexing!).
  ```sql
  -- Correlated: Evaluated for each employee 'e'
  SELECT e.employee_id, e.salary
  FROM employees e
  WHERE e.salary > (
      SELECT AVG(sub.salary) 
      FROM employees sub 
      WHERE sub.department_id = e.department_id
  );
  ```

---

### Recursive CTEs
Used for hierarchical and graph data (organization charts, category trees, flight path graphs).

```sql
-- Find complete management chain for employee ID 105
WITH RECURSIVE ManagementChain AS (
    -- Anchor member: Target employee
    SELECT employee_id, manager_id, employee_name, 1 AS level
    FROM employees
    WHERE employee_id = 105

    UNION ALL

    -- Recursive member: Join with parent manager
    SELECT m.employee_id, m.manager_id, m.employee_name, mc.level + 1
    FROM employees m
    INNER JOIN ManagementChain mc ON m.employee_id = mc.manager_id
)
SELECT * FROM ManagementChain;
```

---

## 8. Database Indexing & B+ Trees (Crucial SDE-1 Topic)

### Clustered vs. Non-Clustered (Secondary) Index

```
CLUSTERED INDEX (The Table IS the Index):
Leaves contain ACTUAL ROW DATA:
                [ Root Node ]
               /             \
       [ Internal ]       [ Internal ]
        /        \         /        \
     [Page 1]  [Page 2]  [Page 3]  [Page 4]  <-- Leaf nodes = Physical data blocks
     (1..100) (101..200)(201..300)(301..400) (Ordered physically on disk)

NON-CLUSTERED INDEX (Secondary Index on 'email'):
Leaves contain the Indexed Column + Clustered Key Pointer:
                [ Root Node ]
               /             \
        [ Leaf: email + PK ]  <-- Points to PK (e.g. "alice@work" -> ID 42)
                 |
                 v (Double Lookup / Bookmark Lookup)
        [ Clustered Index Lookup to fetch full row ]
```

| Feature | Clustered Index | Non-Clustered Index |
| :--- | :--- | :--- |
| **Physical Storage** | Dictates the physical disk order of table data. | Stored in a separate structure; points back to table rows. |
| **Quantity** | **Only 1** per table (data can only be sorted one way).| **Multiple** per table (typically up to dozens). |
| **Leaf Node Stores** | The **complete row data** pages. | The indexed column value + the **Clustered Key (PK)**. |
| **Lookup Cost** | Direct: 1 B+ tree traversal to reach row. | Double Lookup: Traversals secondary index, then traverses clustered index to fetch full row. |

---

### Why B+ Trees over Binary Search Trees or Hash Tables?
*Interviewers ask: "Why don't relational databases use Hash Tables or Red-Black BSTs for indexing?"*

1. **Why not Hash Index?**
   * Hash indexes provide $O(1)$ lookup for exact equality (`WHERE id = 5`), but are **completely useless for range queries** (`WHERE age BETWEEN 20 AND 30` or `WHERE date > '2026-01-01'`) and `ORDER BY` sorting.
2. **Why not Binary Search Trees (BST / AVL / Red-Black)?**
   * BSTs have low fan-out (each node has only 2 children), causing large tree heights ($O(\log_2 N)$). A table with 10 million rows would require 24 disk seeks!
3. **Why B+ Trees?**
   * **High Fan-Out:** Each node can hold hundreds/thousands of keys per disk block (4KB/16KB page). Tree height is typically only **3 to 4 levels**, reducing disk I/O to 3–4 seeks for billions of rows.
   * **Linked Leaves for Fast Range Scans:** All leaf nodes in a B+ tree are linked in a **doubly linked list**. A range query finds the start key in $O(\log N)$ and simply traverses the linked list sequentially at disk streaming speeds!

---

### Composite Indexes & The Leftmost Prefix Rule
When you create an index on multiple columns: `CREATE INDEX idx_user ON users(country, state, city);`

The database sorts the B+ tree by:
1. `country` first
2. then by `state`
3. then by `city`

#### The Rule:
The query optimizer **can only use the index if queries filter by the leftmost prefix**:
- `WHERE country = 'US'` $\rightarrow$ **Uses Index** ✅
- `WHERE country = 'US' AND state = 'CA'` $\rightarrow$ **Uses Index** ✅
- `WHERE country = 'US' AND state = 'CA' AND city = 'SF'` $\rightarrow$ **Uses Index** ✅
- `WHERE state = 'CA'` $\rightarrow$ **CANNOT use index (Full Table Scan)** ❌
- `WHERE city = 'SF'` $\rightarrow$ **CANNOT use index (Full Table Scan)** ❌

---

### Covering Indexes
A covering index is a non-clustered index that contains **all the columns requested by a query**.
```sql
-- Query:
SELECT user_id, email FROM users WHERE email = 'test@example.com';

-- Index:
CREATE INDEX idx_email ON users(email); 
-- In InnoDB, secondary indexes automatically include the PK (user_id)!
```
*Result:* The engine satisfies the query **entirely from the index leaf node** without having to perform an expensive bookmark lookup into the clustered index table!

---

## 9. Transactions, ACID & Concurrency Control

A **Transaction** is a logical unit of work comprising one or more SQL statements that must either completely succeed or completely fail.

### ACID Properties Under the Hood
1. **Atomicity ("All or Nothing"):** If any statement fails, the entire transaction is rolled back.
   * *Mechanism:* **Undo Logs** (records old values before mutation to reverse changes).
2. **Consistency:** Guarantees database moves from one valid state to another, satisfying all schema constraints (FK, UK, CHECK).
3. **Isolation:** Concurrent transactions execute without interfering with one another.
   * *Mechanism:* Locks (Pessimistic) or **MVCC** (Optimistic).
4. **Durability:** Once committed, changes survive system crashes or power failures.
   * *Mechanism:* **Write-Ahead Logging (WAL) / Redo Log**. Changes are flushed to sequential disk logs *before* dirty pages are written to RAM buffer pools.

---

### Read Anomalies

```
+--------------------------------------------------------------------------+
|                        CONCURRENCY READ ANOMALIES                        |
+-------------------+------------------------------------------------------+
| Anomaly           | Description & Scenario                               |
+-------------------+------------------------------------------------------+
| **Dirty Read**    | Transaction A reads uncommitted data written by      |
|                   | Transaction B. If B rolls back, A read phantom data! |
+-------------------+------------------------------------------------------+
| **Non-Repeatable**| Transaction A reads row X. Transaction B modifies or |
| **Read**          | deletes row X and commits. Transaction A re-reads row|
|                   | X and sees a different value!                        |
+-------------------+------------------------------------------------------+
| **Phantom Read**  | Transaction A queries rows matching a condition      |
|                   | (`WHERE age > 30` -> 5 rows). Transaction B inserts  |
|                   | a new matching row and commits. Transaction A re-runs|
|                   | query and finds 6 rows!                              |
+-------------------+------------------------------------------------------+
```

---

### The 4 Transaction Isolation Levels

```
+-------------------------------------------------------------------------------------------------+
|                                 TRANSACTION ISOLATION LEVELS                                    |
+-------------------+--------------------+-----------------------+--------------------+-----------+
| Isolation Level   | Dirty Read Allowed?| Non-Repeatable Read?  | Phantom Read?      | Speed     |
+-------------------+--------------------+-----------------------+--------------------+-----------+
| **Read Uncommitted**| Yes (Dangerous!) | Yes                   | Yes                | Fastest   |
| **Read Committed**  | **No**             | Yes                   | Yes                | Fast (PG/ |
|                   |                    |                       |                    | Oracle def)|
| **Repeatable Read** | **No**             | **No**                | Yes (MySQL prevents| Balanced  |
|                   |                    |                       | via Next-Key locks)| (MySQL def|
| **Serializable**  | **No**             | **No**                | **No**             | Slowest   |
+-------------------+--------------------+-----------------------+--------------------+-----------+
```

---

### MVCC (Multi-Version Concurrency Control)
Modern databases (PostgreSQL, MySQL InnoDB) use MVCC to achieve high concurrency:
> **Golden Principle:** *"Readers never block Writers, and Writers never block Readers."*

- Every row stores hidden metadata: `created_by_tx_id` and `deleted_by_tx_id`.
- When an `UPDATE` occurs, the engine does **not** overwrite the existing row in place. It marks the old row as deleted by the current transaction ID and inserts a brand new version of the row with the updated values.
- Transactions read row snapshots corresponding to their start timestamp, ignoring uncommitted or newer row versions.

---

## 10. Database Normalization & Denormalization

Normalization organizes database schemas to **eliminate data redundancy** and prevent **insertion, update, and deletion anomalies**.

```
Unnormalized Data (Redundancy, anomalies)
       |
       v  1NF: Atomic values only (no comma-separated lists), Unique rows.
       |
       v  2NF: In 1NF + No Partial Functional Dependencies (All non-key cols
       |       depend on the FULL Composite Primary Key).
       |
       v  3NF: In 2NF + No Transitive Dependencies (Non-key cols cannot
       |       depend on other non-key cols).
       |
       v  BCNF (Boyce-Codd Normal Form): Strict 3NF (For every functional
          dependency X -> Y, X must be a Super Key).
```

### Schema Walkthrough: 1NF to 3NF

#### Unnormalized Table:
`Orders(OrderID, CustomerName, CustomerCity, Items)`  
*Problem:* `Items` contains `"Laptop, Mouse"` (violates atomicity).

#### 1. First Normal Form (1NF):
Split multivalued attributes so every cell holds an **atomic (indivisible) value**:
`OrderItems(OrderID, ItemName, Quantity, CustomerName, CustomerCity)`

#### 2. Second Normal Form (2NF):
Composite Primary Key: `{OrderID, ItemName}`.  
`CustomerName` and `CustomerCity` depend only on `OrderID`, not `ItemName` (**Partial Dependency!**).  
*Decomposition into 2NF:*
- `Orders(OrderID, CustomerID, CustomerName, CustomerCity)`
- `OrderItems(OrderID, ItemName, Quantity)`

#### 3. Third Normal Form (3NF):
In `Orders`, `OrderID -> CustomerID` and `CustomerID -> CustomerCity`.  
Here, `CustomerCity` depends on `CustomerID` (a non-key column), which depends on `OrderID` (**Transitive Dependency!**).  
*Decomposition into 3NF:*
- `Customers(CustomerID, CustomerName, CustomerCity)`
- `Orders(OrderID, CustomerID, OrderDate)`
- `OrderItems(OrderID, ItemID, Quantity)`

---

## 11. Top 10 Classic Interview SQL Query Challenges

### 1. Find the Nth Highest Salary (Without using LIMIT/OFFSET)
```sql
SELECT DISTINCT salary
FROM employees e1
WHERE N - 1 = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);

-- Alternative using Window Function (Industry Standard):
WITH Ranked AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM employees
)
SELECT salary FROM Ranked WHERE rnk = :N LIMIT 1;
```

---

### 2. Find Employees Who Earn More Than Their Direct Managers
```sql
SELECT e.employee_name
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

---

### 3. Find Duplicate Emails in a Table
```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### 4. Delete Duplicate Records (Keeping Only the Row with the Smallest ID)
```sql
DELETE FROM users
WHERE user_id NOT IN (
    SELECT min_id FROM (
        SELECT MIN(user_id) AS min_id
        FROM users
        GROUP BY email
    ) AS temp
);
```

---

### 5. Find Consecutive Active Logins (Users who logged in 3+ consecutive days)
```sql
WITH RankedLogins AS (
    SELECT 
        user_id,
        login_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) as rn
    FROM user_logins
),
GroupedConsecutive AS (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL rn DAY) AS streak_group
    FROM RankedLogins
)
SELECT user_id, COUNT(*) AS consecutive_days
FROM GroupedConsecutive
GROUP BY user_id, streak_group
HAVING COUNT(*) >= 3;
```

---

### 6. Calculate Running Total (Cumulative Sum)
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM transactions;
```

---

### 7. Second Highest Salary (Handling Edge Cases with NULL)
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

### 8. Find Customers Who Never Placed an Order
```sql
SELECT c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

---

### 9. Exchange Seats / Swap Consecutive Rows
```sql
SELECT 
    CASE 
        WHEN id % 2 = 1 AND id = (SELECT COUNT(*) FROM seat) THEN id
        WHEN id % 2 = 1 THEN id + 1
        ELSE id - 1
    END AS id,
    student
FROM seat
ORDER BY id ASC;
```

---

### 10. Pivot / Row-to-Column Aggregation (Total Sales by Year & Quarter)
```sql
SELECT 
    year,
    SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS Q1_Sales,
    SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS Q2_Sales,
    SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS Q3_Sales,
    SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS Q4_Sales
FROM quarterly_revenue
GROUP BY year;
```

---

## 12. Top 20 Conceptual Interview Questions & Model Answers

### Q1: What is the difference between `WHERE` and `HAVING`?
**Answer:** `WHERE` filters individual rows *before* any grouping or aggregation takes place; it cannot contain aggregate functions (`WHERE SUM(val) > 10` is invalid). `HAVING` filters aggregated groups *after* the `GROUP BY` clause has processed rows; it is designed specifically to evaluate aggregate expressions (`HAVING COUNT(*) > 5`).

---

### Q2: Why is `SELECT *` considered an anti-pattern in production code?
**Answer:**
1. **Unnecessary Disk & Network I/O:** Transmits unused columns across the database socket.
2. **Defeats Covering Indexes:** Forces the engine to perform bookmark lookups on disk instead of answering queries purely from RAM index leaf pages.
3. **Breaks Application DTOs:** If a column is added or reordered, client code expecting fixed columns may throw mapping exceptions.

---

### Q3: What is the difference between `UNION` and `UNION ALL`?
**Answer:** `UNION` combines result sets and performs an **expensive sorting/de-duplication phase** to eliminate duplicate rows. `UNION ALL` simply concatenates result sets together without checking for duplicates. `UNION ALL` is significantly faster and should always be preferred when results are known to be distinct or duplicates are acceptable.

---

### Q4: What is the difference between a Clustered and a Non-Clustered Index?
**Answer:** A clustered index determines the physical storage order of the rows in the table; therefore, a table can have only one clustered index, and its leaf nodes contain the actual complete row data. A non-clustered index is an auxiliary data structure; its leaf nodes contain the indexed column values paired with a row pointer (the clustered key), requiring a secondary bookmark lookup to retrieve full records.

---

### Q5: What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?
**Answer:** For identical duplicate values:
- `ROW_NUMBER()` assigns strictly unique sequential integers without ties ($1, 2, 3$).
- `RANK()` assigns identical ranks to ties, but leaves gaps in subsequent rankings ($1, 1, 3$).
- `DENSE_RANK()` assigns identical ranks to ties, and does not skip any rankings ($1, 1, 2$).

---

### Q6: Can a table have more than one Primary Key?
**Answer:** No. A table can have **only one Primary Key**. However, a Primary Key can be composed of multiple columns, which is called a **Composite Primary Key**.

---

### Q7: What is the difference between `TRUNCATE` and `DELETE`?
**Answer:** `DELETE` is a DML statement that removes rows one by one, logs each deletion in the transaction log, fires triggers, and can filter with `WHERE`. `TRUNCATE` is a DDL statement that deallocates entire data pages, bypassing individual row logging; it cannot use `WHERE`, resets auto-increment sequences, does not fire row triggers, and is significantly faster.

---

### Q8: What is a Phantom Read, and which isolation level prevents it?
**Answer:** A Phantom Read occurs when Transaction A reads a range of rows satisfying a condition (`WHERE price < 100`), Transaction B inserts a new matching row and commits, and Transaction A re-executes the query and sees new "phantom" rows. It is prevented by the **Serializable** isolation level (or in MySQL InnoDB at **Repeatable Read** using Next-Key gap locking).

---

### Q9: What is the Leftmost Prefix Rule in Composite Indexes?
**Answer:** In a multi-column composite index `(A, B, C)`, the index is sorted by A, then B, then C. The query engine can only utilize the index if the query filters on the leftmost leading column (`A`, or `A and B`, or `A and B and C`). If a query filters only on `B` or `C`, the composite index cannot be used efficiently because the data is not sorted globally by those attributes alone.

---

### Q10: What is a View vs. a Materialized View?
**Answer:** A standard **View** is a saved, virtual query; it holds no data on disk and re-executes its underlying SQL statement every time it is referenced. A **Materialized View** physically executes the query and persists the result set on disk as a concrete table; it must be periodically refreshed, but delivers near-instantaneous read performance for heavy aggregation queries.

---

### Q11: What is a Foreign Key constraint? What does `ON DELETE CASCADE` do?
**Answer:** A Foreign Key ensures referential integrity by linking a column in a child table to the primary key of a parent table, guaranteeing that orphaned child rows cannot exist. `ON DELETE CASCADE` specifies that when a parent record is deleted, all referencing child rows in the foreign key table are automatically deleted as part of the same transaction.

---

### Q12: How does Write-Ahead Logging (WAL) guarantee Durability in ACID?
**Answer:** Disk random writes to data pages are slow. Under WAL, before any dirty data page is written to table storage, the exact delta of the change is appended sequentially to the WAL / Redo log on disk. If the server crashes or loses power mid-transaction, during recovery the engine replays the sequential WAL records to restore committed data to RAM and disk.

---

### Q13: What is the difference between an Inner Join and a Left Outer Join?
**Answer:** An Inner Join returns only rows where there is a matching key in both tables. A Left Outer Join returns all rows from the left table; if a row has no corresponding match in the right table, the right-table columns are filled with `NULL`.

---

### Q14: What is the difference between Optimistic and Pessimistic Locking?
**Answer:**
- **Pessimistic Locking:** Assumes concurrent collisions will happen; locks rows using `SELECT ... FOR UPDATE`, blocking other transactions until release. Best for high-contention systems (e.g., banking).
- **Optimistic Locking:** Assumes collisions are rare; does not lock rows on read. Instead, checks a `version` column before updating (`WHERE id = 1 AND version = 5`). If version changed, transaction aborts and retries. Best for read-heavy systems.

---

### Q15: Why is `NULL = NULL` evaluated to False (or Unknown) in SQL?
**Answer:** In SQL three-valued logic (True, False, Unknown), `NULL` represents an "unknown or missing value." You cannot assert that two unknown values are identical because they could represent completely different realities. To check for nullability, SQL provides the dedicated operator `IS NULL` or `IS NOT NULL`.

---

### Q16: What is Database Denormalization, and why is it used?
**Answer:** Denormalization is the intentional introduction of controlled redundancy into a normalized schema (e.g., duplicating a user's name directly in an `Orders` table). It is used in read-heavy applications to avoid expensive multi-table `JOIN` operations, reducing latency and disk I/O at the expense of extra storage and more complex update logic.

---

### Q17: What is an Execution Plan (`EXPLAIN`)? What do you look for?
**Answer:** `EXPLAIN` displays the query execution plan chosen by the query optimizer. When optimizing queries, engineers look for:
1. **Type:** `ALL` (Full Table Scan - bad) vs. `range` / `ref` / `eq_ref` (Index Scan - good).
2. **Key:** Which index (if any) was chosen.
3. **Rows:** Estimated number of rows scanned (smaller is better).
4. **Extra:** Avoiding `Using filesort` or `Using temporary`.

---

### Q18: What is the difference between `COALESCE()` and `IFNULL()` / `NVL()`?
**Answer:** `IFNULL(a, b)` (MySQL) and `NVL(a, b)` (Oracle) accept only two arguments and return `b` if `a` is NULL. `COALESCE(a, b, c, ...)` is an ANSI SQL standard function that accepts an arbitrary number of arguments and returns the **first non-null value** in the sequence.

---

### Q19: What is the difference between a Correlated and an Uncorrelated Subquery?
**Answer:** An uncorrelated subquery is independent of the outer query, evaluating exactly once. A correlated subquery depends on values from the candidate row of the outer query, re-evaluating once for every row processed by the outer query, which can cause significant performance degradation on large tables unless rewritten as a `JOIN`.

---

### Q20: What happens if you index a column with very low cardinality (e.g., boolean `is_active`)?
**Answer:** The query optimizer will likely **ignore the index and perform a Full Table Scan**. Because a boolean column has only two distinct values, traversing a secondary index and performing millions of bookmark lookups is slower and more disk I/O-intensive than reading the contiguous data pages sequentially via a full scan.
