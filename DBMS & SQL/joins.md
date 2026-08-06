# 09 — Joins Deep Dive

## Combining Data from Multiple Tables

---

### Why Joins?

Data is spread across multiple normalized tables. Joins combine related data for queries.

---

### Types of Joins

```
Table A          Table B
┌───┐           ┌───┐
│ 1 │           │ 2 │
│ 2 │           │ 3 │
│ 3 │           │ 4 │
└───┘           └───┘
```

#### INNER JOIN
Returns rows with matching values in both tables.

```sql
SELECT * FROM A INNER JOIN B ON A.id = B.id;
-- Result: 2, 3 (intersection)
```

#### LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from left table, matching from right.

```sql
SELECT * FROM A LEFT JOIN B ON A.id = B.id;
-- Result: 1, 2, 3 (all from A, NULL for B where no match)
```

#### RIGHT JOIN (RIGHT OUTER JOIN)
Returns all rows from right table, matching from left.

```sql
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;
-- Result: 2, 3, 4 (all from B, NULL for A where no match)
```

#### FULL OUTER JOIN
Returns all rows from both tables.

```sql
SELECT * FROM A FULL OUTER JOIN B ON A.id = B.id;
-- Result: 1, 2, 3, 4 (all from both, NULL where no match)
```

#### CROSS JOIN
Cartesian product—every row from A with every row from B.

```sql
SELECT * FROM A CROSS JOIN B;
-- Result: 3×3 = 9 rows
```

#### SELF JOIN
Table joined with itself.

```sql
-- Find employees and their managers
SELECT e.Name, m.Name AS Manager
FROM Employees e JOIN Employees m ON e.ManagerID = m.ID;
```

---

### Join Algorithms

#### 1. Nested Loop Join

For each row in outer table, scan inner table.

```python
for row_a in table_a:
    for row_b in table_b:
        if row_a.id == row_b.id:
            result.append(row_a + row_b)
```

**Time:** O(n × m)
**Best for:** Small tables, no index on join column.

#### 2. Hash Join

Build hash table on smaller table, probe with larger.

```python
# Build phase: hash smaller table
hash_table = {}
for row_b in table_b:
    hash_table[row_b.id] = row_b

# Probe phase: scan larger table
for row_a in table_a:
    if row_a.id in hash_table:
        result.append(row_a + hash_table[row_a.id])
```

**Time:** O(n + m)
**Best for:** Equi-joins, large tables, no index.

#### 3. Sort-Merge Join

Sort both tables on join column, then merge.

```python
table_a.sort(key=lambda r: r.id)
table_b.sort(key=lambda r: r.id)

i, j = 0, 0
while i < len(table_a) and j < len(table_b):
    if table_a[i].id == table_b[j].id:
        result.append(table_a[i] + table_b[j])
        j += 1
    elif table_a[i].id < table_b[j].id:
        i += 1
    else:
        j += 1
```

**Time:** O(n log n + m log m)
**Best for:** Pre-sorted data, range joins.

---

### Join Algorithm Comparison

| Algorithm | Time | Space | Best For |
|-----------|------|-------|----------|
| Nested Loop | O(n × m) | O(1) | Small tables |
| Hash Join | O(n + m) | O(min(n,m)) | Equi-joins, large tables |
| Sort-Merge | O(n log n) | O(n) | Pre-sorted, range joins |

**Query optimizer chooses the algorithm based on:**
- Table sizes
- Available indexes
- Join condition type
- Memory availability

---

### Join Order Matters

```sql
-- Bad: large cross join first
SELECT * FROM Orders o
JOIN Customers c ON o.CustomerID = c.ID
JOIN Products p ON o.ProductID = p.ID
WHERE c.Country = 'USA';

-- Optimizer reorders to:
-- 1. Filter Customers by Country (fewer rows)
-- 2. Join with Orders
-- 3. Join with Products
```

---

### Interview Questions

**Q: What's the difference between INNER and OUTER joins?**

A: "INNER JOIN returns only matching rows. LEFT JOIN returns all rows from left table (NULL for right where no match). RIGHT JOIN returns all from right. FULL OUTER JOIN returns all from both. INNER is the default and most common."

**Q: When would you use a CROSS JOIN?**

A: "Rarely—it's a Cartesian product (every combination). Use when you need all combinations, like generating a calendar (days × hours). Or when building test data. Always add a WHERE clause to filter, or you'll get millions of rows."

**Q: Explain the three join algorithms.**

A: "Nested Loop: for each row in A, scan B—O(n×m), good for small tables. Hash Join: build hash table on smaller table, probe with larger—O(n+m), best for equi-joins. Sort-Merge: sort both tables, merge—O(n log n), good for pre-sorted data or range joins."

**Q: How does the optimizer choose a join algorithm?**

A: "Based on table sizes, available indexes, join condition type, and memory. Small tables: nested loop. Large equi-joins without index: hash join. Pre-sorted data or range joins: sort-merge. The optimizer estimates costs and picks the cheapest plan."

---

*Next: [10 — Views & Stored Procedures](10-Views-Stored-Procedures.md)*
