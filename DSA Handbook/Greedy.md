# 12 — Greedy

## The Pattern That Makes the Locally Optimal Choice

---

### What It Is

Greedy is a technique where you make the locally optimal choice at each step, hoping to find the global optimum. Unlike DP, which considers all possibilities, Greedy commits to a choice without reconsidering it.

**The key insight:** Greedy works when the problem has the **greedy choice property** (a locally optimal choice leads to a globally optimal solution) and **optimal substructure**.

---

### When to Use It

**The problem has:**
- A sequence of choices where each choice is independent of future choices
- The locally optimal choice is always part of the optimal solution
- No need to reconsider previous choices

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "maximize" / "minimize" | Maximum Subarray |
| "interval scheduling" | Non-overlapping Intervals |
| "jump" | Jump Game |
| "assign" | Task Scheduler |
| "minimum number" | Minimum Number of Arrows |
| "can you reach" | Jump Game |
| "earliest" / "latest" | Meeting Rooms |
| "sort by" + "choose" | Activity Selection |

**The Decision:**

```
Can you make a locally optimal choice that leads to a global optimum?
    ↓
Yes (with proof)
    ↓
GREEDY

Otherwise → DP
```

**How to verify Greedy works:**
1. **Greedy Choice Property:** Making the locally optimal choice never prevents reaching the global optimum
2. **Optimal Substructure:** The problem can be broken into subproblems

If you can't prove these, use DP instead.

---

### Mental Model: The Activity Selection

```
Activities sorted by end time:
A: [1, 4]
B: [3, 5]
C: [0, 6]
D: [5, 7]
E: [3, 9]
F: [5, 9]
G: [6, 10]
H: [8, 11]

Greedy choice: Always pick the activity that ends earliest.

Pick A [1,4] → last_end = 4
Skip B (starts at 3 < 4)
Skip C (starts at 0 < 4)
Pick D [5,7] → last_end = 7
Skip E (starts at 3 < 7)
Skip F (starts at 5 < 7)
Pick G [6,10]? No, 6 < 7. Skip.
Pick H [8,11] → last_end = 11

Selected: A, D, H (3 activities)
```

---

### The 80% Template: Interval Greedy

```python
def greedy_intervals(intervals):
    # Sort by end time (or start time, depending on problem)
    intervals.sort(key=lambda x: x[1])
    
    count = 0
    last_end = float('-inf')
    
    for start, end in intervals:
        if start >= last_end:  # No overlap
            count += 1
            last_end = end
    
    return count
```

---

### Dry Run: Jump Game

**Problem:** Given an array where each element represents max jump length, determine if you can reach the last index.

**Input:** `nums = [2, 3, 1, 1, 4]`

**Recognition:** "can you reach" + "jump" → Greedy (or DP)

**Greedy insight:** Track the farthest reachable position. If at any point your current position exceeds the farthest, you're stuck.

```python
def canJump(nums):
    farthest = 0
    
    for i in range(len(nums)):
        if i > farthest:
            return False  # Can't reach position i
        farthest = max(farthest, i + nums[i])
        if farthest >= len(nums) - 1:
            return True
    
    return True
```

**Step-by-step:**

```
nums = [2, 3, 1, 1, 4]

i=0: farthest = max(0, 0+2) = 2
i=1: 1 ≤ 2 ✓, farthest = max(2, 1+3) = 4
i=2: 2 ≤ 4 ✓, farthest = max(4, 2+1) = 4
i=3: 3 ≤ 4 ✓, farthest = max(4, 3+1) = 4
i=4: 4 ≤ 4 ✓, farthest = max(4, 4+4) = 8

farthest (8) >= len(nums)-1 (4) → return True
```

**Verification:** Path 0 → 1 → 4 reaches the end. ✓

---

### Dry Run: Non-overlapping Intervals

**Problem:** Find the minimum number of intervals to remove to make the rest non-overlapping.

**Input:** `intervals = [[1,2],[2,3],[3,4],[1,3]]`

**Recognition:** "minimum removals" + "non-overlapping" → Greedy (sort by end time)

**Greedy insight:** Sort by end time. Keep intervals that don't overlap. Count how many you skip (= removed).

