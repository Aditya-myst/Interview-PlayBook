# 13 — SQL Fundamentals

## SELECT, WHERE, ORDER BY and More

---

### SQL Command Categories

| Category | Commands | Purpose |
|----------|----------|---------|
| **DDL** | CREATE, ALTER, DROP, TRUNCATE | Define structure |
| **DML** | SELECT, INSERT, UPDATE, DELETE | Manipulate data |
| **DCL** | GRANT, REVOKE | Control access |
| **TCL** | COMMIT, ROLLBACK, SAVEPOINT | Manage transactions |

---

### SELECT Basics

```sql
-- Select specific columns
SELECT Name, Salary FROM Employees;

-- Select all columns
SELECT * FROM Employees;

-- Select with alias
SELECT Name AS EmployeeName, Salary AS MonthlySalary FROM Employees;

-- Select with expression
SELECT Name, Salary, Salary * 12 AS AnnualSalary FROM Employees;

-- Select distinct values
SELECT DISTINCT Department FROM Employees;
```

---

### WHERE Clause

```sql
-- Comparison operators
SELECT * FROM Employees WHERE Salary > 50000;
SELECT * FROM Employees WHERE Department = 'CS';
SELECT * FROM Employees WHERE Status != 'Inactive';

-- Logical operators
SELECT * FROM Employees WHERE Department = 'CS' AND Salary > 50000;
SELECT * FROM Employees WHERE Department = 'CS' OR Department = 'Math';
SELECT * FROM Employees WHERE NOT Department = 'HR';

-- BETWEEN
SELECT * FROM Employees WHERE Salary BETWEEN 40000 AND 80000;

-- IN
SELECT * FROM Employees WHERE Department IN ('CS', 'Math', 'Physics');

-- LIKE (pattern matching)
SELECT * FROM Employees WHERE Name LIKE 'A%';        -- Starts with A
SELECT * FROM Employees WHERE Name LIKE '%son';       -- Ends with son
SELECT * FROM Employees WHERE Name LIKE '%ali%';      -- Contains ali
SELECT * FROM Employees WHERE Name LIKE 'A____';      -- A followed by 4 chars

-- IS NULL / IS NOT NULL
SELECT * FROM Employees WHERE ManagerID IS NULL;
SELECT * FROM Employees WHERE Email IS NOT NULL;

-- EXISTS
SELECT * FROM Employees e
WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.EmployeeID = e.ID);
```

---

### ORDER BY

```sql
-- Ascending (default)
SELECT * FROM Employees ORDER BY Salary;

-- Descending
SELECT * FROM Employees ORDER BY Salary DESC;

-- Multiple columns
SELECT * FROM Employees ORDER BY Department ASC, Salary DESC;

-- By column position
SELECT Name, Salary FROM Employees ORDER BY 2 DESC;  -- 2nd column (Salary)

-- NULL handling (NULLs first or last)
SELECT * FROM Employees ORDER BY ManagerID NULLS LAST;
```

---

### LIMIT / OFFSET

```sql
-- MySQL/PostgreSQL
SELECT * FROM Employees ORDER BY Salary DESC LIMIT 10;

-- With offset (pagination)
SELECT * FROM Employees ORDER BY Salary DESC LIMIT 10 OFFSET 20;

-- SQL Server
SELECT TOP 10 * FROM Employees ORDER BY Salary DESC;

-- Oracle
SELECT * FROM Employees ORDER BY Salary DESC FETCH FIRST 10 ROWS ONLY;
```

---

### INSERT

```sql
-- Insert single row
INSERT INTO Employees (Name, Department, Salary)
VALUES ('Alice', 'CS', 75000);

-- Insert multiple rows
INSERT INTO Employees (Name, Department, Salary)
VALUES 
    ('Bob', 'Math', 65000),
    ('Charlie', 'CS', 80000);

-- Insert from SELECT
INSERT INTO EmployeeBackup (Name, Department)
SELECT Name, Department FROM Employees WHERE Department = 'CS';
```

---

### UPDATE

