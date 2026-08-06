# Pattern Recognition Cheat Sheet

## The One-Page Reference for Identifying Patterns

---

### Array/String Problems

| Trigger Words | Pattern | Chapter |
|---------------|---------|---------|
| "contiguous subarray/substring" | Sliding Window | 01 |
| "longest/shortest/most/least" + contiguous | Sliding Window | 01 |
| "at most K distinct" | Sliding Window | 01 |
| "sorted array" + "find pair" | Two Pointers | 02 |
| "sorted array" + "target" | Binary Search | 02 or 03 |
| "minimum feasible answer" | Binary Search on Answer | 03 |
| "maximum feasible answer" | Binary Search on Answer | 03 |
| "subarray sum equals K" | Prefix Sum + HashMap | 04 |
| "running sum" / "cumulative" | Prefix Sum | 04 |
| "next greater/smaller" | Monotonic Stack | 05 |
| "stock span" / "temperature" | Monotonic Stack | 05 |
| "top K" / "Kth largest" | Heap | 06 |
| "K closest" | Heap | 06 |

---

### Tree Problems

| Trigger Words | Pattern | Chapter |
|---------------|---------|---------|
| "binary tree" + "traversal" | DFS (pre/in/post) | 08 |
| "level order" | BFS | 08 |
| "depth" / "height" | DFS divide & conquer | 08 |
| "path sum" | DFS + Prefix Sum | 08 |
| "validate BST" | DFS with range | 08 |
| "lowest common ancestor" | DFS divide & conquer | 08 |
| "serialize/deserialize" | DFS traversal | 08 |

---

### Graph Problems

| Trigger Words | Pattern | Chapter |
|---------------|---------|---------|
| "connected components" | DFS or Union Find | 09 or 13 |
| "shortest path" (unweighted) | BFS | 09 |
| "shortest path" (positive weights) | Dijkstra | 15 |
| "shortest path" (negative weights) | Bellman-Ford | 15 |
| "all pairs shortest" | Floyd-Warshall | 15 |
| "prerequisites" / "dependencies" | Topological Sort | 14 |
| "can you finish" / "cycle" | Topological Sort | 14 |
| "groups" / "merge sets" | Union Find | 13 |
| "islands" / "flood fill" | DFS/BFS on grid | 09 |
| "minimum steps" | BFS | 09 |

---

### Dynamic Programming

| Trigger Words | Pattern | Chapter |
|---------------|---------|---------|
| "minimum/maximum" | DP | 11 |
| "longest/shortest" | DP | 11 |
| "number of ways" | DP | 11 |
| "can you reach" | DP or Greedy | 11 or 12 |
| "subsequence" | DP | 11 |
| "knapsack" / "choose items" | DP (Knapsack) | 11 |
| "coin change" | DP (Unbounded Knapsack) | 11 |
| "edit distance" | DP (String) | 11 |

---

### Other Patterns

| Trigger Words | Pattern | Chapter |
|---------------|---------|---------|
| "all combinations/permutations" | Backtracking | 10 |
| "generate all valid" | Backtracking | 10 |
| "intervals" + "non-overlapping" | Greedy (sort by end) | 12 |
| "jump" / "reach" | Greedy | 12 |
| "single number" / "unique" | Bit Manipulation | 16 |
| "power of two" | Bit Manipulation | 16 |
| "reverse linked list" | Linked List (three pointers) | 07 |
| "cycle in linked list" | Fast & Slow pointers | 07 |
| "merge sorted lists" | Linked List / Heap | 07 or 06 |

---

### The Decision Flowchart

```
START
  │
  ├─ Is it about arrays/strings?
  │   ├─ Contiguous? → Sliding Window
  │   ├─ Sorted? → Two Pointers or Binary Search
  │   ├─ Subarray sum? → Prefix Sum + HashMap
  │   ├─ Next greater/smaller? → Monotonic Stack
  │   └─ Top K? → Heap
  │
  ├─ Is it about trees?
  │   ├─ Traverse all? → DFS
  │   ├─ Level by level? → BFS
  │   └─ BST property? → DFS with range
  │
  ├─ Is it about graphs?
  │   ├─ Shortest path? → BFS/Dijkstra
  │   ├─ Dependencies? → Topological Sort
  │   ├─ Groups/merges? → Union Find
  │   └─ Explore all? → DFS
  │
  ├─ Is it about optimization (min/max/count)?
  │   ├─ Overlapping subproblems? → DP
  │   ├─ Greedy choice works? → Greedy
  │   └─ All possibilities? → Backtracking
  │
  └─ Is it about bits/unique elements?
      └─ Bit Manipulation
```

---

### Quick Algorithm Selection

| Problem Type | Algorithm | Time |
|--------------|-----------|------|
| Shortest path (unweighted) | BFS | O(V+E) |
| Shortest path (weighted, positive) | Dijkstra | O((V+E) log V) |
| Shortest path (negative weights) | Bellman-Ford | O(V×E) |
| All pairs shortest | Floyd-Warshall | O(V³) |
| Top K elements | Heap | O(n log k) |
| Kth element in sorted matrix | Binary Search | O(n log(max-min)) |
| Connected components | Union Find | O(n α(n)) |
| Dependencies | Topological Sort | O(V+E) |
| Subsets/Combinations | Backtracking | O(2^n) |
| Optimization with overlapping subproblems | DP | Varies |

---

### Pattern Mixing

Some problems combine multiple patterns:

| Problem | Patterns Used |
|---------|---------------|
| Sliding Window Maximum | Sliding Window + Monotonic Stack |
| Shortest Path with K stops | BFS + DP or Bellman-Ford |
| Binary Tree Path Sum III | Tree DFS + Prefix Sum |
| Longest Increasing Path in Matrix | DFS + Memoization |
| Swim in Rising Water | Binary Search + DFS or Dijkstra |
| Kth Smallest in Sorted Matrix | Binary Search + Two Pointers |

---

### The Recognition Exercise

For each problem statement, identify the pattern:

1. "Find the longest substring with at most K distinct characters"
2. "Given a sorted array, find two numbers that sum to target"
3. "Find the minimum speed to finish eating bananas in H hours"
4. "Count subarrays with sum equal to K"
5. "Find the next greater element for each element"
6. "Find the Kth largest element"
7. "Find the number of connected components"
8. "Find the shortest path in an unweighted graph"
9. "Find all permutations of a string"
10. "Find the minimum number of coins to make amount"

**Answers:**
1. Sliding Window
2. Two Pointers
3. Binary Search on Answer
4. Prefix Sum + HashMap
5. Monotonic Stack
6. Heap
7. Union Find or DFS
8. BFS
9. Backtracking
10. DP (Coin Change)
