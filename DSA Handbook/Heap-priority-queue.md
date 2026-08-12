# 06 — Heap / Priority Queue

## The Pattern That Finds Top K in O(n log k)

---

### What It Is

A Heap (or Priority Queue) is a data structure that efficiently maintains the maximum (max-heap) or minimum (min-heap) element at the top. It supports:
- **Insert:** O(log n)
- **Get min/max:** O(1)
- **Remove min/max:** O(log n)

**The key insight:** When you need the "top K" or "smallest K" or "most frequent K" elements, a heap of size K is more efficient than sorting the entire array.

---

### When to Use It

**The problem involves:**
- Finding the Kth largest/smallest element
- Finding the top K most frequent elements
- Merging K sorted lists
- Finding the median in a stream
- Scheduling or priority-based processing
- Any "top K" or "K closest" requirement

**Trigger Words:**

| Trigger Word/Phrase | Example |
|---------------------|---------|
| "Kth largest" | Kth Largest Element |
| "Kth smallest" | Kth Smallest Element |
| "top K" | Top K Frequent Elements |
| "K closest" | K Closest Points to Origin |
| "merge K" | Merge K Sorted Lists |
| "median" | Find Median from Data Stream |
| "priority" | Task Scheduler |
| "most frequent" | Top K Frequent Words |
| "minimum cost" | Minimum Cost to Connect Sticks |
| "sliding window maximum" | Sliding Window Maximum |

**The Decision:**

```
Do you need the Kth element or top K elements?
    ↓
Yes
    ↓
HEAP
```

---

### Mental Model: The VIP Line

Think of a heap like a VIP line at a club:

```
Min-Heap (smallest at top):
         1
        / \
       3   5
      / \ / \
     7  4 8  6

Max-Heap (largest at top):
         8
        / \
       7   6
      / \ / \
     3  4 5  1
```

The bouncer always lets in the highest priority person (top of heap) in O(1). New people join and get sorted in O(log n).

---

### When to Use Min-Heap vs Max-Heap

| Scenario | Heap Type | Why |
|----------|-----------|-----|
| Find Kth LARGEST | Min-heap of size K | The Kth largest is at the top when heap has K elements |
| Find Kth SMALLEST | Max-heap of size K | The Kth smallest is at the top when heap has K elements |
| Merge K sorted lists | Min-heap | Always pick the smallest among the K list heads |
| Find median | Two heaps (max + min) | Max-heap for lower half, min-heap for upper half |

**Counter-intuitive insight:** To find the Kth LARGEST, use a MIN-heap. Why? Because as you process elements, the min-heap keeps the K largest elements, with the smallest of those K at the top—which is exactly the Kth largest.

---

### The 80% Template: Top K Elements

```python
import heapq

def top_k_frequent(nums, k):
    # Count frequencies
    count = {}
    for num in nums:
        count[num] = count.get(num, 0) + 1
    
    # Use min-heap of size k
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)  # Remove smallest frequency
    
    # Extract results
    return [num for freq, num in heap]
```

**Why this works:**
- Push each element with its frequency
- When heap size exceeds K, pop the smallest (lowest frequency)
- At the end, only the K most frequent remain

---

### The 80% Template: Kth Largest Element

```python
import heapq

def findKthLargest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]  # Kth largest is at the top of min-heap
```

**Why min-heap for Kth largest:**
- We maintain exactly K elements in the heap
- The smallest of those K is the Kth largest overall
- It's at the top of the min-heap

---

### Dry Run: Top K Frequent Elements

**Problem:** Given an integer array and integer k, return the k most frequent elements.

**Input:** `nums = [1, 1, 1, 2, 2, 3]`, `k = 2`

```python
import heapq
from collections import Counter

def topKFrequent(nums, k):
    count = Counter(nums)  # {1:3, 2:2, 3:1}
    heap = []
    
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
    
    return [num for freq, num in heap]
```

**Step-by-step:**

```
nums = [1, 1, 1, 2, 2, 3]    k = 2
count = {1: 3, 2: 2, 3: 1}

Process (num, freq):

(1, 3): push → heap = [(3, 1)]
        size=1 ≤ k=2, don't pop

(2, 2): push → heap = [(2, 2), (3, 1)]
        size=2 ≤ k=2, don't pop

(3, 1): push → heap = [(1, 3), (3, 1), (2, 2)]
        size=3 > k=2, pop smallest → (1, 3) removed
        heap = [(2, 2), (3, 1)]

Result: [2, 1] (the two most frequent elements)
```

