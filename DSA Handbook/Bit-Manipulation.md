# 16 — Bit Manipulation

## The Pattern That Operates on Bits

---

### What It Is

Bit manipulation uses bitwise operations (AND, OR, XOR, NOT, shifts) to solve problems efficiently. Many problems have elegant bit-based solutions that are faster and use less space.

**The key insight:** Computers store numbers in binary. By manipulating individual bits, you can solve certain problems in O(1) space and O(n) or even O(1) time.

---

### When to Use It

**The problem involves:**
- Finding a unique element among duplicates
- Checking if a number is a power of 2
- Counting set bits
- Subsets generation
- Swapping without temp variable
- Any problem where elements appear in pairs except one

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "single number" | Single Number |
| "unique" | Single Number II |
| "power of two" | Power of Two |
| "count bits" | Counting Bits |
| "missing number" | Missing Number |
| "swap" | Swap without temp |
| "subsets" | Subsets (bitmask) |
| "XOR" | Various XOR problems |

---

### The Essential Bit Operations

| Operation | Symbol | Example (a=5, b=3) | Result |
|-----------|--------|---------------------|--------|
| AND | `&` | `5 & 3` = `101 & 011` | `001` = 1 |
| OR | `\|` | `5 \| 3` = `101 \| 011` | `111` = 7 |
| XOR | `^` | `5 ^ 3` = `101 ^ 011` | `110` = 6 |
| NOT | `~` | `~5` = `~00000101` | `11111010` = -6 |
| Left Shift | `<<` | `5 << 1` = `1010` | 10 |
| Right Shift | `>>` | `5 >> 1` = `10` | 2 |

---

### Key Properties

#### XOR Properties

```
a ^ a = 0          (anything XOR itself = 0)
a ^ 0 = a          (anything XOR 0 = itself)
a ^ b ^ a = b      (XOR is commutative and associative)
```

**Application:** Find the single number in an array where every other number appears twice.

```python
def singleNumber(nums):
    result = 0
    for num in nums:
        result ^= num
    return result
```

#### AND Properties

```
a & (a-1) clears the lowest set bit of a

Example: a = 12 = 1100
a-1 = 11 = 1011
a & (a-1) = 1100 & 1011 = 1000 = 8
```

**Application:** Count set bits or check power of 2.

```python
def isPowerOfTwo(n):
    return n > 0 and (n & (n - 1)) == 0

def countBits(n):
    count = 0
    while n:
        n &= (n - 1)  # Clear lowest set bit
        count += 1
    return count
```

---

### The 80% Templates

#### Find Single Number (XOR)

```python
def singleNumber(nums):
    result = 0
    for num in nums:
        result ^= num
    return result
```

#### Count Set Bits

```python
def countBits(n):
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count
```

#### Check Power of 2

```python
def isPowerOfTwo(n):
    return n > 0 and (n & (n - 1)) == 0
```

#### Generate Subsets Using Bitmask

```python
def subsets(nums):
    n = len(nums)
    result = []
    
    for mask in range(1 << n):  # 0 to 2^n - 1
        subset = []
        for i in range(n):
            if mask & (1 << i):  # Check if bit i is set
                subset.append(nums[i])
        result.append(subset)
    
    return result
```

---

### Dry Run: Single Number

**Problem:** Given a non-empty array where every element appears twice except one, find the single element.

**Input:** `nums = [4, 1, 2, 1, 2]`

**Recognition:** "single number" + "appears twice" → XOR

```python
def singleNumber(nums):
    result = 0
    for num in nums:
        result ^= num
    return result
```

**Step-by-step:**

```
nums = [4, 1, 2, 1, 2]

result = 0

num=4: result = 0 ^ 4 = 4
num=1: result = 4 ^ 1 = 5
num=2: result = 5 ^ 2 = 7
num=1: result = 7 ^ 1 = 6
num=2: result = 6 ^ 2 = 4

Return 4

Why it works:
0 ^ 4 ^ 1 ^ 2 ^ 1 ^ 2
= 4 ^ (1 ^ 1) ^ (2 ^ 2)
= 4 ^ 0 ^ 0
= 4
```

