# 11 — Dynamic Programming

## The Pattern That Remembers

---

### What It Is

Dynamic Programming (DP) is a technique for solving problems by breaking them into overlapping subproblems and storing the results to avoid redundant computation.

**The key insight:** If a problem has **overlapping subproblems** (the same subproblem is solved multiple times) and **optimal substructure** (the optimal solution can be built from optimal solutions to subproblems), then DP can turn exponential brute force into polynomial time.

---

### When to Use It

**The problem has:**
- **Overlapping subproblems:** The same subproblem appears multiple times
- **Optimal substructure:** The optimal solution contains optimal solutions to subproblems
- **Choices:** At each step, you make a decision that affects future options
- **Counting:** "How many ways..." or "What's the minimum/maximum..."

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "minimum/maximum" | Minimum Path Sum |
| "longest/shortest" | Longest Common Subsequence |
| "number of ways" | Climbing Stairs |
| "can you reach" | Jump Game |
| "is it possible" | Subset Sum |
| "count" | Decode Ways |
| "optimal" | Best Time to Buy/Sell Stock |
| "subsequence" | Longest Increasing Subsequence |
| "substring" | Longest Palindromic Substring |
| "knapsack" | 0/1 Knapsack |
| "coin change" | Coin Change |

**The Decision:**

```
Does the problem ask for min/max/count?
    ↓
Yes
    ↓
Are there overlapping subproblems?
    ↓
Yes
    ↓
DYNAMIC PROGRAMMING
```

---

### The 5-Step Framework

Every DP problem can be solved by answering these 5 questions:

1. **State:** What information defines a subproblem?
2. **Transition:** How does a subproblem relate to smaller subproblems?
3. **Base Case:** What's the smallest subproblem (answer known directly)?
4. **Order:** In what order do we solve subproblems?
5. **Answer:** Which subproblem gives us the final answer?

---

### The Two Implementation Approaches

#### Top-Down (Memoization)

Start from the original problem, recurse down, cache results.

```python
def fib(n, memo={}):
    if n <= 1:
        return n
    if n in memo:
        return memo[n]
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```

#### Bottom-Up (Tabulation)

Start from the smallest subproblems, build up to the original.

```python
def fib(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

**When to use which:**
- Top-down: Easier to write, more intuitive, uses recursion stack
- Bottom-up: Faster (no recursion overhead), easier to optimize space

---

### Mental Model: The DP Table

Think of a table where each cell represents a subproblem:

```
For Fibonacci:
dp[i] = dp[i-1] + dp[i-2]

dp = [0, 1, 1, 2, 3, 5, 8, 13, ...]
      0  1  2  3  4  5  6  7   ...

For LCS (Longest Common Subsequence):
dp[i][j] = longest common subsequence of s1[0..i-1] and s2[0..j-1]

     ""  A  B  C  D
""  [ 0, 0, 0, 0, 0 ]
A   [ 0, 1, 1, 1, 1 ]
C   [ 0, 1, 1, 2, 2 ]
D   [ 0, 1, 1, 2, 3 ]
```

---

### The 80% Template: 1D DP

```python
def dp_1d(nums):
    n = len(nums)
    dp = [0] * n  # or appropriate size
    
    # Base case
    dp[0] = base_value
    
    # Fill the table
    for i in range(1, n):
        dp[i] = transition(dp[i-1], ...)  # or other subproblems
    
    return dp[n-1]  # or max(dp), etc.
