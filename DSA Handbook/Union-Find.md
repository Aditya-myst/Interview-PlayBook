# 13 — Union Find

## The Pattern That Tracks Connected Components

---

### What It Is

Union Find (also called Disjoint Set Union, DSU) is a data structure that efficiently tracks which elements belong to which components and merges components together.

**The key insight:** Union Find answers "are these two elements in the same component?" in nearly O(1) time, making it ideal for problems involving connectivity, groups, or clusters.

---

### When to Use It

**The problem involves:**
- Connected components in a graph
- Detecting cycles
- Grouping elements
- Merging sets
- Counting components

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "connected components" | Number of Connected Components |
| "friend circles" | Friend Circles |
| "islands" (counting) | Number of Islands II |
| "groups" | Accounts Merge |
| "redundant connection" | Redundant Connection |
| "merge" + "sets" | Merging elements |
| "same group" | Similar String Groups |
| "equivalence" | Satisfiability of Equality Equations |

**The Decision:**

```
Does the problem involve grouping or connectivity?
    ↓
Yes
    ↓
Do elements get merged over time (dynamic)?
    ↓
Yes
    ↓
UNION FIND
```

---

### Mental Model: Family Trees

Think of Union Find as tracking family groups:
- Initially, each person is their own family (parent = self)
- `union(A, B)` merges two families into one
- `find(A)` returns the "family representative" (root parent)
- Two people are in the same family if `find(A) == find(B)`

```
Initial:      After union(1,2):   After union(3,4):   After union(2,4):
  1  2  3  4      1    3  4          1    3              1
                  \      |           \    |              / \
                   2    4             2   4             2   3
                                                              \
                                                               4
```

---

### The 80% Template

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n  # Optional: track component count
    
    def find(self, x):
        # Path compression: make every node point to root
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        # Union by rank: attach smaller tree under larger
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False  # Already in same component
        
        if self.rank[root_x] < self.rank[root_y]:
            root_x, root_y = root_y, root_x
        
        self.parent[root_y] = root_x
        if self.rank[root_x] == self.rank[root_y]:
            self.rank[root_x] += 1
        
        self.components -= 1
        return True
    
    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

**Two optimizations:**
1. **Path Compression:** In `find`, make every node point directly to the root. This flattens the tree.
2. **Union by Rank:** Always attach the smaller tree under the larger one. This keeps trees shallow.

With both optimizations, each operation is nearly O(1) — specifically O(α(n)) where α is the inverse Ackermann function (effectively ≤ 4 for any practical n).

---

### Dry Run: Number of Connected Components

**Problem:** Given n nodes and a list of edges, count the number of connected components.

**Input:** `n = 5`, `edges = [[0,1], [1,2], [3,4]]`

```python
def countComponents(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.components
```

**Step-by-step:**

```
n = 5, edges = [[0,1], [1,2], [3,4]]

Initial: parent = [0, 1, 2, 3, 4], components = 5

Process edge [0,1]:
  find(0) = 0, find(1) = 1
  union(0, 1): parent[1] = 0, components = 4
  parent = [0, 0, 2, 3, 4]

Process edge [1,2]:
  find(1) = find(0) = 0 (path compression)
  find(2) = 2
  union(1, 2): parent[2] = 0, components = 3
  parent = [0, 0, 0, 3, 4]

Process edge [3,4]:
  find(3) = 3, find(4) = 4
  union(3, 4): parent[4] = 3, components = 2
  parent = [0, 0, 0, 3, 3]

Return 2 (components: {0,1,2} and {3,4})
```

---

### Dry Run: Redundant Connection

**Problem:** In a graph that started as a tree with n nodes, one extra edge was added. Find the edge that creates a cycle.

**Input:** `edges = [[1,2], [1,3], [2,3]]`

**Recognition:** "redundant" + "connection" + "cycle" → Union Find

**Insight:** When we try to union two nodes that are already connected, that edge creates a cycle.

