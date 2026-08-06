# 09 — DFS and BFS

## The Patterns That Explore Every Node and Edge

---

### What It Is

DFS (Depth-First Search) explores as far as possible along each branch before backtracking. BFS (Breadth-First Search) explores all neighbors at the current depth before moving deeper.

They're the fundamental traversal techniques for graphs and trees, and they form the backbone of many other patterns (backtracking, topological sort, shortest path, etc.).

**The key insight:** Use DFS when you need to explore all possibilities or paths. Use BFS when you need the shortest path or level-by-level processing.

---

### When to Use It

**Trigger Words:**

| Trigger Word/Phrase | Use DFS | Use BFS |
|---------------------|---------|---------|
| "all paths" | ✓ | |
| "find if path exists" | ✓ | ✓ |
| "shortest path" (unweighted) | | ✓ |
| "level by level" | | ✓ |
| "connected components" | ✓ | |
| "islands" | ✓ | ✓ |
| "flood fill" | ✓ | ✓ |
| "maze" | ✓ | ✓ |
| "word ladder" | | ✓ |
| "clone graph" | ✓ | ✓ |
| "number of provinces" | ✓ | |
| "rotting oranges" | | ✓ |

**The Decision:**

```
Graph/Tree problem?
    ↓
Need shortest path (unweighted)?
    ↓
Yes → BFS

Need to explore all possibilities?
    ↓
Yes → DFS

Need level-by-level processing?
    ↓
Yes → BFS

Need connected components?
    ↓
Yes → DFS (or Union Find)
```

---

### Mental Model

**DFS = Go deep, then backtrack (like exploring a maze by always turning left)**

```
        1
       / \
      2   5
     / \
    3   4

DFS: 1 → 2 → 3 → backtrack → 4 → backtrack → backtrack → 5
```

**BFS = Explore layer by layer (like ripples in a pond)**

```
        1
       / \
      2   5
     / \
    3   4

BFS: 1 → 2, 5 → 3, 4
```

---

### The 80% Template: DFS (Recursive)

```python
def dfs(node, visited):
    if node in visited:
        return
    
    visited.add(node)
    
    # Process node here
    
    for neighbor in get_neighbors(node):
        dfs(neighbor, visited)
```

### The 80% Template: DFS (Iterative)

```python
def dfs(start):
    stack = [start]
    visited = set()
    
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        
        visited.add(node)
        
        # Process node here
        
        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                stack.append(neighbor)
```

### The 80% Template: BFS

```python
from collections import deque

def bfs(start):
    queue = deque([start])
    visited = {start}
    
    while queue:
        node = queue.popleft()
        
        # Process node here
        
        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

---

### DFS on Grids

Many problems present a 2D grid as a graph. Each cell is a node, and adjacent cells are neighbors.

```python
def dfs_grid(grid, row, col, visited):
    # Boundary check
    if (row < 0 or row >= len(grid) or 
        col < 0 or col >= len(grid[0]) or
        (row, col) in visited or
        grid[row][col] == 0):  # or whatever condition
        return
    
    visited.add((row, col))
    
    # Process cell
    
    # Visit 4 neighbors (up, down, left, right)
    for dr, dc in [(-1,0), (1,0), (0,-1), (0,1)]:
        dfs_grid(grid, row + dr, col + dc, visited)
```

---

### BFS on Grids

```python
from collections import deque

def bfs_grid(grid, start_row, start_col):
    queue = deque([(start_row, start_col)])
    visited = {(start_row, start_col)}
    distance = {(start_row, start_col): 0}
    
    while queue:
        row, col = queue.popleft()
        
        for dr, dc in [(-1,0), (1,0), (0,-1), (0,1)]:
            nr, nc = row + dr, col + dc
            if (0 <= nr < len(grid) and 
                0 <= nc < len(grid[0]) and
                (nr, nc) not in visited and
                grid[nr][nc] != 0):
                visited.add((nr, nc))
                distance[(nr, nc)] = distance[(row, col)] + 1
                queue.append((nr, nc))
    
    return distance
```

---

### Dry Run: Number of Islands

**Problem:** Given a 2D grid of '1's (land) and '0's (water), count the number of islands.

**Input:**
```
grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
```

**Recognition:** "number of" + "islands" + "grid" → DFS/BFS to count connected components

```python
def numIslands(grid):
    if not grid:
        return 0
    
    count = 0
    visited = set()
    
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1' and (i, j) not in visited:
                dfs(grid, i, j, visited)
                count += 1
    
    return count

def dfs(grid, row, col, visited):
    if (row < 0 or row >= len(grid) or 
        col < 0 or col >= len(grid[0]) or
        (row, col) in visited or
        grid[row][col] == '0'):
        return
    
    visited.add((row, col))
    
    for dr, dc in [(-1,0), (1,0), (0,-1), (0,1)]:
        dfs(grid, row + dr, col + dc, visited)