```

### The 80% Template: 2D DP

```python
def dp_2d(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Base cases
    for i in range(m + 1):
        dp[i][0] = base_value
    for j in range(n + 1):
        dp[0][j] = base_value
    
    # Fill the table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            dp[i][j] = transition(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    
    return dp[m][n]
```

---

### DP Patterns Cheat Sheet

| Pattern | State | Transition | Example |
|---------|-------|------------|---------|
| Fibonacci | dp[i] | dp[i] = dp[i-1] + dp[i-2] | Climbing Stairs |
| Kadane's | dp[i] = max subarray ending at i | dp[i] = max(nums[i], dp[i-1]+nums[i]) | Max Subarray |
| LCS | dp[i][j] = LCS of first i,j chars | if match: dp[i-1][j-1]+1 else max(dp[i-1][j], dp[i][j-1]) | LCS |
| Knapsack | dp[i][w] = max value with first i items, capacity w | max(take, skip) | 0/1 Knapsack |
| LIS | dp[i] = LIS ending at i | dp[i] = max(dp[j]+1) for all j<i where nums[j]<nums[i] | LIS |
| Coin Change | dp[i] = min coins for amount i | dp[i] = min(dp[i-c]+1) for each coin c | Coin Change |

---

### Dry Run: Climbing Stairs

**Problem:** You can climb 1 or 2 steps. How many distinct ways to reach step n?

**Input:** `n = 4`

**Recognition:** "number of ways" + "choices at each step" → DP

**5-Step Framework:**
1. **State:** `dp[i]` = number of ways to reach step i
2. **Transition:** `dp[i] = dp[i-1] + dp[i-2]` (reach step i from i-1 or i-2)
3. **Base Case:** `dp[0] = 1` (one way to be at ground), `dp[1] = 1`
4. **Order:** Left to right (i from 2 to n)
5. **Answer:** `dp[n]`

```python
def climbStairs(n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    dp[2] = 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

**Step-by-step:**

```
n = 4

dp = [0, 1, 2, 0, 0]

i=3: dp[3] = dp[2] + dp[1] = 2 + 1 = 3
dp = [0, 1, 2, 3, 0]

i=4: dp[4] = dp[3] + dp[2] = 3 + 2 = 5
dp = [0, 1, 2, 3, 5]

Return 5

Ways to reach step 4:
1. 1+1+1+1
2. 1+1+2
3. 1+2+1
4. 2+1+1
5. 2+2
```

---

### Dry Run: Longest Common Subsequence

**Problem:** Find the length of the longest common subsequence of two strings.

**Input:** `text1 = "abcde"`, `text2 = "ace"`

**Recognition:** "longest common subsequence" → 2D DP

**5-Step Framework:**
1. **State:** `dp[i][j]` = LCS of `text1[0..i-1]` and `text2[0..j-1]`
2. **Transition:** If `text1[i-1] == text2[j-1]`: `dp[i][j] = dp[i-1][j-1] + 1`, else `max(dp[i-1][j], dp[i][j-1])`
3. **Base Case:** `dp[0][j] = dp[i][0] = 0`
4. **Order:** Row by row, left to right
5. **Answer:** `dp[m][n]`

```python
def longestCommonSubsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]
```

**Step-by-step:**

```
text1 = "abcde", text2 = "ace"

     ""  a  c  e
""  [ 0, 0, 0, 0 ]
a   [ 0, 1, 1, 1 ]
b   [ 0, 1, 1, 1 ]
c   [ 0, 1, 2, 2 ]
d   [ 0, 1, 2, 2 ]
e   [ 0, 1, 2, 3 ]

dp[5][3] = 3

LCS is "ace" with length 3.
```

**How to read the table:**
- `dp[i][j]` represents the LCS of the first `i` characters of text1 and first `j` characters of text2
- When characters match, we take the diagonal + 1
- When they don't, we take the max of the cell above or to the left

---

### Dry Run: 0/1 Knapsack

**Problem:** Given items with weights and values, find the maximum value that fits in a knapsack of capacity W.

**Input:** `weights = [2, 3, 4, 5]`, `values = [3, 4, 5, 6]`, `W = 8`

**Recognition:** "maximum value" + "capacity constraint" + "choose items" → Knapsack DP

**5-Step Framework:**
1. **State:** `dp[i][w]` = max value using first `i` items with capacity `w`
2. **Transition:** `dp[i][w] = max(dp[i-1][w], dp[i-1][w-w_i] + v_i)`
3. **Base Case:** `dp[0][w] = 0` (no items = no value)
4. **Order:** Row by row (each item), left to right (each capacity)
5. **Answer:** `dp[n][W]`

```python
def knapsack(weights, values, W):
    n = len(weights)
    dp = [[0] * (W + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(W + 1):
            # Don't take item i
            dp[i][w] = dp[i-1][w]
            # Take item i (if it fits)
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1])
    
    return dp[n][W]
```

**Step-by-step (simplified):**

```
weights = [2, 3, 4, 5], values = [3, 4, 5, 6], W = 8

dp table (rows=items, cols=capacity 0-8):

Item 1 (w=2, v=3):
  w=0..1: can't take → 0
  w=2..8: take it → 3
  [0, 0, 3, 3, 3, 3, 3, 3, 3]

Item 2 (w=3, v=4):
  w=0..2: skip → [0, 0, 3]
  w=3: max(skip=3, take=4) = 4
  w=4: max(skip=3, take=4) = 4
  w=5: max(skip=3, take=3+4=7) = 7
  w=6..8: max(skip, take) = 7
  [0, 0, 3, 4, 4, 7, 7, 7, 7]

Item 3 (w=4, v=5):
  w=0..3: skip
  w=4: max(skip=4, take=5) = 5
  w=5: max(skip=4, take=3+5=8) = 8
  w=6: max(skip=7, take=4+5=9) = 9
  w=7: max(skip=7, take=3+5=8) = 8? No, 7. Wait...
  Actually: w=7: max(skip=7, take=dp[2][3]+5=4+5=9) = 9
  w=8: max(skip=7, take=dp[2][4]+5=4+5=9) = 9
  [0, 0, 3, 4, 5, 8, 9, 9, 9]

Item 4 (w=5, v=6):
  w=0..4: skip
  w=5: max(skip=8, take=3+6=9) = 9
  w=6: max(skip=9, take=4+6=10) = 10? Let me recalculate...
  Actually this gets complex. Final answer: 10

Return 10 (take items 2 and 4: w=3+5=8, v=4+6=10)
```

---

### Dry Run: Coin Change

**Problem:** Find the minimum number of coins to make a given amount.

**Input:** `coins = [1, 2, 5]`, `amount = 11`

**Recognition:** "minimum number of coins" → DP (unbounded knapsack variant)

**5-Step Framework:**
1. **State:** `dp[i]` = min coins to make amount `i`
2. **Transition:** `dp[i] = min(dp[i-c] + 1)` for each coin `c`
3. **Base Case:** `dp[0] = 0` (0 coins for amount 0)
4. **Order:** Left to right (amount from 1 to target)
5. **Answer:** `dp[amount]`

```python
def coinChange(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i and dp[i - coin] != float('inf'):
                dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1
```

**Step-by-step:**

```
coins = [1, 2, 5], amount = 11

dp[0] = 0

dp[1] = min(dp[1-1]+1) = dp[0]+1 = 1
dp[2] = min(dp[2-1]+1, dp[2-2]+1) = min(2, 1) = 1
dp[3] = min(dp[3-1]+1, dp[3-2]+1) = min(2, 2) = 2
dp[4] = min(dp[4-1]+1, dp[4-2]+1) = min(3, 2) = 2
dp[5] = min(dp[5-1]+1, dp[5-2]+1, dp[5-5]+1) = min(3, 3, 1) = 1
dp[6] = min(dp[6-1]+1, dp[6-2]+1, dp[6-5]+1) = min(2, 2, 2) = 2
dp[7] = min(dp[7-1]+1, dp[7-2]+1, dp[7-5]+1) = min(3, 2, 2) = 2
dp[8] = min(dp[8-1]+1, dp[8-2]+1, dp[8-5]+1) = min(3, 3, 2) = 2
dp[9] = min(dp[9-1]+1, dp[9-2]+1, dp[9-5]+1) = min(3, 3, 2) = 2
dp[10] = min(dp[10-1]+1, dp[10-2]+1, dp[10-5]+1) = min(3, 3, 2) = 2
dp[11] = min(dp[11-1]+1, dp[11-2]+1, dp[11-5]+1) = min(3, 3, 3) = 3

Return 3 (5+5+1)
```

---

### Dry Run: Longest Increasing Subsequence

**Problem:** Find the length of the longest strictly increasing subsequence.

**Input:** `nums = [10, 9, 2, 5, 3, 7, 101, 18]`

**Recognition:** "longest" + "increasing" + "subsequence" → DP

**5-Step Framework:**
1. **State:** `dp[i]` = length of LIS ending at index `i`
2. **Transition:** `dp[i] = max(dp[j] + 1)` for all `j < i` where `nums[j] < nums[i]`
3. **Base Case:** `dp[i] = 1` (every element is a subsequence of length 1)
4. **Order:** Left to right
5. **Answer:** `max(dp)`

```python
def lengthOfLIS(nums):
    n = len(nums)
    dp = [1] * n
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)
```

**Step-by-step:**

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

dp = [1, 1, 1, 1, 1, 1, 1, 1]

i=1 (9): j=0: 10 > 9, skip. dp[1] = 1
i=2 (2): j=0: 10 > 2, skip. j=1: 9 > 2, skip. dp[2] = 1
i=3 (5): j=0: 10 > 5, skip. j=1: 9 > 5, skip. j=2: 2 < 5, dp[3] = max(1, 1+1) = 2
i=4 (3): j=2: 2 < 3, dp[4] = max(1, 1+1) = 2
i=5 (7): j=2: 2 < 7, dp[5] = max(1, 1+1) = 2
         j=3: 5 < 7, dp[5] = max(2, 2+1) = 3
         j=4: 3 < 7, dp[5] = max(3, 2+1) = 3
i=6 (101): ... dp[6] = 4 (2, 5, 7, 101 or 2, 3, 7, 101)
i=7 (18): ... dp[7] = 4 (2, 5, 7, 18 or 2, 3, 7, 18)

dp = [1, 1, 1, 2, 2, 3, 4, 4]

Return max(dp) = 4
```

**Optimized O(n log n) solution using binary search:**

```python
import bisect

def lengthOfLIS(nums):
    tails = []  # tails[i] = smallest tail of all increasing subsequences of length i+1
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)
```

---

### Space Optimization

Many DP problems can be optimized from O(n²) or O(n×m) space to O(n) or O(1).

#### Example: Fibonacci

```python
# O(n) space
dp = [0] * (n + 1)
dp[1] = 1
for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]

