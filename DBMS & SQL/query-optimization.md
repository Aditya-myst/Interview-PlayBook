# 11 — Query Optimization

## Making Queries Faster

---

### Why Optimize?

Slow queries = slow applications. A query that takes 10 seconds vs 100ms is the difference between a usable app and an unusable one.

---

### The Query Execution Pipeline

```
SQL Query
    │
    ▼
┌─────────────┐
│   Parser    │ → Syntax check, create parse tree
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Optimizer  │ → Choose best execution plan
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Executor   │ → Execute the plan
└──────┬──────┘
       │
       ▼
   Results
```

---

### EXPLAIN / Execution Plans

Use EXPLAIN to see how the database executes your query.

```sql
EXPLAIN SELECT * FROM Employees WHERE Department = 'CS';
```

**Key columns in EXPLAIN output:**

| Column | Description |
|--------|-------------|
| **type** | Access type (ALL, index, range, ref, const) |
| **key** | Which index is used |
| **rows** | Estimated rows to scan |
| **Extra** | Additional info (Using filesort, Using temporary) |

**Access types (best to worst):**

```
system > const > eq_ref > ref > range > index > ALL

system: Table has 0 or 1 row
const:  Primary key or unique index lookup (1 row)
eq_ref: Primary key in JOIN (1 row per join)
ref:    Non-unique index lookup
range:  Index range scan (>, <, BETWEEN)
index:  Full index scan
ALL:    Full table scan (BAD!)
```

**Example:**

```sql
-- BAD: Full table scan
EXPLAIN SELECT * FROM Employees WHERE Salary > 50000;
-- type: ALL, rows: 1000000

-- GOOD: Index range scan
CREATE INDEX idx_salary ON Employees(Salary);
EXPLAIN SELECT * FROM Employees WHERE Salary > 50000;
-- type: range, rows: 50000
```

---

### Common Optimization Techniques

#### 1. Use Indexes Effectively

```sql
-- Index on WHERE columns
CREATE INDEX idx_dept ON Employees(Department);
SELECT * FROM Employees WHERE Department = 'CS';  -- Uses index

-- Composite index order matters!
CREATE INDEX idx_dept_salary ON Employees(Department, Salary);
-- Works: WHERE Department = 'CS'
-- Works: WHERE Department = 'CS' AND Salary > 50000
-- Doesn't work: WHERE Salary > 50000 (skips first column)
```

#### 2. Avoid SELECT *

```sql
-- BAD: Fetches all columns
SELECT * FROM Employees WHERE Department = 'CS';

-- GOOD: Fetch only needed columns
SELECT Name, Salary FROM Employees WHERE Department = 'CS';

-- Even better: covering index
CREATE INDEX idx_cover ON Employees(Department, Name, Salary);
```

#### 3. Avoid Functions on Indexed Columns

```sql
-- BAD: Function on column prevents index use
SELECT * FROM Employees WHERE YEAR(HireDate) = 2024;

-- GOOD: Rewrite without function
SELECT * FROM Employees 
WHERE HireDate >= '2024-01-01' AND HireDate < '2025-01-01';
```

#### 4. Use EXISTS Instead of IN for Subqueries

```sql
-- Slower: IN with large subquery
SELECT * FROM Customers 
WHERE ID IN (SELECT CustomerID FROM Orders);

-- Faster: EXISTS stops early
SELECT * FROM Customers c
WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerID = c.ID);
```

#### 5. Optimize JOINs

```sql
-- Index on join columns
CREATE INDEX idx_customer ON Orders(CustomerID);

-- Filter early
SELECT o.ID, c.Name
FROM (SELECT * FROM Orders WHERE Amount > 100) o
JOIN Customers c ON o.CustomerID = c.ID;
```

#### 6. Avoid LIKE with Leading Wildcard

```sql
-- BAD: Can't use index
SELECT * FROM Employees WHERE Name LIKE '%smith';

-- GOOD: Can use index
SELECT * FROM Employees WHERE Name LIKE 'smith%';
```

#### 7. Use LIMIT for Large Result Sets

```sql
-- Don't fetch millions of rows
SELECT * FROM Orders ORDER BY OrderDate DESC LIMIT 100;
```

---

### Index Optimization Rules

#### Leftmost Prefix Rule

Index on (A, B, C):

| Query | Uses Index? |
|-------|-------------|
| WHERE A = 1 | ✓ |
| WHERE A = 1 AND B = 2 | ✓ |
| WHERE A = 1 AND B = 2 AND C = 3 | ✓ |
| WHERE B = 2 | ✗ |
| WHERE B = 2 AND C = 3 | ✗ |
| WHERE A = 1 AND C = 3 | Partial (only A) |

#### Index Selectivity

```sql
-- High selectivity (good): unique values
CREATE INDEX idx_email ON Users(Email);

-- Low selectivity (bad): few distinct values
CREATE INDEX idx_gender ON Users(Gender);  -- Only M/F
```

---

### Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| SELECT * | Fetches unnecessary data | Select specific columns |
| Function on indexed column | Prevents index use | Rewrite condition |
| LIKE '%value' | Can't use index | Full-text search |
| Implicit type conversion | Prevents index use | Match types exactly |
| OR on different columns | May not use index | Use UNION |
| NOT IN with NULLs | May return unexpected results | Use NOT EXISTS |

---

### Query Rewriting Examples

```sql
-- Original: Slow
SELECT * FROM Orders 
WHERE CustomerID IN (SELECT ID FROM Customers WHERE Country = 'USA');

-- Rewritten: Faster
SELECT o.* FROM Orders o
JOIN Customers c ON o.CustomerID = c.ID
WHERE c.Country = 'USA';

-- Original: Slow
SELECT DISTINCT Department FROM Employees;

-- Rewritten: Faster (if indexed)
SELECT Department FROM Employees GROUP BY Department;

-- Original: Slow
SELECT * FROM Employees WHERE Salary != 50000;

-- Rewritten: Faster (uses index for both parts)
SELECT * FROM Employees WHERE Salary < 50000
UNION ALL
SELECT * FROM Employees WHERE Salary > 50000;
```

---

### Interview Questions

**Q: How do you optimize a slow query?**

A: "1) Run EXPLAIN to see the execution plan. 2) Check if indexes are being used—if not, create appropriate indexes. 3) Avoid SELECT *—fetch only needed columns. 4) Avoid functions on indexed columns. 5) Use EXISTS instead of IN for subqueries. 6) Filter early to reduce rows. 7) Consider denormalization if JOINs are expensive."

**Q: What's an execution plan?**

A: "The database's strategy for executing a query—shows which tables to scan, which indexes to use, join algorithms, and estimated rows. Use EXPLAIN to see it. Look for: full table scans (ALL), filesort, temporary tables—these indicate optimization opportunities."

**Q: What's the leftmost prefix rule?**

A: "A composite index on (A, B, C) can be used for queries on A, (A, B), or (A, B, C), but not on B alone or (B, C). The index is used left to right—queries must include the leftmost columns. This is why column order in composite indexes matters."

**Q: When would you denormalize?**

A: "For read-heavy workloads where JOINs are expensive—like analytics dashboards or reporting. The trade-off: faster reads but slower writes and potential inconsistency. Always normalize first, then denormalize strategically based on query patterns."

---

*Next: [12 — NoSQL & Distributed DB](12-NoSQL-Distributed.md)*
