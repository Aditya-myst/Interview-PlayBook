# 14 — Aggregations & GROUP BY

## Summarizing Data

---

### Aggregate Functions

| Function | Description | NULL Handling |
|----------|-------------|---------------|
| **COUNT()** | Count rows | COUNT(*) includes NULLs; COUNT(col) excludes |
| **SUM()** | Total | Ignores NULLs |
| **AVG()** | Average | Ignores NULLs |
| **MIN()** | Minimum | Ignores NULLs |
| **MAX()** | Maximum | Ignores NULLs |

```sql
-- Count all rows
SELECT COUNT(*) FROM Employees;

-- Count non-NULL values
SELECT COUNT(Email) FROM Employees;

-- Count distinct values
SELECT COUNT(DISTINCT Department) FROM Employees;

-- Sum, Avg, Min, Max
SELECT 
    SUM(Salary) AS TotalSalary,
    AVG(Salary) AS AvgSalary,
    MIN(Salary) AS MinSalary,
    MAX(Salary) AS MaxSalary
FROM Employees;
```

---

### GROUP BY

Groups rows with same values, then applies aggregate functions.

```sql
-- Count employees per department
SELECT Department, COUNT(*) AS EmpCount
FROM Employees
GROUP BY Department;

-- Average salary per department
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department;

-- Multiple group by columns
SELECT Department, YEAR(HireDate) AS HireYear, COUNT(*) AS Count
FROM Employees
GROUP BY Department, YEAR(HireDate)
ORDER BY Department, HireYear;
```

**Rules:**
- Every column in SELECT must be in GROUP BY or an aggregate
- NULL values form their own group

---

### HAVING

Filters groups after aggregation.

```sql
-- Departments with more than 5 employees
SELECT Department, COUNT(*) AS EmpCount
FROM Employees
GROUP BY Department
HAVING COUNT(*) > 5;

-- Departments with average salary > 60000
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department
HAVING AVG(Salary) > 60000;

-- Combining WHERE and HAVING
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
WHERE Status = 'Active'           -- Filter rows first
GROUP BY Department
HAVING AVG(Salary) > 60000;       -- Filter groups after
```

**WHERE vs HAVING:**

| | WHERE | HAVING |
|---|-------|--------|
| **When** | Before grouping | After grouping |
| **Filters** | Individual rows | Groups |
| **Aggregates** | Can't use | Can use |

---

### GROUP BY with ROLLUP

Add summary rows (subtotals and grand total).

```sql
SELECT Department, JobTitle, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY Department, JobTitle WITH ROLLUP;
```

Result:
```
Department | JobTitle  | TotalSalary
-----------|-----------|------------
CS         | Developer | 200000
CS         | Manager   | 150000
CS         | NULL      | 350000    ← Subtotal for CS
HR         | Recruiter | 80000
HR         | NULL      | 80000     ← Subtotal for HR
NULL       | NULL      | 430000    ← Grand total
```

---

### GROUP BY with CUBE

All combinations of subtotals.

```sql
-- PostgreSQL syntax
SELECT Department, JobTitle, SUM(Salary)
FROM Employees
GROUP BY CUBE(Department, JobTitle);
```

---

### GROUPING SETS

Custom grouping combinations.

```sql
SELECT Department, JobTitle, SUM(Salary)
FROM Employees
GROUP BY GROUPING SETS (
    (Department, JobTitle),  -- By dept and title
    (Department),            -- By dept only
    (JobTitle),              -- By title only
    ()                       -- Grand total
);
```

---

### Interview Questions

**Q: What's the difference between WHERE and HAVING?**

A: "WHERE filters individual rows before grouping; HAVING filters groups after aggregation. WHERE can't use aggregate functions; HAVING can. Example: WHERE Salary > 50000 filters rows; HAVING AVG(Salary) > 50000 filters groups."

**Q: What's the difference between COUNT(*) and COUNT(column)?**

A: "COUNT(*) counts all rows including NULLs. COUNT(column) counts non-NULL values in that column. COUNT(DISTINCT column) counts unique non-NULL values."

**Q: Can you use aggregate functions without GROUP BY?**

A: "Yes—if you use only aggregate functions in SELECT, the entire table is treated as one group. Example: SELECT AVG(Salary) FROM Employees returns one row with the average."

---

*Next: [15 — Joins in SQL](15-Joins-SQL.md)*
