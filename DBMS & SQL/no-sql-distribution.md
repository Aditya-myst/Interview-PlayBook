# 12 — NoSQL & Distributed Databases

## Modern Database Concepts

---

### SQL vs NoSQL

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Data Model** | Relational (tables) | Document, Key-Value, Graph, Column |
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (bigger server) | Horizontal (more servers) |
| **ACID** | Full support | Varies (eventual consistency) |
| **Joins** | Native support | Limited or none |
| **Query Language** | SQL | Database-specific |
| **Best For** | Complex relationships, transactions | High scalability, flexible data |

---

### Types of NoSQL Databases

#### 1. Document Store

Data stored as JSON/BSON documents.

```json
// MongoDB
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "Alice",
  "age": 30,
  "address": {
    "street": "123 Main St",
    "city": "New York"
  },
  "hobbies": ["reading", "coding"]
}
```

**Examples:** MongoDB, CouchDB, Firestore
**Best for:** Content management, user profiles, catalogs.

#### 2. Key-Value Store

Simple key-value pairs.

```
Key: "user:1001"
Value: {"name": "Alice", "age": 30}

Key: "session:abc123"
Value: {"user_id": 1001, "expires": "2024-12-31"}
```

**Examples:** Redis, DynamoDB, Memcached
**Best for:** Caching, session storage, real-time analytics.

#### 3. Column-Family Store

Data organized by columns rather than rows.

```
Row Key: "user:1001"
├── Column Family "profile"
│   ├── name: "Alice"
│   └── age: "30"
└── Column Family "activity"
    ├── last_login: "2024-01-15"
    └── login_count: "150"
```

**Examples:** Cassandra, HBase, Bigtable
**Best for:** Time-series data, IoT, large-scale analytics.

#### 4. Graph Database

Data stored as nodes and relationships.

```
(Alice)──[FRIENDS_WITH]──>(Bob)
   │                         │
   │                         │
[WORKS_AT]              [WORKS_AT]
   │                         │
   ▼                         ▼
(Google)                (Microsoft)
```

**Examples:** Neo4j, Amazon Neptune
**Best for:** Social networks, recommendation engines, fraud detection.

---

### CAP Theorem

A distributed database can guarantee at most **two out of three**:

```
        Consistency
           /\
          /  \
         /    \
        / CAP  \
       /________\
  Availability   Partition Tolerance
```

| Property | Meaning |
|----------|---------|
| **Consistency** | Every read receives the most recent write |
| **Availability** | Every request receives a response (success/failure) |
| **Partition Tolerance** | System continues despite network partitions |

**In practice:** Network partitions WILL happen, so you choose between CP and AP.

| Type | Description | Examples |
|------|-------------|----------|
| **CP** | Consistent but may be unavailable | MongoDB, HBase, Redis |
| **AP** | Available but may be inconsistent | Cassandra, DynamoDB, CouchDB |
| **CA** | Consistent and available (no partitions) | Traditional RDBMS (single node) |

---

### BASE Properties (NoSQL alternative to ACID)

| Property | Description |
|----------|-------------|
| **Basically Available** | System guarantees availability |
| **Soft State** | State may change over time (no consistency guarantee) |
| **Eventually Consistent** | Given enough time, all replicas converge |

---

### Sharding (Horizontal Partitioning)

Split data across multiple servers.

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Shard 1  │     │ Shard 2  │     │ Shard 3  │
│ Users    │     │ Users    │     │ Users    │
│ A-F      │     │ G-M      │     │ N-Z      │
└─────────┘     └─────────┘     └─────────┘
```

**Sharding strategies:**

| Strategy | Description | Example |
|----------|-------------|---------|
| **Range** | Based on value ranges | Users A-F in shard 1 |
| **Hash** | Hash of key determines shard | hash(user_id) % num_shards |
| **Directory** | Lookup table for shard mapping | Centralized shard map |

**Challenges:**
- Cross-shard queries are expensive
- Rebalancing when adding shards
- Distributed transactions

---

### Replication

Copy data across multiple servers.

#### Primary-Replica (Master-Slave)

```
┌──────────┐
│ Primary  │──── Writes
└────┬─────┘
     │ Replication
     ├──────────────────┐
     │                  │
┌────┴─────┐      ┌────┴─────┐
│ Replica 1│      │ Replica 2│
│ (Reads)  │      │ (Reads)  │
└──────────┘      └──────────┘
```

**Pros:** Read scalability, fault tolerance
**Cons:** Replication lag (eventual consistency)

#### Multi-Primary (Master-Master)

```
┌──────────┐         ┌──────────┐
│ Primary 1│◄───────►│ Primary 2│
│ (R+W)    │         │ (R+W)    │
└──────────┘         └──────────┘
```

**Pros:** Write scalability
**Cons:** Conflict resolution complex

---

### Distributed Transactions

#### Two-Phase Commit (2PC)

```
Phase 1: Prepare
Coordinator → All nodes: "Can you commit?"
All nodes → Coordinator: "Yes" or "No"

Phase 2: Commit/Abort
If all said "Yes": Coordinator → All: "Commit"
If any said "No": Coordinator → All: "Abort"
```

**Problem:** Blocking—if coordinator crashes after phase 1, nodes are blocked.

---

### Interview Questions

**Q: When would you choose NoSQL over SQL?**

A: "NoSQL when: (1) data is unstructured or schema changes frequently, (2) you need massive horizontal scalability, (3) high write throughput is needed, (4) eventual consistency is acceptable. SQL when: data is structured with relationships, ACID transactions are critical, complex queries with joins are needed."

**Q: Explain the CAP theorem.**

A: "A distributed system can guarantee at most 2 of 3: Consistency (every read gets latest write), Availability (every request gets response), Partition Tolerance (works despite network failures). Since partitions are inevitable, choose CP (consistent but may be unavailable) or AP (available but may be inconsistent)."

**Q: What's sharding?**

A: "Horizontal partitioning—splitting data across multiple servers. Each shard holds a subset of data. Strategies: range-based, hash-based, or directory-based. Benefits: horizontal scalability. Challenges: cross-shard queries, rebalancing, distributed transactions."

**Q: What's the difference between sharding and replication?**

A: "Sharding: splits data across servers (each server has different data). Replication: copies data across servers (each server has same data). Sharding for write scalability; replication for read scalability and fault tolerance. Often used together."

---

*Next: [13 — SQL Fundamentals](13-SQL-Fundamentals.md)*