---

### Dry Run: Missing Number

**Problem:** Given an array containing n distinct numbers from 0 to n, find the missing number.

**Input:** `nums = [3, 0, 1]`

**Recognition:** "missing number" + "0 to n" → XOR

```python
def missingNumber(nums):
    n = len(nums)
    result = n  # Start with n (since array is 0 to n)
    for i in range(n):
        result ^= i ^ nums[i]
    return result
```

**Step-by-step:**

```
nums = [3, 0, 1], n = 3

result = 3

i=0: result = 3 ^ 0 ^ 3 = 0
i=1: result = 0 ^ 1 ^ 0 = 1
i=2: result = 1 ^ 2 ^ 1 = 2

Return 2

Why: XOR of [0,1,2,3] ^ XOR of [3,0,1] = 2 (the missing number)
```

---

### Dry Run: Counting Bits

**Problem:** Given an integer n, return an array where ans[i] is the number of 1's in binary representation of i.

**Input:** `n = 5`

**Recognition:** "count bits" → Bit manipulation

```python
def countBits(n):
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        dp[i] = dp[i >> 1] + (i & 1)
    return dp
```

**Step-by-step:**

```
n = 5

dp = [0, 0, 0, 0, 0, 0]

i=1: dp[1] = dp[0] + 1 = 1    (1 = 1)
i=2: dp[2] = dp[1] + 0 = 1    (10 = 10)
i=3: dp[3] = dp[1] + 1 = 2    (11 = 11)
i=4: dp[4] = dp[2] + 0 = 1    (100 = 100)
i=5: dp[5] = dp[2] + 1 = 2    (101 = 101)

dp = [0, 1, 1, 2, 1, 2]

Return [0, 1, 1, 2, 1, 2]
```

---

### Common Bit Manipulation Tricks

| Trick | Code | Use Case |
|-------|------|----------|
| Get ith bit | `(n >> i) & 1` | Check specific bit |
| Set ith bit | `n \| (1 << i)` | Turn on bit |
| Clear ith bit | `n & ~(1 << i)` | Turn off bit |
| Toggle ith bit | `n ^ (1 << i)` | Flip bit |
| Check even/odd | `n & 1` | Odd: 1, Even: 0 |
| Multiply by 2 | `n << 1` | Fast multiplication |
| Divide by 2 | `n >> 1` | Fast division |
| Clear lowest set bit | `n & (n-1)` | Count bits, power of 2 |
| Get lowest set bit | `n & (-n)` | Isolate lowest bit |
| Check power of 2 | `(n & (n-1)) == 0` | Only one bit set |

---

### Code Templates (4 Languages)

#### Python

```python
# Single Number
def singleNumber(nums):
    result = 0
    for num in nums:
        result ^= num
    return result

# Count Bits
def countBits(n):
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        dp[i] = dp[i >> 1] + (i & 1)
    return dp

# Is Power of Two
def isPowerOfTwo(n):
    return n > 0 and (n & (n - 1)) == 0

# Subsets (Bitmask)
def subsets(nums):
    n = len(nums)
    result = []
    for mask in range(1 << n):
        subset = [nums[i] for i in range(n) if mask & (1 << i)]
        result.append(subset)
    return result
```

#### Java

```java
// Single Number
int singleNumber(int[] nums) {
    int result = 0;
    for (int num : nums) result ^= num;
    return result;
}

// Count Bits
int[] countBits(int n) {
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}

// Is Power of Two
boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

#### C++

```cpp
// Single Number
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int num : nums) result ^= num;
    return result;
}

// Count Bits
vector<int> countBits(int n) {
    vector<int> dp(n + 1, 0);
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}

// Is Power of Two
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

#### JavaScript

