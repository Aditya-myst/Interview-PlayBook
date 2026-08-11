# 03 — Normalization

## Eliminating Redundancy and Anomalies

---

### What is Normalization?

Normalization is the process of organizing a database to **reduce redundancy** and **prevent anomalies** (insert, update, delete anomalies).

**Why it matters:** Without normalization, you get duplicate data, inconsistent updates, and wasted space.

---

### The Problem: Anomalies

Consider this unnormalized table:

```
Orders Table:
┌──────────┬─────────┬──────────┬─────────┬───────────┬─────────┐
│ OrderID  │ Customer│ Customer │ Product │ Product   │ Quantity│
│          │ ID      │ Name     │ ID      │ Name      │         │
├──────────┼─────────┼──────────┼─────────┼───────────┼─────────┤
│ 1        │ 101     │ Alice    │ 1       │ Laptop    │ 2       │
│ 2        │ 101     │ Alice    │ 2       │ Mouse     │ 5       │
│ 3        │ 102     │ Bob      │ 1       │ Laptop    │ 1       │
└──────────┴─────────┴──────────┴─────────┴───────────┴─────────┘
```

**Problems:**

| Anomaly | Description | Example |
|---------|-------------|---------|
| **Insert** | Can't insert data without other data | Can't add new customer without an order |
| **Update** | Must update multiple rows | Changing Alice's name requires updating all her orders |
| **Delete** | Deleting data loses other data | Deleting order 3 loses Bob's info |

---

### Functional Dependency

**X → Y** means: "X determines Y" — knowing X uniquely determines Y.

```
Examples:
StudentID → Name (knowing StudentID tells you the Name)
(StudentID, CourseID) → Grade
ZipCode → City, State
```

**Types:**
| Type | Description | Example |
|------|-------------|---------|
| **Full** | Y depends on all of X | (A,B) → C |
| **Partial** | Y depends on part of X | (A,B) → C but A → C |
| **Transitive** | A → B → C | StudentID → DeptID → DeptName |

---

### Normal Forms

```
Unnormalized
     │
     ▼
┌─────────────────────┐
│  1NF (First Normal) │ ← Atomic values, no repeating groups
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  2NF (Second Normal)│ ← No partial dependencies
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  3NF (Third Normal) │ ← No transitive dependencies
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  BCNF (Boyce-Codd)  │ ← Every determinant is a candidate key
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  4NF                 │ ← No multi-valued dependencies
└─────────────────────┘
```

---

### 1NF (First Normal Form)

**Rules:**
1. Each column contains atomic (indivisible) values
2. No repeating groups or arrays
3. Each row is unique (has a primary key)

```sql
-- BAD (violates 1NF):
-- Phone column has multiple values
┌────┬─────────┬─────────────────────┐
│ ID │ Name    │ Phone               │
├────┼─────────┼─────────────────────┤
│ 1  │ Alice   │ 123-456, 789-012    │  ← Not atomic!
└────┴─────────┴─────────────────────┘

-- GOOD (1NF):
┌────┬─────────┬─────────────┐
│ ID │ Name    │ Phone       │
├────┼─────────┼─────────────┤
│ 1  │ Alice   │ 123-456     │
│ 1  │ Alice   │ 789-012     │
└────┴─────────┴─────────────┘

-- Or better: separate table
CREATE TABLE StudentPhone (
    StudentID INT,
    Phone VARCHAR(15),
    PRIMARY KEY (StudentID, Phone),
    FOREIGN KEY (StudentID) REFERENCES Student(ID)
);
```

---

### 2NF (Second Normal Form)

**Rules:**
1. Must be in 1NF
2. No partial dependencies (non-key attribute depends on part of composite key)

```sql
-- BAD (violates 2NF):
-- (StudentID, CourseID) is composite key
-- Grade depends on both (full dependency) ✓
-- StudentName depends only on StudentID (partial dependency) ✗
┌───────────┬───────────┬────────────┬──────────────┐
│ StudentID │ CourseID  │ StudentName│ Grade        │
├───────────┼───────────┼────────────┼──────────────┤
│ 1         │ 101       │ Alice      │ A            │
│ 1         │ 102       │ Alice      │ B            │  ← Alice repeated!
└───────────┴───────────┴────────────┴──────────────┘

-- GOOD (2NF): Split into two tables
CREATE TABLE Student (
    ID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID)
);
```

