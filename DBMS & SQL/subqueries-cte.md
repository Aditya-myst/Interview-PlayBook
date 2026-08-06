# 16 — Subqueries & CTEs

## Complex Query Building Blocks

---

### Subqueries

A query inside another query.

```sql
-- Subquery in WHERE
SELECT Name, Salary
FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees);

-- Subquery in FROM
SELECT DeptName, AvgSalary
FROM (
    SELECT Department AS DeptName, AVG(Salary) AS AvgSalary
    FROM Employees
    GROUP BY Department
) AS DeptAvg
WHERE AvgSalary > 60000;

-- Subquery in SELECT (scalar subquery)
SELECT Name, Salary,
    (SELECT AVG(Salary) FROM Employees) AS CompanyAvg
FROM Employees;
```

---

### Correlated Subqueries

Subquery references the outer query. Executed once per outer row.

```sql
-- Employees earning above their department average
SELECT e.Name, e.Salary, e.Department
FROM Employees e
WHERE e.Salary > (
    SELECT AVG(Salary)
    FROM Employees
    WHERE Department = e.Department  -- References outer query
);

-- EXISTS (correlated)
SELECT c.Name
FROM Customers c
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.CustomerID = c.ID
    AND o.OrderDate > '2024-01-01'
);
```

**Performance:** Correlated subqueries can be slow (executed per row). Consider JOINs or CTEs.

---

### CTEs (Common Table Expressions)

Named temporary result sets. More readable than subqueries.

```sql
-- Basic CTE
WITH DeptAvg AS (
    SELECT Department, AVG(Salary) AS AvgSalary
    FROM Employees
    GROUP BY Department
)
SELECT e.Name, e.Salary, d.AvgSalary
FROM Employees e
JOIN DeptAvg d ON e.Department = d.Department
WHERE e.Salary > d.AvgSalary;
```

**Advantages over subqueries:**
| Feature | Subquery | CTE |
|---------|----------|-----|
| Readability | Nested, hard to read | Named, top-level |
| Reuse | Must repeat | Referenced by name |
| Recursion | Not supported | Supported |
| Multiple | Possible but messy | Clean separation |

---

### Multiple CTEs

```sql
WITH 
HighEarners AS (
    SELECT * FROM Employees WHERE Salary > 100000
),
DeptCounts AS (
    SELECT Department, COUNT(*) AS EmpCount
    FROM Employees
    GROUP BY Department
),
TopDepts AS (
    SELECT Department
    FROM DeptCounts
    WHERE EmpCount > 10
)
SELECT h.Name, h.Salary, h.Department
FROM HighEarners h
JOIN TopDepts t ON h.Department = t.Department;
```

---

### Recursive CTEs

CTE that references itself. Used for hierarchical data.

```sql
-- Employee hierarchy (find all reports)
WITH RECURSIVE EmployeeHierarchy AS (
    -- Base case: CEO (no manager)
    SELECT ID, Name, ManagerID, 1 AS Level
    FROM Employees
    WHERE ManagerID IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT e.ID, e.Name, e.ManagerID, eh.Level + 1
    FROM Employees e
    JOIN EmployeeHierarchy eh ON e.ManagerID = eh.ID
)
SELECT * FROM EmployeeHierarchy ORDER BY Level, Name;
```

Result:
```
ID | Name    | ManagerID | Level
---|---------|-----------|------
1  | Alice   | NULL      | 1
2  | Bob     | 1         | 2
3  | Charlie | 1         | 2
4  | Diana   | 2         | 3
```

---

### Recursive CTE for Series Generation

```sql
-- Generate numbers 1 to 10
WITH RECURSIVE Numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM Numbers WHERE n < 10
)
SELECT * FROM Numbers;

-- Date series
WITH RECURSIVE DateSeries AS (
    SELECT '2024-01-01'::DATE AS date
    UNION ALL
    SELECT date + 1 FROM DateSeries WHERE date < '2024-01-31'
)
SELECT * FROM DateSeries;
```

---

### Interview Questions

**Q: What's a CTE and when would you use it?**

A: "A Common Table Expression is a named temporary result set defined with WITH. Use for: (1) improving readability of complex queries, (2) reusing the same subquery multiple times, (3) recursive queries for hierarchical data. CTEs exist only during query execution."

**Q: What's a correlated subquery?**

A: "A subquery that references the outer query. It's executed once for each outer row. Example: find employees earning above their department average—the subquery filters based on the outer row's department. Can be slow; consider JOINs or CTEs."

**Q: How do you query hierarchical data in SQL?**

A: "Use recursive CTEs. Start with the base case (root nodes), then recursively join to find children. Example: employee hierarchy starts with CEO (no manager), then finds all direct reports, then their reports, etc. Returns level information for indentation."

**Q: When would you use a subquery vs a JOIN?**

A: "Subqueries for: existence checks (EXISTS), aggregations in WHERE, simple lookups. JOINs for: combining columns from multiple tables, better performance with large datasets. EXISTS is often faster than IN for large subqueries because it stops at first match."

---

*Next: [17 — Window Functions](17-Window-Functions.md)*
