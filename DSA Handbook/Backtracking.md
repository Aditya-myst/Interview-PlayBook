# 10 — Backtracking

## The Pattern That Explores All Possibilities

---

### What It Is

Backtracking is a systematic way to explore all possible solutions to a problem by building a solution incrementally and "backtracking" (undoing the last choice) when you hit a dead end.

**The key insight:** Backtracking is DFS on a decision tree. At each step, you make a choice, recurse, then undo the choice to explore other branches.

---

### When to Use It

**The problem asks for:**
- All combinations
- All permutations
- All subsets
- All valid arrangements (N-Queens, Sudoku)
- Generating all possibilities

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "all combinations" | Combination Sum |
| "all permutations" | Permutations |
| "all subsets" | Subsets |
| "generate all" | Generate Parentheses |
| "find all solutions" | N-Queens |
| "valid arrangements" | Sudoku Solver |
| "letter combinations" | Letter Combinations of Phone |
| "word search" | Word Search |
| "partition" | Palindrome Partitioning |

**The Decision:**

```
Does the problem ask for ALL solutions (or all possible arrangements)?
    ↓
Yes
    ↓
BACKTRACKING
```

---

### Mental Model: The Decision Tree

Think of backtracking as exploring a tree where:
- Each node represents a partial solution
- Each branch represents a choice
- Leaves represent complete solutions

```
Subsets of [1, 2, 3]:

                    []
           /        |        \
         [1]       [2]       [3]
        /   \       |
     [1,2] [1,3] [2,3]
       |
   [1,2,3]
```

At each node:
1. **Choose** — Add an element to the current solution
2. **Explore** — Recurse to build the rest
3. **Unchoose** — Remove the element (backtrack) to try other branches

---

### The 80% Template

```python
def backtrack(candidates, current, result):
    # Base case: solution is complete
    if is_solution(current):
        result.append(current.copy())  # or current[:]
        return
    
    for candidate in candidates:
        if is_valid(candidate, current):
            # Choose
            current.append(candidate)
            
            # Explore
            backtrack(remaining_candidates, current, result)
            
            # Unchoose (backtrack)
            current.pop()
```

**The three steps:**
1. **Choose:** Add the candidate to the current solution
2. **Explore:** Recurse with the updated solution
3. **Unchoose:** Remove the candidate to try other options

---

### The Three Classic Variations

#### Variation 1: Subsets (Include or Exclude)

Each element can be included or excluded.

```python
def subsets(nums):
    result = []
    
    def backtrack(index, current):
        result.append(current[:])  # Every prefix is a valid subset
        
        for i in range(index, len(nums)):
            current.append(nums[i])      # Choose
            backtrack(i + 1, current)    # Explore
            current.pop()                # Unchoose
    
    backtrack(0, [])
    return result
```

**Decision tree for [1, 2, 3]:**

```
[]
├── [1]
│   ├── [1,2]
│   │   └── [1,2,3]
│   └── [1,3]
└── [2]
    └── [2,3]
└── [3]
```

#### Variation 2: Permutations (All Orderings)

Each element can be placed at each position.

```python
def permutations(nums):
    result = []
    
    def backtrack(current, remaining):
        if not remaining:
            result.append(current[:])
            return
        
        for i in range(len(remaining)):
            current.append(remaining[i])                    # Choose
            backtrack(current, remaining[:i] + remaining[i+1:])  # Explore
            current.pop()                                   # Unchoose
    
    backtrack([], nums)
    return result
```

**Decision tree for [1, 2, 3]:**

```
[]
├── [1]
│   ├── [1,2]
│   │   └── [1,2,3]
│   └── [1,3]
│       └── [1,3,2]
├── [2]
│   ├── [2,1]
│   │   └── [2,1,3]
│   └── [2,3]
│       └── [2,3,1]
└── [3]
    ├── [3,1]
    │   └── [3,1,2]
    └── [3,2]
        └── [3,2,1]
```

#### Variation 3: Combinations (Choose K)

Choose exactly K elements from N.

```python
def combinations(n, k):
    result = []
    
    def backtrack(start, current):
        if len(current) == k:
            result.append(current[:])
            return
        
        for i in range(start, n + 1):
            current.append(i)          # Choose
            backtrack(i + 1, current)  # Explore (start from i+1 to avoid duplicates)
            current.pop()              # Unchoose
    
    backtrack(1, [])
    return result
```

**Key difference from permutations:** We pass `start` to avoid revisiting earlier elements (no duplicates, no reordering).