```

**Step-by-step:**

```
Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Scan cell (0,0): '1' and not visited
  → DFS from (0,0): visits (0,0), (0,1), (1,0), (1,1)
  → Island 1 found, count = 1

Scan cell (0,1): '1' but already visited → skip
Scan cell (0,2): '0' → skip
...
Scan cell (2,2): '1' and not visited
  → DFS from (2,2): visits (2,2)
  → Island 2 found, count = 2

Scan cell (3,3): '1' and not visited
  → DFS from (3,3): visits (3,3), (3,4)
  → Island 3 found, count = 3

Return 3
```

---

### Dry Run: Rotting Oranges (BFS)

**Problem:** Every minute, rotten oranges rot adjacent fresh oranges. Return the minimum minutes until no fresh orange remains. Return -1 if impossible.

**Input:**
```
grid = [
  [2,1,1],
  [1,1,0],
  [0,1,1]
]
```
(0=empty, 1=fresh, 2=rotten)

**Recognition:** "minimum minutes" + "spread" + "grid" → Multi-source BFS

```python
from collections import deque

def orangesRotting(grid):
    queue = deque()
    fresh = 0
    m, n = len(grid), len(grid[0])
    
    # Find all rotten oranges (multiple sources)
    for i in range(m):
        for j in range(n):
            if grid[i][j] == 2:
                queue.append((i, j, 0))  # (row, col, minute)
            elif grid[i][j] == 1:
                fresh += 1
    
    if fresh == 0:
        return 0
    
    max_minute = 0
    
    while queue:
        row, col, minute = queue.popleft()
        max_minute = max(max_minute, minute)
        
        for dr, dc in [(-1,0), (1,0), (0,-1), (0,1)]:
            nr, nc = row + dr, col + dc
            if (0 <= nr < m and 0 <= nc < n and grid[nr][nc] == 1):
                grid[nr][nc] = 2  # rot it
                fresh -= 1
                queue.append((nr, nc, minute + 1))
    
    return max_minute if fresh == 0 else -1
```

**Step-by-step:**

```
Initial grid:
  2 1 1
  1 1 0
  0 1 1

Queue: [(0,0,0)], fresh=6

Minute 0: Process (0,0)
  Rot (0,1) and (1,0)
  Queue: [(0,1,1), (1,0,1)], fresh=4

Minute 1: Process (0,1)
  Rot (0,2) and (1,1)
  Queue: [(1,0,1), (0,2,2), (1,1,2)], fresh=2

Minute 1: Process (1,0)
  (0,0) already rotten, (1,1) already rotten
  Queue: [(0,2,2), (1,1,2)], fresh=2

Minute 2: Process (0,2)
  (0,1) already rotten, no other neighbors
  Queue: [(1,1,2)], fresh=2

Minute 2: Process (1,1)
  Rot (2,1)
  Queue: [(2,1,3)], fresh=1

Minute 3: Process (2,1)
  Rot (2,2)
  Queue: [(2,2,4)], fresh=0

Minute 4: Process (2,2)
  (2,1) already rotten
  Queue: [], fresh=0

fresh == 0, return 4
```

**Verification:** After 4 minutes, all oranges are rotten. No fresh oranges remain. ✓

---

### The "Visited" Pattern

**Why track visited?**
Without it, you'd revisit nodes, causing infinite loops in graphs with cycles.

```python
# With visited set (correct)
visited = set()
def dfs(node):
    if node in visited:
        return
    visited.add(node)
    for neighbor in graph[node]:
        dfs(neighbor)

# Without visited (infinite loop if cycle exists)
def dfs(node):  # WRONG for graphs with cycles
    for neighbor in graph[node]:
        dfs(neighbor)
```

**Grid optimization:** Instead of a separate visited set, modify the grid in-place (e.g., mark visited cells as '0' or '#').

```python
def dfs(grid, i, j):
    if i < 0 or i >= len(grid) or j < 0 or j >= len(grid[0]) or grid[i][j] != '1':
        return
    grid[i][j] = '#'  # Mark as visited
    for di, dj in [(-1,0), (1,0), (0,-1), (0,1)]:
        dfs(grid, i + di, j + dj)
```

---

### DFS vs BFS: When to Choose Which

| Use DFS When | Use BFS When |
|--------------|--------------|
| Finding all paths | Finding shortest path |
| Checking if path exists | Level-order traversal |
| Topological sort | Minimum steps/layers |
| Backtracking | Multi-source spread |
| Memory is limited (deep trees) | Width is limited |
| Solving mazes | Solving puzzles with fewest moves |

**Memory consideration:**
- DFS uses O(h) space where h = depth (can be O(n) in worst case)
- BFS uses O(w) space where w = max width (can be O(n) in worst case)

For very wide trees, DFS uses less memory. For very deep trees, BFS uses less memory.

---

### Code Templates (4 Languages)

#### Python

```python
# DFS Recursive
def dfs(node, visited):
    if node in visited:
        return
    visited.add(node)
    for neighbor in graph[node]:
        dfs(neighbor, visited)