```python
def findRedundantConnection(edges):
    n = len(edges)
    uf = UnionFind(n + 1)  # 1-indexed
    
    for u, v in edges:
        if not uf.union(u, v):  # Already connected!
            return [u, v]
    
    return []
```

**Step-by-step:**

```
edges = [[1,2], [1,3], [2,3]]

Initial: parent = [0, 1, 2, 3], components = 4

Process [1,2]:
  find(1)=1, find(2)=2
  union(1,2) → True (merged)
  parent = [0, 1, 1, 3], components = 3

Process [1,3]:
  find(1)=1, find(3)=3
  union(1,3) → True (merged)
  parent = [0, 1, 1, 1], components = 2

Process [2,3]:
  find(2)=find(1)=1, find(3)=find(1)=1
  Same component! union returns False
  Return [2, 3] — this is the redundant edge
```

---

### Dry Run: Accounts Merge

**Problem:** Given accounts where each account has a name and emails, merge accounts with common emails.

**Input:** `accounts = [["John","john@mail","john2@mail"],["John","john@mail","john3@mail"],["Mary","mary@mail"]]`

**Insight:** Two accounts belong to the same person if they share an email. Use Union Find to group accounts by shared emails.

```python
def accountsMerge(accounts):
    uf = UnionFind(len(accounts))
    email_to_id = {}
    
    # Build union: if email seen before, merge with previous account
    for i, account in enumerate(accounts):
        for email in account[1:]:
            if email in email_to_id:
                uf.union(i, email_to_id[email])
            email_to_id[email] = i
    
    # Group emails by root account
    from collections import defaultdict
    groups = defaultdict(set)
    for email, id in email_to_id.items():
        root = uf.find(id)
        groups[root].add(email)
    
    # Build result
    result = []
    for root, emails in groups.items():
        result.append([accounts[root][0]] + sorted(emails))
    
    return result
```

**Step-by-step:**

```
accounts = [
  ["John","john@mail","john2@mail"],    # account 0
  ["John","john@mail","john3@mail"],    # account 1
  ["Mary","mary@mail"]                  # account 2
]

Process account 0:
  email_to_id = {"john@mail": 0, "john2@mail": 0}

Process account 1:
  "john@mail" already seen → union(1, 0)
  email_to_id = {"john@mail": 0, "john2@mail": 0, "john3@mail": 1}

Process account 2:
  email_to_id = {"john@mail": 0, "john2@mail": 0, "john3@mail": 1, "mary@mail": 2}

Groups by root:
  find(0) = 0: {"john@mail", "john2@mail"}
  find(1) = find(0) = 0: {"john3@mail"}
  find(2) = 2: {"mary@mail"}

Combined:
  root 0: {"john@mail", "john2@mail", "john3@mail"}
  root 2: {"mary@mail"}

Result: [["John", "john@mail", "john2@mail", "john3@mail"], ["Mary", "mary@mail"]]
```

---

### Template Variations

#### Without Rank (Simpler)

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x != root_y:
            self.parent[root_y] = root_x
            return True
        return False
```

#### With Size Tracking

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x == root_y:
            return False
        if self.size[root_x] < self.size[root_y]:
            root_x, root_y = root_y, root_x
        self.parent[root_y] = root_x
        self.size[root_x] += self.size[root_y]
        return True
```

---

### Code Templates (4 Languages)

#### Python

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x == root_y:
            return False
        if self.rank[root_x] < self.rank[root_y]:
            root_x, root_y = root_y, root_x
        self.parent[root_y] = root_x
        if self.rank[root_x] == self.rank[root_y]:
            self.rank[root_x] += 1
        return True
```

#### Java

```java
class UnionFind {
    int[] parent, rank;
    
    UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    
    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank[rx] < rank[ry]) { int t = rx; rx = ry; ry = t; }
        parent[ry] = rx;
        if (rank[rx] == rank[ry]) rank[rx]++;
        return true;
    }
}
```

#### C++

```cpp
class UnionFind {
public:
    vector<int> parent, rank;
    