---

### Dry Run: Combination Sum

**Problem:** Find all unique combinations of candidates that sum to target. Each candidate can be used unlimited times.

**Input:** `candidates = [2, 3, 6, 7]`, `target = 7`

**Recognition:** "all combinations" + "sum to target" → Backtracking

```python
def combinationSum(candidates, target):
    result = []
    
    def backtrack(start, current, remaining):
        if remaining == 0:
            result.append(current[:])
            return
        if remaining < 0:
            return
        
        for i in range(start, len(candidates)):
            current.append(candidates[i])
            backtrack(i, current, remaining - candidates[i])  # i, not i+1 (reuse allowed)
            current.pop()
    
    backtrack(0, [], target)
    return result
```

**Step-by-step (partial):**

```
candidates = [2, 3, 6, 7], target = 7

backtrack(0, [], 7)
├── Choose 2: backtrack(0, [2], 5)
│   ├── Choose 2: backtrack(0, [2,2], 3)
│   │   ├── Choose 2: backtrack(0, [2,2,2], 1)
│   │   │   ├── Choose 2: backtrack(0, [2,2,2,2], -1) → return (negative)
│   │   │   ├── Choose 3: backtrack(1, [2,2,2,3], -2) → return (negative)
│   │   │   └── ... (all negative, no solution)
│   │   │   Pop 2
│   │   ├── Choose 3: backtrack(1, [2,2,3], 0) → FOUND [2,2,3] ✓
│   │   │   Pop 3
│   │   ├── Choose 6: backtrack(2, [2,2,6], -3) → return
│   │   │   Pop 6
│   │   └── Choose 7: backtrack(3, [2,2,7], -4) → return
│   │       Pop 7
│   │   Pop 2
│   ├── Choose 3: backtrack(1, [2,3], 2)
│   │   ├── Choose 3: backtrack(1, [2,3,3], -1) → return
│   │   │   Pop 3
│   │   ├── Choose 6: backtrack(2, [2,3,6], -4) → return
│   │   │   Pop 6
│   │   └── Choose 7: backtrack(3, [2,3,7], -5) → return
│   │       Pop 7
│   │   Pop 3
│   ├── Choose 6: backtrack(2, [2,6], -1) → return
│   │   Pop 6
│   └── Choose 7: backtrack(3, [2,7], -2) → return
│       Pop 7
│   Pop 2
├── Choose 3: backtrack(1, [3], 4)
│   ├── Choose 3: backtrack(1, [3,3], 1)
│   │   ├── ... (all negative)
│   │   Pop 3
│   ├── Choose 6: backtrack(2, [3,6], -2) → return
│   │   Pop 6
│   └── Choose 7: backtrack(3, [3,7], -3) → return
│       Pop 7
│   Pop 3
├── Choose 6: backtrack(2, [6], 1)
│   └── ... (no solution)
│   Pop 6
└── Choose 7: backtrack(3, [7], 0) → FOUND [7] ✓
    Pop 7

Result: [[2,2,3], [7]]
```

---

### Dry Run: N-Queens

**Problem:** Place N queens on an N×N chessboard so no two queens attack each other.

**Input:** `n = 4`

**Recognition:** "all valid arrangements" → Backtracking

```python
def solveNQueens(n):
    result = []
    
    def backtrack(row, cols, diag1, diag2, board):
        if row == n:
            result.append(["".join(r) for r in board])
            return
        
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue  # Can't place queen here
            
            # Choose
            board[row][col] = 'Q'
            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)
            
            # Explore
            backtrack(row + 1, cols, diag1, diag2, board)
            
            # Unchoose
            board[row][col] = '.'
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)
    
    board = [['.' for _ in range(n)] for _ in range(n)]
    backtrack(0, set(), set(), set(), board)
    return result
```

**Visual for 4-Queens:**

```
Solution 1:       Solution 2:
. Q . .           . . Q .
. . . Q           Q . . .
Q . . .           . . . Q
. . Q .           . Q . .
```

**Key constraints:**
- No two queens in the same column (`cols` set)
- No two queens on the same diagonal (`row - col` constant)
- No two queens on the same anti-diagonal (`row + col` constant)

---

### Dry Run: Generate Parentheses

**Problem:** Generate all valid combinations of n pairs of parentheses.

**Input:** `n = 3`

**Recognition:** "generate all" + "valid" → Backtracking

