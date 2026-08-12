# 03 — Binary Search

## The Pattern That Turns O(n) Into O(log n)

---

### What It Is

Binary Search is a technique for efficiently finding a value in a **sorted** or **monotonic** search space by repeatedly halving the range.

Most people think binary search is just "find an element in a sorted array." That's one use. The real power is **binary search on the answer**—when you're looking for the minimum or maximum value that satisfies a condition, and you can check whether a candidate value works in O(n) or better.

**The key insight:** If you can define a monotonic predicate (true up to some point, then false, or vice versa), you can binary search for the boundary.

---

### When to Use It

**The problem involves:**
- A sorted array (classic binary search)
- Finding the minimum feasible value (binary search on answer)
- Finding the maximum feasible value (binary search on answer)
- A monotonic condition that can be checked efficiently

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "sorted array" | Search in Rotated Sorted Array |
| "find minimum" | Minimum time to complete trips |
| "find maximum" | Maximum candies to distribute |
| "minimum possible answer" | Split array largest sum |
| "maximum possible answer" | Magnetic force between balls |
| "feasible" | Capacity to ship packages |
| "at least" | Koko eating bananas |
| "at most" | Find smallest divisor |
| "kth smallest/largest" | Kth Smallest Element in Sorted Matrix |
| "search" + "sorted" | Search a 2D Matrix |
| "rotated" | Find Minimum in Rotated Sorted Array |
| "first/last occurrence" | Find First and Last Position |

**The Decision:**

```
Is the data sorted?
    ↓
Yes → Classic Binary Search (Type 1)

Is the data sorted?
    ↓
No
    ↓
Can you define "is X a valid answer"?
    ↓
Yes → Binary Search on Answer (Type 2)
```

---

### The Two Types of Binary Search

#### Type 1: Binary Search on Array

The array is sorted. You're searching for a specific value or boundary.

```
Classic: Find target in sorted array
First:   Find first occurrence of target
Last:    Find last occurrence of target
First ≥: Find first element ≥ target
Last ≤:  Find last element ≤ target
```

#### Type 2: Binary Search on Answer

The answer is a number in some range [lo, hi]. You're searching for the minimum or maximum value that satisfies a feasibility check.

```
Given: a function feasible(mid) that returns True/False
Find: the smallest (or largest) value where feasible() is True

The key: the feasibility function must be monotonic
    [False, False, ..., False, True, True, ..., True]
                              ↑
                          Find this boundary
```

---

### Mental Model: The Number Line

```
Type 1 (Array):

    [1, 3, 5, 7, 9, 11, 13]
     ↑              ↑
    lo             hi
          mid = 7
    
    Looking for 9: 9 > 7, so search right half
    [1, 3, 5, 7, 9, 11, 13]
                  ↑     ↑
                 lo    hi
                     mid = 11
    
    9 < 11, so search left half
    [1, 3, 5, 7, 9, 11, 13]
                  ↑
                 lo=hi=mid=9
    Found!


Type 2 (On Answer):

    Answer range: [1, 100]
    
    Is 50 feasible? No → answer must be > 50
    Is 75 feasible? Yes → answer could be ≤ 75
    Is 62 feasible? No → answer must be > 62
    Is 68 feasible? Yes → answer could be ≤ 68
    ...
    Eventually converge to the minimum feasible answer.
```

---

### The 80% Template: Classic Binary Search

```python
def binary_search(nums, target):
    lo, hi = 0, len(nums) - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2  # avoid overflow
        
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    
    return -1  # not found
```

**Why `lo + (hi - lo) // 2` instead of `(lo + hi) // 2`:**
- In languages with fixed-size integers, `(lo + hi)` can overflow
- Python doesn't have this issue, but it's good practice everywhere

---

### The 80% Template: Binary Search on Answer