```javascript
// Single Number
function singleNumber(nums) {
    let result = 0;
    for (const num of nums) result ^= num;
    return result;
}

// Count Bits
function countBits(n) {
    const dp = new Array(n + 1).fill(0);
    for (let i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}

// Is Power of Two
function isPowerOfTwo(n) {
    return n > 0 && (n & (n - 1)) === 0;
}
```

---

### Common Mistakes

#### Mistake 1: Operator Precedence

```python
# WRONG: & has lower precedence than ==
if n & n - 1 == 0:  # This is n & ((n-1) == 0), which is wrong!

# RIGHT: use parentheses
if (n & (n - 1)) == 0:
```

#### Mistake 2: Negative Numbers

```python
# WRONG: negative numbers aren't powers of 2
isPowerOfTwo(-4)  # Should return False

# RIGHT: check n > 0
return n > 0 and (n & (n - 1)) == 0
```

#### Mistake 3: Integer Overflow (in some languages)

```python
# In Java/C++, 1 << 31 can overflow
# Use 1L (long) or handle sign bit carefully
```

---

### Practice Problems

#### Easy

| # | Problem | Key |
|---|---------|-----|
| 1 | [136. Single Number](https://leetcode.com/problems/single-number/) | XOR |
| 2 | [191. Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | Count bits |
| 3 | [231. Power of Two](https://leetcode.com/problems/power-of-two/) | n & (n-1) |
| 4 | [338. Counting Bits](https://leetcode.com/problems/counting-bits/) | DP + bits |
| 5 | [268. Missing Number](https://leetcode.com/problems/missing-number/) | XOR |

#### Medium

| # | Problem | Key |
|---|---------|-----|
| 6 | [137. Single Number II](https://leetcode.com/problems/single-number-ii/) | Count bits per position |
| 7 | [260. Single Number III](https://leetcode.com/problems/single-number-iii/) | XOR + isolate bit |
| 8 | [78. Subsets](https://leetcode.com/problems/subsets/) | Bitmask |
| 9 | [371. Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Bit addition |
| 10 | [318. Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/) | Bitmask for characters |

---

### Interview Tips

1. **State the bit trick.** "I'll use XOR because a ^ a = 0 and a ^ 0 = a."

2. **Explain the binary representation.** "In binary, 5 is 101, so the 0th and 2nd bits are set."

3. **Mention time/space complexity.** "This is O(n) time and O(1) space."

4. **Test with small examples.** Use 1-2 bit numbers to verify.

5. **Edge cases:**
   - n = 0
   - Negative numbers
   - Maximum integer value
   - All bits set

---

## Pattern Recognition Cheat Sheet

Now that you've learned all 16 patterns, here's a quick reference for recognizing which pattern to use:

```
Array/String Problem?
├── Contiguous? → Sliding Window
├── Sorted? → Two Pointers or Binary Search
├── Pairs/Triplets? → Two Pointers + Sort
├── Subarray Sum? → Prefix Sum + HashMap
├── Next Greater/Smaller? → Monotonic Stack
├── Top K? → Heap
├── Min/Max/Count? → Dynamic Programming or Greedy

Tree Problem?
├── Traversal? → DFS (inorder, preorder, postorder)
├── Level Order? → BFS
├── Path Sum? → DFS + Prefix Sum
├── BST? → Use sorted property
├── LCA? → DFS divide and conquer

Graph Problem?
├── Connected Components? → DFS or Union Find
├── Shortest Path (unweighted)? → BFS
├── Shortest Path (weighted)? → Dijkstra or Bellman-Ford
├── Dependencies/Order? → Topological Sort
├── Cycle Detection? → DFS or Union Find
├── All Pairs Shortest? → Floyd-Warshall

Linked List?
├── Reverse? → Three pointers
├── Middle? → Fast & Slow
├── Cycle? → Fast & Slow
├── Merge? → Dummy head

Other?
├── All Combinations? → Backtracking
├── Groups/Sets? → Union Find
├── Single/Unique? → Bit Manipulation
├── Scheduling? → Greedy
```

---

*Congratulations! You've completed the DSA Patterns Handbook. Now go solve problems with pattern recognition, not memorization.*
