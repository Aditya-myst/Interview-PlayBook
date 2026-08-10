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

```c++
#include <unordered_set>
#include <vector>

// Forward declaration – implement according to your graph structure
std::vector<int> get_neighbors(int node);

void dfs(int node, std::unordered_set<int>& visited) {
    if (visited.find(node) != visited.end()) {
        return;
    }
    
    visited.insert(node);
    
    // Process node here (e.g., print, accumulate, etc.)
    
    for (int neighbor : get_neighbors(node)) {
        dfs(neighbor, visited);
    }
}
```
```Java
import java.util.HashSet;
import java.util.List;

// Assume this method is defined elsewhere
List<Integer> getNeighbors(int node);

void dfs(int node, HashSet<Integer> visited) {
    if (visited.contains(node)) {
        return;
    }
    
    visited.add(node);
    
    // Process node here (e.g., print, accumulate, etc.)
    
    for (int neighbor : getNeighbors(node)) {
        dfs(neighbor, visited);
    }
}
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

```c++
#include <stack>
#include <unordered_set>
#include <vector>

// Assume get_neighbors is defined somewhere
std::vector<int> get_neighbors(int node);

void dfs(int start) {
    std::stack<int> stack;
    stack.push(start);
    std::unordered_set<int> visited;
    
    while (!stack.empty()) {
        int node = stack.top();
        stack.pop();
        
        if (visited.find(node) != visited.end()) {
            continue;
        }
        
        visited.insert(node);
        
        // Process node here
        
        for (int neighbor : get_neighbors(node)) {
            if (visited.find(neighbor) == visited.end()) {
                stack.push(neighbor);
            }
        }
    }
}
```

``` Java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.HashSet;
import java.util.List;

// Assume getNeighbors is defined somewhere
List<Integer> getNeighbors(int node);

void dfs(int start) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);
    HashSet<Integer> visited = new HashSet<>();
    
    while (!stack.isEmpty()) {
        int node = stack.pop();
        
        if (visited.contains(node)) {
            continue;
        }
        
        visited.add(node);
        
        // Process node here
        
        for (int neighbor : getNeighbors(node)) {
            if (!visited.contains(neighbor)) {
                stack.push(neighbor);
            }
        }
    }
}
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

```C++
#include <queue>
#include <vector>
#include <utility>   // for std::pair

std::vector<std::vector<int>> bfs_grid(
    const std::vector<std::vector<int>>& grid,
    int start_row,
    int start_col
) {
    int rows = grid.size();
    int cols = (rows > 0) ? grid[0].size() : 0;

    // Distances: -1 means unvisited / unreachable
    std::vector<std::vector<int>> distance(rows, std::vector<int>(cols, -1));
    std::vector<std::vector<bool>> visited(rows, std::vector<bool>(cols, false));

    // If start is out of bounds or blocked, return empty distances (or handle as needed)
    if (start_row < 0 || start_row >= rows || start_col < 0 || start_col >= cols ||
        grid[start_row][start_col] == 0) {
        return distance;   // or you could throw an exception
    }

    std::queue<std::pair<int, int>> q;
    q.push({start_row, start_col});
    visited[start_row][start_col] = true;
    distance[start_row][start_col] = 0;

    // Four directional moves: up, down, left, right
    const int dr[] = {-1, 1, 0, 0};
    const int dc[] = {0, 0, -1, 1};

    while (!q.empty()) {
        auto [r, c] = q.front();
        q.pop();

        for (int i = 0; i < 4; ++i) {
            int nr = r + dr[i];
            int nc = c + dc[i];

            // Check bounds, visit status, and blocked cell (0)
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                !visited[nr][nc] && grid[nr][nc] != 0) {
                visited[nr][nc] = true;
                distance[nr][nc] = distance[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }

    return distance;
}
```

