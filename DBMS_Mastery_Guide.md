# Database Management Systems (DBMS) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master database engine architecture, relational algebra, serializability, concurrency protocols (2PL), recovery (ARIES/WAL), distributed storage (CAP/PACELC), and SQL vs. NoSQL trade-offs.

---

## Table of Contents
1. [DBMS Architecture & Foundations](#1-dbms-architecture--foundations)
   - [File System vs. DBMS (The Core Motivation)](#file-system-vs-dbms-the-core-motivation)
   - [3-Schema Architecture (ANSI/SPARC)](#3-schema-architecture-ansisparc)
   - [Physical vs. Logical Data Independence](#physical-vs-logical-data-independence)
2. [Data Models & Relational Algebra](#2-data-models--relational-algebra)
   - [Relational Model Terminology (Degree, Cardinality, Domain)](#relational-model-terminology)
   - [Relational Algebra Operators ($\sigma, \pi, \times, \cup, -, \bowtie$)](#relational-algebra-operators)
   - [ER Modeling to Relational Schema Conversion Rules](#er-modeling-to-relational-schema-conversion-rules)
3. [Transaction Processing & Serializability](#3-transaction-processing--serializability)
   - [Transaction States Lifecycle](#transaction-states-lifecycle)
   - [Concurrent Schedules (Serial vs. Non-Serial)](#concurrent-schedules)
   - [Conflict Serializability & Precedence Graphs](#conflict-serializability--precedence-graphs)
   - [View Serializability & Blind Writes](#view-serializability--blind-writes)
   - [Recoverability: Recoverable vs. Cascadeless vs. Strict](#recoverability-of-schedules)
4. [Concurrency Control Protocols](#4-concurrency-control-protocols)
   - [Locking Fundamentals (Shared vs. Exclusive)](#locking-fundamentals)
   - [Two-Phase Locking (2PL): Basic, Strict, Rigorous, Conservative](#two-phase-locking-2pl)
   - [Timestamp Ordering Protocol & Thomas Write Rule](#timestamp-ordering-protocol)
   - [Multiple Granularity & Intention Locks (IS, IX, SIX)](#multiple-granularity--intention-locks)
5. [Database Recovery Management](#5-database-recovery-management)
   - [Failure Classifications (Crash, Disk, Transaction)](#failure-classifications)
   - [Log-Based Recovery: Deferred vs. Immediate Update](#log-based-recovery)
   - [Checkpointing & The ARIES Recovery Algorithm](#checkpointing--the-aries-recovery-algorithm)
   - [Shadow Paging](#shadow-paging)
6. [Storage & File Organization](#6-storage--file-organization)
   - [Heap vs. Sorted File Organization](#heap-vs-sorted-file-organization)
   - [Static vs. Dynamic Hashing (Extendible & Linear Hashing)](#static-vs-dynamic-hashing)
7. [Distributed Databases & Modern Storage Paradigms](#7-distributed-databases--modern-storage-paradigms)
   - [SQL vs. NoSQL (Document, Key-Value, Wide-Column, Graph)](#sql-vs-nosql)
   - [ACID vs. BASE Philosophy](#acid-vs-base-philosophy)
   - [The CAP Theorem (Why CA is a Myth in Distributed Systems)](#the-cap-theorem)
   - [The PACELC Theorem](#the-pacelc-theorem)
   - [Replication Strategies & Quorum Consensus ($R + W > N$)](#replication-strategies--quorum-consensus)
   - [Database Sharding & Consistent Hashing](#database-sharding--consistent-hashing)
8. [Top 20 SDE-1 / Intern DBMS Interview Questions & Model Answers](#8-top-20-sde-1--intern-dbms-interview-questions--model-answers)

---

## 1. DBMS Architecture & Foundations

### File System vs. DBMS (The Core Motivation)
Why did the industry move from flat file systems (like CSVs or text files) to a Database Management System?

```
+-------------------------------------------------------------------------------------------------+
|                                    FILE SYSTEM vs. DBMS                                         |
+-------------------+------------------------------------+----------------------------------------+
| Feature           | Traditional File System            | Database Management System (DBMS)      |
+-------------------+------------------------------------+----------------------------------------+
| **Data Redundancy**| High. Same data duplicated across | Minimized via Normalization and        |
|                   | multiple department files.         | centralized schema definitions.        |
| **Data Integrity**| Hard to enforce (e.g., negative    | Hardware-enforced constraints          |
|                   | balances require custom app code). | (`CHECK`, `FOREIGN KEY`, `NOT NULL`).  |
| **Concurrency**   | Poor. File-level locks block       | Granular concurrency (Row-level,       |
|                   | concurrent reads and writes.       | page-level locks, MVCC).               |
| **Crash Recovery**| Catastrophic. Power loss corrupts  | Robust. Write-Ahead Logging (WAL)      |
|                   | entire files midway through write. | guarantees ACID durability & rollback. |
| **Data Security** | OS file-level permissions (rwx)    | Fine-grained access control (DCL:      |
|                   | only; cannot restrict columns.     | column-level grants, row policies).    |
+-------------------+------------------------------------+----------------------------------------+
```

---

### 3-Schema Architecture (ANSI/SPARC)
The ANSI/SPARC architecture separates the user application from the physical database:

```
               [ User View 1 ]       [ User View 2 ]       [ User View 3 ]
                      \                     |                     /
+-------------------------------------------------------------------------------------+
| 1. EXTERNAL LEVEL (View Level):                                                     |
|    - How individual users perceive data (tailored views, hide sensitive columns).   |
+-------------------------------------------------------------------------------------+
                                      |
                      [ Logical Data Independence ]
                                      |
+-------------------------------------------------------------------------------------+
| 2. CONCEPTUAL LEVEL (Logical Level):                                                |
|    - WHAT data is stored in the database and the relationships between entities.    |
|    - Defines tables, columns, data types, constraints, and security rules.          |
+-------------------------------------------------------------------------------------+
                                      |
                      [ Physical Data Independence ]
                                      |
+-------------------------------------------------------------------------------------+
| 3. INTERNAL LEVEL (Physical Storage Level):                                         |
|    - HOW data is physically stored on SSD/HDD.                                      |
|    - Manages data pages (4KB/16KB), B+ tree indexes, compression, and file blocks.  |
+-------------------------------------------------------------------------------------+
                                      |
                           [ PHYSICAL HARDWARE DISK ]
```

---

### Physical vs. Logical Data Independence

```
+--------------------------------------------------------------------------+
|                 PHYSICAL vs. LOGICAL DATA INDEPENDENCE                   |
+-------------------+--------------------------+---------------------------+
| Dimension         | Physical Data Independ.  | Logical Data Independence |
+-------------------+--------------------------+---------------------------+
| Boundary          | Between Conceptual and   | Between External (Views)  |
|                   | Internal levels.         | and Conceptual levels.    |
| Definition        | Ability to modify the    | Ability to modify the     |
|                   | physical storage schema  | logical schema without    |
|                   | without changing logical | breaking external views   |
|                   | tables/queries.          | or client applications.   |
| Real-World        | Switching from HDD to    | Adding a new column to a  |
| Scenario          | NVMe SSD, changing B+    | table; existing user      |
|                   | tree page size, adding   | queries requesting other  |
|                   | a secondary index.       | columns continue to work. |
| Difficulty        | **Easy to achieve**      | **Difficult to achieve**  |
|                   | (Automated by engine).   | (Requires view mapping).  |
+-------------------+--------------------------+---------------------------+
```

---

## 2. Data Models & Relational Algebra

### Relational Model Terminology
- **Relation:** A mathematical table of rows and columns.
- **Tuple:** A single record / row in a relation.
- **Attribute:** A named column in a relation.
- **Domain:** The set of permissible atomic values for an attribute (e.g., `Age INT CHECK (Age >= 0)`).
- **Degree (Arity):** The total number of **attributes (columns)** in a relation.
- **Cardinality:** The total number of **tuples (rows)** in a relation.

```
Relation: STUDENTS (Degree = 3, Cardinality = 2)
+-------------+---------------+----------+  <-- Attributes (Columns)
| RollNo (PK) | Name          | Age      |
+-------------+---------------+----------+
| 101         | Alice         | 21       |  <-- Tuple 1 (Row)
| 102         | Bob           | 22       |  <-- Tuple 2 (Row)
+-------------+---------------+----------+
```

---

### Relational Algebra Operators
Relational Algebra is the formal procedural query language that forms the theoretical backbone of SQL execution plans.

```
+-------------------------------------------------------------------------------------------------+
|                                   RELATIONAL ALGEBRA OPERATORS                                  |
+-------------------+--------+---------------------------------------+----------------------------+
| Operation         | Symbol | Description                           | Equivalent SQL Clause      |
+-------------------+--------+---------------------------------------+----------------------------+
| **Selection**     | $\sigma$| Filters tuples that satisfy predicate | `SELECT * FROM T WHERE P`  |
| **Projection**    | $\pi$  | Selects specific columns (eliminates  | `SELECT col1, col2 FROM T` |
|                   |        | duplicate rows by set definition)     |                            |
| **Cartesian**     | $\times$| Pairs every tuple of $R$ with every  | `SELECT * FROM R CROSS JOIN`|
| **Product**       |        | tuple of $S$                          | `S`                        |
| **Natural Join**  | $\bowtie$| Equijoin on all attributes with the | `SELECT * FROM R NATURAL`  |
|                   |        | same name; eliminates duplicate cols  | `JOIN S`                   |
| **Union**         | $\cup$ | Combines tuples from two union-       | `SELECT * FROM R UNION`    |
|                   |        | compatible relations                  | `SELECT * FROM S`          |
| **Set Difference**| $-$    | Tuples in relation $R$ that are NOT   | `EXCEPT` / `MINUS`         |
|                   |        | present in relation $S$               |                            |
| **Intersection**  | $\cap$ | Tuples present in BOTH $R$ and $S$    | `INTERSECT`                |
+-------------------+--------+---------------------------------------+----------------------------+
```

#### Relational Algebra Query Example:
*Find the names of all students enrolled in the 'CSE' department:*
$$\pi_{\text{Name}}(\sigma_{\text{Dept} = 'CSE'}(\text{Students} \bowtie \text{Departments}))$$

---

### ER Modeling to Relational Schema Conversion Rules
*A staple question in campus technical rounds.*

```
1. Strong Entity Set       ---> Separate Table (Entity PK becomes Table PK).
2. Weak Entity Set         ---> Separate Table (Foreign Key from Owner Entity +
                                 Partial Key / Discriminator forms Composite PK).
3. 1:1 Relationship        ---> Merge into one table OR place PK of one table as
                                 Foreign Key (with UNIQUE constraint) in the other.
4. 1:N Relationship        ---> Place PK of "1-side" entity as Foreign Key inside
                                 the table of the "N-side" entity.
5. M:N Relationship        ---> MUST create a NEW Junction / Bridge Table containing
                                 the PKs of both entities as a Composite PK.
6. Multivalued Attribute   ---> Separate Table containing (Owner PK + Attribute Value).
```

---

## 3. Transaction Processing & Serializability

### Transaction States Lifecycle

```
               [ ACTIVE ] (Initial state; executing read/write statements)
                /      \
               /        \ Failure occurs or abort requested
              v          v
   [ PARTIALLY COMMITTED ]   [ FAILED ]
              |                  |
   Final write flushed to WAL    v
              v             [ ABORTED ] (Rolled back to initial state
         [ COMMITTED ]                   using Undo Log)
      (Changes permanent;
       cannot be reversed)
              |
              v
        [ TERMINATED ]
```

---

### Concurrent Schedules: Serial vs. Non-Serial
- **Schedule ($S$):** An execution sequence of operations from multiple concurrent transactions.
- **Serial Schedule:** Transactions execute strictly one after another ($T_1$ completely finishes before $T_2$ begins).
  * *Pros:* 100% consistent; zero concurrency anomalies.
  * *Cons:* Terrible throughput; CPU idle during I/O waits.
- **Non-Serial Schedule:** Operations of multiple transactions are interleaved to maximize CPU & disk concurrency.

---

### Conflict Serializability & Precedence Graphs
A non-serial schedule is **Conflict Serializable** if it can be transformed into an equivalent serial schedule by swapping non-conflicting adjacent operations.

#### What makes two operations Conflicting?
Two operations $I_i$ and $I_j$ conflict **IF AND ONLY IF ALL THREE conditions are met**:
1. They belong to **different transactions** ($T_i \neq T_j$).
2. They operate on the **exact same data item** ($Q$).
3. At least one of the operations is a **WRITE** (`W(Q)`).

```
Read(X)  and  Read(X)  ---> Non-Conflicting (Both are reading)
Read(X)  and  Write(X) ---> CONFLICTING! (Read-Write conflict)
Write(X) and  Read(X)  ---> CONFLICTING! (Write-Read conflict)
Write(X) and  Write(X) ---> CONFLICTING! (Write-Write conflict)
```

#### Precedence Graph (Serialization Graph) Algorithm:
1. Create a vertex for each transaction $T_i$.
2. Draw a directed edge $T_i \rightarrow T_j$ if $T_i$ executes an operation that conflicts with an operation executed later by $T_j$.
3. **The Golden Theorem:** *A schedule $S$ is conflict serializable IF AND ONLY IF its precedence graph contains **NO CYCLES**.*
4. If acyclic, topological sort gives the equivalent serial schedule order.

```
Example:
T1: R(A), W(A)
T2:           R(A), W(A)

Conflicting pair: W1(A) before R2(A)  ===> Edge: T1 -> T2
No reverse edge exists ===> Graph is Acyclic ===> CONFLICT SERIALIZABLE!
```

---

### View Serializability & Blind Writes
- View Serializability is a broader definition than Conflict Serializability.
- **Every Conflict Serializable schedule is View Serializable**, but the reverse is not always true.
- A schedule that is View Serializable but NOT Conflict Serializable **MUST contain at least one Blind Write** (a write operation performed on an item without reading it first: `W(X)` without prior `R(X)`).
- Testing for View Serializability is an **NP-Complete problem**.

---

### Recoverability of Schedules

```
                       SCHEDULE RECOVERABILITY HIERARCHY
                       
                       +-------------------------------+
                       |      ALL SCHEDULES            |
                       |  +-------------------------+  |
                       |  |   Recoverable           |  |
                       |  |  +-------------------+  |  |
                       |  |  |  Cascadeless      |  |  |
                       |  |  |  +-------------+  |  |  |
                       |  |  |  | Strict      |  |  |  |
                       |  |  |  +-------------+  |  |  |
                       |  |  +-------------------+  |  |
                       |  +-------------------------+  |
                       +-------------------------------+
```

1. **Recoverable Schedule:** If transaction $T_j$ reads data written by $T_i$, then $T_i$ **must commit before** $T_j$ commits. (Prevents reading uncommitted data that might subsequently be aborted).
2. **Cascadeless Schedule:** A transaction can only read data items that have **already been committed** by other transactions. (Eliminates the **Cascading Abort problem**, where the rollback of one transaction forces dozens of dependent transactions to abort).
3. **Strict Schedule:** A transaction can neither read **nor write** a data item until the previous transaction that wrote to it has committed or aborted. (Simplifies recovery to simple undo logging).

---

## 4. Concurrency Control Protocols

### Locking Fundamentals
- **Shared Lock ($S$ / Read Lock):** Multiple transactions can hold a shared lock on the same item concurrently (`LOCK-S(A)`). Allows reading, blocks writing.
- **Exclusive Lock ($X$ / Write Lock):** Only one transaction can hold an exclusive lock on an item (`LOCK-X(A)`). Allows both reading and writing; blocks all other locks.

---

### Two-Phase Locking (2PL)
2PL is a concurrency control protocol that **guarantees conflict serializability**.

```
Lock Points
     ^
     |              [ GROWING PHASE ]              |        [ SHRINKING PHASE ]
Lock |     - Transaction acquires locks            | - Transaction releases locks
Count|     - CANNOT release any locks              | - CANNOT acquire any new locks
     |                                             |
     |                   /-------------------------\
     |                  /                           \
     |                 /                             \
     |                /                               \
     +---------------+---------------------------------+------------------------> Time
                     Lock Point (Peak locks acquired)
```

#### Variations of 2PL:
1. **Basic 2PL:** Growing phase $\rightarrow$ Shrinking phase. Guarantees conflict serializability, but **can suffer from deadlocks and cascading rollbacks**.
2. **Strict 2PL (Most widely implemented):** All **Exclusive ($X$) locks** must be held until the transaction **commits or aborts**. Guarantees cascadeless recovery!
3. **Rigorous 2PL:** All **Shared ($S$) and Exclusive ($X$) locks** must be held until the transaction commits or aborts.
4. **Conservative 2PL (Static 2PL):** Transaction must declare and acquire **all its needed locks upfront** before execution begins. If any lock is unavailable, none are acquired. **Guarantees zero deadlocks**, but drastically reduces concurrency.

---

### Timestamp Ordering Protocol
Instead of locks, each transaction $T_i$ is assigned a unique system timestamp $TS(T_i)$ when it begins (e.g., using system clock or logical counter).

Each data item $Q$ maintains two timestamps:
- **$W\text{-timestamp}(Q)$:** Largest timestamp of any transaction that successfully executed `Write(Q)`.
- **$R\text{-timestamp}(Q)$:** Largest timestamp of any transaction that successfully executed `Read(Q)`.

#### Thomas Write Rule (Optimization):
If $T_i$ issues `Write(Q)` and $TS(T_i) < W\text{-timestamp}(Q)$, then a newer transaction has already overwritten $Q$. Instead of aborting $T_i$, the protocol **simply ignores / discards $T_i$'s outdated write**, allowing the transaction to continue!

---

### Multiple Granularity & Intention Locks
To allow locking at different levels of the hierarchy (Database $\rightarrow$ Table $\rightarrow$ Page $\rightarrow$ Row):

```
                       [ Database Level ]
                               |
                       [ Table Level ]
                               |
                       [ Page Level ]
                               |
                       [ Row / Record Level ]
```

- **Intention Shared (IS):** Explicit locking is being done at a lower level with shared locks.
- **Intention Exclusive (IX):** Explicit locking is being done at a lower level with exclusive locks.
- **Shared Intention Exclusive (SIX):** The current node is locked in shared mode, and lower levels are locked in exclusive mode.
*Benefit:* To lock an entire table with an exclusive lock, the engine checks if the table has an `IS` or `IX` lock without having to inspect millions of individual row records!

---

## 5. Database Recovery Management

### Failure Classifications
1. **Transaction Failure:** Logical error (e.g., divide by zero, constraint violation) or system cancel (deadlock resolution abort).
2. **System Crash:** Hardware malfunction, OS kernel panic, or power cut. Volatile RAM data is lost, but non-volatile disk data is preserved.
3. **Disk Failure (Catastrophic):** Head crash or bad disk sectors. Data must be restored from offsite backups and archive logs.

---

### Log-Based Recovery: Deferred vs. Immediate Update

- **Deferred Update (NO-UNDO / REDO):**
  * Changes are recorded only in the log in memory during execution.
  * Physical database on disk is updated **only after the transaction successfully commits**.
  * *Crash Recovery:* Never need to Undo anything (uncommitted changes were never written to disk). Only requires **REDO** for committed transactions.
- **Immediate Update (UNDO / REDO):**
  * Database on disk can be modified while the transaction is still active.
  * *Crash Recovery:* Requires **UNDO** for uncommitted transactions that wrote to disk, and **REDO** for committed transactions whose changes were still in volatile buffer cache.

---

### Checkpointing & The ARIES Recovery Algorithm

#### Why is Checkpointing needed?
Without checkpoints, during a crash recovery the database engine would have to scan the Write-Ahead Log from the **first day the database was created**, taking hours or days to boot!

```
Log Sequence:
... [Checkpoint Begin] ... [Transactions Active] ... [Checkpoint End] ... CRASH!
                                                           ^
                                             Only need to scan log
                                             from here during reboot!
```

#### The 3 Phases of ARIES Recovery:
1. **Analysis Phase:** Scans the log forward from the last checkpoint to identify:
   * All active transactions at the time of crash (the **Losers Set** that must be undone).
   * All dirty pages in the buffer pool.
2. **Redo Phase:** Scans forward from the earliest unwritten log record to replay all changes (**Repeating History**), bringing the system to the exact state it was in at the instant of failure.
3. **Undo Phase:** Scans backward to undo the actions of all active (uncommitted) transactions in the Losers Set, writing **Compensation Log Records (CLRs)** so recovery itself survives repeated crashes!

---

## 6. Storage & File Organization

### Static vs. Dynamic Hashing

- **Static Hashing:** Number of disk buckets is fixed. When a bucket fills up, overflow buckets are chained via pointers. Degrades to $O(N)$ linked list search during data skews.
- **Dynamic Hashing (Extendible Hashing):**
  * Hash function produces a binary string (e.g., 32 bits).
  * Uses a **Directory** of pointers to physical **Buckets**.
  * **Global Depth ($D$):** Number of binary bits used by the directory ($2^D$ directory entries).
  * **Local Depth ($d$):** Number of binary bits used by an individual bucket.

```
Directory (Global Depth = 2)             Buckets (Local Depth = d)
[ 00 ] --------------------------------> [ Bucket A (d = 2) ] Keys ending in 00
[ 01 ] --------------------------------> [ Bucket B (d = 2) ] Keys ending in 01
[ 10 ] ------------------------+
                               +-------> [ Bucket C (d = 1) ] Keys ending in 0 or 1
[ 11 ] ------------------------+
```

*When Bucket A overflows:*
- If local depth $d = \text{Global Depth } D$: Double the directory size ($D$ becomes $D+1$) and split the bucket.
- If local depth $d < \text{Global Depth } D$: Simply split the bucket and increment its local depth $d$ without resizing the directory!

---

## 7. Distributed Databases & Modern Storage Paradigms

### SQL vs. NoSQL

```
+-------------------------------------------------------------------------------------------------+
|                                     SQL vs. NoSQL TAXONOMY                                      |
+-------------------+------------------------------------+----------------------------------------+
| Feature           | Relational Databases (SQL)         | NoSQL Databases                        |
+-------------------+------------------------------------+----------------------------------------+
| **Data Model**    | Structured tables, rows, columns.  | Flexible: Key-Value, Document, Column, |
|                   | Strict predefined schema.          | Graph. Schema-less / dynamic.          |
| **Scaling**       | Primarily **Vertical** (Bigger CPU,| Primarily **Horizontal** (Scale out    |
|                   | RAM, SSD on a single machine).     | across clusters of cheap nodes).       |
| **Transactions**  | Strict **ACID** guarantees.        | **BASE** philosophy (Eventual          |
|                   |                                    | consistency preferred over locks).     |
| **Joins**         | Native, highly optimized relational| Denormalized; joins avoided or handled |
|                   | algebra multi-table joins.         | in client application code.            |
| **Primary Types** | PostgreSQL, MySQL, Oracle, SQLite. | MongoDB (Doc), Redis (K-V), Cassandra  |
|                   |                                    | (Wide-column), Neo4j (Graph).          |
+-------------------+------------------------------------+----------------------------------------+
```

---

### ACID vs. BASE Philosophy

| Dimension | ACID (Relational Paradigm) | BASE (Distributed NoSQL Paradigm) |
| :--- | :--- | :--- |
| **A** | **Atomicity:** All-or-nothing execution. | **Basically Available:** System guarantees availability; parts may degrade, but it stays up. |
| **C / S** | **Consistency:** Immediate constraint satisfaction. | **Soft State:** Data state may drift over time without user input due to replication delay. |
| **I / E** | **Isolation:** Independent transactions. | **Eventual Consistency:** Given enough time without new updates, all replicas converge to identical data. |
| **D** | **Durability:** Committed data is permanent. | Focuses on horizontal partition tolerance. |

---

### The CAP Theorem
Formulated by Eric Brewer: *In any distributed data store, you can only guarantee **two out of the following three properties simultaneously**:*

```
                           THE CAP TRIANGLE
                                  /\
                                 /  \
                                /    \
                               /  CA  \   <-- Traditional RDBMS (Single host;
                              / (Myth) \      no network partitions)
                             +----------+
                            / \        / \
                           /   \      /   \
                          /  CP \    / AP  \
                         /       \  /       \
     [ CONSISTENCY ]    +---------++---------+   [ AVAILABILITY ]
                         \                  /
                          \                /
                           \   PARTITION  /
                            \  TOLERANCE /
                             \          /
                              +--------+
```

1. **Consistency (C):** Every read receives the most recent write or an error.
2. **Availability (A):** Every non-failing node returns a non-error response for every request (no guarantee it's the latest data).
3. **Partition Tolerance (P):** The system continues to operate despite arbitrary network dropped packets or server disconnects.

> **Why "CA" is impossible in real-world distributed systems:**  
> Network partitions (cable cuts, switch failures) are **inevitable physical facts**. Therefore, a distributed system **MUST choose between CP or AP during a network partition**:
> - **Choose CP (Consistency):** Refuse writes or return errors until partitions heal (e.g., MongoDB, HBase, Google Spanner).
> - **Choose AP (Availability):** Accept writes on both sides of the partition; reconcile conflicts later (e.g., Cassandra, DynamoDB, CouchDB).

---

### The PACELC Theorem
An essential extension of the CAP theorem by Daniel Abadi:
> *"**IF** there is a **P**artition, how does the system choose between **A**vailability and **C**onsistency?  
> **E**LSE (when the system is running normally without partitions), how does the system trade off **L**atency versus **C**onsistency?"*

```
Examples:
- MongoDB:   PC/EC  (Chooses Consistency on partition, Consistency on normal operation)
- Cassandra: PA/EL  (Chooses Availability on partition, Low Latency on normal operation)
```

---

### Replication Strategies & Quorum Consensus
To guarantee strong consistency across $N$ distributed replicas:
- $N$ = Total number of replicas
- $W$ = Write quorum (number of nodes that must acknowledge a write)
- $R$ = Read quorum (number of nodes that must respond to a read)

$$\mathbf{W + R > N}$$

```
If N = 5 replicas:
Set W = 3, R = 3: (W + R = 6 > 5)
By Pigeonhole Principle, at least ONE node in the read quorum is guaranteed
to overlap with the write quorum, ensuring the client ALWAYS reads the newest write!
```

---

### Database Sharding & Consistent Hashing

- **Vertical Partitioning:** Splitting tables by columns (e.g., moving heavy `user_bio` and `avatar_image` blobs to a separate table).
- **Horizontal Partitioning (Sharding):** Splitting rows of a table across multiple database machines based on a **Shard Key**.
  * *Range-Based Sharding:* Shard 1 holds IDs 1–10M, Shard 2 holds 10M–20M. (Risk: Hotspots on newer IDs).
  * *Hash-Based Sharding:* Shard = $\text{Hash}(\text{user\_id}) \pmod N$. (Risk: Adding a node requires rehashing and moving almost all data!).
  * *Consistent Hashing:* Uses a circular hash ring ($0$ to $2^{32}-1$). Adding or removing a server node requires remapping only $K/N$ keys on average, preventing system-wide re-sharding bottlenecks.

---

## 8. Top 20 SDE-1 / Intern DBMS Interview Questions & Model Answers

### Q1: What is the difference between Conflict Serializability and View Serializability?
**Answer:** Conflict serializability guarantees that a schedule can be transformed into an equivalent serial schedule by swapping non-conflicting adjacent operations; it is verified in polynomial time using a Precedence Graph (cycle detection). View serializability is a broader standard requiring identical initial reads, identical final writes, and identical read-from relations. Every conflict serializable schedule is view serializable; a view serializable schedule that is not conflict serializable must contain **blind writes** (`W(X)` without prior `R(X)`). Testing view serializability is NP-Complete.

---

### Q2: What is the difference between Strict 2PL and Rigorous 2PL?
**Answer:** In Basic 2PL, locks can be released during the shrinking phase before transaction commit, which can cause cascading aborts. In **Strict 2PL**, all exclusive ($X$) write locks must be held until the transaction explicitly commits or aborts, ensuring cascadeless recovery. In **Rigorous 2PL**, **both** shared ($S$) read locks and exclusive ($X$) write locks must be held until commit or abort, producing a schedule order identical to the transaction commit order.

---

### Q3: What is a Cascading Rollback, and how do database engines prevent it?
**Answer:** A cascading rollback occurs when Transaction A fails and aborts, forcing Transaction B (which read uncommitted data written by A) to also abort, which in turn forces Transaction C to abort in a chain reaction. Database engines prevent this by enforcing **Cascadeless Schedules** (or using Strict 2PL), requiring that transactions only read data that has already been committed.

---

### Q4: Explain the difference between Physical and Logical Data Independence.
**Answer:** Physical data independence is the ability to modify the physical storage structures (e.g., switching from HDD to SSD, altering B+ tree page sizes, adding indexes) without modifying the conceptual logical schema (tables, columns). Logical data independence is the ability to alter the conceptual schema (e.g., adding a new column, splitting a table) without modifying the external views or breaking client application queries.

---

### Q5: What is a Weak Entity in ER modeling? How is it represented in a relational schema?
**Answer:** A weak entity is an entity that cannot be uniquely identified by its own attributes alone and depends on the existence of an identifying strong entity (e.g., `Dependent` dependent on `Employee`). In a relational schema, a weak entity is converted into a separate table whose primary key is a composite key consisting of the **Primary Key of the identifying strong entity (as a Foreign Key) + the Weak Entity's Partial Key (discriminator)**.

---

### Q6: What is the Write-Ahead Logging (WAL) protocol?
**Answer:** WAL mandates that any change to a data item must be recorded in non-volatile log storage on disk **before** the actual dirty data page in volatile memory is written to table storage on disk. This guarantees the Atomicity and Durability of transactions: if the system crashes midway, uncommitted writes can be reversed using Undo logs, and committed writes can be recovered using Redo logs.

---

### Q7: What are the 3 phases of the ARIES recovery algorithm?
**Answer:** (1) **Analysis Phase:** Scans the log forward from the last checkpoint to identify active uncommitted transactions (Losers Set) and dirty pages in memory at crash time. (2) **Redo Phase:** Scans forward from the earliest unwritten log record, reapplying all logged operations ("repeating history") to return the database to the exact state at the moment of crash. (3) **Undo Phase:** Scans backward, rolling back the actions of all uncommitted loser transactions and writing Compensation Log Records (CLRs) to ensure recovery is idempotent.

---

### Q8: What is a Blind Write? Why does it matter in serializability?
**Answer:** A blind write is a write operation performed on a data item without reading its value first (`W(X)` without a preceding `R(X)` in the same transaction). Blind writes are significant because they are the sole reason a schedule can be **View Serializable without being Conflict Serializable**.

---

### Q9: Why is the CAP Theorem's "CA" option considered a myth in distributed systems?
**Answer:** In a distributed network of computers, network partitions (packet loss, switch failures, disconnected optical lines) are inevitable physical realities. When a network partition occurs, a distributed system cannot be both Consistent and Available: it must either refuse writes to prevent divergent state (**Choose Consistency over Availability - CP**), or accept writes on isolated nodes leading to inconsistent states (**Choose Availability over Consistency - AP**).

---

### Q10: How does Quorum Consensus ensure strong consistency in distributed databases?
**Answer:** In a system of $N$ replicas with write quorum $W$ and read quorum $R$, setting $\mathbf{W + R > N}$ guarantees strong consistency. By the Pigeonhole Principle, the set of nodes read by a client will always overlap with the set of nodes written to by the previous writer by at least one node, ensuring the client always reads the latest written version.

---

### Q11: What is the difference between Extendible Hashing and Linear Hashing?
**Answer:** Extendible Hashing uses a directory of pointers that doubles in size ($2^D$) when a bucket overflows and its local depth equals global depth. Linear Hashing eliminates the directory entirely, handling bucket growth by incrementally splitting buckets one by one in a linear, round-robin order using a split pointer ($s$), avoiding sudden performance spikes caused by directory doubling.

---

### Q12: What is the Thomas Write Rule?
**Answer:** An optimization of the basic Timestamp Ordering protocol. When a transaction $T_i$ attempts to write to data item $Q$, but its timestamp is older than the write-timestamp of $Q$ ($TS(T_i) < W\text{-timestamp}(Q)$), instead of aborting and rolling back $T_i$, the protocol simply ignores and discards $T_i$'s write. This is safe because a newer transaction has already overwritten $Q$ with a more current value.

---

### Q13: What is the difference between Starvation and Deadlock in DBMS concurrency?
**Answer:** A Deadlock is a circular waiting condition where two or more transactions are permanently blocked waiting for locks held by each other; none can proceed without intervention (deadlock detection/abort). Starvation occurs when a transaction is repeatedly denied a requested lock because higher-priority transactions keep jumping ahead in the queue, causing the starved transaction to wait indefinitely even though the system continues to do useful work.

---

### Q14: What is an Intention Lock in Multiple Granularity Locking?
**Answer:** An Intention Lock (`IS` or `IX`) is placed on ancestor nodes (e.g., Database or Table) before explicit shared or exclusive locks are placed on descendant nodes (e.g., Rows). It signals to other transactions that finer-grained locking is active deeper in the tree, allowing a transaction to check if an entire table can be locked without scanning millions of individual row locks.

---

### Q15: What is the difference between Horizontal and Vertical Sharding?
**Answer:** Horizontal sharding (row-based) distributes the tuples of a single table across multiple separate database servers based on a shard key (e.g., users 1–1M on Node A, 1M–2M on Node B). Vertical sharding (column-based) splits a table's attributes across different databases according to domain or usage patterns (e.g., core account data on Server 1, heavy profile image BLOBs on Server 2).

---

### Q16: What is Consistent Hashing, and what problem does it solve in database clustering?
**Answer:** Consistent hashing maps both cluster nodes and data keys to positions on a circular hash ring ($0$ to $2^{32}-1$). In traditional modular hashing ($\text{key} \pmod N$), adding or removing a single node changes $N$, forcing almost 100% of keys to be rehashed and moved across the network. With consistent hashing, adding or removing a node requires remapping only $K/N$ keys on average to immediate neighbors, preventing severe cluster rebalancing storms.

---

### Q17: What is Shadow Paging in database recovery?
**Answer:** Shadow paging is an alternative to log-based recovery that maintains two page tables during transaction execution: a **Current Page Table** in memory and a **Shadow Page Table** on disk. Writes are directed to newly allocated disk pages. If the transaction commits, the shadow page table pointer is updated atomically to point to the current page table. If it aborts, the new pages are discarded, requiring zero log processing.

---

### Q18: What is the difference between a Dense Index and a Sparse Index?
**Answer:** In a **Dense Index**, an index record appears for **every single search key value** in the database table. In a **Sparse Index**, index records appear only for **some** search key values (typically one entry per physical data block/page). Sparse indexes require significantly less memory but are only applicable if the underlying data file is sorted on the indexed attribute.

---

### Q19: What is the difference between Pessimistic Locking and Optimistic Concurrency Control (OCC)?
**Answer:** Pessimistic locking locks data rows upfront upon read, blocking other transactions to prevent conflicts (best for high-write contention like ticket booking). OCC assumes collisions are rare; transactions read data without locking, execute in local private workspaces, and undergo a **Validation Phase** right before commit. If conflicting writes occurred, the transaction aborts and retries.

---

### Q20: What are ACID vs. BASE properties? When would you choose BASE?
**Answer:** ACID guarantees immediate consistency, strict atomicity, and serializable isolation, ideal for financial transactions and relational systems. BASE (Basically Available, Soft state, Eventual consistency) trades immediate consistency for ultra-high availability, low latency, and seamless horizontal scale. You choose BASE for massive distributed web scale (e.g., social media feeds, chat messages, product catalog views) where eventual replica convergence is acceptable.