# O(1) space
prev, curr = 0, 1
for i in range(2, n + 1):
    prev, curr = curr, prev + curr
```

#### Example: LCS

```python
# O(m×n) space
dp = [[0] * (n + 1) for _ in range(m + 1)]

# O(n) space (only need previous row)
prev = [0] * (n + 1)
curr = [0] * (n + 1)
for i in range(1, m + 1):
    for j in range(1, n + 1):
        if s1[i-1] == s2[j-1]:
            curr[j] = prev[j-1] + 1
        else:
            curr[j] = max(prev[j], curr[j-1])
    prev, curr = curr, [0] * (n + 1)
```

---

### Common DP Patterns

#### Pattern 1: Linear DP (1D)

State depends on previous elements.

```
dp[i] depends on dp[i-1], dp[i-2], ...
```

**Examples:** Fibonacci, Climbing Stairs, House Robber, Maximum Subarray

#### Pattern 2: Grid DP (2D)

State depends on cell above and/or to the left.

```
dp[i][j] depends on dp[i-1][j], dp[i][j-1], dp[i-1][j-1]
```

**Examples:** Unique Paths, Minimum Path Sum, LCS, Edit Distance

#### Pattern 3: Knapsack DP

Choose items with capacity constraint.

```
dp[i][w] = max(dp[i-1][w], dp[i-1][w-w_i] + v_i)
```

**Examples:** 0/1 Knapsack, Subset Sum, Partition Equal Subset Sum

#### Pattern 4: Interval DP

Solve for all subarrays/substrings.

```
dp[i][j] depends on dp[i][k] and dp[k+1][j] for all k
```

**Examples:** Matrix Chain Multiplication, Palindrome Partitioning, Burst Balloons

#### Pattern 5: String DP

Compare or transform strings.

```
dp[i][j] = result for first i chars of s1 and first j chars of s2
```

**Examples:** LCS, Edit Distance, Longest Palindromic Subsequence

---

### Code Templates (4 Languages)

#### Python

```python
# Top-Down (Memoization)
from functools import lru_cache