```python
def generateParenthesis(n):
    result = []
    
    def backtrack(current, open_count, close_count):
        if len(current) == 2 * n:
            result.append(current)
            return
        
        if open_count < n:
            backtrack(current + '(', open_count + 1, close_count)
        
        if close_count < open_count:
            backtrack(current + ')', open_count, close_count + 1)
    
    backtrack('', 0, 0)
    return result
```

**Step-by-step:**

```
n = 3

backtrack('', 0, 0)
├── Add '(': backtrack('(', 1, 0)
│   ├── Add '(': backtrack('((', 2, 0)
│   │   ├── Add '(': backtrack('(((', 3, 0)
│   │   │   ├── Can't add '(' (open=3=n)
│   │   │   ├── Add ')': backtrack('((()', 3, 1)
│   │   │   │   ├── Add ')': backtrack('((())', 3, 2)
│   │   │   │   │   ├── Add ')': backtrack('((()))', 3, 3) → FOUND ✓
│   │   │   │   │   Pop ')'
│   │   │   │   Pop ')'
│   │   │   └── Pop ')'
│   │   │   Pop '('
│   │   ├── Add ')': backtrack('(()', 2, 1)
│   │   │   ├── Add '(': backtrack('(()(', 3, 1)
│   │   │   │   ├── Add ')': backtrack('(()()', 3, 2)
│   │   │   │   │   ├── Add ')': backtrack('(()())', 3, 3) → FOUND ✓
│   │   │   │   │   Pop ')'
│   │   │   │   Pop ')'
│   │   │   ├── Add ')': backtrack('(())', 2, 2)
│   │   │   │   ├── Add '(': backtrack('(())(', 3, 2)
│   │   │   │   │   ├── Add ')': backtrack('(())()', 3, 3) → FOUND ✓
│   │   │   │   │   Pop ')'
│   │   │   │   ├── Can't add ')' (close=open=2)
│   │   │   │   Pop '('
│   │   │   Pop ')'
│   │   Pop '('
│   ├── Add ')': backtrack('()', 1, 1)
│   │   ├── ... (continues similarly)
│   │   Pop ')'
│   Pop '('
... (continues)

Result: ["((()))", "(()())", "(())()", "()(())", "()()()"]
```

---

### The Pruning Optimization

Backtracking can be slow without pruning. **Pruning** means skipping branches that can't lead to valid solutions.

```python
# Without pruning: explores all branches
for i in range(start, len(nums)):
    current.append(nums[i])
    backtrack(i + 1, current, remaining - nums[i])
    current.pop()

# With pruning: skip branches where remaining < 0
for i in range(start, len(nums)):
    if nums[i] > remaining:
        continue  # Prune: this branch can't lead to a solution
    current.append(nums[i])
    backtrack(i + 1, current, remaining - nums[i])
    current.pop()
```

**Other common pruning techniques:**
- **Sort candidates** and break early when sum exceeds target
- **Skip duplicates** by checking `if i > start and nums[i] == nums[i-1]: continue`
- **Use constraints** to eliminate impossible placements early

---

### Handling Duplicates

When the input has duplicates, you need to avoid duplicate results.

```python
def subsetsWithDup(nums):
    nums.sort()  # Sort first!
    result = []
    
    def backtrack(start, current):
        result.append(current[:])
        
        for i in range(start, len(nums)):
            # Skip duplicates at the same level
            if i > start and nums[i] == nums[i - 1]:
                continue
            
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()
    
    backtrack(0, [])
    return result
```

**Why sort first?** Sorting ensures duplicates are adjacent, making them easy to skip.

---

### Code Templates (4 Languages)

#### Python

```python
def backtrack(candidates, target, current, result, start):
    if target == 0:
        result.append(current[:])
        return
    
    for i in range(start, len(candidates)):
        if candidates[i] > target:
            break  # Pruning (sorted array)
        
        current.append(candidates[i])
        backtrack(candidates, target - candidates[i], current, result, i)
        current.pop()
```

#### Java

```java
void backtrack(int[] candidates, int target, List<Integer> current, 
               List<List<Integer>> result, int start) {
    if (target == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > target) break;
        current.add(candidates[i]);
        backtrack(candidates, target - candidates[i], current, result, i);
        current.remove(current.size() - 1);
    }
}
```

#### C++

```cpp
void backtrack(vector<int>& candidates, int target, vector<int>& current,
               vector<vector<int>>& result, int start) {
    if (target == 0) {
        result.push_back(current);
        return;
    }
    
    for (int i = start; i < candidates.size(); i++) {
        if (candidates[i] > target) break;
        current.push_back(candidates[i]);
        backtrack(candidates, target - candidates[i], current, result, i);
        current.pop_back();
    }
}
```

