# 08 — B-Trees & B+ Trees

## How Indexes Actually Work

---

### Why B-Trees?

Databases need to:
1. Search quickly (O(log n))
2. Insert/delete efficiently
3. Work with disk storage (minimize disk I/O)

**B-Trees and B+ Trees** are designed for disk-based storage—they minimize disk accesses.

---

### B-Tree

A self-balancing tree where each node can have multiple keys and children.

**Properties:**
- Each node has at most `m` children (m = order)
- Each node (except root) has at least ⌈m/2⌉ children
- Root has at least 2 children (unless leaf)
- All leaves are at the same level
- A node with `k` keys has `k+1` children

```
B-Tree of order 3:
              ┌─────┐
              │ 10  │
              │ 20  │
              └──┬──┘
         ┌─────┼─────┐
         │     │     │
      ┌──┴──┐ ┌┴──┐ ┌┴────┐
      │ 5   │ │15 │ │ 25  │
      └─────┘ └───┘ │ 30  │
                    └─────┘
```

**Each node stores keys AND data (or pointers to data).**

---

### B+ Tree

**Improvement over B-Tree.** Used by most databases (MySQL InnoDB, PostgreSQL, Oracle).

```
B+ Tree:
┌─────────────────────────────────────┐
│ Internal Nodes (only keys + pointers)│
│                                     │
│         ┌───────────┐              │
│         │ 10  │ 20  │              │
│         └──┬─────┬──┘              │
│      ┌─────┘     └─────┐          │
│   ┌──┴───┐         ┌───┴──┐       │
│   │5 │ 8 │         │15│18│       │
│   └──┬───┘         └──┬───┘       │
│      │                 │           │
└──────┼─────────────────┼───────────┘
       │                 │
┌──────┼─────────────────┼───────────┐
│ Leaf Nodes (keys + data, linked)    │
│                                     │
│ ┌───┬───┐  ┌───┬───┐  ┌───┬───┐  │
│ │ 5 │ 8 │──│15 │18 │──│25 │30 │  │
│ └───┴───┘  └───┴───┘  └───┴───┘  │
└─────────────────────────────────────┘
```

**Key differences from B-Tree:**

| Aspect | B-Tree | B+ Tree |
|--------|--------|---------|
| **Data storage** | All nodes | Only leaf nodes |
| **Leaf linking** | No | Yes (linked list) |
| **Range queries** | Traversal needed | Sequential scan of leaves |
| **Internal nodes** | Store data | Only keys + pointers |
| **Fanout** | Lower | Higher (more keys per node) |

**B+ Tree is better for databases because:**
1. Higher fanout (more children per node) → fewer disk I/Os
2. Linked leaves → efficient range queries
3. All data at leaves → predictable performance

---

### B+ Tree Operations

#### Search (e.g., find key 18)

```
Start at root:
         ┌───────────┐
         │ 10  │ 20  │  ← 18 > 10, 18 < 20
         └─────┴─────┘
              │
    Go to middle child
              │
         ┌────┴────┐
         │15  │ 18 │  ← Found! (or go to leaf)
         └─────────┘
```

**Time complexity:** O(log_m n) where m = order, n = keys
**Disk accesses:** Height of tree (typically 3-4 for millions of rows)

#### Insert (e.g., insert 16)

```
1. Find correct leaf node
2. Insert key in sorted order
3. If leaf is full → Split
   - Create new leaf
   - Move half keys to new leaf
   - Insert middle key into parent
4. If parent is full → Split parent (recursive)
```

#### Delete

```
1. Find and remove key
2. If node underflows (too few keys):
   - Borrow from sibling (if sibling has enough)
   - Merge with sibling (if sibling is minimal)
3. May need to update parent keys
```

---

### Disk I/O Calculation

```
B+ Tree with order m = 100, height h = 3:

Level 0 (Root):    1 node    → 1 disk read
Level 1:           100 nodes → 1 disk read (pick right child)
Level 2 (Leaf):    10,000    → 1 disk read (find key)

Total: 3 disk reads for any lookup!

Compare to scanning 1,000,000 rows: ~100,000 disk reads
```

**Rule of thumb:** B+ Tree of height 3 can handle millions of rows with 3 disk accesses.

---

### Choosing m (Order)

Each node = one disk page (typically 4KB or 8KB).

```
Node size = 4KB
Key size = 8 bytes
Pointer size = 8 bytes
m = node_size / (key_size + pointer_size) = 4096 / 16 ≈ 256

Height 3 B+ Tree: 256^3 = ~16 million entries
```

---

### Interview Questions

**Q: What's the difference between B-Tree and B+ Tree?**

A: "B+ Tree stores data only in leaf nodes; B-Tree stores in all nodes. B+ Tree leaves are linked for efficient range queries. B+ Tree has higher fanout (more keys per internal node) because internal nodes don't store data. Databases use B+ Trees for these advantages."

**Q: Why do databases use B+ Trees instead of hash tables?**

A: "Hash tables: O(1) for equality lookups but can't do range queries. B+ Trees: O(log n) for lookups AND efficient range queries (just scan linked leaves). Since SQL queries often use range conditions (>, <, BETWEEN), B+ Trees are more versatile."

**Q: How many disk accesses does a B+ Tree lookup require?**

A: Equal to the tree height. With order 100 and height 3: 3 disk accesses for any lookup. That covers ~1 million entries. With larger pages (8KB), order increases, height decreases—often just 2-3 disk accesses even for billions of rows.

**Q: What happens during a B+ Tree split?**

A: "When a leaf is full during insert: (1) create new leaf, (2) move half the keys to new leaf, (3) copy the middle key up to the parent. If parent is full, split it too (propagates up). The tree grows in height only when the root splits."

---

*Next: [09 — Joins Deep Dive](09-Joins-Deep-Dive.md)*