```python
def eraseOverlapIntervals(intervals):
    if not intervals:
        return 0
    
    intervals.sort(key=lambda x: x[1])
    count = 0
    last_end = intervals[0][0]
    
    for start, end in intervals:
        if start >= last_end:
            last_end = end  # Keep this interval
        else:
            count += 1  # Remove this interval (overlap)
    
    return count
```

**Step-by-step:**

```
intervals = [[1,2],[2,3],[3,4],[1,3]]

Sort by end time: [[1,2],[2,3],[1,3],[3,4]]

last_end = -inf

[1,2]: 1 >= -inf ✓, keep. last_end = 2
[2,3]: 2 >= 2 ✓, keep. last_end = 3
[1,3]: 1 < 3 ✗, overlap! count = 1
[3,4]: 3 >= 3 ✓, keep. last_end = 4

Return count = 1 (remove interval [1,3])
```

**Verification:** Keeping [1,2], [2,3], [3,4] gives 3 non-overlapping intervals. Removing [1,3] is optimal. ✓

---

### Dry Run: Task Scheduler

**Problem:** Given tasks and cooldown n, find minimum time to complete all tasks.

**Input:** `tasks = ["A","A","A","B","B","B"]`, `n = 2`

**Recognition:** "minimum time" + "cooldown" → Greedy

**Greedy insight:** The most frequent task determines the minimum time. Fill the most frequent task first, then fill gaps with other tasks.

```python
from collections import Counter

def leastInterval(tasks, n):
    freq = Counter(tasks)
    max_freq = max(freq.values())
    max_count = sum(1 for f in freq.values() if f == max_freq)
    
    # Formula: (max_freq - 1) * (n + 1) + max_count
    return max(len(tasks), (max_freq - 1) * (n + 1) + max_count)
```

**Step-by-step:**

```
tasks = ["A","A","A","B","B","B"], n = 2

freq = {A: 3, B: 3}
max_freq = 3
max_count = 2 (both A and B have freq 3)

Formula: (3 - 1) * (2 + 1) + 2 = 2 * 3 + 2 = 8

But len(tasks) = 6, so max(6, 8) = 8

Schedule: A B _ A B _ A B
          1 2 3 4 5 6 7 8

Wait, that's 8 slots but we only have 6 tasks. The _ are idle.

Actually: A B idle A B idle A B = 8 units
```

**Verification:** With cooldown 2, we need at least 8 units to schedule A,A,A,B,B,B. ✓

---

### Common Greedy Strategies

#### Strategy 1: Sort by End Time (Interval Problems)

```python
intervals.sort(key=lambda x: x[1])
```

**Use when:** Selecting maximum non-overlapping intervals, or minimum removals.

#### Strategy 2: Sort by Start Time

```python
intervals.sort(key=lambda x: x[0])
```

**Use when:** Merging intervals, or finding overlaps.

#### Strategy 3: Sort by Ratio (Fractional Knapsack)

```python
items.sort(key=lambda x: x.value / x.weight, reverse=True)
```

**Use when:** Maximizing value-to-weight ratio.

#### Strategy 4: Sort by Deadline

```python
tasks.sort(key=lambda x: x.deadline)
```

**Use when:** Scheduling with deadlines.

#### Strategy 5: Two Passes

```python
# Pass 1: Left to right
for i in range(n):
    # Process

# Pass 2: Right to left
for i in range(n-1, -1, -1):
    # Process
```

**Use when:** Need to consider both directions (e.g., Candy problem).

---

### Greedy vs DP: How to Choose

| Greedy | DP |
|--------|-----|
| Makes one choice per step | Considers all choices |
| Never reconsiders | Builds optimal from subproblems |
| Usually O(n log n) | Usually O(n²) or O(n×W) |
| Works when greedy choice property holds | Always works for optimization problems |
| Harder to prove correctness | Easier to prove correctness |

**When in doubt, use DP.** If DP is too slow, look for a greedy insight.

---

### Code Templates (4 Languages)

#### Python

```python
# Interval Greedy
def greedy(intervals):
    intervals.sort(key=lambda x: x[1])  # Sort by end
    count = 0
    last_end = float('-inf')
    for start, end in intervals:
        if start >= last_end:
            count += 1
            last_end = end
    return count

# Jump Game Greedy
def can_jump(nums):
    farthest = 0
    for i in range(len(nums)):
        if i > farthest:
            return False
        farthest = max(farthest, i + nums[i])
    return True
```

