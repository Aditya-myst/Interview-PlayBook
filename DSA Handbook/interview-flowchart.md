# Interview Decision Flowchart

## Use This During Interviews to Identify the Right Pattern

---

### Step 1: What Type of Data Structure?

```
What is the input?
│
├─ Array or String?
│   └─ Go to Step 2A
│
├─ Linked List?
│   └─ Go to Step 2B
│
├─ Tree?
│   └─ Go to Step 2C
│
├─ Graph?
│   └─ Go to Step 2D
│
└─ Other (Matrix, Numbers, etc.)?
    └─ Go to Step 2E
```

---

### Step 2A: Array/String Problems

```
What does the problem ask?
│
├─ Contiguous subarray/substring?
│   ├─ Find longest/shortest? → Sliding Window
│   ├─ Find sum = K? → Prefix Sum + HashMap
│   └─ Fixed window size? → Sliding Window (fixed)
│
├─ Sorted array?
│   ├─ Find pair with target sum? → Two Pointers (opposite ends)
│   ├─ Find specific value? → Binary Search
│   └─ Find first/last occurrence? → Binary Search (variants)
│
├─ Find minimum/maximum feasible answer?
│   └─ Binary Search on Answer
│
├─ Find next greater/smaller element?
│   └─ Monotonic Stack
│
├─ Top K or Kth element?
│   └─ Heap
│
├─ Count subarrays with condition?
│   └─ Prefix Sum + HashMap
│
├─ All combinations/permutations?
│   └─ Backtracking
│
└─ Minimize/maximize with overlapping subproblems?
    └─ Dynamic Programming
```

---

### Step 2B: Linked List Problems

```
What does the problem ask?
│
├─ Reverse the list?
│   └─ Three pointers (prev, curr, next)
│
├─ Find middle?
│   └─ Fast & Slow pointers
│
├─ Detect cycle?
│   └─ Fast & Slow pointers (Floyd's)
│
├─ Merge sorted lists?
│   └─ Dummy head + compare
│
├─ Remove nth from end?
│   └─ Two pointers with gap
│
└─ Check palindrome?
    └─ Find middle + reverse second half + compare
```

---

### Step 2C: Tree Problems

```
What does the problem ask?
│
├─ Traverse all nodes?
│   ├─ Inorder (sorted for BST)? → DFS (inorder)
│   ├─ Preorder (serialization)? → DFS (preorder)
│   └─ Postorder (deletion)? → DFS (postorder)
│
├─ Level-by-level?
│   └─ BFS
│
├─ Find depth/height?
│   └─ DFS (divide & conquer)
│
├─ Path from root to leaf?
│   └─ DFS + backtracking
│
├─ Path sum?
│   └─ DFS + prefix sum
│
├─ Validate BST?
│   └─ DFS with range (lo, hi)
│
├─ Lowest Common Ancestor?
│   └─ DFS (check left and right)
│
└─ Serialize/Deserialize?
    └─ DFS traversal (preorder with nulls)
```

---

### Step 2D: Graph Problems

```
What does the problem ask?
│
├─ Shortest path?
│   ├─ Unweighted? → BFS
│   ├─ Positive weights? → Dijkstra
│   ├─ Negative weights? → Bellman-Ford
│   └─ All pairs? → Floyd-Warshall
│
├─ Connected components?
│   ├─ Static? → DFS or Union Find
│   └─ Dynamic (add edges)? → Union Find
│
├─ Dependencies/ordering?
│   └─ Topological Sort
│
├─ Cycle detection?
│   ├─ Undirected? → DFS or Union Find
│   └─ Directed? → DFS (three colors)
│
├─ Explore all reachable?
│   └─ DFS or BFS
│
└─ Grid (islands, flood fill)?
    └─ DFS/BFS on grid
```

---

### Step 2E: Other Problems

```
What does the problem ask?
│
├─ Minimum/maximum/optimization?
│   ├─ Greedy choice works? → Greedy
│   └─ Need all possibilities? → DP
│
├─ Count something?
│   ├─ Number of ways? → DP
│   └─ Number of valid arrangements? → Backtracking
│
├─ Generate all valid solutions?
│   └─ Backtracking
│
├─ Single/unique element?
│   └─ Bit Manipulation (XOR)
│
├─ Groups/sets merging?
│   └─ Union Find
│
├─ Intervals?
│   ├─ Maximum non-overlapping? → Greedy (sort by end)
│   └─ Merge overlapping? → Greedy (sort by start)
│
└─ Scheduling with constraints?
    ├─ Can finish? → Topological Sort
    └─ Minimum time? → Greedy or DP
```