```Java
import java.util.*;

public class BFSGrid {
    public static int[][] bfsGrid(int[][] grid, int startRow, int startCol) {
        int rows = grid.length;
        int cols = (rows > 0) ? grid[0].length : 0;

        // Distances: -1 means unvisited / unreachable
        int[][] distance = new int[rows][cols];
        boolean[][] visited = new boolean[rows][cols];

        // Initialize distance array with -1
        for (int[] row : distance) {
            Arrays.fill(row, -1);
        }

        // If start is out of bounds or blocked, return (or handle as needed)
        if (startRow < 0 || startRow >= rows || startCol < 0 || startCol >= cols ||
            grid[startRow][startCol] == 0) {
            return distance;
        }

        Queue<int[]> queue = new ArrayDeque<>();
        queue.offer(new int[]{startRow, startCol});
        visited[startRow][startCol] = true;
        distance[startRow][startCol] = 0;

        // Four directional moves: up, down, left, right
        int[] dr = {-1, 1, 0, 0};
        int[] dc = {0, 0, -1, 1};

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int r = cell[0];
            int c = cell[1];

            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                    !visited[nr][nc] && grid[nr][nc] != 0) {
                    visited[nr][nc] = true;
                    distance[nr][nc] = distance[r][c] + 1;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }

        return distance;
    }
}
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

```C++
#include <vector>
#include <unordered_set>

class Solution {
public:
    int numIslands(std::vector<std::vector<char>>& grid) {
        if (grid.empty() || grid[0].empty()) return 0;

        int rows = grid.size();
        int cols = grid[0].size();
        int count = 0;
        std::unordered_set<int> visited; // stores encoded (row,col)

        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                if (grid[i][j] == '1' && visited.find(i * cols + j) == visited.end()) {
                    dfs(grid, i, j, visited, rows, cols);
                    ++count;
                }
            }
        }
        return count;
    }

private:
    void dfs(std::vector<std::vector<char>>& grid,
             int row, int col,
             std::unordered_set<int>& visited,
             int rows, int cols) {
        // Out of bounds, already visited, or water
        if (row < 0 || row >= rows || col < 0 || col >= cols ||
            grid[row][col] == '0' ||
            visited.find(row * cols + col) != visited.end()) {
            return;
        }

        visited.insert(row * cols + col);

        // Four-directional neighbours
        dfs(grid, row - 1, col, visited, rows, cols);
        dfs(grid, row + 1, col, visited, rows, cols);
        dfs(grid, row, col - 1, visited, rows, cols);
        dfs(grid, row, col + 1, visited, rows, cols);
    }
};
```

```Java
import java.util.HashSet;

class Solution {
    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }

        int rows = grid.length;
        int cols = grid[0].length;
        int count = 0;
        HashSet<Integer> visited = new HashSet<>(); // stores encoded (row,col)

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (grid[i][j] == '1' && !visited.contains(i * cols + j)) {
                    dfs(grid, i, j, visited, rows, cols);
                    count++;
                }
            }
        }
        return count;
    }

    private void dfs(char[][] grid,
                     int row, int col,
                     HashSet<Integer> visited,
                     int rows, int cols) {
        // Out of bounds, already visited, or water
        if (row < 0 || row >= rows || col < 0 || col >= cols ||
            grid[row][col] == '0' ||
            visited.contains(row * cols + col)) {
            return;
        }

        visited.add(row * cols + col);

        // Four-directional neighbours
        dfs(grid, row - 1, col, visited, rows, cols);
        dfs(grid, row + 1, col, visited, rows, cols);
        dfs(grid, row, col - 1, visited, rows, cols);
        dfs(grid, row, col + 1, visited, rows, cols);
    }
}
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

```c++
#include <queue>
#include <vector>
#include <algorithm>   // for max

struct Cell {
    int row, col, minute;
};

class Solution {
public:
    int orangesRotting(std::vector<std::vector<int>>& grid) {
        if (grid.empty() || grid[0].empty()) return 0;

        int m = grid.size();
        int n = grid[0].size();
        std::queue<Cell> q;
        int fresh = 0;

        // Initialize: find rotten oranges and count fresh
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 2) {
                    q.push({i, j, 0});
                } else if (grid[i][j] == 1) {
                    ++fresh;
                }
            }
        }

        if (fresh == 0) return 0;

        int maxMinute = 0;
        int dr[] = {-1, 1, 0, 0};
        int dc[] = {0, 0, -1, 1};

        while (!q.empty()) {
            Cell cur = q.front();
            q.pop();
            maxMinute = std::max(maxMinute, cur.minute);

            for (int dir = 0; dir < 4; ++dir) {
                int nr = cur.row + dr[dir];
                int nc = cur.col + dc[dir];

                if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;          // rot the fresh orange
                    --fresh;
                    q.push({nr, nc, cur.minute + 1});
                }
            }
        }

        return (fresh == 0) ? maxMinute : -1;
    }
};
```