```python
def binary_search_on_answer(nums, k):
    def feasible(mid):
        # Check if mid is a valid answer
        # Return True if mid works, False otherwise
        pass
    
    lo, hi = min_possible_answer, max_possible_answer
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        
        if feasible(mid):
            hi = mid      # mid works, try smaller
        else:
            lo = mid + 1  # mid doesn't work, need larger
    
    return lo  # lo == hi == minimum feasible answer
```

**Key difference from classic:**
- Classic: `while lo <= hi`, returns exact match or -1
- On Answer: `while lo < hi`, returns boundary

**For maximum feasible answer:**
```python
while lo < hi:
    mid = lo + (hi - lo + 1) // 2  # round up to avoid infinite loop
    if feasible(mid):
        lo = mid       # mid works, try larger
    else:
        hi = mid - 1   # mid doesn't work, need smaller
return lo
```

---

### The Five Binary Search Variants

| Variant | Goal | Template Change |
|---------|------|-----------------|
| Find exact target | Return index of target | Standard template |
| Find first occurrence | Return first index of target | `hi = mid` when found |
| Find last occurrence | Return last index of target | `lo = mid` when found |
| Find first ≥ target | Return first index where nums[i] ≥ target | `hi = mid` when nums[mid] ≥ target |
| Find last ≤ target | Return last index where nums[i] ≤ target | `lo = mid` when nums[mid] ≤ target |

#### First Occurrence

```python
def find_first(nums, target):
    lo, hi = 0, len(nums) - 1
    result = -1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            result = mid
            hi = mid - 1  # keep searching left
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    
    return result
```

#### Last Occurrence

```python
def find_last(nums, target):
    lo, hi = 0, len(nums) - 1
    result = -1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            result = mid
            lo = mid + 1  # keep searching right
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    
    return result
```

#### First Element ≥ Target (Lower Bound)

```python
def lower_bound(nums, target):
    lo, hi = 0, len(nums) - 1
    result = len(nums)  # default: not found
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] >= target:
            result = mid
            hi = mid - 1
        else:
            lo = mid + 1
    
    return result
```

---

### Dry Run: Classic Binary Search

**Problem:** Find target in a sorted array.

**Input:** `nums = [1, 3, 5, 7, 9, 11, 13]`, `target = 9`

```python
def search(nums, target):
    lo, hi = 0, len(nums) - 1
    
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    
    return -1
```

**Step-by-step:**

```
nums = [1, 3, 5, 7, 9, 11, 13]    target = 9

Iteration 1:
  lo=0, hi=6
  mid = 0 + (6-0)//2 = 3
  nums[3] = 7 < 9 → search right
  lo = 4

Iteration 2:
  lo=4, hi=6
  mid = 4 + (6-4)//2 = 5
  nums[5] = 11 > 9 → search left
  hi = 4

Iteration 3:
  lo=4, hi=4
  mid = 4
  nums[4] = 9 == 9 → FOUND! Return 4
```

---

### Dry Run: Koko Eating Bananas (Binary Search on Answer)

**Problem:** Koko has `h` hours to eat all bananas from `n` piles. She eats at speed `k` bananas/hour from one pile. Find the minimum `k` such that she can finish all piles within `h` hours.

**Input:** `piles = [3, 6, 7, 11]`, `h = 8`

**Recognition:** "minimum speed" + "can finish within h hours" → Binary Search on Answer

