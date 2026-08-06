# 18 — Ranking & Analytics

## Practical Window Function Applications

---

### Find Nth Highest Salary

```sql
-- Find 3rd highest salary
WITH RankedSalaries AS (
    SELECT Name, Salary,
        DENSE_RANK() OVER (ORDER BY Salary DESC) AS rnk
    FROM Employees
)
SELECT Name, Salary
FROM RankedSalaries
WHERE rnk = 3;
```

---

### Top N Per Group

```sql
-- Top 3 highest paid in each department
WITH Ranked AS (
    SELECT Name, Department, Salary,
        ROW_NUMBER() OVER (PARTITION BY Department ORDER BY Salary DESC) AS rn
    FROM Employees
)
SELECT Name, Department, Salary
FROM Ranked
WHERE rn <= 3;
```

---

### Running Totals and Cumulative Sums

```sql
-- Running total of sales
SELECT 
    OrderDate,
    Amount,
    SUM(Amount) OVER (ORDER BY OrderDate) AS RunningTotal
FROM Orders;

-- Running total by category
SELECT 
    Category,
    OrderDate,
    Amount,
    SUM(Amount) OVER (PARTITION BY Category ORDER BY OrderDate) AS CategoryRunningTotal
FROM Orders;
```

---

### Moving Averages

```sql
-- 7-day moving average
SELECT 
    Date,
    Revenue,
    AVG(Revenue) OVER (
        ORDER BY Date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS MovingAvg7Day
FROM DailySales;

-- 3-month moving average
SELECT 
    Month,
    Revenue,
    AVG(Revenue) OVER (
        ORDER BY Month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS MovingAvg3Month
FROM MonthlySales;
```

---

### Year-over-Year Comparison

```sql
SELECT 
    Year,
    Month,
    Revenue,
    LAG(Revenue, 12) OVER (ORDER BY Year, Month) AS SameMonthLastYear,
    Revenue - LAG(Revenue, 12) OVER (ORDER BY Year, Month) AS YoYChange,
    (Revenue - LAG(Revenue, 12) OVER (ORDER BY Year, Month)) * 100.0 
        / LAG(Revenue, 12) OVER (ORDER BY Year, Month) AS YoYGrowthPct
FROM MonthlySales;
```

---

### Percentile Calculation

```sql
-- Salary percentile
SELECT Name, Salary,
    PERCENT_RANK() OVER (ORDER BY Salary) AS Percentile,
    CUME_DIST() OVER (ORDER BY Salary) AS CumulativeDistribution
FROM Employees;
```

**PERCENT_RANK** = (rank - 1) / (total_rows - 1)
**CUME_DIST** = (rows with value ≤ current) / total_rows

---

### Gap and Island Problem

Find consecutive sequences (islands) and gaps.

```sql
-- Find gaps in sequential IDs
WITH Numbered AS (
    SELECT ID,
        ID - ROW_NUMBER() OVER (ORDER BY ID) AS grp
    FROM Sequences
)
SELECT MIN(ID) AS GapStart, MAX(ID) AS GapEnd
FROM Numbered
GROUP BY grp
ORDER BY GapStart;
```

---

### Pivoting Data (Rows to Columns)

```sql
-- Traditional pivot using CASE
SELECT 
    Department,
    SUM(CASE WHEN Gender = 'M' THEN 1 ELSE 0 END) AS MaleCount,
    SUM(CASE WHEN Gender = 'F' THEN 1 ELSE 0 END) AS FemaleCount
FROM Employees
GROUP BY Department;

-- Using PIVOT (SQL Server)
SELECT *
FROM (
    SELECT Department, Gender, Salary
    FROM Employees
) AS SourceTable
PIVOT (
    AVG(Salary)
    FOR Gender IN ([M], [F])
) AS PivotTable;
```

---

### Unpivoting (Columns to Rows)

```sql
-- Traditional unpivot using UNION ALL
SELECT Department, 'Q1' AS Quarter, Q1Sales AS Sales FROM SalesData
UNION ALL
SELECT Department, 'Q2' AS Quarter, Q2Sales AS Sales FROM SalesData
UNION ALL
SELECT Department, 'Q3' AS Quarter, Q3Sales AS Sales FROM SalesData
UNION ALL
SELECT Department, 'Q4' AS Quarter, Q4Sales AS Sales FROM SalesData;

-- Using UNPIVOT (SQL Server)
SELECT Department, Quarter, Sales
FROM SalesData
UNPIVOT (
    Sales FOR Quarter IN (Q1Sales, Q2Sales, Q3Sales, Q4Sales)
) AS UnpivotTable;
```

---

### Median Calculation

```sql
-- Median salary (PostgreSQL)
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY Salary) AS MedianSalary
FROM Employees;

-- Median salary (MySQL)
WITH Ordered AS (
    SELECT Salary,
        ROW_NUMBER() OVER (ORDER BY Salary) AS rn,
        COUNT(*) OVER () AS total
    FROM Employees
)
SELECT AVG(Salary) AS MedianSalary
FROM Ordered
WHERE rn IN (FLOOR((total + 1) / 2), CEIL((total + 1) / 2));
```

---

### Consecutive Days Problem

```sql
-- Find users who logged in for 3+ consecutive days
WITH LoginGroups AS (
    SELECT UserID, LoginDate,
        LoginDate - INTERVAL ROW_NUMBER() OVER (
            PARTITION BY UserID ORDER BY LoginDate
        ) DAY AS grp
    FROM Logins
)
SELECT UserID, MIN(LoginDate) AS StartDate, MAX(LoginDate) AS EndDate,
    COUNT(*) AS ConsecutiveDays
FROM LoginGroups
GROUP BY UserID, grp
HAVING COUNT(*) >= 3;
```

---

### Interview Questions

**Q: How do you find the Nth highest salary?**

A: "Use DENSE_RANK: SELECT * FROM (SELECT *, DENSE_RANK() OVER (ORDER BY Salary DESC) AS rnk FROM Employees) WHERE rnk = N. Or subquery: SELECT MAX(Salary) FROM Employees WHERE Salary < (SELECT MAX(Salary) FROM Employees) for 2nd highest."

**Q: How do you find the top N records per group?**

A: "Use ROW_NUMBER with PARTITION BY: SELECT * FROM (SELECT *, ROW_NUMBER() OVER (PARTITION BY Department ORDER BY Salary DESC) AS rn FROM Employees) WHERE rn <= N."

**Q: How do you calculate running totals?**

A: "Use SUM with window function: SELECT Date, Amount, SUM(Amount) OVER (ORDER BY Date) AS RunningTotal FROM Orders. The OVER clause defines the window—ORDER BY Date means sum all rows up to current date."

**Q: How do you pivot data in SQL?**

A: "Use CASE with GROUP BY: SELECT Category, SUM(CASE WHEN Month='Jan' THEN Sales END) AS Jan, SUM(CASE WHEN Month='Feb' THEN Sales END) AS Feb FROM Sales GROUP BY Category. Or use PIVOT keyword in SQL Server."

---

*Next: [19 — Advanced SQL](19-Advanced-SQL.md)*