```Java

import java.util.ArrayDeque;
import java.util.Queue;

class Solution {
    public int orangesRotting(int[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }

        int m = grid.length;
        int n = grid[0].length;
        Queue<int[]> queue = new ArrayDeque<>(); // stores {row, col, minute}
        int fresh = 0;

        // Initialize: find rotten oranges and count fresh
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 2) {
                    queue.offer(new int[]{i, j, 0});
                } else if (grid[i][j] == 1) {
                    fresh++;
                }
            }
        }

        if (fresh == 0) return 0;

        int maxMinute = 0;
        int[] dr = {-1, 1, 0, 0};
        int[] dc = {0, 0, -1, 1};

        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int row = cur[0], col = cur[1], minute = cur[2];
            maxMinute = Math.max(maxMinute, minute);

            for (int dir = 0; dir < 4; dir++) {
                int nr = row + dr[dir];
                int nc = col + dc[dir];

                if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;          // rot the fresh orange
                    fresh--;
                    queue.offer(new int[]{nr, nc, minute + 1});
                }
            }
        }

        return (fresh == 0) ? maxMinute : -1;
    }
}
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

```c++
# Correct DFS with visited set
#include <unordered_set>
#include <vector>

void dfs_correct(int node, 
                 const std::vector<std::vector<int>>& graph, 
                 std::unordered_set<int>& visited) {
    if (visited.find(node) != visited.end()) {
        return;
    }
    visited.insert(node);
    for (int neighbor : graph[node]) {
        dfs_correct(neighbor, graph, visited);
    }
}

// Usage example:
// std::unordered_set<int> visited;
// dfs_correct(start, graph, visited);

 Incorrect DFS (no visited – infinite loop on cycles)
cpp
#include <vector>


// WARNING: This will recurse infinitely if the graph has a cycle.
void dfs_incorrect(int node, const std::vector<std::vector<int>>& graph) {
    for (int neighbor : graph[node]) {
        dfs_incorrect(neighbor, graph);
    }
}
```

```Java

1. Correct DFS with visited set
java
import java.util.HashSet;
import java.util.List;

public class GraphDFS {
    public static void dfsCorrect(int node, 
                                  List<List<Integer>> graph, 
                                  HashSet<Integer> visited) {
        if (visited.contains(node)) {
            return;
        }
        visited.add(node);
        for (int neighbor : graph.get(node)) {
            dfsCorrect(neighbor, graph, visited);
        }
    }
}

// Usage example:
// HashSet<Integer> visited = new HashSet<>();
// dfsCorrect(start, graph, visited);
2. Incorrect DFS (no visited – infinite loop on cycles)
java
import java.util.List;

public class GraphDFS {
    // WARNING: This will recurse infinitely if the graph has a cycle.
    public static void dfsIncorrect(int node, List<List<Integer>> graph) {
        for (int neighbor : graph.get(node)) {
            dfsIncorrect(neighbor, graph);
        }
    }
}

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

```c++
#include <vector>

void dfs(std::vector<std::vector<char>>& grid, int i, int j) {
    int rows = grid.size();
    int cols = grid[0].size();

    // Bounds and land check
    if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] != '1') {
        return;
    }

    // Mark as visited
    grid[i][j] = '#';

    // Four-directional neighbours
    const int di[] = {-1, 1, 0, 0};
    const int dj[] = {0, 0, -1, 1};

    for (int dir = 0; dir < 4; ++dir) {
        dfs(grid, i + di[dir], j + dj[dir]);
    }
}

```
```Java
class Solution {
    public void dfs(char[][] grid, int i, int j) {
        int rows = grid.length;
        int cols = grid[0].length;

        // Bounds and land check
        if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] != '1') {
            return;
        }

        // Mark as visited
        grid[i][j] = '#';

        // Four-directional neighbours
        int[] di = {-1, 1, 0, 0};
        int[] dj = {0, 0, -1, 1};

        for (int dir = 0; dir < 4; dir++) {
            dfs(grid, i + di[dir], j + dj[dir]);
        }
    }
}
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

```c++
 Incorrect: marks visited too late
