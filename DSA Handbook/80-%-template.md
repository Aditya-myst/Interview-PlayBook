# 80% Templates Cheatsheet

## The Reusable Code Skeletons for Each Pattern

---

### 1. Sliding Window

```python
def sliding_window(nums, k):
    left = 0
    state = {}  # or counter, sum, set
    answer = 0  # or float('inf') if minimizing
    
    for right in range(len(nums)):
        # Expand: add nums[right]
        state[nums[right]] = state.get(nums[right], 0) + 1
        
        # Shrink while invalid
        while is_invalid(state, k):
            state[nums[left]] -= 1
            if state[nums[left]] == 0:
                del state[nums[left]]
            left += 1
        
        # Update answer
        answer = max(answer, right - left + 1)
    
    return answer
```

---

### 2. Two Pointers

```python
# Opposite Ends (sorted array)
def two_pointer(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        curr = nums[left] + nums[right]
        if curr == target:
            return [left, right]
        elif curr < target:
            left += 1
        else:
            right -= 1
    return []

# Same Direction (remove duplicates, partition)
def slow_fast(nums):
    slow = 0
    for fast in range(len(nums)):
        if should_keep(nums[fast]):
            nums[slow] = nums[fast]
            slow += 1
    return slow
```

---

### 3. Binary Search

```python
# Classic
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

# On Answer (minimize)
def bs_min_answer(nums, k):
    def feasible(mid):
        # return True if mid is valid
        pass
    
    lo, hi = min_possible, max_possible
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if feasible(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo

# On Answer (maximize)
def bs_max_answer(nums, k):
    def feasible(mid):
        pass
    
    lo, hi = min_possible, max_possible
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2  # Round up!
        if feasible(mid):
            lo = mid
        else:
            hi = mid - 1
    return lo
```

---

### 4. Prefix Sum + HashMap

```python
def subarray_sum(nums, k):
    count = 0
    prefix_sum = 0
    seen = {0: 1}  # IMPORTANT: initialize with 0:1
    
    for num in nums:
        prefix_sum += num
        if prefix_sum - k in seen:
            count += seen[prefix_sum - k]
        seen[prefix_sum] = seen.get(prefix_sum, 0) + 1
    
    return count
```

---

### 5. Monotonic Stack

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

# Next Smaller Element
def next_smaller(nums):
    n = len(nums)
    result = [-1] * n
    stack = []
    
    for i in range(n):
        while stack and nums[stack[-1]] > nums[i]:
            result[stack.pop()] = nums[i]
        stack.append(i)
    
    return result
```

---

### 6. Heap / Priority Queue

```python
import heapq

# Kth Largest (min-heap of size K)
def kth_largest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]

# Top K Frequent
def top_k_frequent(nums, k):
    from collections import Counter
    count = Counter(nums)
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)
    return [num for freq, num in heap]

# Max-Heap (Python trick: negate values)
heap = []
heapq.heappush(heap, -val)
max_val = -heapq.heappop(heap)
```

---

### 7. Linked List

```python
# Reverse
def reverse(head):
    prev, curr = None, head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev, curr = curr, nxt
    return prev

# Find Middle
def middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow

# Detect Cycle
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False