**Note:** In Python's heapq, the heap order is maintained internally. The output order may vary.

---

### Dry Run: Merge K Sorted Lists

**Problem:** Merge K sorted linked lists into one sorted list.

**Input:** `lists = [[1,4,5], [1,3,4], [2,6]]`

**Recognition:** "merge K sorted" → Min-heap

```python
import heapq

def mergeKLists(lists):
    heap = []
    
    # Initialize: push first element from each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))
    
    result = []
    while heap:
        val, list_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        
        # Push next element from the same list
        if elem_idx + 1 < len(lists[list_idx]):
            next_val = lists[list_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, list_idx, elem_idx + 1))
    
    return result
```

**Step-by-step:**

```
lists = [[1,4,5], [1,3,4], [2,6]]

Initialize:
  Push (1, 0, 0) from list 0
  Push (1, 1, 0) from list 1
  Push (2, 2, 0) from list 2
  heap = [(1,0,0), (1,1,0), (2,2,0)]

Step 1: Pop (1, 0, 0) → result = [1]
  Push next from list 0: (4, 0, 1)
  heap = [(1,1,0), (4,0,1), (2,2,0)]

Step 2: Pop (1, 1, 0) → result = [1, 1]
  Push next from list 1: (3, 1, 1)
  heap = [(2,2,0), (4,0,1), (3,1,1)]

Step 3: Pop (2, 2, 0) → result = [1, 1, 2]
  Push next from list 2: (6, 2, 1)
  heap = [(3,1,1), (4,0,1), (6,2,1)]

Step 4: Pop (3, 1, 1) → result = [1, 1, 2, 3]
  Push next from list 1: (4, 1, 2)
  heap = [(4,0,1), (4,1,2), (6,2,1)]

Step 5: Pop (4, 0, 1) → result = [1, 1, 2, 3, 4]
  Push next from list 0: (5, 0, 2)
  heap = [(4,1,2), (5,0,2), (6,2,1)]

Step 6: Pop (4, 1, 2) → result = [1, 1, 2, 3, 4, 4]
  List 1 exhausted, nothing to push
  heap = [(5,0,2), (6,2,1)]

Step 7: Pop (5, 0, 2) → result = [1, 1, 2, 3, 4, 4, 5]
  List 0 exhausted
  heap = [(6,2,1)]

Step 8: Pop (6, 2, 1) → result = [1, 1, 2, 3, 4, 4, 5, 6]
  List 2 exhausted
  heap = []

Result: [1, 1, 2, 3, 4, 4, 5, 6]
```

---

### The Median Pattern: Two Heaps

For finding the median in a stream of numbers, use two heaps:
- **Max-heap** for the lower half
- **Min-heap** for the upper half

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.lo = []  # max-heap (inverted for Python's min-heap)
        self.hi = []  # min-heap
    
    def addNum(self, num):
        heapq.heappush(self.lo, -num)  # push to max-heap
        
        # Ensure max of lo ≤ min of hi
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        
        # Balance sizes (lo can have at most 1 more than hi)
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))
    
    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

**Visual:**

```
Stream: [5, 15, 1, 3]

Add 5:  lo=[5] hi=[]           median = 5
Add 15: lo=[5] hi=[15]         median = (5+15)/2 = 10
Add 1:  lo=[5,1] hi=[15]       median = 5
Add 3:  lo=[3,1] hi=[5,15]     median = (3+5)/2 = 4
```

---

### Code Templates (4 Languages)

#### Python

```python
import heapq

# Kth Largest
def kth_largest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]

# Top K Frequent
def top_k(nums, k):
    from collections import Counter
    count = Counter(nums)
    return [num for num, freq in heapq.nlargest(k, count.items(), key=lambda x: x[1])]

# Min-Heap
heap = []
heapq.heappush(heap, val)
smallest = heapq.heappop(heap)

# Max-Heap (invert values)
heap = []
heapq.heappush(heap, -val)
largest = -heapq.heappop(heap)
```

#### Java

```java
// Kth Largest
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();
        }
    }
    return minHeap.peek();
}

// Max-Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

#### C++

```cpp
// Kth Largest
int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    for (int num : nums) {
        minHeap.push(num);
        if (minHeap.size() > k) {
            minHeap.pop();
        }
    }
    return minHeap.top();
}