```sql
-- Update specific rows
UPDATE Employees SET Salary = 80000 WHERE ID = 1;

-- Update multiple columns
UPDATE Employees 
SET Salary = 80000, Department = 'Engineering'
WHERE Name = 'Alice';

-- Update with subquery
UPDATE Employees 
SET Salary = Salary * 1.1
WHERE Department IN (SELECT Name FROM Departments WHERE Location = 'NYC');
```

---

### DELETE

```sql
-- Delete specific rows
DELETE FROM Employees WHERE ID = 1;

-- Delete with condition
DELETE FROM Employees WHERE Status = 'Inactive';

-- Delete all rows (but keep structure)
DELETE FROM Employees;

-- Truncate (faster, can't rollback, resets auto-increment)
TRUNCATE TABLE Employees;
```

---

### NULL Handling

```sql
-- NULL comparisons
SELECT * FROM Employees WHERE ManagerID IS NULL;
SELECT * FROM Employees WHERE ManagerID IS NOT NULL;

-- COALESCE (return first non-NULL value)
SELECT Name, COALESCE(Phone, Email, 'No contact') AS Contact FROM Employees;

-- NULLIF (return NULL if values are equal)
SELECT Name, NULLIF(DeptID, 0) AS Dept FROM Employees;
-- Returns NULL if DeptID is 0

-- CASE with NULL
SELECT Name,
    CASE 
        WHEN ManagerID IS NULL THEN 'No Manager'
        ELSE 'Has Manager'
    END AS Status
FROM Employees;
```

---

### CASE Expression

```sql
-- Simple CASE
SELECT Name, Department,
    CASE Department
        WHEN 'CS' THEN 'Technology'
        WHEN 'HR' THEN 'People'
        WHEN 'Finance' THEN 'Money'
        ELSE 'Other'
    END AS DeptCategory
FROM Employees;

-- Searched CASE
SELECT Name, Salary,
    CASE
        WHEN Salary >= 100000 THEN 'High'
        WHEN Salary >= 60000 THEN 'Medium'
        ELSE 'Low'
    END AS SalaryBand
FROM Employees;
```

---

### String Functions

```sql
SELECT 
    UPPER(Name) AS UpperName,
    LOWER(Name) AS LowerName,
    LENGTH(Name) AS NameLength,
    SUBSTRING(Name, 1, 3) AS First3Chars,
    CONCAT(Name, ' - ', Department) AS FullName,
    REPLACE(Name, 'A', '@') AS Modified,
    TRIM(Name) AS Trimmed,
    LEFT(Name, 3) AS Left3,
    RIGHT(Name, 3) AS Right3
FROM Employees;
```

---

### Date Functions

```sql
SELECT 
    NOW() AS CurrentDateTime,
    CURRENT_DATE AS Today,
    DATE_FORMAT(HireDate, '%Y-%m-%d') AS FormattedDate,
    DATEDIFF(NOW(), HireDate) AS DaysEmployed,
    YEAR(HireDate) AS HireYear,
    MONTH(HireDate) AS HireMonth,
    DATE_ADD(HireDate, INTERVAL 1 YEAR) AS OneYearLater
FROM Employees;
```

---

### Interview Questions

**Q: What's the difference between DELETE, TRUNCATE, and DROP?**

A: "DELETE: removes specific rows, can use WHERE, can rollback, doesn't reset auto-increment. TRUNCATE: removes all rows, can't use WHERE, faster, resets auto-increment. DROP: removes entire table structure and data. DELETE is DML; TRUNCATE and DROP are DDL."

**Q: What's the difference between WHERE and HAVING?**

A: "WHERE filters rows before grouping (before GROUP BY). HAVING filters groups after aggregation (after GROUP BY). WHERE can't use aggregate functions; HAVING can. Example: WHERE Salary > 50000 (row filter) vs HAVING COUNT(*) > 5 (group filter)."

**Q: How does NULL work in SQL?**

A: "NULL represents unknown/missing data. NULL != NULL (use IS NULL). NULL in arithmetic produces NULL. Aggregate functions ignore NULLs (except COUNT(*)). Use COALESCE to provide defaults: COALESCE(column, 'default'). NULL sorts first in ASC order (in most databases)."

---

*Next: [14 — Aggregations & GROUP BY](14-Aggregations-GroupBy.md)*
