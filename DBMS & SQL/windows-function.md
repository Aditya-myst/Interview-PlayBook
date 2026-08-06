# 17 — Window Functions

## Advanced Analytics Without GROUP BY

---

### What are Window Functions?

Window functions perform calculations across a set of rows **related to the current row**—without collapsing them (unlike GROUP BY).

```sql
-- GROUP BY: collapses rows
SELECT Department, AVG(Salary) FROM Employees GROUP BY Department;
-- Returns one row per department

-- Window function: keeps all rows
SELECT Name, Department, Salary,
    AVG(Salary) OVER (PARTITION BY Department) AS DeptAvg
FROM Employees;
-- Returns all rows with department average added
```

---

### Window Function Syntax

```sql
function_name() OVER (
    [PARTITION BY column]      -- Grouping (like GROUP BY)
    [ORDER BY column]          -- Ordering within partition
    [ROWS/RANGE frame]         -- Window frame
)
```

---

### Basic Window Functions

```sql
-- Running total
SELECT Name, Salary,
    SUM(Salary) OVER (ORDER BY ID) AS RunningTotal
FROM Employees;

-- Department average
SELECT Name, Department, Salary,
    AVG(Salary) OVER (PARTITION BY Department) AS DeptAvg
FROM Employees;

-- Percentage of department total
SELECT Name, Department, Salary,
    Salary / SUM(Salary) OVER (PARTITION BY Department) * 100 AS PctOfDept
FROM Employees;
```

---

### Aggregate Functions as Window Functions

```sql
SELECT 
    Name,
    Department,
    Salary,
    COUNT(*) OVER () AS TotalEmployees,
    COUNT(*) OVER (PARTITION BY Department) AS DeptCount,
    SUM(Salary) OVER () AS TotalSalary,
    SUM(Salary) OVER (PARTITION BY Department) AS DeptTotalSalary,
    AVG(Salary) OVER () AS CompanyAvg,
    AVG(Salary) OVER (PARTITION BY Department) AS DeptAvg,
    MIN(Salary) OVER () AS MinSalary,
    MAX(Salary) OVER () AS MaxSalary
FROM Employees;
```

---

### ROW_NUMBER()

Assigns unique sequential integer to each row.

```sql
SELECT Name, Department, Salary,
    ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum
FROM Employees;

-- Within each department
SELECT Name, Department, Salary,
    ROW_NUMBER() OVER (PARTITION BY Department ORDER BY Salary DESC) AS DeptRowNum
FROM Employees;
```

Result:
```
Name    | Department | Salary | DeptRowNum
--------|------------|--------|------------
Alice   | CS         | 100000 | 1
Bob     | CS         | 90000  | 2
Charlie | CS         | 80000  | 3
Diana   | HR         | 70000  | 1
Eve     | HR         | 60000  | 2
```

---

### RANK() and DENSE_RANK()

```sql
SELECT Name, Salary,
    ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum,
    RANK() OVER (ORDER BY Salary DESC) AS RankNum,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRankNum
FROM Employees;
```

Result (with ties):
```
Name  | Salary | RowNum | RankNum | DenseRankNum
------|--------|--------|---------|-------------
Alice | 100000 | 1      | 1       | 1
Bob   | 100000 | 2      | 1       | 1
Charlie| 90000 | 3      | 3       | 2
Diana | 80000  | 4      | 4       | 3
Eve   | 80000  | 5      | 4       | 3
Frank | 70000  | 6      | 6       | 4
```

| Function | Ties | Gaps |
|----------|------|------|
| ROW_NUMBER | Always unique | No gaps |
| RANK | Same rank for ties | Gaps after ties |
| DENSE_RANK | Same rank for ties | No gaps |

---

### NTILE()

Divides rows into N equal groups.

```sql
SELECT Name, Salary,
    NTILE(4) OVER (ORDER BY Salary DESC) AS Quartile
FROM Employees;
```

Result (12 employees, 4 groups):
```
Name  | Salary | Quartile
------|--------|----------
Alice | 100000 | 1  (Top 25%)
Bob   | 90000  | 1
Charlie| 85000 | 1
Diana | 80000  | 2  (25-50%)
Eve   | 75000  | 2
Frank | 70000  | 2
Grace | 65000  | 3  (50-75%)
Heidi | 60000  | 3
Ivan  | 55000  | 3
Judy  | 50000  | 4  (Bottom 25%)
Karl  | 45000  | 4
Liam  | 40000  | 4
```

---

### LEAD() and LAG()

Access data from subsequent or preceding rows.

```sql
-- Compare with previous and next month's sales
SELECT Month, Revenue,
    LAG(Revenue, 1) OVER (ORDER BY Month) AS PrevMonth,
    LEAD(Revenue, 1) OVER (ORDER BY Month) AS NextMonth,
    Revenue - LAG(Revenue, 1) OVER (ORDER BY Month) AS Change
FROM MonthlySales;
```

Result:
```
Month  | Revenue | PrevMonth | NextMonth | Change
-------|---------|-----------|-----------|-------
Jan    | 1000    | NULL      | 1200      | NULL
Feb    | 1200    | 1000      | 1100      | 200
Mar    | 1100    | 1200      | 1400      | -100
Apr    | 1400    | 1100      | NULL      | 300
```

---

### FIRST_VALUE() and LAST_VALUE()

Access the first or last value in the window frame.

```sql
SELECT Name, Department, Salary,
    FIRST_VALUE(Name) OVER (
        PARTITION BY Department 
        ORDER BY Salary DESC
    ) AS HighestPaid,
    LAST_VALUE(Name) OVER (
        PARTITION BY Department 
        ORDER BY Salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS LowestPaid
FROM Employees;
```

---

### Window Frame Specification

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW    -- All rows up to current
ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING             -- 5-row window
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- Entire partition
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING     -- Current to end

-- Running sum
SUM(Salary) OVER (ORDER BY ID ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- Moving average (3 rows)
AVG(Salary) OVER (ORDER BY ID ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
```

---

### Interview Questions

**Q: What's the difference between GROUP BY and window functions?**

A: "GROUP BY collapses rows—one row per group. Window functions keep all rows and add calculated columns. GROUP BY: SELECT dept, AVG(sal) FROM emp GROUP BY dept. Window: SELECT name, dept, sal, AVG(sal) OVER (PARTITION BY dept) FROM emp."

**Q: What's the difference between RANK and DENSE_RANK?**

A: "Both assign same rank to ties. RANK skips numbers after ties (1,1,3,4,4,6). DENSE_RANK doesn't skip (1,1,2,3,3,4). ROW_NUMBER always assigns unique numbers (1,2,3,4,5,6)."

**Q: How do you find the second highest salary?**

A: "Several ways: (1) DENSE_RANK: SELECT * FROM (SELECT *, DENSE_RANK() OVER (ORDER BY Salary DESC) AS rnk FROM Employees) t WHERE rnk = 2. (2) Subquery: SELECT MAX(Salary) FROM Employees WHERE Salary < (SELECT MAX(Salary) FROM Employees)."

**Q: What's a window frame?**

A: "The set of rows included in a window function calculation. Defined by ROWS/RANGE BETWEEN ... AND ... . Default: RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW. Specify: ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING for a 5-row moving average."

---

*Next: [18 — Ranking & Analytics](18-Ranking-Analytics.md)*