#### Java

```java
// Interval Greedy
int greedy(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]);
    int count = 0, lastEnd = Integer.MIN_VALUE;
    for (int[] interval : intervals) {
        if (interval[0] >= lastEnd) {
            count++;
            lastEnd = interval[1];
        }
    }
    return count;
}
```

#### C++

```cpp
// Interval Greedy
int greedy(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(), 
         [](const vector<int>& a, const vector<int>& b) { return a[1] < b[1]; });
    int count = 0, lastEnd = INT_MIN;
    for (auto& interval : intervals) {
        if (interval[0] >= lastEnd) {
            count++;
            lastEnd = interval[1];
        }
    }
    return count;
}
```

#### JavaScript

```javascript
// Interval Greedy
function greedy(intervals) {
    intervals.sort((a, b) => a[1] - b[1]);
    let count = 0, lastEnd = -Infinity;
    for (const [start, end] of intervals) {
        if (start >= lastEnd) {
            count++;
            lastEnd = end;
        }
    }
    return count;
}
```

---

### Common Mistakes

#### Mistake 1: Greedy Doesn't Always Work

```python
# WRONG: Greedy doesn't work for 0/1 Knapsack
# Greedy (by value/weight) gives wrong answer for:
# weights=[10, 20, 30], values=[60, 100, 120], capacity=50
# Greedy picks item 1 (ratio 6), then can't fit others: value=60
# Optimal: items 2+3: value=220

# RIGHT: Use DP for 0/1 Knapsack
```

#### Mistake 2: Sorting by Wrong Key

```python
# WRONG: sorting by start time for "maximum non-overlapping"
intervals.sort(key=lambda x: x[0])

# RIGHT: sort by end time
intervals.sort(key=lambda x: x[1])
```

#### Mistake 3: Not Proving Greedy Works

```python
# Before using Greedy, ask yourself:
# 1. Does the locally optimal choice lead to global optimum?
# 2. Can I prove it by contradiction or exchange argument?
# If unsure, use DP instead.
```

---

### Practice Problems

#### Easy-Medium

| # | Problem | Greedy Strategy |
|---|---------|-----------------|
| 1 | [55. Jump Game](https://leetcode.com/problems/jump-game/) | Track farthest |
| 2 | [45. Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Track farthest |
| 3 | [452. Minimum Number of Arrows](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Sort by end |
| 4 | [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Sort by end |
| 5 | [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Sort by start |
| 6 | [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Sort + heap |
| 7 | [134. Gas Station](https://leetcode.com/problems/gas-station/) | Track deficit |
| 8 | [763. Partition Labels](https://leetcode.com/problems/partition-labels/) | Track last occurrence |

#### Medium-Hard

| # | Problem | Greedy Strategy |
|---|---------|-----------------|
| 9 | [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Frequency analysis |
| 10 | [452. Minimum Number of Arrows](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Sort by end |
| 11 | [135. Candy](https://leetcode.com/problems/candy/) | Two passes |
| 12 | [406. Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/) | Sort + insert |
| 13 | [1029. Two City Scheduling](https://leetcode.com/problems/two-city-scheduling/) | Sort by difference |
| 14 | [1710. Maximum Units on a Truck](https://leetcode.com/problems/maximum-units-on-a-truck/) | Sort by value |
| 15 | [1833. Maximum Ice Cream Bars](https://leetcode.com/problems/maximum-ice-cream-bars/) | Sort + take |

---

### Interview Tips

1. **State the greedy choice.** "At each step, I choose the interval that ends earliest."

2. **Explain why it's optimal.** "Choosing the earliest ending interval maximizes room for future intervals."

3. **Sort first.** "I'll sort by [end time / start time / value / etc.] to enable the greedy choice."

4. **Prove correctness (briefly).** "Any optimal solution can be transformed to include this greedy choice without making it worse."

5. **If unsure, mention DP as alternative.** "If greedy doesn't work, we can use DP with state [X]."

6. **Edge cases:**
   - Empty input
   - Single element
   - All identical elements
   - Already sorted / reverse sorted

---

*Next: [13 — Union Find](13-Union-Find.md)*
