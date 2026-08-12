# 05 — Monotonic Stack

## The Pattern That Finds "Next Greater" in O(n)

---

### What It Is

A Monotonic Stack is a stack that maintains its elements in either strictly increasing or strictly decreasing order. It's used to efficiently find the **next greater element**, **next smaller element**, or the **nearest larger/smaller element** for each element in an array.

**The key insight:** When you process elements left-to-right and maintain a monotonic stack, each element is pushed and popped at most once, giving you O(n) time complexity for problems that would otherwise require O(n²).

---

### When to Use It

**The problem involves:**
- Finding the next greater/smaller element
- Finding the previous greater/smaller element
- Histogram problems (largest rectangle)
- Temperature/stock span problems
- Any "next" or "nearest" relationship in an array

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "next greater" | Next Greater Element |
| "next smaller" | Next Smaller Element |
| "previous greater" | Previous Greater Element |
| "nearest larger" | Daily Temperatures |
| "stock span" | Online Stock Span |
| "histogram" | Largest Rectangle in Histogram |
| "temperature" | Daily Temperatures |
| "warmer day" | Daily Temperatures |
| "trapping water" | Trapping Rain Water |
| "width of ramp" | Maximum Width Ramp |

**The Decision:**

```
Does each element need to find its "next" or "previous" greater/smaller?
    ↓
Yes
    ↓
MONOTONIC STACK
```

---

### Mental Model: The Waiting Line

Imagine a line of people waiting, each with a number on their shirt. You process them one by one from left to right.

```
Stack (decreasing):  [8, 5, 3]

New person arrives with number 6:
- 3 < 6, so 3's "next greater" is 6. Pop 3.
- 5 < 6, so 5's "next greater" is 6. Pop 5.
- 8 > 6, so 6 doesn't pop 8. Push 6.

Stack becomes: [8, 6]
```

The stack maintains the invariant: elements are in decreasing order from bottom to top. When a new element violates this, it means we've found the "next greater" for elements being popped.

---

### The Two Flavors

#### Decreasing Stack (for Next Greater Element)

Elements in the stack are in decreasing order. When a new element is larger, it pops smaller elements—it's their "next greater."

```python
stack = []
for i, num in enumerate(nums):
    while stack and nums[stack[-1]] < num:
        idx = stack.pop()
        result[idx] = num  # num is the next greater element for nums[idx]
    stack.append(i)
```

#### Increasing Stack (for Next Smaller Element)

Elements in the stack are in increasing order. When a new element is smaller, it pops larger elements—it's their "next smaller."

```python
stack = []
for i, num in enumerate(nums):
    while stack and nums[stack[-1]] > num:
        idx = stack.pop()
        result[idx] = num  # num is the next smaller element for nums[idx]
    stack.append(i)
```

---

### The 80% Template: Next Greater Element

```python
def next_greater(nums):
    n = len(nums)
    result = [-1] * n  # -1 means no next greater element
    stack = []  # stores indices
    
    for i in range(n):
        # Pop elements that are smaller than current
        while stack and nums[stack[-1]] < nums[i]:
            idx = stack.pop()
            result[idx] = nums[i]
        
        stack.append(i)
    
    return result
```

**What this does:**
- For each element, finds the next element to its right that is greater
- Elements remaining in the stack have no next greater element (result stays -1)

---

### Dry Run: Daily Temperatures

**Problem:** Given an array of temperatures, return an array where `answer[i]` is the number of days until a warmer temperature. If no warmer day exists, put 0.

**Input:** `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]`

**Recognition:** "next" + "warmer" → Monotonic Stack (decreasing)

```python
def dailyTemperatures(temps):
    n = len(temps)
    result = [0] * n
    stack = []  # stores indices
    
    for i in range(n):
        while stack and temps[stack[-1]] < temps[i]:
            idx = stack.pop()
            result[idx] = i - idx  # days until warmer
        
        stack.append(i)
    
    return result
```