def dp_topdown(nums):
    @lru_cache(maxsize=None)
    def solve(i):
        if i >= len(nums):
            return base_case
        return transition(solve(i+1), ...)
    
    return solve(0)

# Bottom-Up (Tabulation)
def dp_bottomup(nums):
    n = len(nums)
    dp = [0] * n
    dp[0] = base_case
    
    for i in range(1, n):
        dp[i] = transition(dp[i-1], ...)
    
    return dp[n-1]
```

#### Java

```java
// Top-Down
int solve(int i, int[] memo) {
    if (i >= n) return baseCase;
    if (memo[i] != -1) return memo[i];
    memo[i] = transition(solve(i+1, memo), ...);
    return memo[i];
}

// Bottom-Up
int solve(int[] nums) {
    int[] dp = new int[n];
    dp[0] = baseCase;
    for (int i = 1; i < n; i++) {
        dp[i] = transition(dp[i-1], ...);
    }
    return dp[n-1];
}
```

#### C++

```cpp
// Top-Down
int solve(int i, vector<int>& memo) {
    if (i >= n) return baseCase;
    if (memo[i] != -1) return memo[i];
    memo[i] = transition(solve(i+1, memo), ...);
    return memo[i];
}

// Bottom-Up
int solve(vector<int>& nums) {
    vector<int> dp(n, 0);
    dp[0] = baseCase;
    for (int i = 1; i < n; i++) {
        dp[i] = transition(dp[i-1], ...);
    }
    return dp[n-1];
}
```

#### JavaScript

```javascript
// Top-Down
function solve(i, memo = {}) {
    if (i >= n) return baseCase;
    if (memo[i] !== undefined) return memo[i];
    memo[i] = transition(solve(i+1, memo), ...);
    return memo[i];
}

