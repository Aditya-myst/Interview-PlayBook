# 07 — Indexing

## Making Queries Fast

---

### What is an Index?

An **index** is a data structure that speeds up data retrieval. It's like a book's index—you look up a topic and jump directly to the page.

**Without index:** Full table scan (read every row) — O(n)
**With index:** Direct lookup — O(log n)

```
Table: Employees (1 million rows)
┌────┬─────────┬───────────┬────────┐
│ ID │ Name    │ Department│ Salary │
├────┼─────────┼───────────┼────────┤
│ 1  │ Alice   │ CS        │ 100000 │
│ 2  │ Bob     │ Math      │ 80000  │
│ ...│ ...     │ ...       │ ...    │
└────┴─────────┴───────────┴────────┘

Without index:
SELECT * FROM Employees WHERE Name = 'Alice';
→ Scan all 1 million rows → Slow!

With index on Name:
SELECT * FROM Employees WHERE Name = 'Alice';
→ B-tree lookup → O(log n) → Fast!
```

---

### Types of Indexes

#### 1. Primary Index

Created automatically on primary key.

```sql
CREATE TABLE Students (
    ID INT PRIMARY KEY,  -- Clustered index created here
    Name VARCHAR(100),
    Email VARCHAR(100)
);
```

#### 2. Secondary Index (Non-Clustered)

Created on non-primary key columns.

```sql
CREATE INDEX idx_name ON Employees(Name);
CREATE INDEX idx_dept ON Employees(Department);
```

#### 3. Unique Index

Ensures all values in the indexed column are unique.

```sql
CREATE UNIQUE INDEX idx_email ON Employees(Email);
```

#### 4. Composite Index

Index on multiple columns.

```sql
CREATE INDEX idx_dept_salary ON Employees(Department, Salary);
```

**Column order matters!** Index on (A, B) can serve queries on A or (A, B), but not on B alone.

---

### Clustered vs Non-Clustered Index

```
Clustered Index:                Non-Clustered Index:
┌─────────────────────┐        ┌─────────────────────┐
│ Data is sorted by   │        │ Separate structure  │
│ clustered key       │        │ Points to data rows │
│                     │        │                     │
│ ┌───┬──────┐       │        │ ┌───┬──────┐       │
│ │ 1 │Alice │       │        │ │ 50│→row  │       │
│ │ 2 │Bob   │       │        │ │ 80│→row  │       │
│ │ 3 │Charlie│       │       │ │100│→row  │       │
│ └───┴──────┘       │        │ └───┴──────┘       │
└─────────────────────┘        └─────────────────────┘
```

| Aspect | Clustered | Non-Clustered |
|--------|-----------|---------------|
| **Data Storage** | Data sorted by index key | Separate structure |
| **Number per table** | Only one | Multiple |
| **Speed** | Faster (direct access) | Slower (extra lookup) |
| **Leaf nodes** | Contain actual data | Contain pointers to data |

**InnoDB (MySQL):** Clustered index is always the primary key.
**Heap tables (PostgreSQL):** No clustered index by default; use CLUSTER command.

---

### Dense vs Sparse Index

```
Dense Index:                 Sparse Index:
(Entry for every row)        (Entry for every block)

┌───┬──────┐                ┌───┬──────┐
│ A │→row  │                │ A │→block1│
│ B │→row  │                │ F │→block2│
│ C │→row  │                │ K │→block3│
│ D │→row  │                └───┴──────┘
│ E │→row  │
│ F │→row  │
│ ...     │
└─────────┘
```

| Type | Pros | Cons |
|------|------|------|
| **Dense** | Fast lookup | More space |
| **Sparse** | Less space | Must search within block |

**Sparse indexes only work with sorted data (clustered indexes).**

---

### Multi-Level Index

```
Level 2 (Top):
┌──────────────────────────────┐
│ 1    │ 50    │ 100   │ 150  │
└──┬───┴───┬───┴───┬───┴──┬───┘
   │       │       │      │
Level 1 (Middle):
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│ 1│25│50│60│75│80│100│120│150│...│
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
   │  │  │  │  │  │   │   │   │
Level 0 (Leaf - Data):
┌──────────────────────────────┐
│ Actual data rows             │
└──────────────────────────────┘
```

**This is essentially a B+ Tree.**

---

### When to Use Indexes

| Create Index When | Don't Create Index When |
|-------------------|------------------------|
| Column frequently in WHERE | Table is small |
| Column in JOIN conditions | Column rarely queried |
| Column in ORDER BY | High write, low read table |
| High cardinality (many unique values) | Low cardinality (e.g., boolean) |
| Read-heavy workload | Write-heavy workload |

---

### Index Selectivity

**Selectivity** = (Number of distinct values) / (Total rows)

| Selectivity | Example | Index Useful? |
|-------------|---------|---------------|
| High (near 1) | Email, SSN | Very useful |
| Medium | Department | Somewhat useful |
| Low (near 0) | Gender (M/F) | Usually not useful |

---

### Covering Index

An index that contains all columns needed by a query—no need to access the table.

```sql
-- Query
SELECT Name, Salary FROM Employees WHERE Department = 'CS';

-- Covering index (includes all columns in query)
CREATE INDEX idx_cover ON Employees(Department, Name, Salary);

-- Query uses only the index, not the table → Very fast!
```

---

### Index Scan vs Index Seek

| Operation | Description | Performance |
|-----------|-------------|-------------|
| **Index Seek** | Direct lookup (B-tree traversal) | O(log n) — Fast |
| **Index Scan** | Read entire index | O(n) — Slower |
| **Table Scan** | Read entire table | O(n) — Slowest |

```
WHERE ID = 5          → Index Seek (fast)
WHERE Name LIKE 'A%'  → Index Scan (medium)
WHERE Salary > 50000  → Index Scan or Seek depending on range
WHERE 1=1             → Table Scan (no index helps)
```

---

### Interview Questions

**Q: What is an index and how does it work?**

A: "An index is a data structure (typically B+ Tree) that speeds up data retrieval. It creates a sorted structure on one or more columns, allowing O(log n) lookups instead of O(n) full table scans. The index stores column values and pointers to the actual rows."

**Q: What's the difference between clustered and non-clustered indexes?**

A: "Clustered: data is physically sorted by the index key; there's only one per table; leaf nodes contain actual data. Non-clustered: separate structure from data; multiple allowed per table; leaf nodes contain pointers to data. Clustered is faster for range queries; non-clustered is flexible."

**Q: When should you create an index?**

A: "On columns frequently in WHERE, JOIN, or ORDER BY. High cardinality columns (many unique values). Read-heavy workloads. Don't index small tables, boolean columns, or write-heavy tables. Too many indexes slow down writes."

**Q: What's a covering index?**

A: "An index that contains all columns needed by a query. The query can be answered entirely from the index without accessing the table—very fast. Example: query needs (Department, Name, Salary); create index on all three."

**Q: What's the difference between index scan and index seek?**

A: "Index seek: direct lookup using B-tree—O(log n), very fast. Index scan: reads entire index—O(n). Table scan: reads entire table—O(n). Seek happens with equality conditions; scan happens with range queries or when index doesn't match query."

---

*Next: [08 — B-Trees & B+ Trees](08-B-Trees.md)*