---

### Step 3: Verify Your Choice

After choosing a pattern, verify:

1. **Does the pattern fit the constraints?**
   - Sliding Window: needs contiguous elements
   - Binary Search: needs monotonic property
   - Greedy: needs greedy choice property

2. **What's the time complexity?**
   - Is it acceptable for the input size?
   - n ≤ 20: O(2^n) backtracking is fine
   - n ≤ 1000: O(n²) DP is fine
   - n ≤ 10^5: O(n log n) is needed
   - n ≤ 10^6: O(n) is needed

3. **What's the space complexity?**
   - Can you optimize it?
   - DP: can you use O(n) instead of O(n²)?

---

### Step 4: Implement

1. **State your approach clearly**
2. **Define the data structures you'll use**
3. **Walk through a small example**
4. **Code the solution**
5. **Test with edge cases**

---

### Common Edge Cases

| Problem Type | Edge Cases |
|--------------|------------|
| Array | Empty, single element, all same, sorted/reverse sorted |
| String | Empty, single char, all same char, palindrome |
| Tree | Empty, single node, skewed, balanced |
| Graph | Disconnected, self-loops, cycles, single node |
| Linked List | Empty, single node, cycle, head removal |
| DP | n=0, n=1, all zeros, negative numbers |

---

### The "I'm Stuck" Checklist

If you're stuck during an interview:

1. **Can I solve it with brute force?** Start there, then optimize.
2. **Is the data sorted?** If not, can I sort it?
3. **Can I use a hash map for O(1) lookup?**
4. **Is there a monotonic property?** Binary search or monotonic stack.
5. **Do I need all possibilities or just one?** Backtracking vs greedy/DP.
6. **Can I reduce this to a known problem?** (e.g., "This is just shortest path in disguise")

---

### Pattern Combinations

Some problems use multiple patterns:

| Combination | Example |
|-------------|---------|
| Sliding Window + Monotonic Stack | Sliding Window Maximum |
| Binary Search + DFS | Swim in Rising Water |
| DFS + Memoization | Longest Increasing Path |
| Tree DFS + Prefix Sum | Path Sum III |
| BFS + DP | Minimum Steps |
| Greedy + Heap | Task Scheduler |
| Union Find + Binary Search | Minimum Spanning Tree |

---

### The 5-Minute Interview Strategy

**Minute 1:** Understand the problem. Ask clarifying questions.

**Minute 2:** Identify the pattern. Use this flowchart.

**Minute 3:** Outline the approach. State time/space complexity.

**Minute 4-8:** Code the solution.

**Minute 9-10:** Test with examples and edge cases.

---

### Pattern Priority for Interviews

If you're short on preparation time, prioritize:

1. **Sliding Window** — highest ROI, appears everywhere
2. **Two Pointers** — classic, easy to recognize
3. **Binary Search** — both classic and on-answer
4. **BFS/DFS** — fundamental for trees and graphs
5. **Dynamic Programming** — hardest but most common in top companies
6. **Heap** — common for top-K problems
7. **Backtracking** — for "all combinations" problems
8. **Monotonic Stack** — less common but very recognizable
9. **Prefix Sum** — useful optimization
10. **Union Find** — niche but elegant

---

### The Mental Checklist

Before coding, run through this:

- [ ] Can I solve it with a hash map?
- [ ] Can I solve it with two pointers?
- [ ] Can I solve it with sliding window?
- [ ] Can I solve it with binary search?
- [ ] Can I solve it with BFS/DFS?
- [ ] Can I solve it with DP?
- [ ] Can I solve it with a heap?
- [ ] Can I solve it with a stack?
- [ ] Can I solve it greedily?
- [ ] Can I solve it with bit manipulation?

If none fit, it might be a combination or a problem requiring a novel approach.

---

### Final Advice

1. **Don't panic if you don't recognize the pattern immediately.** Start with brute force and optimize.

2. **Talk through your thought process.** Interviewers want to see how you think.

3. **Ask for hints if stuck.** It's better than silence.

4. **Practice pattern recognition, not memorization.** The goal is to solve unseen problems.

5. **Time yourself.** In an interview, you have 20-30 minutes per problem.

---

*Good luck! You've got this.*