// Bottom-Up
function solve(nums) {
    const dp = new Array(n).fill(0);
    dp[0] = baseCase;
    for (let i = 1; i < n; i++) {
        dp[i] = transition(dp[i-1], ...);
    }
    return dp[n-1];
}
```

---

### Common Mistakes

#### Mistake 1: Wrong Base Case

```python
# WRONG: dp[0] should represent 0 items, not 1
dp = [0] * n
dp[0] = nums[0]  # This is item 1, not 0 items

# RIGHT:
dp = [0] * (n + 1)
dp[0] = 0  # 0 items = 0 value
```

#### Mistake 2: Off-by-One in Indices

```python
# WRONG: mixing 0-indexed and 1-indexed
for i in range(n):
    dp[i] = dp[i-1] + nums[i]  # dp[-1] is wrong when i=0

# RIGHT: be consistent
dp = [0] * (n + 1)
for i in range(1, n + 1):
    dp[i] = dp[i-1] + nums[i-1]
```

#### Mistake 3: Not Initializing Infinity for Min Problems

```python
# WRONG: initializes to 0, which might be a valid answer
dp = [0] * (amount + 1)

# RIGHT: initialize to infinity for minimization
dp = [float('inf')] * (amount + 1)
dp[0] = 0
```

#### Mistake 4: Wrong Order for Bottom-Up

```python
# WRONG: for unbounded knapsack, processing items in wrong order
for w in range(W + 1):
    for i in range(n):
        # This doesn't allow using item i multiple times correctly

# RIGHT: for 0/1 knapsack, process items outer loop
for i in range(n):
    for w in range(W, weights[i]-1, -1):  # Reverse for 0/1!
        dp[w] = max(dp[w], dp[w-weights[i]] + values[i])
```

---

### Complexity Analysis

| Problem | Time | Space | Space (optimized) |
|---------|------|-------|-------------------|
| Fibonacci | O(n) | O(n) | O(1) |
| Climbing Stairs | O(n) | O(n) | O(1) |
| LCS | O(m×n) | O(m×n) | O(min(m,n)) |
| 0/1 Knapsack | O(n×W) | O(n×W) | O(W) |
| LIS | O(n²) or O(n log n) | O(n) | O(n) |
| Coin Change | O(amount × coins) | O(amount) | O(amount) |

---

### Practice Problems

#### Easy

| # | Problem | Pattern |
|---|---------|---------|
| 1 | [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) | Linear DP |
| 2 | [198. House Robber](https://leetcode.com/problems/house-robber/) | Linear DP |
| 3 | [121. Best Time to Buy/Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Linear DP |
| 4 | [509. Fibonacci Number](https://leetcode.com/problems/fibonacci-number/) | Linear DP |
| 5 | [746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/) | Linear DP |

#### Medium

| # | Problem | Pattern |
|---|---------|---------|
| 6 | [322. Coin Change](https://leetcode.com/problems/coin-change/) | Unbounded Knapsack |
| 7 | [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | Linear DP |
| 8 | [1143. Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) | String DP |
| 9 | [62. Unique Paths](https://leetcode.com/problems/unique-paths/) | Grid DP |
| 10 | [64. Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/) | Grid DP |
| 11 | [416. Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | 0/1 Knapsack |
| 12 | [139. Word Break](https://leetcode.com/problems/word-break/) | Linear DP |
| 13 | [152. Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Linear DP |
| 14 | [213. House Robber II](https://leetcode.com/problems/house-robber-ii/) | Circular DP |
| 15 | [5. Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | Interval DP |

#### Hard

| # | Problem | Pattern |
|---|---------|---------|
| 16 | [72. Edit Distance](https://leetcode.com/problems/edit-distance/) | String DP |
| 17 | [312. Burst Balloons](https://leetcode.com/problems/burst-balloons/) | Interval DP |
| 18 | [10. Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) | String DP |
| 19 | [44. Wildcard Matching](https://leetcode.com/problems/wildcard-matching/) | String DP |
| 20 | [115. Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/) | String DP |
| 21 | [188. Best Time to Buy/Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) | State Machine DP |

---

### Interview Tips

1. **Start with brute force.** "The brute force approach is exponential because we'd explore all possibilities. I can optimize with DP."

2. **Define the state clearly.** "dp[i] represents [what exactly]."

3. **Write the transition.** "To compute dp[i], I look at [which subproblems]."

4. **State the base case.** "The base case is dp[0] = [value]."

5. **Mention space optimization if applicable.** "I can optimize from O(n²) to O(n) by only keeping the previous row."

6. **Edge cases:**
   - Empty input
   - Single element
   - All same elements
   - Target is 0
   - No valid solution exists

---

*Next: [12 — Greedy](12-Greedy.md)*
