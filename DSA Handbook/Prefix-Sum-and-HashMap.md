# 04 — Prefix Sum + HashMap

## The Pattern That Answers "How Many?" in O(n)

---

### What It Is

Prefix Sum is a technique where you precompute cumulative sums so you can answer "what's the sum from index i to j?" in O(1) time.

HashMap is a data structure that gives O(1) lookup.

**Together**, they solve a class of problems that seem to require O(n²): "How many subarrays have sum equal to K?" or "Does a subarray with sum K exist?"

**The key insight:** If `prefix[j] - prefix[i] = K`, then the subarray from `i+1` to `j` has sum `K`. So for each `j`, you just need to know: "How many previous prefix sums equal `prefix[j] - K`?"

---

### When to Use It

**The problem involves:**
- Sum of contiguous subarrays
- Counting subarrays with a specific sum
- Finding subarrays with a specific sum
- Running totals or cumulative calculations

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "subarray sum" | Subarray Sum Equals K |
| "count subarrays" | Count subarrays with sum divisible by K |
| "continuous subarray" | Continuous Subarray Sum |
| "running sum" | Running Sum of 1d Array |
| "sum equals" | Find subarray with given sum |
| "sum divisible by" | Subarray Sums Divisible by K |
| "balance" | Find the highest altitude |
| "cumulative" | Product of Array Except Self (variation) |

**The Decision:**

```
Does the problem ask about sums of contiguous subarrays?
    ↓
Yes
    ↓
Do you need to COUNT or FIND subarrays with a specific sum?
    ↓
Yes
    ↓
PREFIX SUM + HASHMAP
```

---

### Mental Model: The Running Total

Think of prefix sum as a "running total" you keep as you scan the array.

```
Array:        [3,  4,  7,  2, -3,  1,  4,  2]
Prefix Sum: [0, 3, 7, 14, 16, 13, 14, 18, 20]
              ↑
              Start with 0 (sum of empty prefix)
```

Now, `sum(i, j) = prefix[j+1] - prefix[i]`

```
Sum from index 2 to 5:
= prefix[6] - prefix[2]
= 14 - 7
= 7

Verify: 7 + 2 + (-3) + 1 = 7 ✓
```

The HashMap part: instead of storing all prefix sums and checking all pairs (O(n²)), store them in a HashMap and look up the complement in O(1).

---

### The Core Formula

```
prefix[j] - prefix[i] = target_sum
                ↓
prefix[i] = prefix[j] - target_sum
```

For each position `j`:
1. Calculate `prefix[j]`
2. Look up `prefix[j] - target_sum` in the HashMap
3. The count tells you how many subarrays ending at `j` have sum = `target_sum`
4. Add `prefix[j]` to the HashMap

---

### The 80% Template

```c++
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

int subarray_sum(vector<int>& nums, int k) {
    int count = 0;
    int prefix_sum = 0;
    unordered_map<int, int> seen;

    // Base case: prefix sum 0 has occurred once
    seen[0] = 1;

    for (int num : nums) {
        prefix_sum += num;

        // Check if there is a previous prefix sum such that
        // prefix_sum - previous_prefix_sum = k
        if (seen.find(prefix_sum - k) != seen.end()) {
            count += seen[prefix_sum - k];
        }

        // Record the current prefix sum
        seen[prefix_sum]++;
    }

    return count;
}

int main() {
    vector<int> nums = {1, 1, 1};
    int k = 2;

    cout << subarray_sum(nums, k) << endl; // Output: 2

    return 0;
}
```

**Why start with `{0: 1}`:**
- If `prefix_sum == k` at some point, the subarray from index 0 to that point has sum `k`
- We need to count this, so we pretend we've "seen" a prefix sum of 0 before we started

---

### Dry Run: Subarray Sum Equals K

**Problem:** Given an array of integers and an integer `k`, find the total number of continuous subarrays whose sum equals `k`.

**Input:** `nums = [1, 1, 1]`, `k = 2`

**Recognition:** "count" + "subarrays" + "sum equals k" → Prefix Sum + HashMap