cpp
#include <unordered_set>
#include <vector>

// WRONG: node is added to 'visited' only after exploring all neighbours.
// In a cyclic graph, the same node can be reached from different paths
// before it is marked, causing repeated processing.
void dfs_wrong(int node,
               const std::vector<std::vector<int>>& graph,
               std::unordered_set<int>& visited) {
    for (int neighbor : graph[node]) {
        if (visited.find(neighbor) == visited.end()) {
            dfs_wrong(neighbor, graph, visited);
        }
    }
    visited.insert(node);   // Too late! Node was already processed multiple times.
}
✅ Correct: marks before exploring
cpp
#include <unordered_set>
#include <vector>

// RIGHT: mark the current node as visited immediately upon entry.
// This ensures each node is processed exactly once, even in cyclic graphs.
void dfs_correct(int node,
                 const std::vector<std::vector<int>>& graph,
                 std::unordered_set<int>& visited) {
    visited.insert(node);   // Mark before recursing
    for (int neighbor : graph[node]) {
        if (visited.find(neighbor) == visited.end()) {
            dfs_correct(neighbor, graph, visited);
        }
    }
}
```

```Java
 Incorrect: marks visited too late
java
import java.util.HashSet;
import java.util.List;

public class DFS {
    // WRONG: node is added to 'visited' only after exploring all neighbours.
    // In a cyclic graph, the same node can be reached from different paths
    // before it is marked, causing repeated processing.
    public static void dfsWrong(int node,
                                List<List<Integer>> graph,
                                HashSet<Integer> visited) {
        for (int neighbor : graph.get(node)) {
            if (!visited.contains(neighbor)) {
                dfsWrong(neighbor, graph, visited);
            }
        }
        visited.add(node);   // Too late! Node was already processed multiple times.
    }
}
✅ Correct: marks before exploring
java
import java.util.HashSet;
import java.util.List;

public class DFS {
    // RIGHT: mark the current node as visited immediately upon entry.
    // This ensures each node is processed exactly once, even in cyclic graphs.
    public static void dfsCorrect(int node,
                                  List<List<Integer>> graph,
                                  HashSet<Integer> visited) {
        visited.add(node);   // Mark before recursing
        for (int neighbor : graph.get(node)) {
            if (!visited.contains(neighbor)) {
                dfsCorrect(neighbor, graph, visited);
            }
        }
    }
}
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

```c++
C++ Versions
❌ Incorrect: out‑of‑bounds access possible
cpp
#include <vector>

// WRONG: This accesses grid[i][j] before verifying i and j are valid.
// If i or j is negative or out of range, the program may crash or behave unpredictably.
void dfs_wrong(std::vector<std::vector<char>>& grid, int i, int j) {
    if (grid[i][j] == '0') {  // DANGER: i or j may be out of bounds!
        return;
    }
    // Process land...
}
✅ Correct: check bounds first
cpp
#include <vector>

// RIGHT: Always validate indices before accessing the grid.
void dfs_correct(std::vector<std::vector<char>>& grid, int i, int j) {
    int rows = grid.size();
    int cols = grid[0].size();

    if (i < 0 || i >= rows || j < 0 || j >= cols) {
        return;  // Out of bounds – safe exit
    }
    if (grid[i][j] == '0') {
        return;  // Water – stop exploring
    }
    // Now it's safe to process land ('1')...
}
```