# DFS Iterative
def dfs_iterative(start):
    stack, visited = [start], set()
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)

# BFS
from collections import deque
def bfs(start):
    queue, visited = deque([start]), {start}
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

#### Java

```java
// DFS Recursive
void dfs(int node, Set<Integer> visited) {
    if (visited.contains(node)) return;
    visited.add(node);
    for (int neighbor : graph[node]) {
        dfs(neighbor, visited);
    }
}

// BFS
void bfs(int start) {
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> visited = new HashSet<>();
    queue.offer(start);
    visited.add(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int neighbor : graph[node]) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
}
```

#### C++

```cpp
// DFS Recursive
void dfs(int node, unordered_set<int>& visited) {
    if (visited.count(node)) return;
    visited.insert(node);
    for (int neighbor : graph[node]) {
        dfs(neighbor, visited);
    }
}

// BFS
void bfs(int start) {
    queue<int> q;
    unordered_set<int> visited;
    q.push(start);
    visited.insert(start);
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int neighbor : graph[node]) {
            if (!visited.count(neighbor)) {
                visited.insert(neighbor);
                q.push(neighbor);
            }
        }
    }
}
```

#### JavaScript

```javascript
// DFS Recursive
function dfs(node, visited) {
    if (visited.has(node)) return;
    visited.add(node);
    for (const neighbor of graph[node]) {
        dfs(neighbor, visited);
    }
}

// BFS
function bfs(start) {
    const queue = [start];
    const visited = new Set([start]);
    while (queue.length) {
        const node = queue.shift();
        for (const neighbor of graph[node]) {
            if (!visited.has(neighbor)) {
                visited.add(neighbor);
                queue.push(neighbor);
            }
        }
    }
}
```

---

### Common Mistakes

#### Mistake 1: Forgetting to Mark Visited Before Recursing

```python
# WRONG: might visit same node multiple times in same path
def dfs(node, visited):
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, visited)
    visited.add(node)  # Too late!

# RIGHT: mark as visited when you first encounter it
def dfs(node, visited):
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(neighbor, visited)
```

#### Mistake 2: Off-by-One in Grid Boundaries

```python
# WRONG: might access out-of-bounds
def dfs(grid, i, j):
    if grid[i][j] == '0':  # What if i or j is negative?
        return

# RIGHT: check bounds first
def dfs(grid, i, j):
    if i < 0 or i >= len(grid) or j < 0 or j >= len(grid[0]):
        return
    if grid[i][j] == '0':
        return
```

#### Mistake 3: Using DFS for Shortest Path

```python
# WRONG: DFS doesn't guarantee shortest path
def shortest_path_dfs(graph, start, end):
    # DFS might find a longer path first

# RIGHT: use BFS for shortest path in unweighted graph
def shortest_path_bfs(graph, start, end):
    queue = deque([(start, 0)])
    visited = {start}
    while queue:
        node, dist = queue.popleft()
        if node == end:
            return dist
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
```

---

### Practice Problems

#### Easy

| # | Problem | Technique |
|---|---------|-----------|
| 1 | [733. Flood Fill](https://leetcode.com/problems/flood-fill/) | DFS/BFS on grid |
| 2 | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | DFS/BFS grid components |
| 3 | [733. Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | DFS grid |
| 4 | [100. Same Tree](https://leetcode.com/problems/same-tree/) | DFS tree |
| 5 | [617. Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/) | DFS tree |

#### Medium

| # | Problem | Technique |
|---|---------|-----------|
| 6 | [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Multi-source BFS |
| 7 | [130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | DFS from borders |
| 8 | [133. Clone Graph](https://leetcode.com/problems/clone-graph/) | DFS/BFS + HashMap |
| 9 | [207. Course Schedule](https://leetcode.com/problems/course-schedule/) | DFS cycle detection |
| 10 | [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Topological sort |
| 11 | [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | DFS components |
| 12 | [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | DFS |
| 13 | [417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | DFS from edges |
| 14 | [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | BFS |

#### Hard

| # | Problem | Technique |
|---|---------|-----------|
| 15 | [127. Word Ladder](https://leetcode.com/problems/word-ladder/) | BFS |
| 16 | [329. Longest Increasing Path in Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) | DFS + Memoization |
| 17 | [773. Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/) | BFS on state space |

---

### Interview Tips

1. **State your choice clearly.** "I'm using BFS because I need the shortest path." or "I'm using DFS because I need to explore all paths."

2. **Mention the visited set.** "I'll track visited nodes to avoid cycles."

3. **For grids, mention the 4-directional movement.** "Each cell has up to 4 neighbors."

4. **Complexity analysis.** "For a grid, we visit each cell at most once, so it's O(m × n)."

5. **Edge cases:**
   - Empty grid/graph
   - Single node
   - Disconnected components
   - No valid path

---

*Next: [10 — Backtracking](10-Backtracking.md)*