**Step-by-step:**

```
temperatures = [73, 74, 75, 71, 69, 72, 76, 73]

i=0, temp=73:
  stack empty, push 0
  stack = [0]

i=1, temp=74:
  73 < 74 → pop 0, result[0] = 1 - 0 = 1
  stack empty, push 1
  stack = [1]

i=2, temp=75:
  74 < 75 → pop 1, result[1] = 2 - 1 = 1
  stack empty, push 2
  stack = [2]

i=3, temp=71:
  75 > 71, can't pop. push 3
  stack = [2, 3]

i=4, temp=69:
  71 > 69, can't pop. push 4
  stack = [2, 3, 4]

i=5, temp=72:
  69 < 72 → pop 4, result[4] = 5 - 4 = 1
  71 < 72 → pop 3, result[3] = 5 - 3 = 2
  75 > 72, can't pop. push 5
  stack = [2, 5]

i=6, temp=76:
  72 < 76 → pop 5, result[5] = 6 - 5 = 1
  75 < 76 → pop 2, result[2] = 6 - 2 = 4
  stack empty, push 6
  stack = [6]

i=7, temp=73:
  76 > 73, can't pop. push 7
  stack = [6, 7]

Remaining in stack: indices 6, 7 → result stays 0

result = [1, 1, 4, 2, 1, 1, 0, 0]
```

**Verification:**
- Day 0 (73): next warmer is day 1 (74) → 1 day ✓
- Day 1 (74): next warmer is day 2 (75) → 1 day ✓
- Day 2 (75): next warmer is day 6 (76) → 4 days ✓
- Day 3 (71): next warmer is day 5 (72) → 2 days ✓
- Day 4 (69): next warmer is day 5 (72) → 1 day ✓
- Day 5 (72): next warmer is day 6 (76) → 1 day ✓
- Day 6 (76): no warmer day → 0 ✓
- Day 7 (73): no warmer day → 0 ✓

---

### Dry Run: Largest Rectangle in Histogram

**Problem:** Given an array of heights representing a histogram, find the area of the largest rectangle.

**Input:** `heights = [2, 1, 5, 6, 2, 3]`

**Recognition:** "histogram" + "largest rectangle" → Monotonic Stack (increasing)

**Key insight:** For each bar, find how far left and right it can extend (until a shorter bar). The area is `height × width`.

```python
def largestRectangleArea(heights):
    stack = []  # stores indices
    max_area = 0
    heights.append(0)  # sentinel to flush remaining stack
    
    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            # Width: from the bar after the previous smaller bar to current bar
            width = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, height * width)
        
        stack.append(i)
    
    return max_area
```

**Step-by-step (simplified):**

```
heights = [2, 1, 5, 6, 2, 3]

i=0, h=2: stack=[], push 0. stack=[0]
i=1, h=1: 2>1, pop 0. height=2, width=1. area=2. stack=[], push 1. stack=[1]
i=2, h=5: push 2. stack=[1,2]
i=3, h=6: push 3. stack=[1,2,3]
i=4, h=2: 6>2, pop 3. height=6, width=1. area=6.
           5>2, pop 2. height=5, width=2. area=10.
           1<2, stop. push 4. stack=[1,4]
i=5, h=3: push 5. stack=[1,4,5]
i=6, h=0: 3>0, pop 5. height=3, width=1. area=10.
           2>0, pop 4. height=2, width=4. area=10.
           1>0, pop 1. height=1, width=6. area=10.

Return 10. The largest rectangle is height 5, width 2 (indices 2-3).
```

---

### Common Patterns

#### Pattern 1: Next Greater Element to the Right

```python
stack = []
for i in range(n):
    while stack and nums[stack[-1]] < nums[i]:
        result[stack.pop()] = nums[i]
    stack.append(i)
```

#### Pattern 2: Next Greater Element to the Left

