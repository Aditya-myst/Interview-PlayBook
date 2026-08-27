# 06 — Database Mastery

## SQL, NoSQL, Optimization, Indexing — 40+ Interview Questions

---

### SQL vs NoSQL

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Data Model** | Relational (tables) | Document, Key-Value, Graph |
| **Schema** | Fixed | Flexible |
| **Scaling** | Vertical | Horizontal |
| **ACID** | Full support | Varies |
| **Joins** | Native | Limited |
| **Best For** | Complex relationships | High scalability |

---

### Indexing

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Covering index
CREATE INDEX idx_cover ON orders(customer_id, order_date, amount);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

---

### Query Optimization

```sql
-- Use EXPLAIN
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;

-- Avoid SELECT *
SELECT id, name, email FROM users WHERE status = 'active';

-- Avoid functions on indexed columns
-- BAD: WHERE YEAR(created_at) = 2024
-- GOOD: WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'

-- Use EXISTS instead of IN
SELECT * FROM customers c WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

---

### Transactions & ACID

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

### Interview Questions (40+)

**Q1: SQL vs NoSQL—when would you use each?**
A: "SQL for structured data with relationships, ACID requirements, complex queries. NoSQL for flexible schemas, horizontal scaling, high throughput. Most applications start with SQL; add NoSQL when specific needs arise."

**Q2: What is indexing and why is it important?**
A: "Data structure (B-tree, hash) that speeds up data retrieval. Without index: full table scan O(n). With index: O(log n). Trade-off: slower writes, more storage."

**Q3: What's the difference between clustered and non-clustered indexes?**
A: "Clustered: data sorted by index, one per table, leaf nodes contain data. Non-clustered: separate structure, multiple allowed, leaf nodes contain pointers."

**Q4: What is the leftmost prefix rule?**
A: "Composite index on (A, B, C) can serve queries on A, (A, B), or (A, B, C), but not B alone. Column order in index matters."

**Q5: How do you optimize slow queries?**
A: "1) EXPLAIN to see execution plan. 2) Add appropriate indexes. 3) Avoid SELECT *. 4) Avoid functions on indexed columns. 5) Use LIMIT. 6) Optimize JOINs."

**Q6: What is connection pooling?**
A: "Reuse database connections. Creating connections is expensive. Pool maintains ready connections. Configure max based on concurrent requests."

**Q7: What are ACID properties?**
A: "Atomicity: all or nothing. Consistency: valid state transitions. Isolation: concurrent transactions don't interfere. Durability: committed data survives crashes."

**Q8: What is sharding?**
A: "Horizontal partitioning—split data across servers. Each server holds subset. Strategies: range, hash, directory. Benefits: write scalability. Challenges: cross-shard queries."

**Q9: What is replication?**
A: "Copy data across servers. Primary-replica: writes to primary, reads from replicas. Multi-primary: writes to any. Benefits: read scalability, high availability."

**Q10: What is the CAP theorem?**
A: "Consistency, Availability, Partition Tolerance—pick two. CP: consistent but may be unavailable. AP: available but may be inconsistent. CA: single node."

**Q11: What is normalization?**
A: "Organize data to reduce redundancy. 1NF: atomic values. 2NF: no partial dependencies. 3NF: no transitive dependencies. BCNF: every determinant is candidate key."

**Q12: When would you denormalize?**
A: "Read-heavy workloads where JOINs are expensive. Analytics, reporting. Trade-off: faster reads, slower writes, potential inconsistency."

**Q13: What is a covering index?**
A: "Index that contains all columns needed by query. Database doesn't need to access table. Very fast. Example: query needs (dept, name, salary); index on all three."

**Q14: What is query execution plan?**
A: "Database's strategy for executing query. Shows table scans, index usage, join algorithms. Use EXPLAIN to see. Optimize based on plan."

**Q15: What is deadlocks and how do you prevent them?**
A: "Circular wait for resources. Prevention: consistent lock ordering, timeouts, deadlock detection. Most databases detect and abort one transaction."

**Q16: What is isolation level?**
A: "Defines how transactions interact. READ UNCOMMITTED: dirty reads. READ COMMITTED: no dirty reads. REPEATABLE READ: consistent reads. SERIALIZABLE: full isolation."

**Q17: What is optimistic vs pessimistic locking?**
A: "Optimistic: no locks, check for conflicts at commit (version number). Pessimistic: lock before read (SELECT FOR UPDATE). Optimistic for low contention; pessimistic for high."

**Q18: What is N+1 query problem?**
A: "Fetching list, then querying related data for each item. 1 query + N queries = N+1. Solution: eager loading (JOIN), DataLoader (batching)."

**Q19: What is database migration?**
A: "Version control for database schema. Tools: Flyway, Alembic, Prisma Migrate. Track changes, rollback safely. Run in CI/CD pipeline."

**Q20: What is read replica?**
A: "Copy of primary database for reads. Primary handles writes. Replicas handle reads. Reduces load on primary. Eventually consistent."

**Q21: What is database indexing strategy?**
A: "Index columns in WHERE, JOIN, ORDER BY. Composite index for multi-column queries. Don't over-index (slower writes). Monitor unused indexes."

**Q22: What is query plan caching?**
A: "Database caches execution plans. Parameterized queries reuse plans. Avoid plan cache bloat. SQL Server: sp_executesql. PostgreSQL: prepared statements."

**Q23: What is materialized view?**
A: "View stored physically. Faster reads, needs refresh. Use for expensive aggregations. Refresh on schedule or trigger."

**Q24: What is database connection string?**
A: "Contains host, port, database, user, password. Store in environment variables. Never hardcode. Use connection pooling libraries."

**Q25: What is NoSQL data modeling?**
A: "Design for query patterns, not normalization. Denormalize for read performance. Use appropriate data model (document, key-value, graph)."

**Q26: What is MongoDB aggregation pipeline?**
A: "Process documents through stages: $match, $group, $sort, $project. Similar to SQL GROUP BY. Powerful for analytics."

**Q27: What is Redis data structures?**
A: "Strings, hashes, lists, sets, sorted sets, streams, HyperLogLog. Choose based on use case. Sorted sets for leaderboards, hashes for objects."

**Q28: What is database partitioning?**
A: "Split table into smaller pieces. Horizontal: rows across partitions. Vertical: columns across partitions. Improves query performance, maintenance."

**Q29: What is database vacuuming?**
A: "PostgreSQL: reclaim storage from deleted rows. Autovacuum runs automatically. Important for performance. Configure based on workload."

**Q30: What is query parameterization?**
A: "Use parameters instead of string concatenation. Prevents SQL injection. Improves plan cache reuse. All modern ORMs support this."

**Q31: What is database connection leak?**
A: "Connections not returned to pool. Causes exhaustion. Prevention: use connection pool, close connections in finally blocks, use ORM."

**Q32: What is database sharding key?**
A: "Column used to distribute data across shards. Choose for even distribution. Common: user_id, tenant_id. Avoid hotspots."

**Q33: What is eventual consistency?**
A: "Replicas converge to same state over time. Used in distributed systems. Trade-off: may read stale data. Acceptable for many use cases."

**Q34: What is database cursor?**
A: "Pointer to result set. Process rows one at a time. Useful for large datasets. Avoid for small results (overhead)."

**Q35: What is database transaction isolation?**
A: "Defines visibility of changes between concurrent transactions. Higher isolation = more consistency, less concurrency."

**Q36: What is database backup strategy?**
A: "Full backup: periodic complete backup. Incremental: only changes since last backup. Point-in-time recovery. Test restore regularly."

**Q37: What is database replication lag?**
A: "Delay between primary write and replica availability. Causes stale reads. Monitor lag. Configure synchronous replication for critical data."

**Q38: What is database connection timeout?**
A: "Maximum time to wait for connection. Set appropriately. Handle timeout errors gracefully. Configure per environment."

**Q39: What is database query profiling?**
A: "Analyze query execution time, resource usage. Tools: EXPLAIN ANALYZE, pg_stat_statements, slow query log. Identify bottlenecks."

**Q40: What is database schema design best practices?**
A: "Normalize for OLTP, denormalize for OLAP. Use appropriate data types. Add constraints (NOT NULL, CHECK). Document relationships."

---

### Complete Database Implementation

```javascript
const { Pool } = require('pg');