```c++
#include <vector>
#include <unordered_map>
using namespace std;

int subarraySum(vector<int>& nums, int k) {
    int count = 0;
    int prefix_sum = 0;
    unordered_map<int, int> seen;

    // Prefix sum 0 occurs once before the array starts
    seen[0] = 1;

    for (int num : nums) {
        prefix_sum += num;

        if (seen.find(prefix_sum - k) != seen.end()) {
            count += seen[prefix_sum - k];
        }

        seen[prefix_sum]++;
    }

    return count;
}
```

**Step-by-step:**

```
nums = [1, 1, 1]    k = 2

Initial: seen = {0: 1}, prefix_sum = 0, count = 0

i=0, num=1:
  prefix_sum = 0 + 1 = 1
  Is (1 - 2) = -1 in seen? No.
  seen = {0: 1, 1: 1}

i=1, num=1:
  prefix_sum = 1 + 1 = 2
  Is (2 - 2) = 0 in seen? Yes, seen[0] = 1 → count += 1
  count = 1 (subarray [1,1] from index 0 to 1)
  seen = {0: 1, 1: 1, 2: 1}

i=2, num=1:
  prefix_sum = 2 + 1 = 3
  Is (3 - 2) = 1 in seen? Yes, seen[1] = 1 → count += 1
  count = 2 (subarray [1,1] from index 1 to 2)
  seen = {0: 1, 1: 1, 2: 1, 3: 1}

Return 2
```

**Verification:**
- Subarray [1,1] (indices 0-1): sum = 2 ✓
- Subarray [1,1] (indices 1-2): sum = 2 ✓
- Total: 2

---

### Dry Run: Continuous Subarray Sum

**Problem:** Given an integer array and integer `k`, return `true` if the array has a continuous subarray of size ≥ 2 that sums to a multiple of `k`.

**Input:** `nums = [23, 2, 4, 6, 7]`, `k = 6`

**Recognition:** "continuous subarray" + "sum divisible by k" → Prefix Sum + HashMap (using modulo)

**Key insight:** If `prefix[j] % k == prefix[i] % k` and `j - i >= 2`, then the subarray from `i+1` to `j` has sum divisible by `k`.

```c++
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> seen;
        seen[0] = -1;  // remainder -> earliest index

        int prefix_sum = 0;

        for (int i = 0; i < nums.size(); i++) {
            prefix_sum += nums[i];
            int remainder = prefix_sum % k;

            if (seen.find(remainder) != seen.end()) {
                if (i - seen[remainder] >= 2) {
                    return true;
                }
            } else {
                // Store only the earliest occurrence
                seen[remainder] = i;
            }
        }

        return false;
    }
};
```

**Step-by-step:**

```
nums = [23, 2, 4, 6, 7]    k = 6

Initial: seen = {0: -1}, prefix_sum = 0

i=0, num=23:
  prefix_sum = 23
  remainder = 23 % 6 = 5
  5 not in seen → seen = {0: -1, 5: 0}

i=1, num=2:
  prefix_sum = 25
  remainder = 25 % 6 = 1
  1 not in seen → seen = {0: -1, 5: 0, 1: 1}

i=2, num=4:
  prefix_sum = 29
  remainder = 29 % 6 = 5
  5 in seen at index 0 → i - seen[5] = 2 - 0 = 2 ≥ 2 → return True!

Return True
```

**Verification:** Subarray [23, 2, 4] (indices 0-2) has sum 29. Wait, 29 % 6 ≠ 0.

Actually, let me recalculate: 23 + 2 + 4 = 29, 29 % 6 = 5. But we're comparing remainders: prefix[2] % 6 = 5 and prefix[0] % 6 = 5. The subarray from index 1 to 2 has sum = 2 + 4 = 6, which IS divisible by 6. ✓

The formula works: same remainder means the difference (subarray sum) is divisible by `k`.

---

### Variations

#### Variation 1: Subarray Sum Divisible by K

Same as above but count (not just find) all subarrays.

```python
def subarraysDivByK(nums, k):
    count = 0
    prefix_sum = 0
    seen = {0: 1}
    
    for num in nums:
        prefix_sum += num
        remainder = prefix_sum % k
        count += seen.get(remainder, 0)
        seen[remainder] = seen.get(remainder, 0) + 1
    
    return count
```

#### Variation 2: Product of Array Except Self