```jAVA
Java Versions
❌ Incorrect: out‑of‑bounds access possible
java
class Solution {
    // WRONG: This accesses grid[i][j] before checking bounds.
    // If i or j is out of range, an ArrayIndexOutOfBoundsException will be thrown.
    public void dfsWrong(char[][] grid, int i, int j) {
        if (grid[i][j] == '0') {  // DANGER: i or j may be invalid!
            return;
        }
        // Process land...
    }
}
✅ Correct: check bounds first
java
class Solution {
    // RIGHT: Always validate indices before accessing the grid.
    public void dfsCorrect(char[][] grid, int i, int j) {
        int rows = grid.length;
        int cols = grid[0].length;

        if (i < 0 || i >= rows || j < 0 || j >= cols) {
            return;  // Out of bounds – safe exit
        }
        if (grid[i][j] == '0') {
            return;  // Water – stop exploring
        }
        // Now it's safe to process land ('1')...
    }
}
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

```❌ WRONG: DFS (does NOT guarantee shortest path)
This implementation returns the distance of the first path it finds to end. Because DFS explores deeply before backtracking, that path may be longer than the shortest one.

C++
cpp
#include <unordered_set>
#include <vector>
#include <climits>

// Returns distance from 'node' to 'end', or -1 if not found.
// NOT GUARANTEED SHORTEST – it may return a longer path.
int dfs_wrong(int node, int end, int depth,
              const std::vector<std::vector<int>>& graph,
              std::unordered_set<int>& visited) {
    if (node == end) return depth;

    visited.insert(node);

    for (int neighbor : graph[node]) {
        if (visited.find(neighbor) == visited.end()) {
            int result = dfs_wrong(neighbor, end, depth + 1, graph, visited);
            if (result != -1) return result;  // returns on first found path
        }
    }
    return -1;
}

// Wrapper
int shortest_path_dfs_wrong(int start, int end, const std::vector<std::vector<int>>& graph) {
    std::unordered_set<int> visited;
    return dfs_wrong(start, end, 0, graph, visited);
}
Java
java
import java.util.HashSet;
import java.util.List;

public class GraphDFSWrong {
    // Returns distance from 'node' to 'end', or -1 if not found.
    // NOT GUARANTEED SHORTEST – it may return a longer path.
    private static int dfsWrong(int node, int end, int depth,
                                List<List<Integer>> graph,
                                HashSet<Integer> visited) {
        if (node == end) return depth;

        visited.add(node);

        for (int neighbor : graph.get(node)) {
            if (!visited.contains(neighbor)) {
                int result = dfsWrong(neighbor, end, depth + 1, graph, visited);
                if (result != -1) return result;  // returns on first found path
            }
        }
        return -1;
    }

    public static int shortestPathDfsWrong(int start, int end, List<List<Integer>> graph) {
        HashSet<Integer> visited = new HashSet<>();
        return dfsWrong(start, end, 0, graph, visited);
    }
}
✅ RIGHT: BFS for shortest path in unweighted graph
BFS processes nodes in order of increasing distance, so the first time we reach end we know it is the shortest distance.

C++
cpp
#include <queue>
#include <unordered_set>
#include <vector>
#include <utility>

int shortest_path_bfs(int start, int end, const std::vector<std::vector<int>>& graph) {
    std::queue<std::pair<int, int>> q;  // {node, distance}
    std::unordered_set<int> visited;

    q.push({start, 0});
    visited.insert(start);

    while (!q.empty()) {
        auto [node, dist] = q.front();
        q.pop();

        if (node == end) return dist;

        for (int neighbor : graph[node]) {
            if (visited.find(neighbor) == visited.end()) {
                visited.insert(neighbor);
                q.push({neighbor, dist + 1});
            }
        }
    }
    return -1;  // no path found
}
Java
java
import java.util.*;

public class GraphBFS {
    public static int shortestPathBfs(int start, int end, List<List<Integer>> graph) {
        Queue<int[]> queue = new ArrayDeque<>();  // stores {node, distance}
        HashSet<Integer> visited = new HashSet<>();

        queue.offer(new int[]{start, 0});
        visited.add(start);

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int node = curr[0];
            int dist = curr[1];

            if (node == end) return dist;

            for (int neighbor : graph.get(node)) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    queue.offer(new int[]{neighbor, dist + 1});
                }
            }
        }
        return -1;  // no path found
    }
}

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