// Max-Heap (default)
priority_queue<int> maxHeap;

// Min-Heap
priority_queue<int, vector<int>, greater<int>> minHeap;
```

#### JavaScript

```javascript
// Using a simple heap implementation or library
// JS doesn't have built-in heap, but you can implement one

class MinHeap {
    constructor() { this.heap = []; }
    push(val) { /* heapify up */ }
    pop() { /* heapify down */ }
    peek() { return this.heap[0]; }
    get size() { return this.heap.length; }
}

// Kth Largest
function findKthLargest(nums, k) {
    const heap = new MinHeap();
    for (const num of nums) {
        heap.push(num);
        if (heap.size > k) heap.pop();
    }
    return heap.peek();
}
```

---

### Common Mistakes

#### Mistake 1: Using Max-Heap for Kth Largest

```python
# WRONG: max-heap keeps the largest, not the Kth largest
heap = [-num for num in nums]  # This just sorts everything

# RIGHT: min-heap of size K
heap = []
for num in nums:
    heapq.heappush(heap, num)
    if len(heap) > k:
        heapq.heappop(heap)
```

#### Mistake 2: Python's Heap is Min-Heap Only

```python
# WRONG: trying to create max-heap directly
heapq.heapify([3, 1, 4])  # This gives [1, 3, 4], not [4, 3, 1]

# RIGHT: negate values for max-heap
heap = [-x for x in values]
heapq.heapify(heap)
max_val = -heapq.heappop(heap)
```

#### Mistake 3: Comparing Tuples Incorrectly

```python
# WRONG: when pushing (frequency, value), Python compares by frequency first
# If two elements have same frequency, it compares by value
# This might not be what you want

# RIGHT: if you want to sort by frequency only, use (frequency, index, value)
heapq.heappush(heap, (freq, i, num))  # i breaks ties
```

---

### Complexity Analysis

| Operation | Heap | Sorted Array | Unsorted Array |
|-----------|------|--------------|----------------|
| Insert | O(log n) | O(n) | O(1) |
| Find min/max | O(1) | O(1) | O(n) |
| Delete min/max | O(log n) | O(n) | O(n) |
| Build | O(n) | O(n log n) | O(n) |
| Kth largest | O(n log k) | O(n log n) | O(n log n) |

**For Top K problems:** Heap approach is O(n log k), which is better than O(n log n) sorting when k << n.

---

### Practice Problems

#### Easy-Medium

| # | Problem | Key |
|---|---------|-----|
| 1 | [215. Kth Largest Element](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Min-heap of size K |
| 2 | [703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Streaming Kth largest |
| 3 | [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Heap + frequency |
| 4 | [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Heap + distance |
| 5 | [1046. Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Max-heap simulation |
| 6 | [692. Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/) | Heap + frequency + lexicographic |

#### Medium-Hard

| # | Problem | Key |
|---|---------|-----|
| 7 | [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Min-heap across K lists |
| 8 | [295. Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Two heaps |
| 9 | [767. Reorganize String](https://leetcode.com/problems/reorganize-string/) | Greedy + max-heap |
| 10 | [355. Design Twitter](https://leetcode.com/problems/design-twitter/) | Merge feeds with heap |
| 11 | [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Greedy + heap |
| 12 | [1642. Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/) | Greedy + min-heap |
| 13 | [778. Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Dijkstra with heap |

---

### Interview Tips

1. **Explain why a heap is appropriate.** "I need the Kth largest, so a min-heap of size K keeps track of the K largest elements efficiently."

2. **Clarify min vs max heap.** "Python's heapq is a min-heap. For max-heap, I negate the values."

3. **Mention the time complexity.** "This is O(n log k) because I process n elements, each taking O(log k) heap operations."

4. **For the median problem, explain the two-heap approach.** "The max-heap holds the lower half, the min-heap holds the upper half. The median is at the top of one or both heaps."

5. **Edge cases:**
   - K = 1 (just find max/min)
   - K = n (sort the whole array)
   - All elements the same
   - Stream is empty

---

*Next: [07 — Linked Lists](https://github.com/Aditya-myst/Interview-PlayBook/blob/main/DSA%20Handbook/Linked-Lists.md)*
