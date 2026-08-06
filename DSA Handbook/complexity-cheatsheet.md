# Complexity Cheatsheet

## Time and Space Complexity for Every Pattern

---

### Data Structure Operations

| Data Structure | Access | Search | Insert | Delete | Space |
|----------------|--------|--------|--------|--------|-------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Dynamic Array | O(1) | O(n) | O(1)* | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1) | O(1) | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash Map | - | O(1)* | O(1)* | O(1)* | O(n) |
| Binary Search Tree | O(log n)* | O(log n)* | O(log n)* | O(log n)* | O(n) |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Heap (Binary) | - | O(n) | O(log n) | O(log n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(m) | O(n×m) |

*Amortized or average case

---

### Pattern Complexities

| Pattern | Time | Space | Notes |
|---------|------|-------|-------|
| **Sliding Window** | O(n) | O(k) | k = window size or character set |
| **Two Pointers** | O(n) | O(1) | After sorting: O(n log n) |
| **Binary Search** | O(log n) | O(1) | On answer: O(n × log(max)) |
| **Prefix Sum + HashMap** | O(n) | O(n) | Single pass |
| **Monotonic Stack** | O(n) | O(n) | Each element pushed/popped once |
| **Heap** | O(n log k) | O(k) | k = heap size |
| **Linked List** | O(n) | O(1) | Recursive: O(n) stack space |
| **Trees (DFS)** | O(n) | O(h) | h = height (O(log n) balanced) |
| **Trees (BFS)** | O(n) | O(w) | w = max width |
| **DFS (Graph)** | O(V + E) | O(V) | V = vertices, E = edges |
| **BFS (Graph)** | O(V + E) | O(V) | |
| **Backtracking** | O(2^n) | O(n) | Subsets; Permutations: O(n!) |
| **DP (1D)** | O(n) | O(n) or O(1) | Can often optimize space |
| **DP (2D)** | O(m × n) | O(m × n) | Can optimize to O(min(m,n)) |
| **DP (Knapsack)** | O(n × W) | O(n × W) | Can optimize to O(W) |
| **Greedy** | O(n log n) | O(1) or O(n) | Sorting dominates |
| **Union Find** | O(α(n)) | O(n) | α = inverse Ackermann ≈ O(1) |
| **Topological Sort** | O(V + E) | O(V) | Kahn's or DFS |
| **Dijkstra** | O((V + E) log V) | O(V) | With binary heap |
| **Bellman-Ford** | O(V × E) | O(V) | |
| **Floyd-Warshall** | O(V³) | O(V²) | All pairs shortest path |
| **Bit Manipulation** | O(n) | O(1) | |

---

### Common Algorithm Complexities

| Algorithm | Time | Space | Use Case |
|-----------|------|-------|----------|
| Binary Search | O(log n) | O(1) | Search in sorted array |
| Merge Sort | O(n log n) | O(n) | Stable sort |
| Quick Sort | O(n log n)* | O(log n) | In-place sort |
| Heap Sort | O(n log n) | O(1) | In-place, not stable |
| Counting Sort | O(n + k) | O(k) | k = range of values |
| Radix Sort | O(d × n) | O(n + k) | d = digits |
| BFS | O(V + E) | O(V) | Shortest path (unweighted) |
| DFS | O(V + E) | O(V) | Exploration |
| Dijkstra | O((V + E) log V) | O(V) | Shortest path (positive weights) |
| Bellman-Ford | O(V × E) | O(V) | Negative weights |
| Kruskal's MST | O(E log E) | O(V) | Minimum spanning tree |
| Prim's MST | O(E log V) | O(V) | Minimum spanning tree |

*Average case

---

### Input Size vs. Acceptable Complexity

| Input Size | Acceptable Time Complexity | Example Problems |
|------------|---------------------------|------------------|
| n ≤ 10 | O(n!), O(2^n) | Permutations, subsets |
| n ≤ 20 | O(2^n) | Bitmask DP |
| n ≤ 100 | O(n³) | Floyd-Warshall, interval DP |
| n ≤ 500 | O(n³) | Matrix chain multiplication |
| n ≤ 5000 | O(n²) | LIS, LCS |
| n ≤ 10^5 | O(n log n) | Sorting, heap operations |
| n ≤ 10^6 | O(n) | Linear scan, hash map |
| n ≤ 10^9 | O(log n) | Binary search |
| n ≤ 10^18 | O(log n) | Binary search, fast exponentiation |

---

### Space Complexity Notes

| Optimization | Technique | Example |
|--------------|-----------|---------|
| DP 2D → 1D | Keep only previous row | LCS, edit distance |
| DP 1D → O(1) | Keep only last 2 values | Fibonacci, climbing stairs |
| Recursive → Iterative | Use explicit stack | Tree traversal |
| In-place | Modify input array | Two pointers, partitioning |

---

### Amortized Analysis

Some operations are O(1) amortized even though individual operations may be O(n):

| Operation | Worst Case | Amortized | Why |
|-----------|------------|-----------|-----|
| Dynamic array append | O(n) | O(1) | Doubling strategy |
| Union Find (with compression) | O(log n) | O(α(n)) | Path compression |
| Monotonic stack (per element) | O(n) | O(1) | Each element pushed/popped once |

---

### The Master Theorem

For divide-and-conquer algorithms with recurrence T(n) = aT(n/b) + O(n^d):

| Condition | Complexity | Example |
|-----------|------------|---------|
| d < log_b(a) | O(n^(log_b(a))) | Strassen's matrix multiply |
| d = log_b(a) | O(n^d × log n) | Merge sort |
| d > log_b(a) | O(n^d) | Quickselect (average) |

---

### Complexity Quick Reference

**O(1)** — Constant: Hash map lookup, array access, stack push/pop

**O(log n)** — Logarithmic: Binary search, balanced BST operations, heap insert/delete

**O(n)** — Linear: Single pass through array, linked list traversal, DFS/BFS

**O(n log n)** — Linearithmic: Sorting, heap sort, divide-and-conquer

**O(n²)** — Quadratic: Nested loops, bubble sort, LIS (basic)

**O(n³)** — Cubic: Floyd-Warshall, matrix multiplication (basic)

**O(2^n)** — Exponential: Subsets, backtracking without pruning

**O(n!)** — Factorial: Permutations, brute force TSP

---

### Interview Complexity Answers

When asked "What's the time complexity?":

| Pattern | Standard Answer |
|---------|-----------------|
| Sliding Window | "O(n) because each element enters and leaves the window at most once." |
| Two Pointers | "O(n) because each pointer moves at most n steps." |
| Binary Search | "O(log n) because we halve the search space each step." |
| Prefix Sum | "O(n) for one pass. O(1) for each range query after precomputation." |
| Monotonic Stack | "O(n) because each element is pushed and popped at most once." |
| Heap | "O(n log k) because we do n heap operations of size k." |
| DFS/BFS | "O(V + E) because we visit each vertex and edge once." |
| Backtracking | "O(2^n) for subsets, O(n!) for permutations." |
| DP | "O(n × m) where n and m are the dimensions of the DP table." |
| Greedy | "O(n log n) due to sorting. The greedy part is O(n)." |
| Union Find | "Nearly O(1) per operation due to path compression and union by rank." |
| Dijkstra | "O((V + E) log V) with a binary heap." |

---

### Common Mistakes in Complexity Analysis

| Mistake | Correct |
|---------|---------|
| "DFS is O(n)" | "DFS is O(V + E) for graphs, O(n) for trees" |
| "Binary search is O(log n) always" | "Only if the data is sorted or has monotonic property" |
| "DP is always O(n²)" | "Depends on state and transition; could be O(n), O(n×W), etc." |
| "Sorting is O(n log n)" | "Comparison sorts are O(n log n); counting/radix sort can be O(n)" |
| "Hash map is O(1)" | "Average O(1), worst case O(n) with collisions" |

---

### Final Notes

1. **Always state both time and space complexity** in interviews.

2. **Mention the worst case** unless the average case is significantly different and relevant.

3. **Explain your reasoning** — interviewers want to see you understand *why* it's that complexity.

4. **Consider the input size** to verify your solution will run in time.

5. **Space complexity matters** — especially for recursive solutions (stack space) and DP (table size).

---

*Use this cheatsheet to verify your complexity analysis during problem-solving.*