---

### 3NF (Third Normal Form)

**Rules:**
1. Must be in 2NF
2. No transitive dependencies (non-key → non-key)

```sql
-- BAD (violates 3NF):
-- StudentID → DeptID → DeptName
-- DeptName transitively depends on StudentID
┌───────────┬────────────┬──────────┬────────────┐
│ StudentID │ Name       │ DeptID   │ DeptName   │
├───────────┼────────────┼──────────┼────────────┤
│ 1         │ Alice      │ CS       │ Comp Sci   │
│ 2         │ Bob        │ CS       │ Comp Sci   │  ← DeptName repeated!
└───────────┴────────────┴──────────┴────────────┘

-- GOOD (3NF): Remove transitive dependency
CREATE TABLE Student (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    DeptID VARCHAR(10),
    FOREIGN KEY (DeptID) REFERENCES Department(ID)
);

CREATE TABLE Department (
    ID VARCHAR(10) PRIMARY KEY,
    Name VARCHAR(100)
);
```

---

### BCNF (Boyce-Codd Normal Form)

**Stricter than 3NF:** Every determinant must be a candidate key.

```sql
-- Example: Course → Professor (each course taught by one professor)
-- But Professor → Department (each professor in one department)
-- Keys: (Student, Course) and (Student, Professor)
-- Course is a determinant but not a candidate key → violates BCNF

-- Split:
CREATE TABLE CourseProfessor (
    CourseID INT PRIMARY KEY,
    ProfID INT
);

CREATE TABLE StudentCourse (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID)
);
```

**3NF vs BCNF:** Every relation in BCNF is in 3NF, but not every 3NF relation is in BCNF.

---

### Denormalization

**Intentionally adding redundancy** for performance.

```sql
-- Normalized: requires JOIN
SELECT o.OrderID, c.Name
FROM Orders o JOIN Customers c ON o.CustomerID = c.ID;

-- Denormalized: no JOIN needed
SELECT OrderID, CustomerName FROM Orders;  -- CustomerName stored in Orders
```

**When to denormalize:**
- Read-heavy workloads (analytics, reporting)
- JOINs are expensive
- Caching frequently accessed data

**Trade-off:** Faster reads, but slower writes and potential inconsistency.

---

### Normal Forms Summary

| Normal Form | Rule | Eliminates |
|-------------|------|------------|
| 1NF | Atomic values, no repeating groups | Multi-valued attributes |
| 2NF | No partial dependencies | Partial redundancy |
| 3NF | No transitive dependencies | Transitive redundancy |
| BCNF | Every determinant is candidate key | Anomalies from overlapping keys |
| 4NF | No multi-valued dependencies | Multi-valued redundancy |

---

### Interview Questions

**Q: What is normalization and why is it important?**

A: "Normalization organizes a database to reduce redundancy and prevent anomalies (insert, update, delete). It ensures data integrity by eliminating duplicate data and ensuring each fact is stored once. The trade-off is more tables and JOINs, but data consistency is guaranteed."

**Q: Explain 1NF, 2NF, 3NF with examples.**

A: "1NF: atomic values, no repeating groups. 2NF: 1NF + no partial dependencies (non-key depends on full composite key). 3NF: 2NF + no transitive dependencies (non-key doesn't depend on another non-key). Example: Student(StudentID, DeptID, DeptName) violates 3NF because DeptName depends on DeptID, not StudentID."

**Q: What's the difference between 3NF and BCNF?**

A: "3NF: no transitive dependencies. BCNF: every determinant is a candidate key. BCNF is stricter. A relation can be in 3NF but not BCNF if there's a determinant that's not a candidate key. In practice, most databases aim for 3NF."

**Q: When would you denormalize?**

A: "Denormalize for read-heavy workloads where JOINs are expensive—like analytics dashboards or caching layers. The trade-off: faster reads but slower writes and potential inconsistency. Always normalize first, then denormalize strategically for performance."

**Q: What are the anomalies normalization prevents?**

A: "Insert anomaly: can't add data without unrelated data. Update anomaly: must update multiple rows for one fact change. Delete anomaly: deleting data loses unrelated facts. Normalization ensures each fact is stored exactly once."

---

*Next: [04 — Transactions & ACID](04-Transactions-ACID.md)*
