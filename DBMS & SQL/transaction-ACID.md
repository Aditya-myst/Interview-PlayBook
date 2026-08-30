# 04 — Transactions & ACID

## Ensuring Data Integrity

---

### What is a Transaction?

A **transaction** is a logical unit of work that consists of one or more SQL operations. It's **all or nothing**—either all operations succeed, or none do.

```sql
-- Bank transfer: $100 from Alice to Bob
BEGIN TRANSACTION;
UPDATE Account SET Balance = Balance - 100 WHERE Name = 'Alice';
UPDATE Account SET Balance = Balance + 100 WHERE Name = 'Bob';
COMMIT;  -- Both succeed, or neither does
```

---

### ACID Properties

```
┌─────────────────────────────────────────────────────────┐
│                    ACID Properties                       │
├─────────────┬─────────────┬─────────────┬───────────────┤
│ Atomicity   │ Consistency │ Isolation   │ Durability    │
│ (All or     │ (Valid      │ (No         │ (Committed    │
│  nothing)   │  state)     │  interference)│  = permanent)│
└─────────────┴─────────────┴─────────────┴───────────────┘
```

#### A — Atomicity

**"All or nothing."** Either all operations in a transaction complete, or none do.

```sql
-- If the second UPDATE fails, the first is also rolled back
BEGIN;
UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;  -- Succeeds
UPDATE Account SET Balance = Balance + 100 WHERE ID = 2;  -- Fails!
ROLLBACK;  -- Undo BOTH operations
```

**Implementation:** Transaction log (undo log) records all changes. If transaction fails, changes are undone.

#### C — Consistency

**"Database moves from one valid state to another."** All constraints, triggers, and rules are satisfied.

```sql
-- Constraint: Balance >= 0
-- Before: Alice has $50
UPDATE Account SET Balance = Balance - 100 WHERE Name = 'Alice';
-- Would result in Balance = -50 → violates constraint → ROLLBACK
```

**Example constraints:** Foreign keys, unique constraints, check constraints, triggers.

#### I — Isolation

**"Concurrent transactions don't interfere."** Each transaction appears to execute in isolation.

```
Transaction A:              Transaction B:
Read Balance ($100)         Read Balance ($100)
Balance = Balance - 50      Balance = Balance - 30
Write Balance ($50)         Write Balance ($70)
-- A's write overwritten!   -- Lost update problem!
```

**Solution:** Isolation levels (covered in Chapter 5).

#### D — Durability

**"Once committed, always committed."** Committed transactions survive system crashes.

**Implementation:** Write-Ahead Logging (WAL)—changes are written to log before database.

```
1. Transaction commits
2. Changes written to transaction log (on disk)
3. Acknowledgment sent to client
4. Later: changes applied to database

If crash occurs between 2 and 4:
→ Replay log on recovery
```

---

### Transaction States

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Active  │────>│ Partially│────>│Committed │
│(executing)     │Committed │     │          │
└────┬─────┘     └──────────┘     └──────────┘
     │
     │ Failure
     ▼
┌──────────┐
│ Failed   │
└────┬─────┘
     │
     ▼
┌──────────┐
│Aborted   │──→ (Restart or Kill)
└──────────┘
```

---

### Transaction Log (Write-Ahead Log)

```
Transaction Log:
┌──────────┬────────┬────────┬──────────┬──────────┐
│ LSN      │ TransID│ Type   │ Table    │ Data     │
├──────────┼────────┼────────┼──────────┼──────────┤
│ 1        │ T1     │ BEGIN  │          │          │
│ 2        │ T1     │ UPDATE │ Account  │ ID=1,    │
│          │        │        │          │ Bal:100→50│
│ 3        │ T2     │ BEGIN  │          │          │
│ 4        │ T1     │ COMMIT │          │          │
│ 5        │ T2     │ UPDATE │ Account  │ ID=2,    │
│          │        │        │          │ Bal:200→150│
│ 6        │ T2     │ COMMIT │          │          │
└──────────┴────────┴────────┴──────────┴──────────┘

If crash after LSN 5:
→ T1 committed → keep changes (replay if needed)
→ T2 not committed → undo changes
```

---

### Concurrent Transaction Problems

| Problem | Description | Example |
|---------|-------------|---------|
| **Dirty Read** | Read uncommitted data | T1 reads T2's uncommitted write |
| **Non-Repeatable Read** | Same read, different result | T1 reads, T2 updates, T1 reads again |
| **Phantom Read** | New rows appear | T1 reads, T2 inserts, T1 reads again |
| **Lost Update** | One write overwrites another | T1 and T2 update same row |

```
Dirty Read:
T1: UPDATE Account SET Balance=50 WHERE ID=1;  -- Not committed
T2: SELECT Balance FROM Account WHERE ID=1;     -- Reads 50 (dirty!)
T1: ROLLBACK;                                   -- Balance back to 100
-- T2 read data that never existed!

Non-Repeatable Read:
T1: SELECT Balance FROM Account WHERE ID=1;     -- Returns 100
T2: UPDATE Account SET Balance=50 WHERE ID=1;   -- Commits
T1: SELECT Balance FROM Account WHERE ID=1;     -- Returns 50!
-- Same query, different result in same transaction!

Phantom Read:
T1: SELECT * FROM Orders WHERE Amount > 100;    -- Returns 5 rows
T2: INSERT INTO Orders VALUES (6, 200);         -- Commits
T1: SELECT * FROM Orders WHERE Amount > 100;    -- Returns 6 rows!
-- New row "appeared" in same transaction!
```

---

### Commit and Rollback

```sql
-- Commit: save all changes permanently
COMMIT;

-- Rollback: undo all changes in transaction
ROLLBACK;

-- Savepoint: partial rollback point
SAVEPOINT sp1;
-- ... some operations ...
ROLLBACK TO sp1;  -- Undo only operations after sp1
```

---

### Interview Questions

**Q: What are ACID properties?**

A: "Atomicity: all or nothing—either all operations succeed or none do. Consistency: database moves from one valid state to another, respecting all constraints. Isolation: concurrent transactions don't interfere—each appears to run alone. Durability: committed transactions survive crashes—written to non-volatile storage."

**Q: What's the difference between atomicity and durability?**

A: "Atomicity ensures a transaction is complete or not applied at all—no partial updates. Durability ensures committed data survives crashes. Atomicity is about the transaction's integrity; durability is about persistence. Both use the transaction log: atomicity uses it to undo failed transactions; durability uses it to replay committed transactions after crashes."

**Q: What's a dirty read?**

A: "Reading data that another transaction has modified but not yet committed. If the other transaction rolls back, the read data never actually existed. Prevented by isolation levels: READ COMMITTED and higher prevent dirty reads."

**Q: What's the difference between COMMIT and ROLLBACK?**

A: "COMMIT makes all transaction changes permanent and visible to other transactions. ROLLBACK undoes all changes in the transaction, restoring the database to the state before the transaction began. After COMMIT, you can't ROLLBACK."

**Q: How does the transaction log ensure durability?**

A: "Write-Ahead Logging (WAL): before any change is written to the database, it's written to the log on stable storage. If a crash occurs, the recovery process replays committed transactions from the log and undoes uncommitted ones. This guarantees durability."

---

*Next: [05 — Isolation Levels](05-Isolation-Levels.md)*