Uses prefix and suffix products (no division).

```python
def productExceptSelf(nums):
    n = len(nums)
    result = [1] * n
    
    # Left pass: result[i] = product of all elements to the left
    left = 1
    for i in range(n):
        result[i] = left
        left *= nums[i]
    
    # Right pass: multiply by product of all elements to the right
    right = 1
    for i in range(n - 1, -1, -1):
        result[i] *= right
        right *= nums[i]
    
    return result
```

#### Variation 3: 2D Prefix Sum

For submatrix sums.

```python
def build_prefix_2d(matrix):
    m, n = len(matrix), len(matrix[0])
    prefix = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            prefix[i][j] = (matrix[i-1][j-1] 
                           + prefix[i-1][j] 
                           + prefix[i][j-1] 
                           - prefix[i-1][j-1])
    return prefix

def sum_region(prefix, r1, c1, r2, c2):
    return (prefix[r2+1][c2+1] 
            - prefix[r1][c2+1] 
            - prefix[r2+1][c1] 
            + prefix[r1][c1])
```

---

### Common Mistakes

#### Mistake 1: Forgetting `{0: 1}`

```python
# WRONG: misses subarrays starting at index 0
seen = {}

# RIGHT: include 0 with count 1
seen = {0: 1}
```

#### Mistake 2: Not Handling Negative Numbers

```python
# Prefix sum works with negative numbers!
# nums = [-1, -1, 1], k = 0
# Prefix sums: [0, -1, -2, -1]
# Subarray [-1, 1] (indices 1-2) has sum 0 ✓
```

#### Mistake 3: Using Prefix Sum for Non-Contiguous Elements

```python
# WRONG: prefix sum only works for CONTIGUOUS subarrays
# For non-contiguous subsequences, use DP instead
```

#### Mistake 4: Integer Overflow (in some languages)

```python
# In Java/C++, prefix sums can overflow if values are large
# Use long long (C++) or long (Java) for prefix_sum
```

---

### Complexity Analysis

| Aspect | Complexity | Why |
|--------|-----------|-----|
| Time | O(n) | Single pass through the array |
| Space | O(n) | HashMap stores up to n prefix sums |

---

### Practice Problems

#### Easy

| # | Problem | Key |
|---|---------|-----|
| 1 | [303. Range Sum Query](https://leetcode.com/problems/range-sum-query-immutable/) | Basic prefix sum |
| 2 | [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/) | Basic prefix sum |
| 3 | [724. Find Pivot Index](https://leetcode.com/problems/find-pivot-index/) | Left sum = right sum |

#### Medium

| # | Problem | Key |
|---|---------|-----|
| 4 | [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Classic prefix + hashmap |
| 5 | [523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/) | Prefix + modulo |
| 6 | [974. Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | Prefix + modulo |
| 7 | [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | Prefix + suffix products |
| 8 | [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Treat 0 as -1, find sum=0 |
| 9 | [930. Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) | Prefix + hashmap |
| 10 | [1248. Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | Prefix + hashmap (odd count) |
| 11 | [304. Range Sum Query 2D](https://leetcode.com/problems/range-sum-query-2d-immutable/) | 2D prefix sum |

#### Hard

| # | Problem | Key |
|---|---------|-----|
| 12 | [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/) | Prefix sum on tree |
| 13 | [666. Path Sum IV](https://leetcode.com/problems/path-sum-iv/) | Prefix sum on tree |
| 14 | [1590. Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/) | Prefix + modulo |

---

### Interview Tips

1. **Always initialize with `{0: 1}`.** Explain why: "To handle subarrays starting at index 0."

2. **Explain the formula.** "If two prefix sums have the same value (or same remainder), the subarray between them has sum 0 (or divisible by k)."

3. **Mention it handles negative numbers.** "Unlike sliding window, prefix sum + hashmap works with negative numbers."

4. **For 2D problems, draw the inclusion-exclusion diagram.** "The sum of a submatrix is the big rectangle minus the two overlapping rectangles plus the double-counted corner."

5. **Edge cases:**
   - All zeros
   - Single element equals k
   - No subarray sums to k
   - Negative numbers

---

*Next: [05 — Monotonic Stack](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Monotonic-stack.md)*