    UnionFind(int n) : parent(n), rank(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }
    
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank[rx] < rank[ry]) swap(rx, ry);
        parent[ry] = rx;
        if (rank[rx] == rank[ry]) rank[rx]++;
        return true;
    }
};
```

#### JavaScript

```javascript
class UnionFind {
    constructor(n) {
        this.parent = Array.from({length: n}, (_, i) => i);
        this.rank = new Array(n).fill(0);
    }
    
    find(x) {
        if (this.parent[x] !== x) {
            this.parent[x] = this.find(this.parent[x]);
        }
        return this.parent[x];
    }
    
    union(x, y) {
        let rx = this.find(x), ry = this.find(y);
        if (rx === ry) return false;
        if (this.rank[rx] < this.rank[ry]) [rx, ry] = [ry, rx];
        this.parent[ry] = rx;
        if (this.rank[rx] === this.rank[ry]) this.rank[rx]++;
        return true;
    }
}
```

---

### Common Mistakes

#### Mistake 1: Forgetting Path Compression

```python
# WRONG: without path compression, find can be O(n)
def find(self, x):
    while self.parent[x] != x:
        x = self.parent[x]
    return x

# RIGHT: with path compression
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])
    return self.parent[x]
```

#### Mistake 2: 0-Indexed vs 1-Indexed

```python
# WRONG: if nodes are 1-indexed but array is 0-indexed
uf = UnionFind(n)  # Only n nodes, but nodes go from 1 to n

# RIGHT: allocate n+1 for 1-indexed nodes
uf = UnionFind(n + 1)
```

#### Mistake 3: Not Checking if Already Connected

```python
# WRONG: always merges (wastes time)
def union(self, x, y):
    self.parent[self.find(y)] = self.find(x)

# RIGHT: check first
def union(self, x, y):
    root_x, root_y = self.find(x), self.find(y)
    if root_x == root_y:
        return False  # Already connected
    self.parent[root_y] = root_x
    return True
```

---

### Complexity Analysis

| Operation | Time (with optimizations) | Space |
|-----------|---------------------------|-------|
| Find | O(α(n)) ≈ O(1) | O(n) |
| Union | O(α(n)) ≈ O(1) | O(n) |
| Connected | O(α(n)) ≈ O(1) | O(n) |
| Build | O(n) | O(n) |

**α(n)** is the inverse Ackermann function, which is ≤ 4 for any practical input size.

---

### Practice Problems

#### Easy-Medium

| # | Problem | Key |
|---|---------|-----|
| 1 | [323. Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Basic Union Find |
| 2 | [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | Friend circles |
| 3 | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | Grid Union Find |
| 4 | [684. Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Cycle detection |
| 5 | [721. Accounts Merge](https://leetcode.com/problems/accounts-merge/) | Email grouping |
| 6 | [990. Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) | Equality/inequality |
| 7 | [1254. Number of Closed Islands](https://leetcode.com/problems/number-of-closed-islands/) | Grid |

#### Medium-Hard

| # | Problem | Key |
|---|---------|-----|
| 8 | [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Size tracking |
| 9 | [839. Similar String Groups](https://leetcode.com/problems/similar-string-groups/) | String similarity |
| 10 | [1319. Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) | Count extra edges |
| 11 | [1631. Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Binary search + UF |
| 12 | [305. Number of Islands II](https://leetcode.com/problems/number-of-islands-ii/) | Dynamic grid UF |

---

### Interview Tips

1. **Explain the data structure.** "Union Find tracks which elements are in the same group using a parent array."

2. **Mention optimizations.** "With path compression and union by rank, operations are nearly O(1)."

3. **Show the pattern.** "For each edge, I union the two endpoints. If they're already connected, I've found [cycle / redundancy]."

4. **Edge cases:**
   - No edges (each node is its own component)
   - Self-loops
   - Disconnected components
   - All nodes in one component

---

*Next: [14 — Topological Sort](14-Topological-Sort.md)*
