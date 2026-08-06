# 05 — Isolation Levels

## Balancing Consistency and Performance

---

### Why Isolation Levels?

Perfect isolation (serializable) is expensive. Lower isolation levels allow more concurrency but risk anomalies. The choice depends on your application's tolerance for inconsistencies.

---

### The Four Isolation Levels

```
Most Isolation ◄─────────────────────────────► Least Isolation
SERIALIZABLE → REPEATABLE READ → READ COMMITTED → READ UNCOMMITTED
   (Safest)                                               (Fastest)
```

---

### Isolation Levels vs Anomalies

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|------------|---------------------|--------------|
| READ UNCOMMITTED | ✓ Possible | ✓ Possible | ✓ Possible |
| READ COMMITTED | ✗ Prevented | ✓ Possible | ✓ Possible |
| REPEATABLE READ | ✗ Prevented | ✗ Prevented | ✓ Possible |
| SERIALIZABLE | ✗ Prevented | ✗ Prevented | ✗ Prevented |

---

### 1. READ UNCOMMITTED

**Lowest isolation.** Transactions can read uncommitted changes from other transactions.

```
T1: UPDATE Account SET Balance=50 WHERE ID=1;  -- Not committed
T2: SELECT Balance FROM Account WHERE ID=1;     -- Reads 50 (dirty!)
T1: ROLLBACK;                                   -- Balance back to 100
-- T2 has invalid data!
```

**Use case:** Approximate counts, analytics where exact values don't matter.

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

---

### 2. READ COMMITTED

**Transactions only read committed data.** Default in PostgreSQL and Oracle.

```
T1: SELECT Balance FROM Account WHERE ID=1;     -- Returns 100
T2: UPDATE Account SET Balance=50 WHERE ID=1;
T2: COMMIT;
T1: SELECT Balance FROM Account WHERE ID=1;     -- Returns 50 (non-repeatable!)
```

**Prevents:** Dirty reads
**Allows:** Non-repeatable reads, phantom reads

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

### 3. REPEATABLE READ

**Same read returns same result within transaction.** Default in MySQL.

```
T1: SELECT Balance FROM Account WHERE ID=1;     -- Returns 100
T2: UPDATE Account SET Balance=50 WHERE ID=1;
T2: COMMIT;
T1: SELECT Balance FROM Account WHERE ID=1;     -- Still returns 100!
```

**Prevents:** Dirty reads, non-repeatable reads
**Allows:** Phantom reads (in theory; MySQL/InnoDB prevents them too)

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

---

### 4. SERIALIZABLE

**Highest isolation.** Transactions appear to execute serially (one after another).

```
T1: SELECT * FROM Orders WHERE Amount > 100;    -- Returns 5 rows
T2: INSERT INTO Orders VALUES (6, 200);
T2: COMMIT;
T1: SELECT * FROM Orders WHERE Amount > 100;    -- Still returns 5 rows!
```

**Prevents:** All anomalies (dirty reads, non-repeatable reads, phantom reads)
**Cost:** Lowest concurrency, potential deadlocks

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

### How Isolation Levels Work

#### Multi-Version Concurrency Control (MVCC)

Modern databases (PostgreSQL, MySQL InnoDB) use MVCC instead of locks for reads.

```
Each row has hidden columns:
- xmin: transaction ID that created this version
- xmax: transaction ID that deleted/updated this version

When T1 reads:
- Sees versions where xmin ≤ T1 and xmax > T1 (or xmax is NULL)
- Doesn't see uncommitted versions
```

**MVCC allows readers and writers to not block each other.**

#### Lock-Based

```
READ UNCOMMITTED: No read locks
READ COMMITTED: Read locks released immediately after read
REPEATABLE READ: Read locks held until transaction end
SERIALIZABLE: Range locks (prevents phantom inserts)
```

---

### Choosing the Right Level

| Scenario | Isolation Level | Why |
|----------|-----------------|-----|
| Banking/Financial | SERIALIZABLE | Can't risk any inconsistency |
| E-commerce (orders) | REPEATABLE READ | Consistent view during checkout |
| Analytics/Reporting | READ COMMITTED | Speed matters, exact values don't |
| Approximate counts | READ UNCOMMITTED | Rough estimates are fine |

**Default recommendations:**
- PostgreSQL: READ COMMITTED (default)
- MySQL: REPEATABLE READ (default)
- Oracle: READ COMMITTED (default)
- SQL Server: READ COMMITTED (default)

---

### Interview Questions

**Q: What are the four isolation levels?**

A: "READ UNCOMMITTED: can read dirty data. READ COMMITTED: only reads committed data (prevents dirty reads). REPEATABLE READ: same query returns same result (prevents non-repeatable reads). SERIALIZABLE: transactions appear serial (prevents all anomalies). Higher isolation = more consistency but less concurrency."

**Q: What's the difference between REPEATABLE READ and SERIALIZABLE?**

A: "REPEATABLE READ prevents non-repeatable reads (same row returns same values) but may allow phantom reads (new rows appearing). SERIALIZABLE prevents both—transactions execute as if serial. SERIALIZABLE uses range locks to prevent phantom inserts."

**Q: What is MVCC?**

A: "Multi-Version Concurrency Control maintains multiple versions of each row. Readers see a snapshot of data at transaction start; writers create new versions. This allows readers and writers to not block each other. Used by PostgreSQL, MySQL InnoDB, Oracle."

**Q: Which isolation level should I use?**

A: "Depends on the application. Banking: SERIALIZABLE. E-commerce: REPEATABLE READ. Analytics: READ COMMITTED. Default levels (READ COMMITTED in PostgreSQL/Oracle, REPEATABLE READ in MySQL) work for most applications. Only increase isolation if you encounter specific anomalies."

**Q: What's a phantom read?**

A: "When a transaction re-executes a query and sees new rows that weren't there before (inserted by another committed transaction). Example: first query returns 5 rows, another transaction inserts a row, second query returns 6 rows. SERIALIZABLE prevents this."

---

*Next: [06 — Locks & Concurrency](06-Locks-Concurrency.md)*