```python
stack = []
for i in range(n):
    while stack and nums[stack[-1]] < nums[i]:
        stack.pop()
    result[i] = nums[stack[-1]] if stack else -1
    stack.append(i)
```

#### Pattern 3: Next Smaller Element to the Right

```python
stack = []
for i in range(n):
    while stack and nums[stack[-1]] > nums[i]:
        result[stack.pop()] = nums[i]
    stack.append(i)
```

#### Pattern 4: Previous Smaller Element

```python
stack = []
for i in range(n):
    while stack and nums[stack[-1]] >= nums[i]:
        stack.pop()
    result[i] = stack[-1] if stack else -1
    stack.append(i)
```

---

### Code Templates (4 Languages)

#### Python

```python
# Next Greater Element
def next_greater(nums):
    n = len(nums)
    result = [-1] * n
    stack = []
    for i in range(n):
        while stack and nums[stack[-1]] < nums[i]:
            result[stack.pop()] = nums[i]
        stack.append(i)
    return result

# Largest Rectangle in Histogram
def largest_rectangle(heights):
    stack = []
    max_area = 0
    heights.append(0)
    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, height * width)
        stack.append(i)
    return max_area
```

#### Java

```java
// Next Greater Element
int[] nextGreater(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

#### C++

```cpp
// Next Greater Element
vector<int> nextGreater(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            result[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}
```

#### JavaScript

```javascript
// Next Greater Element
function nextGreater(nums) {
    const n = nums.length;
    const result = new Array(n).fill(-1);
    const stack = [];
    for (let i = 0; i < n; i++) {
        while (stack.length && nums[stack[stack.length - 1]] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
```

---

### Common Mistakes

#### Mistake 1: Storing Values Instead of Indices

```python
# WRONG: can't compute width for histogram problems
stack.append(nums[i])

# RIGHT: store indices
stack.append(i)
```

#### Mistake 2: Wrong Comparison Direction

```python
# For Next GREATER: use < (pop when current is greater)
while stack and nums[stack[-1]] < nums[i]:

# For Next SMALLER: use > (pop when current is smaller)
while stack and nums[stack[-1]] > nums[i]:
```

#### Mistake 3: Forgetting Sentinel Value

```python
# In histogram problems, add a 0 at the end to flush the stack
heights.append(0)
```

---

### Complexity Analysis

| Aspect | Complexity | Why |
|--------|-----------|-----|
| Time | O(n) | Each element pushed and popped at most once |
| Space | O(n) | Stack can hold up to n elements |

---

### Practice Problems

#### Easy-Medium

| # | Problem | Key |
|---|---------|-----|
| 1 | [496. Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Basic next greater |
| 2 | [503. Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Circular array |
| 3 | [739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Classic application |
| 4 | [1475. Final Prices With a Special Discount](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/) | Next smaller or equal |

#### Medium-Hard

| # | Problem | Key |
|---|---------|-----|
| 5 | [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Histogram |
| 6 | [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Two pointers or stack |
| 7 | [901. Online Stock Span](https://leetcode.com/problems/online-stock-span/) | Monotonic stack + span |
| 8 | [316. Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/) | Lexicographically smallest |
| 9 | [402. Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | Greedy + stack |
| 10 | [321. Create Maximum Number](https://leetcode.com/problems/create-maximum-number/) | Monotonic stack + merge |

---

### Interview Tips

1. **State the invariant.** "The stack maintains elements in decreasing order."

2. **Explain the popping logic.** "When I encounter a larger element, it's the answer for all smaller elements in the stack."

3. **Mention the time complexity.** "Each element is pushed and popped at most once, so it's O(n)."

4. **Draw a small example.** Walk through 3-4 elements to show how the stack evolves.

5. **Edge cases:**
   - All elements the same
   - Strictly increasing or decreasing
   - Single element
   - Circular array (for NGE II)

---

*Next: [06 — Heap / Priority Queue](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Heap-priority-queue.md)*