# Merge Two Sorted
def merge(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next
```

---

### 8. Trees

```python
# Recursive Template
def solve(root):
    if not root:
        return base_case
    left = solve(root.left)
    right = solve(root.right)
    return combine(root.val, left, right)

# BFS Level Order
from collections import deque
def level_order(root):
    if not root:
        return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    return result

# BST Validation
def is_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root:
        return True
    if root.val <= lo or root.val >= hi:
        return False
    return is_bst(root.left, lo, root.val) and is_bst(root.right, root.val, hi)
```

---

### 9. DFS & BFS

```python
# DFS (Graph)
def dfs(node, visited):
    if node in visited:
        return
    visited.add(node)
    for neighbor in graph[node]:
        dfs(neighbor, visited)

# BFS (Graph)
from collections import deque
def bfs(start):
    queue, visited = deque([start]), {start}
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

# Grid DFS
def dfs_grid(grid, i, j, visited):
    if i < 0 or i >= len(grid) or j < 0 or j >= len(grid[0]):
        return
    if (i, j) in visited or grid[i][j] == 0:
        return
    visited.add((i, j))
    for di, dj in [(-1,0),(1,0),(0,-1),(0,1)]:
        dfs_grid(grid, i+di, j+dj, visited)
```

---

### 10. Backtracking

```python
def backtrack(candidates, current, result, start):
    if is_solution(current):
        result.append(current[:])  # Copy!
        return
    
    for i in range(start, len(candidates)):
        if not is_valid(candidates[i], current):
            continue  # Pruning
        
        current.append(candidates[i])  # Choose
        backtrack(candidates, current, result, i + 1)  # Explore
        current.pop()  # Unchoose
```

---

### 11. Dynamic Programming

```python
# 1D DP
def dp_1d(nums):
    n = len(nums)
    dp = [0] * n
    dp[0] = base_case
    for i in range(1, n):
        dp[i] = transition(dp[i-1], ...)
    return dp[n-1]

# 2D DP
def dp_2d(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    # Base cases: dp[0][j] and dp[i][0]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            dp[i][j] = transition(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[m][n]

# Top-Down (Memoization)
from functools import lru_cache
@lru_cache(maxsize=None)
def solve(i):
    if i >= n:
        return base_case
    return transition(solve(i+1), ...)
```

---

### 12. Greedy

```python
# Interval Greedy (sort by end time)
def greedy_intervals(intervals):
    intervals.sort(key=lambda x: x[1])
    count, last_end = 0, float('-inf')
    for start, end in intervals:
        if start >= last_end:
            count += 1
            last_end = end
    return count

# Jump Game
def can_jump(nums):
    farthest = 0
    for i in range(len(nums)):
        if i > farthest:
            return False
        farthest = max(farthest, i + nums[i])
    return True
```

---

### 13. Union Find

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        return True
```

---

### 14. Topological Sort

```python
from collections import deque, defaultdict

def topo_sort(n, edges):
    graph = defaultdict(list)
    indegree = [0] * n
    for u, v in edges:
        graph[u].append(v)
        indegree[v] += 1
    
    queue = deque([i for i in range(n) if indegree[i] == 0])
    order = []
    
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    
    return order if len(order) == n else []  # Empty if cycle
```

---

### 15. Shortest Path (Dijkstra)

```python
import heapq

def dijkstra(graph, start):
    dist = {start: 0}
    heap = [(0, start)]
    
    while heap:
        d, node = heapq.heappop(heap)
        if d > dist.get(node, float('inf')):
            continue
        for neighbor, weight in graph[node]:
            new_dist = d + weight
            if new_dist < dist.get(neighbor, float('inf')):
                dist[neighbor] = new_dist
                heapq.heappush(heap, (new_dist, neighbor))
    
    return dist
```

---

### 16. Bit Manipulation

```python
# Single Number (XOR)
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

---

### Quick Reference

| Pattern | Key Operation | Time |
|---------|---------------|------|
| Sliding Window | expand/shrink | O(n) |
| Two Pointers | move based on condition | O(n) |
| Binary Search | halve search space | O(log n) |
| Prefix Sum | cumulative sum + lookup | O(n) |
| Monotonic Stack | pop when violated | O(n) |
| Heap | push/pop min/max | O(n log k) |
| Linked List | pointer manipulation | O(n) |
| Trees | recursion | O(n) |
| DFS/BFS | visit all nodes | O(V+E) |
| Backtracking | choose/explore/unchoose | O(2^n) |
| DP | build from subproblems | O(n²) or O(n×W) |
| Greedy | local optimal choice | O(n log n) |
| Union Find | find/union with compression | O(α(n)) |
| Topological Sort | process by indegree | O(V+E) |
| Dijkstra | min-heap traversal | O((V+E) log V) |
| Bit Manipulation | bitwise operations | O(n) or O(1) |