// Connection Pool Configuration
const pool = new Pool({
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    max: 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
    statement_timeout: 30000,
    query_timeout: 30000
});

// Query helper with logging
async function query(text, params) {
    const start = Date.now();
    try {
        const result = await pool.query(text, params);
        const duration = Date.now() - start;
        console.log('Query executed', { text, duration, rows: result.rowCount });
        return result;
    } catch (error) {
        console.error('Query error', { text, error: error.message });
        throw error;
    }
}

// Transaction helper
async function transaction(callback) {
    const client = await pool.connect();
    try {
        await client.query('BEGIN');
        const result = await callback(client);
        await client.query('COMMIT');
        return result;
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
}

// Pagination helper
function paginate(tableName, { page = 1, limit = 10, filters = {}, sort = 'id', order = 'DESC' }) {
    const offset = (page - 1) * limit;
    const where = Object.keys(filters).map((key, i) => `${key} = $${i + 1}`).join(' AND ');
    const values = Object.values(filters);
    
    const dataQuery = `SELECT * FROM ${tableName} ${where ? 'WHERE ' + where : ''} ORDER BY ${sort} ${order} LIMIT $${values.length + 1} OFFSET $${values.length + 2}`;
    const countQuery = `SELECT COUNT(*) FROM ${tableName} ${where ? 'WHERE ' + where : ''}`;
    
    return { dataQuery, countQuery, values: [...values, limit, offset] };
}

// Health check
async function healthCheck() {
    try {
        const result = await pool.query('SELECT NOW()');
        return { status: 'healthy', timestamp: result.rows[0].now };
    } catch (error) {
        return { status: 'unhealthy', error: error.message };
    }
}

// Graceful shutdown
async function close() {
    await pool.end();
}
```

---

### Additional Interview Questions (15+)

**Q41: How do you handle database migrations in production?**
A: "Version-controlled migrations (Flyway, Alembic). Run before deployment. Support rollback. Zero-downtime migrations. Blue-green deployment compatible."

**Q42: What is database connection pooling best practices?**
A: "Set max connections based on CPU cores. Configure idle timeout. Handle connection errors. Monitor pool usage. Close connections in finally blocks."

**Q43: How do you implement soft deletes?**
A: "Add deletedAt timestamp column. Filter out in queries (WHERE deletedAt IS NULL). Use ORM scopes. Preserve data for audit."

**Q44: What is database query optimization techniques?**
A: "EXPLAIN analysis. Proper indexing. Avoid SELECT *. Use LIMIT. Optimize JOINs. Denormalize for read-heavy. Connection pooling."

**Q45: How do you handle database failover?**
A: "Primary-replica setup. Automatic failover (Patroni, repmgr). Health checks. Connection retry logic. Application-level failover."

**Q46: What is database caching strategy?**
A: "Application cache (Redis). Query cache (MySQL). Materialized views. Connection pooling. Choose based on consistency requirements."

**Q47: How do you implement database auditing?**
A: "Audit table with triggers. Track changes (who, when, what). CDC (Change Data Capture). Application-level logging."

**Q48: What is database connection string security?**
A: "Store in environment variables. Use secret managers. Never in code. Rotate credentials. Use IAM authentication."

**Q49: How do you handle database scaling?**
A: "Read replicas for read scaling. Sharding for write scaling. Connection pooling. Caching. Query optimization."

**Q50: What is database monitoring best practices?**
A: "Track query performance. Monitor connections. Alert on slow queries. Tools: pg_stat_statements, Prometheus, Grafana."

**Q51: How do you implement database versioning?**
A: "Migration files with version numbers. Sequential execution. Rollback support. CI/CD integration. Test migrations."

**Q52: What is database partitioning strategies?**
A: "Range partitioning (dates). Hash partitioning (even distribution). List partitioning (categories). Improves query performance."

**Q53: How do you handle database constraints?**
A: "PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK. Enforce data integrity. Application + database validation."

**Q54: What is database indexing best practices?**
A: "Index WHERE, JOIN, ORDER BY columns. Composite index order matters. Don't over-index. Monitor unused indexes."

**Q55: How do you implement database backup?**
A: "Full backup daily. Incremental backup hourly. Point-in-time recovery. Test restore regularly. Offsite storage."

---

*Next: [07 — ORMs & Data Access](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/BACKEND/ORM.md)*