**Why Binary Search works here:**
- If Koko eats at speed `k`, she needs `ceil(pile/k)` hours per pile
- If `k` is too small, she can't finish in time (infeasible)
- If `k` is large enough, she finishes in time (feasible)
- The feasibility is monotonic: increasing `k` always helps (or doesn't hurt)

```python
import math

def minEatingSpeed(piles, h):
    def feasible(k):
        # Can Koko finish all piles within h hours at speed k?
        total_hours = sum(math.ceil(pile / k) for pile in piles)
        return total_hours <= h
    
    lo, hi = 1, max(piles)
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if feasible(mid):
            hi = mid       # mid works, try smaller
        else:
            lo = mid + 1   # too slow, need larger
    
    return lo
```

**Step-by-step:**

```
piles = [3, 6, 7, 11]    h = 8

Search range: [1, 11]

mid = 6:
  hours = ceil(3/6) + ceil(6/6) + ceil(7/6) + ceil(11/6)
        = 1 + 1 + 2 + 2 = 6 ≤ 8 ✓ feasible
  → try smaller: hi = 6

mid = 3:
  hours = ceil(3/3) + ceil(6/3) + ceil(7/3) + ceil(11/3)
        = 1 + 2 + 3 + 4 = 10 > 8 ✗ not feasible
  → need larger: lo = 4

mid = 5:
  hours = ceil(3/5) + ceil(6/5) + ceil(7/5) + ceil(11/5)
        = 1 + 2 + 2 + 3 = 8 ≤ 8 ✓ feasible
  → try smaller: hi = 5

mid = 4:
  hours = ceil(3/4) + ceil(6/4) + ceil(7/4) + ceil(11/4)
        = 1 + 2 + 2 + 3 = 8 ≤ 8 ✓ feasible
  → try smaller: hi = 4

lo == hi == 4. Return 4.
```

**Verification:** At speed 4, Koko needs 8 hours exactly. At speed 3, she needs 10 hours (too slow). Minimum speed is 4.

---

### Dry Run: Find Minimum in Rotated Sorted Array

**Problem:** Find the minimum element in a rotated sorted array.

**Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`

**Recognition:** "rotated sorted array" + "find minimum" → Binary Search (modified)

**Key insight:** In a rotated sorted array, one half is always sorted. The minimum is in the unsorted half.

```python
def findMin(nums):
    lo, hi = 0, len(nums) - 1
    
    while lo < hi:
        mid = lo + (hi - lo) // 2
        
        if nums[mid] > nums[hi]:
            # Minimum is in the right half (unsorted part)
            lo = mid + 1
        else:
            # Minimum is in the left half (including mid)
            hi = mid
    
    return nums[lo]
```

**Step-by-step:**

```
nums = [4, 5, 6, 7, 0, 1, 2]

Iteration 1:
  lo=0, hi=6, mid=3
  nums[3]=7 > nums[6]=2 → minimum is right of mid
  lo = 4

Iteration 2:
  lo=4, hi=6, mid=5
  nums[5]=1 ≤ nums[6]=2 → minimum is at mid or left
  hi = 5

Iteration 3:
  lo=4, hi=5, mid=4
  nums[4]=0 ≤ nums[5]=1 → minimum is at mid or left
  hi = 4

lo == hi == 4. Return nums[4] = 0.
```

---

### When to Use Binary Search on Answer

This is the hardest variant to recognize. Here's the checklist:

```
1. The answer is a number (not an index)
2. You can check if a candidate answer is feasible in O(n) or better
3. The feasibility is monotonic (if x works, then x+1 also works, or vice versa)
4. The search space has clear bounds

If all four are true → Binary Search on Answer
```

**Common problems:**

| Problem | What We're Searching | Feasibility Check |
|---------|---------------------|-------------------|
| Koko Eating Bananas | Minimum eating speed | Can finish in h hours? |
| Capacity to Ship Packages | Minimum ship capacity | Can ship in d days? |
| Split Array Largest Sum | Minimum largest sum | Can split into k subarrays? |
| Magnetic Force Between Balls | Maximum minimum distance | Can place m balls with this distance? |
| Find Smallest Divisor | Minimum divisor | Division sum ≤ threshold? |

---

### Code Templates (4 Languages)

#### Python

```python
# Classic Binary Search
def binary_search(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1

# Binary Search on Answer (minimize)
def bs_min_answer(nums, k):
    def feasible(mid):
        # return True if mid is a valid answer
        pass
    
    lo, hi = 1, max(nums)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo

# Binary Search on Answer (maximize)
def bs_max_answer(nums, k):
    def feasible(mid):
        pass
    
    lo, hi = 1, max(nums)
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2  # round up!
        if feasible(mid):
            lo = mid
        else:
            hi = mid - 1
    return lo
```

#### Java

```java
// Classic
int binarySearch(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// Binary Search on Answer
int bsMinAnswer(int[] nums, int k) {
    int lo = 1, hi = Arrays.stream(nums).max().getAsInt();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid, nums, k)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

#### C++

```cpp
// Classic
int binarySearch(vector<int>& nums, int target) {
    int lo = 0, hi = nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// Using std library
int binarySearchSTL(vector<int>& nums, int target) {
    auto it = lower_bound(nums.begin(), nums.end(), target);
    if (it != nums.end() && *it == target) {
        return it - nums.begin();
    }
    return -1;
}
```

#### JavaScript

```javascript
// Classic
function binarySearch(nums, target) {
    let lo = 0, hi = nums.length - 1;
    while (lo <= hi) {
        const mid = lo + Math.floor((hi - lo) / 2);
        if (nums[mid] === target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// Binary Search on Answer
function bsMinAnswer(nums, k) {
    function feasible(mid) { /* ... */ }
    
    let lo = 1, hi = Math.max(...nums);
    while (lo < hi) {
        const mid = lo + Math.floor((hi - lo) / 2);
        if (feasible(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

---

### Common Mistakes

#### Mistake 1: Infinite Loop in Binary Search on Answer

```python
# WRONG: if feasible(mid) is always True
while lo < hi:
    mid = (lo + hi) // 2  # If lo=5, hi=6, mid=5
    if feasible(mid):
        hi = mid  # hi stays 5, lo stays 5 → done
    # But if we're maximizing:
    if feasible(mid):
        lo = mid  # lo stays 5, infinite loop!

# RIGHT: when maximizing, round mid UP
while lo < hi:
    mid = lo + (hi - lo + 1) // 2  # Round up
    if feasible(mid):
        lo = mid
    else:
        hi = mid - 1
```

#### Mistake 2: Off-by-One in Range

```python
# WRONG: might miss the answer
lo, hi = 0, len(nums) - 1  # for search on answer, hi should be max possible value

# RIGHT: set bounds correctly
lo, hi = min_possible_answer, max_possible_answer
```

#### Mistake 3: Using <= When You Should Use <

```python
# For classic search (finding exact value):
while lo <= hi:  # CORRECT

# For binary search on answer (finding boundary):
while lo < hi:   # CORRECT
```

#### Mistake 4: Wrong Feasibility Direction

```python
# For MINIMUM feasible answer:
if feasible(mid):
    hi = mid       # mid works, try smaller
else:
    lo = mid + 1   # mid too small

# For MAXIMUM feasible answer:
if feasible(mid):
    lo = mid       # mid works, try larger
else:
    hi = mid - 1   # mid too large
```

---

### Complexity Analysis

| Aspect | Complexity | Why |
|--------|-----------|-----|
| Time | O(log n) | Search space halves each iteration |
| Space | O(1) | Only pointers (iterative) or O(log n) (recursive) |

**For Binary Search on Answer:**
- Time: O(n × log(max_answer)) if feasibility check is O(n)
- Example: Koko eating bananas with 10^4 piles and max pile size 10^9 → O(10^4 × 30) = O(3 × 10^5)

---

### Practice Problems

#### Classic Binary Search

| # | Problem | Variant |
|---|---------|---------|
| 1 | [704. Binary Search](https://leetcode.com/problems/binary-search/) | Exact match |
| 2 | [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/) | First ≥ target |
| 3 | [34. Find First and Last Position](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | First and last occurrence |
| 4 | [744. Find Smallest Letter Greater Than Target](https://leetcode.com/problems/find-smallest-letter-greater-than-target/) | First > target |
| 5 | [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Modified classic |
| 6 | [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Find pivot |

#### Binary Search on Answer

| # | Problem | What We're Searching |
|---|---------|---------------------|
| 7 | [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Minimum speed |
| 8 | [1011. Capacity To Ship Packages](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Minimum capacity |
| 9 | [410. Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Minimum largest sum |
| 10 | [1283. Find the Smallest Divisor](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/) | Minimum divisor |
| 11 | [1482. Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) | Minimum days |
| 12 | [1552. Magnetic Force Between Balls](https://leetcode.com/problems/magnetic-force-between-balls/) | Maximum minimum distance |
| 13 | [2064. Minimized Maximum of Products](https://leetcode.com/problems/minimized-maximum-of-products-distributed-to-any-store/) | Minimize maximum |
| 14 | [1760. Minimum Limit of Balls in a Bag](https://leetcode.com/problems/minimum-limit-of-balls-in-a-bag/) | Minimum penalty |
| 15 | [1870. Minimum Speed to Arrive on Time](https://leetcode.com/problems/minimum-speed-to-arrive-on-time/) | Minimum speed |

#### Advanced

| # | Problem | Notes |
|---|---------|-------|
| 16 | [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Binary search on partition |
| 17 | [719. Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/) | Binary search on answer + two pointers |
| 18 | [1231. Divide Chocolate](https://leetcode.com/problems/divide-chocolate/) | Binary search on answer |

---

### Recognition Exercises

**Exercise 1:** "Find the minimum eating speed so that Koko can finish all banana piles within h hours."

<details>
<summary>Answer</summary>

- Type: Binary Search on Answer
- Search space: [1, max(piles)]
- Feasibility: Can finish all piles in ≤ h hours at speed k?
- Direction: Minimize → when feasible, search left (hi = mid)

</details>

**Exercise 2:** "Given a sorted array of 0s and 1s, find the index of the first 1."

<details>
<summary>Answer</summary>

- Type: Classic Binary Search (First Occurrence / Lower Bound)
- Search for the first element ≥ 1
- When nums[mid] ≥ 1, record answer and search left

</details>

**Exercise 3:** "What is the maximum distance between any two gas stations if we add k new stations?"

<details>
<summary>Answer</summary>

- Type: Binary Search on Answer
- Search space: [0, max_gap]
- Feasibility: Can we achieve maximum gap ≤ D with k new stations?
- Direction: Maximize → when feasible, search right (lo = mid)

Wait—actually we want to MINIMIZE the maximum gap. So:
- Direction: Minimize → when feasible, search left (hi = mid)

</details>

---

### Related Patterns

| Pattern | When to Use Instead |
|---------|---------------------|
| Two Pointers | When the array is sorted and you're looking for pairs |
| Sliding Window | When you're looking for contiguous subarrays |
| Heap | When you need the Kth element but data isn't sorted |
| Divide and Conquer | When the problem naturally splits into subproblems |

---

### Interview Tips

1. **Always explain the search space.** "I'm searching for the answer in the range [lo, hi]."

2. **State the feasibility function.** "I'll check if a candidate answer of `mid` is feasible by doing [X]."

3. **Explain monotonicity.** "If speed k works, then speed k+1 also works. This lets me binary search."

4. **Handle the "minimize" vs "maximize" distinction clearly.** This determines whether you round `mid` up or down and which pointer you move.

5. **Test with small examples.** Walk through 2-3 iterations manually before coding.

6. **Edge cases to consider:**
   - Empty array
   - Single element
   - All elements the same
   - Target not in array
   - Answer is at the boundary of the range

7. **If stuck, ask:** "Is the answer a number? Can I check if a number is feasible? Is feasibility monotonic?" If all yes, binary search on answer.

---

*Next: [04 — Prefix Sum and HashMap](https://github.com/Aditya-myst/Interview-PlayBook/edit/main/DSA%20Handbook/Prefix-Sum-and-HashMap.md)*
