# 06 — Locks & Concurrency

## Managing Simultaneous Access

---

### Why Locks?

Without locks, concurrent transactions can cause lost updates, dirty reads, and inconsistent data. Locks ensure only one transaction modifies data at a time.

---

### Lock Types

#### Shared Lock (S-Lock)

Multiple transactions can hold shared locks simultaneously. Used for **reading**.

```
T1: SELECT Balance FROM Account WHERE ID=1;  -- Acquires S-lock
T2: SELECT Balance FROM Account WHERE ID=1;  -- Also acquires S-lock (OK!)
T3: UPDATE Account SET Balance=50 WHERE ID=1;  -- Needs X-lock (BLOCKED!)
```

#### Exclusive Lock (X-Lock)

Only one transaction can hold an exclusive lock. Used for **writing**.

```
T1: UPDATE Account SET Balance=50 WHERE ID=1;  -- Acquires X-lock
T2: SELECT Balance FROM Account WHERE ID=1;     -- Needs S-lock (BLOCKED!)
T3: UPDATE Account SET Balance=30 WHERE ID=1;   -- Needs X-lock (BLOCKED!)
```

#### Lock Compatibility Matrix

| | S-Lock | X-Lock |
|---|--------|--------|
| **S-Lock** | ✓ Compatible | ✗ Conflict |
| **X-Lock** | ✗ Conflict | ✗ Conflict |

Multiple S-locks can coexist. X-lock conflicts with everything.

---

### Lock Granularity

```
Database Level ◄──────────────────────────► Row Level
   (Coarsest)                                 (Finest)
```

| Granularity | Description | Pros | Cons |
|-------------|-------------|------|------|
| **Database** | Lock entire database | Simple | No concurrency |
| **Table** | Lock entire table | Simple | Blocks all table access |
| **Page** | Lock a page (8KB) | Balance | Some false conflicts |
| **Row** | Lock single row | Maximum concurrency | High overhead |

**Modern systems:** Use row-level locking with intention locks.

---

### Two-Phase Locking (2PL)

**Protocol for ensuring serializability:**

```
Phase 1: Growing (acquire locks, never release)
Phase 2: Shrinking (release locks, never acquire)

Transaction:
   Acquire S-lock    Acquire X-lock
        │                  │
        ▼                  ▼
   ┌──────────────────────────┐
   │     Growing Phase        │
   └──────────┬───────────────┘
              │ Lock Point
              ▼
   ┌──────────────────────────┐
   │     Shrinking Phase      │
   └──────────┬───────────────┘
              │
              ▼
        Release all locks
```

**Strict 2PL:** Release all locks only at commit/abort (prevents cascading rollbacks). **Used by most databases.**

---

### Deadlocks with Locks

```
T1: LOCK(Account 1)         T2: LOCK(Account 2)
T1: ... working ...          T2: ... working ...
T1: LOCK(Account 2) → BLOCKED (T2 holds it)
T2: LOCK(Account 1) → BLOCKED (T1 holds it)

DEADLOCK! Both wait forever.
```

**Deadlock Solutions:**

| Solution | Description |
|----------|-------------|
| **Timeout** | Abort transaction if waits too long |
| **Wait-For Graph** | Detect cycles in waiting graph |
| **Wait-Die** | Older transaction waits, younger dies |
| **Wound-Wait** | Older transaction wounds younger |
| **No-Wait** | If can't lock immediately, abort |

---

### Optimistic vs Pessimistic Locking

#### Pessimistic Locking
**Assume conflicts will happen.** Lock before reading/writing.

```sql
-- Lock the row
SELECT * FROM Account WHERE ID = 1 FOR UPDATE;
-- Now only this transaction can modify this row
UPDATE Account SET Balance = Balance - 100 WHERE ID = 1;
COMMIT;
```

**Use when:** High contention (many concurrent updates to same rows).

#### Optimistic Locking
**Assume conflicts are rare.** Check for conflicts at commit time.

```sql
-- Read without locking
SELECT Balance, Version FROM Account WHERE ID = 1;
-- Balance=100, Version=5

-- Update with version check
UPDATE Account SET Balance = 50, Version = Version + 1
WHERE ID = 1 AND Version = 5;
-- If version changed (another transaction modified), update fails (0 rows affected)
```

**Use when:** Low contention (conflicts are rare).

---

### Intent Locks

Used with multi-granularity locking. Declare intention to lock at a finer level.

| Lock | Meaning |
|------|---------|
| **IS** (Intent Shared) | Intend to acquire S-locks on finer level |
| **IX** (Intent Exclusive) | Intend to acquire X-locks on finer level |
| **SIX** (Shared + Intent Exclusive) | S-lock on this level, IX on finer |

```
Table has IS lock → Can acquire S-lock on rows
Table has IX lock → Can acquire X-lock on rows
```

---

### Interview Questions

**Q: What's the difference between shared and exclusive locks?**

A: "Shared locks (S-locks) are for reading—multiple transactions can hold them simultaneously. Exclusive locks (X-locks) are for writing—only one transaction can hold them. S-locks conflict with X-locks; X-locks conflict with everything. This ensures readers don't see partial writes and writers don't interfere."

**Q: What is Two-Phase Locking?**

A: "2PL ensures serializability by dividing lock operations into two phases: growing (acquire locks, never release) and shrinking (release locks, never acquire). Strict 2PL releases all locks only at commit/abort. This guarantees serializable schedules but can cause deadlocks."

**Q: What's the difference between pessimistic and optimistic locking?**

A: "Pessimistic: lock before accessing data—assumes conflicts are common. Use SELECT ... FOR UPDATE. Optimistic: no locks during read, check for conflicts at commit—assumes conflicts are rare. Use version numbers or timestamps. Pessimistic is safer but slower; optimistic is faster but may need retries."

**Q: How do databases prevent deadlocks?**

A: "Timeouts (abort if wait too long), wait-for graph detection (find cycles), wait-die/wound-wait (priority-based), or no-wait (abort immediately if can't lock). Most databases detect deadlocks periodically and abort one transaction to break the cycle."

---

*Next: [07 — Indexing](07-Indexing.md)*