#### JavaScript

```javascript
function backtrack(candidates, target, current, result, start) {
    if (target === 0) {
        result.push([...current]);
        return;
    }
    
    for (let i = start; i < candidates.length; i++) {
        if (candidates[i] > target) break;
        current.push(candidates[i]);
        backtrack(candidates, target - candidates[i], current, result, i);
        current.pop();
    }
}
```

---

### Common Mistakes

#### Mistake 1: Not Making a Copy of the Current Solution

```python
# WRONG: appends a reference that gets modified
result.append(current)

# RIGHT: append a copy
result.append(current[:])
```

#### Mistake 2: Forgetting to Backtrack

```python
# WRONG: doesn't undo the choice
current.append(nums[i])
backtrack(i + 1, current)
# Missing: current.pop()

# RIGHT:
current.append(nums[i])
backtrack(i + 1, current)
current.pop()  # Undo the choice
```

#### Mistake 3: Wrong Start Index

```python
# For combinations (no reuse): start from i+1
backtrack(i + 1, current, remaining - nums[i])

# For combinations with reuse: start from i
backtrack(i, current, remaining - nums[i])

# For permutations: rebuild remaining list
backtrack(current + [nums[i]], nums[:i] + nums[i+1:])
```

#### Mistake 4: Not Handling Duplicates

```python
# WRONG: produces duplicate results
for i in range(start, len(nums)):
    current.append(nums[i])
    backtrack(i + 1, current)
    current.pop()

# RIGHT: skip duplicates at the same level
nums.sort()
for i in range(start, len(nums)):
    if i > start and nums[i] == nums[i - 1]:
        continue
    current.append(nums[i])
    backtrack(i + 1, current)
    current.pop()
```

---

### Complexity Analysis

| Problem Type | Time | Space |
|--------------|------|-------|
| Subsets | O(2^n) | O(n) |
| Permutations | O(n! × n) | O(n) |
| Combinations C(n,k) | O(C(n,k) × k) | O(k) |
| N-Queens | O(n!) | O(n²) |

**Why exponential?** Backtracking explores a decision tree. The number of leaf nodes grows exponentially.

---

### Practice Problems

#### Easy-Medium

| # | Problem | Type |
|---|---------|------|
| 1 | [78. Subsets](https://leetcode.com/problems/subsets/) | Subsets |
| 2 | [90. Subsets II](https://leetcode.com/problems/subsets-ii/) | Subsets + Duplicates |
| 3 | [46. Permutations](https://leetcode.com/problems/permutations/) | Permutations |
| 4 | [47. Permutations II](https://leetcode.com/problems/permutations-ii/) | Permutations + Duplicates |
| 5 | [77. Combinations](https://leetcode.com/problems/combinations/) | Combinations |
| 6 | [39. Combination Sum](https://leetcode.com/problems/combination-sum/) | Combinations + Target |
| 7 | [40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) | Combinations + Duplicates |
| 8 | [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) | Generate Valid |
| 9 | [17. Letter Combinations of Phone](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | Cartesian Product |
| 10 | [131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | Partition |

#### Hard

| # | Problem | Type |
|---|---------|------|
| 11 | [51. N-Queens](https://leetcode.com/problems/n-queens/) | Constraint Satisfaction |
| 12 | [37. Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | Constraint Satisfaction |
| 13 | [79. Word Search](https://leetcode.com/problems/word-search/) | Grid Backtracking |
| 14 | [93. Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/) | Partition |
| 15 | [212. Word Search II](https://leetcode.com/problems/word-search-ii/) | Trie + Backtracking |

---

### Interview Tips

1. **State the decision tree.** "At each step, I'm choosing [X] from the remaining candidates."

2. **Explain the three steps.** "I choose a candidate, recurse, then unchoose to explore other branches."

3. **Mention pruning.** "I prune branches where [condition] to avoid unnecessary exploration."

4. **Handle duplicates.** "If the input has duplicates, I sort first and skip duplicate values at the same level."

5. **Complexity.** "The time complexity is O(2^n) for subsets because there are 2^n possible subsets."

6. **Edge cases:**
   - Empty input
   - Single element
   - All duplicates
   - Target = 0

---

*Next: [11 — Dynamic Programming](11-Dynamic-Programming.md)*
